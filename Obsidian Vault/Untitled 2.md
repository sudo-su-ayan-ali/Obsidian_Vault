# Phase 3: MVP Development & Integration — Detailed Task Breakdown

This document provides the complete, granular task breakdown, implementation specifications, and sprint schedule for **Phase 3: MVP Development & Integration** of the Trividha platform across Flutter clients, Cloud Functions, and the Admin web portal.

---

## 📋 Task Breakdown Structure

```text
PHASE 3: MVP DEVELOPMENT & INTEGRATION (12 Weeks / 6 Sprints)
│
├── Sprint 1: Authentication, Multi-Language & User Onboarding (Weeks 1–2)
│   ├── [Task 3.1] Phone Number OTP Authentication UI & Riverpod State Notifier
│   ├── [Task 3.2] Multilingual Language Switcher (Hindi, Gujarati, Rajasthani, English)
│   ├── [Task 3.3] Role Selection & Custom Claims Onboarding (Patient vs Doctor)
│   ├── [Task 3.4] Patient Basic Demographics & Medical Profile Form
│   └── [Task 3.5] Doctor Professional Profile Setup & Council Registration Entry
│
├── Sprint 2: Doctor Discovery & Availability Scheduling Engine (Weeks 3–4)
│   ├── [Task 3.6] Patient Home Dashboard with Specialty Grid & Quick Symptom Search
│   ├── [Task 3.7] Doctor Search Engine with Multi-Parameter Filters (Specialty, AYUSH/Allopathy, Fee)
│   ├── [Task 3.8] Doctor Detailed Profile Screen, Qualifications & Reviews View
│   ├── [Task 3.9] Doctor Schedule & Recurring Slot Creation Management UI (Doctor App)
│   └── [Task 3.10] Dynamic Slot Availability Sync with Real-time Firestore Stream
│
├── Sprint 3: Booking Concurrency & Razorpay Payment Integration (Weeks 5–6)
│   ├── [Task 3.11] Appointment Checkout Screen with Pricing Breakdown & Platform Fee
│   ├── [Task 3.12] Atomic 10-Minute Optimistic Slot Locking Integration (`lockAppointmentSlot`)
│   ├── [Task 3.13] Razorpay SDK UPI Intent & NetBanking Checkout Integration
│   ├── [Task 3.14] Payment Failure Handling & Lock Expiration Countdown Timer
│   └── [Task 3.15] Appointment Confirmation & Digital Booking Receipt Screen
│
├── Sprint 4: Agora Real-Time Teleconsultation & In-Call Experience (Weeks 7–8)
│   ├── [Task 3.16] Pre-Call Waiting Room, Hardware Permissions & Network Quality Check
│   ├── [Task 3.17] Agora RTC Video Calling Surface Integration (Adaptive 720p/360p)
│   ├── [Task 3.18] In-Call Floating Controls (Mic, Video, Flip Camera, Audio Routing)
│   ├── [Task 3.19] Real-time In-Call Text Chat & File Attachment Stream
│   └── [Task 3.20] Call End Signaling, Duration Logger & Post-Consult Rating Modal
│
├── Sprint 5: Digital Tri-Fold Prescription Builder & Medical Vault (Weeks 9–10)
│   ├── [Task 3.21] Doctor Tri-Fold Prescription Builder: Clinical Diagnosis & Medicines Form
│   ├── [Task 3.22] Doctor Tri-Fold Prescription Builder: Dietary Macros & Food Rules Form
│   ├── [Task 3.23] Doctor Tri-Fold Prescription Builder: Guided Yoga & Physical Exercises Form
│   ├── [Task 3.24] Cloud Functions Headless PDF Compilation & Firebase Storage Dispatch
│   ├── [Task 3.25] Patient Prescription Viewer Screen with Interactive Tri-Fold Tabs
│   └── [Task 3.26] Patient Health Locker / Vault Screen with PDF Viewer & Encrypted Uploads
│
└── Sprint 6: Admin Operations Portal & Doctor KYC Workflow (Weeks 11–12)
    ├── [Task 3.27] Doctor KYC Document Upload Screen (Medical Council Certificate, ID Proof)
    ├── [Task 3.28] Next.js Admin Operations Web Portal Scaffolding & Firebase Auth
    ├── [Task 3.29] Admin Doctor KYC Verification Queue & Approval/Rejection Engine
    └── [Task 3.30] Automated Notifications Engine (FCM Push Alerts & MSG91 SMS)
```

---

## 1. 🔑 Sprint 1: Authentication, Multi-Language & User Onboarding
* **Sprint Timeline:** Weeks 1–2
* **Target Output:** Functional login flow, language selector, and patient/doctor profile creation.

| Task ID | Task Title | Status | Execution Details & Technical Requirements |
| :--- | :--- | :---: | :--- |
| **Task 3.1** | **Phone OTP Auth UI & Riverpod State** | `⏳ Ready` | Build phone number input with India `+91` prefix, 6-digit OTP boxes, resend timer (45s), and auto-SMS verification using `firebase_auth`. |
| **Task 3.2** | **Multilingual Language Switcher** | `⏳ Ready` | Implement localized UI strings and persistent language switcher for **Hindi (हिंदी)**, **Gujarati (ગુજરાતી)**, **Rajasthani (राजस्थानी)**, and **English**. |
| **Task 3.3** | **Role Selection & Onboarding** | `⏳ Ready` | Implement post-login role onboarding screen invoking `registerUser` Cloud Function to inject JWT custom claims (`patient` or `doctor`). |
| **Task 3.4** | **Patient Profile Setup Form** | `⏳ Ready` | Create form collecting full name, age, gender, blood group, city/state, emergency contact, and existing allergies. |
| **Task 3.5** | **Doctor Profile Setup Form** | `⏳ Ready` | Create form collecting degrees (`MBBS`, `MD`, `BAMS`, `BHMS`), medical council registration number, consultation fee (₹), and years of experience. |

---

## 2. 🔍 Sprint 2: Doctor Discovery & Availability Scheduling Engine
* **Sprint Timeline:** Weeks 3–4
* **Target Output:** Searchable doctor directory and doctor schedule management UI.

| Task ID | Task Title | Status | Execution Details & Technical Requirements |
| :--- | :--- | :---: | :--- |
| **Task 3.6** | **Patient Home Dashboard** | `⏳ Ready` | Build home screen with Arogya brand header, symptom search bar, top specialty cards (Skin, Digestion, PCOS, Sexual Health), and active appointments banner. |
| **Task 3.7** | **Doctor Search & Multi-Filter Engine** | `⏳ Ready` | Build searchable doctor list with multi-select filters: Specialty, Medicine Type (Allopathy vs AYUSH), Consultation Fee Range, Spoken Languages, and Gender. |
| **Task 3.8** | **Doctor Detailed Profile View** | `⏳ Ready` | Display doctor bio, verified council badge, degrees, clinical focus areas, consultation pricing, patient reviews, and clinic location. |
| **Task 3.9** | **Doctor Schedule Management UI** | `⏳ Ready` | In Doctor App, build weekly recurring schedule editor allowing doctors to define morning/evening slots, slot duration (15/30 min), and emergency time-off toggles. |
| **Task 3.10**| **Real-time Slot Availability Stream** | `⏳ Ready` | Connect `AppointmentSlotPicker` to real-time Firestore collection stream (`doctors/{doctorId}/slots`), automatically dimming reserved/booked slots. |

---

## 3. 💳 Sprint 3: Booking Concurrency & Razorpay Payment Integration
* **Sprint Timeline:** Weeks 5–6
* **Target Output:** End-to-end checkout with atomic slot locking and instant UPI payments.

| Task ID | Task Title | Status | Execution Details & Technical Requirements |
| :--- | :--- | :---: | :--- |
| **Task 3.11**| **Appointment Checkout Screen** | `⏳ Ready` | Display appointment summary (Doctor name, date/time, mode: Video/Audio), itemized bill (Doctor fee + ₹19 convenience fee), and patient details. |
| **Task 3.12**| **Atomic Slot Locking Integration** | `⏳ Ready` | Call `lockAppointmentSlot` Cloud Function to acquire 10-min optimistic lock and obtain Razorpay Order ID. |
| **Task 3.13**| **Razorpay Checkout Integration** | `⏳ Ready` | Integrate `razorpay_flutter` invoking native UPI Intent (Google Pay, PhonePe, Paytm, BHIM) and NetBanking/Card options. |
| **Task 3.14**| **Payment Failure & Expiry Handling** | `⏳ Ready` | Render 10-minute lock countdown timer; handle user cancellation, payment retry, and lock expiration auto-release. |
| **Task 3.15**| **Booking Confirmation & Receipt** | `⏳ Ready` | Render success animation, confirmed appointment card with "Add to Calendar", instructions for joining video call, and downloadable receipt. |

---

## 4. 📹 Sprint 4: Agora Real-Time Teleconsultation & In-Call Experience
* **Sprint Timeline:** Weeks 7–8
* **Target Output:** Low-latency video teleconsultation room with in-call messaging.

| Task ID | Task Title | Status | Execution Details & Technical Requirements |
| :--- | :--- | :---: | :--- |
| **Task 3.16**| **Pre-Call Waiting Room** | `⏳ Ready` | Request camera & microphone permissions, test microphone audio levels, display self-preview mirror, and check internet speed. |
| **Task 3.17**| **Agora RTC Video Engine Setup** | `⏳ Ready` | Initialize `agora_rtc_engine` with channel token from `generateAgoraRtcToken` Cloud Function; configure dual-stream 720p/360p adaptive bitrate. |
| **Task 3.18**| **In-Call Controls Overlay** | `⏳ Ready` | Implement `ConsultationCallOverlay` with mute audio, disable camera, flip camera, switch audio output (Earpiece/Speaker/Bluetooth), and end call. |
| **Task 3.19**| **In-Call Real-Time Chat** | `⏳ Ready` | Slide-over chat panel using Firestore subcollection (`appointments/{appointmentId}/messages`) allowing real-time text and prescription photo exchange. |
| **Task 3.20**| **Call End Signaling & Rating** | `⏳ Ready` | Auto-terminate session on call end, record consultation duration, update appointment status to `completed`, and prompt 5-star patient review. |

---

## 5. 📋 Sprint 5: Digital Tri-Fold Prescription Builder & Medical Vault
* **Sprint Timeline:** Weeks 9–10
* **Target Output:** Doctor prescription builder and patient health record locker.

| Task ID | Task Title | Status | Execution Details & Technical Requirements |
| :--- | :--- | :---: | :--- |
| **Task 3.21**| **Rx Builder: Clinical Diagnosis & Meds**| `⏳ Ready` | Fast autocomplete medicine selector, dosage input (`500mg`, `2 tsp`), frequency picker (`1-0-1`), meal timing (`Before/After Food`), and duration (days). |
| **Task 3.22**| **Rx Builder: Dietary Macro Plan** | `⏳ Ready` | Structured food planner with target calorie/protein breakdown, recommended meal list, and strictly forbidden foods checklist. |
| **Task 3.23**| **Rx Builder: Guided Yoga Recovery** | `⏳ Ready` | Curated yoga asana picker with duration, frequency, precaution notes, and animated posture reference diagrams. |
| **Task 3.24**| **Cloud Function PDF Compilation** | `⏳ Ready` | Invoke `generatePrescriptionPdf` to generate branded PDF with official doctor signature/stamp and upload to Cloud Storage. |
| **Task 3.25**| **Patient Tri-Fold Prescription View** | `⏳ Ready` | Implement `TriFoldPrescriptionViewer` widget displaying the 3 tabs (Meds, Diet, Yoga) with single-tap PDF download and WhatsApp share. |
| **Task 3.26**| **Patient Health Locker / Vault** | `⏳ Ready` | Medical records vault allowing patients to upload lab reports, past prescriptions, and scans with 24-hour doctor sharing permission controls. |

---

## 6. 🛡️ Sprint 6: Admin Operations Portal & Doctor KYC Workflow
* **Sprint Timeline:** Weeks 11–12
* **Target Output:** Next.js Admin portal for doctor credential verification and automated notifications.

| Task ID | Task Title | Status | Execution Details & Technical Requirements |
| :--- | :--- | :---: | :--- |
| **Task 3.27**| **Doctor KYC Document Upload** | `⏳ Ready` | Doctor app screen for uploading Medical Registration Certificate, Degree Certificates, and Government ID (Aadhaar/PAN). |
| **Task 3.28**| **Next.js Admin Operations Portal** | `⏳ Ready` | Initialize Next.js web portal with Firebase Auth, role guard for `role === 'admin'`, and Tailwind CSS dashboard layout. |
| **Task 3.29**| **Admin KYC Review & Approval Queue**| `⏳ Ready` | Document preview modal, Medical Council verification checklist, single-click "Approve Doctor" or "Reject with Reason" action buttons. |
| **Task 3.30**| **Automated Notifications Engine** | `⏳ Ready` | High-priority FCM push notifications for 15-minute appointment reminders, incoming call rings, and MSG91 transactional SMS alerts. |

---

## 🏁 Phase 3 Execution Status & Sign-Off Target
* **Phase Status:** `⚡ ACTIVE / IN PROGRESS`
* **Target Completion:** 6 two-week sprints (12 weeks)
* **Immediate Focus:** **Sprint 1 (Tasks 3.1 – 3.5: Phone OTP Auth, Multi-Language & Onboarding)**
* **Master Lifecycle Document:** [`docs/09-project-management/execution-phases.md`](../09-project-management/execution-phases.md)