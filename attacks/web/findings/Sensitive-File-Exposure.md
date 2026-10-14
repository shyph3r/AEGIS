AEGIS-WEB-003 — Public File Route Exposes Confidential Documents
Finding summary: The Juice Shop training application exposes an /ftp/ route for directory browsing and file downloads. The exercise demonstrates the risk of placing confidential documents in a publicly served directory.

Finding details
Field	Value
Finding ID	AEGIS-WEB-003
Severity	Medium (provisional; depends on content exposed)
Status	Challenge/remediation reviewed; attach local confidential-file HTTP response to complete evidence
Affected routes	GET /ftp/, GET /ftp/acquisitions.md (standard Juice Shop challenge path)
Target	http://192.168.219.20:3000 — private Ubuntu/VMware lab
Authentication needed	Intended unauthenticated access test
Category	Sensitive File Exposure / Security Misconfiguration
CWE	CWE-552 — Files or Directories Accessible to External Parties; related CWE-548 — Directory Listing
OWASP Top 10 (2021)	A05 — Security Misconfiguration
Tools	Browser, Burp Suite


1. Executive summary
The web application makes files available from an /ftp/ URL namespace. A legitimate public file link can disclose that the directory exists, and directory browsing or direct file requests may reveal documents that were not intended for public access.
The AEGIS exercise included investigating this location and reviewing Juice Shop's suggested patch that removes its public file-serving routes. The local HTTP response for the confidential file was not included in the supplied evidence, so this report distinguishes the standard exercise behavior from the tester's independently documented HTTP proof.
2. Scope and preconditions
- Target was the tester's intentionally vulnerable Juice Shop instance in Docker.
- Intended validation was an unauthenticated browser/Burp request to a private document.
- The test concerns read access; it does not claim modifying or uploading files.
- Findings here describe the /ftp/ route, not an FTP network protocol service.
3. Discovery and steps to reproduce
Step 1 — Identify the file-serving directory
1. Browse Juice Shop normally and inspect a public document link such as /ftp/legal.md.
2. Record the URL, response headers, and whether authentication was required.
3. Request the parent directory:
GET /ftp/ HTTP/1.1
Host: 192.168.219.20:3000
4. Record whether a directory listing reveals file names.
Step 2 — Check the confidential document
The standard Juice Shop Confidential Document exercise identifies acquisitions.md as the confidential file. Confirm the exact path in your own lab before publishing evidence.
GET /ftp/acquisitions.md HTTP/1.1
Host: 192.168.219.20:3000
- Repeat without a bearer token or authenticated session cookie.
- Record the actual HTTP status, Content-Type, and whether document text is returned.
- Record why this specific file is not intended for public access.
Local response details to insert: [HTTP status], [Content-Type], [redacted excerpt or screenshot demonstrating file access].
4. Evidence
ID	Evidence / observation	Verification
E01	Legitimate public file link reveals /ftp/	Discovery method discussed; attach capture
E02	Directory request GET /ftp/	Attach actual response
E03	Anonymous request to confidential document	Attach request with authentication removed
E04	Confidential document content returned	HTTP proof not supplied in conversation — attach
E05	Juice Shop Fix It view deleting /ftp routes	Screenshot reviewed


Suggested screenshots:
evidence/finding-03/
├── ftp-directory-response.png
├── confidential-document-response.png
└── proposed-route-removal.png
<!-- Add images after placing them in the evidence directory:
![Confidential file response](evidence/finding-03/confidential-document-response.png)
-->

Publication note: Do not commit complete private documents or customer order confirmations. Use redacted screenshots and minimal excerpts showing the authorization failure.
5. Technical analysis and root cause
If a server exposes confidential files directly through a public static route, knowing or discovering the URL may be sufficient to retrieve them. The problem is access to the file contents, not merely the ability to see file names.
Two distinct weaknesses can occur:
Weakness	Meaning	Why it matters
Directory listing	GET /ftp/ reveals filenames.	Makes sensitive content easier to discover.
Public file access	GET /ftp/<file> serves data without authorization.	Directly discloses private content.


Turning off directory listing alone does not prevent a direct request to a known file path. The exercise's proposed code fix removes middleware serving the /ftp listing and download routes; whether any other routes remain publicly accessible requires separate checks.
Expected vs. observed / documented
Expected secure behavior	Observed or documented behavior
Confidential files are not served as public assets.	Juice Shop challenge uses a public /ftp/ file route.
Private downloads require an authorization check.	Challenge demonstrates a design where direct file URLs are exposed.
Public assets remain available after remediation.	The proposed route removal alone does not restore needed download functions.


6. Security impact
- Confidentiality: Internal business documents may be retrieved by unauthorized visitors if served without access checks.
- Information discovery: Directory listings may reveal internal file names and document structure.
- Privacy: Publicly stored order-confirmation PDFs may contain customer/order data; this is a potential risk described in the remediation explanation, not a separately confirmed leak in the provided lab evidence.
Severity rationale: Medium is an initial classification. The final rating should reflect actual unauthorized accessibility and the sensitivity of the confirmed document.
7. Recommended remediation
1. Remove public listing and file-serving middleware for /ftp/.
2. Inventory the folder, remove unneeded files, and relocate intentionally public assets to an appropriate public directory.
3. Store private documents and invoices outside any publicly served web root.
4. Create protected download endpoints that check identity and entitlement to each requested file.
5. Disable unnecessary directory indexing as an additional control, not as a substitute for authorization.
6. Test legitimate downloads after the change to prevent functional regression.
Illustrative replacement route (proposed design only):
app.get('/api/orders/:id/invoice', authenticate, authorizeOrderOwner, downloadInvoice);
This replacement endpoint was not shown as implemented by the Juice Shop fix; it is an example of how to preserve legitimate functionality safely.
8. Retesting / acceptance criteria
Retest	Pass condition
Browse /ftp/	No internal file listing
Request old confidential file URL anonymously	Confidential content not returned
Access relocated public documentation	Intended public documents still work
Download an invoice as its owner	Authorized download succeeds
Attempt invoice download as another user	Request denied; no file content
Search for alternative public routes	No second exposure path within the test scope


Retest status: Not performed; the patch shown in the exercise was reviewed, not deployed.
References
- MITRE CWE-552
- MITRE CWE-548
- OWASP Authorization Cheat Sheet
- OWASP Juice Shop Project