---
title: "Migrating from Wordpress to Hugo"
date: 2026-01-31
slug: "migrating-wordpress-to-hugo"
aliases:
  - "/migrating-wordpress-to-hugo"
categories:
  - "Wordpress"
  - "Hugo"
tags:
  - "Wordpress"
  - "Hugo"
  - "Powershell"
---

# From WordPress to Hugo: Rebuilding My Blog for Speed, Security, and Maintainability

For years, my blog lived on WordPress. It worked, but it never felt like something I fully controlled. Between PHP vulnerabilities, plugin bloat, performance issues, and the constant need for updates, the platform carried more risk and overhead than I wanted for a personal site.

Eventually, I decided it was time for a change - not just a redesign, but a complete rebuild on a platform that aligned with how I like to work: fast, secure, version‑controlled, and future‑proof. That’s what led me to Hugo.

This article walks through why I migrated, how I built a fully automated conversion pipeline, and what I learned along the way.

---

## Why I Moved Away from WordPress

### 1. Security Concerns

WordPress is built on PHP, and while the core team does a solid job, the ecosystem introduces unavoidable risks:

- PHP vulnerabilities can expose entire sites.
- Plugins - even reputable ones - frequently become attack vectors.
- A single outdated dependency can compromise the whole installation.
- Shared hosting environments amplify risk.

For a personal blog, that level of exposure felt unnecessary. I wanted something static, immutable, and safe by design.

### 2. Plugin Bloat and Fragility

WordPress sites tend to accumulate plugins over time:

- SEO plugins  
- Image optimizers  
- Syntax highlighters  
- Backup tools  
- Caching layers  
- Theme‑specific helpers  

Each plugin adds code, complexity, and potential breakage. Updates sometimes fix issues, sometimes introduce new ones, and occasionally take down the entire site.

I wanted a system where *nothing* ran on the server. No plugins. No PHP. No database. Low attack surface.

### 3. Performance

Even with caching, WordPress can feel sluggish. Hugo, on the other hand, generates the entire site in milliseconds. The result is:

- Instant page loads  
- Zero server processing  
- No database queries  
- Perfect Lighthouse scores  

Static HTML is hard to beat.

### 4. Maintainability and Ownership

With WordPress, content lives in a database. With Hugo, every post becomes:


Everything is:

- Version‑controlled  
- Portable  
- Editable in any text editor  
- Easy to back up  
- Easy to automate  

It’s the difference between renting and owning.

---

## The Challenge: WordPress Exports Are Messy

WordPress exports posts as XML, and inside that XML is a mixture of:

- HTML inside CDATA blocks  
- WordPress shortcodes  
- Embedded scripts  
- Caption wrappers  
- Escaped entities  
- Inconsistent formatting  
- Images wrapped in `<a>` tags  
- Gists embedded via `<script>` tags  

A simple HTML‑to‑Markdown converter wasn’t going to cut it. I needed a custom migration pipeline.

---

## Building the Migration Pipeline

I wrote a PowerShell script that processes each post and outputs a clean Hugo content bundle. The pipeline handles:

### 1. Cleaning HTML Entities

WordPress escapes everything. I unescaped:

- `&lt;` → `<`  
- `&gt;` → `>`  
- `&amp;` → `&`  
- `&quot;` → `"`  
- `&#39;` → `'`  

This made the HTML parseable.

### 2. Removing WordPress Caption Shortcodes

WordPress stores captions like this:

`[caption id="..." align="..."]
<img ... /> Caption text here
[/caption]`  

If you remove the shortcode wrapper incorrectly, you either:

- Lose the image  
- Keep the caption text  
- Break the HTML  

The final solution extracts **only the `<img>` tag**, discarding the caption text entirely. This preserves the image while avoiding duplicated captions in Markdown.

### 3. Converting Links

I rewrote:

`<a href="URL">text</a>`

into:

`[text](URL)`

But with one critical rule:

**If the link wraps an image, leave it alone.**

Otherwise, the image disappears before extraction.

### 4. Extracting and Downloading Images

Every `<img>` tag is:

- Parsed  
- Downloaded  
- Saved into the post’s `img/` folder  
- Replaced with a Markdown image reference  

This ensures the site is fully self‑contained and not dependent on external WordPress media URLs.

### 5. Converting GitHub Gists

WordPress embeds Gists using `<script>` tags. Hugo uses:

`{{ < gist user id > }}`

The script automatically converts these.

### 6. Stripping Remaining HTML

Once images, links, and shortcodes are handled, the remaining HTML is safe to strip or convert.

### 7. Generating Hugo Front Matter

Each post gets:

- title  
- date  
- slug
- categories  
- tags  
- aliases (for redirects)  
- featured image  

Everything is clean, consistent, and ready for Hugo.

---

## The Result: A Clean, Secure, Future‑Proof Blog

After running the script, every WordPress post became a tidy Hugo content bundle. Images were local, links were Markdown, captions were fixed, and the entire site built instantly.

Deploying to GitHub Pages completed the transformation. Now my blog is:

- Static  
- Secure  
- Fast  
- Version‑controlled  
- Easy to maintain  

No PHP. No plugins. Low attack surface.

---

## Lessons Learned

### 1. WordPress hides a lot of complexity

You don’t notice it until you try to leave.

### 2. HTML is messy

Even CMS‑generated HTML is full of edge cases.

### 3. Automation is worth the investment

Once the script was right, I could re‑run the entire migration in seconds.

### 4. Hugo rewards structure

Content bundles, shortcodes, and Markdown make everything predictable.

---

## Would I Do It Again? Absolutely.

Moving from WordPress to Hugo wasn’t a one‑click migration - it was a project. But it gave me:

- a faster site  
- a more secure site  
- a cleaner workflow  
- full ownership of my content  
- a setup I can maintain for years  

If you’re considering the same move, it’s absolutely worth it. 