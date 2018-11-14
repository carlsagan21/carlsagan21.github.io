---
title: 그런 REST API로 괜찮은가 정리
---

슬라이드: <https://slides.com/eungjun/rest#/7>
데뷰 영상: <https://tv.naver.com/v/2292653>

## 강연자의 정리

- 오늘날 대부분의 "REST API"는 사실 REST를 따르지 않고 있다.
- REST의 제약조건 중에서 특히 **Self-descriptive**와 **HATEOAS**를 잘 만족하지 못한다.
- REST는 **긴 시간에 걸쳐(수십년) 진화**하는 웹 애플리케이션을 위한 것이다.
- REST를 따를 것인지는 API를 설계하는 이들이 스스로 판단하여 결정해야한다.
- REST를 따르겠다면, Self-descriptive와 HATEOAS를 만족시켜야한다.
  - Self-descriptive는 **custom  media type**이나 **profile link relation** 등으로 만족시킬 수 있다.
  - HATEOAS는 HTTP 헤더나 본문에 **링크**를 담아 만족시킬 수 있다.
- REST를 따르지 않겠다면, "REST를 만족하지 않는 REST API"를 뭐라고 부를지 결정해야 할 것이다.
  - HTTP API라고 부를 수도 있고
  - 그냥 이대로 REST API라고 부를 수도 있다. ~~(roy가 싫어합니다)~~

---

## REST 를 구성하는 스타일

client-server
stateless
cache
uniform interface
layered system
code-on-demand

## Uniform interface

현재의 API는 조건 중 self-descriptive messages, HATESOAS 를 지키지 못하고 있다.

## Self descriptive message

메세지만 보고 해석할 수 있어야

왜? 확장 가능한 커뮤니케이션. 서버와 클라이언트가 바뀌어도, 항상 해석 가능함.
이게 없으면 API문서를 항상 만들어야 함. API만 보고 의미를 알 수 없기 때문이다.

### 방법1: 미디어 타입 을 IANA에 등록하기. 매번 정의하는게 번거로울 수 있음

<https://www.iana.org/assignments/media-types/media-types.xhtml>

### 방법2: Profile 의미가 뭔지 정보가 담긴 문서를 링크를 달기

```http
Link: <https://example.org/docs/todos>; rel="profile"
```

#### 단점

1. 클라이언트가 Link헤더와 profile을 이해해야 한다
2. 그리고 Content negotitation을 할 수 없다. 컨텐트 타입이 미디어 타입으로 정의된 것이 아니라 링크의 프로필로 정의된다. 따라서 서버 입장에서 이 컨텐트를 클라이언트가 지원하는지 알기 어렵다.
    <https://developer.mozilla.org/ko/docs/Web/HTTP/Content_negotiation>

## HATEOS

하이퍼링크를 통한 상태 이동(전이)이 일어나야
링크가 없으면 상태 전이가 가능하지 않다.

왜? HATEOS는 애플리케이션 상태 전이의 late binding을 가능하게 하기 때문에 필요하다.
어디서 어디로 전이가 가능한지 미리 결정되지 않는다. 어떤 상태로 전이가 완료되고 나서야 그 다음 전이될 수 있는 상태가 결정된다. 링크는 동적으로 변경될 수 있다. 언제나 서버가 마음대로 바꿀 수 있음.

### 방법1

link를 넣어서 보냄. 대신 무엇이 링크인지를 표현하는, 링크를 표현하는 방법을 정의해야 한다. Link헤더에 프로필로.
또는 정의된 미디어 타입을 활용할 수도 있다. JSON API, HAL, UBER, Siren, Collection+json ... 대신 이에 맞추기 위해서 기존 API를 많이 고쳐야 함.

### 방법2

HTTP 헤더에 Location, Link이 있으니 잘 활용하는 것도 방법.

```http
Location: /todos/1
Link: </todos/>; rel="collection"
```

데이터, 헤더 모두 사용해서 표현해도 된다.

## 그래서 어디까지 해야하나

의도한 저자가 이해할 수 있다면(사내 API 등) IANA에 등록할 필요는 없다. 하지만 등록하면 모두가 사용가능해진다.

무시하고 해도 된다. 마이크로소프트의 API 가이드라인.
<https://github.com/Microsoft/api-guidelines/blob/vNext/Guidelines.md>

---

## Related materials

Webcast: Pragmatic REST: The Next Generation
<https://www.slideshare.net/apigee/webcast-pragmatic-rest-the-next-generation>
<https://www.youtube.com/watch?v=kTmqc7Cnqlw>

### about Service-oriented vs Data-oriented

링크를 프로시져로 보지 말고, 자료에 대한 primary key로 보자. 그러면 각 요청에 대한 200, 400 같은걸 따로 만들 필요가 없다. 그냥 자료의 식별자로 GET, POST같은 표준 요청을 하면 됨.

Data oriented에서, Collection이나 Cursor를 받아올 때가 문제일 수 있는데, link를 이용한 pagenation도 고려해야 한다.
Etag를 body에 넣는 이유? 헤더에 넣어도 되지만, collection 을 받아왔을때 각 요소에 대한 etag를 알고 싶을 수 있다.
또 바디에 mimeType을 넣기도 한다. 구글 드라이브 API 참조.

## Query url vs Permelink url

Query도 좋은데, 변한다. (약간 이해 안됨)

## Versioning

하는 명확한 이유가 없다.
그리고 동일 자료에 대해 여러 API를 허용하는 것 자체가 문제가 될 가능성이 높다.
micro service를 깨버리리 가능성이 높다.
클라이언트가 minor change에 견디도록 하라. 새로운 타입 추가, 새로운 프로퍼티 추가에도 동작하도록.
