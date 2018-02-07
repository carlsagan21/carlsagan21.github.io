---
layout: post
title: Github page 에 CloudFront 적용하기
---

내가 따로 글을 쓸 것도 없다. 아래 링크를 정독하면 된다.
<http://strd6.com/2016/02/github-pages-custom-domain-with-ssltls/>

주의점1. Cloudfront 와 Route53 을 동시에 만져야하고 반영되는데 시간도 좀 걸리므로(몇분에서 몇십분) 실험하기가 어렵다. 가이드를 믿고 바로 반영이 안되더라도 천천히 기다려보자.

주의점2.

> You need to be careful with your urls (you’re careful with them anyway, right?!). You must include the trailing slash like `https://danielx.net/editor/`, because if you don’t and do `https://danielx.net/editor` GitHub will respond with a 301 Redirect to your .github.io domain, and it won’t even keep the https!
