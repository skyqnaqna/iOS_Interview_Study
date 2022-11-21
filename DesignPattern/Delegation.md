# Delegation 패턴은 무엇인가요.


# 답변

> 어떤 객체가 처리해야할 일을 다른 객체에게 위임하여 대신 처리하도록 하는 패턴입니다.
> 

# 내용

## 구성 및 역할

- **위임자**
    - 대리자가 필요한 위임자 객체
    - 위임자가 해야 할 일을 대리자에게 요청
    - 대리자는 보통 위임자가 대리자를 유지할 때 **Retain Cycle**을 피하기 위해 **약한 속성(`weak`)**으로 유지됩니다.
        - ex) UITableView의 delegate 프로퍼티
            
            ```swift
            weak var delegate: UITableViewDelegate? { get set }
            ```
            
- **delegate protocol**
    - 대리자가 수행할(구현할) 메서드 정의한 것으로 **대리자가 준수해야할 프로토콜**이다.
    - 위임자가 대리자에게 맡길 일을 적어둔 것으로 대리자는 이걸 보고 해야 할 일을 알 수 있다.
    - 위임자는 delegate protocol 타입의 delegate 변수를 가진다. → delegate protocol을 준수하는 대리자를 위임한다는 뜻
- **대리자**
    - delegate protocol을 구현하는 객체
    - 위임자가 요청한 일을 **대신 처리하는 역할**

## 언제, 왜 사용하는걸까?

1. 애플 프레임워크 중 자주 사용하는 `UIKit`의 컴포넌트들이 `delegate`와 `dataSource`라는 이름의 속성을 가지고 있는데, 모두 Delegation Pattern을 따른다. → 애플 프레임워크에선 일반적이고 자주 사용되는 패턴
    - 왜냐하면 우리는 `UIKit`의 내부 구현을 볼 수 없고 제공되는 API대로 사용하는데, 객체간 데이터를 제공하거나 행위를 요청하기 때문에 필요하다.
    
2. 클래스를 분리하거나 **재사용 가능한 컴포넌트**를 만들기 위해 사용한다.

<img src=https://user-images.githubusercontent.com/31722496/203113243-d19290df-109d-4055-a6c5-aea2e4ae5485.png>

- 만약 우리가 `UITableView`를 직접 만들었다고 가정해보자.
- `A`객체와 `B`객체는 `UITableView` 를 사용하기 위해 각 객체가 `UITableView` 객체를 갖고있는 상태다.
- `UITableView`에는 어떤 행이 선택된 것을 감지하여 작업을 수행하는 함수가 있다. → `[func tableView(UITableView, didSelectRowAt: IndexPath)](https://developer.apple.com/documentation/uikit/uitableviewdelegate/1614877-tableview)`
    - 이 함수가 처리할 작업이 `A`와 `B`에서 서로 다를 가능성이 높다.
    - 따라서 `UITableView`는 `A`타입 객체인지 `B` 타입 객체체인지에 따라 처리할 작업을 분류해야 한다.
    - 즉, `UITableView`가 `A`와 `B`의 존재를 알아야한다는 것이다.
        
		<img src=https://user-images.githubusercontent.com/31722496/203113211-4af07f9e-0f03-400d-8872-ea7b6083ddd3.png>
        
    - `UITableView`를 사용하는 객체가 많아질수록 점점 방대해지고 복잡해지는 문제가 생길 것이다.
    - 이런 문제를 해결하기 위해 **Delegation Pattern**을 사용한다.

<img src=https://user-images.githubusercontent.com/31722496/203113219-c49e82ae-111f-47e7-a09c-df325bc2ea16.png>

- Delegation Pattern을 사용하면 `UITableView`가 `A`와 `B`를 몰라도 문제가 없다. → 결합도 낮춤
- 대리자(delegate 프로퍼티)에게 할 일을 넘기면 되기 때문이다.
- 따라서 `UITableView`의 코드를 수정할 필요가 없어지면서 재사용성을 높이게 된다.

3. ViewController간 데이터 전달 시 사용한다.
- 현재 ViewController에서 이전 ViewController에 데이터를 전달하거나 이벤트를 요청할 때 사용

## 주의할 점

- 한 객체가 많은 대리자를 필요로 한다면 너무 많은 일을 하고있다는 증거이므로, 해당 객체를 분해하는 것을 고려해야 한다.
- 한 객체가 여러가지에 대한 대리자 역할을 하게 되는 상황이라면, 정말 해당 객체만 대리자 역할을 할 수 있는지 의심해봐야 한다.
- **Retain Cycle**을 피하기 위해 대부분 delegate 속성은 `weak`이어야 한다.
    - 그래서 delegate protocol에 `class` 키워드를 사용하면 클래스에서만 사용할 수 있게 제한하며, 대리자를 `weak` 키워드로 선언하여 Retain Cycle을 피할 수 있도록 만든다.
    - 하지만 현재 `class`키워드는 앞으로 사라질 것이라 `AnyObject` 키워드를 사용하면 된다.
        
        ```swift
        protocol TapDelegate: class {
            func didTapped()
        }
        ```
        
		<img src=https://user-images.githubusercontent.com/31722496/203113232-5f50ec5a-e762-4e6f-81c5-a5189fb82667.png>
        
        ```swift
        protocol TapDelegate: AnyObject {
            func didTapped()
        }
        ```
        
- 객체에 대리자가 무조건 있어야 한다면 객체 이니셜라이저에 대리자를 추가하고, 옵셔널(`?`) 대신 암시적 추출 옵셔널(`!`)을 사용하여 속성 타입을 표시하는 것도 방법이다. → 대리자를 설정하도록 강요하는 것
    - 또는, 전략 패턴과 같은 다른 디자인 패턴을 사용하는 것도 방법이다.

# 추가 질문

## delegate가 꼭 프로토콜을 사용해야 하나요?

- 꼭 프로토콜을 사용하지 않아도 된다.
    - 프로토콜을 사용하지 않고 클래스 인스턴스 변수를 만들어서 위임할 수 있다.
- 하지만 프로토콜을 사용하면 추상화와 구현을 분리하여 코드 재사용성을 높이고 결합도를 낮출 수 있다는 장점이 있다.

## DataSource?

- DataSource는 데이터와 관련된 작업을 Delegation 패턴으로 그룹화 한 것.
    - ex) tableView의 row 개수, 각 row에 맞는 cell 반환 등
- Delegate는 데이터나 이벤트를 수신하는 메서드를 Delegation 패턴으로 그룹화 한 것.
    - ex) tableView의 row가 선택되었을 때

## Struct도 Retain Cycle이 발생하는가?

- 구조체는 값 타입이기 때문에 메모리 누수로부터 자유롭다고 생각했지만, Retain Cycle을 피하기 위해 클래스로 제한하여 약한 참조를 하는 것에서 의문이 생겼다.
- Struct와 Class를 함께 사용하면서 문제가 발생할 수 있다. → [https://stackoverflow.com/questions/42632122/retain-cycle-between-class-and-struct](https://stackoverflow.com/questions/42632122/retain-cycle-between-class-and-struct)
    
    ```swift
    struct A {
    	let someClass: B
    }
    
    class B {
    	var someStruct: A?
    }
    
    let b = B()
    let a = A(someClass: b) // 1
    
    b.someStruct = a // 2
    ```
    
    1. a가 b만 가리키고 있음
    
	<img src=https://user-images.githubusercontent.com/31722496/203113236-cac93758-d4aa-4bee-be35-68cc7ad388e2.png>
    
    2. b가 a를 가리키게 되면서 자기 자신을 강한 참조하게 됨 → Retain Cycle
    
	<img src=https://user-images.githubusercontent.com/31722496/203113240-54ac0557-5a5c-4490-95f7-2795530bf740.png>
    
- `weak` 또는 `unowned`를 잘 사용하자!

# 참고 자료

[https://developer.apple.com/documentation/uikit/uitableview](https://developer.apple.com/documentation/uikit/uitableview)

[https://www.amazon.com/Design-Patterns-Tutorials-Learning-patterns/dp/1942878664](https://www.amazon.com/Design-Patterns-Tutorials-Learning-patterns/dp/1942878664)

[https://www.goodreads.com/en/book/show/39336590-swift-design-patterns](https://www.goodreads.com/en/book/show/39336590-swift-design-patterns)
