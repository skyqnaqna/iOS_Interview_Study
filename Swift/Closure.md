
# 답변

> 클로저는 일정 기능을 하는 코드를 하나의 블록으로 모아둔 것이며 어떤 상수나 변수의 참조를 캡쳐해 저장할 수 있습니다.
> 

# 내용

- 함수형 프로그래밍 패러다임을 접할 때 꼭 알아야할 클로저
- 다른언어의 람다와 비슷
- 클로저 방식 : 기본 클로저 표현 / 후행 클로저 표현
- 함수는 클로저의 한 형태
    - Named Closure : func 키워드 붙인함수 → 일반적으로 함수라고 부르지만 클로저의 일종
    - Unnamed Closure : 익명 함수 (이름을 붙이지 않고 사용) → 보통 클로저로 불림
- 클로저는 참조타입, Heap에 존재

### 클로저의 3가지 형태

- 전역 함수 : 이름 O, 어떤 값도 캡쳐 X
- 중첩 함수 : 이름 O, 값 캡쳐 O
- 클로저 표현 : 이름 X, 문맥(context)으로 값을 캡쳐 O ( + 경량화 된 문법 )

<img width="415" alt="%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-12-04_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_12 45 24" src="https://user-images.githubusercontent.com/37897873/206861120-c85b3c17-dcab-44b1-9f4e-c140b865047e.png">


### 클로저 표현

예시로 `sorted(by:)` 에 어떤 방법으로 정렬을 수행할 것인지 전달인자로 클로저를 넣으면 해당 정렬로 배열 얻을 수 있다.

1. 기본 함수 : 매개변수, 리턴타입 모두 존재하는 함수를 sorted의 매개변수로 전달

```swift
func backward(_ s1: String,_ s2: String) -> Bool {
	return s1 > s2
}

var reversedNames = names.sorted(by: backward)
```

2. 인라인 클로저 표현 : 매개변수에 클로저 자체를 전달한다.

```swift
// 일반적인 클로저 형태
{  (parameters) -> return type in 
			//in 뒤에 body가 온다.
			실행코드
}
```

```swift
reversedNames = names.sorted(by: { (_ s1: String,_ s2: String) -> Bool in {
	return s1 > s2
}) 
//()소괄호도 생략 가능
```

3. 클로저 표현 간소화 방법
- 문맥에서 인자타입(parameter type)과 반환 타입(return type)을 생략하여 추론한다.

```swift
reversedNames = names.sorted(by: { (s1, s2) in {
	return s1 > s2
}) 
```

- 축약된 인자 이름 : s1, s2를 지우고 첫번째 전달인자부터 $0, $1, $2 … 순서로 표현할 수 있다. in 키워드도 사용할 필요 없어진다.

```swift
reversedNames = names.sorted {
	return $0 > $1
}
```

- 암시적 반환 표현 : 이전 코드에서 return 문까지 생략해서 표현할 수 있다.

```swift
let reversed: [String] = named.sorted { $0 > $1 } 
```

- 연산자 함수 : 전달한 클로저와 동일한 조건의 제공 함수가 있으면 사용할 수 있다. 즉 연산자 함수를 클로저의 역할로 사용.

```swift
let reversed: [String] = named.sorted(by: >) 
```

### 후위 클로저 (Trailing Clousers)

1. 함수의 마지막 파라미터가 클로저일 때
2. 파라미터 값 형식이 아닌 함수 뒤에 붙여 작성하는 문법이다. (꼬리처럼 덧붙여서)
3. 이때, ArgumentLabel은 생략됨

예를 들어, 매개변수도 리턴타입도 없는 클로저를 전달한다면 이렇게 축약할 수도 있다.

```swift
//1.
func someFunction(closure: () -> Void) {
		
}

//2. clouser라는 ArgumentLabel 생략, 호출구문인 ()도 생략
func someFunction {
		
}
```

파라미터가 여러개인 함수라면?

```swift
func fetchData(success: () -> (), fail: () -> ()) {
    //do something...
}
```

- 인라인 클로저 표현

```swift
//인라인 클로저 표현
fetchData(success: { () -> () in
    print("Success!")
}, fail: { () -> () in
    print("Fail!")
})
```

- 트레일링 클로저 표현하여 마지막 인자인 fail 클로저를 함수 뒤에 붙여서 사용할 수 있다.

```swift
fetchData(success:  { () -> () in
    print("Success!")
}) { () -> () in
    print("Fail!")
}
```
    

## 값 캡쳐

클로저는 주변 문맥을 통해 상수나 변수를 획득(Capture) 할수 있다.

값 캡쳐시 Value/Reference 타입 상관없이 Reference Capture 한다.

즉, num은 Int 타입의 구조체 형식이라 Value 타입이기에 값을 복사해서 저장해야 하지만, 클로저는 Value 타입이든 Reference 타입이든 상관없이 캡쳐하는 값을 참조한다

```swift
func doSomething() {
    var num: Int = 0
    print("num check #1 = \(num)")
    
    let closure = {
        print("num check #3 = \(num)") //외부 변수 num을 참조
    }
    
    num = 20
    print("num check #2 = \(num)")
    closure()
}

//결과
//num check #1 = 0
//num check #2 = 20
//num check #3 = 20
```

이 때 함수 내부의 clouser만 떼놓으면 num이 어디있는지 알 수 없어 동작할 수 없는 형태가 된다. 그래서 앞의 변수를 참조하여 획득 후에 실행하는 것이다.

```swift
let closure = {
    print("num check #3 = \(num)") //외부 변수 num을 참조
}
```

## 클로저 캡쳐 리스트

일반적인 Closure는 참조타입이므로 Reference Type을 캡쳐하지만, Value Type으로 캡쳐하고 싶다면?

Value Capture 하고 싶은 변수를 리스트로 명시해서 사용할 수 있다. 

- [ 캡쳐 리스트 ] in : `let closure = { [s1, s2] in`

```swift
func doSomething() {
    var num: Int = 0
    print("num check #1 = \(num)")
     
    let closure = { [num] in //캡쳐 리스트를 사용해 값 타입으로 캡쳐하므로
        print("num check #3 = \(num)") //num은 0임
    }
    
    num = 20
    print("num check #2 = \(num)")
    closure()
}

//결과
//num check #1 = 0
//num check #2 = 20
//num check #3 = 0
```

- 이때 캡쳐 리스트들은 상수로 캡쳐되기 때문에 Value Capture 된 값을 변경할 수 없다. ( 변경하고 싶다면 Value → Reference 타입으로 변경시키는 `inout` 키워드 붙여서 사용)

```swift
let closure = { [num] in //캡쳐 리스트를 사용해 값 타입으로 캡쳐하므로
    num = 2 //error!
}

func sayHello(name: inout String) {
    name = "Banana"
}
 
var name: String = "Apple"
sayHello(name: &name)
print(name) //Banana
```

## @escaping

함수의 전달인자로 전달한 클로저가 함수 종료 후에도 호출될 때 클로저가 함수를 탈출(Escape)한다고 표현한다.

명시하지 않는다면, 기본적으로 함수 내에서 종료되는 `@non-escaping` 클로저 이다.

예를 들어, 3초 뒤에 클로저를 실행하고 싶다면 이렇게 작성하면 에러가 발생한다. 함수가 종료되고선 클로저를 실행할 수 없기 때문인데 함수의 흐름을 벗어나서 호출되기 때문이다.

```swift
func delayFunction(completion: () -> ()) {
    DispatchQueue.main.asyncAfter(deadline: .now() + 3) {
        completion()
    }
}
```

그래서, 함수의 실행 흐름과 상관없이 실행되는 클로저! 라고 `@escaping` 키워드를 붙여 알려주므로써 실행할 수 있다. → func delayFunction(completion: `@escaping` () -> ()) {

- 예시)

```swift
//비탈출
func nonescape(closure: () -> Void) {
		closure()
}

//탈출
func escape(completionHandler: @escaping () -> Void) -> (() -> Void) {
		return completionHandler
}

class SomeClass {
    var x = 10
    
    func test1() {
        nonescape { x = 200 }
    }
    
    func test2() -> (() -> Void) {
        return escape { self.x = 300 }
    }
}

let instance = SomeClass()
instance.test1()
print("test1", instance.x)

let returnClosure: () -> () = instance.test2()
returnClosure()
print("test2", instance.x)

//결과
//test1 200
//test2 300
```

즉, 함수 내부에서만 동작되고 종료되지만 @escaping 키워드를 붙이면 함수 생애주기와 별개로 그 이후에 해당 작업이 실행된다!

# 추가 질문

### 클로저는 1급 객체인데 특징은 무엇인가요?

1. 변수나 데이터에 할당 할 수 있어야 한다.
2. 객체의 인자로 넘길 수 있어야 한다.
3. 객체의 리턴값으로 리턴 할수 있어야 한다.

### @autoclosure는 무엇인가요?

> 파라미터로 전달된 일반구문과 함수를 **클로저로 래핑(Wrapping)** 한 것이다.
> 

`@autoclosure` 키워드를 붙이면 실제론 클로저를 전달받지 않지만, 클로저처럼 사용가능하다.

바로 실행되어야 할 구문이 **지연되어 실행**한다는 특징이 있다. 클로저가 호출되어야만 비로소 실행됨

- 해당 예시에서 updateNames()가 실행되기 전까지는 (당연히) names.removeFirst()가 실행되지 않는다.

```swift
var names = ["Kim", "Park", "Lee"]

func updateNames(closure: () -> String) {
  print("Updated names array with \(closure())")
}

names
// ["Kim", "Park", "Lee"]

updateNames() {
  return names.removeFirst() //클로저를 받아 print문 실행
}

names
// ["Park", "Lee"]
```

위를 더 짧게 한다면 updatesNames(names.removeFirst()) 이렇게 줄 수 있으나 에러가 발생한다.

1. 함수 호출 전에 이미 names.removeFirst() 가 실행 후 
2. 결과인 String이 updatesNames() 함수의 인자로 전달되려 하면 
3. `() → String` ≠ `String` 타입이 맞지 않아 오류가 발생된다. 

그래서 @autoclosure로 구문을 클로저로 감싸고 지연되어 실행되는 것이다. 

- `assert` 함수도 @autoclosure 형태로 구현되어 있다.
    
    ```swift
    func assert(condition: @autoclosure () -> Bool, 
    	_ message: @autoclosure () -> String = default, 
    	file: StaticString = default, 
    	line: UWord = default)
    ```
    
- 참고로 기본적으로 `@nonescape` 속성을 부여한다. 없애고 싶다면 이렇게 수정해야 함
    
    ```swift
    func updateNames(@autoclosure(escaping) closure: () -> String) {
      print("Updated names array with \(closure())")
    }
    ```
    

# 참고 자료

- [https://jusung.gitbook.io/the-swift-language-guide/language-guide/07-closures](https://jusung.gitbook.io/the-swift-language-guide/language-guide/07-closures)
- [https://babbab2.tistory.com/82?category=828998](https://babbab2.tistory.com/82?category=828998)
- @autoclouser
    - [https://jusung.github.io/AutoClosure/](https://jusung.github.io/AutoClosure/)
    - [http://seorenn.blogspot.com/2016/04/swift-autoclosure.html](http://seorenn.blogspot.com/2016/04/swift-autoclosure.html)