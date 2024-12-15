---
layout: post
title: Setting Personal Website on custom domain using Cloudflare
subtitle: Notes and challenges faced
---

I've finally decided to work on my personal website again: update remaining sections, and get a domain. Here, I'm going to share the mistakes I made, exact steps I followed to get my domain running. I'm pretty much writing this at the time of setting up the domain so this will contain even the smallest or silliest mistakes made, to remember and avoid them in the future.

## Purchasing the Domain

Looked at the following domains, ordered by preference:
- `arayofcode.tech`
- `arayofcode.me`
- `arayofcode.dev`
- `arayofcode.com`
- `arayofcode.site`

I was looking at a domain which wouldn't be too expensive to start with, and shouldn't become expensive to renew later. A recommendation: check out [TLD-List](https://tld-list.com/)!

While Hostinger, Godaddy and NameCheap were pretty cool, I noticed Cloudflare Registrar would've been the cheapest in longer run unless I chose a cheaper one and then moved to a different provider later. Finally went ahead with buying `arayofcode.com` on Cloudflare given this was the cheapest domain. 

### The first issue

While registering for the domain, here's what I saw:
![Payment Issue](/assets/posts/2024-12-15-dns-setting-personal-website/cloudflare-payment-issue.png)

Initially, I couldn't understand what went wrong. Forums suggested create ticket. I did. Eventually, figured out this was a horrible mistake from my end: the payment amount exceeded limit for international transactions enabled on my credit card. Took the L, increased limit on my card, and moved on.

## Configuration + Domain Mapping

With the new domain, I looked around in Cloudflare website to see all options available to me. Enabled the following features and settings:
- *Certificate Transparency Monitoring*: Useful to notify when a new certificate is issued
- *Always Use HTTPS*
- *Enable HSTS*
- *Security > Bots > Block AI Bots*

While I don't really expect much to happen there, probably a good idea to keep them enabled.

Now, to domain mapping: I have no idea! Did a course in Computer Networks back at college. I remember what CNAME records are, but no idea how to make them. If I had focused more on my classes, it would've helped. So, here's what I did: *trial and error* :P

Created the following CNAME:
![CNAME showing error](/assets/posts/2024-12-15-dns-setting-personal-website/CNAME_errors.png)

On going to my website, it either shows `Error 1016: Origin DNS error`, or Chrome's `ERR_NAME_NOT_RESOLVED` error. I figured it might be because GitHub Pages would need similar information too, so added my domain there and saw the following error:
![GitHub Pages Routing Error](/assets/posts/2024-12-15-dns-setting-personal-website/github-pages-routing-error.png)

A couple minutes later, the website began working. However, the warning still remains.

Looked up a little more, then set records according to [this documentation](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site#dns-records-for-your-custom-domain). This fixed the error shown earlier. However, I haven't understood what exactly was missing earlier that lead to this error.

