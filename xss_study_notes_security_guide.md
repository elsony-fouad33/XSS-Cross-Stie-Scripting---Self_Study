# Web Security & Vulnerabilities: XSS (Cross-Site Scripting)

Welcome to the **Cross-Site Scripting (XSS)** study guide and reference repository. This repository contains structured study notes, concepts, attack vectors, evasion methods, and code-level remediation examples for understanding and preventing XSS vulnerabilities.

---

## Table of Contents
1. [Introduction to XSS](#1-introduction-to-xss)
2. [Types of XSS](#2-types-of-xss)
3. [Same-Origin Policy (SOP)](#3-same-origin-policy-sop)
4. [Implications & Impact of XSS](#4-implications--impact-of-xss)
5. [Understanding the DOM (Document Object Model)](#5-understanding-the-dom-document-object-model)
6. [Injection Context & Filter Evasion](#6-injection-context--filter-evasion)
7. [Defense, Sanitization & Encoding](#7-defense-sanitization--encoding)
8. [Code Examples (Prevention Across Languages)](#8-code-examples-prevention-across-languages)

---

## 1. Introduction to XSS

* **Definition:** XSS (Cross-Site Scripting) happens when a web application handles untrusted user input without adequate validation or escaping, allowing the victim's browser to execute untrusted scripts.
* **Core Problem:** The vulnerability does not lie in JavaScript itself, but in the fact that the application allows unsanitized inputs to be treated as executable client-side code.

### XSS Execution Lifecycle
```text
[ Attacker Input ]  --> Injects malicious JS payload
        │
        ▼
[ Vulnerable Web App ] --> Fails to escape / sanitize input
        │
        ▼
[ Victim's Browser ] --> Interprets and executes payload inside site context
```

---

## 2. Types of XSS

| Type | Payload Storage | Execution Trigger | Vulnerable Path / Context Example |
| :--- | :--- | :--- | :--- |
| **Reflected XSS** *(Non-Persistent)* | In the Request (URL / Parameters) | Opening a specially crafted link | `/reflected?q=<script>...` |
| **Stored XSS** *(Persistent)* | Database / Server Storage | Every visitor loading the affected page | `/guestbook`, `/comments` |
| **Blind XSS** | Saved in Server / DB | Executed inside admin/internal panels | `/contact`, `/admin/messages` |
| **DOM-based XSS** | Dynamic execution via Client-Side DOM | Browser processes malicious DOM input | Client-side JS processing (`location.hash`) |

* **Path / URL Concept:** "Path" refers to the route or URL within the application where the injection occurs or gets executed.
* **Basic Test Payload:**
  ```html
  <script>alert(1)</script>
  ```

---

## 3. Same-Origin Policy (SOP)

* **Definition:** Browser security mechanism that prevents scripts on one origin from reading or modifying data on another origin.
* **An Origin is defined by 3 components:**
  1. **Protocol:** (e.g., `http://` vs `https://`)
  2. **Hostname:** (e.g., `example.com`)
  3. **Port:** (e.g., `:80`, `:443`, `:8080`)
* **XSS & SOP Impact:** An XSS attack effectively runs within the application's origin, completely bypassing SOP protections for that specific site.

---

## 4. Implications & Impact of XSS

1. **Session Hijacking:** Stealing session tokens (`document.cookie`) to impersonate legitimate users.
2. **Credential Theft & Phishing:** Injecting fake login forms directly into trusted pages.
3. **Content Manipulation & Defacement:** Altering the appearance or behavior of a web page.
4. **Data Exfiltration:** Exfiltrating sensitive user or administrative data to attacker-controlled servers.
5. **Malware / Hooking:** Redirecting users to malware sites or hooking the session via frameworks like BeEF.

---

## 5. Understanding the DOM (Document Object Model)

* **Definition:** An API interface representing an HTML document as a dynamic tree structure, allowing JavaScript to dynamically access and manipulate elements.
* **DOM Tree Representation:**
  ```text
  document
     └── html
          ├── head
          │     ├── title
          │     ├── meta
          │     └── style
          └── body
  ```
* **Common DOM Methods:**
  * `document.createElement()` → Dynamically creates a new HTML node.
  * `element.appendChild()` → Appends a child node to an existing element.
* **DOM XSS Key Takeaway:** Execution occurs locally inside the browser. It does not necessarily require round-trip server interaction if client-side scripts blindly trust DOM sources (e.g., `location.search`, `location.hash`).

---

## 6. Injection Context & Filter Evasion

### Context
Understanding *where* the input lands inside the source code dictates the required payload format:
* **Between HTML Tags:** `<div> USER_INPUT </div>` → Needs HTML tags (e.g., `<script>`).
* **Inside HTML Attributes:** `<input value="USER_INPUT">` → Requires breaking out of quotes (e.g., `" onfocus="alert(1)`).
* **Inside JS Code Block:** `let username = 'USER_INPUT';` → Requires closing JS syntax (e.g., `'; alert(1); //`).

### Filter Evasion Techniques
Methods used by attackers to bypass restrictive application filters:
* **Alternative Character Encodings (Hex Whitespace Characters):**
  * `TAB` → `0x09` (Horizontal Tab)
  * `LF`  → `0x0A` (Line Feed / New Line)
  * `CR`  → `0x0D` (Carriage Return)
* **Payload Variations:** Case variations (`<sCrIpT>`), tag variations (`<img src=x onerror=alert(1)>`), or short payloads.

---

## 7. Defense, Sanitization & Encoding

Security requires a defense-in-depth approach covering validation, sanitization, and context-aware output encoding.

```text
               ┌────────────────────────┐
               │    User Input Received │
               └───────────┬────────────┘
                           │
                           ▼
               ┌────────────────────────┐
               │    Input Validation    │  (Format / Type Checking)
               └───────────┬────────────┘
                           │
                           ▼
               ┌────────────────────────┐
               │   Sanitization / Clean │  (Remove dangerous tags)
               └───────────┬────────────┘
                           │
                           ▼
               ┌────────────────────────┐
               │ Context Output Encoding│  (HTML Entity Encoding)
               └────────────────────────┘
```

### Key Differences
* **Validation:** Ensuring input conforms to expected formats (e.g., checking if an email matches a regex pattern). *Validation alone does not prevent XSS.*
* **Sanitization:** Stripping or transforming unsafe HTML elements and attributes before saving/processing.
* **Output Encoding:** Converting special characters into safe HTML entity equivalents before rendering them in the client browser.

### HTML Entity Encoding Rules
* `<`  ──>  `&lt;`
* `>`  ──>  `&gt;`
* `&`  ──>  `&amp;`
* `"`  ──>  `&quot;`
* `'`  ──>  `&#x27;`

---

## 8. Code Examples (Prevention Across Languages)

### 1. JavaScript (Client-Side Sanitization)
```javascript
// Function to encode HTML special characters before DOM injection
function sanitizeHTML(str) {
    return str.replace(/[&<>"']/g, function(match) {
        const escapeMap = {
            '&': '&amp;',
            '<': '&lt;',
            '>': '&gt;',
            '"': '&quot;',
            "'": '&#x27;'
        };
        return escapeMap[match];
    });
}

// Safe usage with innerText / textContent instead of innerHTML
const userInput = "<script>alert('xss')</script>";
document.getElementById("output").textContent = userInput;
```

### 2. Python (Flask Backend Encoding)
```python
from flask import Flask, request, render_template_string
import html

app = Flask(__name__)

@app.route('/comment', methods=['POST'])
def add_comment():
    raw_comment = request.form.get('comment', '')
    
    # Secure encoding before storing/rendering
    safe_comment = html.escape(raw_comment)
    
    return f"Comment received safely: {safe_comment}"

if __name__ == '__main__':
    app.run()
```

### 3. ASP.NET / C#
```csharp
using System.Web;

public class SecurityUtility
{
    public static string EncodeInput(string userInput)
    {
        // Encodes user input before displaying stored content
        return HttpUtility.HtmlEncode(userInput);
    }
}
```

---

## 📁 Suggested Repository Structure

```text
.
├── README.md               # Main repository documentation & notes
└── assets/                 # (Optional) Place notebook photos here for reference
    ├── note_page_1.png
    ├── note_page_2.png
    └── note_page_3.png
```