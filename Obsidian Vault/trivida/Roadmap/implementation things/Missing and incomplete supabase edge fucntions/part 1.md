**Big Picture**

The document says Trividha currently has important frontend/admin UI pieces and API specifications, but several real backend systems are either missing or mocked.

The 4 missing backend systems are:

1. `requestDoctorPayout`
2. `refund-appointment`
3. `ask-medical-ai`
4. `process-prescription-ocr`

Together, these cover money movement, refunds, medical AI assistance, and prescription digitization.

---

**1. requestDoctorPayout**

This function is meant to handle doctor payouts.

Right now, patients pay Trividha through Razorpay. The money first lands in Trividha’s Razorpay merchant account. From a ₹499 consultation, the platform keeps a portion, and the doctor should receive the remaining earnings.

The document gives this example:

- Consultation fee: ₹499
- Platform take-rate: 15%, which is ₹74.85
- Convenience fee: ₹19
- GST applies on platform fees
- Doctor earns around 85%, which is ₹424.15

The issue is that Trividha cannot simply keep doctor funds sitting in its main account forever. Because of RBI Payment Aggregator rules, doctor earnings need to be settled properly, usually through escrow or automated payouts.

Currently, the admin portal only pretends to execute a payout. It waits for 800 milliseconds and generates a fake UTR number like:

```
const utr = `UTR-RZP-${Date.now().toString().slice(-8)}`;
```

That means:

- No real Razorpay payout happens.
- No doctor bank account is credited.
- No payout status is persisted in the database./
- No real financial audit trail exists.

The proposed real version would:

- Verify the doctor’s KYC.
- Check whether the doctor has an active verified bank account.
- Calculate unsettled earnings from completed consultations.
- Ensure the doctor actually has enough balance to withdraw.
- Insert a payout record into the database.
- Call RazorpayX Payouts API or Razorpay Route.
- Receive a real UTR or payout status.
- Mark appointments or ledger entries as settled.
- Return success/failure to the admin or doctor.

The database needs tables like:

- `doctor_bank_accounts`
- `doctor_payouts`

The Edge Function would live at:

```
supabase/functions/request-doctor-payout/index.ts
```

A key requirement is idempotency. That means if the same payout request is accidentally submitted twice, the system should not pay the doctor twice.

---

**2. refund-appointment**

This function is meant to automate patient refunds.

The document lists real-world refund scenarios:

- Doctor does not join the video call.
- Patient cancels early enough.
- The call fails because of severe technical issues.
- Appointment is cancelled and money needs to go back.

Currently, the system may have an appointment status like `cancelled`, but it does not actually talk to Razorpay to refund the payment.

The admin UI also fakes refund processing by generating a mock ARN:

```
const arn = `ARN-${Math.floor(10000000000 + Math.random() * 90000000000)}`;
```

An ARN is a refund tracking/reference number, but here it is fake.

The real backend should:

- Accept an `appointmentId` and refund `reason`.
- Fetch the appointment from the database.
- Confirm that the appointment is eligible for refund.
- Check the cancellation policy.
- Get the original `razorpay_payment_id`.
- Call Razorpay Refund API:

```
POST /v1/payments/{payment_id}/refund
```

- Update the appointment status to `cancelled`.
- Free the doctor’s slot.
- Broadcast a realtime event so other users immediately see the slot as available.
- Insert a notification saying the refund was initiated.
- Return the real refund ID and ARN.

This function would live at:

```
supabase/functions/refund-appointment/index.ts
```

The document also mentions partial refunds. For example, if a patient cancels late, Trividha may deduct a ₹19 platform fee and refund only the remaining amount.

This is important because refund logic is not just “return all money.” It depends on policy, timing, dispute reason, and possibly whether the doctor or patient was responsible.

---

**3. ask-medical-ai / RAG Medical AI Pipeline**

This is the medical AI assistant behind the “Ask AI” button in the prescription viewer.

The current Flutter UI already has an “Ask AI” button, but it does not have a real backend. It only opens a placeholder or modal.

The intended feature is a conversational medical assistant that helps patients understand their prescription, diet instructions, and yoga recommendations.

The document calls this a RAG pipeline.

RAG means Retrieval-Augmented Generation. Instead of asking an AI model to answer from general memory, the system first retrieves trusted medical knowledge from a database, then asks the AI to answer using that grounded context.

The pipeline has two parts.

First, offline indexing:

- Collect verified medical documents.
- Examples:
    - Ayush Pharmacopoeia
    - ICMR guidelines
    - Yoga/asana safety information
- Break the documents into chunks.
- Convert each chunk into an embedding.
- Store embeddings in Supabase using `pgvector`.

The table would be:

```
medical_knowledge_vectors
```

Second, runtime answering:

- Patient taps “Ask AI.”
- Edge Function receives `prescriptionId` and `userQuestion`.
- It fetches the patient’s active prescription.
- It checks medicines, allergies, diet, and yoga plan.
- It searches the vector database for relevant medical knowledge.
- It builds a guarded prompt for the LLM.
- The LLM answers in the patient’s preferred language.
- The response is streamed back to the Flutter chat sheet.

The Edge Function would live at:

```
supabase/functions/ask-medical-ai/index.ts
```

The most important part is clinical safety.

The AI should not freely prescribe medicines. The guardrails should:

- Refuse to prescribe new Schedule-H drugs.
- Explain existing prescriptions only.
- Warn about contraindications.
- Flag red-flag symptoms.
- Tell the patient to seek emergency care when appropriate.
- Avoid replacing the doctor.

This is especially important because the feature deals with medical advice. The AI must act as an explainer and safety assistant, not as an independent doctor.

---

**4. process-prescription-ocr**

This function is for digitizing uploaded paper prescriptions.

The document explains that many patients in rural and semi-urban India have paper prescriptions from local clinics or hospitals. They may upload photos or scans to Trividha’s Health Vault.

Currently, those uploaded files are just stored as images or PDFs. That means the doctor has to manually open, zoom, inspect, and decipher the file.

The proposed OCR pipeline would automatically extract structured information from those prescription images.

OCR means Optical Character Recognition. It converts image text into machine-readable text.

The pipeline would work like this:

1. Patient uploads a prescription photo.
2. File lands in Supabase Storage, likely in the `medical-vault` bucket.
3. A storage/database hook triggers an Edge Function.
4. The Edge Function downloads the uploaded file.
5. It sends the file to an OCR engine.
6. OCR returns raw text and layout coordinates.
7. A medical extraction model parses the text.
8. The system extracts:
    - Doctor name
    - Registration number
    - Diagnosis
    - Symptoms or complaints
    - Medicines
    - Strength
    - Dosage
    - Frequency
    - Duration
    - Lab tests
9. The extracted result is saved as JSONB.
10. The patient or doctor gets a notification.

The Edge Function would live at:

```
supabase/functions/process-prescription-ocr/index.ts
```

The database needs extra fields such as:

```
extracted_entities JSONB
ocr_status VARCHAR(20) DEFAULT 'pending'
```

This would allow a doctor to quickly review the patient’s medical history in structured form instead of manually reading a blurry image.

---

**Summary of What the Document Is Saying**

The document is essentially identifying backend gaps in Trividha.

The frontend/admin UI already appears to expose some features, but the real operational backend is not fully implemented.

The missing pieces are serious because they involve:

- Real money movement to doctors.
- Real refunds to patients.
- Medical safety and explainability.
- Digitization of medical records.

The most production-critical functions are likely:

1. `refund-appointment`
2. `requestDoctorPayout`

Those involve money, compliance, disputes, and trust.

The most product-differentiating functions are:

1. `ask-medical-ai`
2. `process-prescription-ocr`

Those create the intelligent healthcare experience: patients understand care plans, and doctors get structured history instead of raw uploads.

In simple terms: the document is a backend implementation roadmap for turning mocked or planned Trividha features into real, auditable, production-grade systems.