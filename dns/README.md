# DNS and domain fixes

## CAA

Names the certificate authorities allowed to issue certificates for your domain, so another one can't be tricked into issuing one for an attacker.

```
example.com.  CAA  0 issue "letsencrypt.org"
example.com.  CAA  0 issuewild ";"
example.com.  CAA  0 iodef "mailto:security@example.com"
```

- One `issue` line per authority you use. Common values: `letsencrypt.org`, `pki.goog` (Google Trust Services), `amazon.com` (AWS Certificate Manager), `digicert.com`, `sectigo.com`.
- `issuewild ";"` forbids wildcard certificates. Leave it out if you use one, or name the authority instead.
- Before adding CAA, check which authorities issued your current certificates (search your domain on [crt.sh](https://crt.sh)), including the ones your CDN or hosting provider gets for you. A missing authority means its next renewal fails.
- Cloudflare adds the entries its own certificates need once you have CAA records.

## DNSSEC

Signs your DNS answers so they can't be forged on the way to the person looking you up.

1. Turn on DNSSEC (zone signing) at your DNS provider. Most have a one-click setting.
2. Add the DS record it gives you at your registrar. If the registrar is also your DNS provider, this is usually automatic.

If you move DNS providers later, turn DNSSEC off (remove the DS record) first and wait a day, or the domain stops resolving during the move.

## Registration

A domain that lapses takes the website, email and every login reset link with it.

- Turn on **auto-renew**, with a card that won't expire before the renewal.
- Turn on the **transfer lock** (often called "registrar lock" or "domain lock") so the domain can't be moved to another registrar without you unlocking it first.
- Turn on two-factor login at your registrar.
- Make sure the registrar's contact email is one you still read, and isn't on the domain itself.

## Dangling records

A DNS record that points at something you've deleted (a cloud app, a storage bucket, a CDN endpoint) can let someone else create a resource with that name and serve their content on your subdomain.

- Delete the record for any hostname you no longer use.
- If you still need the name, re-create what it points to before anyone else can.
- Make deleting the DNS record part of shutting down any service.
