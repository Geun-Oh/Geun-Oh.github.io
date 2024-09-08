+++
title = 'V8 Isolate'
date = 2024-09-08T12:11:56+09:00
draft = false
+++

### v8-isolate

[v8 isolate class reference](https://v8.github.io/api/head/classv8_1_1Isolate.html)

isolate은 v8엔진의 일종의 복사본으로, 격리된 환경에서 독립적인 병렬 실행이 가능하도록 도와주는 기능이다.

하나의 프로세스 내에서 여러 작업들이 서로 간섭 없이 실행되도록 한다.

특히 서버리스 환경에서 조금 더 작은 단위의 여러 작업들을 병렬 실행하고자 할 때, 프로세스를 여러 격리된 환경으로 분리하는 것에 탁월하다.

실제 Cloudflare Workers 는 모든 동작에 대해 http call 을 통해 서비스를 제공하는데, 이 경우 v8-isolate를 사용하여 경량 작업들을 하나의 인스턴스에서 병렬로 실행하게 만들어두었다.
[Cloudflare Docs](https://developers.cloudflare.com/workers/reference/how-workers-works/#isolates)
이와 같은 작업을 통해 서버리스 환경에서 한 번의 콜드스타트 만으로 여러 작업들의 병렬 실행이 가능하도록 하였다.

> 이러한 구조를 Cloudflare에서는 Nanoservice라고 부른다.

v8-isolate은 내부적인 구현을 통해 독립적인 heap과 Garbage Collector를 가지는 등의 방법을 통해 하나의 독립 개체를 생성한다.

그러나 이는 물리적으로 격리된 환경(컴퓨팅 자원 차원에서의 격리)까지 제공하지는 못하기 때문에, 정말 독립적인 실행 환경을 위해서는 추가적으로 물리적인 격리가 가능한 환경을 만들어 감싸줄 필요가 있다고 생각했다.

다음 장에서 조금 더 세부적인 구현과, 물리적으로 격리된 isolate 풀을 생성하여 요청을 처리할 수 있도록 환경을 구성해보자.