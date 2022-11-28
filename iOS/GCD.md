## 답변

> GCD (Grand Central Dispatch)는 직접적으로 쓰레드를 관리하지 않고, “Queue(대기열)”에 보내기만 하면 알아서 분배해주는 방식입니다. 메인 쓰레드에 몰린 작업을 다른 스레드에서도 동시에 작업하여 같은 작업을 여러개 처리해야 할 때 효과적입니다. (동시성 프로그래밍)
> 

## 내용

- 시스템에서 알아서 쓰레드 숫자를 관리한다.
- 쓰레드 보다 높은 차원/레벨 에서 일을 한다
- 주로 다른 쓰레드에서 오래걸리는 작업들이(보통 네트워크 처리) 비동기적으로 동작 하도록 만들어줍니다.

(GCD를 이해할 때 정리했던 동기/비동기, 동시성/직렬성 개념이 선행되어야 한다.)

### DispatchQueue

- GCD에서 사용하는 queue 이름이, DispatchQueue (dispatch 뜻 자체가 ‘보내다’ 이므로 큐에 보내는 단역할이군!)
- URLSession에서 네트워크 처리 후 받은 데이터를 UI로 처리할 때 주로 사용한 코드일 것이다. 이 때 task는 하나의 작업 단위이기 때문에, 내부에서는 순차적으로 동작된다!

```swift
//큐에 보낼거야.메인큐에.비동기적으로
DispatchQueue.main.async {
  //task (작업의 한 단위)
}
```

- 끝나는 시점은 completionHandler를 통해 해당 시점을 알 수 있다.

## GCD의 종류

### MainQueue

- 메인쓰레드(1번쓰레드)에서 동작되며 종류는 단 한개.
- 주로 네트워킹 처리가 끝나고 UI 작업을 할 때 메인큐로 넘겨준다.
- Serial(직렬)

```swift
// 메인큐
let mainQueue = DispatchQueue.main

timeCheck {
    task1()
}

// 위와 아래의 코드는 (의미상)같다.
//timeCheck{
//    mainQueue.sync {
//        task1()
//    }
//}
```

### GlobalQueue

- 6가지 종류로 qos(quailty of service)를 사용해 작업의 중요도를 결정할 수 있다.
- Concurrent(동시)


### ❓ 왜 qos가 필요하지
예를 들어, 느리게 처리를 해도 되는 작업에 빨리 처리되는 큐가 사용된다면 자원이 낭비될 것이다. 
한정된 자원과 에너지를 효율적으로 사용하기 위해 작업을 나누어 처리하는 qos가 필요하다!

### 종류 (중요도 순서)

1. Dispatch.global(qos: `.userInteractive`) : 거의 즉시 동시에, 매우 중요한 작업! 
    - 유저와 직접 interactive : UI업데이트, 애니메이션, UI의 대부분
2. Dispatch.global(qos: `.userInitiated`) : 몇초
    - 유저가 즉시 필요하긴 하지만, 비동기적으로 처리된 작업(ex. 앱 내에 pdf 파일을 여는)
3. Dispatch.global`()` : 디폴트, 일반적인 작업
4. Dispatch.global(qos: `.utility`) : 몇초에서 몇분
    - 보통 인디케이터랑 같이 실행되는 긴 작업, 계산, I/O, Networking …
5. Dispatch.global(qos: `.background`) : 몇초에서 몇분
    - 시간이 중요하지 않은(유저가 직접 인지하지 않고) 작업, 데이터 미리 받아오기, DB유지
6. Dispatch.global(qos: `.unspecified`) : legacy API시 사용 
    - 사용하지 말아야한다고 문서에 명시

```swift
let userInteractiveQueue = DispatchQueue.global(qos: .userInteractive)
let userInitiatedQueue = DispatchQueue.global(qos: .userInitiated)
let defaultQueue = DispatchQueue.global()  // 디폴트 글로벌큐
let utilityQueue = DispatchQueue.global(qos: .utility)
let backgroundQueue = DispatchQueue.global(qos: .background)
let unspecifiedQueue = DispatchQueue.global(qos: .unspecified)

timeCheck {

    defaultQueue.async {
        task1()
    }

    defaultQueue.async(qos: .userInitiated) { //작업의 qos를 높게 보냄 
        task2()
    }

    defaultQueue.async {
        task3()
    }
}

/* 
결과 : 시작은 순서대로나 끝나는 건 알 수 없음!
Task 1 시작
Task 2 시작
Task 3 시작
Task 2 완료~
Task 3 완료~
Task 1 완료~
*/
```

### ❓ 중요도에 따라서 다른 큐에 보내는게 무슨 의미지
iOS가 알아서 중요도를 인지하여(추론하여) 높은 qos일 수록, 쓰레드에 우선순위를 매겨 더 여러개의 쓰레드를 배치하고 배터리도 더 많이 사용하도록 한다.


### Private(custom)Queue

- 기본 설정은 Serial이지만 Concurrent로 설정 가능하며, Qos도 커스텀으로 설정할 수 있다. 사용 방식은 뒤에 레이블을 붙이면 된다.
- 둘다 가능 : Serial(디폴트), Concurrent(설정 가능)

```swift
let queue = DispatchQueue(label: "jiinQueue") 
privateQueue.async {
    task1()
}

privateQueue.async {
    task2()
}

privateQueue.async {
    task3()
}

/* 
결과 : Private 큐는 기본이 Serial이기 때문에 비동기적으로 보냈지만, 순차적으로 시작-끝
Task 1 시작
Task 1 완료~
Task 2 시작
Task 2 완료~
Task 3 시작
Task 3 완료~
*/

//Concurrency로 설정도 가능하다!
let queue = DispatchQueue(label: "jiinQueue", qos: .utility, attributes: .concurrency) 
```

## 추가 질문

### 생성한 큐의 qos보다 작업을 보낼 때 더 높은 수준으로 보내면 어떻게 되나요?

```swift
let queue = DispatchQueue.global(qos: .background) //생성한 큐는 5순위
queue.async(qos: .utility) { //작업을 보낼 때는 4순위로
	//
}
queue.async() { //작업을 보낼 때는 4순위로
	//
}
queue.async(qos: .utility) { //작업을 보낼 때는 4순위로
	//
}
```

- 큐의 품질이 작업의 영향을 받아 상승한다!

```swift
let queue = DispatchQueue.global(qos: .utility) //background -> utility
```

## 참고 자료

- 공식문서 : [https://developer.apple.com/library/archive/documentation/Performance/Conceptual/EnergyGuide-iOS/PrioritizeWorkWithQoS.html#//apple_ref/doc/uid/TP40015243-CH39-SW1](https://developer.apple.com/library/archive/documentation/Performance/Conceptual/EnergyGuide-iOS/PrioritizeWorkWithQoS.html#//apple_ref/doc/uid/TP40015243-CH39-SW1)
- 강의 : [https://www.inflearn.com/course/iOS-Concurrency-GCD-Operation/dashboard](https://www.inflearn.com/course/iOS-Concurrency-GCD-Operation/dashboard)
- 블로그(정리굿!) : [https://sujinnaljin.medium.com/ios-차근차근-시작하는-gcd-grand-dispatch-queue-1-397db16d0305](https://sujinnaljin.medium.com/ios-%EC%B0%A8%EA%B7%BC%EC%B0%A8%EA%B7%BC-%EC%8B%9C%EC%9E%91%ED%95%98%EB%8A%94-gcd-grand-dispatch-queue-1-397db16d0305)
