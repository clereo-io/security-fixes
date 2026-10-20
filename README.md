# Security fixes

Copy-paste fixes for the website and email security findings that come up most often in security reviews and outside scans. Each one says what the setting does, a safe value to start with, and how to roll it out without breaking anything.

Replace `example.com` with your own domain everywhere.

## Website

| Finding | Fix |
| --- | --- |
| HTTP doesn't redirect to HTTPS | [web/README.md#https-redirect](web/README.md#https-redirect) |
| Old TLS versions (1.0, 1.1) enabled | [web/README.md#tls-versions](web/README.md#tls-versions) |
| No HSTS | [web/README.md#hsts](web/README.md#hsts) |
| No Content-Security-Policy, or one that allows inline scripts | [web/README.md#content-security-policy](web/README.md#content-security-policy) |
| Pages can be framed (clickjacking) | [web/README.md#clickjacking](web/README.md#clickjacking) |
| No `X-Content-Type-Options: nosniff` | [web/README.md#x-content-type-options](web/README.md#x-content-type-options) |
| Server version disclosed | [web/README.md#server-version](web/README.md#server-version) |
| Cookies without Secure, HttpOnly or SameSite | [web/README.md#cookies](web/README.md#cookies) |
| Mixed content (HTTP resources on an HTTPS page) | [web/README.md#mixed-content](web/README.md#mixed-content) |
| CDN scripts without Subresource Integrity | [web/README.md#subresource-integrity](web/README.md#subresource-integrity) |

Ready-made configs with all of the headers above:

- [nginx](web/nginx.conf)
- [Apache](web/apache.conf)
- [Cloudflare](web/cloudflare.md)
- [Vercel](web/vercel.json)
- [Netlify](web/netlify/_headers) (the same file works on Cloudflare Pages)

## Email

| Finding | Fix |
| --- | --- |
| No SPF, or more than one SPF record | [email/README.md#spf](email/README.md#spf) |
| No DKIM | [email/README.md#dkim](email/README.md#dkim) |
| No DMARC, or `p=none` | [email/README.md#dmarc](email/README.md#dmarc) |
| No MTA-STS | [email/README.md#mta-sts](email/README.md#mta-sts) |
| A domain that sends no email can be spoofed | [email/README.md#domains-that-send-no-email](email/README.md#domains-that-send-no-email) |

## DNS and domain

| Finding | Fix |
| --- | --- |
| No CAA record | [dns/README.md#caa](dns/README.md#caa) |
| No DNSSEC | [dns/README.md#dnssec](dns/README.md#dnssec) |
| No transfer lock, or auto-renew off | [dns/README.md#registration](dns/README.md#registration) |
| Records pointing at services you no longer use | [dns/README.md#dangling-records](dns/README.md#dangling-records) |

## Security contact

| Finding | Fix |
| --- | --- |
| No security.txt | [security-txt/README.md](security-txt/README.md) |

## Before you paste

- Test on a staging site first when you can. HSTS and Content-Security-Policy in particular can break things if they're rolled out in one step; each section says how to stage them.
- These are starting points, not a full security review. They cover what can be seen from outside your systems.

## About

Maintained by [Clereo](https://clereo.io), which checks these settings from the outside every day. Found a mistake or have a better fix? Open an issue or a pull request.

MIT licensed: copy anything here into your own configs.
