# Joshua Robbins — Blog

This repository contains the source code for my personal blog, built with **Hugo** and the **Blowfish** theme.  
The site is deployed via **GitHub Pages** and published at:

👉 **https://blog.joshuarobbins.tech**

I write about automation, scripting, cybersecurity, web aesthetics, and the micro‑projects I build for fun and learning.

---

## 🚀 Tech Stack

- **Hugo** (static site generator)
- **Blowfish** theme (customized)
- **GitHub Pages** for hosting
- **Custom domain:** `blog.joshuarobbins.tech`

---

## 📁 Repository Structure

This repo follows the standard Hugo layout:

```
├── archetypes/        # Content templates
├── assets/            # Custom CSS, JS, and pipeline assets
├── content/           # Blog posts, pages, and sections
├── data/              # Optional structured data
├── layouts/           # Template overrides for Blowfish
├── static/            # Images, icons, and static files
├── themes/            # Blowfish theme (submodule or local copy)
└── config/_default/   # Site configuration
```

Customizations live primarily in:

- `assets/css/` — mesh background, animations, theme tweaks  
- `assets/js/` — mesh logic, parallax, timing fixes  
- `layouts/partials/` — homepage overrides, theme toggle suppression  

---

## 🛠️ Local Development

Run the development server:

`hugo server -D`

The -D flag includes draft posts.

## 🧱 Production Build

Generate the optimized static site:

`hugo --minify`

The output is placed in the public/ directory.GitHub Pages deploys this automatically via your workflow.

## 🌐 Deployment (GitHub Pages)

This site is deployed using GitHub Pages with a custom subdomain.

## DNS Setup

Create a **CNAME** record:

'blog  →  JoshRSec.github.io'

## GitHub Pages Settings

*   **Custom domain:** blog.joshuarobbins.tech
    
*   **Enforce HTTPS:** enabled
    

A CNAME file is automatically included during deployment.

## ✍️ Creating New Posts

Use Hugo’s built‑in generator:

`hugo new content/blog/my-post/index.md`

Each post gets its own folder so images and assets stay organized.

###🎨 Custom Enhancements

This blog includes several custom visual and functional improvements:

*   Interactive mesh background with clustering + pulse animations
    
*   Parallax layers for subtle depth
    
*   Homepage‑only dark‑mode lock
    
*   CSS/JS overrides for Blowfish
    
*   Timing fixes for theme initialization
    
*   Clean Slate‑inspired color palette
    

These enhancements are designed to be performant, maintainable, and visually cohesive.
