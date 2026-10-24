# Web Security & Vulnerabilities: Cross-Site Scripting (XSS)

A structured cybersecurity study repository covering Cross-Site Scripting (XSS), its types, browser security mechanisms, injection contexts, filter-evasion techniques, security impacts, and prevention methods.

This repository documents concepts learned through self-study, web security labs, and practical cybersecurity exercises.

## Table of Contents

- [1. Introduction to XSS](#1-introduction-to-xss)
- [2. Types of XSS](#2-types-of-xss)
- [3. Same-Origin Policy (SOP)](#3-same-origin-policy-sop)
- [4. Security Impact of XSS](#4-security-impact-of-xss)
- [5. DOM Fundamentals](#5-dom-fundamentals)
- [6. Injection Contexts](#6-injection-contexts)
- [7. Filter Evasion Techniques](#7-filter-evasion-techniques)
- [8. XSS Prevention](#8-xss-prevention)
- [9. Secure Code Examples](#9-secure-code-examples)
- [10. Key Takeaways](#10-key-takeaways)
- [References](#references)

---

## 1. Introduction to XSS

### What is Cross-Site Scripting?

Cross-Site Scripting (XSS) is a web security vulnerability that allows an attacker to execute malicious client-side code, usually JavaScript, in another user's browser through a vulnerable web application.

### Root Cause

XSS occurs when an application handles untrusted data in an unsafe way, allowing the browser to interpret that data as executable code rather than ordinary content.

### How XSS Works

```text
[ Attacker-Controlled Input ]
              |
              v
[ Vulnerable Web Application ]
              |
              v
[ Unsafe HTML / JavaScript Context ]
              |
              v
[ Victim's Browser Executes Code ]
```

**Important:** The vulnerability is not JavaScript itself. The problem is the unsafe handling of untrusted data.

### Basic Testing Payload

Use the following harmless proof-of-concept payload in an authorized lab:

```html
<script>alert(1)</script>
```

If a JavaScript alert executes unexpectedly, it may indicate an XSS vulnerability. Always investigate the context and confirm the behavior before drawing conclusions.

---

## 2. Types of XSS

| Feature | Reflected XSS | Stored XSS | DOM-Based XSS |
|---|---|---|---|
| Data source | Usually a request parameter | Persisted application data | Client-side source |
| Typical location | URL or request | Database or other storage | Browser DOM |
| Execution | When a crafted request is processed | When affected content is viewed | When unsafe client-side code processes data |
| Server involvement | Usually involved in the response | Usually involved in storing or delivering content | Not necessarily involved in the vulnerable data flow |
| Example | Search results page | Comments or guestbook | Unsafe processing of `location.hash` |

### 2.1 Reflected XSS

The application immediately includes untrusted request data in its response without appropriate context-aware encoding.

**Example:** A search page displays a supplied search term unsafely.

### 2.2 Stored XSS (Persistent XSS)

The application stores attacker-controlled content, which is later rendered unsafely to users.

**Example:** A malicious comment executes when visitors open a vulnerable comments page.

Stored XSS can affect multiple users, including administrators.

### 2.3 DOM-Based XSS

The vulnerability occurs in client-side code when untrusted data flows into an unsafe DOM operation.

**Potential sources:**
- `location.search`
- `location.hash`
- `document.URL`

**Risky sinks:**
- `innerHTML`
- `document.write()`
- Other operations that interpret untrusted data as HTML or code

DOM-based XSS does not necessarily require the server to generate a vulnerable response.

### 2.4 Blind XSS

Blind XSS is a testing scenario in which a payload is submitted to an application but executes later in a context the tester does not immediately observe.

**Example:** Submitted feedback is reviewed later through an internal administrative dashboard.

Blind XSS is commonly associated with stored input, but it is a description of how execution is observed rather than a separate fundamental XSS mechanism.

---

## 3. Same-Origin Policy (SOP)

### Definition

The Same-Origin Policy is a browser security mechanism that restricts how documents and scripts from one origin can access certain resources belonging to another origin.

An origin consists of three components:

1. **Scheme:** `http` or `https`
2. **Hostname:** `example.com`
3. **Port:** `443`, `8080`, etc.

### Example

| URL | Same origin as `https://example.com`? |
|---|---|
| `https://example.com/page` | Yes |
| `http://example.com/page` | No — different scheme |
| `https://example.com:8443` | No — different port |
| `https://sub.example.com` | No — different hostname |

### XSS and SOP

XSS allows injected JavaScript to execute within the vulnerable website's origin. Consequently, the script may access data and perform actions available to ordinary scripts on that site, subject to browser and application protections.

**Key distinction:** XSS does not disable SOP globally. It abuses the trust that the vulnerable origin places in its own scripts.

---

## 4. Security Impact of XSS

| Impact | Description |
|---|---|
| Session compromise | Attempts to steal accessible session credentials or abuse an authenticated session |
| Credential phishing | Displays deceptive login forms or captures user-entered information |
| Content manipulation | Changes page content or redirects users |
| Unauthorized actions | Performs actions available to the victim's account |
| Sensitive data exposure | Reads information accessible to the injected script |
| Administrative compromise | May affect privileged users who visit the vulnerable page |

### Important Cookie Limitation

JavaScript cannot directly read cookies marked `HttpOnly` through `document.cookie`. However, XSS may still abuse an authenticated session by making requests from the compromised page, depending on the application's protections and the user's permissions.

---

## 5. DOM Fundamentals

### What Is the DOM?

The Document Object Model (DOM) represents an HTML document as a tree of objects that JavaScript can read and manipulate.

### Simplified DOM Tree

```text
document
└── html
    ├── head
    │   ├── title
    │   ├── meta
    │   └── style
    └── body
        ├── h1
        ├── p
        └── div
```

### Common DOM Methods and Properties

| Method / Property | Purpose |
|---|---|
| `document.createElement()` | Creates a new HTML element |
| `element.append()` | Adds content or nodes to an element |
| `element.textContent` | Reads or sets text content |
| `element.innerHTML` | Reads or parses HTML content |
| `document.write()` | Writes content into a document |

### DOM XSS: Source and Sink

- **Source:** Where potentially untrusted data originates.
- **Sink:** Where the data is used in a way that could cause unsafe execution.

Example of a risky data flow:

```javascript
const input = location.hash;
document.getElementById("output").innerHTML = input;
```

The problem is that `innerHTML` interprets the input as HTML.

A safer alternative when HTML is not required:

```javascript
const input = location.hash;
document.getElementById("output").textContent = input;
```

The second example treats the value as text instead of parsing it as HTML.

---

## 6. Injection Contexts

The location where untrusted data enters a page determines which defenses are appropriate.

| Context | Example | Security Consideration |
|---|---|---|
| Between HTML tags | `<div>USER_INPUT</div>` | HTML text encoding |
| Inside an HTML attribute | `<input value="USER_INPUT">` | Attribute-safe encoding and quoted attributes |
| Inside JavaScript | `const name = 'USER_INPUT';` | Avoid inserting untrusted data into executable JavaScript |
| Inside a URL | `<a href="USER_INPUT">Link</a>` | Validate allowed URL schemes and encode appropriately |
| Inside the DOM | `element.innerHTML = input` | Prefer safe DOM APIs such as `textContent` |

### Example: HTML Text Context

Unsafe pattern:

```javascript
output.innerHTML = userInput;
```

Safer pattern:

```javascript
output.textContent = userInput;
```

### Example: JavaScript Context

Avoid building executable JavaScript by concatenating untrusted input. Prefer passing data separately through JSON serialization or DOM APIs instead of embedding user-controlled values inside script source.

**Key takeaway:** There is no single encoding technique that safely handles every context.

---

## 7. Filter Evasion Techniques

Filter evasion describes attempts to bypass weak input filters or security controls.

Common categories include:

- **Case variations:** Changing letter capitalization.
- **Alternative HTML elements:** Testing how different elements are handled.
- **Encoding variations:** Using character encodings or representations accepted by the application.
- **Whitespace variations:** Testing how parsers and filters process whitespace.
- **Context manipulation:** Determining whether input can escape an attribute or another parsing context.

### Common Whitespace Character Values

| Character | Hexadecimal | Meaning |
|---|---|---|
| TAB | `0x09` | Horizontal tab |
| LF | `0x0A` | Line feed |
| CR | `0x0D` | Carriage return |

These values describe character representations; they do not automatically bypass a security filter.

### Defensive Lesson

Blacklist-based filtering is unreliable because HTML and JavaScript parsers support many different syntactic forms.

Use context-aware output encoding, safe DOM operations, and appropriate sanitization rather than relying on a list of blocked strings.

---

## 8. XSS Prevention

A secure implementation should combine multiple defenses.

### 8.1 Input Validation

Check whether input matches the application's expected format.

Examples:
- Validate email address structure.
- Restrict numeric fields to expected ranges.
- Apply reasonable length limits.

**Note:** Validation alone does not prevent XSS.

### 8.2 Output Encoding

Convert special characters into representations that are treated as data in the relevant output context.

Common HTML text encoding examples:

| Character | Encoded representation |
|---|---|
| `<` | `&lt;` |
| `>` | `&gt;` |
| `&` | `&amp;` |
| `"` | `&quot;` |
| `'` | `&#x27;` |

Encoding must match the context. HTML encoding is not a universal substitute for JavaScript, URL, or CSS encoding.

### 8.3 HTML Sanitization

When an application intentionally allows users to submit limited HTML, use a maintained HTML sanitization library configured with an appropriate allowlist.

For JavaScript applications, [DOMPurify](https://github.com/cure53/DOMPurify) is a widely used option.

Sanitization and output encoding solve different problems:

- **Sanitization:** Allows selected markup while removing unsafe constructs.
- **Output encoding:** Ensures data is interpreted as text in a specific output context.

### 8.4 Safe DOM APIs

Prefer:

```javascript
element.textContent = userInput;
```

Avoid inserting untrusted input directly into:

```javascript
element.innerHTML = userInput;
```

### 8.5 Content Security Policy (CSP)

CSP provides an additional layer of defense by restricting which scripts and other resources a page may load or execute.

It is a defense-in-depth measure, not a replacement for fixing the underlying injection vulnerability.

### 8.6 Secure Cookies

Use cookie attributes such as `HttpOnly`, `Secure`, and an appropriate `SameSite` setting. These reduce certain risks but do not eliminate XSS.

---

## 9. Secure Code Examples

### 9.1 JavaScript — Safely Display User Input

```javascript
const userInput = "<script>alert('xss')</script>";
const output = document.getElementById("output");

output.textContent = userInput;
```

**Why it is safer:** `textContent` displays the supplied value as text rather than interpreting it as HTML.

### 9.2 Python — Flask

```python
from flask import Flask, request, render_template_string

app = Flask(__name__)

@app.route("/comment", methods=["POST"])
def add_comment():
    comment = request.form.get("comment", "")

    return render_template_string(
        "<p>{{ comment }}</p>",
        comment=comment
    )

if __name__ == "__main__":
    app.run()
```

**Why it is safer:** Flask's standard Jinja template environment auto-escapes variables in HTML templates. In production, prefer separate template files and avoid disabling auto-escaping or marking untrusted data as safe.

Do not confuse input validation with output encoding, and avoid storing manually encoded data as a substitute for encoding it correctly when rendered.

### 9.3 ASP.NET / C#

```csharp
using System.Web;

public class SecurityUtility
{
    public static string EncodeForHtml(string userInput)
    {
        return HttpUtility.HtmlEncode(userInput);
    }
}
```

**Why it is safer:** `HtmlEncode` encodes characters that could otherwise be interpreted as HTML in an HTML text context.

Use the appropriate framework encoder for the actual output context.

---

## 10. Key Takeaways

- XSS occurs when untrusted input is interpreted as executable browser content.
- Reflected XSS involves request data, stored XSS involves persisted data, and DOM-based XSS involves unsafe client-side data flows.
- Blind XSS describes delayed or indirectly observed execution.
- SOP restricts cross-origin access, but XSS can execute with the vulnerable site's origin privileges.
- The DOM is central to understanding client-side web vulnerabilities.
- Injection context determines the appropriate defense.
- Input validation alone is insufficient.
- Prefer safe DOM APIs and context-aware output encoding.
- Use HTML sanitization when user-supplied markup is intentionally permitted.
- CSP and secure cookie settings provide additional protection but do not replace proper remediation.

---

**Disclaimer:** These notes are intended for educational purposes and authorized security testing in controlled environments.
