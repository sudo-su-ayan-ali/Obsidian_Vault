# 📊 Remaining Implementation Tasks & Production Launch Checklist

While the core codebase, UI/UX screens, domain models, database migrations, and 201/201 automated tests (100% pass rate) are fully written and verified locally,

the remaining work centers on Cloud Infrastructure Deployment, Third-Party Live Credential Provisioning, Production Build Pipeline, and Growth Ecosystem Features.

---

## 1. ⚡ Backend & Infrastructure Tasks Remaining

These tasks involve deploying serverless Edge Functions to Supabase Cloud and setting up secrets/automation in production.

### A. Supabase Edge Functions Cloud Deployment

Currently, the Edge Functions in `supabase/functions/` are written and unit-tested locally/in mock mode. They need to be deployed to the live Supabase Cloud project

`xnflvmvdkrfeuwivjetx.supabase.co` in ap-south-1 / Mumbai:

1. `lock-appointment-slot`: Atomic slot reservation & 10-minute lock with Razorpay Order ID generation.
    
2. `razorpay-webhook`: Webhook endpoint for HMAC-SHA256 signature validation and automatic payment confirmation.
    
3. `generate-agora-token`: Serverless dynamic Agora RTC token generator for secure video/audio consultations.
    
4. `generate-prescription-pdf`: Deno HTML-to-PDF compiler for digital Tri-Fold prescriptions (Medicines + Diet + Yoga).
    
5. `dispatch-notifications`: FCM / OneSignal push notification dispatcher with MSG91 DLT SMS fallback.
    
6. `clean-expired-slots`: Automated cron job for releasing 10-minute expired slot reservations.
    
7. `refund-appointment`: Automated Razorpay refund processor for patient cancellations.
    
8. `request-doctor-payout`: Escrow settlement and doctor earnings payout request engine.
    

**Deployment Command:**

```bash
supabase functions deploy --project-ref xnflvmvdkrfeuwivjetx
```

---

### B. Supabase Production Vault Secrets Configuration (Paused)

Populate the live environment variables in the Supabase Vault (`supabase secrets set`):

|Secret Name|Purpose|Target Service|
|---|---|---|
|`RAZORPAY_KEY_ID` & `RAZORPAY_KEY_SECRET`|Live UPI payment checkout & webhooks|Razorpay|
|`RAZORPAY_WEBHOOK_SECRET`|Signature verification header validation|Razorpay Webhook|
|`AGORA_APP_ID` & `AGORA_APP_CERTIFICATE`|Live video/audio RTC token minting|Agora RTC|
|`ONESIGNAL_APP_ID` & `ONESIGNAL_REST_API_KEY`|Live push notifications to mobile devices|OneSignal|
|`MSG91_AUTH_KEY` & `MSG91_SENDER_ID`|Trai DLT SMS fallback (TRIVID)|MSG91 SMS|

---

### C. Backend Automation & Storage Configuration

1. **pg_cron Background Schedule Verification:** Verify that `supabase/migrations/20260923_pg_cron_background_automation.sql` is active on Supabase Cloud to run the
    
    `clean-expired-slots` function every 5 minutes.
    
2. **Supabase Storage Bucket CORS & Signed Link Lifetimes:**
    
    - `prescriptions`: Signed URLs set to 24-hour expiration.
        
    - `kyc-documents`: Signed URLs restricted to Admin role and verified Doctors.
        
    - `medical-vault`: Patient-only access policies verified.
        

---

## 2. 💻 Admin Portal Remaining Tasks

The Next.js 14 Admin Portal (`package.json`) is built with `@supabase/ssr` and Tailwind CSS. The remaining steps are:

1. **Vercel / Cloudflare Deployment:** Deploy `admin/` to Vercel or Cloudflare Pages.
    
2. **Admin Auth Role Guarding:** Ensure `middleware.ts` enforces `public.users.role == 'admin'` for access to:
    
    - KYC Verification Queue (`/kyc`)
        
    - Doctor Directory & Onboarding Approval (`/doctors`)
        
    - Financial Ledger & Doctor Payouts (`/finance`)
        
    - Platform Settings (`/settings`)
        

---

## 3. 📱 Mobile App (Flutter) & Play Store Release Tasks

1. **Production Build Credentials:**
    
    Provide production `--dart-define` parameters during Flutter build:
    
    ```bash
    flutter build appbundle --release \
      --dart-define=SUPABASE_URL=https://xnflvmvdkrfeuwivjetx.supabase.co \
      --dart-define=SUPABASE_ANON_KEY=your_production_anon_key \
      --dart-define=ONESIGNAL_APP_ID=your_onesignal_app_id \
      --dart-define=SENTRY_DSN=your_sentry_dsn
    ```
    
2. **Android Release Keystore:**
    
    - Generate `upload-keystore.jks` and configure `android/key.properties` for Play Store signing.
        
3. **App Store & Play Store Assets:**
    
    - High-resolution screenshots (360x640, 800x1280), privacy policy link, and app listing metadata.
        

---

## 4. ⏳ Future Horizon (Phase 6 Post-Launch Expansion)

After the MVP launch, the following ecosystem expansions are planned in `roadmap.md`:

- **Diagnostic Labs Integration:** Home sample collection booking via local lab partners (Thyrocare, Metropolis).
    
- **E-Pharmacy Integration:** Automated prescription dispatch to local chemists for medicine fulfillment.
    
- **Clinic SaaS & Offline Appointments:** In-clinic walk-in booking module for verified doctors.
    

---

## 🎯 Summary Checklist

| Task Area         | Action Needed                                                             | Priority  |
| ----------------- | ------------------------------------------------------------------------- | --------- |
| Backend Functions | Deploy 8 Edge Functions to Supabase Cloud (ap-south-1)                    | 🔴 High   |
| Backend Secrets   | Set Razorpay, Agora, OneSignal & MSG91 keys in Supabase Vault             | 🔴 High   |
| Admin Portal      | Deploy Next.js `admin/` app to Vercel/Cloudflare                          | 🟡 Medium |
| Mobile Release    | Build Android Release App Bundle (`.aab`) with production `--dart-define` | 🟡 Medium |
| Store Listing     | Upload Privacy Policy, Terms, and Play Store graphics                     | 🟢 Low    |
