# Penetration Test Report: OWASP Juice Shop

| | |
| :--- | :--- |
| **Project Name** | Portfolio Sample: Web Application Penetration Test |
| **Target** | OWASP Juice Shop (`juice-shop.herokuapp.com`) |
| **Test Date** | November 17, 2025 |
| **Report Date** | November 17, 2025 |
| **Classification** | Public  |

---

## 1. Executive Summary

A penetration test was performed against the OWASP Juice Shop web application. The assessment identified several vulnerabilities, ranging from **Critical** to **Low** severity. The most significant findings include a **Cross-Site Scripting (XSS)** vulnerability allowing for arbitrary script execution and a **Broken Access Control** vulnerability that permits unauthorized access to other users' data.

These vulnerabilities expose the application and its users to significant risk, including data theft, account compromise, and reputational damage. This report details all findings and provides actionable recommendations for remediation.

---

## 2. Scope & Methodology

### 2.1 Scope

The scope of this assessment was limited to the web application publicly available at:

* `https://juice-shop.herokuapp.com/`

### 2.2 Methodology

This assessment simulated an external attacker with no prior knowledge of the application's internal structure. The testing methodology included:

* **Manual Inspection:** Manually browsing the application, analyzing traffic, and testing for common vulnerability classes.
* **Automated Scanning:** Using tools like OWASP ZAP to identify common vulnerabilities (e.g., version disclosure, missing security headers).
* **Vulnerability Validation:** Manually confirming and validating all findings to remove false positives and assess real-world impact.

---

## 3. Summary of Findings

The following table provides a high-level overview of the findings, prioritized by risk.

| Severity | Vulnerability Class | Finding |
| :--- | :--- | :--- |
| **Critical** | Cross-Site Scripting (XSS) | Stored XSS in Product Review |
| **High** | Broken Access Control | Insecure Direct Object Reference (IDOR) on User Baskets |
| **Medium** | Information Exposure | Sensitive Data (Error Messages) |
| **Low** | Security Misconfiguration | Server Version Disclosure |

---

## 4. Detailed Findings

This section provides a detailed description of each vulnerability, including steps to reproduce, impact analysis, and remediation recommendations.

### 4.1. Stored Cross-Site Scripting (XSS)

* **Severity:** <font color="red">**Critical**</font>
* **Vulnerability Class:** A03:2021-Injection
* **Location:** Product Review Function

**Description:**
The application's product review feature fails to properly sanitize user-supplied input before storing and rendering it on the page. An attacker can submit a review containing a malicious JavaScript payload. This payload will be stored in the database and executed in the browser of any user (including administrators) who views that product's reviews.

**Steps to Reproduce:**
1.  Log in as a user and navigate to any product page.
2.  Leave a review for the product.
3.  In the review text field, submit the following payload: `<script>alert('XSS Demo')</script>`
4.  Log out and log in as a different user (or an admin).
5.  View the product page. The script will execute, and an alert box will appear.

**Business Impact:**
A successful XSS attack could lead to:
* Stealing user session cookies and hijacking their accounts.
* Redirecting users to malicious websites (phishing).
* Modifying the content of the page (defacement).
* Capturing keystrokes (keylogging).

**Recommendation:**
Implement **context-aware output encoding** on all user-supplied data before it is rendered in the HTML. For example, characters like `<` and `>` should be encoded to `&lt;` and `&gt;`. Use a trusted, modern web framework that provides this functionality automatically.

---

### 4.2. Broken Access Control (IDOR)

* **Severity:** <font color="orange">**High**</font>
* **Vulnerability Class:** A01:2021-Broken Access Control
* **Location:** User Basket API (`/rest/basket/`)

**Description:**
The application's API endpoint for retrieving a user's shopping basket uses a non-sequential, but guessable, identifier (the basket ID). The server fails to check if the user making the request is the actual owner of the basket. An authenticated attacker can intercept their own traffic, identify their basket ID, and then send requests with slightly different IDs to access the baskets of other users.

**Steps to Reproduce:**
1.  Log in as "User A" and add an item to the basket.
2.  Using an intercepting proxy (like Burp Suite), observe the `GET` request to `/rest/basket/1`. (Note: '1' is the user's basket ID).
3.  Modify the request and send it to the server again, but change the ID to `2` (i.e., `GET /rest/basket/2`).
4.  **Result:** The server returns the basket contents for "User B" (associated with ID 2), allowing "User A" to see what is in their cart.

**Business Impact:**
* Violation of user privacy through exposure of personal data (what items a user is purchasing).
* Potential for further attacks, such as modifying another user's basket (e.g., changing quantities, removing items).

**Recommendation:**
The server must **enforce access control** on all API endpoints. When a user requests a resource (like a shopping basket), the server must perform two checks:
1.  Is the user authenticated? (Check for a valid session token).
2.  Is the user *authorized* to access this specific resource? (Check that `requested_basket_id` matches the `session_user_id`).

If the user is not the owner of the resource, the server should return a `403 Forbidden` or `404 Not Found` error.

---

## 5. Conclusion

The OWASP Juice Shop application contains several critical vulnerabilities that demonstrate a lack of secure coding practices. We recommend prioritizing the remediation of the Stored XSS and Broken Access Control vulnerabilities, as they pose the most significant and immediate risk to the application and its users.
