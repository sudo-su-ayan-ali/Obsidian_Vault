### 1. ⚡ Missing & Incomplete Supabase Edge Functions  
  
 1. requestDoctorPayout Endpoint (Specified in api-specification.md:36):  
     • Needs an Edge Function or Next.js API route to verify doctor earnings in the consultation ledger,  
     validate bank account / IFSC details, and initiate automated transfers (e.g., via Razorpay Route /  
     Payouts API).  
     • Currently, doctor payouts and financial transfers are simulated using in-memory mock data in  
     finance-data.ts.  
 2. refund-appointment / Dispute Resolution Endpoint:  
     • Endpoint to process appointment cancellations, trigger automated Razorpay refunds, log refund ARN  
     numbers, and release doctor slots.  
 3. RAG Medical AI Pipeline (AI-601):  
     • Backend service (using LangChain / Gemini / OpenAI embeddings) indexed with verified clinical AYUSH  
     & Allopathy guidelines to power the patient prescription Q&A ("Ask AI" button).  
 4. Prescription OCR Ingestion Pipeline (ROADMAP Phase 2):  
     • Serverless document-processing function to ingest scanned paper prescription photos, run OCR (e.g.,  
     Google Cloud Vision), and extract structured medication, dosage, and frequency items.  
  
 ──────  
### 2. 🗄 Database Schemas & PostgreSQL Migrations Remaining  
  
 While core tables (users, patients, doctors, doctor_slots, appointments, prescriptions, medical_records,  
 doctor_kyc_documents, device_tokens, notifications_queue) are fully migrated, the following database  
 structures are not yet created in migrations:  
  
 1. public.patient_vitals (Time-Series Health Log):  
     • Current State: Vitals (Blood Pressure, Blood Sugar, Heart Rate, SpO2, Weight & BMI) entered via the +  
     Log Vital modal are only saved locally in the Flutter widget's in-memory setState().  
     • Remaining: PostgreSQL schema with RLS policies to record historical vitals timestamps, enabling  
     longitudinal graphs and doctor access during teleconsultations.  
 2. public.doctor_reviews Table & Aggregation Trigger:  
     • Current State: public.doctors stores rating and review_count, and the Flutter app features a  
     PostConsultationRatingModal.  
     • Remaining: A dedicated public.doctor_reviews table storing individual review records (appointment_id,  
     patient_id, doctor_id, rating, review_text) with a PostgreSQL trigger to automatically recalculate the  
     doctor's average rating and total review count.  
 3. public.financial_transactions & public.payouts:  
     • Current State: The Next.js Admin portal finance dashboard uses mock data.  
     • Remaining: Relational ledger tables tracking platform commission (15–20%), GST deductions,  
     settlement states (pending_escrow, settled, refunded), and bank UTR numbers.  
  
 ──────  
### 3. ⏰ Background Automation & Cron Jobs (pg_cron)  
  
 4. Slot Lock Auto-Expiry Cleaner:  
     • Currently, if a patient locks a slot and abandons checkout without triggering back navigation, the  
     slot is only released on the next query after 10 minutes.  
     • Remaining: A periodic pg_cron worker or Edge Function to reset expired locks (is_locked = false,  
     locked_by = NULL WHERE locked_until < NOW()) and emit a slot_unlocked event over Supabase Realtime  
     broadcast so doctor availability updates immediately for all users.  
 5. Scheduled Notifications Worker:  
     • While dispatch-notifications handles push and SMS dispatch, an automated schedule (e.g., pg_cron or  
     Supabase Scheduled Function running every minute) is needed to query notifications_queue and trigger  
     15-minute pre-call reminders automatically.  
  
 ──────  
### 4. 🌐 Production Deployment & External Gateway Bindings  
  
 The existing 5 Edge Functions are fully written locally, but the following deployment and configuration  
 tasks remain:  
  
 6. Supabase Cloud Remote Deployment:  
     • Run supabase functions deploy to push the 5 Edge Functions to Supabase Cloud Mumbai (ap-south-1):  
         • lock-appointment-slot  
         • razorpay-webhook  
         • generate-agora-token  
         • generate-prescription-pdf  
         • dispatch-notifications  
  
 7. Production Secrets Configuration (supabase secrets set):  
     • Razorpay: Configure live keys (RAZORPAY_KEY_ID, RAZORPAY_KEY_SECRET, RAZORPAY_WEBHOOK_SECRET) and  
     register the production webhook URL in the Razorpay Dashboard.  
     • Agora RTC: Set live production AGORA_APP_ID and AGORA_APP_CERTIFICATE.  
     • MSG91 DLT: Set production MSG91_AUTH_KEY linked to registered TRAI DLT Entity ID and templates  
     (1207168940001–1207168940004).  
     • OneSignal: Set production ONESIGNAL_APP_ID and ONESIGNAL_REST_API_KEY.  
 8. ABDM / ABHA Sandbox & Gateway Integration (ABHA-106):  
     • Connect backend endpoints to the official National Health Authority (NHA) ABDM Sandbox for:  
         • Milestone 1: ABHA Number creation & Orification via Aadhaar.  
         • Milestone 2: HIP (Health Information Provider) record linking.  
         • Milestone 3: HIU (Health Information User) consent management.