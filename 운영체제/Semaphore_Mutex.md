# 세마포어와 뮤텍스의 차이를 설명하시오.

## 1. 답변

> 뮤텍스(mutex)는 상호 배제(mutual exclustion)의 약자로 하나의 Thread가 접근하는 것을 막아줌.
세마포어는 여러 Thread를 실행하는 환경(멀티 쓰레딩)에서 자원에 대한 접근에 제한을 강제하기 위한 동기화 메커니즘
> 

## 2. 내용

### 교착상태가 발생할 수 있는 4가지 조건

1. 상호배제 : 한 프로세스가 자원을 독점, 다른 프로세스 접근 불가
2. 점유대기 : 특정 프로세스가 점유한 자원을 다른 프로세스가 요청
3. 비선점 : 다른 프로세스의 자원을 강제적으로 가져올 수 업슴
4. 환형대기 : 프로세스가 서로가 서로의 자원을 요구하는 상황

### 경쟁 상태(race condition)

공유 자원을 두 개 이상의 프로세스가 동시에 읽거나 쓰는 상황, 동시에 접근시도할 때 접근의 타이밍이나 순서 등이 결과값에 영향을 줄 수 있다.

### 임계 영역(critical section)

- 공유 자원에 접근할 때 순서 등의 이유로 결과가 달라지는 영역
- 공유 데이터의 일관성을 보장하기 위해 하나의 프로세스/스레드만 진입해서 실행 가능  → mutial exclustion
- lock을 이용해서 해결하자!
<img width="577" alt="스크린샷 2023-01-20 오후 5 05 39" src="https://user-images.githubusercontent.com/37897873/214743099-3b431e1a-a7f0-45b9-9e14-e81859046531.png">

    
    → TestAndSet() : 원래 값을 가져와서 **1로 바꿔주고** 반환 
    

### 뮤텍스

- 동기화 대상 : 1개
- Mutual Exclustion : Critical Section을 가지는 쓰레드들이 Running time이 서로 겹치지 않도록 하는 기법
- Lock / UnLock
    - 자원을 점유하고 있는 대상이 Lock을 할 수 있는 권한을 가지고 있어서 자원을 점유하기 시작할 때 들어가서 Lock을 걸어버림. 이렇게 되면 다른 대상들은 Unlock 상태가 될 때까지 기다렸다가 나중에 해당 공유 자원에 접근
- 공유 자원을 사용하기 전에 설정하고 사용한 후에 잠금. 사용한 후에 해제하는 잠금. 하나의 상태만 가짐
- **Key**에 해당하는 (하나의) 오브젝트가 있고, 이를 소유한 쓰레드/프로세스만이 공유자원에 접근 가능
- 대기큐를 생성해두고 임계영역에 스레드가 있으면 다른스레드가 공유자원을 사용하려 한다면 스레드를 blocking하고 대기큐에 sleep 시킨다.


![2](https://user-images.githubusercontent.com/37897873/214743181-ab8314b3-09d7-4524-bea8-a2d416cff2c8.png)

### 스핀락(spin lock)

while 루프를 탈출할 때까지 반복해서 계속 시도, 락을 취득하기 위해 계속 시도하는 방법

- 단점 : Busy waiting, CPU cycle을 낭비
- 개선 : mutex 등장! 락 취득을 계속 확인하는 것이 아닌, 락이 준비되면 나를 깨워달라고 요청하고 대기/상태로 전환하는 것!
- 언제 스핀락 사용?
    1. context switching 시간이 더 짧은가? 
    = 대기실까지 이동하는 시간보다 식당에 진입하게 될 시간이 더 짧은가
    2. 멀티코어 프로세스인가 ?
    = 직원이 여러명인가?
    3. 임계영역에 머무르는 시간이 짧아서 wakeup 시간보다 더 짧을때 → 큐에 넣고 빼는 시간

### 세마포어

- 동기화 대상 : 1개 이상
- 멀티프로그래밍 환경에서 다수의 프로세스나 스레드의 여러개의 공유자원에 대한 접근을 제한하는 방법
- 일반화된 뮤텍스
- 간단한 정수값과 두가지 함수로 wait(P함수), signal(V함수)로 공유 자원에 대한 접근을 처리.
- 공통으로 관리하는 하나의 값을 이용해 상호 배제를 달성
- 현재 공유자원에 접근할 수 있는 쓰레드/프로세스의 수를 나타내는 값을 두어 상호배제를 달성
![3](https://user-images.githubusercontent.com/37897873/214743222-4dd0e726-7fe2-4137-94a1-14c967b13239.png)


### 모니터

- 세마포어를 실제로 구현한 프로그램, 동기화 고급 기법
- 둘 이상의 스레드나 프로세스가 공유자원에 안전하게 접근하도록 공유자원을 숨기고 해당 접근 인터페이스만 제공
- 상호배제가 자동임

### 정리
> 출처 : [우테코 유튜브](https://youtu.be/oazGbhBCOfU)
<img width="842" alt="4" src="https://user-images.githubusercontent.com/37897873/214743234-13167f3d-36a6-4096-a680-05ed3587e99c.png">




## 3. 추가 질문

### 세마포어와 뮤텍스의 관계

- 세마포어 → 뮤텍스 (O) / 뮤텍스 → 세마포어 (X)
- 세마포어는 1개 이상 자원을 소유할 수 있으므로 뮤텍스가 될 수 있지만, 뮤텍스는 1개만 소유하므로 세마포어 될 수 없음

### 자원 소유의 여부

- 뮤텍스 : 자원소유 가능, 상태가 0/1 뿐이므로 Lock 가질 수 있음
- 세마포어 : 자원소유 불가

### 해제하는 스레드

- 뮤텍스 : 소유하고 있는 스레드만 해제가능
- 세마포어 : 소유하지 않은 스레드가 세마포어 해제 가능

## 4. 참고 자료

- [https://heeonii.tistory.com/14](https://heeonii.tistory.com/14)
- [https://medium.com/@kwoncharles/뮤텍스-mutex-와-세마포어-semaphore-의-차이-de6078d3c453](https://medium.com/@kwoncharles/%EB%AE%A4%ED%85%8D%EC%8A%A4-mutex-%EC%99%80-%EC%84%B8%EB%A7%88%ED%8F%AC%EC%96%B4-semaphore-%EC%9D%98-%EC%B0%A8%EC%9D%B4-de6078d3c453)(https://medium.com/@kwoncharles/%EB%AE%A4%ED%85%8D%EC%8A%A4-mutex-%EC%99%80-%EC%84%B8%EB%A7%88%ED%8F%AC%EC%96%B4-semaphore-%EC%9D%98-%EC%B0%A8%EC%9D%B4-de6078d3c453)
- [https://l-o-g-a-n.github.io/posts/semaphore-mutex/](https://l-o-g-a-n.github.io/posts/semaphore-mutex/)
- [https://blog.naver.com/ding-dong/222626631093](https://blog.naver.com/ding-dong/222626631093)
- [https://www.youtube.com/watch?v=gTkvX2Awj6g&t=1130s](https://www.youtube.com/watch?v=gTkvX2Awj6g&t=1130s)
