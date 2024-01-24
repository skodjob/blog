# skodjob.io

This repository contains blog pods that are published automatically to our [blog](skodjob.io).

## How To

#### Follow the structure
All pods are structured in [posts](posts) directory.

#### Use images
Images are better than thousands of words.
It would be great to use as many images as you can and to use them properly, all ahs to be saved in [_images](_images) directory.
Images are also automatically sync with our website, so you don't need to worry about uploading them somewhere.

#### Fill-up author name and category
Here is an exmaple how the very basic blog post should look like with author and catagories defined. 
For more examples see already existing blog posts.

```markdown
---
title: New post
menu_order: 666
post_status: publish
post_excerpt: Short description what a reader should looking for
featured_image: _images/path_to_preview_image.jpg
author: your_name
taxonomy:
    category:
        - automation

---

## New post
The best blog post ever!
```
