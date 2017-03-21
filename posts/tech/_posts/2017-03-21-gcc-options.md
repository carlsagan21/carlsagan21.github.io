---
layout: post
title: 리눅스 커널 빌드 및 artik flash를 위한 gcc options
---

-Wall: warning flag all

-Wextra: optional warnings

-Idir: include dir to look for headers.  
일반적으로 C 소스에서 "AAA.h" 라고 하면 현재 작업디렉토리에서 AAA.h 를 찾고 <BBB.h> 라고 하면 컴파일러가 참조하는 디렉토리를 탐색해서 해당파일을 찾습니다. -v 옵션에서 보았듯이 gcc 컴파일러는 헤더파일의 디렉토리를 /usr/include, /usr/local/include, /usr/lib/gcc-lib/i386-redhat-linux/3.2.2/include 로 부터 탐색합니다. 그러나 실무에서는 특정디렉토리에 회사 고유의 헤더파일을 만들어 보관합니다. 이럴때 컴파일러로 하여금 특정 디렉토리를 기존의 디렉토리와 함께 탐색할수 있게 해 주는 옵션이 -I 입니다.  
<http://egloos.zum.com/sunnmoon/v/1825047>  
리눅스 커널을 artik을 위해 빌드할때 I를 써야하는 이유!

-v: verbose. show from where the includes came.
