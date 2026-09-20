Here is an in-depth, architectural breakdown of the 3 remaining database schemas and PostgreSQL migrations, explaining why they are needed, how the current codebase behaves, and how the production schemas should be designed.  
 ──────  
   ┌──────────────────────────────────────────────────────────────────────────────────┐  
   │                   TRIVIDHA BACKEND PERSISTENCE & DATA PIPELINES                  │  
   ├──────────────────────────────────────────────────────────────────────────────────┤  
   │                                                                                  │  
   │   1. Time-Series Health Vitals      2. Doctor Review Engine     3. Financial     │  
   │             (Telemetry)                    (Trust & Quality)        Ledger (RBI) │  
   │                                                                                  │  
   │      📱 Patient Dashboard               ⭐ Post-Call Modal        💳 RazorpayX   │  
   │      [+ Log Vitals Modal]               [Submit 5-Star]           Admin Finance  │  
   │               │                                │                        │        │  
   │        (Currently in-memory)             (No review table)        (Mocked state) │  
   │               ▼                                ▼                        ▼        │  
   │     ┌───────────────────┐            ┌───────────────────┐    ┌────────────────┐ │  
   │     │ patient_vitals    │            │ doctor_reviews    │    │ financial_     │ │  
   │     │ • BP, Sugar, SpO2 │            │ • Text, Stars     │    │ transactions   │ │  
   │     │ • Time-Series Log │            │ • Auto-Trigger    │    │ • 85/15 Splits │ │  
   │     │ • Doctor Pre-Call │            │   to doctors.rate │    │ • GST & Escrow │ │  
   │     └───────────────────┘            └───────────────────┘    └────────────────┘ │  
   └──────────────────────────────────────────────────────────────────────────────────┘  
 ──────  
## 1. public.patient_vitals (Time-Series Health Log)  
  
 ### A. The Current State  
  
 In the patient mobile app (patient_home_dashboard.dart:2017-2070), there is a "+ Log Vital" button that opens an update dialog (_showLogVitalDialog). When the user inputs their vitals, the app runs:  
  
   // Current implementation: purely ephemeral in-memory state  
   setState(() {  
     _bp = bpCtrl.text.trim();  
     _sugar = sugarCtrl.text.trim();  
     _pulse = pulseCtrl.text.trim();  
     _spo2 = spo2Ctrl.text.trim();  
     _weight = weightCtrl.text.trim();  
   });  
  
 ### B. The Clinical & Architectural Problem  
  
 • Data Loss on Restart: Because this state lives only in the widget's ephemeral memory, closing or refreshing the app completely wipes the patient's entries.  
 • No Longitudinal Clinical Trend: In chronic conditions like Hypertension (उच्च रक्तचाप) and Type-2 Diabetes (मधुमेह), a single isolated reading is medically insufficient. Doctors need to see blood pressure or fasting sugar variations  
 over weeks to adjust medication dosages.  
 • Invisible to Doctors in Teleconsultation: When a doctor enters the Agora video room, they cannot see the patient's pre-call vitals history to inform their clinical diagnosis.  
  
 ### C. Proposed PostgreSQL Production Schema  
  
 A dedicated time-series table tracking standardized numeric values:  
  
   CREATE TABLE public.patient_vitals (  
       id UUID PRIMARY KEY DEFAULT gen_random_uuid(),  
       patient_id UUID NOT NULL REFERENCES public.patients(id) ON DELETE CASCADE,  
       appointment_id UUID REFERENCES public.appointments(id) ON DELETE SET NULL,  
       systolic_bp INT CHECK (systolic_bp BETWEEN 50 AND 300),  
       diastolic_bp INT CHECK (diastolic_bp BETWEEN 30 AND 200),  
       blood_sugar_mg_dl NUMERIC(5, 1) CHECK (blood_sugar_mg_dl BETWEEN 20.0 AND 800.0),  
       sugar_context VARCHAR(20) CHECK (sugar_context IN ('fasting', 'post_prandial', 'random')),  
       heart_rate_bpm INT CHECK (heart_rate_bpm BETWEEN 30 AND 250),  
       spo2_percentage INT CHECK (spo2_percentage BETWEEN 50 AND 100),  
       weight_kg NUMERIC(5, 2) CHECK (weight_kg BETWEEN 2.0 AND 400.0),  
       height_cm NUMERIC(5, 2) CHECK (height_cm BETWEEN 30.0 AND 250.0),  
       recorded_at TIMESTAMPTZ NOT NULL DEFAULT timezone('utc'::text, now()),  
       created_at TIMESTAMPTZ NOT NULL DEFAULT timezone('utc'::text, now())  
   );  
  
   -- Compound index for fast time-series graph querying:  
   CREATE INDEX idx_patient_vitals_timeline  
   ON public.patient_vitals(patient_id, recorded_at DESC);  
  
 ### D. Row-Level Security (RLS) Policy  
  
 • Patient Self-Access: A patient can view and insert their own vitals.  
 • Doctor Attending Access: A verified doctor can read vitals for a patient only if they have an assigned appointment together.  
 ──────  
## 2. public.doctor_reviews Table & Atomic Aggregation Trigger  
  
 ### A. The Current State  
  
 • The 20260901_initial_schema.sql:115 table already defines summary columns:  
   rating NUMERIC(3, 2) DEFAULT 5.00 CHECK (rating >= 0 AND rating <= 5.0),  
   review_count INT DEFAULT 0 CHECK (review_count >= 0)  
  
 • In Flutter, post_consultation_rating_modal.dart renders a 5-star rating selector and a feedback input field.  
 • However, there is no table in the database to persist individual reviews or patient feedback text.  
  
 ### B. The Integrity & Trust Problem  
  
 1. No Audit Trail: Patients cannot read text reviews left by previous patients, nor can clinic administrators audit complaints or negative reviews.  
 2. Vulnerability to Client-Side Manipulation: If the client calculates the doctor's average rating in JavaScript/Dart and updates doctors.rating directly, it is prone to race conditions, double-counting, or malicious rating  
 manipulation.  
 3. No Consultation Verification: Anyone could write arbitrary reviews unless verified against a completed consultation (appointment_id).  
  
 ### C. Proposed Production Schema & Trigger Architecture  
  
   CREATE TABLE public.doctor_reviews (  
       id UUID PRIMARY KEY DEFAULT gen_random_uuid(),  
       appointment_id UUID NOT NULL UNIQUE REFERENCES public.appointments(id) ON DELETE CASCADE,  
       patient_id UUID NOT NULL REFERENCES public.patients(id) ON DELETE CASCADE,  
       doctor_id UUID NOT NULL REFERENCES public.doctors(id) ON DELETE CASCADE,  
       rating INT NOT NULL CHECK (rating BETWEEN 1 AND 5),  
       review_text TEXT,  
       is_verified_consultation BOOLEAN NOT NULL DEFAULT true,  
       created_at TIMESTAMPTZ NOT NULL DEFAULT timezone('utc'::text, now()),  
       updated_at TIMESTAMPTZ NOT NULL DEFAULT timezone('utc'::text, now())  
   );  
  
   CREATE INDEX idx_doctor_reviews_doctor ON public.doctor_reviews(doctor_id, rating);  
  
 ### D. Database Trigger for Automatic Rating Aggregation  
  
 Instead of trusting frontend code to calculate averages, an atomic PostgreSQL trigger recalculates the doctor's average rating and review count whenever a review is added, updated, or removed:  
  
   CREATE OR REPLACE FUNCTION public.recalculate_doctor_rating()  
   RETURNS TRIGGER AS $$  
   DECLARE  
       target_doctor_id UUID;  
   BEGIN  
       target_doctor_id := COALESCE(NEW.doctor_id, OLD.doctor_id);  
  
       UPDATE public.doctors  
       SET  
           rating = COALESCE((  
               SELECT ROUND(AVG(rating)::numeric, 2)  
               FROM public.doctor_reviews  
               WHERE doctor_id = target_doctor_id  
           ), 5.00),  
           review_count = (  
               SELECT COUNT(*)  
               FROM public.doctor_reviews  
               WHERE doctor_id = target_doctor_id  
           ),  
           updated_at = NOW()  
       WHERE id = target_doctor_id;  
  
       RETURN NEW;  
   END;  
   $$
    LANGUAGE plpgsql SECURITY DEFINER;  
  
   CREATE TRIGGER trg_recalculate_doctor_rating  
   AFTER INSERT OR UPDATE OR DELETE ON public.doctor_reviews  
   FOR EACH ROW EXECUTE FUNCTION public.recalculate_doctor_rating();  
 ──────  
## 3. public.financial_transactions & Double-Entry Ledger Architecture  
  
 ### A. The Current State  
  
 • The Next.js Admin portal finance dashboard ((dashboard/finance/page.tsx)) was initially using mock data (INITIAL_TRANSACTIONS in finance-data.ts).  
 • While we recently implemented public.doctor_payouts and public.appointment_refunds for the outward transfers, the central financial transactions ledger that connects incoming payments from Razorpay to platform fees and escrow  
 obligations needs full database schema unification.  
  
 ### B. The Legal & Regulatory Problem (RBI Payment Aggregator Norms)  
  
 Under the Reserve Bank of India (RBI) Guidelines on Regulation of Payment Aggregators and Payment Gateways (2020/2023):  
  
 4. No Co-mingling of Funds: Marketplace platforms like Trividha cannot pool doctor earnings directly into their own operational current accounts. Funds must sit in an escrow/nodal settlement pipeline.  
 5. The 85/15 Fee Split: On every ₹499 consultation:  
     • Gross Payment: ₹499.00  
     • Platform Take-Rate (15%): ₹74.85  
     • GST Liability (18% on platform take-rate): ₹13.47  
     • Doctor Payable (Escrow): ₹424.15  
 6. Dispute Audit Trail: Every transaction must trace backward to the Razorpay Payment ID (pay_...), Razorpay Order ID (order_...), the appointment ID, and forward to the IMPS bank UTR upon settlement.  
  
 ### C. Proposed Relational Ledger Schema  
  
   CREATE TABLE public.financial_transactions (  
       id UUID PRIMARY KEY DEFAULT gen_random_uuid(),  
       appointment_id UUID NOT NULL REFERENCES public.appointments(id) ON DELETE RESTRICT,  
       patient_id UUID NOT NULL REFERENCES public.patients(id) ON DELETE RESTRICT,  
       doctor_id UUID NOT NULL REFERENCES public.doctors(id) ON DELETE RESTRICT,  
       razorpay_payment_id VARCHAR(100) NOT NULL UNIQUE,  
       razorpay_order_id VARCHAR(100) NOT NULL,  
       gross_amount NUMERIC(10, 2) NOT NULL CHECK (gross_amount > 0),  
       platform_fee NUMERIC(10, 2) NOT NULL DEFAULT 0.00,  
       gst_amount NUMERIC(10, 2) NOT NULL DEFAULT 0.00,  
       doctor_payout_amount NUMERIC(10, 2) NOT NULL CHECK (doctor_payout_amount >= 0),  
       payment_method VARCHAR(50) NOT NULL DEFAULT 'UPI',  
       payment_status VARCHAR(30) NOT NULL DEFAULT 'captured'  
           CHECK (payment_status IN ('captured', 'refunded', 'failed')),  
       settlement_status VARCHAR(30) NOT NULL DEFAULT 'pending_escrow'  
           CHECK (settlement_status IN ('pending_escrow', 'settled', 'refund_processed')),  
       payout_id UUID REFERENCES public.doctor_payouts(id) ON DELETE SET NULL,  
       refund_id UUID REFERENCES public.appointment_refunds(id) ON DELETE SET NULL,  
       created_at TIMESTAMPTZ NOT NULL DEFAULT timezone('utc'::text, now()),  
       updated_at TIMESTAMPTZ NOT NULL DEFAULT timezone('utc'::text, now())  
   );  
  
   CREATE INDEX idx_financial_transactions_doctor  
   ON public.financial_transactions(doctor_id, settlement_status);  
  
   CREATE INDEX idx_financial_transactions_rzp  
   ON public.financial_transactions(razorpay_payment_id);  
 ──────  
## 📋 Summary Table  
  
  Schema Table           | Clinical / Financial Purpose                        | Current Gap Solved                                                        | Key Mechanism  
 ------------------------|-----------------------------------------------------|---------------------------------------------------------------------------|----------------------------------------------------------------------------  
  patient_vitals         | Time-series health telemetry (BP, Sugar, SpO2,      | Replaces local setState() in Flutter; preserves history across sessions   | Composite index (patient_id, recorded_at DESC) + strict patient/doctor  
                         | Weight)                                             | and consultations.                                                        | RLS.  
  doctor_reviews         | Post-consultation rating & patient feedback         | Replaces unbacked summary ratings on doctors with individual, verified    | PostgreSQL AFTER INSERT OR UPDATE trigger recalculating AVG(rating) and  
                         |                                                     | consultation reviews.                                                     | review_count.  
  financial_transactions | Comprehensive double-entry escrow ledger            | Connects incoming Razorpay charges with the 85/15 split, GST accounting,  | Foreign key linkage to appointments, doctor_payouts, and  
                         |                                                     | and RBI nodal compliance.                                                 | appointment_refunds.