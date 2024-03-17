---
layout: post
title:  "Weak Message Authentication Implementation of SHA1"
summary: "Learn how to perform a message/length extension attack"
author: 0x0m03ii
date: '2024-03-15 10:35:23 +0530'
category: ['encryption', 'guides']
tags: encryption
thumbnail: /assets/img/posts/sha1.jpg
keywords: encryption, hacking, security, mac, weak encryption, weak, sha1, sha, hashing
usemathjax: false
permalink: /blog/length-extension-attack/
---

## Adding Multiple Categories in Posts

To add categories in blog posts all you have to do is add a **category** key with category values in frontmatter of the post :

```yml
---
category: ['encryption', 'guides']
---
```

Then to render this category using link and pages. All we need to do is,

1. Create a new file with [your_category_name].md inside categories folder.

2. Copy categories/sample_category.md file and replace the content in [your_category_name].md in that. (Please don't copy the code below its just sample, since it renders the jekyll syntax dynamically)

```jsx
---
layout: page
title: Guides
permalink: /blog/categories/your_category_name/
---

<h5> Posts by Category : {{ page.title }} </h5>

<div class="card">
{% for post in site.categories.your_category_name %}
 <li class="category-posts"><span>{{ post.date | date_to_string }}</span> &nbsp; <a href="{{ post.url }}">{{ post.title }}</a></li>
{% endfor %}
</div>
```

Using the category, all the posts associated with the category will be listed on
`http://localhost:4000/blog/categories/your_category_name`