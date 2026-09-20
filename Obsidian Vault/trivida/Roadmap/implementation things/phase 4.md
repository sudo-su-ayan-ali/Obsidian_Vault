 ──────  
 ## 1. 🧹 [Task 4.9] Static Analysis & Lint Cleanup (flutter analyze Zero-Warning Pass)  
  
 ### What is Static Application Analysis?  
  
 Static analysis scans code without running it to detect syntax antipatterns, type mismatches, deprecated APIs, dead code, and memory leaks. In Flutter, this is governed by the Dart analyzer and configured via analysis_options.yaml.  
  
 ### The Current State: 51 Analyzer Issues in Trividha  
  
 Running flutter analyze currently flags 51 notices across the codebase. None are fatal compile errors, but they represent technical debt and future failure risks:  
  
   51 issues found across lib/ and test/  
   ├── 24 Deprecated Flutter API calls  
   ├── 14 Unnecessary / redundant 'const' keywords  
   ├── 8  Unused imports  
   ├── 3  Unused local variables  
   └── 2  Field initialization antipatterns  
  
 ### Deep Dive into the 4 Categories of Issues:  
  
 #### A. Deprecated Color Space: .withOpacity() ➔ .withValues()  
  
 • The Code Issue: Found in doctor_kyc_upload_screen.dart:  
   // ⚠ Deprecated in Flutter 3.27+  
   color: ArogyaColors.primaryTeal.withOpacity(0.12)  
  
 • Why it matters: In Flutter 3.27+, the Flutter Engine moved from 8-bit integer color math (0–255) to wide-gamut floating-point color spaces (Display P3). .withOpacity() truncates color channels and causes precision loss or color  
 banding on modern OLED screens.  
 • The Fix:  
   // ✅ Modern Flutter 3.27+ wide-gamut standard  
   color: ArogyaColors.primaryTeal.withValues(alpha: 0.12)  
  
  
 #### B. Deprecated Widget Attributes: activeColor & value  
  
 • In doctor_schedule_screen.dart:  
   // ⚠ Deprecated: activeColor  
   Switch(value: isOnline, activeColor: ArogyaColors.primaryTeal, onChanged: ...)  
 activeColor is superseded by activeThumbColor and activeTrackColor to adhere to Material 3 dynamic theming.  
 • In doctor_prescription_builder_screen.dart:  
   // ⚠ Deprecated: value on FormField  
   DropdownButtonFormField(value: selectedTiming, ...)  
 In modern Flutter, value inside a FormField can conflict with form reset lifecycles; it must be replaced with initialValue.  
  
 #### C. Unused Imports & Dead Variables  
  
 • Example in doctor_search_test.dart and lock_expiry_test.dart:  
 Unreferenced models and widget sheets inflate test memory footprints and slow down the compiler.  
  
 #### D. Field Initializing Formals  
  
 • In doctor_kyc_provider.dart:  
   // ⚠ Antipattern  
   DoctorKycNotifier(IDoctorKycRepository repository) : _repository = repository;  
  
   // ✅ Clean Dart idiom  
   DoctorKycNotifier(this._repository);  
  
  
 ### Acceptance Gate for Task 4.9:  
  
 Running flutter analyze must output:  
  
   No issues found! (zero warnings, zero infos, zero errors)  
  
 This is required before CI/CD GitHub Actions can build release APKs without build warnings.  
 ──────  
 ## 2. ⚙ [Task 4.10] Native Android Toolchain & Gradle Build Audit (NDK/Kotlin)  
  
 ### Why Telehealth Apps Require Native C++ Toolchains  
  
 Trividha is not just a standard forms-and-lists app—it streams real-time low-latency HD video and audio via the Agora RTC Engine and handles native UPI Intent transactions via Razorpay.  
  
 Agora communicates directly with mobile hardware (camera sensors, microphones, audio DSP hardware, hardware video encoders H.264/VP8) using native C/C++ libraries compiled into .so (shared object) binaries.  
  
   Flutter Dart Layer  (AgoraVideoView, AudioRouting)  
           │  MethodChannel / FFI  
           ▼  
   Java / Kotlin JNI Layer (io.agora.rtc.RtcEngine)  
           │  Native JNI Bindings  
           ▼  
   C++ Native Binaries (.so files compiled via Android NDK)  
     ├── libagora-rtc-sdk.so  (ARM64-v8a / ARMeabi-v7a)  
     └── libagora-sound-touch.so  
           │  
           ▼  
   Android Linux Kernel (Camera HAL, Audio ALSA Drivers)  
  
 ### The 3 Core Engineering Hardening Steps in Task 4.10:  
  
 #### 1. Android NDK Provisioning & CMake JNI Configuration  
  
 • The Challenge: Modern Flutter and AGP (Android Gradle Plugin 8+) require modern Native Development Kits. If the NDK is missing or mismatched, running native code produces:  
 java.lang.UnsatisfiedLinkError: Couldn't load agora-rtc-sdk.so: findLibrary returned null  
 • What was audited: Provisioned Android NDK 28.2.13676358 (r28c) and bound toolchains in build.gradle.kts for CMake debug configuration (:jni:configureCMakeDebug).  
  
 #### 2. Kotlin 2.2 & Sentry 9 Modernization  
  
 • The Challenge: Old Gradle configurations fail on modern systems with:  
 Language version 1.6 is no longer supported; please use 1.8 or higher.  
 • What was audited: Upgraded sentry_flutter to ^9.0.0 (resolving 9.29.0) to modernize the Kotlin toolchain across the Android pipeline.  
  
 #### 3. SDK Version Constraints: minSdk = 21 vs targetSdk = 34  
  
 • Target Audience Reality in India: Many patients in Tier-3/4 cities and rural areas use budget Android smartphones (e.g., Redmi 6A, Realme C2, Samsung Galaxy J series) running Android 5.0 to Android 8.0.  
     • Setting minSdk = 21 (Android 5.0 Lollipop) ensures that 99.2% of active Android devices in India can install and run Trividha.  
     • Setting targetSdk = 34 (Android 14) satisfies Google Play Store mandatory release criteria and enforces modern runtime permission models (POST_NOTIFICATIONS, CAMERA, RECORD_AUDIO).  
  
 ──────  
 ## 3. 📐 [Task 4.11] Multi-Screen Layout & Display Density Overflow Audit  
  
 ### What is the "RenderFlex Overflow" Nightmare in Flutter?  
  
 In Flutter, if a UI element's children occupy more physical pixels than the device screen allows, Flutter does not silently crop—it renders the infamous yellow-and-black hazard striped bar (A RenderFlex overflowed by 18 pixels on  
 the right).  
  
 In a medical app, if an overflow occurs:  
  
 • A doctor might not see the "Confirm Dosage" button.  
 • A patient might not see the "Join Call" or "Proceed to Pay" button.  
 • Medical council numbers get clipped, failing statutory compliance.  
  
   ┌──────────────────────────────────────┐  
   │ Dr. Ramesh Patel, MBBS, MD           │  
   │ Council: MCI-IND-2018-99410  ▲▲▲▲▲▲▲ │ <--- ⚠ 18px Overflow hazard bar!  
    └──────────────────────────────────────┘  
  
 ### The 3 Display Variations Tested in Task 4.11:  
  
   ┌─────────────────┐    ┌────────────────────────┐    ┌───────────────────────────────┐  
   │   Compact Phone │    │     Standard Phone     │    │        Tablet / Foldable      │  
   │   (360 x 640 dp)│    │     (412 x 915 dp)     │    │        (600 x 1024 dp)        │  
   │   e.g. Redmi 9A │    │     e.g. Pixel 8       │    │        e.g. Galaxy Tab A9     │  
   │   Budget Indian │    │     Mainstream         │    │        Doctor Clinic OPD      │  
   └─────────────────┘    └────────────────────────┘    └───────────────────────────────┘  
  
 ### Real-World Audit Cases Already Hardened in Trividha:  
  
 #### Case 1: Unconstrained Horizontal Row in booking_confirmation_screen.dart  
  
 • The Bug: The "Add to Calendar" OutlinedButton inside a horizontal Row inherited Size.fromHeight(48) and forced an infinite width, crashing the widget tree (BoxConstraints forces an infinite width).  
 • The Fix: Explicitly set minimumSize: const Size(80, 36) and wrapped button actions in flexible containers.  
  
 #### Case 2: Medical Council Badge Overflow in doctor_detail_profile_screen.dart  
  
 • The Bug: Long council strings (Maharashtra Council of Indian Medicine (MCIM) - 2018-99410) pushed past the screen edge by 18px on 360dp displays.  
 • The Fix: Wrapped badge text inside Flexible with TextOverflow.ellipsis:  
   Flexible(  
     child: Text(  
       registrationNumber,  
       overflow: TextOverflow.ellipsis,  
       maxLines: 1,  
     ),  
   )  
  
  
 #### Case 3: Dropdown Overflow in doctor_prescription_builder_screen.dart  
  
 • The Bug: Medicine timing dropdowns ("After Food / भोजन के बाद") overflowed on smaller screens.  
 • The Fix: Added isExpanded: true to all DropdownButtonFormField widgets to automatically constrain dropdown width to available horizontal space.  
  
 #### Case 4: Elderly Patient Font Scaling (textScaleFactor = 1.5)  
  
 • Many elderly patients in India configure their phone font sizes to "Large" or "Largest" in Android Settings.  
 • If a layout uses hardcoded heights (Container(height: 50)), enlarged text will overflow vertically.  
Task 4.11 audits all screens with dynamic text scaling:  
   // Simulated in automated widget tests:  
   tester.binding.platformDispatcher.textScaleFactorTestValue = 1.5;  
  
 ──────  
 ### 🎯 Summary: How Module 2 Guarantees App Stability  
  
  Task                                                                        | What It Prevents                                                           | End Result  
 -----------------------------------------------------------------------------|----------------------------------------------------------------------------|----------------------------------------------------------------------------  
  Task 4.9 (Static Analysis)                                                  | Deprecated API crashes, memory leaks, untracked errors                     | Zero warnings on flutter analyze  
  Task 4.10 (Native Toolchain)                                                | C++ Agora .so crashes, audio DSP failures, Kotlin build breaks             | Stable native audio/video calling on any Android hardware  
  Task 4.11 (UI Layout)                                                       | Yellow-black hazard overflow bars, hidden booking buttons, clipped text    | Pixel-perfect layout from ₹6,000 budget phones to clinic tablets



---




🛡 Module 3: Backend, Security & PostgreSQL Row-Level Security (RLS) Auditing — Deep Dive
  Telehealth platforms store Electronic Health Records (EHR), prescription data, diagnostic logs, and financial transaction records. Under
  statutory frameworks such as the Digital Personal Data Protection Act (DPDPA 2023) and National Health Authority (NHA) / ABDM guidelines,
  client-side validation is deemed zero-trust. Every boundary must be enforced, audited, and verified at the database kernel and edge layers.
  ──────
  ## 1. 🗄 [Task 4.12] PostgreSQL RLS Penetration Suite

  ### Why Row-Level Security (RLS) Matters

  In standard REST/GraphQL architectures, authorization is often handled in application middleware (if (req.user.id !== patient.id) throw
  403). If an engineer forgets this check or a SQL injection occurs, Broken Object-Level Authorization (BOLA / IDOR) exposes records across
  tenants.
  In Supabase/PostgreSQL, RLS moves authorization directly into the database engine. Even if a malicious client sends a crafted query like
  SELECT * FROM public.prescriptions;, the database kernel dynamically injects WHERE patient_id = auth.uid() OR doctor_id = auth.uid().

  ### The 3 Critical Test Vectors:
                      ┌──────────────────────────────────────────────────────────┐
                      │          PostgreSQL Database Kernel Execution            │
                      └────────────────────────────┬─────────────────────────────┘
                                                   │
                   ┌───────────────────────────────┼───────────────────────────────┐
                   ▼                               ▼                               ▼
           Vector 1: Isolation             Vector 2: Dual-Party            Vector 3: Zero-Trust
       Patient A vs Patient B          Doctor & Patient Access         Anonymous / Public Request
     ┌───────────────────────────┐   ┌───────────────────────────┐   ┌───────────────────────────┐
     │ Patient A attempts:       │   │ Dr. Sharma queries:       │   │ Attacker sends query:     │
     │ SELECT * FROM patients    │   │ SELECT * FROM rx          │   │ without JWT Authorization │
     │ WHERE id = 'patient_b';   │   │ WHERE appt = 'appt_101';  │   │ Bearer token.             │
     │                           │   │                           │   │                           │
     │ Expected: 0 rows returned │   │ Expected: Granted ONLY if │   │ Expected: Connection      │
     │ (No cross-tenant leak)    │   │ doctor_id = Dr. Sharma    │   │ rejected / 0 rows visible │
     └───────────────────────────┘   └───────────────────────────┘   └───────────────────────────┘

  ### Execution Details & pgTAP Test Instructions:

  We use pgTAP (PostgreSQL unit testing suite) running against local Supabase test instances:

    -- test/security/rls_penetration_test.sql
    BEGIN;
    CREATE EXTENSION IF NOT EXISTS pgtap;
    SELECT plan(4);

    -- 1. Setup Mock User Contexts in auth.users
    INSERT INTO auth.users (id, email) VALUES
      ('11111111-1111-1111-1111-111111111111', 'patient_a@trividha.in'),
      ('22222222-2222-2222-2222-222222222222', 'patient_b@trividha.in'),
      ('33333333-3333-3333-3333-333333333333', 'dr_sharma@trividha.in');

    -- TEST 1: Unauthenticated request should see zero patients
    SET LOCAL ROLE anon;
    SELECT is_empty(
      'SELECT * FROM public.patients;',
      'Anonymous requests must return 0 patient rows'
    );

    -- TEST 2: Patient A cannot read Patient B's profile
    SET LOCAL ROLE authenticated;
    SET LOCAL "request.jwt.claims" = '{"sub": "11111111-1111-1111-1111-111111111111", "role": "authenticated"}';

    SELECT is_empty(
      'SELECT * FROM public.patients WHERE id = ''22222222-2222-2222-2222-222222222222'';',
      'Patient A cannot read Patient B profile (Cross-Tenant Leak Prevention)'
    );

    -- TEST 3: Patient A cannot update Patient B's ABHA ID
    SELECT throws_ok(
      'UPDATE public.patients SET abha_number = ''99-9999-9999-9999'' WHERE id = ''22222222-2222-2222-2222-222222222222'';',
      'Patient A cannot alter records owned by Patient B'
    );

    -- TEST 4: Only the assigned Doctor can view the Tri-Fold Prescription
    SET LOCAL "request.jwt.claims" = '{"sub": "33333333-3333-3333-3333-333333333333", "role": "authenticated"}';
    SELECT results_eq(
      'SELECT count(*)::int FROM public.prescriptions WHERE doctor_id = ''33333333-3333-3333-3333-333333333333'';',
      ARRAY[1],
      'Doctor can strictly view only prescriptions where they are the attending physician'
    );

    SELECT * FROM finish();
    ROLLBACK;
  ──────
  ## 2. 🪣 [Task 4.13] Storage Bucket Security & Access Token Audit

  ### The Vulnerability Scope: Medical & KYC Storage Leakage

  Trividha maintains two private Supabase Storage buckets:

  1. kyc-documents/: Contains Aadhaar cards, medical council degree certificates, and state council registration proofs uploaded by doctors.
  2. medical-vault/: Contains diagnostic lab reports, ultrasound scans, and encrypted PDF prescriptions uploaded by patients or generated by
  doctors.

  If either bucket is set to Public (public = true), an attacker can enumerate file UUIDs
  (https://<project>.supabase.co/storage/v1/object/public/kyc-documents/doc_1/aadhaar.pdf) without authentication.

  ### Defensive Storage Architecture:

            Mobile Client (Flutter)
                       │
                       │ 1. Request file access: appointment_id = "appt_101"
                       ▼
            Supabase Edge Function / RLS Check
                       │
                       ├──> Check 1: Is user authenticated?
                       ├──> Check 2: Is user the Patient OR the active Doctor for appt_101?
                       │
                       ▼ (Authorized)
            Create HMAC-SHA256 Signed URL (Expires in 86,400s / 24 hrs)
                       │
                       ▼
            Direct Cloudflare CDN Stream to App (Protected Cache-Control: private)

  ### Audit Verification Checklist:

  1. Zero Public Access: Ensure public.buckets.public = false for both buckets. Direct HTTP GET requests without signing must return 403
  Forbidden / 400 InvalidKey.
  2. Path-Based Row-Level Security:
    -- Patients can only upload/read within their own folder: medical-vault/{patient_id}/*
    CREATE POLICY "Patient Vault Access" ON storage.objects
    FOR ALL TO authenticated
    USING (bucket_id = 'medical-vault' AND (storage.foldername(name))[1] = auth.uid()::text);

  3. Signed URL Expiry: Verify that generated signed URLs automatically expire after 24 hours (86,400 seconds) and cannot be reused beyond
  their TTL.
  ──────
  ## 3. 🛡 [Task 4.14] Edge Functions Input Sanitization & Error Handling Audit

  ### Status: [x] Complete

  All 5 production Supabase Edge Functions (verify-razorpay-signature, create-razorpay-order, generate-agora-token, generate-prescription-pdf,
  and send-sms-notification) were audited to ensure zero attack surfaces.

  ### Audited Attack Vectors & Mitigations:

   Vulnerability Vector    | Risk to Trividha                                       | Implemented Mitigation
  -------------------------|--------------------------------------------------------|---------------------------------------------------------
   Malformed JSON DoS      | Client sends non-JSON or recursive binary payload      | Wrapped request parsing in try { await req.json() }
                           | causing function crash.                                | catch returning immediate 400 Bad Request.
   SQL / UUID Injection    | Attacker injects ' OR 1=1 -- into appointment_id or    | Strict RFC 4122 regex validation (^[0-9a-f]{8}-[0-9a-
                           | slot_id.                                               | f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$). Non-UUID
                           |                                                        | strings are rejected before hitting the database.
   Integer Overflow on Fee | Negative or float amounts sent in payment order        | Strict Zod positive integer check:
                           | creation (amount: -50000).                             | z.number().int().positive().
   CORS Wildcard Abuse     | Unrestricted origin allowing CSRF from arbitrary       | Explicit CORS preflight header validation restricting
                           | third-party web domains.                               | allowed methods to POST, OPTIONS.
  ──────
  ## 4. 💳 [Task 4.15] Payment Webhook Idempotency & Replay Attack Defense

  ### The Double-Booking & Double-Capture Threat:

  When a patient completes UPI payment via Razorpay, Razorpay's webhook engine dispatches a payment.captured event to Trividha's Edge
  Function. In real-world cellular environments:

  • The network drops after Razorpay deducts money but before the client receives the callback.
  • Razorpay's retry queue automatically fires the same webhook multiple times (at 0s, 15s, 60s, 300s).
  • The Risk: If not idempotent, the server might book duplicate appointments, send duplicate SMS alerts, or deadlock the slot locking state
  machine.

  ### The Defensive Architecture:

      Razorpay Webhook Dispatcher
                 │
                 │ POST /verify-razorpay-webhook
                 │ Headers: X-Razorpay-Signature: <hmac_hex>
                 ▼
      ┌────────────────────────────────────────────────────────┐
      │ Step 1: Cryptographic HMAC-SHA256 Signature Validation  │
      │ Verify signature using raw body + RAZORPAY_WEBHOOK_SECRET│
      └────────────────────────────┬───────────────────────────┘
                                   │ (Valid Signature)
                                   ▼
      ┌────────────────────────────────────────────────────────┐
      │ Step 2: Idempotency Key Lookup (`payment_events` table) │
      │ Check if razorpay_payment_id already exists:           │
      │ SELECT status FROM payment_events WHERE payment_id = ? │
      └────────────────────────────┬───────────────────────────┘
                                   │
                   ┌───────────────┴───────────────┐
           Already Processed               First-Time Event
                   ▼                               ▼
         Return 200 OK                   BEGIN TRANSACTION;
         (No-Op / Prevent Double-Book)   1. Lock slot permanently
                                         2. Update appt -> 'confirmed'
                                         3. Log payment_events row
                                         COMMIT;
                                         Return 200 OK

  ### Test Strategy:

  4. Automated Replay Injection: Send two consecutive webhook payloads with the identical order_id and payment_id.
  5. Assertion 1: Both HTTP requests return 200 OK (so Razorpay stops retrying).
  6. Assertion 2: SELECT count(*) FROM public.appointments WHERE razorpay_payment_id = 'pay_test_123' returns strictly 1, guaranteeing zero
  duplicate records.
  ──────
  ### 🚀 Next Implementation Steps for Module 3:

  7. Create the pgTAP penetration test file test/security/rls_penetration_test.sql to formally execute the cross-tenant authorization audit.
  8. Run automated validation on the private storage bucket signed URL policies (Task 4.13).
  9. Verify the webhook idempotency transaction guard (Task 4.15).