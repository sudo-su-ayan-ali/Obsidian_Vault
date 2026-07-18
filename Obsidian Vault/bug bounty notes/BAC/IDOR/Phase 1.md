## **Phase 1: Application Mapping - Brief Version**

### **Step 1: Create 2 Test Accounts (5 minutes)**

Open the target website and sign up twice:

```
Account 1: hunter1@gmail.com (note down the user ID if visible)
Account 2: hunter2@gmail.com (note down the user ID if visible)
```

Write them down. You'll switch between them.

---

### **Step 2: Open Burp Suite & Set Up Proxy (5 minutes)**

1. Open **Burp Suite Community** (free version)
2. Go to **Proxy tab** → Click the checkbox: **"Intercept is off"** (important!)
3. Configure your browser to use Burp as proxy:
    - **Firefox:** Settings → Network → Manual proxy → `127.0.0.1:8080`
    - **Chrome:** Use FoxyProxy extension → Add proxy `127.0.0.1:8080`

**Done.** Now every request will be captured.

---

### **Step 3: Use the App Normally (10-15 minutes)**

**Logged in as Account 1 (hunter1), do this:**

1. View your **profile/account page**
2. View your **settings**
3. View any **orders/invoices/documents** (if app has them)
4. Click **any feature that shows "your" data**

**Why?** Burp is recording every request in the background.

---

### **Step 4: Open Burp's HTTP History (5 minutes)**

Go to **Proxy → HTTP History**

You'll see a list of all requests. Look for URLs with **numbers/IDs in them:**

```
GET /api/user/123
GET /profile/456
GET /invoices/789
POST /orders
GET /settings/123
```

**Copy these endpoints into a spreadsheet** (or even a notepad):

```
/api/user/123
/profile/456
/invoices/789
/settings/123
```

---

### **Step 5: Create a Simple Spreadsheet (5 minutes)**

Open **Google Sheets** or **Excel** and create:

|Endpoint|ID Type|What It Shows|Sensitive?|
|---|---|---|---|
|`/api/user/123`|Numeric|Name, email, phone|YES|
|`/profile/456`|Numeric|Bio, avatar|YES|
|`/invoices/789`|Numeric|Bill details, amount|YES|
|`/settings/123`|Numeric|Account preferences|YES|

**For the "Sensitive?" column:** Ask yourself: _Should only the owner see this?_

- Profile data? **YES**
- Orders? **YES**
- Invoices? **YES**
- Public posts? **NO**

---

### **Step 6: Test Each Endpoint (5-10 minutes)**

For each endpoint in your spreadsheet, test this:

**As Account 1 (hunter1), access your own data:**

```
GET /api/user/123        → Works? ✓ (you should see your data)
GET /invoices/789        → Works? ✓
```

If they work, move to next step. If they don't, that endpoint is broken—skip it.

---

### **Step 7: Test With Different User ID (THE BUG TEST) (5-10 minutes)**

**Still logged in as Account 1, change the ID to Account 2's ID:**

Example: If your Account 1 ID is 123, and Account 2 ID is 456:

```
Original request:
GET /api/user/123        → Returns your profile ✓

Modified request:
GET /api/user/456        → What happens?
```

**In Burp Repeater:**

1. Find the request in HTTP History
2. Right-click → **Send to Repeater**
3. Change `123` to `456`
4. Click **Send**

**Check the response:**

- **If 403/401/error:** ✓ It's protected (good)
- **If 200 OK + returns Account 2 data:** 🔴 **IDOR FOUND!**

---

### **Your Checklist for Today**

- [x] Create 2 test accounts
- [x] Set up Burp proxy
- [x] Use the app for 15 minutes (logged in as Account 1)
- [ ] Go to **Proxy → HTTP History** and list endpoints with IDs
- [ ] Create spreadsheet with 5-10 endpoints
- [ ] Test your own data (account 1 ID) — confirm it works
- [ ] Test other user's data (change ID to account 2) — **check if IDOR exists**

---

## **Example: Real Target**

Let's say you're testing `dummyapp.com`:

**Account 1:** hunter1 (ID: 101)  
**Account 2:** hunter2 (ID: 102)

**Burp captures these requests:**

```
GET /user/profile/101
GET /orders/list/101
GET /invoices/view/789
```

**Your spreadsheet:**

|Endpoint|Sensitive?|
|---|---|
|`/user/profile/101`|YES|
|`/orders/list/101`|YES|
|`/invoices/view/789`|YES|

**Test:**

```
Logged in as hunter1 (101), try:
GET /user/profile/102        → 403 Forbidden (protected ✓)
GET /orders/list/102         → 403 Forbidden (protected ✓)
GET /invoices/view/790       → 200 OK + Returns invoice data (VULNERABLE! 🔴)
```

**Result:** Found IDOR in `/invoices/view/` endpoint.

---

