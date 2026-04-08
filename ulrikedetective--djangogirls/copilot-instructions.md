## djangogirls

> This project is a Django-based blog application. It allows users to create, view, and edit blog posts with rich text formatting and image uploads.

# Project Overview

This project is a Django-based blog application. It allows users to create, view, and edit blog posts with rich text formatting and image uploads.

# Changes Made

The following changes were made to implement image uploads and rich text editing for blog posts:

*   **Model:**
    *   Added an `ImageField` to the `Post` model in `blog/models.py` to store uploaded images.
    *   Changed the `text` field from a `TextField` to a `RichTextField` to allow for rich text editing.
*   **Database:**
    *   Installed the `Pillow` library to handle image processing.
    *   Installed the `django-ckeditor` library to provide a rich text editor.
    *   Created and applied database migrations to add the new `image` column and modify the `text` column in the `blog_post` table.
*   **Forms:** Updated `blog/forms.py` to include the `image` field in the `PostForm`.
*   **Templates:**
    *   Modified `blog/templates/blog/post_detail.html` to display the uploaded image and render rich text content.
    *   Updated `blog/templates/blog/post_edit.html` to support file uploads by adding `enctype="multipart/form-data"` to the form and `{{ form.media }}` to load the rich text editor.
    *   Added a link to the edit page in `blog/templates/blog/post_detail.html`.
    *   Updated `blog/templates/blog/post_list.html` to display images and a longer preview of the rich text content.
*   **Settings:**
    *   Configured `mysite/settings.py` to handle media files by adding `MEDIA_URL` and `MEDIA_ROOT`.
    *   Added `ckeditor` and `ckeditor_uploader` to the `INSTALLED_APPS`.
    *   Configured the `CKEDITOR_UPLOAD_PATH` and `CKEDITOR_CONFIGS` for the rich text editor.
*   **URLs:**
    *   Updated `mysite/urls.py` to serve media files during development and include the `ckeditor_uploader` URLs.
    *   Added a URL pattern in `blog/urls.py` for editing posts.
*   **Views:**
    *   Modified the `post_new` and created the `post_edit` views in `blog/views.py` to handle image uploads (`request.FILES`).
    *   Updated the `post_list` view to sort posts in descending order.
*   **Static Files:**
    *   Ran `collectstatic` to gather the necessary static files for the rich text editor.
    *   Added CSS to `blog/static/css/blog.css` to style the images and add a border to each blog post.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/UlrikeDetective)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/UlrikeDetective)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
