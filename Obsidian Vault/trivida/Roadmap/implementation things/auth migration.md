# Roadmap: Migration from Phone OTP (Twilio) to Supabase Native Email OTP

## 1. Overview & Motivation

The Trividha platform currently uses SMS-based phone OTP authentication backed by Twilio via Supabase. To improve delivery reliability, eliminate per-SMS billing overheads, and streamline user onboarding, the auth system is transitioning to **Supabase Native Email OTP**.

### Comparison

| Feature / Aspect | Current Phone OTP (Twilio) | Target Email OTP (Supabase Native) |
| :--- | :--- | :--- |
| **Cost** | ₹0.50 – ₹1.00+ per SMS to Indian numbers | **₹0.00** (Free tier included) |
| **Third-Party Integrations** | Twilio / MSG91 credentials, DLT registration | **None** (Built directly into Supabase Auth) |
| **Reliability** | Dependent on telecom carriers, DLT templates, network reception | High reliability direct to user inbox |
| **Recoverability** | Ephemeral (lost if SMS deleted) | Persistent in user's email client |
| **Architectural Impact** | Phone number as unique identifier | Email as unique identifier |

---

## 2. Step-by-Step Implementation Roadmap

### Phase 1: Database Schema Migration (PostgreSQL / Supabase)
1. **Add `email` column**:
   Add a unique email column to `public.users`.
   ```sql
   ALTER TABLE public.users 
   ADD COLUMN email VARCHAR(255) UNIQUE;
   ```
2. **Data Migration (if migrating existing users)**:
   Backfill email addresses or assign temporary migration emails:
   ```sql
   UPDATE public.users 
   SET email = phone_number || '@temp.arogya.local' 
   WHERE email IS NULL;
   ```
3. **Drop `phone_number` column (or make it nullable during transition)**:
   ```sql
   ALTER TABLE public.users 
   DROP COLUMN phone_number;
   ```
4. **Performance Indexing**:
   ```sql
   CREATE INDEX idx_users_email ON public.users(email);
   ```



### Deep Dive: Phase 1 — Database Schema Migration (PostgreSQL / Supabase)  
  
 Migrating an authentication identifier in a production PostgreSQL database isn't just about changing column names—it involves constraints, indexes, transaction safety, and handling existing rows.  
 ──────  
 ### Step 1: Add email Column  
  
   ALTER TABLE public.users  
   ADD COLUMN email VARCHAR(255) UNIQUE;  
  
 #### What this does under the hood:  
  
 1. Column Addition (ADD COLUMN email VARCHAR(255)):  
     • Creates a new column named email capable of storing text strings up to 255 characters (standard RFC 5321 length recommendation for email addresses).  
     • Because no DEFAULT or NOT NULL is specified, existing rows receive NULL for this column. In modern PostgreSQL (11+), adding a nullable column without a default is a metadata-only operation and executes in milliseconds with  
     minimal table lock.  
 2. The UNIQUE Constraint:  
     • PostgreSQL automatically provisions a unique B-tree index behind the scenes (e.g., users_email_key) to enforce that no two accounts share the same email address.  
     • Crucial PostgreSQL Behavior with NULL: Under standard SQL / PostgreSQL rules, NULL values are treated as distinct from one another. This means you can have multiple existing rows with email IS NULL without violating the  
     UNIQUE constraint at this point.  
  
 ──────  
 ### Step 2: Data Migration (Backfilling Existing Users)  
  
   UPDATE public.users  
   SET email = phone_number || '@temp.arogya.local'  
   WHERE email IS NULL;  
  
 #### Why this step exists:  
  
 • If you have existing accounts that registered via phone numbers (e.g., +919876543210), their email field will be empty (NULL).  
 • If you later want to enforce a NOT NULL constraint or query user profiles by email, having missing emails will cause runtime lookup failures or unhandled exceptions in the client code.  
  
 #### How it works:  
  
 1. String Concatenation (phone_number || '@temp.arogya.local'):  
     • The || operator joins the existing phone number string with a dummy, deterministic domain (e.g., +919876543210@temp.arogya.local).  
     • Because phone_number was already unique per user, appending a constant domain guarantees the generated emails remain 100% unique, satisfying the UNIQUE constraint added in Step 1.  
 2. Filter Guard (WHERE email IS NULL):  
     • Ensures you only update rows that haven't already been populated, preventing overwriting legitimate emails if you run this script multiple times (idempotency).  
 3. Application Recovery Strategy:  
     • When users with @temp.arogya.local emails next open the app or request support, the client or admin portal can detect the temporary domain and prompt them: "Please update and verify your primary email address".  
  
 ──────  
 ### Step 3: Dropping or Nullifying phone_number  
  
   ALTER TABLE public.users  
   DROP COLUMN phone_number;  
  
 #### Architectural implications:  
  
 4. Irreversible Data Deletion:  
     • Dropping the column permanently deletes the phone numbers and removes any dependent indexes or check constraints on phone_number.  
 5. Production Staging vs. Fresh Database:  
     • On a Fresh/Development Project: Running DROP COLUMN phone_number immediately is completely safe and cleans up the schema cleanly.  
     • On a Live Production Database (Best Practice):  
     Instead of dropping the column immediately, the industry standard is a two-phase rollout:  
         • Phase 3A (Transition): Make the column nullable so new email-only registrations don't fail:  
           ALTER TABLE public.users ALTER COLUMN phone_number DROP NOT NULL;  
  
         • Phase 3B (Cleanup): Once the new app version is adopted by 100% of users, drop the column completely.  
  
  
 ──────  
 ### Step 4: Performance Indexing  
  
   CREATE INDEX idx_users_email ON public.users(email);  
  
 #### Why and how this operates:  
  
 6. Why do we index email?:  
     • During every login, token refresh, and profile check (getUserProfile(email) or checking if an account exists), the application runs:  
       SELECT * FROM public.users WHERE email = 'user@example.com' LIMIT 1;  
  
     • Without an index, PostgreSQL must execute a Sequential Scan (Seq Scan), scanning every single block in the table row by row (O(N) complexity).  
     • With an index, PostgreSQL performs an Index Scan (O(log N) complexity), locating the record in under 1 millisecond regardless of whether the table contains 100 or 1,000,000 users.  
 7. Note on Redundancy with UNIQUE in Step 1:  
 │ Note  
 │ When you create email VARCHAR(255) UNIQUE in Step 1, PostgreSQL automatically creates an index (named users_email_key).  
 │  
 │ Therefore, running an explicit CREATE INDEX idx_users_email is either:  
 │  
 │     • Omitable if you already declared UNIQUE in Step 1 (to save storage overhead of having two identical B-tree indexes).  
 │     • Or, if you prefer an explicit named index, you declare:  
 │       ALTER TABLE public.users ADD COLUMN email VARCHAR(255);  
 │       CREATE UNIQUE INDEX idx_users_email ON public.users (LOWER(email));  
 │     (Pro Tip: Using LOWER(email) creates a case-insensitive index so User@Example.com and user@example.com are treated identically!)  
  
 ──────  
 ### Recommended Production-Grade SQL Script  
  
 Putting it all together into an atomic, safe transaction block:  
  
   BEGIN;  
  
   -- 1. Add email column (case-insensitive citation)  
   ALTER TABLE public.users  
   ADD COLUMN IF NOT EXISTS email VARCHAR(255);  
  
   -- 2. Backfill existing phone users with deterministic temporary emails  
   UPDATE public.users  
   SET email = LOWER(phone_number || '@temp.arogya.local')  
   WHERE email IS NULL;  
  
   -- 3. Make email required (now that all rows have a value)  
   ALTER TABLE public.users  
   ALTER COLUMN email SET NOT NULL;  
  
   -- 4. Create case-insensitive unique index (prevents user@domain vs USER@domain duplicates)  
   CREATE UNIQUE INDEX IF NOT EXISTS idx_users_email_unique ON public.users (LOWER(email));  
  
   -- 5. Make phone_number optional (or drop if strictly no longer needed)  
   ALTER TABLE public.users  
   ALTER COLUMN phone_number DROP NOT NULL;  
  
   -- Optional: Drop column if you do not want to keep phone numbers at all  
   -- ALTER TABLE public.users DROP COLUMN phone_number;  
  
   COMMIT;


---

### Phase 2: Supabase Console Configuration
1. Open **Authentication** → **Providers** → **Email** in the Supabase Dashboard.
2. Ensure **Enable Email provider** is active and toggle **Confirm email / Email OTP**.
3. Set **OTP expiry** to `600` seconds (10 minutes).
4. Configure the Email Template with customized branding:
   ```html
   <h2>Trividha Healthcare Verification</h2>
   <p>Your 6-digit login/registration security code is:</p>
   <h1 style="letter-spacing: 4px; color: #00796B;">{{ .Token }}</h1>
   <p>This code expires in 10 minutes. Do not share this code with anyone.</p>
   ```



### Deep Dive: Phase 2 — Supabase Console Configuration (Email OTP)  
 Configuring Supabase's authentication service for Email OTP shifts the authentication mechanism from an external SMS aggregator (Twilio) to Supabase's built-in email infrastructure (GoTrue / Supabase Auth engine).  
 ──────  
 ### Step 1: Navigating to Authentication Providers  
 1. In your web browser, navigate to your Supabase Dashboard https://supabase.com/dashboard.  
 2. Select your project (trividha).  
 3. In the left-hand navigation sidebar:  
     • Click the Authentication icon (the padlock / shield icon).  
     • Under the Configuration subsection, click on Providers.  
     • In the list of providers (Phone, Google, Apple, GitHub, etc.), select Email.  
  
 ──────  
 ### Step 2: Enabling Email Provider & Configuring Email OTP  
  
 Once inside the Email provider settings:  
  
 4. Toggle "Enable Email provider" (ON):  
     • This activates Supabase's internal auth engine for email identities.  
     • It allows calling methods like signInWithOtp({ email: ... }) from the Flutter SDK.  
 5. Understanding the Two Email Auth Modes:  
 Supabase offers two distinct modes for email authentication:  
     • Magic Link (Default URL redirect): Sends a clickable hyperlink (https://<project>.supabase.co/auth/v1/verify?token=...&type=email&redirect_to=...). While great for web apps, magic links on mobile require deep  
     linking/universal links (trividha://...), which can break or fail if users open the email on a laptop instead of the mobile phone where the app is installed.  
     • Email OTP (One-Time Password / Token): Sends a plain 6-digit numeric verification code directly in the email body.  
 6. Toggle "Confirm email" / "Secure email change":  
     • Make sure "Confirm email" is enabled. When a new user logs in via Email OTP, Supabase will only create/validate their verified session once the 6-digit code has been correctly submitted.  
  
 ──────  
 ### Step 3: Setting OTP Expiry (TTL / Security Limits)  
  
 7. Still inside Authentication → Navigate to Rate Limits or Email Auth Settings (or within the Email provider card depending on your Supabase dashboard version).  
 8. Look for OTP Expiry (seconds) (default is often 3600 seconds / 1 hour):  
     • Change this value to 600 seconds (10 minutes).  
  
  
 #### Why 600 seconds (10 minutes)?  
  
 • Balance between Security and User Experience (UX):  
     • In Tier-2/3 Indian regions or areas with intermittent internet/poor network connectivity, email sync may take 30–60 seconds. A short 60-second window is too stressful for users.  
     • A 1-hour window (3600s), however, is insecure: if someone leaves an unverified email sitting around, anyone with brief device access can reuse that code.  
     • 10 minutes (600s) matches the industry standard (used by banks, Aadhaar/ABHA, and telemedicine portals) and aligns with the app's countdown timer in resend_countdown_timer.dart.  
  
 ──────  
 ### Step 4: Configuring the Email Template with Custom Branding  
  
 When Supabase Auth generates an OTP code, it compiles an email message using the Go HTML template engine and sends it out.  
  
 9. In the Supabase Dashboard, navigate to:  
 Authentication → Email Templates (in the left-hand sub-menu under Configuration).  
 10. Select the template named "Magic Link / Confirmation" or "Email Change / Confirmation" (in OTP mode, this is the template sent when calling signInWithOtp).  
 11. You will see two fields:  
     • Subject: Change this to something recognizable and professional:  
       Your Trividha Security Verification Code: {{ .Token }}  
  
     • Body (HTML): Replace the generic default template with our branded template:  
  
   <!DOCTYPE html>  
   <html>  
   <head>  
     <meta charset="utf-8">  
     <meta name="viewport" content="width=device-width, initial-scale=1.0">  
     <style>  
       body {  
         font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;  
         background-color: #F8FAF9;  
         margin: 0;  
         padding: 24px;  
         color: #1A2E2B;  
       }  
       .card {  
         max-width: 480px;  
         margin: 0 auto;  
         background: #FFFFFF;  
         border-radius: 12px;  
         border: 1px solid #E2E8F0;  
         padding: 32px;  
         box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.05);  
       }  
       .header {  
         font-size: 20px;  
         font-weight: 700;  
         color: #0D6E6E;  
         margin-bottom: 12px;  
       }  
       .otp-box {  
         background: #E6F4F1;  
         border-radius: 8px;  
         padding: 16px;  
         text-align: center;  
         margin: 24px 0;  
       }  
       .otp-code {  
         font-size: 32px;  
         font-weight: 800;  
         letter-spacing: 6px;  
         color: #0D6E6E;  
         margin: 0;  
         font-family: monospace, monospace;  
       }  
       .footer {  
         font-size: 12px;  
         color: #718096;  
         margin-top: 24px;  
         border-top: 1px solid #E2E8F0;  
         padding-top: 16px;  
       }  
     </style>  
   </head>  
   <body>  
     <div class="card">  
       <div class="header">🩺 त्रिविधा • Trividha Healthcare</div>  
           <p>Please use the following 6-digit verification code to complete your login or registration:</p>  
  
       <div class="otp-box">  
         <div class="otp-code">{{ .Token }}</div>  
       </div>  
  
       <p>This code will expire in <strong>10 minutes</strong>. For your security, do not share this code with anyone.</p>  
  
       <div class="footer">  
         If you did not request this verification code, you can safely ignore this email.  
       </div>  
     </div>  
   </body>  
   </html>  

 #### What does {{ .Token }} do?  
  
 • In Supabase GoTrue templates, {{ .Token }} (or {{ .Code }} depending on version) is the dynamic variable placeholder where Supabase injects the generated 6-digit numeric string (e.g. 481920).  
 • By placing it prominently in an .otp-code container with #0D6E6E (Trividha's primary teal brand color), users immediately see their code without having to search through text or click any external links.  
 ──────  
 ### Step 5: How the Sending Works (Behind the Scenes)  
  
 • Zero Extra Credentials Needed: Unlike Twilio where you had to configure Account SID, Auth Token, and Sender IDs, Supabase uses its internal transactional email sender service (noreply@mail.app.supabase.io) by default.  
 • Future Custom SMTP (Optional): If you later decide to send emails from your own domain (e.g., verify@trividha.com), you can simply go to Authentication → Email Settings → Custom SMTP and plug in SendGrid, Amazon SES, Resend, or  
 Postmark credentials without changing a single line of your Flutter app code!



---


### Phase 3: Data Layer & Repository Updates

#### 1. Update `UserModel` (`lib/models/user_model.dart`)
- Replace `final String phoneNumber` with `final String email`.
- Update `fromJson` and `toJson` serialization to handle `email`.

#### 2. Update `IAuthRepository` (`lib/features/auth/data/auth_repository.dart`)
- Update interface signatures:
  - `Future<String> sendOtp({required String email});`
  - `Future<String> resendOtp({required String email});`
  - `Future<UserModel> verifyOtp({required String verificationId, required String smsCode, required String email});`
  - `Future<UserModel> registerUser({required String uid, required String email, required UserRole role, required String fullName, required LanguageCode preferredLanguage, Map<String, dynamic>? roleProfileData});`

#### 3. Update `SupabaseAuthRepository` (`lib/features/auth/data/supabase_auth_repository.dart`)
- Update `sendOtp`:
  ```dart
  await _supabaseClient.auth.signInWithOtp(
    email: email.trim(),
    shouldCreateUser: true,
  );
  ```
- Update `verifyOtp`:
  ```dart
  final res = await _supabaseClient.auth.verifyOTP(
    email: email.trim(),
    token: smsCode.trim(),
    type: OtpType.email,
  );
  ```
- Update `registerUser`:
  - Upsert `email` into `public.users` table instead of `phone_number`.

#### 4. Update Mock `AuthRepository` (`lib/features/auth/data/auth_repository.dart`)
- Update mock methods to accept and validate email format.
- Generate mock `uid` from email (e.g. `'user_${email.split('@').first}'`).
- Maintain test OTP `'123456'` for offline development.





### Deep Dive: Phase 3 — Data Layer & Repository Updates  
  
 The Data Layer is the architectural bridge between your Flutter UI/state and the backend services. Because Trividha uses the Repository Pattern and clean domain separation, transitioning from phone to email authentication only  
 requires swapping the contact identifier across 4 targeted files without rewriting business rules.  
 ──────  
 ### Step 1: Update UserModel (user_model.dart)  
  
 UserModel represents an authenticated user's core identity in memory across the entire Flutter application.  
  
 #### Detailed Code Changes:  
  
   class UserModel {  
     final String uid;  
     // 🔄 REPLACED: final String phoneNumber; -> final String email;  
     final String email;  
     final UserRole role;  
     final String fullName;  
     final LanguageCode preferredLanguage;  
     final DateTime createdAt;  
  
     const UserModel({  
       required this.uid,  
       required this.email, // 👈 Required field  
       required this.role,  
       required this.fullName,  
       this.preferredLanguage = LanguageCode.hi,  
       required this.createdAt,  
     });  
  
     factory UserModel.fromJson(Map<String, dynamic> json) {  
       return UserModel(  
         uid: json['uid'] as String? ?? json['id'] as String? ?? '',  
         // 🔄 Look for 'email'. Fallback to empty string for null-safety:  
         email: json['email'] as String? ?? '',  
         role: UserRole.values.firstWhere(  
           (r) => r.name == json['role'],  
           orElse: () => UserRole.patient,  
         ),  
         fullName: json['fullName'] as String? ?? json['full_name'] as String? ?? '',  
         preferredLanguage: LanguageCode.values.firstWhere(  
           (l) => l.name == (json['preferredLanguage'] ?? json['preferred_language']),  
           orElse: () => LanguageCode.hi,  
         ),  
         createdAt: json['createdAt'] != null  
             ? DateTime.tryParse(json['createdAt'].toString()) ?? DateTime.now()  
             : (json['created_at'] != null  
                 ? DateTime.tryParse(json['created_at'].toString()) ?? DateTime.now()  
                 : DateTime.now()),  
       );  
     }  
  
     Map<String, dynamic> toJson() {  
       return {  
         'uid': uid,  
         'email': email, // 👈 Serializes email instead of phoneNumber  
         'role': role.name,  
         'fullName': fullName,  
         'preferredLanguage': preferredLanguage.name,  
         'createdAt': createdAt.toIso8601String(),  
       };  
     }  
   }  
  
 #### Why this is important:  
  
 • Null Safety: Guaranteeing email is non-null prevents random null pointer exceptions in downstream widgets (like profile cards or drawer headers).  
 • Dual Snake/Camel Case parsing: PostgreSQL columns use snake_case (full_name, email), whereas frontend code often serializes camelCase (fullName). Handling both ensures database queries and local caching both deserialize smoothly.  
 ──────  
 ### Step 2: Update IAuthRepository (auth_repository.dart)  
  
 The abstract interface defines the strict contract that any authentication service (mock, Supabase, Firebase, etc.) must fulfill.  
  
 #### Detailed Code Changes:  
  
   abstract class IAuthRepository {  
     // 🔄 Signatures updated from phoneNumber -> email  
     Future<String> sendOtp({required String email});  
  
     Future<String> resendOtp({required String email});  
  
     Future<UserModel> verifyOtp({  
       required String verificationId,  
       required String smsCode, // Note: can keep name smsCode or rename to otpCode  
       required String email,  
     });  
  
     Future<UserModel> registerUser({  
       required String uid,  
       required String email,  
       required UserRole role,  
       required String fullName,  
       required LanguageCode preferredLanguage,  
       Map<String, dynamic>? roleProfileData,  
     });  
  
     Future<UserModel?> getUserProfile(String uid);  
     Future<UserModel?> getCurrentUser();  
     void clearSession();  
     Future<void> signOut();  
   }  
  
 #### Why this is important:  
  
 • By enforcing required String email in the abstract class, the Dart compiler will immediately highlight every location in your code that was passing a phone number, making refactoring safe, deterministic, and error-free.  
 ──────  
 ### Step 3: Update SupabaseAuthRepository (supabase_auth_repository.dart)  
  
 This is the production implementation that directly invokes the Supabase Flutter SDK and PostgreSQL tables.  
  
 #### 1. Modifying sendOtp:  
  
   @override  
   Future<String> sendOtp({required String email}) async {  
     // 🛡 Guard against stale pending callbacks from previous attempts  
     if (_pendingVerification != null && !_pendingVerification!.isCompleted) {  
       _pendingVerification!.completeError(  
         Exception('Previous verification cancelled by new request.'),  
       );  
     }  
  
     final currentSequence = ++_sessionSequence;  
     final thisCompleter = Completer<String>();  
     _pendingVerification = thisCompleter;  
  
     if (_supabaseClient == null) {  
       // Offline/Mock fallback  
       Future.delayed(const Duration(milliseconds: 200), () {  
         if (!thisCompleter.isCompleted && currentSequence == _sessionSequence) {  
           thisCompleter.complete('ver_${DateTime.now().millisecondsSinceEpoch}');  
         }  
       });  
       return thisCompleter.future;  
     }  
  
     try {  
       // 🚀 NEW: signInWithOtp using email instead of phone  
       await _supabaseClient.auth.signInWithOtp(  
         email: email.trim().toLowerCase(),  
         shouldCreateUser: true, // Automatically registers temporary auth identity  
       );  
       _resendTokenTimestamp = DateTime.now();  
       if (!thisCompleter.isCompleted && currentSequence == _sessionSequence) {  
         thisCompleter.complete('ver_${DateTime.now().millisecondsSinceEpoch}');  
       }  
     } catch (e) {  
       if (!thisCompleter.isCompleted && currentSequence == _sessionSequence) {  
         thisCompleter.completeError(  
           Exception(e.toString().replaceAll('AuthException:', '').trim()),  
         );  
       }  
     }  
  
     return thisCompleter.future;  
   }  
  
 • shouldCreateUser: true: Crucial! If this is false, new users attempting to register would receive a "User not found" error. Setting it to true allows both login (existing users) and registration (new users) to proceed through the  
 same email entry point.  
  
 #### 2. Modifying verifyOtp:  
  
   @override  
   Future<UserModel> verifyOtp({  
     required String verificationId,  
     required String smsCode,  
     required String email,  
   }) async {  
     final cleanEmail = email.trim().toLowerCase();  
  
     if (_supabaseClient == null) {  
       // Offline test fallback  
       await Future.delayed(const Duration(milliseconds: 700));  
       if (smsCode != '123456' && !smsCode.startsWith('123')) {  
         throw Exception('Invalid OTP. Please enter a valid 6-digit code.');  
       }  
       return UserModel(  
         uid: 'user_${cleanEmail.split('@').first}',  
         email: cleanEmail,  
         role: UserRole.patient,  
         fullName: '',  
         preferredLanguage: LanguageCode.hi,  
         createdAt: DateTime.now(),  
       );  
     }  
  
     try {  
       // 🚀 NEW: verifyOTP with OtpType.email  
       final res = await _supabaseClient.auth.verifyOTP(  
         email: cleanEmail,  
         token: smsCode.trim(),  
         type: OtpType.email,  
       );  
  
       final user = res.user;  
       if (user == null) {  
         throw Exception('Authentication failed. No user returned.');  
       }  
  
       // Check if user already completed onboarding in public.users  
       final profile = await getUserProfile(user.id);  
       if (profile != null) {  
         return profile; // Existing user -> Login completed!  
       }  
  
       // New user without completed profile -> Proceed to Role Selection  
       return UserModel(  
         uid: user.id,  
         email: user.email ?? cleanEmail,  
         role: UserRole.patient,  
         fullName: '', // Empty fullName triggers onboarding navigation  
         preferredLanguage: LanguageCode.hi,  
         createdAt: DateTime.now(),  
       );  
     } catch (e) {  
       throw Exception('Invalid OTP code. Please enter the valid code sent to $email.');  
     }  
   }  
  
 #### 3. Modifying registerUser:  
  
   @override  
   Future<UserModel> registerUser({  
     required String uid,  
     required String email,  
     required UserRole role,  
     required String fullName,  
     required LanguageCode preferredLanguage,  
     Map<String, dynamic>? roleProfileData,  
   }) async {  
     final now = DateTime.now();  
     final cleanEmail = email.trim().toLowerCase();  
  
     final userModel = UserModel(  
       uid: uid,  
       email: cleanEmail,  
       role: role,  
       fullName: fullName.trim(),  
       preferredLanguage: preferredLanguage,  
       createdAt: now,  
     );  
  
     if (_supabaseClient != null) {  
       try {  
         // 1. Insert/Update public.users with 'email' column instead of 'phone_number'  
         await _supabaseClient.from('users').upsert({  
           'id': uid,  
           'email': cleanEmail, // 👈 Upsert email!  
           'full_name': fullName.trim(),  
           'role': role.name,  
           'preferred_language': preferredLanguage.name,  
           'created_at': now.toIso8601String(),  
         });  
  
         // 2. Insert into role-specific table (patients or doctors)  
         if (role == UserRole.doctor) {  
           await _supabaseClient.from('doctors').upsert({  
             'id': uid,  
             'full_name': fullName.trim(),  
             'system_of_medicine': roleProfileData?['systemOfMedicine'] ?? 'allopathy',  
             'council_registration_number': roleProfileData?['councilRegistrationNumber'] ?? '',  
             'specialty': roleProfileData?['specialty'] ?? 'General Medicine',  
             'experience_years': roleProfileData?['experienceYears'] ?? 0,  
             'consultation_fee': roleProfileData?['consultationFee'] ?? 500,  
             'is_available': true,  
             'rating': 5.0,  
             'review_count': 0,  
             'created_at': now.toIso8601String(),  
           });  
         } else {  
           await _supabaseClient.from('patients').upsert({  
             'id': uid,  
             'full_name': fullName.trim(),  
             'age': roleProfileData?['age'] ?? 30,  
             'gender': roleProfileData?['gender'] ?? 'other',  
             'blood_group': roleProfileData?['bloodGroup'] ?? 'Unknown',  
             'abha_number': roleProfileData?['abhaNumber'] ?? '',  
             'abha_address': roleProfileData?['abhaAddress'] ?? '',  
             'chronic_conditions': roleProfileData?['chronicConditions'] ?? [],  
             'allergies': roleProfileData?['allergies'] ?? [],  
             'created_at': now.toIso8601String(),  
           });  
         }  
       } catch (e) {  
         if (e.toString().contains('permission-denied') || e.toString().contains('row-level security')) {  
           throw Exception('Security rules permission denied. Please verify user authentication credentials.');  
         }  
         rethrow;  
       }  
     }  
  
     return userModel;  
   }  
 ──────  
 ### Step 4: Update Mock AuthRepository (auth_repository.dart)  
  
 Used in widget tests, integration tests, and local simulator sessions when running without live Supabase credentials.  
  
   class AuthRepository implements IAuthRepository {  
     final Map<String, UserModel> _mockUserDb = {};  
  
     @override  
     Future<String> sendOtp({required String email}) async {  
       await Future.delayed(const Duration(milliseconds: 300));  
       final cleanEmail = email.trim().toLowerCase();  
  
       // Basic format validation  
       if (!cleanEmail.contains('@') || !cleanEmail.contains('.')) {  
         throw Exception('Invalid email address format.');  
       }  
       return 'ver_${DateTime.now().millisecondsSinceEpoch}';  
     }  
  
     @override  
     Future<String> resendOtp({required String email}) async {  
       return sendOtp(email: email);  
     }  
  
     @override  
     Future<UserModel> verifyOtp({  
       required String verificationId,  
       required String smsCode,  
       required String email,  
     }) async {  
       await Future.delayed(const Duration(milliseconds: 400));  
       if (smsCode != '123456' && !smsCode.startsWith('123')) {  
         throw Exception('Invalid OTP. Please enter a valid 6-digit code (Demo code: 123456).');  
       }  
  
       final cleanEmail = email.trim().toLowerCase();  
       final uid = 'user_${cleanEmail.split('@').first.replaceAll(RegExp(r'[^a-zA-Z0-9]'), '')}';  
  
       // If user already registered in mock memory, return full profile  
       if (_mockUserDb.containsKey(uid)) {  
         return _mockUserDb[uid]!;  
       }  
  
       // New mock user  
       return UserModel(  
         uid: uid,  
         email: cleanEmail,  
         role: UserRole.patient,  
         fullName: '',  
         preferredLanguage: LanguageCode.hi,  
         createdAt: DateTime.now(),  
       );  
     }  
  
     @override  
     Future<UserModel> registerUser({  
       required String uid,  
       required String email,  
       required UserRole role,  
       required String fullName,  
       required LanguageCode preferredLanguage,  
       Map<String, dynamic>? roleProfileData,  
     }) async {  
       await Future.delayed(const Duration(milliseconds: 300));  
  
       final user = UserModel(  
         uid: uid,  
         email: email.trim().toLowerCase(),  
         role: role,  
         fullName: fullName.trim(),  
         preferredLanguage: preferredLanguage,  
         createdAt: DateTime.now(),  
       );  
  
       _mockUserDb[uid] = user;  
       return user;  
     }  
  
     @override  
     Future<UserModel?> getUserProfile(String uid) async {  
       await Future.delayed(const Duration(milliseconds: 200));  
       return _mockUserDb[uid];  
     }  
  
     @override  
     Future<UserModel?> getCurrentUser() async => null;  
  
     @override  
     void clearSession() {}  
  
     @override  
     Future<void> signOut() async {  
       await Future.delayed(const Duration(milliseconds: 150));  
     }  
   }  
 ──────  
 ### Key Architectural Benefits of this Refactoring:  
  
 1. Zero Breaking Changes to Domain Logic: Downstream models like PatientModel and DoctorModel stay exactly the same.  
 2. Determinism: Case-normalizing with .trim().toLowerCase() prevents duplicate users (e.g. User@Gmail.com vs user@gmail.com).  
 3. Seamless Test Compatibility: All unit/widget tests relying on AuthRepository continue working offline with test OTP 123456.




---

### Phase 4: State Management (`Riverpod`)

#### 1. Update `AuthState` (`lib/features/auth/domain/auth_state.dart`)
- Replace `phoneNumber` with `email` in constructor, properties, and factory methods (`codeSent`, `authenticating`, `error`, `copyWith`).

#### 2. Update `AuthNotifier` (`lib/features/auth/presentation/providers/auth_provider.dart`)
- Update `sendOtp(String email)`: validate email format, guard against double-taps, update state with `email`.
- Update `verifyOtp(String code)`: pass `state.email` to repository.
- Update `registerUser(...)`: forward email to repository.

#### 3. Update `bootstrapProvider` (`lib/features/auth/presentation/providers/bootstrap_provider.dart`)
- Update `BootstrapIncompleteRegistration` class to hold `email` instead of `phoneNumber`.






### Deep Dive: Phase 4 — State Management (Riverpod)  
  
 State management controls how data travels between user actions in the UI, asynchronous background calls in the repository layer, and app-wide state listeners. In Riverpod, state objects are immutable, and transitions are managed  
 through a centralized StateNotifier.  
 ──────  
 ### Step 1: Update AuthState (auth_state.dart)  
  
 AuthState models the state machine of the authentication flow:  
 initial → authenticating (sending OTP) → codeSent → authenticating (verifying code) → authenticated (or error).  
 #### Detailed Code Changes:  
   import '../../../models/user_model.dart';  
  
   enum AuthStatus {  
     initial,  
     codeSent,  
     authenticating,  
     authenticated,  
     error,  
   }  
  
   class AuthState {  
     final AuthStatus status;  
     // 🔄 REPLACED: final String? phoneNumber; -> final String? email;  
     final String? email;  
     final String? verificationId;  
     final UserModel? user;  
     final String? errorMessage;  
     final int resendTimeout;  
  
     const AuthState({  
       this.status = AuthStatus.initial,  
       this.email,  
       this.verificationId,  
       this.user,  
       this.errorMessage,  
       this.resendTimeout = 45,  
     });  
  
     factory AuthState.initial() => const AuthState();  
  
     // 1. When OTP email is dispatched  
     factory AuthState.codeSent({  
       required String email,  
       required String verificationId,  
       int resendTimeout = 45,  
     }) =>  
         AuthState(  
           status: AuthStatus.codeSent,  
           email: email,  
           verificationId: verificationId,  
           resendTimeout: resendTimeout,  
         );  
  
     // 2. While waiting for network responses (prevents double-taps)  
     factory AuthState.authenticating({String? email, String? verificationId}) =>  
         AuthState(  
           status: AuthStatus.authenticating,  
           email: email,  
           verificationId: verificationId,  
         );  
  
     // 3. User verified or profile loaded  
     factory AuthState.authenticated(UserModel user) => AuthState(  
           status: AuthStatus.authenticated,  
           user: user,  
         );  
  
     // 4. Any error state (preserves active email so user doesn't re-type it)  
     factory AuthState.error(String errorMessage, {String? email, String? verificationId}) =>  
         AuthState(  
           status: AuthStatus.error,  
           errorMessage: errorMessage,  
           email: email,  
           verificationId: verificationId,  
         );  
  
     // 5. Immutable copyWith  
     AuthState copyWith({  
       AuthStatus? status,  
       String? email,  
       String? verificationId,  
       UserModel? user,  
       String? errorMessage,  
       int? resendTimeout,  
     }) {  
       return AuthState(  
         status: status ?? this.status,  
         email: email ?? this.email,  
         verificationId: verificationId ?? this.verificationId,  
         user: user ?? this.user,  
         errorMessage: errorMessage ?? this.errorMessage,  
         resendTimeout: resendTimeout ?? this.resendTimeout,  
       );  
     }  
   }  
  
 #### Why this is important:  
  
 • Error Retention: When an OTP attempt errors out (e.g. wrong code entered), preserving email: state.email in AuthState.error(...) ensures the verification screen doesn't wipe the target email or break the user's resend timer.  
 ──────  
 ### Step 2: Update AuthNotifier (auth_provider.dart)  
  
 AuthNotifier encapsulates all business operations and concurrency guards.  
  
 #### Detailed Code Changes:  
  
   class AuthNotifier extends StateNotifier<AuthState> {  
     final IAuthRepository _repository;  
     final TelemetryService? _telemetry;  
     final void Function(String uid)? _onAuthenticated;  
  
     AuthNotifier(  
       this._repository, [  
       this._telemetry,  
       this._onAuthenticated,  
     ]) : super(AuthState.initial());  
  
     // ==========================================  
     // 1. Request OTP via Email  
     // ==========================================  
     Future<bool> sendOtp(String email) async {  
       // 🛡 Rapid Double-Tap Guard: ignore concurrent clicks while processing  
       if (state.status == AuthStatus.authenticating) return false;  
  
       final cleanEmail = email.trim().toLowerCase();  
  
       // Quick regex validation before making network requests  
       final emailRegex = RegExp(r'^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$');  
       if (!emailRegex.hasMatch(cleanEmail)) {  
         state = AuthState.error('Please enter a valid email address.');  
         return false;  
       }  
  
       state = AuthState.authenticating(email: cleanEmail);  
       _telemetry?.logEvent(AuthTelemetryEvent.otpRequested, parameters: {'email': cleanEmail});  
  
       try {  
         final verificationId = await _repository.sendOtp(email: cleanEmail);  
         state = AuthState.codeSent(  
           email: cleanEmail,  
           verificationId: verificationId,  
         );  
         return true;  
       } catch (e) {  
         _telemetry?.logError('sendOtp failed', error: e);  
         state = AuthState.error(  
           e.toString().replaceAll('Exception:', '').trim(),  
           email: cleanEmail,  
         );  
         return false;  
       }  
     }  
  
     // ==========================================  
     // 2. Verify OTP Code  
     // ==========================================  
     Future<bool> verifyOtp(String smsCode) async {  
       // 🛡 Rapid Double-Tap Guard  
       if (state.status == AuthStatus.authenticating) return false;  
  
       if (state.verificationId == null || state.email == null) {  
         state = AuthState.error('Verification session expired. Please request a new OTP.');  
         return false;  
       }  
  
       final verificationId = state.verificationId!;  
       final email = state.email!;  
  
       state = AuthState.authenticating(  
         email: email,  
         verificationId: verificationId,  
       );  
  
       try {  
         final user = await _repository.verifyOtp(  
           verificationId: verificationId,  
           smsCode: smsCode.trim(),  
           email: email, // 👈 Passes email to repository  
         );  
         _telemetry?.logEvent(AuthTelemetryEvent.otpVerified, parameters: {'uid': user.uid});  
         state = AuthState.authenticated(user);  
         _onAuthenticated?.call(user.uid);  
         return true;  
       } catch (e) {  
         _telemetry?.logError('verifyOtp failed', error: e);  
         state = AuthState.error(  
           e.toString().replaceAll('Exception:', '').trim(),  
           email: email,  
           verificationId: verificationId,  
         );  
         return false;  
       }  
     }  
  
     // ==========================================  
     // 3. Resend OTP  
     // ==========================================  
     Future<bool> resendOtp() async {  
       if (state.email == null || state.status == AuthStatus.authenticating) return false;  
       return await sendOtp(state.email!);  
     }  
  
     void resetError() {  
       if (state.status == AuthStatus.error) {  
         if (state.verificationId != null && state.email != null) {  
           state = AuthState.codeSent(  
             email: state.email!,  
             verificationId: state.verificationId!,  
           );  
         } else {  
           state = AuthState.initial();  
         }  
       }  
     }  
  
     // ==========================================  
     // 4. Register / Onboard User Profile  
     // ==========================================  
     Future<bool> registerUser({  
       required UserRole role,  
       required String fullName,  
       required LanguageCode preferredLanguage,  
       Map<String, dynamic>? roleProfileData,  
     }) async {  
       if (state.status == AuthStatus.authenticating) return false;  
       if (state.user == null) {  
         state = AuthState.error('Authentication required before onboarding.');  
         return false;  
       }  
  
       final currentUser = state.user!;  
       state = AuthState.authenticating(  
         email: currentUser.email,  
         verificationId: state.verificationId,  
       );  
  
       _telemetry?.logEvent(AuthTelemetryEvent.registrationStarted, parameters: {  
         'uid': currentUser.uid,  
         'role': role.name,  
       });  
  
       const maxRetries = 3;  
       int attempt = 0;  
  
       while (attempt < maxRetries) {  
         try {  
           final user = await _repository.registerUser(  
             uid: currentUser.uid,  
             email: currentUser.email, // 👈 Passes verified email  
             role: role,  
             fullName: fullName,  
             preferredLanguage: preferredLanguage,  
             roleProfileData: roleProfileData,  
           );  
  
           _telemetry?.logEvent(AuthTelemetryEvent.registrationCompleted, parameters: {  
             'uid': user.uid,  
             'role': role.name,  
             'attempt': attempt + 1,  
           });  
  
           state = AuthState.authenticated(user);  
           _onAuthenticated?.call(user.uid);  
           return true;  
         } catch (e) {  
           attempt++;  
           if (attempt >= maxRetries) {  
             _telemetry?.logEvent(AuthTelemetryEvent.registrationFailed, parameters: {  
               'uid': currentUser.uid,  
               'role': role.name,  
               'error': e.toString(),  
               'attempts': attempt,  
             });  
             state = AuthState.error(  
               'Registration failed. Please tap retry.',  
               email: currentUser.email,  
               verificationId: state.verificationId,  
             );  
             return false;  
           }  
           await Future.delayed(Duration(milliseconds: 500 * (1 << (attempt - 1))));  
         }  
       }  
  
       return false;  
     }  
   }  
 ──────  
 ### Step 3: Update bootstrapProvider (bootstrap_provider.dart)  
  
 When a user opens the app, bootstrapProvider runs to check for an existing session and handle "Ghost Auth Recovery" (a scenario where a user successfully verified their OTP in Supabase Auth, but closed the app before completing  
 their profile in public.users).  
  
 #### Detailed Code Changes:  
  
   sealed class AuthBootstrapState {  
     const AuthBootstrapState();  
   }  
  
   class BootstrapUnauthenticated extends AuthBootstrapState {  
     const BootstrapUnauthenticated();  
   }  
  
   // 🔄 Updated: Holds email instead of phoneNumber  
   class BootstrapIncompleteRegistration extends AuthBootstrapState {  
     final String uid;  
     final String? email;  
     const BootstrapIncompleteRegistration({required this.uid, this.email});  
   }  
  
   class BootstrapAuthenticated extends AuthBootstrapState {  
     final UserModel user;  
     const BootstrapAuthenticated(this.user);  
   }  
  
   class BootstrapError extends AuthBootstrapState {  
     final String message;  
     const BootstrapError(this.message);  
   }  
  
   final bootstrapProvider = FutureProvider<AuthBootstrapState>((ref) async {  
     final authRepo = ref.watch(authRepositoryProvider);  
  
     // 1. Check if active Supabase session exists  
     final currentUser = await authRepo.getCurrentUser();  
     if (currentUser == null) {  
       final authState = ref.watch(authNotifierProvider);  
       if (authState.user != null && authState.user!.fullName.isNotEmpty) {  
         Future.microtask(() {  
           ref.read(notificationNotifierProvider.notifier).initialize(authState.user!.uid);  
         });  
         return BootstrapAuthenticated(authState.user!);  
       }  
       return const BootstrapUnauthenticated();  
     }  
  
     // 2. Query Supabase public.users table  
     try {  
       final profile = await authRepo.getUserProfile(currentUser.uid);  
       if (profile != null && profile.fullName.isNotEmpty) {  
         Future.microtask(() {  
           ref.read(notificationNotifierProvider.notifier).initialize(profile.uid);  
         });  
         return BootstrapAuthenticated(profile);  
       } else {  
         // 🛡 Ghost Auth Recovery: Supabase Auth user exists, public.users row missing  
          return BootstrapIncompleteRegistration(  
            uid: currentUser.uid,  
            email: currentUser.email, // 👈 Passes email to onboarding screen  
          );  
        }  
      } catch (e) {  
        return BootstrapError(e.toString().replaceAll('Exception:', '').trim());  
      }  
    });  
 ──────  
 ### Architectural Flow Summary  
  
   📊 Diagram (unsupported type)  
   ──────────────────────────────  
   stateDiagram-v2  
       [*] --> Unauthenticated: App Launch (No session)  
       [*] --> IncompleteRegistration: Ghost Auth (Auth user exists, no public.users row)  
       [*] --> Authenticated: Session Valid + Profile complete  
  
       Unauthenticated --> Authenticating: sendOtp(email)  
       Authenticating --> CodeSent: OTP Sent via Supabase  
       Authenticating --> Error: Invalid email or network drop  
  
       CodeSent --> Authenticating: verifyOtp(token)  
       Authenticating --> IncompleteRegistration: First-time user (fullName empty)  
       Authenticating --> Authenticated: Existing user (fullName present)  
  
       IncompleteRegistration --> Authenticated: registerUser(role, fullName)  
  
 With these 3 updates:  
  
 1. All reactive components receive type-safe email state.  
 2. Rapid double-taps and concurrent clicks are gracefully discarded.  
 3. Ghost Auth recovery cleanly pre-fills the user's verified email if their profile setup was interrupted.








---

### Phase 5: UI & Presentation Layer

#### 1. Convert `PhoneLoginScreen` → `EmailLoginScreen`
- Rename `lib/features/auth/presentation/screens/phone_login_screen.dart` to `email_login_screen.dart`.
- Change input field:
  - `keyboardType: TextInputType.emailAddress`.
  - Icon: Email icon instead of country code `+91` flag badge.
  - Validator: Validate presence of `@`, valid domain, and standard email regex.
- Update localized strings (English, Hindi, Gujarati) for email prompts and validation messages.

#### 2. Update `OtpVerificationScreen`
- Update header text: "Code sent to: {email}".
- Allow user to navigate back and edit their email address.

#### 3. Update `RoleSelectionScreen`
- Replace phone number badge with email indicator (`✉️ {email}`).

#### 4. Update Navigation Entry Points
- Update `main.dart`, `patient_home_dashboard.dart`, `doctor_dashboard_screen.dart`, and `ui_showcase_screen.dart` to reference `EmailLoginScreen`.







### Deep Dive: Phase 5 — UI & Presentation Layer  
 The presentation layer is what users and practitioners interact with directly. In Trividha, this layer is localized across Hindi, Gujarati, and English, follows Material 3 guidelines, and enforces DPDPA security standards.  
 ──────  
 ### Step 1: Convert PhoneLoginScreen → EmailLoginScreen  
  
 Rename phone_login_screen.dart to email_login_screen.dart.  
  
 #### 1. Controller & Handler Changes  
 Replace phone-specific parsing with clean email trimming:  
  
   class EmailLoginScreen extends ConsumerStatefulWidget {  
     const EmailLoginScreen({super.key});  
  
     @override  
     ConsumerState<EmailLoginScreen> createState() => _EmailLoginScreenState();  
   }  
  
   class _EmailLoginScreenState extends ConsumerState<EmailLoginScreen> {  
     final _emailController = TextEditingController(); // 🔄 Replaces _phoneController  
     final _formKey = GlobalKey<FormState>();  
  
     @override  
     void dispose() {  
       _emailController.dispose();  
       super.dispose();  
     }  
  
     void _handleSendOtp() async {  
       if (!_formKey.currentState!.validate()) return;  
  
       final email = _emailController.text.trim().toLowerCase();  
       final notifier = ref.read(authNotifierProvider.notifier);  
  
       final success = await notifier.sendOtp(email);  
       if (success && mounted) {  
         Navigator.of(context).push(  
           MaterialPageRoute(  
             // 🔄 Passes email to OTP screen  
             builder: (_) => OtpVerificationScreen(email: email),  
           ),  
         );  
       }  
     }  
  
 #### 2. Input Field & UX Modifications  
  
 Replace the Indian flag (🇮🇳) and +91 country code container with an email-focused prefix icon, and adjust the keyboard type:  
  
   TextFormField(  
     controller: _emailController,  
     keyboardType: TextInputType.emailAddress, // 👈 Shows @ and .com on keyboard  
     autocorrect: false,  
     enableSuggestions: true,  
     style: const TextStyle(  
       fontSize: 16,  
       fontWeight: FontWeight.w600,  
       color: ArogyaColors.neutralDark,  
     ),  
     decoration: InputDecoration(  
       hintText: strings.emailHint, // "e.g. rahul.sharma@example.com"  
       prefixIcon: const Icon(  
         Icons.mail_outline_rounded,  
         color: ArogyaColors.primaryTeal,  
         size: 22,  
       ),  
     ),  
     validator: (value) {  
       if (value == null || value.trim().isEmpty) {  
         return strings.emailEmptyError; // "Please enter your email address"  
       }  
       final emailRegex = RegExp(r'^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$');  
       if (!emailRegex.hasMatch(value.trim())) {  
         return strings.emailInvalidError; // "Please enter a valid email address"  
       }  
       return null;  
     },  
   ),  
  
 #### 3. Updating Localized Strings (app_strings.dart)  
  
 Add localized keys for the new email labels across all supported languages:  
  
   // English:  
   emailLabel: 'Email Address',  
   emailHint: 'name@example.com',  
   emailEmptyError: 'Please enter your email address',  
   emailInvalidError: 'Please enter a valid email address',  
   emailOtpSubtext: 'We will send a 6-digit verification code to your email',  
  
   // Hindi:  
   emailLabel: 'ईमेल पता',  
   emailHint: 'name@example.com',  
   emailEmptyError: 'कृपया अपना ईमेल पता दर्ज करें',  
   emailInvalidError: 'कृपया एक मान्य ईमेल पता दर्ज करें',  
   emailOtpSubtext: 'हम आपके ईमेल पर 6 अंकों का सत्यापन कोड भेजेंगे',  
  
   // Gujarati:  
   emailLabel: 'ઇમેઇલ સરનામું',  
   emailHint: 'name@example.com',  
   emailEmptyError: 'કૃપા કરીને તમારું ઇમેઇલ સરનામું દાખલ કરો',  
   emailInvalidError: 'કૃપા કરીને માન્ય ઇમેઇલ સરનામું દાખલ કરો',  
   emailOtpSubtext: 'અમે તમારા ઇમેઇલ પર 6-અંકનો ચકાસણી કોડ મોકલીશું',  
 ──────  
 ### Step 2: Update OtpVerificationScreen (otp_verification_screen.dart)  
  
 #### Detailed Code Changes:  
  
 1. Change Constructor Parameter:  
   class OtpVerificationScreen extends ConsumerStatefulWidget {  
     final String email; // 🔄 Replaces phoneNumber  
     const OtpVerificationScreen({super.key, required this.email});  
     ...  
  
 2. Display Destination Email with "Change" Action:  
   Row(  
     children: [  
       Expanded(  
         child: Text(  
           '${strings.codeSentTo} ${widget.email}',  
           style: theme.textTheme.bodyMedium?.copyWith(  
             color: ArogyaColors.neutralMuted,  
           ),  
           overflow: TextOverflow.ellipsis,  
         ),  
       ),  
       const SizedBox(width: 8),  
       InkWell(  
         onTap: () {  
           ref.read(authRepositoryProvider).clearSession();  
           Navigator.of(context).pop();  
         },  
         child: Text(  
           strings.changePhoneOrEmailBtn, // "Change"  
           style: const TextStyle(  
             color: ArogyaColors.primaryTeal,  
             fontWeight: FontWeight.w700,  
             decoration: TextDecoration.underline,  
           ),  
         ),  
       ),  
     ],  
   )  
  
 3. Verify Handler:  
 Submits the 6-digit numeric token to the notifier:  
   Future<void> _handleVerify(String otp) async {  
     if (otp.length != 6) return;  
     final notifier = ref.read(authNotifierProvider.notifier);  
     final success = await notifier.verifyOtp(otp);  
  
     if (success && mounted) {  
       // Direct to Role Selection Screen if user profile is new  
       Navigator.of(context).pushReplacement(  
         MaterialPageRoute(builder: (_) => const RoleSelectionScreen()),  
       );  
     }  
   }  
  
 ──────  
 ### Step 3: Update RoleSelectionScreen (role_selection_screen.dart)  
  
 When a first-time user arrives at the Role Selection screen to select between Patient and Doctor, they should see their verified email badge:  
  
   if (authState.user?.email != null && authState.user!.email.isNotEmpty) ...[  
     const SizedBox(height: 8),  
     Container(  
       padding: const EdgeInsets.symmetric(horizontal: 10, vertical: 4),  
       decoration: BoxDecoration(  
         color: ArogyaColors.primaryLight,  
         borderRadius: ArogyaDimensions.roundedPill,  
       ),  
       child: Row(  
         mainAxisSize: MainAxisSize.min,  
         children: [  
           const Icon(Icons.mark_email_read_outlined, size: 14, color: ArogyaColors.primaryTeal),  
           const SizedBox(width: 6),  
           Text(  
             authState.user!.email,  
             style: const TextStyle(  
               color: ArogyaColors.primaryTeal,  
               fontWeight: FontWeight.w700,  
               fontSize: 12,  
             ),  
           ),  
         ],  
       ),  
     ),  
   ],  
 ──────  
 ### Step 4: Update Navigation Entry Points Across the App  
  
 Whenever the user logs out or the session expires, the app redirects to the primary authentication screen. Update imports and routes across the 4 key files:  
  
 4. **main.dart**:  
   // In root AuthBootstrapGate / StreamBuilder:  
   return bootstrapState.when(  
     data: (state) => switch (state) {  
       BootstrapUnauthenticated() => const EmailLoginScreen(), // 👈 Replaces PhoneLoginScreen  
       BootstrapIncompleteRegistration() => const RoleSelectionScreen(),  
       BootstrapAuthenticated(:final user) => user.role == UserRole.doctor  
           ? const DoctorDashboardScreen()  
           : const PatientHomeDashboard(),  
       BootstrapError(:final message) => AuthErrorScreen(message: message),  
     },  
     loading: () => const AppSplashScreen(),  
     error: (err, stack) => AuthErrorScreen(message: err.toString()),  
   );  
  
 5. **patient_home_dashboard.dart**:  
     • In the Drawer or Settings "Logout" action:  
       await ref.read(authNotifierProvider.notifier).signOut();  
       if (context.mounted) {  
         Navigator.of(context).pushAndRemoveUntil(  
           MaterialPageRoute(builder: (_) => const EmailLoginScreen()),  
           (route) => false,  
         );  
       }  
  
 6. **doctor_dashboard_screen.dart**:  
     • Same logout redirect replaced with EmailLoginScreen().  
 7. **ui_showcase_screen.dart**:  
     • Update showcase navigation tile:  
       ListTile(  
         title: const Text('Email OTP Authentication Screen'),  
         subtitle: const Text('Native Supabase email OTP flow'),  
         trailing: const Icon(Icons.arrow_forward_ios, size: 16),  
         onTap: () => Navigator.of(context).push(  
           MaterialPageRoute(builder: (_) => const EmailLoginScreen()),  
         ),  
       ),  
  
  
 ──────  
 ### UI Migration Comparison  
  
   BEFORE (Phone OTP)                      AFTER (Email OTP)  
   ┌─────────────────────────────────┐    ┌─────────────────────────────────┐  
   │        त्रिविधा हेल्थकेयर        │    │        त्रिविधा हेल्थकेयर        │  
             │                                 │    │                                 │  
             │ Mobile Number                   │    │ Email Address                   │  
             │ [ 🇮🇳 +91 ] [ 98765 43210      ] │    │ [ ✉ ] [ name@example.com      ] │  
               │                                 │    │                                 │  
               │ [        Get OTP Button       ] │    │ [        Get OTP Button       ] │  
               │                                 │    │                                 │  
               │ 🔒 DPDPA Compliant Secure SMS   │    │ 🔒 DPDPA Compliant Secure Email │  
               └─────────────────────────────────┘    └─────────────────────────────────┘  
  
 By completing these 4 steps, your UI presentation remains cohesive, fully multilingual, and naturally aligned with the new email OTP backend.
















---

### Phase 6: Testing & Quality Assurance

1. **Unit Tests (`test/auth_test.dart`)**:
   - Update `sendOtp` and `verifyOtp` test cases with test emails (`test@example.com`).
   - Validate state transitions (`initial` → `codeSent` → `authenticated`).
   - Test email validation rules.
2. **Widget Tests**:
   - Verify `EmailLoginScreen` renders localized labels and handles empty/invalid email input.
   - Verify `OtpVerificationScreen` displays the recipient email.
3. **Integration & E2E Testing**:
   - Request OTP to a live test email address.
   - Verify OTP code delivery from Supabase.
   - Confirm profile creation in `public.users` with `email` column.










### Deep Dive: Phase 6 — Testing & Quality Assurance
  Testing ensures that switching from Phone OTP to Email OTP creates zero regressions across the codebase. In Flutter and Riverpod, this is validated across three testing layers: Unit Tests (business logic and state machines), Widget
  Tests (UI interactions and localization), and Integration/E2E Tests (live Supabase communication).
  ──────
  ### Step 1: Unit Tests (auth_test.dart)

  Unit tests verify the business logic in AuthNotifier and AuthRepository in complete isolation without running a real Flutter UI engine.
  #### 1. Testing Initial State & Successful Email Dispatch:
    test('sendOtp with email transitions state to AuthStatus.codeSent', () async {
      final repository = AuthRepository(); // Mock in-memory repo
      final notifier = AuthNotifier(repository);

      expect(notifier.state.status, AuthStatus.initial);

      // Send OTP to test email
      final success = await notifier.sendOtp('patient.rahul@example.com');

      expect(success, isTrue);
      expect(notifier.state.status, AuthStatus.codeSent);
      expect(notifier.state.email, 'patient.rahul@example.com');
      expect(notifier.state.verificationId, isNotNull);
    });

  #### 2. Testing Email Validation Guardrails:

    test('sendOtp rejects malformed email strings immediately', () async {
      final repository = AuthRepository();
      final notifier = AuthNotifier(repository);

      // Missing @ and domain
      final resultBad1 = await notifier.sendOtp('notanemail');
      expect(resultBad1, isFalse);
      expect(notifier.state.status, AuthStatus.error);
      expect(notifier.state.errorMessage, contains('valid email'));

      // Missing domain extension
      final resultBad2 = await notifier.sendOtp('user@domain');
      expect(resultBad2, isFalse);
      expect(notifier.state.status, AuthStatus.error);
    });

  #### 3. Testing OTP Verification & Authentication:

    test('verifyOtp transitions state to AuthStatus.authenticated with UserModel', () async {
      final repository = AuthRepository();
      final notifier = AuthNotifier(repository);

      await notifier.sendOtp('doctor.suresh@example.com');

      // Verify using standard mock OTP code
      final success = await notifier.verifyOtp('123456');

      expect(success, isTrue);
      expect(notifier.state.status, AuthStatus.authenticated);
      expect(notifier.state.user, isNotNull);
      expect(notifier.state.user?.email, 'doctor.suresh@example.com');
    });

    test('verifyOtp fails with invalid OTP code', () async {
      final repository = AuthRepository();
      final notifier = AuthNotifier(repository);

      await notifier.sendOtp('patient@example.com');
      final success = await notifier.verifyOtp('000000'); // Bad code

      expect(success, isFalse);
      expect(notifier.state.status, AuthStatus.error);
      expect(notifier.state.errorMessage, isNotEmpty);
    });

  #### 4. Concurrency Guard Tests (Double-Tap Protection):

    test('Rapid double-tap on sendOtp rejects redundant concurrent execution', () async {
      final repository = AuthRepository();
      final notifier = AuthNotifier(repository);

      // User taps "Send OTP" twice in rapid succession (<10ms)
      final future1 = notifier.sendOtp('test@example.com');
      final future2 = notifier.sendOtp('test@example.com');

      final results = await Future.wait([future1, future2]);
      expect(results[0], isTrue);  // First request initiated
      expect(results[1], isFalse); // Second request blocked by authenticating guard
    });
  ──────
  ### Step 2: Widget & UI Tests

  Widget tests pump actual Flutter UI widgets into a headless test environment, simulating user taps, keyboard entries, and validating multi-language string rendering.

  #### 1. Testing EmailLoginScreen Form Validation & Localization:

    testWidgets('EmailLoginScreen validates empty and invalid email in Hindi and English', (WidgetTester tester) async {
      await tester.pumpWidget(
        ProviderScope(
          child: MaterialApp(
            theme: ArogyaTheme.lightTheme,
            home: const EmailLoginScreen(),
          ),
        ),
      );

      // Default Language is Hindi: check localized header and button
      expect(find.text('त्रिविधा हेल्थकेयर'), findsOneWidget);
      expect(find.text('ओटीपी प्राप्त करें'), findsOneWidget);

      // 1. Tap "Get OTP" without entering any email
      await tester.tap(find.text('ओटीपी प्राप्त करें'));
      await tester.pump();
      expect(find.text('कृपया अपना ईमेल पता दर्ज करें'), findsOneWidget);

      // 2. Enter invalid email format
      await tester.enterText(find.byType(TextFormField), 'bad-email-format');
      await tester.tap(find.text('ओटीपी प्राप्त करें'));
      await tester.pump();
      expect(find.text('कृपया एक मान्य ईमेल पता दर्ज करें'), findsOneWidget);

      // 3. Switch language to English via dropdown
      await tester.tap(find.byType(DropdownButton<AppLanguage>));
      await tester.pumpAndSettle();
      await tester.tap(find.text('English').last);
      await tester.pumpAndSettle();

      // Verify English validation message
      await tester.tap(find.text('Get OTP'));
      await tester.pump();
      expect(find.text('Please enter a valid email address'), findsOneWidget);
    });

  #### 2. Testing OtpVerificationScreen Email Display:

    testWidgets('OtpVerificationScreen displays target email and accepts 6-digit code', (WidgetTester tester) async {
      const testEmail = 'patient.anita@example.com';

      await tester.pumpWidget(
        ProviderScope(
          overrides: [
            authNotifierProvider.overrideWith((ref) {
              final notifier = AuthNotifier(AuthRepository());
              notifier.state = AuthState.codeSent(
                email: testEmail,
                verificationId: 'ver_test_123',
              );
              return notifier;
            }),
          ],
          child: MaterialApp(
            theme: ArogyaTheme.lightTheme,
            home: const OtpVerificationScreen(email: testEmail),
          ),
        ),
      );

      // Verify target email is visible to the user
      expect(find.textContaining(testEmail), findsOneWidget);

      // Enter 6-digit OTP code into pin input
      await tester.enterText(find.byType(TextField).first, '123456');
      await tester.pumpAndSettle();
    });
  ──────
  ### Step 3: Integration & End-to-End (E2E) Testing

  Integration tests connect to your live Supabase project instance (using credentials passed via --dart-define or test configuration).

     ┌────────────────────────────┐   ┌──────────────────────┐  ┌────────────────────┐┌─────────────────────────┐┌────────┐  ┌─────┐   ┌─────┐
     │ Flutter App (Supabase SDK) │   │ Supabase Auth Engine │  │ User's Email Inbox ││ PostgreSQL public.users ││ Tester │  │ SB- │   │ DB- │
     └────────────────────────────┘   └──────────────────────┘  └────────────────────┘└─────────────────────────┘└────────┘  └─────┘   └─────┘
                    │                             │                        │                       │                  │         │         │
                    ◄───────────────────────────Enters test email (e.g. dev@trividha.com)─────────────────────────────│         │         │
                    │                             │                        │                       │                  │         │         │
                    │signInWithOtp(email, shouldCr►ateUser: true)          │                       │                  │         │         │
                    │                             │                        │                       │                  │         │         │
                    │                             │Dispatches HTML Email wi►h 6-digit Token        │                  │         │         │
                    │                             │                        │                       │                  │         │         │
                    │                             │                        ◄Inspects inbox and copies {{ .Token }} (e.g. 782194)│         │
                    │                             │                        │                       │                  │         │         │
                    ◄─────────────────────────────Inputs 782194 on OtpVerificationScreen──────────────────────────────│         │         │
                    │                             │                        │                       │                  │         │         │
                    │verifyOTP(email, token: "7821►4", type: OtpType.email)│                       │                  │         │         │
                    │                             │                        │                       │                  │         │         │
                    ◄─────────────────────────────────Returns verified AuthSession (JWT + UID)──────────────────────────────────│         │
                    │                             │                        │                       │                  │         │         │
                    │──────────────────Query public.users where id = auth.uid()────────────────────►                  │         │         │
   ┤ alt Profile Missing (New User) ├
                    │                             │                        │                       │                  │         │         │
                    │───────────────────────────────────Shows RoleSelectionScreen─────────────────────────────────────►         │         │
                    │                             │                        │                       │                  │         │         │
                    ◄───────────────────────────────Selects Doctor + Enters Full Name─────────────────────────────────│         │         │
                    │                             │                        │                       │                  │         │         │
                    │────────────Upsert into public.users (id, email, full_name, role)─────────────►                  │         │         │
                    │                             │                        │                       │                  │         │         │
                    ◄───────────────────────────────────────────────────────200 OK────────────────────────────────────────────────────────│
                    │                             │                        │                       │                  │         │         │
                    │──────────────────────────────Navigates straight to Home Dashboard───────────────────────────────►         │         │
                    │                             │                        │                       │                  │         │         │

  #### E2E Verification Checklist:

  1. Inbox Delivery:
      • Verify the email arrives in under 5–10 seconds.
      • Confirm proper HTML rendering, Trividha logo/colors, and legible 6-digit token.
  2. Expired OTP Handling:
      • Wait 10 minutes (past the 600s TTL) or trigger an expired token test.
      • Verify the app displays: "Invalid or expired OTP code".
  3. Database Assertion:
      • In Supabase Table Editor → public.users:
      • Inspect the new row:
          • id: Valid UUID matching auth.users.id.
          • email: Contains lowercase verified email (e.g. dev@trividha.com).
          • role: Correctly populated as patient or doctor.
          • created_at: Valid ISO timestamp.

  4. Automated Test Suite Run:
      • Execute:
        flutter test

      • Ensure all tests pass with zero failures and zero analyzer warnings.

