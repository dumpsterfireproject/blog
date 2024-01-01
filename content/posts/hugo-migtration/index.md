---
title: "Hugo Migration Complete"
date: 2024-01-01
slug: "/hugo-migration"
description: Announcement of the migration to hugo from gatsby
tags:
  - Hugo
---
I had previously migrated this blog to [Gatsby Cloud](https://www.gatsbyjs.com/docs/reference/cloud/what-is-gatsby-cloud/) from WordPress. The process of crafting a blog entry as a markdown file and seamlessly publishing it to Gatsby Cloud by simply pushing it to Github proved to be a significant time-saver compared to managing a WordPress site. In February 2023, Netlify acquired Gatsby and in August, announced the [sunset of Gatsby Cloud](https://www.netlify.com/blog/gatsby-cloud-evolution/). This left me in search of a new platform or home for my blog.

As someone more inclined toward Go programming rather than JavaScript, Hugo has always piqued my interest. When considering alternatives for my Gatsby blog, I stumbled upon [How to convert a simple blog from Gatsby to Hugo](https://dev.to/geekgalgroks/how-to-convert-a-simple-blog-from-gatsby-to-hugo-3jic) and felt compelled to explore Hugo's potential as a replacement for my blog platform. My conversion really took no time at all. I didn't need to change the folder structure for my entries and kept the `content/posts/<slug>/index.md` structure. I did have to change the file extensions from .mdx to .md. Beyond that, the [Hugo Quick Start](https://gohugo.io/getting-started/quick-start/) guide was helpful, as well was [Deploy a Hugo site to Azure Static Web Apps](https://learn.microsoft.com/en-us/azure/static-web-apps/publish-hugo). The [Set up an apex domain in Azure Static Web Apps](https://learn.microsoft.com/en-us/azure/static-web-apps/apex-domain-external#set-up-with-an-a-record) article was the last bit that I needed, and I was up and running. The last snag that I hit was an 'unable to determine the location of the app artifacts' error messsage on the initial deployment. This answer on [Stack Overflow](https://stackoverflow.com/a/74074788) helped me identify that I needed to set `output_location: "public"` in my github action.

I decided to go with the [Anubis](https://themes.gohugo.io/themes/hugo-theme-anubis/) theme. I'm looking forward to getting back to writing more in 2024 and discovering the expanded functionalities that Hugo offers.
