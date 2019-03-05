# kookie

Web | 10 points

Upon first look, the site offers us a set of standard credentials, `cookie`/`monster`.
The name seems to suggest that web cookies are used in some way. Sure enough, when we check
the cookies after logging in, we see an account cookie. Changing this value to `admin`
and reloading returns the flag.

```
CTF{kookie_cookies}
```
