---
layout: post
title: 글줄을 띄어쓰기에서만 끊기
---

개인적으로 자간이 일정하지 않은 양쪽 정렬을 싫어한다. 그래서 왼쪽 정렬을 주로 쓰는데, 왼쪽 정렬도 한글의 경우 단어가 중간에 끊기는 편집이 많다. 웹에서도 이는 마찬가지이다.

단어 도중에 끊더라도 대강의 오른쪽에서의 정렬을 맞추는게 가독성에 유리한지, 아니면 철저하게 단어 단위로 끊는게 가독성에 유리한지는 모르겠다. 그래도 선호하는 방식인 글줄이 띄어쓰기에서만 끊기도록 해보자.

놀랍게도, W3스펙에서 한글을 위한 이 기능을 제공한다!

[W3 스펙](https://www.w3.org/TR/css-text-3/#word-break)에 따르면 다음과 같다.

> **‘keep-all’**  
> Implicit soft wrap opportunities between letters are suppressed, i.e. breaks are prohibited between pairs of letters (including those explicitly allowed by ‘line-break’) except where opportunities exist due to dictionary-based breaking. Otherwise this option is equivalent to ‘normal’. In this style, sequences of CJK characters do not break.
>
> > This is sometimes seen in Korean (which uses spaces between words), and is also useful for mixed-script text where CJK snippets are mixed into another language that uses spaces for separation.

요약하면, `word-break: keep-all;`은 한글에서 단어 단위의 줄 변경을 위한 기능이다. 한글의 한국어 용법 이외에 CJK 문자 및 언어, 그러니까 일본어와 중국어는 띄어쓰기를 하지 않기 때문. 공식 스펙에 있는 기능이니 잘 쓰도록 하자.
