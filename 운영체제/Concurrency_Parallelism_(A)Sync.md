# 동시성/병렬/동기/비동기 각각의 차이는 무엇일까요?	

## 답변

> **동기**는 어떤 작업이 시작하면 끝나기를 기다리는 방식이고, **비동기**는 기다리지 않고 다른 작업을 시작하는 방식으로 서로 상반되는 개념이라고 할 수 있습니다.  
**동시성**은 여러 작업이 물리적으로 동시가 아니라 번갈아가면서 처리되는 것이고, **병렬**은 멀티 코어에서 물리적으로 동시에 여러 작업을 처리하는 것입니다.
>

## 내용

### 동시성 프로그래밍이 필요한 이유?

- 동시성 프로그래밍은 여러 스레드를 이용해서 효율적으로 작업할 수 있습니다. 효율적으로 동작하는다는 것은 **사용성**과 **반응성**이 좋아진다는 것인데요, 사용자 입장에서는 여러 작업이 동시에 실행되는 것으로 느껴지기 때문입니다. 동시성을 사용하지 않으면 사용자는 한 가지 작업만 할 수 있기 때문에 그동안 다른 작업을 못하여 답답함을 느낄 것입니다. 예를 들어 파일을 다운 받는 동안 다른 작업을 못하거나, 이미지를 불러오는 동안 화면이 멈춰 있는 현상, 화면이 뚝뚝 끊기는 현상 등이 있습니다.

### 하드웨어 스레드 vs 소프트웨어 스레드

- 하드웨어 스레드는 **물리적인 코어**를 의미한다
    - 코어는 하나의 스레드 작업을 처리할 수 있다
    - 만약 하이퍼스레딩을 이용하면 하나의 코어에서 두 개의 스레드 작업을 처리할 수 있다
- 소프트웨어 스레드는 **프로세스 내부의 작업 단위**이다
    - 가상으로 만들어진 스레드이기 때문에
    

### 동시성 vs 병렬 프로그래밍

- **동시성 프로그래밍 (Concurrency)**
    - 하나의 코어가 여러 작업을 동시에 처리하는 것 → 물리적으로 동시가 아닌 시분할과 context switching을 통해 번갈아가면서 작업
        - 따라서 하나의 작업이 완료될 때까지 기다리는 것이 아니라 여러 작업을 번갈아가면서 처리하기 때문에 동시성의 목적은 처리하는 작업의 양을 늘리는 것이다. → 응답성 증가 및 CPU 사용량 증가
    - 싱글코어, 멀티코어 상관없다
    - 동시성 프로그래밍과 반대되는 개념은 직렬성 프로그래밍**(Sequential)**이다
        - 직렬성 프로그래밍 : 단 하나의 스레드에서만 작업을 하는 것으로 동시에 작업을 처리하지 못하고, 순서대로 작업을 처리해야한다
- **병렬 프로그래밍 (Parallelism)**
    - **멀티코어**에서 **물리적으로 동시에** 여러 작업을 처리하는 것 → **simultaneously**
        - 하나의 작업(Task)을 분담해서 처리하는 것
        - 병렬처리는 주로 하나의 작업에 대한 처리량을 늘려 연산 속도를 향상시키는 것이 목적이다. → 처리량(Throughput)을 늘리는 것
    - **싱글코어**에서는 **불가능**하다 → 멀티코어에서만 가능
    - OS가 처리해주기 때문에 소프트웨어 개발에서는 직접 구현할 일이 없다
- 동시성과 병렬은 반대되는 개념이 아니다
    - 병렬성(물리)이 동시성(논리)을 만족할 수 있지만, 동시성이 병렬성을 만족시키진 않는다.
    - 여러 가지 일을 멀티 코어에서 담당하는 것은 병렬 프로그래밍이 아니다??
        - 병렬 프로그래밍이 한 가지 일을 여러 코어가 분담해서 하는 것이라면
            - 4가지 일을 4개의 코어가 하나씩 담당하는 것은 병렬 프로그래밍이 아니다
        - 하지만 여러 작업을 여러 CPU(or core)를 사용하여 작업 처리 속도를 줄이는 것을 의미한다면
            - 하나의 작업을 하위 작업으로 나누든, 여러 작업을 돌리든 병렬성이라고 할 수 있다

|동시성|병렬|
|:---:|:---:|
| <img src=https://user-images.githubusercontent.com/31722496/199961449-b2a59165-891d-4bf4-a412-630b3d848264.png> | <img src=https://user-images.githubusercontent.com/31722496/199961510-1f7b7b63-04c2-40ba-8f07-f723f60cd829.png>|
|여러 Task를 처리하는 동시성|하나의 Task를 Subtask로 나눠서 처리하는 병렬성|

<img src=https://user-images.githubusercontent.com/31722496/199961983-83581ec9-2352-40b5-9fad-706a0b5e22fe.png width=70%>

### 동기 vs 비동기

- 동기 : 어떤 작업을 시작하면 끝나기를 기다리는 방식
- 비동기 : 작업이 끝나기를 기다리지 않고 다음 작업을 시작할 수 있는 방식

**실행 종료 시점을 알 수 있는가?**

- 동기 : 작업의 종료를 기다리기 때문에 종료 이후에 할 일을 정할 수 있다
- 비동기 : 언제 작업이 끝나는지 알 수 없다

<img src=https://user-images.githubusercontent.com/31722496/199962407-42c181dc-82a7-48cb-9337-5df8162a3836.png width=70%>

### 동시성, 병렬, 동기, 비동기의 개념은 어떤 상관관계를 가지는가?

- 동기와 비동기는 반대되는 개념이라고 할 수 있다
- 하지만 동시성과 병렬은 반대되는 개념이 아니다
- **동시성**의 반대는 **직렬성**
- 그래서 병렬과 동시성은 동시에 활용할 수 있고, 병렬에서 직렬 또는 싱글에서 동시성을 활용할 수도 있다
- 동시성에서 동기/비동기로 처리하거나, 병렬에서 동기/비동기로 처리가 가능하다

<img src=https://user-images.githubusercontent.com/31722496/199962584-c112bca4-c455-4bfb-98d4-01b1dc037448.png>


||싱글 스레드|멀티 스레드|
|:---:|:---:|:---:|
|동기|<img src=https://user-images.githubusercontent.com/31722496/199962602-ceea559d-6328-4313-bb51-5387201dd04c.png>직렬성|<img src=https://user-images.githubusercontent.com/31722496/199962632-60515601-0790-44a6-ae57-693db05e76e7.png>병렬성|
|비동기|<img src=https://user-images.githubusercontent.com/31722496/199962620-a4d0363e-2608-4bce-8532-6b4d31526d07.png>동시성|<img src=https://user-images.githubusercontent.com/31722496/199962642-5e5029c6-f8db-4080-bb2b-f23d43f120d7.png>동시성, 병렬성|

<img src=https://user-images.githubusercontent.com/31722496/199962683-f5cae1ef-b85d-4954-a319-5287a592ce54.png width=70%>

## 추가 질문

### 인터럽트는 동기인가 비동기인가?

- I/O 관련 인터럽트 같은 경우는 I/O 관련 작업이 완료돼서 인터럽트가 발생하기까지 다른 작업을 하기 때문에 비동기 방식이고,
시스템콜은 작업을 하다가 내가 필요해서 커널에 요청하고 처리가 끝나기까지 기다리기 때문에 동기 방식이다.
- 이 외에도 divide by zero, debug breakpoint 등은 CPU가 해당 명령을 실행하는 과정에서 잠시 멈추게 동작하는 부분이므로 동기식 인터럽트이다.

<img src=https://user-images.githubusercontent.com/31722496/199963697-53a05f7c-22c1-4f81-a11c-b5db0e433eb7.png width=70%>

- **동기식 입출력**
    - 입출력 요청 후 입출력 작업이 완료된 후에 사용자 프로그램에 제어가 넘어감
    - 구현 방법 1
        - 입출력이 끝날 때까지 CPU를 낭비시킴
        - 매시점 하나의 입출력만 일어날 수 있음
    - 구현 방법 2
        - 입출력이 완료될 때까지 해당 프로그램에게서 CPU를 빼앗음
        - 입출력 처리를 기다리는 줄에 그 프로그램을 줄 세움
        - 다른 프로그램에게 CPU를 줌
    - 보통 읽기 작업에 적합
        - ex) 디스크에서 어떤 데이터를 읽어온 후 작업을 수행하는 경우
- **비동기식 입출력**
    - 입출력이 시작된 후 입출력 작업이 끝나기를 기다리지 않고 사용자 프로그램에 제어가 즉시 넘어감
    - 쓰기 작업에 적합
        - ex) 데이터를 저장/수정하거나 화면에 출력하기 위해 데이터를 보내고나서 다음 작업 진행
- 두 방식 모두 입출력의 완료는 **인터럽트**로 알려준다.
- CPU를 점유하고 있는지와는 상관이 없다
    - **다음 명령어를 바로 실행할 수 있는가**의 차이이다.

### Multiprocessing vs Parallel processing

- Multiprocessing
    - 한 컴퓨터 시스템 내에 CPU가 여러 개 존재
- Parallel processing
    - 프로그램을 더 짧은 시간 안에 실행할 목적으로 하나의 프로그램 명령어를 여러 프로세서(or 코어)로 나누어 처리하는 것

[https://stackoverflow.com/questions/18841095/comparison-between-multiprocessing-and-parallel-processing](https://stackoverflow.com/questions/18841095/comparison-between-multiprocessing-and-parallel-processing)


## 참고 자료

[https://yagom.net/courses/동시성-프로그래밍-concurrency-programming/](https://yagom.net/courses/%eb%8f%99%ec%8b%9c%ec%84%b1-%ed%94%84%eb%a1%9c%ea%b7%b8%eb%9e%98%eb%b0%8d-concurrency-programming/)

[https://www.geeksforgeeks.org/difference-between-concurrency-and-parallelism/](https://www.geeksforgeeks.org/difference-between-concurrency-and-parallelism/)

[https://stackoverflow.com/questions/748175/asynchronous-vs-synchronous-execution-what-is-the-difference/49386140#49386140](https://stackoverflow.com/questions/748175/asynchronous-vs-synchronous-execution-what-is-the-difference/49386140#49386140)

[https://core.ewha.ac.kr/publicview/C0101020140314151238067290?vmode=f](https://core.ewha.ac.kr/publicview/C0101020140314151238067290?vmode=f)