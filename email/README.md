# Email fixes

Without these, anyone can send email that looks like it came from your domain, and your customers are the ones who get phished. All of them are DNS records.

Order that works: SPF and DKIM first, then DMARC in monitor mode, then DMARC enforcement once everything you send passes.

## SPF

Lists the servers allowed to send mail as your domain. One TXT record at the domain itself (the apex, `example.com`):

```
v=spf1 include:_spf.google.com -all
```

Replace the include with your provider's. Common ones:

| Provider | Include |
| --- | --- |
| Google Workspace | `include:_spf.google.com` |
| Microsoft 365 | `include:spf.protection.outlook.com` |
| Amazon SES | `include:amazonses.com` |
| Mailgun | `include:mailgun.org` |
| SendGrid | `include:sendgrid.net` |

Check your provider's docs for the exact value, and add one include for each service that sends as you (newsletter tool, support desk, billing).

- **Exactly one** SPF record. Two records make SPF fail for everything: merge them into one.
- **At most 10 DNS lookups**. Each `include`, `a`, `mx` and `redirect` counts, and so do the includes inside them. Too many and SPF fails.
- **SPF doesn't inherit.** A record on `mail.example.com` covers mail from `@mail.example.com` only, not `@example.com`.
- End with `-all` (reject the rest) once every sender is listed. `~all` (soft fail) is fine while you're checking.

## DKIM

A signature on each message, checked against a public key in your DNS. Your mail provider generates the key: turn on DKIM signing in its admin console and publish the record it gives you, usually a TXT or CNAME at `SELECTOR._domainkey.example.com`.

- Google Workspace: Admin console, Apps, Google Workspace, Gmail, Authenticate email.
- Microsoft 365: Microsoft Defender portal, Email authentication settings, DKIM.

Do this for every service that sends as you, not only the main mailbox provider.

## DMARC

Tells receivers what to do with mail that fails SPF and DKIM, and where to send reports. One TXT record at `_dmarc.example.com`.

Start in monitor mode. Nothing is blocked; you get daily reports of who sends as you:

```
v=DMARC1; p=none; rua=mailto:dmarc@example.com
```

Once the reports show all your own mail passing (usually two to four weeks), move to quarantine, then reject:

```
v=DMARC1; p=quarantine; rua=mailto:dmarc@example.com
```

```
v=DMARC1; p=reject; rua=mailto:dmarc@example.com
```

`p=none` only watches. Spoofed mail is still delivered until you reach `quarantine` or `reject`.

The reports are XML and hard to read by hand; a free DMARC report reader helps. To send reports to an address on another domain, that domain has to allow it with a record at `example.com._report._dmarc.otherdomain.com` containing `v=DMARC1`.

## MTA-STS

Makes other mail servers use encryption when they deliver to you, and refuse to deliver if someone strips it. Two parts.

1. A policy file served at exactly `https://mta-sts.example.com/.well-known/mta-sts.txt`, with a valid certificate and no redirect. See [mta-sts.txt](mta-sts.txt):

```
version: STSv1
mode: testing
mx: mail.example.com
max_age: 86400
```

List every MX host exactly as your MX records name them. A wildcard like `*.example.net` covers one label only.

2. A TXT record at `_mta-sts.example.com`:

```
v=STSv1; id=20261020
```

Change the `id` every time you change the policy file, or senders keep the old one.

Run in `mode: testing` for a couple of weeks, then switch to `mode: enforce` and raise `max_age` to `604800` (a week) or more. For reports of delivery failures, add a TLS-RPT record at `_smtp._tls.example.com`:

```
v=TLSRPTv1; rua=mailto:tls-reports@example.com
```

## Domains that send no email

Spare domains, old brands and redirect-only domains can still be spoofed. Lock them down completely:

```
example.org.          TXT  "v=spf1 -all"
_dmarc.example.org.   TXT  "v=DMARC1; p=reject"
example.org.          MX   0 .
```

The last line is a "null MX" (RFC 7505): it says the domain receives no mail either.
