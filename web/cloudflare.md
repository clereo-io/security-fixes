# Cloudflare

Most of the website fixes are switches in the dashboard. Menu names move around; search the dashboard for the setting's name if it isn't where this says.

## SSL/TLS, Edge Certificates

- **Always Use HTTPS**: on. Redirects every HTTP request to HTTPS.
- **Minimum TLS Version**: 1.2.
- **TLS 1.3**: on.
- **HTTP Strict Transport Security (HSTS)**: enable it with a short max-age first, then raise it to 12 months once every subdomain works over HTTPS. Turn on "include subdomains" only when that's true. See [README.md#hsts](README.md#hsts).

## SSL/TLS, Overview

Set the encryption mode to **Full (strict)**. "Flexible" encrypts only between the visitor and Cloudflare: the connection from Cloudflare to your server is plain HTTP.

## Security headers

- **Cloudflare Pages** (and Workers static assets): add a [`_headers`](netlify/_headers) file to the folder you deploy. The same file works on Netlify.
- **Any other site behind Cloudflare**: Rules, Transform Rules, **Modify Response Header**. Create a rule for all incoming requests that sets:

| Header | Value |
| --- | --- |
| `X-Content-Type-Options` | `nosniff` |
| `X-Frame-Options` | `DENY` |
| `Referrer-Policy` | `strict-origin-when-cross-origin` |
| `Content-Security-Policy-Report-Only` | `default-src 'self'; script-src 'self'; object-src 'none'; base-uri 'self'; frame-ancestors 'none'; upgrade-insecure-requests` |

Rename `Content-Security-Policy-Report-Only` to `Content-Security-Policy` once the browser console shows nothing you need is blocked. See [README.md#content-security-policy](README.md#content-security-policy).

## DNS

- **DNSSEC**: DNS, Settings, enable DNSSEC, then add the DS record it shows at your registrar (automatic if the domain is registered with Cloudflare).
- **CAA**: if you add CAA records, Cloudflare adds the entries its own certificates need. See [../dns/README.md#caa](../dns/README.md#caa).
