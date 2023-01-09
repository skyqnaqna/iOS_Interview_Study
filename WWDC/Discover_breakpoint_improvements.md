# Discover breakpoint improvements

# 내용

## Source file breakpoints

> 단일 파일에 설정되는 breakpoints
> 

일반적으로 **Line breakpoint**가 있음

하지만 라인 중단점은 컴파일러가 LLDB가 멈출 수 있는 두 개 이상의 위치를 생성한다 → 함수와 파라미터의 생성자 같은 경우

따라서 중단점이 세분화되지 않음

70번 라인에 있는 convertedToVolume이 실행되기 직전에 일시 중지하고 싶은 상황

![](https://user-images.githubusercontent.com/31722496/211304590-e05a9d51-c13c-4299-9ffc-01d8e0ba3e26.png)


![Untitled 1](https://user-images.githubusercontent.com/31722496/211304830-1009e324-09cd-4dc4-a687-e9bedfc71206.png)

![Untitled 2](https://user-images.githubusercontent.com/31722496/211304843-18ca9548-afef-4548-b7a9-fa9c9d345633.png)

![Untitled 3](https://user-images.githubusercontent.com/31722496/211304855-4ba28d09-484c-4a7b-aeba-75bd580c148b.png)

convertedToVolume으로 들어가기 전에 adjustedDensity가 먼저 실행되어야 하므로 먼저 보여지게 되고 

Step Out해서 adjustedDensity를 나오고나서 

다시 Step Into를 해야 convertedToVolume으로 들어감 → 반복적인 일이라 불편하다


![Untitled 4](https://user-images.githubusercontent.com/31722496/211304863-e1fc2300-3131-4baa-b37a-b0010c986b8a.png)

그래서 Xcode13에서 **Column breakpoints**가 등장 → 해당 라인의 원하는 지점에서 일시 정지 가능

![Untitled 5](https://user-images.githubusercontent.com/31722496/211304870-5723704b-c43c-4a30-8974-4f5eb0d8f24b.png)

![Untitled 6](https://user-images.githubusercontent.com/31722496/211304877-f564c6b4-1d90-4ecf-92a0-7686f40bb6ce.png)

원하는 지점에서 Column breakpoint를 지정하면 된다.

![Untitled 7](https://user-images.githubusercontent.com/31722496/211304881-513068e5-54fa-4e55-a178-38682bd142a1.png)

Line breakpoint처럼 클릭해서 활성/비활성할 수 있고, 더블 클릭하게 되면 수정할 수 있다.

![Untitled 8](https://user-images.githubusercontent.com/31722496/211304887-96d48b64-c92a-4830-8c08-be349879bf66.png)

Xcode는 중지된 라인을 하이라이트하고 중단된 지점에 밑줄을 그려준다. (Xcode 11.4에서 나온 column PC)

Column PC는 중지된 지점에 초록색 밑줄을 그려줌

![Untitled 9](https://user-images.githubusercontent.com/31722496/211304889-0bb15019-627f-403f-ba4c-56b9648e6a8d.png)


![Untitled 10](https://user-images.githubusercontent.com/31722496/211304900-2e0fe7c7-19f8-4dd1-a3f8-9ad27c1c2c20.png)

이제 Step Into하면 바로 convertedToVolume으로 이동함

Column breakpoints는 특히 클로저에서 유용하다.

![Untitled 11](https://user-images.githubusercontent.com/31722496/211305221-98d9513b-df21-40a3-9f7d-804477f93a04.png)

떄때로 한 줄에 여러 클로저를 가질 수 있다.

컴파일러가 디버그 상황에서 컴파일할 때 소스의 line과 column을 컴파일된 주소로 매핑하는 라인 테이블이라는 맵을 생성한다.

그래서 컴파일러는 해당 라인에서 각 클로저에 대한 라인 테이블 엔트리를 생성하여 디버거가 일시 정지할 때 사용할 수 있도록 합니다.

## Symbolic breakpoints

> 프로세스를 중지할 때 사용하는 함수 이름의 breakpoints
> 

Source file breakpoints를 사용할 수 없거나 불편한 상황에서 유용하다.

예를 들어 소스 파일에 접근하지 못하면 디버그 정보를 컴파일할 수 없다.

또는, 공통 기능을 하는 서브 클래스가 많다면 각 클래스에 source file breakpoint를 설정하는 것이 번거로울 것이다.

![Untitled 12](https://user-images.githubusercontent.com/31722496/211305235-a25fc92c-64a8-4104-b023-e7d170d2385e.png)

breakpoint navigator에서 add버튼을 누르면 생성할 수 있는 breakpoint 목록이 나타난다.

![Untitled 13](https://user-images.githubusercontent.com/31722496/211305244-ebacfe4a-2dff-4341-820b-4c2153755716.png)

Symbolic breakpoint를 선택하면 곧바로 breakpoint 편집기가 나타난다.

![Untitled 14](https://user-images.githubusercontent.com/31722496/211305257-c241c790-657a-4f72-bb2b-39f8c17646ec.png)

몇 개의 클래스에서 사용되는 toggle 함수에 관심이 있다고 가정해보자. 각 클래스별로 찾지 말고 여기에 toggle을 입력한다. 하지만 일반적인 단어인 함수 이름은 주의해야 한다.

LLDB가 시스템 라이브러리를 포함한 모든 라이브러리의 이름을 매칭하기 때문이다.

그래서 제한이 없으면 중단점 위치가 매우 많아질 수 있다. 이 중 많은 수가 실행 경로에 지속적인 영향을 주면 문제가 발생할 수 있다.

그래서 특정 모듈로 검색을 제한할 수 있다.

모듈은 메인 바이너리를 포함한 실행 중에 로드할 수 있는 바이너리 또는 이미지이다.

<img src=https://user-images.githubusercontent.com/31722496/211305268-0d0b318a-53a9-4097-bdf7-3b11f5fea963.png width=50%>

![Untitled 16](https://user-images.githubusercontent.com/31722496/211305269-81c7cae0-900d-4bbe-bd8c-765187a7adb9.png)

모듈에 Fruta를 입력 → 해당 앱의 binary name

<img src=https://user-images.githubusercontent.com/31722496/211305271-e3c81f18-52da-4d01-851d-1b3b32f0aec3.png width=50%>

그 결과 현재 앱에서 사용하는 3개의 위치를 특정

<img src=https://user-images.githubusercontent.com/31722496/211305273-19a19814-437c-4970-ba1e-d2693a07b336.png width=50%>

이제 하트 버튼을 터치해서 토글 함수를 수행하게 되면 해당 지점에서 일시 정지할 것이다.

하지만 Symbolic breakpoints는 우리가 오타를 내기 쉬우므로 프로그램 실행 중에 왜 안멈추지?라는 상황이 발생하기 쉽다.

convertToMass라는 것을 만들어보자

![Untitled 19](https://user-images.githubusercontent.com/31722496/211305280-3cd99432-9f50-4378-83f9-9db73f146866.png)

Xcode13의 새로운 기능으로 LLDB에 주어지는 breakpoint 위치가 없다면 점선 아이콘으로 알려준다

아이콘에 마우스를 올리면 breakpoint가 해결되지 않은 이유에 대한 설명이 나온다.

처음 두 가지 이유는 breakpoint 종류와 관련이 있다.

이름 철자가 정확한지, 해당 라이브러리에 포함된 것인지…

세 번째는 더 일반적이다. breakpoint에 대한 라이브러리를 로드해야 하기 때문이다.

때때로 사용자 작업을 수행한 후에 로드되는 라이브러리가 있다. 이땐 LLDB가 자동으로 breakpoint를 해결한다.

따라서 이 경우엔 오타를 의심할 수 있다. 보통 네비게이터를 사용하여 검색하지만 LLDB를 통해 다른 트릭을 사용해보자.

![Untitled 20](https://user-images.githubusercontent.com/31722496/211305282-843ec924-6cf4-4fa6-96b9-dbe05e400a1c.png)

![Untitled 21](https://user-images.githubusercontent.com/31722496/211305585-8c364d2c-9dfa-4661-a2ea-8fe414c8153a.png)

Xcode 콘솔에서 모듈을 의미하는 image, regex를 위한 lookup -r, 이름 n과 검색하려는 이름 convert 그리고 모듈 이름인 Fruta를 입력하여 검색 제한을 둔다.

검색 결과 일치하는 항목이 4개이며 함수 이름의 철자를 잘못 쓴 것을 볼 수 있다.

convertToMass → convertedToMass로 고치면 잘 해결된 것을 볼 수 있다.

![Untitled 22](https://user-images.githubusercontent.com/31722496/211305593-a5b7e3f4-e72a-4214-868b-7752e1a6fef7.png)


![Untitled 23](https://user-images.githubusercontent.com/31722496/211305596-ea1ed8d8-4c3b-40d6-b6d8-6b1e34d70aba.png)


이제 다른 파일로 가보자

해결되지 않은 breakpoint는 source file breakpoints에서도 볼 수 있다.

![Untitled 24](https://user-images.githubusercontent.com/31722496/211305600-077ddd3c-bf29-4f2f-9d22-df71804aa1c4.png)


![Untitled 25](https://user-images.githubusercontent.com/31722496/211305603-d30996e1-b6ae-42c0-af46-e84f93daa4b9.png)


두 가지 이유가 있다.

먼저 breakpoint 행을 컴파일해야 한다.

해당 라인은 컴파일러 조건 분기로 컴파일되지 않았다.

또한 컴파일러가 모듈에 대한 디버그 정보를 생성해야 한다.

그렇지 않으면 빌드 설정을 확인해야 한다.

## Runtime issue breakpoints

> 백그라운드 스레드에서 UI 상태를 변경하는 등 런타임에 발생하는 문제의 breakpoints
> 

이 경우는 충돌만큼 신각한 경우는 아니다.

기본적으로 Xcode는 다른 버그에 집중할 때 방해가 될 수 있기 때문에 프로세스를 일시 중지하지 않는다.

대신 런타임 문제가 발생하면 역추적을 기록하여 이슈 탐색기에 표시한다.

하지만 이슈는 이미 과거에 발생한 것이라 현재 프로세스 상태를 점검하는 것은 의미가 없다.

그래서 가끔 해당 이슈를 잡고 싶을 것이다.

![Untitled 26](https://user-images.githubusercontent.com/31722496/211305607-8650f291-573d-47b1-b5a7-b2be3c85047a.png)


![Untitled 27](https://user-images.githubusercontent.com/31722496/211305610-dbade6ea-670b-47ca-bb7d-d1a7b1bfcee1.png)


Runtime issue breakpoint가 있으면 디버거에서 일시 중지하고 프로세스를 살펴볼 수 있다.

Runtime issue breakpoint는 여러 유형이 있다.

![Untitled 28](https://user-images.githubusercontent.com/31722496/211305613-400bd992-8487-4fa3-9684-c64c27b2f8f7.png)


이 중 일부는 스키마 편집기의 진단 탭에서 해당 기능을 활성화해야 한다.

![Untitled 29](https://user-images.githubusercontent.com/31722496/211305616-9945c3c7-7c6a-4422-9490-b85f51ec4808.png)

![Untitled 30](https://user-images.githubusercontent.com/31722496/211305617-26424b99-66c1-46a7-a673-6a6e316e4a04.png)


메인 스레트 체커를 사용하기 위해 활성화한다.

> breakpoint는 디버깅 능력을 크게 향상시킬 수 있으며, 반드시 레퍼토리의 일부가 되어야 한다.

# 연관 주제

LLDB에 대한 추가적인 팁과 트릭들 → WWDC19 [LLDB: Beyond ‘po’](https://developer.apple.com/videos/play/wwdc2019/429/)

# 참고 자료

[https://developer.apple.com/wwdc21/10209](https://developer.apple.com/wwdc21/10209)
