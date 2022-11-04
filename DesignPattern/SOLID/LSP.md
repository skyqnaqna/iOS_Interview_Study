# Liskov Substitution Principle 리스코프 치환 원칙

> **자식 타입은 부모 타입의 정의를 위반해서는 안된다.**  
→ 하위 타입은 상위 타입에서 가능한 행위를 수행할 수 있어야 한다.  
→ 하위 클래스의 인스턴스는 상위 객체 참조 변수에 대입해 상위 클래스의 인스턴스 역할을 하는데 문제가 없어야 한다.
>
> 리스코프 치환 : **자식 타입 클래스는 언제나 부모 타입 클래스로 바꿀 수 있어야 한다.**
>
> 리스코프 치환 원칙을 지키지 않으면 개방 폐쇄 원칙을 위반하게 되는 것이다.  
→ 기능 확장을 위해 기존의 코드를 수정해야 하므로
>

## 적용방법

1. 여러 객체가 같은 일을 한다면 하나의 클래스로 묶고 이들을 구분할 수 있는 필드 생성
    - ex) 직사각형, 정사각형
2. 같은 연산을 약간씩 다르게 한다면 공통 인터페이스를 만들고 이를 구현
    - ex) 사각형, 원
3. 두 객체가 공통 연산 외에 약간의 차이를 가진다면 상속을 통해 구현
    - ex) 가득 찬 원, 빈 원

## Override 주의사항

> 부모 클래스의 메소드를 Override해서 재정의할 때, LSP가 깨질 정도로 그 양상이 달라진다면
인터페이스를 사용해야 했거나,  
사실은 부모-자식 관계가 아니었던 것이다.
> 

## LSP가 깨지는 예시

<img src=https://user-images.githubusercontent.com/31722496/199669542-5ad75d0e-5666-4686-ad7f-bc71346b8915.png>

<details>
<summary>Code</summary>
<div markdown="1">       

```swift
class Rectangle {
  private var width: Int
  private var height: Int

  init(w: Int, h: Int) {
    self.width = w
    self.height = h
  }

  func getPerimeter() -> Int {
    return 2 * (width + height)
  }

  func setWidth(w: Int) { self.width = w }
  func setHeigth(h: Int) { self.height = h }
}

class Square: Rectangle {
  init(w: Int) {
    super.init(w: w, h: w)
  }

  override func setWidth(w: Int) {
    super.setWidth(w: w)
    super.setHeigth(h: w)
  }

  override func setHeigth(h: Int) {
    super.setHeigth(h: h)
    super.setWidth(w: h)
  }
}

var r = Rectangle(w: 3, h: 5)
print(r.getPerimeter())

let s = Square(w: 3)
print(s.getPerimeter())

r = s
r.setWidth(w: 3)
r.setHeigth(h: 5)

print(r.getPerimeter())
```

</div>
</details>
</br>

<img src=https://user-images.githubusercontent.com/31722496/199669769-2e5396de-b9ef-4525-b41b-8e82d1369c93.png width=70%>

- 마지막에 16을 기대하지만 20이 출력된다. 따라서 Square가 Rectangle을 완벽하게 대체할 수 없다.
- 이를 해결하기 위해서는 다음과 같은 방법이 있을 것이다.
    1. 둘레나 넓이를 구하는 함수를 인터페이스를 사용하여 구현한다.
    2. 상속을 하지 않는다.

## Ex1
<img src=https://user-images.githubusercontent.com/31722496/199670182-c13ccec5-a9a2-485d-a098-ead3c71a2e58.png width=50%>

<details>
<summary>Code</summary>
<div markdown="1">       

```swift
class Vehicle {
  func startEngine() {

  }

  func accelerate() {

  }
}

class Driver {
  func go(vehicel: Vehicle) {
    vehicel.startEngine()
    vehicel.accelerate()
  }
}

class Car: Vehicle {
  override func startEngine() {
    self.engageIgnition()
    super.startEngine()
  }

  private func engageIgnition() {

  }
}

class ElectricBus: Vehicle {

  override func accelerate() {
    self.increaseVoltage()
    self.connectIndividualEngines()
  }

  private func increaseVoltage() {

  }

  private func connectIndividualEngines() {

  }
}
```

</div>
</details>

- Driver는 Car 또는 ElectricBus를 사용할 수 있다.

## Ex2

<details>
<summary>Code</summary>
<div markdown="1">       

```swift
let requestKey: String = "NSURLRequestKey"

// NSError 서브클래스. 추가적인 기능을 제공하지만 원래 기능을 엉망으로 만들진 않는다.
class RequestError: NSError {

    var request: NSURLRequest? {
        return self.userInfo[requestKey] as? NSURLRequest
    }
}

// 데이터를 가져오지 못하면 RequestError를 반환한다.
func fetchData(request: NSURLRequest) -> (data: NSData?, error: RequestError?) {

    let userInfo: [String:Any] = [requestKey : request]

    return (nil, RequestError(domain:"DOMAIN", code:0, userInfo: userInfo))
}

// RequestError가 무엇인지 모르고 실패할 것이며, NSError를 반환한다.
func willReturnObjectOrError() -> (object: AnyObject?, error: NSError?) {

    let request = NSURLRequest()
    let result = fetchData(request: request)

    return (result.data, result.error)
}

let result = willReturnObjectOrError()

// ⚠️ 확인. 이것은 내 관점에서 완벽한 NSError 인스턴스이다.
let error: Int? = result.error?.code

// ⚠️ 하지만 이봐! 이게 무슨 일이죠? RequestError이기도 하다! 대단해!
if let requestError = result.error as? RequestError {
    requestError.request
}
```

</div>
</details>
