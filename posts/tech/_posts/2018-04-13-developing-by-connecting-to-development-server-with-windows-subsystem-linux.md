---
title: Windows Subsystem Linux로 개발서버에 접속하여 개발하기
---

- 개발서버는 리눅스인데 회사에서 지급한 노트북은 윈도우이고
- 코드 작성 및 빌드는 서버에서 할 것이라면

일반적으로 택하는 것은 putty이다. 하지만 맥이나 리눅스에 비하면 putty의 터미널로서의 기능성과 디자인은 극악이다. 복사 붙여넣기도 예상 밖으로 동작할 때가 많다. 그렇다고 ComEmu나 Cmder는 너무 과다한 것처럼 느껴진다. Cygwin을 깔아야 할 것 같고, Cygwin을 깔면 git bash도 깔리고 등등... 저주받은 의존성을 가진 컴퓨터가 되고 마는 것이다.

그게 싫어서 윈도우10부터 기본 제공되는 windows bash를 써서 개발을 해보았는데 괜찮은 것 같아 공유한다.
windows bash를 설치하는 법은 인터넷에 널려있으니 아무 글이나 읽으시라. 버튼 클릭으로 다 할 수 있다.
<http://sanghaklee.tistory.com/39>
깔고나면 cmd로 킨 창에서 bash를 치면 bash상태로 진입한다.

이제 예쁜 터미널로 bash를 사용하면 된다. [cmder](http://cmder.net/)는 훌륭한 콘솔 에뮬레이터이고 mini 버젼이 존재한다. 즉 Cygwin없이 사용 가능하다. cmder-mini를 받아서 실행해보자.

처음에는 cmd였나 powershell 이었나? 가 기본으로 실행된다. 이걸 setting에서 wsl bash를 기본으로 하도록 설정해주자.

```shell
# settings -> startup -> tasks
# click "+" to make new task
# name: whateveryouwant. mine is "wsl bash"
# set icon if you want
# commands
cmd /c "bash"
```

이렇게 만든 task를 제일 위로 올려주자. 그럼 cmder이 켜질때 bash가 켜진다. 문제는 시작 디렉토리가 linux위치가 아니라는 거다. 이걸 잡기 위해서 다음과 같이 바꿔주자(혹은 startup dir... 을 누르자)

```shell
-new_console:d:C:\Users\{YOUR_USER_NAME}\AppData\Local\lxss\home cmd /c "bash"
# find the path of wsl in your pc and put it.
# if you cannot find, do search linux specific file such as `.bashrc`. It may be in somewhere down "AppData".
```

이러면 bash가 자동으로 실행된다. wsl은 ssh도 같이 오기때문에 ssh를 써서 서버에 붙을 수 있다.
Happy hacking!
