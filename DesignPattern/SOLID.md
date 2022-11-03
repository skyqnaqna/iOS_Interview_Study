
# SOLID 원칙에 대해 아는대로 설명하시오


## 답변

> 좋은 객체 지향 설계를 위한 5가지 기본 원칙들입니다.  
SOLID 원칙을 적용하면 결합도를 낮추고 응집도를 높이면서, 유지보수와 확장이 쉬운 소프트웨어를 만들 수 있습니다.

## 내용

- [Single Responsibility Principle 단일 책임 원칙]()
	- 한 클래스는 하나의 기능만 가져야 한다

- [Open/Colsed Principle 개방-폐쇄 원칙]()

	- 코드를 변경하지 않고 확장할 수 있어야 한다

- [Liskov Substitution Principle 리스코프 치환 원칙]()

	- 자식 타입 클래스는 언제나 부모 타입 클래스로 바꿀 수 있어야 한다

- [Interface Segregation Principle 인터페이스 분리 원칙]()

	- 하나의 범용 인터페이스보다 여러 개의 구체적인 인터페이스가 낫다

- [Dependency Inversion Principle 의존관계 역전 원칙]()

	- 상위 모듈이 하위 모듈에 의존해서는 안되며, 둘 다 추상화에 의존해야 한다.

## 추가 질문

### 응집도와 결합도?
- 모듈의 **독립성**을 판단하는 지표 → 독립성이 높을 수록 좋은 모듈 (모듈 수정 시 다른 모듈에 미치는 영향이 감소하기 때문)
    - 모듈 : 하나의 기능을 수행하는 단위
    - 모듈화 : 소프트웨어를 각 기능별로 나누는 것

응집도 | 결합도
|:----|:----|
|모듈에 포함된 내부 요소들 사이의 밀접한 정도 |다른 모듈과의 의존성 정도|
|높을수록 좋다 → High Cohesion | 낮을수록 좋다 → Loose Coupling|
|응집도가 높은 모듈은 필요한 함수나 데이터와 같은 구성 요소들이 존재 | 각 모듈이 서로 관련성이 적어 결합도가 낮을수록 모듈 간의 독립성이 높아진다|
|응집도가 낮은 모듈은 서로 관련 없는 함수나 데이터들이 존재 | 프로그램의 일부 기능을 수정해야 할 때 그 기능을 담당하는 모듈만 수정하면 되기 때문에 유지보수가 쉬워진다|
| |각 모듈이 파라미터 등을 통해 데이터만 주고받는 결합도가 가장 낮은 결합도이다|


- [응집도와 결합도의 유형](https://www.geeksforgeeks.org/software-engineering-coupling-and-cohesion/)

### 객체 지향 설계(OOD)?
- 어떤 객체가 무엇을 할 것인지, 객체들이 어떻게 상호작용할지 책임을 할당하는 것
- 분석(OOA) 단계에서 문제와 요구사항을 조사하여 기술한다. → 개념 식별
- 설계(OOD) 단계에서는 객체의 속성과 행동을 정의한다. → 책임 할당
- 구현(OOP) 단계는 코딩을 하는 단계 → 프로그래밍 언어로 구현

### 개방폐쇄와 의존관계 원칙의 차이?
OCP의 전략 패턴 구조와 DIP의 추상화 계층을 삽입한 구조가 유사하다고 느껴 차이가 무엇인지 헷갈렸다.
찾아본 결과, DIP는 OCP와 LSP로부터 비롯되었다는 내용이 있다.

> In this column, we discuss the structural implications of the OCP and the LSP. The structure that results from rigorous use of these principles can be generalized into a principle all by itself. I call it “The Dependency Inversion Principle” (DIP).  
Robert C. Martin, Engineering Notebook, C++ Report, 1996.
> 

따라서 DIP를 준수하면 OCP를 준수하기 쉬워질 수 있지만 어느 한쪽이 다른 한쪽을 보장하는 관계는 아니다. 예를 들어 옵저버 패턴은 OCP를 준수하지만 DIP를 준수하진 않는다.  
그리고 목적에서 차이가 발생한다고 생각한다. OCP는 확장을 위해 다형성을 사용, DIP는 상위모듈과 하위모듈 간 의존성을 위해 추상화 계층을 사용한다는 점이다. 

- 참고한 곳
	- [https://stackoverflow.com/questions/18428027/what-is-difference-between-the-open-closed-principle-and-the-dependency-inversio](https://stackoverflow.com/questions/18428027/what-is-difference-between-the-open-closed-principle-and-the-dependency-inversio)
	- [https://tothefullest08.github.io/blog/](https://tothefullest08.github.io/blog/)


## 참고 자료
[https://www.geeksforgeeks.org/solid-principle-in-programming-understand-with-real-life-examples/?ref=gcse](https://www.geeksforgeeks.org/solid-principle-in-programming-understand-with-real-life-examples/?ref=gcse)

[https://ko.wikipedia.org/wiki/SOLID_(객체_지향_설계)](https://ko.wikipedia.org/wiki/SOLID_(%EA%B0%9D%EC%B2%B4_%EC%A7%80%ED%96%A5_%EC%84%A4%EA%B3%84))

https://github.com/ochococo/OOD-Principles-In-Swift

[https://velog.io/@trequartista/TIL-SRP](https://velog.io/@trequartista/TIL-SRP)

[http://egloos.zum.com/metashower/v/9925190](http://egloos.zum.com/metashower/v/9925190)

[https://medium.com/@jang.wangsu/설계-용어-응집도와-결합도-b5e2b7b210ff](https://medium.com/@jang.wangsu/%EC%84%A4%EA%B3%84-%EC%9A%A9%EC%96%B4-%EC%9D%91%EC%A7%91%EB%8F%84%EC%99%80-%EA%B2%B0%ED%95%A9%EB%8F%84-b5e2b7b210ff)

[https://www.leafcats.com/68](https://www.leafcats.com/68)

[https://velog.io/@yu-jin-song/SW공학-결합도와-응집도](https://velog.io/@yu-jin-song/SW%EA%B3%B5%ED%95%99-%EA%B2%B0%ED%95%A9%EB%8F%84%EC%99%80-%EC%9D%91%EC%A7%91%EB%8F%84)

[https://www.geeksforgeeks.org/object-oriented-analysis-and-design/?ref=rp](https://www.geeksforgeeks.org/object-oriented-analysis-and-design/?ref=rp)

[http://contents.kocw.or.kr/document/lec/2011/33/04/Ch01.pdf](http://contents.kocw.or.kr/document/lec/2011/33/04/Ch01.pdf)