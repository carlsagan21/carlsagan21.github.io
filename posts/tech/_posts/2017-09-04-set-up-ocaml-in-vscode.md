---
title: Vscode 에 Ocaml 개발환경 세팅하기 01
---

## Ocaml, Opam 설치와 Vscode 연동

현존하는 에디터 중 Ocaml을 가장 잘 지원하는 에디터는 Emacs 이다.

하지만 Emacs 는 동시에 그 뭐랄까... 까다로움으로 정평이 나 있는데, 다음 그림을 보면 한번에 이해할 수 있다.

![editor learning curve](https://i.stack.imgur.com/phbAr.png)

터미널 지향적인 vi 와 emacs 에 약하고 Atom, Intellij 등 GUI 기반 에디터/IDE 에 익숙한 사람이, Ocaml 만을 위해서 emacs 를 익힌다는 것은 배보다 배꼽이 더 큰 상황이다.

그런 이유로 GUI 환경에서 개발을 하고 싶어 여러 에디터를 검토한 끝에, Visual studio code 가 Ocaml 에 한해서는 최적의 환경이라고 판단되어 개발 환경을 세팅해 보았고 성공한 것 같아 공유한다.

왜 vscode 가 최상인가? Ocaml 개발환경 세팅의 꽃은 에디터와 merlin 의 연동이다. merlin 은 Ocaml 개발에서 빼놓을 수 없는 필수 패키지로, 이용해야만 Ocaml 개발을 할 수 있는 정도이다. 따라서 merlin 지원이 가장 원활한 Vscode 가 선택되었다.

세팅 순서는 다음과 같다.

### 1. opam 설치

<https://opam.ocaml.org/doc/Install.html>

opam 은 ocaml 패키지 매니저이자 ocaml 환경을 관리하는 패키지이다. 완전히 같진 않지만 nodejs 로 치면 `npm, yarn`, 루비로 치면 `gem`, python 으로 치면 `pip` 에 해당된다. 링크로 들어가 따라하여 설치해주자. 나는 맥을 사용하므로

```sh
brew install opam
```

으로 간단하게 설치하였다. 개인적으로 ocaml 을 windows 에서 사용해 본 적은 없는데, 들리는 말로는 좀... 지원이 잘되지 않는 다는 말도 있다. 하지만 내가 해본 적은 없어서 단언은 할 수 없다. 그래도 가능하면 Linux (또는 Mac) 환경을 추천하는 바이다.

opam 을 설치하면, opam 의 의존성[^1]에 의해 ocaml 도 자동으로 brew 에 의해 설치된다. 설치된 것을 확인하려면 `which -a ocaml` 을 해보자. `/usr/local/bin/ocaml` 이 나온다면 성공이다.

그 다음으로 터미널 메세지가 말하는대로

```sh
opam init
```

을 해야한다. `opam init` 은 `~/.opam` 이라는 opam 의 상태를 관리하는 디렉토리를 생성한다. `opam switch` 등으로 생성된 컴파일러 버젼들이 이 안에 세팅된다.

여튼 또 터미널이 시키는 대로

```sh
1. To configure OPAM in the current shell session, you need to run:

      eval `opam config env`

2. To correctly configure OPAM for subsequent use, add the following
   line to your profile file (for instance ~/.bash_profile):

      . /Users/soo/.opam/opam-init/init.sh > /dev/null 2> /dev/null || true

3. To avoid issues related to non-system installations of `ocamlfind`
   add the following lines to ~/.ocamlinit (create it if necessary):

      let () =
        try Topdirs.dir_directory (Sys.getenv "OCAML_TOPLEVEL_PATH")
        with Not_found -> ()
      ;;
```

을 자동이든 수동이든 설정해주자. 터미널 메세지를 꼼꼼히 읽어보기를 추천한다.

이제 opam 세팅이 끝났다. `opam switch` 를 치면 다음과 같이 나와야 한다.

```sh
--     -- 3.07    Official 3.07 release
--     -- 3.08.0  Official 3.08.0 release
--     -- 3.08.1  Official 3.08.1 release
--     -- 3.08.2  Official 3.08.2 release
--     -- 3.08.3  Official 3.08.3 release
--     -- 3.08.4  Official 3.08.4 release
--     -- 3.09.0  Official 3.09.0 release
--     -- 3.09.1  Official 3.09.1 release
--     -- 3.09.2  Official 3.09.2 release
--     -- 3.09.3  Official 3.09.3 release
--     -- 3.10.0  Official 3.10.0 release
--     -- 3.10.1  Official 3.10.1 release
--     -- 3.10.2  Official 3.10.2 release
--     -- 3.11.0  Official 3.11.0 release
--     -- 3.11.1  Official 3.11.1 release
--     -- 3.11.2  Official 3.11.2 release
--     -- 3.12.0  Official 3.12.0 release
--     -- 3.12.1  Official 3.12.1 release
--     -- 4.00.0  Official 4.00.0 release
--     -- 4.00.1  Official 4.00.1 release
--     -- 4.01.0  Official 4.01.0 release
--     -- 4.02.0  Official 4.02.0 release
--     -- 4.02.1  Official 4.02.1 release
--     -- 4.02.2  Official 4.02.2 release
--     -- 4.02.3  Official 4.02.3 release
--     -- 4.03.0  Official 4.03.0 release
--     -- 4.04.0  Official 4.04.0 release
--     -- 4.04.1  Official 4.04.1 release
--     -- 4.04.2  Official 4.04.2 release
--     -- 4.05.0  Official 4.05.0 release
system  C system  System compiler (4.05.0)
# 247 more patched or experimental compilers, use '--all' to show

# C 는 current, I 는 installed, -- 은 not-installed 이다. opam help switch 참고.
```

### 2. ocaml 설치

앞에서 opam 을 설치하면서 의존성에 의해 ocaml 도 설치되었다고 말헀다. 하지만 우리는 그 ocaml 바이너리를 직접 사용하지는 않을 것이다. brew 가 아니라 opam 이 앞으로 ocaml 바이너리를 관리하게 한다.

`opam` 은 패키지 매니저이면서 동시에 ocaml 컴파일러/인터프리터 매니저이기도 하다. node 생태계로 치면 npm 과 nvm 이 결합된 형태이다. 패키지 다운로드는 나중에 해보고, 프로그래밍 언어 수업에서 사용할 컴파일러부터 세팅해보자.

`opam switch` 는 컴파일러 매니저 기능을 수행하며, alias-of 라는 옵션을 통해 버전을 네이밍 할 수 있도록 해준다.

```sh
opam switch program-language --alias-of=4.05.0
```

이 커맨드는 4.05.0 버전 ocaml 을, program-language 라는 이름으로 사용하게 해준다. alias 를 사용하는 이유는 다음과 같은 경우도 수행하고 싶기 떄문이다.

```sh
opam switch sparrow --alias-of=4.05.0
```

이 커맨드는 4.05.0 버전, 즉 이미 설치된 program-langauge 와 동일한 버전의 컴파일러를, sparrow 라는 이름으로도 설치하여 서로 환경을 분리하도록 해준다. 버전만으로 ocaml 을 설치했다면 이렇게는 할 수 없다.

여기까지 했으면 ocaml, opam 개발환경 세팅은 거의 다 한 것이다. 이 과정은 에디터와 독립적이었으므로, 어떤 에디터를 사용하든 여기까지는 필수적이라 하겠다.

### 3. merlin, ocp-indent 설치

merlin 과 ocp-indent 는 기본적으로 opam 패키지로, 에디터에서의 개발을 크게 도와주는 도구이다. merlin 은 타입추론부터.. 아무튼 모든 것들을 해주고, ocp-indent 는 소스 인덴트를 자동으로 해준다. 설치해보자.

우리는 프로젝트별로 opam switch 로 환경을 격리했기 때문에, 기본적으로 그 환경으로 진입하여 설치해 주어야 한다.

```sh
opam switch program-language
eval `opam config env`
```

여기서 `eval opam config env` 은 switch 를 할 때마다 해줘야 하는 것으로, `opam config env` 만 해서 실상을 보면 shell 의 PATH 등 글로벌 변수를 바꿔주는 역할을 하는 것을 알 수 있다.

여튼 이 상태에서 merlin 과 ocp-indent 를 설치하자.

```sh
opam install merlin
opam install ocp-indent
```

`opam list` 로 잘 설치했는지 확인해보자.

### 4. vscode 와 연결하기

vscode 의 마켓플레이스에서 ocaml 을 검색하여 설치하자.
<https://marketplace.visualstudio.com/items?itemName=hackwaly.ocaml>

그리고 여태껏 프로젝트 디렉토리를 안만들고 진행해 왔는데, 하나 만들어보자. `~/Document/GitHub/program-langauge` 라던가.

디렉토리 구조는 취향이지만, 개인적으로는 src/ 디렉토리를 만들어 그 안에서 작성하길 추천한다. 프로젝트 루트에는 .gitignore, .merlin(프로젝트별 merlin 설정), _oasis(oasis 라는 ocaml 빌드 헬프 툴), opam(프로젝트별 opam 의존성 설정) 등 파일들이 깔릴 예정이기 때문이다.

src/main.ml 을 만들고 간단한 코드를 써보자.

```sh
let rec merge (l1, l2) =
  (*let rec merge ((l1: int list), (l2: int list)): int list =*)
  match l1, l2 with
  | [], l2 -> l2
  | l1, [] -> l1
  | h1 :: t1, h2 :: t2 ->
    if h1 >= h2
    then h1 :: merge (t1, l2)
    else h2 :: merge (l1, t2);;
```

이 때 merge 등 토큰 위에 마우스를 호버하거나 커서를 이동했을때, 타입이 뜨면 merlin 이 제대로 연결된 것이다. merge 함수의 경우 호버하면 `type _ = 'a list * 'a list -> 'a list` 가 나와야 한다.ocp-indent 의 경우 확장프로그램 설정에서 lintOnChange, lintOnSave 를 취향에 따라 설정하면 편하게 쓸 수 있다.

혹시 잘 안되면 vscode 를 껐다 켜보는 것도 추천한다. 그래도 잘 안될 경우, ocaml 확장프로그램의 설정에서 path 가 제대로 잡혀있는지 확인해보고 건드려보자.

여기까지 왔으면 기본적인 세팅은 완료된 것이다! 이 다음 글에서는

- .vscode/launch.json, .vscode/tasks.json 을 이용한 breakpoint 디버거(메모리를 실행상황에서 보는 기능)와 빌드 스크립트화
- .merlin 및 oasis 세팅을 통한 대규모 패키지용 개발환경 구축 (sparrow가 oasis 를 이용한다. flow 는 oasis 비슷한걸 커스텀하게 만들어 쓰는거 같다)
- oUnit 을 이용한 ocaml unit test 환경 / 프언 과제용 예시 테스트 케이스 만들기

을 해보겠다.

[^1]: v. 1.2.2 기준 `aspcud, camlp4, clasp, gringo, ocaml, ocamlbuild`
