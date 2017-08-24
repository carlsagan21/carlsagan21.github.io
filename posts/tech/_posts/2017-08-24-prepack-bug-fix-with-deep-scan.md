---
layout: post
title: DeepScan 으로 Prepack 버그 잡기
---

javascript static analysis 툴을 찾다가 [DeepScan](https://deepscan.io/) 을 발견해서 적용해 보았다.

대상은 Prepack. [fork](https://github.com/carlsagan21/soo-prepack)를 떠논 것에 적용해 보았다.

[결과](https://deepscan.io/dashboard/#view=project&pid=385&bid=594)로 나온 것 중 명백한 것들만 골라내 [PR](https://github.com/facebook/prepack/pull/928)을 보냈다.

사족: DeepScan 이 나쁘지 않은 것 같아 적용을 여기저기 해보려다, 이 사람들이 코드를 암호화해서 저장하는지 여부가 궁금해져서 물어봤는데, 암호화는 안한다고 한다. 커머셜 코드를 돌리기에는 불안한 듯. 로컬 바이너리로 제공해주면 될거같은데.
