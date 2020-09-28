# TIL a bit about GitHub Pages and Cloudflare

How to check which name servers are being used for a domain:

```bash
dig -t ns your-domain.com
```
## Custom domin for GitHub Pages

When you you activate GitHub Pages for a repo you get a URL under the `github.io` domain (in my case `https://davidxcheng.github.io/utf-8-illustrator/`). Adding a custom domain is done by adding a `CNAME` which makes github redirect from your `github.io` URL to whatever is in your `CNAME` file.

### How to setup a custom domain with https for GitHub Pages when DNS is handled by Cloudflare

While trying to setup a custom domain for `https://davidxcheng.github.io/utf-8-illustrator` I struggled a bit trying to activate `https` for the custom domain (`utf-8-illustrator.com`). In the `GitHub Pages` section on the `Settings` tab I could not enable `Enforce HTTPS` and was given this error message:

> Unavailable for your site because your domain is not properly configured to support HTTPS

And the DNS lookup would not return GitHub's IP addresses that I had added as A-records over at Cloudflare. Instead `dig utf-8-illustrator.com` returned IP addresses that belonged to Cloudflare. Turns out it's because requests to the domain are proxied by Cloudflare.

The solution was to opt out of the proxy feature. Maybe not a smart move but at least that was the reason why GitHub was complaining.
