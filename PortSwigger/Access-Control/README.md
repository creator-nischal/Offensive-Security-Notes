# PortSwigger Access Control Vulnerabilities — Lab Writeups

This repository contains my notes and solutions for the **Access Control** labs in PortSwigger Web Security Academy. I wrote these while solving the labs to document not only the steps I used, but also what I learned from each vulnerability.

> **Scope:** These notes are based on intentionally vulnerable PortSwigger Web Security Academy labs and are intended for learning and authorized security testing only.

## Labs covered

1. [Unprotected admin functionality](#lab-1-unprotected-admin-functionality)
2. [Unprotected admin functionality with unpredictable URL](#lab-2-unprotected-admin-functionality-with-unpredictable-url)
3. [User role controlled by request parameter](#lab-3-user-role-controlled-by-request-parameter)
4. [User role can be modified in user profile](#lab-4-user-role-can-be-modified-in-user-profile)
5. [User ID controlled by request parameter](#lab-5-user-id-controlled-by-request-parameter)
6. [User ID controlled by request parameter, with unpredictable user IDs](#lab-6-user-id-controlled-by-request-parameter-with-unpredictable-user-ids)
7. [User ID controlled by request parameter with data leakage in redirect](#lab-7-user-id-controlled-by-request-parameter-with-data-leakage-in-redirect)
8. [User ID controlled by request parameter with password disclosure](#lab-8-user-id-controlled-by-request-parameter-with-password-disclosure)
9. [Insecure direct object references](#lab-9-insecure-direct-object-references)
10. [URL-based access control can be circumvented](#lab-10-url-based-access-control-can-be-circumvented)
11. [Method-based access control can be circumvented](#lab-11-method-based-access-control-can-be-circumvented)
12. [Multi-step process with no access control on one step](#lab-12-multi-step-process-with-no-access-control-on-one-step)
13. [Referer-based access control](#lab-13-referer-based-access-control)

---

## Lab 1: Unprotected admin functionality

![Lab 1](images/Pasted%20image%2020260927113816.png)

### How I solved it

One of the first things I check when testing a web application is `/robots.txt`, because it can reveal interesting paths that the application owner does not want search-engine crawlers to visit.

I requested:

```text
/robots.txt
```

and found:

```text
User-agent: *
Disallow: /administrator-panel
```

I then visited `/administrator-panel`, accessed the unprotected admin interface, and deleted the user `carlos` to solve the lab.

### What I learned

Do the basics first. Before trying complicated attacks, explore the application normally and check obvious discovery points such as `robots.txt`, page source, links, and common paths. Also, `robots.txt` is **not** an access-control mechanism; sensitive functionality must still be protected by server-side authorization.

---

## Lab 2: Unprotected admin functionality with unpredictable URL

![Lab 2](images/Pasted%20image%2020260927114654.png)

### How I solved it

I checked the page source to see whether the developers had exposed anything interesting in the client-side code.

![Admin path in source](images/Pasted%20image%2020260927115321.png)

I found this JavaScript line:

```javascript
adminPanelTag.setAttribute('href', '/admin-9xhy72');
```

The admin panel used an unpredictable path, but the path itself was exposed in the page source. I visited that path and deleted `carlos`.

### What I learned

A hidden or unpredictable URL is not a replacement for authorization. Client-side code can disclose paths, so sensitive endpoints still need proper server-side access control.

---

## Lab 3: User role controlled by request parameter

![Lab 3](images/Pasted%20image%2020260927115516.png)

### How I solved it

I logged in with the normal user account and tried to access `/admin`. Access was denied, so I reviewed the requests in Burp Suite.

The application stored the user's admin state in a client-controlled value:

```text
Admin=false
```

![Admin parameter](images/Pasted%20image%2020260927115737.png)

I changed the value to:

```text
Admin=true
```

and replayed the request. The application trusted the modified value and allowed me to access the admin functionality. I then used the same modified session state to delete `carlos`.

![Modified request](images/Pasted%20image%2020260927120401.png)

### What I learned

Never trust a client-controlled parameter to determine authorization. While reviewing requests, pay attention to cookies, query parameters, body parameters, and headers that appear to control a user's role or privilege level.

---

## Lab 4: User role can be modified in user profile

![Lab 4](images/Pasted%20image%2020260927121633.png)

### How I solved it

The lab description indicated that users with `roleid` equal to `2` could access the admin panel. I first checked the login requests, but I could not find an obvious role parameter that I could modify.

I then looked for a normal authenticated request that updated my account. The profile page allowed me to change my email address, which generated a request to update my user profile.

I added an extra parameter to that request:

```json
"roleid": 2
```

![Adding roleid](images/Pasted%20image%2020260927122219.png)

The server accepted the additional property and changed my account to an administrator. I then accessed the admin panel and deleted `carlos`.

### What I learned

Not every server-side parameter is exposed by the normal UI. When an endpoint accepts structured input, it is worth checking whether additional sensitive properties can be submitted and whether the server properly restricts which fields a normal user is allowed to modify.

---

## Lab 5: User ID controlled by request parameter

![Lab 5](images/Pasted%20image%2020260929143617.png)

### How I solved it

After logging in, I refreshed my profile and reviewed the request. I noticed that the requested user was controlled by an `id` parameter.

![User ID parameter](images/Pasted%20image%2020260929145842.png)

I replaced my own username with:

```text
carlos
```

The server returned Carlos's account information even though I was authenticated as a different user.

### What I learned

This is a classic access-control failure. The server trusted the object identifier supplied by the client without verifying that the authenticated user was authorized to access that object. Authorization must be checked server-side for every requested resource.

---

## Lab 6: User ID controlled by request parameter, with unpredictable user IDs

![Lab 6](images/Pasted%20image%2020260929150650.png)

### How I solved it

This lab used unpredictable user identifiers rather than simple usernames, but the underlying problem was the same: the server did not properly verify whether I was authorized to access another user's account.

![Request using user ID](images/Pasted%20image%2020260929151908.png)

I looked through the blog posts and found one written by the target user `carlos`. The application exposed Carlos's user ID when loading his blog content.

Once I had his ID, I supplied it to the account request and accessed his information.

### What I learned

Using an unpredictable identifier does not fix broken access control. If an identifier can be discovered somewhere else in the application, it can still be used to access another user's resources unless the server performs an authorization check.

---

## Lab 7: User ID controlled by request parameter with data leakage in redirect

![Lab 7](images/Pasted%20image%2020260929182841.png)

### How I solved it

After logging in, the application sent a GET request to load the profile identified by the `id` parameter.

I changed the parameter to:

```text
id=carlos
```

and inspected the response in Burp Suite. Even though the application attempted to prevent me from viewing another user's account through its normal navigation flow, the response body still contained Carlos's account data, including his API key.

![Carlos API key in response](images/Pasted%20image%2020261007200545.png)

### What I learned

A redirect does not protect sensitive data if that data has already been included in the HTTP response. The server must perform authorization **before** returning the protected resource.

---

## Lab 8: User ID controlled by request parameter with password disclosure

![Lab 8](images/Pasted%20image%2020260929162854.png)

### How I solved it

While reviewing the account page response, I noticed that the password value was present in the HTML response even though it was masked in the UI.

![Password present in response](images/Pasted%20image%2020260929162932.png)

I changed the account identifier to:

```text
administrator
```

The server returned the administrator's account page and disclosed the administrator password in the response.

### What I learned

There were two serious problems: the application exposed the password in the response, and it allowed one authenticated user to request another user's account data simply by changing an identifier. Sensitive credentials should not be returned unnecessarily, and every object-level request needs an authorization check.

---

## Lab 9: Insecure direct object references

![Lab 9](images/Pasted%20image%2020260929164337.png)

### How I solved it

I found functionality that allowed a user to download their live-chat transcript. When I reviewed the download request, I noticed that the transcript filename was controlled by the client and used a predictable numeric value.

For example:

```text
1.txt
```

By changing the filename, I could retrieve chat transcripts belonging to other users. One of the transcripts contained a password that was required to solve the lab.

![Transcript request](images/Pasted%20image%2020260929165505.png)

### What I learned

This is an IDOR/BOLA-style access-control issue. A server should never assume that knowing an object's filename or identifier means the requester is authorized to access it. Ownership or permission must be checked for every object request.

---

## Lab 10: URL-based access control can be circumvented

![Lab 10](images/Pasted%20image%2020260929165838.png)

### How I solved it

Direct access to `/admin` was blocked, so I tested whether the back-end framework trusted an alternative URL header.

I sent a request to an allowed path and added:

```http
X-Original-URL: /admin
```

The front-end access-control layer treated the request as a normal request to the public path, while the back end used the value in `X-Original-URL` when routing it.

To delete `carlos`, I used the same technique with the protected delete endpoint and supplied the username as a query parameter:

```http
GET /?username=carlos HTTP/2
X-Original-URL: /admin/delete
```

![X-Original-URL bypass](images/Pasted%20image%2020260929180833.png)

### What I learned

Access-control decisions must be consistent across proxies, front ends, and application servers. Headers such as `X-Original-URL` can affect routing, so relying on only one layer to block a URL can create a bypass.

---

## Lab 11: Method-based access control can be circumvented

![Lab 11](images/Pasted%20image%2020261007202112.png)

### How I solved it

I logged in to both accounts: `wiener` in the normal browser and `administrator` in an incognito window.

From the administrator account, I promoted `wiener` and captured the role-change request in Burp Suite. I then copied Wiener's session cookie into the request and replayed it. The server correctly responded with an unauthorized message when the request used `POST`.

To understand how the endpoint handled other HTTP methods, I changed the method from `POST` to `POSTX`. The response changed to a missing/invalid username-style error. This did **not** complete the bypass, but it showed that the application handled the request differently depending on the HTTP method.

I then used Burp Suite's **Change request method** option to convert the request from `POST` to `GET`. This moved the parameters into the query string. When I sent the GET request using Wiener's session, the role change succeeded and the normal user was promoted to administrator.

### What happened

The application enforced the authorization check for the `POST` version of the role-change request, but the same sensitive operation was also reachable using another HTTP method where the authorization check was missing.

### What I learned

Authorization must be enforced independently of the HTTP method. If an endpoint supports multiple methods, each method must apply the same authorization policy before performing a privileged action.

---

## Lab 12: Multi-step process with no access control on one step

![Lab 12](images/Pasted%20image%2020261007202724.png)

### How I solved it

I logged in as both `wiener` and `administrator` and saved Wiener's session cookie.

From the administrator account, I started the process of promoting `wiener` to administrator. The application then displayed a confirmation step.

I intercepted the request for this final confirmation step and replaced the administrator's session cookie with Wiener's session cookie. I then forwarded the request.

The server accepted the confirmation request and promoted the normal user to administrator.

### What I learned

A multi-step workflow is only as secure as its weakest step. Authorization must be checked at **every security-sensitive step**, especially the final request that actually performs the action.

---

## Lab 13: Referer-based access control

![Lab 13](images/Pasted%20image%2020261007203214.png)

### How I solved it

I logged in to both the `wiener` and `administrator` accounts and saved Wiener's session cookie.

From the administrator account, I captured the request used to promote `wiener`. I then replayed that request with Wiener's session cookie.

The role-change endpoint relied on the request's `Referer` header as part of its access-control decision. Because the captured request still contained a trusted admin-page Referer value, the request was accepted even though it was now being sent with a non-admin session.

### What I learned

The `Referer` header is client-controlled and must not be trusted for authorization. The server should verify the authenticated user's permissions directly before performing privileged actions.

---

## Main takeaway

Across all of these labs, the recurring lesson is simple: **access control must be enforced on the server for every sensitive request and every protected object**. Hiding a URL, using an unpredictable identifier, checking only one HTTP method, trusting a header, or validating only one step of a workflow does not provide reliable authorization.
