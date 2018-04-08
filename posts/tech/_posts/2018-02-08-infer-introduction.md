---
title: Infer 소개 영상
---

Finding inter-procedural bugs at scale with Infer static analyzer

Infer라는 정적 분석 도구의 fosdem2018 소개 영상. 원리에 대한 설명이라기보다 infer가 지금 할 수 있는 것들에 대한 간략한 소개.
Diff기반으로 분석해서 분석 시간을 줄인 점이 인상깊음.
또 facebook의 ci 프로세스가 얼핏 드러남. Performance 분석을 정적분석/리뷰 이후에 하는구나

—-

Infer is a static analysis tool - if you give Infer some Java or C/C++/Objective-C code it produces a list of potential bugs. Anyone can use Infer to intercept critical bugs before they have shipped to users, and help prevent crashes or poor performance. Infer has been successfully deployed at Facebook, where it identifies hundreds of potential bugs per month in mobile apps and backend code.
This talk will present Infer, and how it can be used to help developers move fast and with more confidence.

<https://fosdem.org/2018/schedule/event/code_finding_inter_procedural_bugs_at_scale_with_infer_static_analyzer/>
