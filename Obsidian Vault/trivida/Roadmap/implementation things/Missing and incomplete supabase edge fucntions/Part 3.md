### 3. ⏰ Background Automation & Cron Jobs (pg_cron) Deep-Dive  
  
 In high-concurrency telemedicine platforms, user flows are asynchronous and prone to sudden drop-offs (e.g., app termination during payment, unstable rural 4G, or app backgrounding). Relying on passive client-triggered logic  
 creates stale locks and missed clinical notifications.  
  
 Here is the in-depth architectural breakdown of the two critical background workers:  
 ──────  
 ## 4. Slot Lock Auto-Expiry Cleaner  
  
 ### A. The Real-World Clinical Problem  
  
 When a patient initiates booking:  
 1. The app invokes lock_appointment_slot(p_slot_id, p_user_id, p_duration_minutes => 10) in PostgreSQL.  
 2. The slot is marked with is_locked = TRUE, locked_by = <patient_uuid>, and locked_until = NOW() + INTERVAL '10 minutes'.  
 3. If the patient closes the app, experiences battery death, or abandons the Razorpay checkout screen without tapping "Cancel":  
     • The slot remains locked for 10 full minutes.  
     • The Passive Problem: Currently, the lock is only cleared if another patient attempts to query or lock that exact slot after NOW() > locked_until.  
     • In peak hours, other patients visiting that doctor's schedule will see the slot marked "Unavailable" or "Booked", preventing legitimate consultations and reducing doctor utilization.  
     • Even when the 10 minutes expire, other users' screens do not know the slot has freed up unless they perform a manual pull-to-refresh.  
 ──────  
 ### B. The Production Solution: Active pg_cron Cleaner + Supabase Realtime Broadcast  
 Instead of passive discovery, a scheduled worker runs every 60 seconds to:  
  
 4. Find all slots where is_locked = TRUE AND is_booked = FALSE AND locked_until < NOW().  
 5. Reset them to is_locked = FALSE, locked_by = NULL, locked_until = NULL.  
 6. Emit a Realtime broadcast event (slot_unlocked) on the channel doctor_slots_<doctor_id> so any patient currently viewing the doctor's calendar sees the slot immediately flip back to available green without refreshing.  
  
   ┌──────────────────────────┐┌─────────────────────┐   ┌─────────────────────┐      ┌──────────────────────────┐      ┌───────────────────────────────┐      ┌───────────┐  
   │ Patient A (Abandons App) ││ pg_cron (Every 60s) │   │ public.doctor_slots │      │ Supabase Realtime Engine │      │ Patient B (Browsing Schedule) │      │ Realtime- │  
   └──────────────────────────┘└─────────────────────┘   └─────────────────────┘      └──────────────────────────┘      └───────────────────────────────┘      └───────────┘  
                 │                        │                         │                               │                                   │                            │  
                 │───────Lock Slot (locked_until = 12:10 PM)────────►                               │                                   │                            │  
                 │                        │                         │                               │                                   │                            │  
                 │                        │EXECUTE clean_expired_slo►_locks()                       │                                   │                            │  
                 │                        │                         │                               │                                   │                            │  
                 │                        │                         │ UPDATE doctor_slots SET is_locked = false WHERE locked_until < NOW() ─┐                        │  
                 │                        │                         │◄─┘                            │                                   │                            │  
                 │                        │                         │                               │                                   │                            │  
                 │                        │                         │pg_notify or Supabase Realtime ►roadcast ('slot_unlocked')         │                            │  
                 │                        │                         │                               │                                   │                            │  
                 │                        │                         │                               │                                   ◄Live UI Update (Slot flips from Locked t  
                 │                        │                         │                               │                                   │                            │  
  
 #### SQL Implementation Architecture (pg_cron):  
  
   -- 1. Cleaner Function with Returning Doctor & Slot IDs  
   CREATE OR REPLACE FUNCTION public.clean_expired_slot_locks()  
   RETURNS TABLE (  
       unlocked_slot_id UUID,  
       doctor_id UUID  
   ) AS $$  
   BEGIN  
       RETURN QUERY  
       UPDATE public.doctor_slots s  
       SET  
           is_locked = FALSE,  
           locked_by = NULL,  
           locked_until = NULL,  
           updated_at = NOW()  
       WHERE  
           s.is_locked = TRUE  
           AND s.is_booked = FALSE  
           AND s.locked_until < NOW()  
       RETURNING s.id, s.doctor_id;  
   END;  
   $$ LANGUAGE plpgsql SECURITY DEFINER;  
  
   -- 2. Schedule via pg_cron every minute  
   SELECT cron.schedule(  
       'clean-expired-doctor-slot-locks',  
       '* * * * *', -- every minute  
       $$SELECT public.clean_expired_slot_locks();$$  
   );  
  
 #### Realtime Integration:  
  
 When triggered via a lightweight Supabase Edge Function (cron-slot-cleaner) or PostgreSQL supabase_functions.http_request:  
  
   // For each unlocked slot, broadcast to doctors' slot channel  
   await supabase.channel(`doctor_slots_${doctor_id}`).send({  
     type: "broadcast",  
     event: "slot_released",  
     payload: { slotId, doctorId, status: "available" }  
   });  
 ──────  
 ## 5. Scheduled Notifications Worker (notifications_queue)  
  
 ### A. The Clinical & Patient Adherence Problem  
  
 In telemedicine, no-shows are the primary cause of lost doctor revenue and disrupted care schedules.  
  
 • Trividha's notifications_queue table stores scheduled communications (SMS, WhatsApp, FCM Push) with a scheduled_for timestamp.  
 • Examples of critical scheduled notifications:  
     7. T-15 Minute Pre-Call Alert: "Your video consultation with Dr. Sharma begins in 15 minutes. Tap here to test your camera and enter the waiting room."  
     8. Medication Adherence Reminders: Morning/Night dose reminders generated from the Tri-Fold prescription.  
     9. Doctor Joining Notification: Sent when the doctor enters the Agora video room.  
 • The Gap: Without a background cron runner, rows placed into notifications_queue with future timestamps (scheduled_for = appointment.scheduled_at - INTERVAL '15 minutes') sit in the database unprocessed unless triggered by an  
 explicit external HTTP ping.  
 ──────  
 ### B. The Production Solution: 1-Minute Cron Dispatch Pipeline  
  
 ┌────────────────────────────────────────────────────────────┐  
 │                                                            │  
 │ pg_cron / GitHub Actions / Cloudflare Worker (Every 1 Min) │  
 │                                                            │  
 └──────────────────────────────┬─────────────────────────────┘  
             POST Authorization:│Bearer SERVICE_ROLE  
                                ▼  
 ┌────────────────────────────────────────────────────────────┐  
 │                                                            │  
 │         Edge Function: cron-dispatch-notifications         │  
 │                                                            │  
 └──────────────────────────────┬─────────────────────────────┘  
                                │  
                                ▼  
 ┌────────────────────────────────────────────────────────────┐  
 │                                                            │  
 │                 Query notifications_queue                  │  
 │                                                            │  
 │                  WHERE status = 'pending'                  │  
 │                                                            │  
 │                 AND scheduled_for <= NOW()                 │  
 │                                                            │  
 └──────────────────────────────┬─────────────────────────────┘  
                                │  
                                ▼  
 ┌────────────────────────────────────────────────────────────┐  
 │                                                            │  
 │           Batch Process (Up to 50 notifications)           ├───┬─────────────────────────┐  
 │                                                            │   │                         │  
 └──────────────────────────────┬─────────────────────────────┘   └─────────────────────────┼───────────────────────────────────────────────────┐  
                                │                                                           │                                                   │  
                                ▼                                                           ▼                                                   ▼  
 ┌────────────────────────────────────────────────────────────┐   ┌──────────────────────────────────────────────────┐   ┌─────────────────────────────────────────────┐  
 │                                                            │   │                                                  │   │                                             │  
 │             Google FCM v1 (Push Notifications)             │   │ MSG91 / Twilio (SMS Fallback for Feature Phones) │   │ WhatsApp Business API (Prescription alerts) │  
 │                                                            │   │                                                  │   │                                             │  
 └──────────────────────────────┬─────────────────────────────┘   └─────────────────────────┬────────────────────────┘   └──────────────────────┬──────────────────────┘  
                                │                                                           │                                                   │  
                                ▼                                                           │                                                   │  
 ┌────────────────────────────────────────────────────────────┐                             │                                                   │  
 │                                                            │                             │                                                   │  
 │                 UPDATE notifications_queue                 │                             │                                                   │  
 │                                                            │                             │                                                   │  
 │            SET status = 'sent', sent_at = NOW()            │◄────────────────────────────┴───────────────────────────────────────────────────┘  
 │                                                            │  
 │           (Or status = 'failed', retry_count++)            │  
 │                                                            │  
 └────────────────────────────────────────────────────────────┘  
  
 #### Core Components:  
  
 10. Enqueueing Phase (Trigger or Booking Completion):  
 When an appointment is booked and confirmed:  
   INSERT INTO public.notifications_queue (  
       user_id,  
       notification_type,  
       channel,  
       title,  
       body,  
       scheduled_for,  
       metadata  
   ) VALUES (  
       new_appointment.patient_id,  
       'APPOINTMENT_REMINDER_15MIN',  
       'PUSH',  
       'Teleconsultation in 15 minutes',  
       'Dr. Sharma will be online shortly. Tap to join the waiting room.',  
       new_appointment.scheduled_at - INTERVAL '15 minutes',  
       jsonb_build_object('appointment_id', new_appointment.id)  
   );  
  
 11. Cron Dispatcher Phase (Runs Every 60s):  
 A lightweight Edge Function or pg_cron job selects rows due for dispatch using row-locking (FOR UPDATE SKIP LOCKED) to ensure horizontal scalability with zero duplicate messages:  
   CREATE OR REPLACE FUNCTION public.fetch_due_notifications(batch_size INT DEFAULT 50)  
   RETURNS TABLE (  
       id UUID,  
       user_id UUID,  
       notification_type VARCHAR,  
       channel VARCHAR,  
       title TEXT,  
       body TEXT,  
       metadata JSONB  
   ) AS $$  
   BEGIN  
       RETURN QUERY  
       UPDATE public.notifications_queue n  
       SET  
           status = 'processing',  
           updated_at = NOW()  
       WHERE n.id IN (  
           SELECT sub.id  
           FROM public.notifications_queue sub  
           WHERE sub.status = 'pending'  
             AND sub.scheduled_for <= NOW()  
           ORDER BY sub.scheduled_for ASC  
           LIMIT batch_size  
           FOR UPDATE SKIP LOCKED  
       )  
       RETURNING n.id, n.user_id, n.notification_type, n.channel, n.title, n.body, n.metadata;  
   END;  
   $$ LANGUAGE plpgsql SECURITY DEFINER;  
  
 12. Multi-Channel Fallback:  
 If the patient has no active FCM device token or if push delivery fails, the dispatcher automatically falls back to transactional SMS (essential for rural patients in Rajasthan and Gujarat with intermittent data connectivity).  
 ──────  
 ### Comparison of Hosting Options for pg_cron  
  
  Mechanism                                            | Pros                                                     | Cons                                                     | Best Used For  
 ------------------------------------------------------|----------------------------------------------------------|----------------------------------------------------------|----------------------------------------------------------  
  PostgreSQL pg_cron (Native Supabase)                 | Runs directly inside database engine; zero network       | Cannot execute heavy external HTTP requests or third-    | Slot Lock Expiry Cleaner (pure SQL updates + NOTIFY).  
                                                       | latency; handles row updates and triggers in microsecond | party SDKs directly without pg_net.                      |  
                                                       | time.                                                    |                                                          |  
  Supabase Scheduled Edge Function (edge-runtime cron) | Full TypeScript ecosystem; access to FCM SDK,            | Slight execution overhead; subject to Edge Function cold | Scheduled Notifications Dispatcher (external push/SMS  
                                                       | Twilio/MSG91 REST APIs, and Supabase client libraries.   | starts.                                                  | delivery).