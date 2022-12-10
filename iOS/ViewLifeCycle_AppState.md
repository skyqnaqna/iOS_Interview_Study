# 답변

View-related Notifications(=View LifeCycle)은 

1. ViewController가 메모리에 올라가면 뷰가 로드되고(`viewDidLoad`)
2. 뷰가 나타나기 직전에 `viewWillApper` 가 호출되고 
3. 뷰가 나타납니다.(`viewDidAppear`) 
4. 뷰가 사라질 땐 직전에 `ViewWillDisapper` 가 호출되고
5. `ViewDidDisapper` 가 호출되며 뷰가 사라집니다.

App의 상태는 NotRunning, Inactive, Active, Background, suspended 상태가 있습니다.

App LifeCycle은 iOS 13을 기점으로 나뉘는데, 이후는 Scene을 기준으로 응답합니다. 
주로 앱의 실행상태에 따라 active / background 상태에서 변동하고, 앱이 비활성화 될 때 화면여부에 따라 inactive or background 상태로 바뀌기도 합니다.

## 내용

### UIKit 의 핵심

event-driven user interface

### ViewController의 주요 책임

- 데이터 변경에 대한 응답으로, 뷰의 content 업데이트
- 뷰와 유저의 상호작용에 대한 응답
- 뷰 resizing 및 전체 인터페이스 레이아웃을 관리
- 다른 viewcontroller를 포함한 다른 개체와 coordinating(조정)

### 공식문서 - ViewController

ViewController는 `UIResponder`의 객체이며, 뷰 컨트롤러의 루트뷰와 다른 뷰 컨트롤러에 속하는 해당 뷰의 슈퍼뷰 사이에 responder chain이 삽입된다.

ViewController는 단독으로 거의 사용되지 않고 새로운 뷰 set을 보이기위해 다른 ViewController를 보여주거나, 다른 ViewController들의 컨텐츠에 대한 컨테이너 역할을 하고 원하는대로 뷰를 움질일 수 있다.

RootView의 크기나 위치는, 상위 ViewController 또는 앱의 window를 소유한 객체에 의해 결정된다. 또한 한번만 호출!

ViewController는 뷰를 느리게 로드하는데, 뷰 프로퍼티에 처음 접근하면 뷰가 로드되거나 생성된다.

- ViewController가 View를 지정하는 방법
    - 스토리보드 사용
    - nib file 생성
    - loadView() 메서드 사용해서 뷰컨에 대한 뷰를 지정

<aside>
📢 뷰컨은 뷰와 뷰가 생성하는 모든 하위 뷰들의 ‘유일한’ 소유자이다.
뷰컨은 해제될 때 같이 적절한 시점에 뷰의 소유권을 포기하는 책임이 있다.
스토리보드나 nib 사용해서 뷰 객체를 저장하는 경우, 각 뷰컨 객체는 자동으로 뷰의 복사본을 가져온다.
그러나, 수동으로 생성해야 하는 경우(loadView를 말하나?) 각 뷰컨은 고유한 뷰 집합이 있어야 한다. 뷰컨 간의 뷰를 공유할 수 없다.

</aside>

### ****View-related Notifications (=View LifeCycle)****
<img src="https://user-images.githubusercontent.com/37897873/206861575-82285daa-eee9-4c4d-822f-79dc80bebbe7.png" width=350>

1. `loadView` :  뷰를 만드는 역할. 이 메서드를 직접 호출하면 안되고, 뷰의 초기화는 viewDidLoad에서 해야한다. 
2. `viewDidLoad` : 뷰의 컨트롤러가 메모리에 로드 된 후(loadView에서 뷰를 만들고 메모리에 올린 후)에 호출되며, 뷰 로딩이 완료되었을 때 시스템에 의해 자동으로 호출되어서 주로 리소스 초기화하거나, 처음 한번만 실행하므로  초기 화면 구성할 때 사용해야한다.
3. `viewWillAppear` : 뷰가 나타나기 직전에 호출된다. B→A 상황일 때 처리할 코드 작성하면 된다.
4. `viewDidAppear` : 뷰가 나타났다고 컨트롤러에게 알리고 화면에 적용될 애니메이션을 그린다. 즉, 화면에 나타난 직후
5. `ViewWillDisapper` : 뷰가 사라지기 직전에 호출, 뷰가 사라지려고 하는걸 뷰컨에게 알린다.
6. `ViewDidDisapper` : 뷰컨에 뷰가 제거되었음을 알린다.

### App State
<img src="https://user-images.githubusercontent.com/37897873/206861592-4fa05268-8774-435f-bbac-b11d37a37d3d.png" width=350>

1. NotRunning : 실행되지 않았거나, 시스템에 의해 종료된 상태
2. Inactive : 실행 중이지만 이벤트를 받고있지 않은 상태. 예를들어, 앱 실행 중 미리알림 또는 일정 얼럿이 화면에 덮여서 앱이 실질적으로 이벤트는 받지 못하는 상태등을 뜻합니다.
3. Active : 어플리케이션이 실질적으로 활동하고 있는 상태.
4. Background : 백그라운드 상태에서 실질적인 동작을 하고 있는 상태. 예를 들어 백그라운드에서 음악을 실행하거나, 걸어온 길을 트래킹 하는 등의 등 뜻합니다.
5. Suspended : 백그라운드 상태에서 활동을 멈춘 상태. 빠른 재실행을 위하여 메모리에 적재된 상태지만 실질적으로 동작하고 있지는 않습니다. 메모리가 부족할때 비로소 시스템이 강제종료하게 됩니다.

## ****App Life Cycle****

앱의 현재 상태에 따라 수행할 수 있는 것과 없는 것이 결정되는데, 예를 들어 foreground 라면 유저의 주의를 끌기 때문에 (화면이 보이니까) CPU를 포함한 시스템 리소스보다 우선순위가 높다. 반대로 background 작업은 가능한 적은 작업을 수행해야 하고 화면 외에 있기 때문에 아무 작업도 하지 않는게 좋다.

앱의 상태가 변경되면 UIKit은 적절한 delegate object를 호출해서 알려준다.

### Scene-based life-cycle events
<img src="https://user-images.githubusercontent.com/37897873/206861622-de7e4fef-def8-44f9-a9dd-ba2e289d3e8f.png" width=300>



`Scene`이란 기기에서 실행되는 앱 UI의 한 instance를 의미한다. 각각의 scene은 고유한 life-cycle이 있으므로, 다른 실행 상태에 있을 수 있다. (예를들어, A scene은 foreground B scene은 background로)

`Scene`은 한 Scene 세션에 대한 Configuration이 AppDelegate에서 설정되고, 이를 기반으로 SceneDelegate의 willConnectToSession에서 UI를 만들어낸다.

- iOS 13 ~ 이후 : `UISceneDelegate` 를 사용하며 scene을 제공하므로 별도의 Scene life-cycle 이벤트를 제공
- iOS 12 ~ 이전 : `UIApplicationDelegate` 를 사용하고, 그냥 life-cycle event에 응답

<img src="https://user-images.githubusercontent.com/37897873/206861646-077c4d70-fb81-4ac7-95e3-02f2570f36f3.png" width=350>


자세한 동작 사진
<img src="https://user-images.githubusercontent.com/37897873/206861643-5f739b99-f3e1-4515-981a-43b69a1807f6.png" width=350>
    
1. scene의 초기 UI를 구성하고 필요한 data를 로드한다.
2. Foreground-Active : (우리가 짠) UI를 구성하고, 유저와 interact 하기위해 준비한다. [Preparing your UI to run in the foreground](https://developer.apple.com/documentation/uikit/app_and_environment/scenes/preparing_your_ui_to_run_in_the_foreground)
3. Foreground 상태를 벗어날 때 데이터를 저장하고, 앱의 동작을 quite하게 한다.
4. Background 상태에 진입할 때, 중요한 작업을 완료하고 가능한 많은 메모리를 확보하고 app snapshot을 준비한다. [Preparing your UI to run in the background](https://developer.apple.com/documentation/uikit/app_and_environment/scenes/preparing_your_ui_to_run_in_the_background)
5. Scene 관련 이벤트 외에도 UIApplicationDelegate 객체를 이용해서 앱 시작시 수행해야 될 작업들에 응답해야한다. [Responding to the launch of your app](https://developer.apple.com/documentation/uikit/app_and_environment/responding_to_the_launch_of_your_app)

### App-based life-cycle events

iOS 12 이전에는 scene을 제공하지 않아서, UIKit의 모든 life-cycle을 `UIApplicationDelegate` 에 전달한다. App delegate는 앱의 모든 창을 관리하는데, 앱의 상태 전환은 외부 화면의 content를 포함해 앱의 전체 UI에 영향을 끼친다. 

launch 이후, UI가 화면에 표시되는지 여부에 따라 앱을 inactive or background 상태로 전환한다.

foreground로 실행되면, 자동으로 앱을 활성화 상태로 전환한다.

그 이후에는 앱이 종료될 때까지 active ↔ background 상태에서 변동된다.

<img src="https://user-images.githubusercontent.com/37897873/206861676-169d8332-db26-4486-9d1b-059c44defa73.png" width=350>


### 정리

- 앱이 처음 실행될 때
    1. `**func** scene()` :  UIWindow의 scene을 선택적으로 구성하고, 제공된 window에 scene을 연결한다.
        
        스토리보드는 window를 자동적으로 초기화해서 scene에 붙여주지만, 코드로 UI 짤 때는 이렇게 직접 연결해야한다.
        
        <img width="600" alt="%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-11-24_%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB_11 30 48" src="https://user-images.githubusercontent.com/37897873/206861687-87108521-1e04-45a6-9990-c15d7e215736.png">

        
    2. `func sceneWillEnterForeground` 
    3. `func sceneDidBecomeActive`
- 앱이 Inactive 상태로 전환될 때 : `func sceneWillResignActive`
- 다시 active 상태로 바뀌면 : `func sceneDidBecomeActive`
- 앱이 아예 꺼지면 : `func sceneDidDisconnect`

### ****Respond to other significant events****

추가적으로 앱은 밑의 처리들을 해야하는데, UIApplicationDelegate 객체를 사용해서 처리해야 한다.

- 메모리 warning : 앱이 메모리를 너무 많이쓰면 메모리 사용량을 줄인다. [Responding to memory warnings](https://developer.apple.com/documentation/uikit/app_and_environment/managing_your_app_s_life_cycle/responding_to_memory_warnings)
- Protected data를 available/unavailable : 유저가 기기를 잠그거나 잠금해제 될 때 수신된다.
- Handoff tasks : `[NSUserActivity](https://developer.apple.com/documentation/foundation/nsuseractivity)` 객체를 사용해서 받을 수 있는데, handoff란 Ipad에서 작업하다가 Mac으로 바로 전환해서 이어서 작업할 수 있는 기능 ( [참고](https://support.apple.com/ko-kr/HT209455) )
- Time changes : 이동통신사가 시간 업데이트를 보내는 경우와 같이 여러 가지 다른 시간 변경에 대해 수신
- Open URLs : 앱에서 리소스를 열어야 할 때 수신

## 추가 질문

### A → B → A 순으로 네비게이션을 이용해 push, pop한다면?

- A 등장 : ViewDidLoad → ViewWillApper → ViewDidAppear
- A → B 전환 : ViewWillDisappear(A) → ViewDidLoad(B) → ViewWillAppear(B) → ViewDidDisappear(A) → ViewDidAppear(B)
- B → A 전환 : ViewWillDisappear(B) → ViewWillAppear(A) → ViewDidDisapper(B) → ViewDidAppear(A)

### 카톡 잠금을 설정해놓고 멀티태스킹 화면으로 이동하면 가려지는데 어떻게 구현된걸까?

앱을 시리를 부르거나 멀티테스킹 화면으로 옮길 때, Inactive 상태로 바뀌는 데 이때 `sceneWillResignActive` 가 호출되므로 여기에 UI를 잠금화면 배경으로 변경할 것 이다.

### iOS 13부터 Scene Life Cycle이 생겼는데 왜 생겼을까요?

iPad 13부터 하나의 앱을 두개의 Scene으로 나눠서 사용가능해진다.
ex) 한 아이패드 화면에 하나의 앱을 두개 띄울 수 있게해서 각각 독립적으로 동작
- 참고 WWDC19 : [https://developer.apple.com/videos/play/wwdc2019/258](https://developer.apple.com/videos/play/wwdc2019/258)

## 참고 자료

- App Lifecycle : [https://jinshine.github.io/2018/05/28/iOS/앱의 생명주기(App Life Cycle)와 앱의 구조(App Structure)](https://jinshine.github.io/2018/05/28/iOS/%EC%95%B1%EC%9D%98%20%EC%83%9D%EB%AA%85%EC%A3%BC%EA%B8%B0(App%20Life%20Cycle)%EC%99%80%20%EC%95%B1%EC%9D%98%20%EA%B5%AC%EC%A1%B0(App%20Structure)/)
- [https://velog.io/@minni/iOS-Application-Life-Cycle](https://velog.io/@minni/iOS-Application-Life-Cycle)
- ViewLifeCycle
    - [https://zeddios.tistory.com/43](https://zeddios.tistory.com/43)
    - [https://zeddios.tistory.com/44](https://zeddios.tistory.com/44)
- [https://jeonyeohun.tistory.com/226](https://jeonyeohun.tistory.com/226)

### 공식문서

- [https://developer.apple.com/documentation/uikit/uiviewcontroller](https://developer.apple.com/documentation/uikit/uiviewcontroller)
- [https://developer.apple.com/documentation/uikit/app_and_environment/managing_your_app_s_life_cycle](https://developer.apple.com/documentation/uikit/app_and_environment/managing_your_app_s_life_cycle)
- [https://developer.apple.com/documentation/uikit/app_and_environment/managing_your_app_s_life_cycle](https://developer.apple.com/documentation/uikit/app_and_environment/managing_your_app_s_life_cycle)
- [https://developer.apple.com/documentation/uikit/app_and_environment/managing_your_app_s_life_cycle](https://developer.apple.com/documentation/uikit/app_and_environment/managing_your_app_s_life_cycle)