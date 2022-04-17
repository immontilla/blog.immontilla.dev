---
layout: post
title: How to fix an unexpected access denied message received when a visitor click on a post from my blog main page
date: 2022-04-17
---

Everything worked fine. I pushed my blog and its CircleCI workflow passed. However, when I clicked on my "Hello Jekyll" post from my blog main page, and I got an AWS access denied message. Why?

I checked my S3 bucket files and I found the /posts/2022/04/15/hello-jekyll/index.html file as expected. Then, I appended index.html in the browser and I could see my post.

So, visiting https://blog.immontilla.dev/posts/2022/04/15/hello-jekyll/ raised an access denied message whereas visiting https://blog.immontilla.dev/posts/2022/04/15/hello-jekyll/index.html rendered the post.

Digging online, I found the reasons and the trick to fix this issue or, should I say bug?, explained by Steve Papa in <a target="_blank" alt="Specifying a default root object for static website subdirectories on AWS Cloudfront " href="https://stevepapa.com/how-to-specify-a-default-root-object-for-static-website-subdirectories-on-aws-cloudfront/">https://stevepapa.com/how-to-specify-a-default-root-object-for-static-website-subdirectories-on-aws-cloudfront/</a>. 

The reason is a malfunction when an AWS S3 static website is exposed online from an AWS Cloudfront distribution. Regardless you set index.html as the default root object on both AWS services, no index.html is served unless it is requested explicitly.

The trick is _Instead of setting the source from the bucket shown in the dropdown list, you’ll need to grab the static web hosting endpoint for that resource from its S3 settings page and pop it in manually_.
