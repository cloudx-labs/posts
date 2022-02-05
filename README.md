# cloudx-labs/posts

This repository holds the content being posted to our dev.to organization. Given that the process of publishing is automated, the directory structure of where each user places their content is standardized and mandatory.

## Directory structure

Every content creator owns their specific portion of the [posts](posts) directory.
Within this directory is where all articles and assets are to be placed.

The **mandatory** part is that you must create your own directory within `posts` named exactly as your Github username.

How you decide to organize the contents within your directory is up to you. However, we do recommend the following structure:

```text
cloudx-labs/posts/
├── README.md
├── ...
├── posts
│   └──<your github username>
│      ├── one-article.md
│      ├── another-article.md
│      ├── ...
│      └── assets
│           ├── cover1.png
│           └── cover2.png
│           └── ...
```

## Getting started

1. Contact Pablo Romeo or Axel Navarro on Slack to get invited into the DEV.to Organization and to set up your API key.
2. Read the [Dev.to - Editor Guide](https://dev.to/p/editor_guide) to get insights on what formatting is supported and the recommended sizes for cover and content images.
3. Create a feature branch, start writing your first draft and open a pull-request once you want to get some feedback from reviewers :D

## Important considerations

### Images

Articles published by the automated process support referencing your content such as images using **relative** paths. 

So for including images from your `assets` directory you'd have something similar to:

```md
![My text](./assets/path-to-my-image.png)
```

### Custom Header

Besides the usual Markdown, articles that are published automatically must contain a Header called [Front Matter](https://jekyllrb.com/docs/front-matter/).

Such as:

```yaml
---
title: The title of your article
published: false
description: A description for your article
tags: 'one-tag, another-tag'
cover_image: ./assets/path-to-cover.png
---
```

You can see an actual published article with the header [here](./posts/pabloromeo/1.docker-multi-arch.md).

The most important attribute in the header is `published`. It determines if your article is still a draft or if it should be made public.

Therefore, if you open a pull-request with:

- `published: false`: Once merged the article will be available as a **Draft** in DEV.to for you to review online.
- `published: true`: Once merged your article is **public**.

Once the PR is merged, additional properties will be added automatically into the Front Matter header, such `id` and `date`:

```yaml
id: 123456
date: '2022-02-02T18:50:58Z'
```

> **IMPORTANT**: Don't manually edit those two properties, you could lose content or create duplicates.
