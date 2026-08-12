# Deploying to terence-egbelo.com

Three things have to happen: the repo goes to GitHub, GitHub Pages is switched on,
and DNS points the domain at GitHub. The DNS change is the slow one, so start it
first — it is independent of everything else.

---

## 1. DNS records (do this first)

Log in wherever **terence-egbelo.com** was bought. If you can't recall the
registrar, `whois terence-egbelo.com | grep -i registrar` will tell you.

Find the DNS / nameserver / zone editor. Add these.

### Apex domain — four A records

| Type | Name | Value | TTL |
|---|---|---|---|
| A | `@` | `185.199.108.153` | 300 (or lowest offered) |
| A | `@` | `185.199.109.153` | 300 |
| A | `@` | `185.199.110.153` | 300 |
| A | `@` | `185.199.111.153` | 300 |

All four. They are GitHub's Pages load balancers and the redundancy is the point.

### Apex domain — four AAAA records (IPv6, recommended)

| Type | Name | Value |
|---|---|---|
| AAAA | `@` | `2606:50c0:8000::153` |
| AAAA | `@` | `2606:50c0:8001::153` |
| AAAA | `@` | `2606:50c0:8002::153` |
| AAAA | `@` | `2606:50c0:8003::153` |

### www subdomain — one CNAME

| Type | Name | Value |
|---|---|---|
| CNAME | `www` | `terentivs.github.io.` |

Note the trailing dot — some registrars require it, most add it silently. The value
is the GitHub Pages host, **not** the custom domain.

### Things that go wrong here

- **Delete any existing A record or parking-page record at the apex first.** A
  leftover record from the registrar's default landing page will fight yours and
  you will get intermittent wrong results that look like caching.
- **`@` means the apex.** Some registrars want `@`, some want the field left blank,
  and a few want `terence-egbelo.com` typed in full. All three mean the same thing.
- **Never put a CNAME on the apex.** It is invalid DNS and breaks email and
  everything else on the domain. Apex gets A records; only `www` gets a CNAME.
  (If your registrar offers ALIAS or ANAME, that is the legitimate exception, but
  the A records are simpler and work everywhere.)
- **Cloudflare:** set the records to "DNS only" (grey cloud), not proxied (orange).
  Proxying prevents GitHub from issuing the TLS certificate. You can turn proxying
  on later, once HTTPS is working.

### Checking it took

```
dig +short terence-egbelo.com
dig +short www.terence-egbelo.com
```

The first should return the four `185.199.x.153` addresses. The second should
return `terentivs.github.io` followed by those addresses. Usually minutes; allow up
to 24 hours before assuming something is wrong.

---

## 2. Push the repo

The repo must be named **`TERENTIVS.github.io`** exactly — GitHub treats a repo
named `<username>.github.io` as a user site, which is what makes the apex domain
straightforward. It must be public.

From this directory:

```
gh repo create TERENTIVS.github.io --public --source=. --remote=origin --push
```

That creates the repo, adds the remote and pushes `main` in one step.

---

## 3. Turn on Pages and attach the domain

In the repo on GitHub: **Settings → Pages**.

1. **Source:** "Deploy from a branch" → branch `main`, folder `/ (root)`. Save.
2. Wait a minute, then confirm the site is live at `https://terentivs.github.io`.
   If it is, the build works and anything remaining is DNS.
3. **Custom domain:** enter `terence-egbelo.com` and save. GitHub runs a DNS check;
   it will fail until the records above have propagated, which is fine — it
   re-checks.
4. Once the check passes, tick **Enforce HTTPS**. It may be greyed out for up to 24
   hours while the certificate is issued. This is normal and needs no action.

The `CNAME` file in this repo already contains `terence-egbelo.com`, so step 3 will
find it consistent. Don't delete that file — GitHub reads it on every build, and
losing it drops the custom domain.

---

## 4. Before any of this: tell David

The write-up describes a bug in a live Koslicki Lab project and names the lab. The
story is favourable — the project caught the error and hardened its standards
because of it — but David should not first encounter it as a public post. One line
over Slack, with the link, before the repo goes public.

This is the only real risk in publishing and it costs nothing to remove.

---

## Afterwards

- Submit `https://terence-egbelo.com` to Google Search Console to speed up
  indexing. Owning the search result for your own name is the point of the site.
- Add the URL to the LinkedIn profile (Contact info → Website) and to both CVs.
- The LinkedIn feed post is a **separate, self-contained 300–500 word piece**, not
  a teaser — LinkedIn suppresses reach on posts with outbound links, so the post
  has to be worth reading on its own and close with the link. See
  `publications-memo.md` §2.
