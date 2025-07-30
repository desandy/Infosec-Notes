==============================
🛡️ WEB APP PEN TESTING LOG
Target: https://example.com
Date: YYYY-MM-DD
Tester: [Your Name]
==============================

=== [APP OVERVIEW] ===
- App Name/Domain:
- App Functionality Summary:
- Authentication Required: Yes/No
- User Roles (if any): [e.g. Guest, User, Admin]
- Notes:
  - Found X hidden fields on /profile
  - Uses JSON API heavily
  - Requires login for most features

=============================================================
📌 INPUT INVENTORY (Mapped from Burp)
=============================================================

[1] Endpoint: /login
Method: POST
Parameters:
  - username (form)
  - password (form)
Headers: Standard + Custom if any
Tested:
  [x] SQLi
  [x] XSS
  [ ] Command Injection
  [ ] Brute Force
Notes:
  - Login returns 302 on success
  - Rate-limiting not observed

-------------------------------------------------------------

[2] Endpoint: /search
Method: GET
Parameters:
  - q (URL param)
Tested:
  [x] Reflected XSS
  [ ] SQLi
  [ ] Open Redirect
Notes:
  - Input is reflected unescaped in HTML body

-------------------------------------------------------------

[3] Endpoint: /api/user/1234
Method: GET
Parameters:
  - id (path)
Authentication: Yes
Tested:
  [x] IDOR
  [ ] Auth Bypass
  [ ] Rate Limiting
Notes:
  - Changing ID leaks other users' info

-------------------------------------------------------------

[4] Endpoint: /upload
Method: POST
Parameters:
  - file (multipart/form-data)
Tested:
  [ ] File Upload Restrictions
  [ ] Path Traversal
  [ ] MIME type bypass
Notes:
  - Only available to logged-in users

=============================================================
🧪 VULNERABILITY CHECKLIST
=============================================================
- [x] SQL Injection
- [x] Cross-Site Scripting (XSS)
- [ ] Command Injection
- [ ] File Upload / RCE
- [ ] IDOR / Broken Access Control
- [ ] Authentication Bypass
- [ ] CSRF
- [ ] SSRF
- [ ] Open Redirect
- [ ] Rate Limiting / Brute Force

=============================================================
🧷 FINDINGS / CONFIRMED ISSUES
=============================================================
1. 🔥 IDOR at /api/user/1234 — Able to access other users' data
2. ⚠️ Reflected XSS at /search?q= — Triggers alert with <script>
3. 🚧 File upload does not check MIME type or file extension

=============================================================
📝 NEXT STEPS / TODO
=============================================================
- Test /upload for RCE with polyglot files
- Recheck IDOR with higher privilege roles
- Try CSRF on sensitive actions like password change
- Look for hidden parameters or debug routes
- Fuzz headers like X-Forwarded-For, Referer, Host

=============================================================
📚 NOTES & RESOURCES
=============================================================
- HackTricks Reference: https://book.hacktricks.xyz/
- PayloadAllTheThings: https://github.com/swisskyrepo/PayloadsAllTheThings
- XSS Cheat Sheet: https://owasp.org/www-community/xss-filter-evasion-cheatsheet
