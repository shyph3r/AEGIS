AEGIS-WEB-002 — Customer Can Enumerate Users Through a Restricted API
Finding summary: A normal customer account cannot open the Administration page, but its bearer token can be used to call GET /api/Users and retrieve multiple user records.

Finding details
Field	Value
Finding ID	AEGIS-WEB-002
Severity	Medium (provisional; depends on exposed fields and intended policy)
Status	Customer data access reproduced; role-policy/baseline documentation incomplete
Affected endpoint	GET /api/Users
Frontend route	/#/administration
Target	http://192.168.219.20:3000 — private Ubuntu/VMware lab
Authentication needed	Valid customer session (as tested)
Category	Broken Access Control / Vertical Authorization Bypass
CWE	CWE-863 — Incorrect Authorization (CWE-862 if no relevant check exists)
OWASP Top 10 (2021)	A01 — Broken Access Control
Tools	Burp Suite Repeater, browser


1. Executive summary
During testing, the Juice Shop frontend prevented an ordinary customer from opening /#/administration. However, a direct request to GET /api/Users with the customer's own bearer token returned HTTP 200 OK and a list of multiple registered users.
This is evidence of an inconsistent access restriction between the Administration interface and its supporting API. Whether the API violates a formal administrator-only policy, and how much sensitive data it exposes, should be recorded before assigning a final severity.
2. Scope and preconditions
- The tester used a normal account registered in the authorized Juice Shop lab.
- The test used that customer's bearer token, not the admin session obtained in a separate SQL injection test.
- Only read access to the account listing was tested; no changes to other user accounts were attempted.
- Tests without authentication and with a confirmed administrator identity still need to be documented as controls.
3. Steps to reproduce
Step 1 — Establish customer context
1. Log out of the administrator session.
2. Log in using a normal customer account.
3. Confirm the account identity and keep its token private.
4. Navigate to http://192.168.219.20:3000/#/administration.
5. Record that the normal customer cannot view the Administration interface.
Step 2 — Call the backend endpoint directly
1. In Burp Suite → Proxy → HTTP history, identify a request from the customer session.
2. Send it to Repeater.
3. Change method/path as shown, preserving the customer token:
GET /api/Users HTTP/1.1
Host: 192.168.219.20:3000
Authorization: Bearer <REDACTED_CUSTOMER_TOKEN>
Accept: application/json
4. Send the request and inspect status and response JSON.
Observed: HTTP 200 OK with records belonging to multiple users.
Step 3 — Add control cases
Repeat the same request under two different identities:
Caller	Action	Result to record
Anonymous	Remove bearer token and session cookies	[PENDING: status and response]
Customer	Send customer's token	200 OK; multiple user records
Administrator	Send valid administrator token	[PENDING: status and response]


These controls are required to distinguish authentication checks from role-based authorization checks.
4. Evidence
ID	Evidence / observation	Verification
E01	Customer login / customer identity	Reported by tester
E02	Customer blocked from /#/administration	Reported during Day 10
E03	GET /api/Users request with customer token	Reproduced in Repeater
E04	200 OK and multiple user records	Confirmed by tester
E05	Anonymous control response	Pending attachment
E06	Admin control response	Pending attachment


Suggested screenshots:
evidence/finding-02/
├── customer-route-denied.png
├── customer-users-200.png
├── anonymous-control.png
└── admin-control.png
<!-- Add images after saving them:
![Customer user enumeration](evidence/finding-02/customer-users-200.png)
-->

Document from the actual response: [PENDING: exact exposed JSON field names]. Redact user emails, JWTs, passwords, session cookies, and other personal information in public evidence.
5. Technical analysis and root cause
Frontend restrictions operate within the browser and cannot substitute for server-side authorization. The observed API behavior suggests that the backend may allow an authenticated customer to retrieve a listing intended for administrators.
Root cause is provisional: The test proves the API response from the customer's account. It does not by itself prove which backend middleware is missing or that every administrative API is vulnerable.
Expected vs. observed
Expected secure behavior (if admin-only)	Observed behavior
Customer is blocked from Administration UI.	Customer was blocked from the route.
Customer is denied access to admin-only user listings.	Customer received 200 OK plus multiple user records.
Access control is enforced by the API.	Role-based restriction was not evident from the response.


6. Security impact
- Confidentiality: Customer account holders can enumerate other user records.
- Account discovery: Exposed identifiers may support targeted abuse; no follow-on testing was performed.
- Scope limitation: No anonymous access, account modification, or broader administrative privilege escalation has been demonstrated here.
Severity rationale: Medium is preliminary; the actual risk depends on authorization requirements, the fields disclosed, and reachable users.
7. Recommended remediation
1. Enforce server-side authorization at GET /api/Users, based on intended role or permission.
2. Apply deny-by-default access policies; frontend route guards are not an authorization boundary for APIs.
3. Return only necessary fields to authorized callers.
4. Add automated tests for anonymous, customer, and administrator roles.
5. Review adjacent administrative endpoints for the same policy gap.
Illustrative design (not copied from Juice Shop source):
app.get('/api/Users', authenticate, requireRole('admin'), async (req, res) => {
  const users = await getMinimalAuthorizedUserList();
  return res.json(users);
});
8. Retesting / acceptance criteria
Caller	Expected result if admin-only policy applies
Anonymous	Denied (typically 401); no user list
Customer	Denied (typically 403); no user list
Administrator	Allowed (200) with only required fields


Additional checks: ensure logout invalidates the previous test context, verify no sensitive response body is returned on denial, and confirm normal administrative functions still operate.
Retest status: Not performed; remediation has not been applied.
References
- OWASP Web Security Testing Guide — Bypassing Authorization Schema
- OWASP Authorization Cheat Sheet
- MITRE CWE-863