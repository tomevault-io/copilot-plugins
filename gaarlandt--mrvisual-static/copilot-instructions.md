## mrvisual-static

> How to Build a Modern Static Website with a Generator

How to Build a Modern Static Website with a Generator
Overview
This guide provides a modern approach to creating a simple, multi-page static website using a Static Site Generator (SSG). This method enhances the traditional static site model by allowing you to use templates, reusable components (like headers and footers), and centralized data. This makes your site much easier to manage and scale. The end result is still a collection of fast, secure, and easy-to-deploy static files.

For this guide, we will use Eleventy (11ty), a popular and flexible SSG.

Technology Stack
The website is built using a powerful and modern stack:

HTML & Templating: You'll write your content in HTML or Markdown. A templating language (like Nunjucks, which Eleventy uses by default) lets you create reusable layouts and inject data directly into your pages.

CSS: The styling is handled by a standard stylesheet (e.g., style.css), providing a consistent look and feel across all pages.

JavaScript: Used for any client-side interactivity, just like in a traditional static site.

Node.js & Eleventy (SSG): You need Node.js installed on your machine to run Eleventy. Eleventy is the command-line tool that will take all your source files, data, and templates and compile them into a finished, ready-to-publish website.

File Structure
An SSG uses a source directory for your development files and generates a final output directory (called _site by default) with the finished website. You only work in the source directory.

Here is a recommended source structure:

/
├── _data/
│   └── links.json      // Central file for your external links
├── _includes/
│   └── layout.html     // The main website template/layout
├── index.md            // Homepage content (using Markdown)
├── about.md            // About page content
├── contact.html        // A page written in pure HTML
└── style.css           // Your global stylesheet

_data/ directory: Any .json file you create here is automatically available as global data in all your templates. This is the perfect place to manage external links or site-wide information.

_includes/ directory: This is for reusable pieces of code, often called "partials" or "layouts." Your main page template (layout.html) lives here.

Content Files (.md, .html): These are your individual pages. They contain the content specific to that page and use "front matter" at the top to specify details like the title and which layout to use.

Static Files (.css, images, etc.): Eleventy copies any files it doesn't recognize (like CSS, images, or fonts) directly to the output folder, preserving the directory structure.

Getting Started
1. Project Setup
First, make sure you have Node.js installed. Then, open your terminal in a new project folder and run these commands:

# 1. Initialize a Node.js project (this creates a package.json file)
npm init -y

# 2. Install Eleventy as a development dependency in your project
npm install @11ty/eleventy --save-dev

2. Centralize Your Links
Create the _data folder and add a file named links.json. This becomes your single source of truth for external links. Note the corrected, simple string format.

// In _data/links.json
{
  "twitter": "[https://twitter.com/yourprofile](https://twitter.com/yourprofile)",
  "github": "[https://github.com/yourusername](https://github.com/yourusername)",
  "linkedin": "[https://www.linkedin.com/in/yourprofile/](https://www.linkedin.com/in/yourprofile/)"
}

3. Create the Main Layout
Create the _includes folder and add a file named layout.html. This is the master template for your pages.

<!-- In _includes/layout.html -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{{ title }}</title>  <!-- 'title' is a variable from our content pages -->
    <link rel="stylesheet" href="/style.css">
</head>
<body>
    <header>
        <h1>My Awesome Website</h1>
        <nav>
            <a href="/">Home</a>
            <a href="/about/">About</a>
        </nav>
    </header>

    <main>
        <!-- Page-specific content will be injected here -->
        {{ content | safe }}
    </main>

    <footer>
        <!-- Here's how you use your centralized link data! -->
        <p>
            Find me on <a href="{{ links.twitter }}">Twitter</a> and
            <a href="{{ links.github }}">GitHub</a>.
        </p>
    </footer>
</body>
</html>

4. Create Your Homepage
Now, create index.md in the root directory. The text at the top between --- is called front matter. It tells Eleventy which layout to use and provides data for the variables in that layout.

---
title: "Welcome to My Website"
layout: "layout.html"
---

## Hello World!

This is the homepage for my new website, built with Eleventy. It's much easier to manage content and links this way!

5. Add Your Stylesheet
Create a style.css file in your root directory. Eleventy will automatically copy this over when it builds the site.

6. Run the Eleventy Development Server
You're all set! In your terminal, run the following command:

npx @11ty/eleventy --serve

Eleventy will now build your site into a _site folder and start a local server, typically at http://localhost:8080. Open that URL in your browser. The best part is that it will automatically watch for changes and rebuild the site instantly whenever you save a file.

Conclusion
Using an SSG like Eleventy is the modern way to build static websites. You get the maintainability of a dynamic site (reusable templates, centralized data) while producing a final product that is just simple, fast, and secure HTML, CSS, and JavaScript files. This approach is the perfect solution for managing links and other repeated content efficiently across your entire site.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gaarlandt)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/gaarlandt)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
