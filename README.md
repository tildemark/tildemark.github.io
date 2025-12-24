# 👨‍💻 Alfredo Sanchez Jr (Tildemark)

[![Build Status](https://img.shields.io/github/actions/workflow/status/tildemark/tildemark.github.io/pages-deploy.yml?style=flat-square)](https://github.com/tildemark/tildemark.github.io/actions)
[![Live Site](https://img.shields.io/website?url=https%3A%2F%2Fblog.sanchez.ph&style=flat-square&label=blog.sanchez.ph)](https://blog.sanchez.ph)

> **Welcome!** This repository houses the source code for my personal website and digital garden, accessible at [blog.sanchez.ph](https://blog.sanchez.ph).

## 📖 About Me
I am a technology enthusiast and writer based in the Philippines. I use this platform to document my learning journey, share technical notes, and write about topics that interest me.

* **📍 Location:** Philippines
* **🕸️ Website:** [blog.sanchez.ph](https://blog.sanchez.ph)

## ⚠️ Note to Forkers (CNAME)
**If you fork this repository to build your own blog, please read this:**

This repository contains a `CNAME` file in the root directory that binds the GitHub Pages deployment to my custom domain (`blog.sanchez.ph`).
1.  **Delete or Edit the `CNAME` file:** Immediately after forking, delete this file or change the content to your own custom domain.
2.  **Update `_config.yml`:** Change the `url` and `avatar` fields to match your own details.

## 🛠️ Tech Stack
This blog is built on the **JAMstack** architecture, designed for speed and security.

* **Framework:** [Jekyll](https://jekyllrb.com/)
* **Theme:** [Chirpy](https://github.com/cotes2020/jekyll-theme-chirpy) (customized)
* **Hosting:** GitHub Pages
* **Deployment:** GitHub Actions

## 🚀 Running Locally
To test this site on your local machine:

1.  **Clone the repo**
    ```bash
    git clone [https://github.com/tildemark/tildemark.github.io.git](https://github.com/tildemark/tildemark.github.io.git)
    cd tildemark.github.io
    ```

2.  **Install dependencies**
    ```bash
    bundle install
    ```

3.  **Run the server**
    ```bash
    bundle exec jekyll serve
    ```

## 📝 License
Content © **Alfredo Sanchez Jr**.
Theme source code is licensed under MIT by [Cotes Chung](https://github.com/cotes2020).