# security.txt

A short text file that tells people who find a security problem how to reach you (RFC 9116). Without it, a researcher's report can end up in a sales inbox, or nowhere.

1. Copy [security.txt](security.txt) and fill in your own contact and date.
2. Serve it at `https://example.com/.well-known/security.txt`, over HTTPS, as `text/plain`.
3. Put a reminder in the calendar to renew `Expires` before it passes.

Fields:

- `Contact` (required): a `mailto:` address or an `https://` page for reports. More than one is fine; list the preferred one first.
- `Expires` (required): when the file should no longer be trusted, under a year away, in the format `2027-10-20T00:00:00.000Z`.
- `Policy`: a page describing how you handle reports.
- `Preferred-Languages`: for example `en`.
- `Canonical`: the file's own URL.

Make sure the contact address is read by a person and isn't caught by a spam filter.
