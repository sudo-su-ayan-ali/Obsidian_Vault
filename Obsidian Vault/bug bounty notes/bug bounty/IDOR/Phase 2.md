## **Phase 2: Permission Testing - Complete Walkthrough**

You already have your endpoint list from Phase 1. Now you'll test each one systematically.

### **Before You Start**

Make sure you have:

- ✓ **2 test accounts created** (Account A and Account B with different IDs)
- ✓ **Burp Suite open** with Proxy listening on `127.0.0.1:8080`
- ✓ **Your endpoint list** (endpoints with IDs)
- ✓ **Burp Repeater** open (we'll use this for testing)

---

## **Test 1: Owner Access (Baseline)**

**Goal:** Confirm you CAN access your own data. This is your baseline.

### **Setup:**

Log in as **User A** (your first account)

### **Test:**

For each endpoint in your list, access it with your own ID:

```
Logged in as: hunter1@gmail.com (ID: 111)

Test these requests:
GET /api/profile/111              → Should return: Your profile data ✓
GET /api/orders/456               → Should return: Your orders ✓
GET /api/invoices/789             → Should return: Your invoices ✓
```

### **What To Do In Burp:**

1. Open **Proxy → HTTP History**
2. Find the request: `GET /api/profile/111`
3. Right-click → **Send to Repeater**
4. Look at the **Response** tab

**You should see:**

```
HTTP/1.1 200 OK

{
  "id": 111,
  "name": "hunter1",
  "email": "hunter1@gmail.com",
  "phone": "9876543210"
}
```

**Document this:**

|Endpoint|User A ID|Status|Result|
|---|---|---|---|
|`/api/profile`|111|200 OK|✓ Own data accessible|
|`/api/orders`|111|200 OK|✓ Own data accessible|
|`/api/invoices`|111|200 OK|✓ Own data accessible|

If any return **403/401/error**, that endpoint is broken—skip it.

---

## **Test 2: Different User Access (THE KEY TEST)**

**Goal:** Try to access User B's data while logged in as User A. This is where you find IDOR.

### **Setup:**

Still logged in as **User A** (hunter1, ID: 111)  
But you're going to try accessing **User B's data** (hunter2, ID: 222)

### **Test:**

For each endpoint, change the ID from your own to the other user's:

```
Logged in as: User A (ID: 111)

Original request:
GET /api/profile/111

Modified request (change ID):
GET /api/profile/222              ← Changed from 111 to 222

Same for other endpoints:
GET /api/orders/456               → Change to some other order ID
GET /api/invoices/789             → Change to some other invoice ID
```

### **What To Do In Burp:**

1. In **Repeater tab**, find your request: `GET /api/profile/111`
2. **Change 111 to 222** in the URL bar
3. Click **Send**
4. Look at the **Response tab**

**Possible Outcomes:**

**Outcome A - PROTECTED (Good):**

```
HTTP/1.1 403 Forbidden

{
  "error": "Unauthorized",
  "message": "You don't have permission to access this resource"
}
```

✓ This endpoint is properly protected. No IDOR here.

---

**Outcome B - VULNERABLE (IDOR Found!):**

```
HTTP/1.1 200 OK

{
  "id": 222,
  "name": "hunter2",
  "email": "hunter2@gmail.com",
  "phone": "9876543211"
}
```

🔴 **You just found an IDOR!** You saw User B's data without authorization.

---

### **Document Your Findings:**

|Endpoint|User A ID|Attempted Access|Status Code|Response|Vulnerable?|
|---|---|---|---|---|---|
|`/api/profile`|111|222|200|Returns User B data|**YES - IDOR**|
|`/api/orders`|111|456|403|Access denied|NO|
|`/api/invoices`|111|789|200|Returns invoice|**YES - IDOR**|

---

## **Test 3: No Authentication (Unauthenticated Access)**

**Goal:** Try to access resources without logging in at all. This tests broken authentication.

### **Setup:**

**Log out completely** from the application.

### **Test:**

Try to access the same endpoints without being logged in:

```
NOT logged in

GET /api/profile/111              → What happens?
GET /api/profile/222              → What happens?
GET /api/orders/456               → What happens?
GET /api/invoices/789             → What happens?
```

### **What To Do In Burp:**

1. In **Repeater**, look at your request
2. Find the **Authorization/Cookie header**:
    
    ```
    Cookie: session=abc123def456Authorization: Bearer token123
    ```
    
3. **Delete that line completely**
4. Click **Send**

**Possible Outcomes:**

**Expected - PROTECTED:**

```
HTTP/1.1 401 Unauthorized

{
  "error": "Authentication required",
  "message": "Please log in to access this resource"
}
```

✓ Good, it requires authentication.

---

**Vulnerable - NO AUTHENTICATION REQUIRED:**

```
HTTP/1.1 200 OK

{
  "id": 111,
  "name": "hunter1",
  "email": "hunter1@gmail.com"
}
```

🔴 **CRITICAL VULNERABILITY!** The endpoint is accessible without any authentication.

---

### **Document:**

|Endpoint|Auth Required?|Without Auth Status|Response|Vulnerable?|
|---|---|---|---|---|
|`/api/profile/111`|YES (expected)|401|Error message|NO|
|`/api/invoices/789`|YES (expected)|200|Returns data|**YES - BROKEN AUTH**|

---

## **Test 4: Different User Roles (If Applicable)**

**Goal:** If the app has roles (Admin, User, Moderator), test if a regular user can access admin data.

### **Setup:**

This test only works if:

- You have access to an **admin account**, OR
- You can create accounts with **different roles**

If you only have regular user accounts, **skip this test.**

### **Test:**

**As a regular User A, try to access admin endpoints:**

```
Logged in as: Regular User (ID: 111)

Try accessing:
GET /api/admin/users              → Should be forbidden
GET /api/admin/all-invoices       → Should be forbidden
GET /api/admin/reports            → Should be forbidden
GET /api/admin/users/222/details  → Should be forbidden
```

### **What To Do In Burp:**

1. In **Repeater**, send a request to an admin endpoint
2. Check the response

**Expected:**

```
HTTP/1.1 403 Forbidden

{
  "error": "Admin access required"
}
```

**Vulnerable:**

```
HTTP/1.1 200 OK

{
  "all_users": [...data...],
  "sensitive_admin_data": [...]
}
```

🔴 Regular user accessed admin data.

---

### **Document:**

|Endpoint|User Role|Status|Vulnerable?|
|---|---|---|---|
|`/api/admin/users`|Regular User|403|NO|
|`/api/admin/reports`|Regular User|200|**YES**|

---

## **Complete Testing Template**

Create this spreadsheet and fill it as you test:

```
Target: [app name]
Date: [today]
User A: hunter1@gmail.com (ID: 111)
User B: hunter2@gmail.com (ID: 222)
```

**Endpoint Testing Matrix:**

|Endpoint|Test 1: Own ID|Test 2: Other ID|Test 3: No Auth|Test 4: Admin Access|VERDICT|
|---|---|---|---|---|---|
|`/api/profile`|200 ✓|403 ✓|401 ✓|N/A|SAFE|
|`/api/orders`|200 ✓|**200 🔴**|401 ✓|N/A|**IDOR**|
|`/api/invoices`|200 ✓|403 ✓|**200 🔴**|N/A|**NO AUTH**|
|`/api/admin/data`|200 ✓|403 ✓|401 ✓|**200 🔴**|**PRIV ESC**|

---

## **Real Example: Step-by-Step**

Let's say you're testing `fakeshop.com`

### **Your endpoint list from Phase 1:**

```
/api/user/profile
/api/orders
/api/invoices
/api/account-settings
```

### **Your test accounts:**

```
Account A: testuser1@email.com (ID: 1001)
Account B: testuser2@email.com (ID: 1002)
```

---

### **Test 1: Own Data Access**

**Logged in as Account A (1001):**

```
Burp Repeater Request:
GET /api/user/profile/1001

Response:
200 OK
{
  "id": 1001,
  "name": "Test User 1",
  "email": "testuser1@email.com"
}

✓ Baseline works. Move to Test 2.
```

---

### **Test 2: Other User's Data (Change ID)**

**Still logged in as Account A (1001):**

```
Burp Repeater Request (CHANGED ID):
GET /api/user/profile/1002          ← Changed from 1001 to 1002

Response:
200 OK
{
  "id": 1002,
  "name": "Test User 2",
  "email": "testuser2@email.com"
}

🔴 VULNERABLE! Account A can see Account B's profile.
This is IDOR.
```

---

### **Test 3: No Authentication**

```
Burp Repeater Request (REMOVED AUTH COOKIE):
GET /api/user/profile/1001
[No Cookie header]

Response:
200 OK
{
  "id": 1001,
  "name": "Test User 1",
  "email": "testuser1@email.com"
}

🔴 VULNERABLE! Profile accessible without login.
This is BROKEN AUTHENTICATION.
```

---

### **Test 4: Invoice Endpoint**

```
Logged in as Account A (1001):

Test 2 - Different User:
GET /api/invoices/5002              ← Account B's invoice

Response:
403 Forbidden
{
  "error": "Access denied"
}

✓ PROTECTED. No IDOR here.
```

---

### **Your Final Spreadsheet:**

|Endpoint|Test 1|Test 2|Test 3|Result|
|---|---|---|---|---|
|`/api/user/profile`|200 ✓|**200 🔴**|**200 🔴**|**IDOR + BROKEN AUTH**|
|`/api/orders`|200 ✓|403 ✓|401 ✓|SAFE|
|`/api/invoices`|200 ✓|403 ✓|401 ✓|SAFE|
|`/api/account-settings`|200 ✓|**200 🔴**|401 ✓|**IDOR**|

---

## **What To Do After Testing**

Once you've completed all 4 tests for each endpoint:

1. **Document vulnerabilities** (which endpoints are vulnerable)
2. **Test with different ID values** (not just one other user):
    
    ```
    User A (1001) tries:/api/user/profile/1002 → Works/api/user/profile/1003 → Works/api/user/profile/1004 → Works→ Pattern confirmed, all IDs accessible
    ```
    
3. **Prepare your bug report** (which we'll cover next)

---

## **Your Task Right Now**

Pick **one target** and:

1. **Create 2 accounts** (5 min)
2. **Run Test 1** (own data access) - 5 min
3. **Run Test 2** (different user) - 10 min
4. **Run Test 3** (no auth) - 5 min
5. **Fill in your spreadsheet**
