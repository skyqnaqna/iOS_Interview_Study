# 디자인패턴과 아키텍처(패턴)의 차이점?

# 답변

> 소프트웨어 **아키텍처**는 소프트웨어 **시스템**의 **기본 구조**이며
>
> **디자인 패턴**은 소프트웨어 **디자인** 과정에서 발생하는 문제에 대한 **일반적이고 재사용 가능한 해결책**입니다.
> 
> **아키텍처 패턴**은 소프트웨어의 **구조적 문제**에 대한 **해결책**입니다.
> 

# 내용

 어떤 **문제**에 대한 **일반적인 해결책**을 제공한다는 점은 같습니다.

차이점이라면 **아키텍쳐**는 **설계단**에서 정하는 **청사진**이고, **디자인 패턴**은 **구현단**에서의 **실제 구현**입니다.

**디자인 패턴**은 문제 해결을 위해 **클래스**를 **구성**하는 **방법**이기 때문입니다.

**아키텍처**는 구성 **요소**가 **시스템**에서 **어떻게** **동작**하고 **위치**하는지 **선택**하는 것이고,

**디자인**은 특정 **요소**의 **구현**과 관련된 **세부사항**을 **결정**합니다.

따라서 모든 **아키텍처**는 **디자인 패턴**이 될 수 **있지만**, 모든 **디자인 패턴**이 **아키텍처**일 수는 **없습니다**.

예를 들어, MVC는 둘 다 가능하지만 싱글톤은 아키텍처 패턴이 될 수 없습니다.


# 디자인 패턴

## 생성 패턴

### 1. Singleton

> 클래스의 인스턴스가 하나만 존재하도록 하면서 전역 접근을 제공하는 디자인 패턴입니다.
> 

### 2. Factory Method

> 객체 생성 인터페이스를 정의하여 서브클래스가 어떤 객체를 생성할 지 결정할 수 있도록 하는 디자인 패턴입니다. (객체 생성을 서브 클래스에서 하도록 미룰 수 있음)
> 

### 3. Abstract Factory

> 구체적인 클래스를 명시하지 않고 관련된 혹은 의존적인 객체들을 생성할 수 있는 인터페이스를 제공합니다.
> 

### 4. Builder

> 복잡한 객체들을 단계별로 생성할 수 있도록 합니다.
> 

### 5. Prototype

> 클래스에 의존시키지 않고 기존 객체들을 복사할 수 있도록 하는 디자인 패턴입니다.
> 

## 구조 패턴

### 1. Adapter (Wrapper)

> 호환되지 않는 객체들이 협업할 수 있는 인터페이스를 제공합니다.
> 

### 2. Bridge

> 대형 클래스 또는 관련된 클래스 집합을 추상화와 구현이라는 두 개의 개별 계층으로 분할하는 디자인 패턴입니다.
> 

### 3. Decorator

> 객체 추가적인 책임을 동적으로 부여하여 상속을 사용하지 않아도 유연한 기능 확장을 가능하게 하는 디자인 패턴입니다.
> 

### 4. Facade

> 라이브러리, 프레임워크 또는 복잡한 클래스 집합에 대한 단순화된 인터페이스를 제공합니다.
> 

### 5. Composite

> 객체를 트리구조로 구성해서 부분-전체 계층구조를 구현하여 개별 객체와 복합 객체를 똑같이 다룰 수 있습니다.
> 

### 6. Flyweight

> 어떤 클래스의 인스턴스 하나로 여러 개의 가상 인스턴스를 제공합니다.
> 

### 7. Proxy

> 특정 객체로의 접근을 제어하는 대리인을 제공합니다.
> 

## 행동 패턴

### 1. Iterator

> 여러 요소를 담고 있는 객체 내부 구조에 대한 이해없이 각 요소를 순서대로 접근할 수 있습니다.
> 

### 2. Command

> 요구사항(요청, 명령)을 객체로 캡슐화시켜 매개변수화, 큐에 넣거나 작업 취소 기능을 지원할 수 있습니다.
> 

### 3. Observer

> 객체간 1:다 의존 관계를 정의하여 관찰 중인 객체에 발생하는 이벤트를 여러 객체에 알림을 보내는 구독 메커니즘을 제공합니다.
> 

### 4. Strategy

> 각 알고리즘을 캡슐화하여 클라이언트가 알고리즘을 바꿔 사용할 수 있도록 합니다.
> 

### 5. Template Method

> 슈퍼 클래스에서 알고리즘의 뼈대를 정의하고 서브 클래스는 이 알고리즘의 구조를 변경하지 않고 일부 내용을 재정의할 수 있습니다.
> 

### 6. State

> 객체의 내부 상태가 변경될 때 객체의 동작을 변경할 수 있도록 합니다.
> 

### 7. Mediator

> 서로 관련된 객체 사이의 복잡한 통신과 제어를 한곳으로 집중시킵니다.
> 

### 8. Chain of Responsibility

> 1개의 요청을 2개 이상의 객체에서 처리해야 할 때 사용합니다.
> 

### 9. Memento

> 객체의 이전 상태를 저장하고 복원할 수 있습니다.
> 

### 10. Visitor

> 다양한 객체에 새로운 기능을 추가해야 하는데 캡슐화가 별로 중요하지 않다면 비지터 객체를 사용해서 모든 복합 객체를 대상으로 원하는 작업을 처리할 수 있습니다.
> 

## 아키텍처 패턴

### 1. Layered

> 하위 모듈들의 그룹으로 나눌 수 있는 구조화된 시스템에서 사용할 수 있습니다. 각 계층은 상위 계층에 서비스를 제공합니다.
> 

### 2. Client-Server

> 클라이언트가 서버에 요청하면 서버는 클라이언트에 서비스를 제공합니다.
> 

### 3. Master-Slave

> 마스터는 슬레이브 컴포넌트들의 작업을 분산하고, 슬레이브가 반환한 결과로부터 최종 결과를 계산합니다.
> 

### 4. Pipe-Filter

> 데이터 스트림을 생성하고 처리하는 시스템에서 사용할 수 있습니다. 각 처리 과정은 필터 컴포넌트에서 이루어지며 처리되는 데이터는 파이프를 통해 흐릅니다.
> 

### 5. Broker

> 분리된 컴포넌트들로 이루어진 분산 시스템에서 사용됩니다. 브로커는 컴포넌트 간의 통신을 조정하는 역할을 합니다.
> 

### 6. Peer-To-Peer

> 피어는 클라이언트와 서버 역할을 모두 할 수 있습니다.
> 

### 7. Event-Bus

> 이벤트 소스, 리스너, 채널, 버스라는 4가지 컴포넌트들을 가집니다. 소스는 버스를 통해 특정 채널로 메시지를 발행하며 리스너는 특정 채널에서 메시지를 구독합니다.
> 

### 8. Model-View-Controller

> 핵심 기능과 데이터를 포함하는 모델, 사용자에게 정보를 표시하는 뷰, 사용자로부터의 입력을 처리하는 컨트롤러로 분리하여 코드의 효율적인 재사용을 가능하게 합니다.
> 

### 9. Blackboard

> 결정 가능한 해결 전략이 알려지지 않은 문제에 유용합니다. 솔류션의 객체를 포함하는 구조화된 전역 메모리인 블랙보드, 자체 표현을 가진 특수 모듈인 지식 소스, 모듈 선택과 설정 및 실행을 담당하는 제어 컴포넌트로 구성됩니다.
> 

### 10. Interpreter

> 특정 언어로 작성된 프로그램을 해석하는 컴포넌트를 설계할 때 사용됩니다.
> 

# 추가 질문

## 소프트웨어 디자인 vs 소프트웨어 아키텍처

- 소프트웨어 아키텍처
    - 시스템의 청사진 역할을 합니다.
    - 시스템의 복잡성을 관리하고 구성 요소 간의 통신 및 조정 메커니즘을 설정하기 위한 추상화를 제공합니다.
    - 아키텍처가 정해지면 소프트웨어 디자인을 수행합니다.
- 소프트웨어 디자인
    - 시스템의 요소와 시스템의 요구 사항을 충족하기 위한 설계 계획을 제공합니다.
    - 각 개별 모듈, 구성 요소를 설계하는 것으로 모듈이 수행하는 것, 클래스, 기능 및 사용 등을 정의하는 것입니다.

# 참고 자료

디자인 패턴

[https://github.com/DovAmir/awesome-design-patterns](https://github.com/DovAmir/awesome-design-patterns)

[https://refactoring.guru/design-patterns](https://refactoring.guru/design-patterns)

[https://en.wikipedia.org/wiki/Software_design_pattern](https://en.wikipedia.org/wiki/Software_design_pattern)

아키텍처 패턴

[https://www.ou.nl/documents/40554/791670/IM0203_03.pdf/30dae517-691e-b3c7-22ed-a55ad27726d6](https://www.ou.nl/documents/40554/791670/IM0203_03.pdf/30dae517-691e-b3c7-22ed-a55ad27726d6)

[https://towardsdatascience.com/10-common-software-architectural-patterns-in-a-nutshell-a0b47a1e9013](https://towardsdatascience.com/10-common-software-architectural-patterns-in-a-nutshell-a0b47a1e9013)

[https://mingrammer.com/translation-10-common-software-architectural-patterns-in-a-nutshell/](https://mingrammer.com/translation-10-common-software-architectural-patterns-in-a-nutshell/)

[https://medium.com/@liams_o/fundamental-software-architectural-patterns-663440c5f9a5](https://medium.com/@liams_o/fundamental-software-architectural-patterns-663440c5f9a5)

[https://www.geeksforgeeks.org/types-of-software-architecture-patterns/](https://www.geeksforgeeks.org/types-of-software-architecture-patterns/)

소프트웨어 아키텍처와 디자인

[https://www.geeksforgeeks.org/difference-between-software-design-and-software-architecture/?ref=rp](https://www.geeksforgeeks.org/difference-between-software-design-and-software-architecture/?ref=rp)

[https://www.tutorialspoint.com/software_architecture_design/introduction.htm](https://www.tutorialspoint.com/software_architecture_design/introduction.htm)

[https://velog.io/@toma/소프트웨어-아키텍쳐디자인-패턴의-개념과-차이점](https://velog.io/@toma/%EC%86%8C%ED%94%84%ED%8A%B8%EC%9B%A8%EC%96%B4-%EC%95%84%ED%82%A4%ED%85%8D%EC%B3%90%EB%94%94%EC%9E%90%EC%9D%B8-%ED%8C%A8%ED%84%B4%EC%9D%98-%EA%B0%9C%EB%85%90%EA%B3%BC-%EC%B0%A8%EC%9D%B4%EC%A0%90)

[https://juyeop.tistory.com/28](https://juyeop.tistory.com/28)

[https://dongmindevloper.tistory.com/46](https://dongmindevloper.tistory.com/46)

[https://stackoverflow.com/questions/4243187/whats-the-difference-between-design-patterns-and-architectural-patterns](https://stackoverflow.com/questions/4243187/whats-the-difference-between-design-patterns-and-architectural-patterns)

[https://stackoverflow.com/questions/4192887/software-architecture-design-patterns/46419722#46419722](https://stackoverflow.com/questions/4192887/software-architecture-design-patterns/46419722#46419722)

[https://stackoverflow.com/questions/54894326/whats-the-difference-between-design-pattern-architectural-pattern-architectur](https://stackoverflow.com/questions/54894326/whats-the-difference-between-design-pattern-architectural-pattern-architectur)

[https://singhdivesh.medium.com/according-to-wikipedia-b1afa6de08c](https://singhdivesh.medium.com/according-to-wikipedia-b1afa6de08c)

[https://en.wikipedia.org/wiki/Software_architecture](https://en.wikipedia.org/wiki/Software_architecture)

[https://www.geeksforgeeks.org/fundamentals-of-software-architecture/](https://www.geeksforgeeks.org/fundamentals-of-software-architecture/)

[https://www.geeksforgeeks.org/types-of-software-architecture-patterns/](https://www.geeksforgeeks.org/types-of-software-architecture-patterns/)

MVC는 디자인 패턴인가 아키텍처인가

[https://stackoverflow.com/questions/1866821/is-mvc-a-design-pattern-or-architectural-pattern](https://stackoverflow.com/questions/1866821/is-mvc-a-design-pattern-or-architectural-pattern)

[https://www.quora.com/Is-MVC-is-a-pattern-or-architecture](https://www.quora.com/Is-MVC-is-a-pattern-or-architecture)
