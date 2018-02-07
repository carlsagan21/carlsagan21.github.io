---
layout: post
title: 맥을 위한 텐서플로우 커스텀빌드
---

맥에서 `pip` 을 이용하여 텐서플로우를 받아서 사용하면 다음과 같은 에러가 뜬다.([링크](https://www.tensorflow.org/install/install_mac))

```
>> sess = tf.Session()
 ... : W tensorflow/core/platform/cpu_feature_guard.cc:45] The TensorFlow library wasn't compiled to use SSE4.2 instructions, but these are available on your machine and could speed up CPU computations.
 ... : W tensorflow/core/platform/cpu_feature_guard.cc:45] The TensorFlow library wasn't compiled to use AVX instructions, but these are available on your machine and could speed up CPU computations.
 ... : W tensorflow/core/platform/cpu_feature_guard.cc:45] The TensorFlow library wasn't compiled to use AVX2 instructions, but these are available on your machine and could speed up CPU computations.
 ... : W tensorflow/core/platform/cpu_feature_guard.cc:45] The TensorFlow library wasn't compiled to use FMA instructions, but these are available on your machine and could speed up CPU computations.
```

이 컴퓨터에서 해당 인스트럭션 셋, 이 경우에는 [SSE4.2](https://ko.wikipedia.org/wiki/SSE4#SSE4.2), [AVX](https://en.wikipedia.org/wiki/Advanced_Vector_Extensions), [AVX2](https://en.wikipedia.org/wiki/Advanced_Vector_Extensions#Advanced_Vector_Extensions_2), [FMA](https://ko.wikipedia.org/wiki/FMA_%EB%AA%85%EB%A0%B9%EC%96%B4_%EC%A7%91%ED%95%A9) 를 지원하지만 현재의 텐서플로우 빌드는 사용하지 못하고 있다는 뜻이다. 이 인스트럭션 셋의 구체적인 스펙은 링크를 참조.

따라서, 워닝이 뜨는 것도 당연한게 우리가 `pip` 으로 받은 텐서플로우는 범용으로 빌드된 것이기 때문에 내 컴퓨터에 최적화되어 있지 못하다. 따라서 더 나은 퍼포먼스를 위해서 해당 인스트럭션을 사용하도록 커스텀 빌드하면 보다 빨라질 여지가 있다.

내가 쓰는 맥북의 CPU스팩은 다음과 같다.

```
MacBook Pro (Retina, 13-inch, Early 2015)
2.7 GHz Intel Core i5 // 브로드웰
```

각 인스트럭션 셋의 세부를 살펴보면,

1. SSE4.2: 네할렘(1세대) 부터 적용되므로 브로드웰에서 사용 가능하다.
2. AVX1,2: AVX2도 하스웰부터 가능하므로 다음 세대인 브로드웰에서 가능하다.
3. FMA: 인텔에서는 하스웰부터 FMA3를 지원했다. 따라서 가능하다.

와 같으므로 워닝은 적합한 것으로 보인다. 따라서 커스텀 빌드하여 이 워닝을 없애고 퍼포먼스를 최적화하기로 하였다.

---

스텍오버플로에서 가장 많이들 참조하는 답변은 [이거](https://stackoverflow.com/questions/41293077/how-to-compile-tensorflow-with-sse4-2-and-avx-instructions) 인 것 같다. 이 방법도 좋지만, 내 머신에서 돌아갈 텐서플로우를 컴파일 할 때는 아래와 같이 더 간편한 방법으로 가능하다. 따라해보자.

[소스부터 텐서플로우 설치하기](https://www.tensorflow.org/install/install_sources)를 찬찬히 따라하면 할 수 있다.

소스부터 받아오자. `git clone https://github.com/tensorflow/tensorflow` 으로 할 수 있다. 그다음 `cd tensorflow` 로 디렉토리에 들어가, branch 를 원하는 버전으로 바꾸자. 1.2 의 경우, `git checkout r1.2` 로 된다. 브랜치의 목록은 깃헙에서 확인해보면 된다. 이걸 하지 않으면 현재 마스터 소스로 빌드되는데, 그것보다는 안정화된 버전을 쓰는 것이 나을것으로 생각한다.

빌드툴인 [`bazel`](https://bazel.build/) 을 설치하자. `brew install bazel` 로 충분하다. 빌드시 사용할 파이선으로 기본 파이선이 되어있는지 확인하자. 나는 개인적으로 `pipenv` 로 환경을 만드는 걸 좋아해 텐서플로우 빌드 환경을 만들었다.

설치 디펜던시 모듈들을 받아주자. `sudo pip install six numpy wheel` 를 하면된다. 이 시점에서 이미 기본 `python` 과 `pip` 이 환경 내에 생성한 것인지 확인해주는 것이 좋다.

그 다음 소스 루트 디렉토리에서 `./configure`를 실행하면 다음과 같은 화면이 나온다.

```
$ ./configure
Please specify the location of python. [Default is /Users/ ... /bin/python]:
Found possible Python library paths:
  /Users/ ... /lib/python2.7/site-packages
Please input the desired Python library path to use.  Default is [/Users/ ... /lib/python2.7/site-packages]

Using python library path: /Users/ ... /lib/python2.7/site-packages
Do you wish to build TensorFlow with MKL support? [y/N]
No MKL support will be enabled for TensorFlow
Please specify optimization flags to use during compilation when bazel option "--config=opt" is specified [Default is -march=native]:
Do you wish to build TensorFlow with Google Cloud Platform support? [y/N]
No Google Cloud Platform support will be enabled for TensorFlow
Do you wish to build TensorFlow with Hadoop File System support? [y/N]
No Hadoop File System support will be enabled for TensorFlow
Do you wish to build TensorFlow with the XLA just-in-time compiler (experimental)? [y/N]
No XLA support will be enabled for TensorFlow
Do you wish to build TensorFlow with VERBS support? [y/N]
No VERBS support will be enabled for TensorFlow
Do you wish to build TensorFlow with OpenCL support? [y/N]
No OpenCL support will be enabled for TensorFlow
Do you wish to build TensorFlow with CUDA support? [y/N]
No CUDA support will be enabled for TensorFlow
INFO: Starting clean (this may take a while). Consider using --async if the clean takes more than several minutes.

Configuration finished
```

여기서 특별히 옵션을 고를게 있는게 아니라면 엔터를 연타하면 된다. 인스트럭션 셋 옵션은 `--config=opt` 를 고르는 부분에서 이루어지게 되는데, `-march=native` 옵션은 현 기기를 기준으로 이 옵션을 세팅해주므로(gcc 에도 있는 옵션임) 그대로 엔터만 치면 된다.

그 다음 `bazel build --config=opt //tensorflow/tools/pip_package:build_pip_package` 를 해보자. 내 경우 약 50분 가까이 소요되어서 빌드가 완료되었다.

그리고 `bazel-bin/tensorflow/tools/pip_package/build_pip_package /tmp/tensorflow_pkg` 를 하면 .whl 파일이 생성된다. 혹시 whl 파일을 다시 쓸 계획이라면 어딘가 옮겨두자. `/tmp`는 언제 삭제될지 모르기 때문에.

그 다음 자신이 텐서플로우를 설치할 `pip` 을 활성화하여, `sudo pip install ${YOUR_WHL_FILE.whl}` 을 실행하면 마무리된다. 내 경우는 `sudo` 가 필요하였다.

제대로 되었는지는 다음과 같이 테스트해보자.

```
import tensorflow as tf
hello = tf.constant('Hello, TensorFlow!')
sess = tf.Session()
print(sess.run(hello))
```

`tf.Session()` 를 했을 때 워닝이 뜨지 않는다면 성공한 것이다.

이상 맥북에서 텐서플로우 커스텀 빌드하는 법을 정리해 보았다. 두 가지의 퍼포먼스 비교는 MNIST 나 요즘 내가 다루는 seq2seq 을 이용해서 나중에 해보겠다.

---

커스텀 빌드 버젼과 배포 버전을 나이브하게 한번씩 MNIST 를 돌려보았다. 코드는 [이것](https://github.com/tensorflow/tensorflow/blob/r1.2/tensorflow/examples/tutorials/mnist/mnist_deep.py)을 사용했다. 커스텀으로 한번 돌리고, 텐서플로우 패키지만 바꿔서 설치하여 다시 돌려보았다. 그 결과

```
// 커스텀 빌드
test accuracy 0.992

real	31m38.759s
user	94m55.585s
sys	9m9.918s

// 배포판
test accuracy 0.9928

real	54m53.825s
user	174m54.532s
sys	13m58.391s
```

깜짝 놀란만큼 큰 차이가 났다. 거짓말 좀 보태면 거의 두 배 정도이다. 사실 얼마 차이가 나지 않을까봐 걱정이었는데, 의외이다. 정말로 어느정도 차이가 나는지는 몇번을 해서 평균을 비교해야 할텐데, 내 맥북의 건강을 위해서 이쯤에서 끝낸다. 여튼 이득이 있어보인다!

```
// 간단한 DNN 으로 테스트
// 커스텀
real  0m9.949s
user  0m9.926s
sys 0m1.053s

// 배포판
real  0m9.987s
user  0m9.977s
sys 0m1.047s
```
