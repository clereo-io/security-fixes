# Website fixes

Each section has the header or setting, a safe value, and how to roll it out. For the whole set in one file, see [nginx.conf](nginx.conf), [apache.conf](apache.conf), [cloudflare.md](cloudflare.md), [vercel.json](vercel.json) or [netlify/_headers](netlify/_headers).

## HTTPS redirect

Every `http://` request should get a permanent redirect (301) to the same address on `https://`.

nginx:

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name example.com www.example.com;
    return 301 https://$host$request_uri;
}
```

Apache:

```apache
<VirtualHost *:80>
    ServerName example.com
    ServerAlias www.example.com
    Redirect permanent / https://example.com/
</VirtualHost>
```

Caddy, Vercel and Netlify redirect for you once HTTPS is on. In Cloudflare, turn on **Always Use HTTPS** (SSL/TLS, Edge Certificates).

## TLS versions

Offer TLS 1.2 and 1.3 only. TLS 1.0 and 1.1 are retired, and every current browser supports 1.2.

```nginx
ssl_protocols TLSv1.2 TLSv1.3;
```

```apache
SSLProtocol -all +TLSv1.2 +TLSv1.3
```

In Cloudflare: SSL/TLS, Edge Certificates, **Minimum TLS Version** set to 1.2.

Check every host, not only the main site: an old API or mail host is often the one still offering TLS 1.0.

## HSTS

Tells browsers to only ever use HTTPS for your site, so a first plain-HTTP request can't be intercepted.

```
Strict-Transport-Security: max-age=31536000; includeSubDomains
```

Roll it out in steps, because browsers remember it for `max-age` seconds and you can't take it back early:

1. Make sure every subdomain works over HTTPS (with `includeSubDomains`, a subdomain without HTTPS becomes unreachable).
2. Start with `max-age=300` (5 minutes), check nothing breaks, then raise it to a week, then to a year.
3. Only add `preload` and submit to [hstspreload.org](https://hstspreload.org) once you're sure: removal from the preload list takes months.

## Content-Security-Policy

Limits where scripts can load from, so an injected `<script>` doesn't run. What matters most is the script rule: a policy that still allows `'unsafe-inline'` scripts, or scripts from any `https:` origin, doesn't stop injected scripts.

Start in report-only mode. The browser reports what it would have blocked, in the developer console, without blocking anything:

```
Content-Security-Policy-Report-Only: default-src 'self'; script-src 'self'; object-src 'none'; base-uri 'self'; frame-ancestors 'none'
```

Add the hosts you really load from (analytics, payment forms, fonts) until the console is quiet, then rename the header to `Content-Security-Policy` to enforce it.

If your pages need inline scripts, give each `<script>` tag a random value generated on every request (a nonce), and allow that nonce instead of `'unsafe-inline'`:

```
Content-Security-Policy: script-src 'nonce-RANDOM_PER_REQUEST' 'strict-dynamic'; object-src 'none'; base-uri 'self'
```

```html
<script nonce="RANDOM_PER_REQUEST">...</script>
```

- The nonce must change on every response. A fixed nonce protects nothing.
- Move `onclick="..."` style handlers into script files: nonces don't cover them.
- Remove `*`, `https:` and `data:` from `script-src`, and from `default-src` if there's no `script-src`.
- Most frameworks have nonce support built in (Next.js middleware, Rails `content_security_policy_nonce_generator`, Django `django-csp`, Helmet for Express).

## Clickjacking

Stops other sites from showing your pages inside a frame and tricking users into clicking.

```
X-Frame-Options: DENY
Content-Security-Policy: frame-ancestors 'none'
```

`frame-ancestors` is the modern version; send both for older browsers. Use `SAMEORIGIN` and `frame-ancestors 'self'` if your own site frames its pages, or list the sites allowed to embed you.

## X-Content-Type-Options

Stops browsers from guessing a file's type, so an uploaded file can't be run as a script.

```
X-Content-Type-Options: nosniff
```

Send it on every response, on every host. Make sure your server sends the right `Content-Type` for each file, since browsers will now trust it.

## Server version

Version numbers in `Server` or `X-Powered-By` headers tell an attacker which known vulnerabilities to try.

```nginx
server_tokens off;
```

```apache
ServerTokens Prod
ServerSignature Off
```

Remove `X-Powered-By` in your framework:

- Express: `app.disable("x-powered-by")`
- PHP: `expose_php = Off` in php.ini
- ASP.NET: `<httpRuntime enableVersionHeader="false" />` and remove the header in IIS

Hiding versions isn't a fix for being out of date. Keep the software updated too.

## Cookies

Session and login cookies should be:

```
Set-Cookie: session=...; Secure; HttpOnly; SameSite=Lax; Path=/
```

- `Secure`: only sent over HTTPS.
- `HttpOnly`: not readable from JavaScript, so an injected script can't steal it.
- `SameSite=Lax` (or `Strict`): not sent with most cross-site requests, which blocks most cross-site request forgery.

These are usually set in your framework's session settings, not in the web server.

## Mixed content

An HTTPS page that loads anything over plain `http://` lets someone on the network change it. A form that submits to `http://` sends what the user typed in the clear.

1. Change every `http://` address in your pages, scripts, stylesheets, frames and form `action`s to `https://`, or to a relative path for your own files.
2. If a third-party host doesn't serve HTTPS, self-host the file or switch provider.
3. To have browsers upgrade any you missed, add this to your Content-Security-Policy:

```
Content-Security-Policy: upgrade-insecure-requests
```

## Subresource Integrity

A script loaded from a public CDN runs with full access to your page. If the CDN file is changed, so is your site. Pin the exact version and add its hash, so the browser refuses a changed file:

```html
<script src="https://cdn.jsdelivr.net/npm/example-lib@1.2.3/dist/lib.min.js"
        integrity="sha384-HASH"
        crossorigin="anonymous"></script>
```

Generate the hash at [srihash.org](https://www.srihash.org), or with:

```sh
curl -s URL | openssl dgst -sha384 -binary | openssl base64 -A
```

Or self-host the file. Scripts that change on purpose (analytics tags, chat widgets) can't use a hash; keep that list short.
