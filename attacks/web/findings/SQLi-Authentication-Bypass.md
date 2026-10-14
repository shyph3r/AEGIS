AEGIS-WEB-001 — SQL Injection in Login Allows Authentication Bypass
Finding summary: An unauthenticated user can manipulate the email field in the Juice Shop login request to bypass password verification and obtain an administrative session in the authorized lab.

Finding details
Field	Value
Finding ID	AEGIS-WEB-001
Severity	High (provisional; CVSS not calculated)
Status	Confirmed in lab; remediation/retest pending
Affected endpoint	POST /rest/user/login
Target	http://192.168.219.20:3000 — private Ubuntu/VMware lab
Authentication needed	None
Category	SQL Injection / Authentication Bypass
CWE	CWE-89 — SQL Injection; additional CWE-209 — Information Exposure Through an Error Message
OWASP Top 10 (2021)	A03 — Injection
Tools	Burp Suite Repeater, browser


1. Executive summary
The Juice Shop login endpoint handles user-controlled email input in an unsafe SQL context. A single-quote test produced a SQLite syntax error containing query details and a stack trace. A subsequent injection using a true boolean condition and SQL comment syntax bypassed the password requirement, resulting in an administrative session.
The confirmed impact in this exercise is authentication bypass. No database export, file access, or remote code execution was demonstrated.
2. Scope and preconditions
- Testing took place against an intentionally vulnerable Juice Shop container hosted on the tester's Ubuntu VM.
- Kali Linux and Burp Suite were used to send HTTP requests.
- No valid account credentials were needed for the injection attempt.
- These observations apply to the local training instance, not a production deployment.
3. Steps to reproduce
Step 1 — Capture the normal login request
1. Open http://192.168.219.20:3000/#/login.
2. Submit an invalid email/password pair.
3. In Burp Suite → Proxy → HTTP history, locate POST /rest/user/login.
4. Select Send to Repeater and record the unsuccessful baseline response.
Step 2 — Confirm SQL input handling
Set the email to a single quote and send the request:
POST /rest/user/login HTTP/1.1
Host: 192.168.219.20:3000
Content-Type: application/json

{"email":"'","password":"Abc123"}
Observed: The server returned SQLITE_ERROR: unrecognized token and exposed a SQL query excerpt and stack trace.
Step 3 — Test authentication bypass
In the same Repeater request, submit:
POST /rest/user/login HTTP/1.1
Host: 192.168.219.20:3000
Content-Type: application/json

{"email":"' OR 1=1 -- ","password":"Abc123"}
Observed: The tester reported a successful login as the administrator without supplying that account's valid password. Preserve the relevant redacted response or identity confirmation as evidence.
Interpretation: The OR 1=1 condition can make the predicate true, and -- can comment out trailing SQL when the generated query is on the appropriate line. The payload's effect depends on the application's exact query construction.

4. Evidence
ID	Evidence / observation	Verification
E01	Login request with email set to '	Observed in Burp screenshot
E02	SQLite error, query excerpt, and stack trace	Observed in Burp screenshot
E03	Injection request with ' OR 1=1 --	Performed in Day 10 lab
E04	Administrator session established without valid password	Confirmed by tester; attach redacted screenshot


Suggested screenshots (save alongside this report):
evidence/finding-01/
├── sqlite-error-request-response.png
├── bypass-request-response.png
└── admin-session-confirmation.png
<!-- When the screenshot exists, uncomment and adjust the relevant line:
![Burp SQLite error](evidence/finding-01/sqlite-error-request-response.png)
-->

Still to record from Burp: exact HTTP status and sanitized response body for the successful bypass. Do not publish bearer tokens, cookies, or user secrets.
5. Technical analysis and root cause
The malformed-input error indicates SQL syntax was influenced by the email value. The successful bypass indicates that user input affected the authentication query's logic. This is consistent with a raw SQL query that incorporates untrusted input into SQL syntax rather than using bound parameters.
The error response also exposed backend details, representing a separate information-disclosure weakness. The black-box evidence supports the vulnerability without needing repository access to the backend implementation.
Expected vs. observed
Expected secure behavior	Observed behavior
Incorrect credentials are rejected regardless of input syntax.	The injection bypassed credential verification.
User input is bound as data, not interpreted as SQL.	A single quote triggered SQL parsing error.
Internal errors are not exposed to clients.	SQLite query details and stack trace were returned.


6. Security impact
- Confidentiality: An attacker may access information available to the impersonated account.
- Integrity: An attacker may gain access to privileged operations if authorized by the obtained session; these operations were not tested here.
- Authentication: Login can be bypassed without the correct password.
- Information disclosure: Detailed SQL errors reveal implementation information.
Severity rationale: High is appropriate as a preliminary rating because an unauthenticated request produced an administrative session. Final risk and CVSS depend on deployment context and exposure.
7. Recommended remediation
1. Replace string-built SQL with parameterized queries / prepared statements, including when using ORM raw-query APIs.
2. Keep password hashing and credential verification independent of SQL syntax construction.
3. Return generic client-facing authentication errors; send detailed diagnostics only to protected server logs.
4. Add regression tests for quote characters, boolean expressions, SQL comments, and invalid credential combinations.
5. Review neighboring endpoints for similar raw-SQL construction.
Illustrative parameter-binding pattern (not extracted from target source):
const users = await db.query(
  'SELECT * FROM Users WHERE email = ? AND password = ? AND deletedAt IS NULL',
  { replacements: [email, passwordHash], type: QueryTypes.SELECT }
);
The example demonstrates parameter binding; password verification must still follow the application's approved credential-storage design.
8. Retesting / acceptance criteria
Retest	Pass condition
Invalid email/password	Authentication rejected
Previously successful SQLi payload	Authentication rejected; no admin session
Input containing '	No SQL diagnostic details returned
Valid account login	Normal login still works
Error handling	Internal SQL and stack traces stay server-side


Retest status: Not performed; remediation has not been applied to the lab instance.
References
- OWASP Web Security Testing Guide — SQL Injection
- OWASP SQL Injection Prevention Cheat Sheet
- MITRE CWE-89