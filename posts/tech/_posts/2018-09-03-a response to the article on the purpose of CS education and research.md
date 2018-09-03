---
title: CS 학부 교육, 연구의 목적에 대한 글의 감상
---

Meyer, B. (2017). Fourteen years of software engineering at ETH Zurich. arXiv preprint arXiv:1712.05078.

이하 본문 일부

### 4.4 Concepts and skills

A central question in teaching programming today, already discussed at some length in the “Software Engineering in the Academy” article [1], is whether to focus on concepts or skills. Teach only computer science concepts, and you produce people who are not ready for the practical software engineering tasks that industry needs filled. Teach only skills, and you produce technicians that do not have the big picture necessary for a successful career.

The easy answer that we should teach both is not enough: education is a fixedpie business in which the total number of coursework hours is set. One has to make choices.

I hit an example of the need to teach more than skills when giving some lectures in 2004 at a renowned Chinese university, which was proud that its software engineering students were at the top. I devised a small questionnaire to find out for myself, and have used it often since. One of the questions, made up almost as a joke (but then the joke was on me) was:

> Your boss gives you the source code of a C compiler and asks you to adapt it so that it will also find out if the program being compiled will not run forever (i.e. it will terminate its execution). Check one:
> 1. Yes, I can, it’s straightforward.
> 2. It i hard, but doable.
> 3. It is not feasible for C, but is feasible for Java.
> 4. It is not feasible for C, but is feasible for Eiffel.
> 5. It cannot be done for any realistic programming language

It is touching to see how many enthusiastic students select one of the first four options (#4 undoubtedly from an unconscious hope that the instructor must be on to something). Now one may argue that ordinary software development does not require dealing with undecidability, but it does not sound right that a competent software engineer should have no idea of the issue. Particularly one coming out of a good university. Electricians may not know Maxwell’s equations; but electrical engineers should.

Skills are critical too. Students do not have to know all the technologies of the moment; in fact they cannot, since they are too many of them, and they come and go anyway (ASP.NET yesterday, Angular JS today, something else next year). But they have to know some, both for their relevance to industry and to make sure software concepts do not remain theoretical, as they would without an actual implementation using the tools of the moment.

The philosophy we adopted was skills supporting concepts. Teach skills so that students get a good grip on the practice (and program a lot), and use these skills to drill the fundamental concepts of programming — [^1] names some 20 of them, from abstraction and information hiding to recursion and invariant-based reasoning — into their heads.

...

## 13 Assessment

Still, one should not lie. The outcomes that matter in research are not numerous publications, best-paper awards, completed PhD theses, keynote invitations, software tools, citations and other measurable signs of progress. I was after real success, in the sense of changing the way the IT industry develops software. That was the only justification for putting in parentheses my career as a technology entrepreneur. When you have an ambitious goal, you should be judged by that goal. Such absurd pieties as “it’s the journey that matters” or “what’s important is to participate” (by, of all people, Coubertin, founder of modern Olympics) are even more wrong in research than anywhere else. Only the result counts. By that standard, the story told in this article is one of glaring, unremitted and probably definitive failure.

---

학부교육은 기술 위주여야 하나? 개념(흔히 말하는 이론) 위주여야 하나?
저자가 이 글에서 이 과목은 여기, 이 과목은 저기라고 선을 긋는 것은 아니지만, 대략적인 경계는 제안하는 듯. Halting Problem은 그 예시 중 하나일 것이고.
"Software Engineering in the Academy" 를 읽으면 보다 명확해질 것 같다.
다만 중국 학교라고 명시할 필요가 있었을까 하는 생각은 드는데.. 기계적인 동양인이라는 편견의 일환일까봐. 내가 과민한 거겠지?

연구에 대한 강력한 의식은 마음에 든다. "changing the way the IT industry develops software". 나의 신조와 같지는 않지만, 공부가 된다. 나의 목표는 무엇이 되어야 하는가?

[^1]: Bertrand Meyer: *Software Engineering in the Academy*, in Computer (IEEE), vol. 34, no. 5, May 2001, pages 28-35.
