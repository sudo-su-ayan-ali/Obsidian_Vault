# 📋 Master Credential Checklist

| Gateway / Service             | Required Keys & Environment Variables      | Priority        |
| :---------------------------- | :----------------------------------------- | :-------------- |
| **1. Supabase Cloud**         | Project Ref, Project URL, Service Role Key | 🔴 Mandatory    |
| **2. Razorpay (Standard)**    | Key ID, Key Secret, Webhook Secret         | 🔴 Mandatory    |
| **3. RazorpayX (Payouts)**    | Account Number, Key ID, Key Secret         | 🔴 Mandatory    |
| **4. Agora RTC Engine**       | App ID, App Certificate                    | 🔴 Mandatory    |
| **5. MSG91 (DLT SMS)**        | Auth Key, Sender ID (TRIVID)               | 🔴 Mandatory    |
| **6. OneSignal / FCM**        | App ID, REST API Key                       | 🟡 High         |
| **7. Medical AI & OCR**       | Gemini / OpenAI API Key                    | 🟡 High         |
| **8. ABDM (Ayushman Bharat)** | NHA Sandbox Client ID, Client Secret       | 🟢 Phase 2/Govt |

---

## 1. 🗄 Supabase Cloud Project (ap-south-1 Mumbai) 

> Needed to deploy all PostgreSQL migrations and the 10 serverless Edge Functions.

| Variable Name | Where to find it in Dashboard |
| :--- | :--- |
| `SUPABASE_PROJECT_REF` | Supabase Dashboard ➔ Project Settings ➔ General ➔ Reference ID (e.g. abcdefghijklm) |
| `SUPABASE_URL` | Supabase Dashboard ➔ Project Settings ➔ API ➔ Project URL (`https://<ref>.supabase.co`) |
| `SUPABASE_ANON_KEY` | Supabase Dashboard ➔ Project Settings ➔ API ➔ Project API Keys (anon / public) |
| `SUPABASE_SERVICE_ROLE_KEY` | Supabase Dashboard ➔ Project Settings ➔ API ➔ service_role (secret) |

---

## 2. 💳 Razorpay Standard (Patient Payments & Refunds)

> Needed for patient UPI, Credit/Debit card consultation fee collection and automated refunds.

| Variable Name | Where to find it in Dashboard |
| :--- | :--- |
| `RAZORPAY_KEY_ID` | Razorpay Dashboard ➔ Settings ➔ API Keys ➔ Key Id (Live Mode: `rzp_live_...`) |
| `RAZORPAY_KEY_SECRET` | Razorpay Dashboard ➔ Settings ➔ API Keys ➔ Key Secret |
| `RAZORPAY_WEBHOOK_SECRET` | Razorpay Dashboard ➔ Settings ➔ Webhooks ➔ Secret entered during webhook setup |
| **Webhook URL to register in Razorpay:** | `https://<ref>.supabase.co/functions/v1/razorpay-webhook` (Events: `payment.captured`, `payment.failed`, `refund.processed`) |

---

## 3. 🏦 RazorpayX (RBI PA-Compliant Doctor Bank Payouts)

> Needed for automated IMPS/NEFT earnings disbursement from the nodal escrow account to doctor bank accounts.

| Variable Name | Where to find it in Dashboard |
| :--- | :--- |
| `RAZORPAYX_ACCOUNT_NUMBER` | RazorpayX Dashboard ➔ My Account ➔ Virtual Nodal / Current Account Number (e.g. 232323...) |
| `RAZORPAYX_KEY_ID` | RazorpayX Dashboard ➔ Settings ➔ API Keys (Separate from standard Razorpay keys) |
| `RAZORPAYX_KEY_SECRET` | RazorpayX Dashboard ➔ Settings ➔ API Keys |

---

## 4. 📹 Agora RTC (HD Video & Audio Teleconsultation)

> Needed for dynamic WebRTC token generation and encrypted in-call video rooms.

| Variable Name | Where to find it in Dashboard |
| :--- | :--- |
| `AGORA_APP_ID` | Agora Console (`console.agora.io`) ➔ Project Management ➔ App ID (32-character string) |
| `AGORA_APP_CERTIFICATE` | Agora Console ➔ Project Management ➔ Project Edit ➔ App Certificate (Click "Enable / Primary Certificate") |

---

## 5. 📱 MSG91 (TRAI DLT-Approved Transactional SMS)

> Needed for rural OTPs and appointment SMS notifications on feature phones without active mobile data.

| Variable Name | Where to find it in Dashboard |
| :--- | :--- |
| `MSG91_AUTH_KEY` | MSG91 Dashboard (`msg91.com`) ➔ Settings ➔ API Key |
| `MSG91_SENDER_ID` | Approved 6-character TRAI DLT Sender ID (Default: `TRIVID`) |
| **DLT Flow Template IDs** | 1. Booking Confirmed: `12071689400012`<br>2. 15-Min Reminder: `12071689400023`<br>3. Prescription Ready: `12071689400034`<br>4. KYC Status: `1207168940004` (or your own registered IDs) |

---

## 6. 🔔 OneSignal (Push Notifications)

> Needed for mobile background notifications, in-app call ringing, and appointment alerts.

| Variable Name | Where to find it in Dashboard |
| :--- | :--- |
| `ONESIGNAL_APP_ID` | OneSignal Dashboard ➔ App Settings ➔ Keys & IDs ➔ OneSignal App ID |
| `ONESIGNAL_REST_API_KEY` | OneSignal Dashboard ➔ App Settings ➔ Keys & IDs ➔ REST API Key |

---

## 7. 🧠 Medical AI & OCR Engine

> Needed for the ask-medical-ai RAG pipeline and process-prescription-ocr paper prescription digitizer.

| Variable Name | Where to find it in Dashboard |
| :--- | :--- |
| `GEMINI_API_KEY` (or `OPENAI_API_KEY`) | Google AI Studio (`aistudio.google.com`) ➔ Get API Key (for embeddings & medical Q&A) |
| `OCR_API_KEY` (Optional) | Google Cloud Console ➔ Cloud Vision API Key (If using Cloud Vision instead of local OCR engine) |

---

## 8. 🇮🇳 ABDM / Ayushman Bharat Digital Mission (Phase 2 NHA Sandbox)

> Needed to connect to the Government health registry for 14-digit ABHA creation and EHR linking.

| Variable Name | How to register & obtain |
| :--- | :--- |
| `ABDM_CLIENT_ID` | Register on NHA ABDM Sandbox Portal (`sandbox.abdm.gov.in`) ➔ Developer Portal ➔ Client ID |
| `ABDM_CLIENT_SECRET` | NHA ABDM Developer Portal ➔ Client Secret |

---

### What to Provide First:

You can provide the credentials in batches. The **top 4** to start with immediately are:

1. **Supabase Project Credentials** (`SUPABASE_URL`, `SUPABASE_SERVICE_ROLE_KEY`)
2. **Razorpay Keys** (`RAZORPAY_KEY_ID`, `RAZORPAY_KEY_SECRET`, `RAZORPAY_WEBHOOK_SECRET`)
3. **Agora RTC Keys** (`AGORA_APP_ID`, `AGORA_APP_CERTIFICATE`)
4. **Gemini API Key** (`GEMINI_API_KEY`)

Whenever you have any of these ready, paste them here (or set them into a local `.env.production` file), and I will immediately bind the gateway configurations and deploy the Edge Functions!
