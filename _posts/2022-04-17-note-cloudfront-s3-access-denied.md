---
layout: post
title: How I fixed an unexpected access denied message on my blog posts
date: 2022-04-17
---

Accessing https://blog.immontilla.dev/posts/2022/04/15/hello-jekyll/ was wrong whereas accessing https://blog.immontilla.dev/posts/2022/04/15/hello-jekyll/index.html was right. The former returned an access denied message while the latter rendered the post as expected.

I found the reason and the solution to fix this in a blog post from <a target="_blank" alt="Specifying a default root object for static website subdirectories on AWS Cloudfront " href="https://stevepapa.com/how-to-specify-a-default-root-object-for-static-website-subdirectories-on-aws-cloudfront/">Steve Papa</a>. 

The reason is a malfunction when an AWS S3 static website is exposed online from an AWS Cloudfront distribution. Regardless you set index.html as the default root object on both AWS services, no web page will be served unless index.html is requested explicitly. Apparently, for these services the only root a website has is the main domain url.

To solve it, as Steve Papa explained, I had to change the Cloudfront distribution origin from the bucket name to the static web hosting endpoint.
