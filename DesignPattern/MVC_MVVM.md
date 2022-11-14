## 답변

MVC는 Model-View-Controller 로 구성되어 있다.
Model은 사용되는 데이터나 그 데이터를 처리하는 부분
View는 레이어에 표현되어 있는 것들을 책임지는 부분, 주로 ‘UI’가 접두로 붙는다.
Controller는 사용자의 입력을 받고 처리하는 부분이다. 

MVVM은 Model-View-ViewModel로 구성되어 있으며, MVC의 앞부분 MV와 동일하다.
MVVM에서 Controller는 View라고 일컫으며 View와 Model이 서로 연결되어 있지 않고 ViewModel이 중간다리 역할을 한다.
ViewModel은 View를 표현하기 위해 만든 (단어 그대로) View를 나타내기 위한 Model이다.

## 내용

### 전통적인 MVC

기존 MVC 패턴과 애플에서 쓰는 MVC 패턴이 다르다는 걸 아시나요 ?! ~~저도 첨에 몰라씀~~

![전통mvc](https://user-images.githubusercontent.com/37897873/201581085-cc92d15d-a1f4-424b-8f0e-241d80de2c6e.png)


전통적인 MVC의 경우 View의 범위가 명확하지 않은데, Model이 바뀌면 Controller에 의해 한번 렌더링된다. 이는 보면 View ↔ Controller, View ↔ Model이 꽉 묶여 있고 재사용성을 심각하게 줄인다. 이런 이유로 Apple의 MVC를 만들었다.

### Apple’s MVC

기존의 MVC와 비교해보면, 단점이였던 두 묶음의 의존성을 떨어뜨린 것을 한눈에 알 수 있다. 

![apple'smvc](https://user-images.githubusercontent.com/37897873/201581078-704f2017-7b5a-42b9-97bd-abb93948bd55.png)


Controller는 Model과 View를 연결시켜주는 중간다리 역할을 하기 때문에 서로를 알 필요가 없다. 그 중 많은 역할과 책임이 Controller에게 주어지고 가장 재사용 불가능 한 부분이다. MVC를 사용하면 ViewController가 거대해진다고(Massive) 들어본 적 있을 것이다. 왜 전통적인 MVC를 개선했는데도 이런 문제가 발생한 걸까?

이론상은 위와 같지만, 실제로 동작은..

![applemvc2](https://user-images.githubusercontent.com/37897873/201581076-d211e7dc-e8ff-4ecb-b009-0c284cd79eea.png)

View Life Cycle에서 View들이 뒤엉켜져 분리하기 어렵기 때문이다. 대부분 View에서 반응하면 Controller로 보내게 되고, ViewController는 결국 모든 것의 delegate나 data source가 된다. 네트워킹 처리를 하면 그 로직도 Controller가 담당하게 될 것이다.

```swift
var userCell = tableView.dequeueReusableCellWithIdentifier("identifier") as UserCell
userCell.configureWithUser(user)
```

테이블 뷰의 셀을 구성하는 cell은 MVC 가이드라인을 위반하는데, 우리는 개발할 때 자주 이 코드를 썼을 것이다. 

왜 위반되냐, Cell 자체는 View이다. 근데 Cell(View)에 user(Model)을 직접 주입해버린다. 위 사진을 보면 분명 View와 Model 사이는 직접 접근하지 않고 Controller를 통해서만 접근 가능한데, 엄밀히 따지면 어긋난 코드인 것이다.

MVC를 명확히 따르려면, cell을 Controller에 구성하고, View안에 Model을 직접 거치지 않아야 한다. 근데 이렇게 수정하면 Controller가 더 커질 것이다!

간단하게 MVC 예시를 구현해 보았는데, 

- Distrubution : View와 Model은 분리되 있지만, ViewController는 View에 딱 붙어 있다.
- Testability : 이는 테스트를 위한 View를 작성하기 어려워진다. Model만 테스트 가능할 것이다.
- Ease of use : 사용하기엔 MVC 만큼 쉬운 패턴이 없을 것 같다. 가장 초보단계일 때 사실 접하고 짠 코드가 아마 모두 MVC일 것이다.

```swift
import UIKit

//model
struct Stuff {
    enum Sort {
        case iPhone
        case water
    }
    let name: Sort
}

//View+Controller
class MVCViewController: UIViewController {
    
    var iPhone: Stuff!
    var water: Stuff!
    
    @IBOutlet weak var descriptionLabel: UILabel!
    
    override func viewDidLoad() {
        super.viewDidLoad()
    }
    
    @IBAction func tapiPhone(_ sender: UIButton) {
        iPhone = Stuff(name: .iPhone) //모델 값 변경
        descriptionLabel.text = "당신은 \(self.iPhone.name)을 좋아합니다!" //Controller는 View에게 변화를 알림
    }
    
    @IBAction func tapWater(_ sender: UIButton) {
        water = Stuff(name: .water)
        descriptionLabel.text = "당신은 \(self.water.name)을 좋아합니다!"
    }

    
}
```

### MVVM

Massive한 ViewController의 역할을 덜기 위해 MVP를 거쳐 MVVM이 등장했다. ViewModel은 Model에서 변경이 일어났다고 알리면, Model 자체를 갱신한다. 

![mvvm](https://user-images.githubusercontent.com/37897873/201581034-f9e0aa10-eff9-46a0-bee3-00796d206a05.png)


사실 ViewModel을 구현하는 방법에는 여러가지가 있지만, 그 중 Input-Ouptut 구조를 사용하였다. 입력으로 값을 받으면 모델의 값을 갱신하여 View의 label에 보이기 위한 String 값을 Output으로 내보낸다.

(사실 이 예제는 아주 간단해서 굳이 모델이 필요할까? 라고 생각할 수 있지만, 좀 더 변수가 많은 모델과 코드가 길어진다고 생각하면 유용하게 사용될 것 이다)

```swift

enum Sort {
    case iPhone
    case water
}

//model
struct _Stuff {
    let name: Sort
}

class ViewModel {
    struct Input {
        let stuff: Sort
    }
    
    struct Output {
        let labelText: String
    }
    
    func transform(input: Input) -> Output {
        var model: _Stuff?
        switch input.stuff {
        case .iPhone:
            model = _Stuff(name: .iPhone)
        case .water:
            model = _Stuff(name: .water)
        }
        let result = model?.name ?? .water
        return Output(labelText: "\(result) 를 좋아군요!")
    }
}
```

MVVM에서 Controller는 View라고 일컫기 때문에, View가 현재 ViewModel을 소유하고 있고, Model은 직접적으로 알지 못한다. ViewModel.Input에 전달하면 View를 위한 Model 값으로 Output이 나온다.

이 변경된 ViewModel의 Output을 바탕으로 View에게 바인딩 한다. IBAction을 통해 유저 액션이 연결되어 있다. 

```swift
import UIKit

class MVVMViewController: UIViewController {
    
    var viewModel: ViewModel = .init() //ViewController는 ViewModel을 소유
    
    @IBOutlet weak var labelText: UILabel!
    
    override func viewDidLoad() {
        super.viewDidLoad()
    }
    
    @IBAction func tapiPhone(_ sender: UIButton) {
        let output = viewModel.transform(
            input: ViewModel.Input(stuff: .iPhone) //1. ViewModel에 값을 주입
        )
        labelText.text = output.labelText //2. 변경된 ViewModel의 output 값을 View에 알림
    }
    
    
    @IBAction func tapWatter(_ sender: UIButton) {
        let output = viewModel.transform(
            input: ViewModel.Input(stuff: .water)
        )
        labelText.text = output.labelText
    }
    
}
```

- Distribution : 작은 예제라서 명료하게 보이진 않지만, View를 위한 값들을 ViewModel이 갖고 처리하기 때문에 MVC에서 문제였던 View와 Controller의 엄청난 의존성으로 View가 Model을 소유했던 (tableview의 cell 예시) 경우도 발생했지만, MVVM에선 ViewModel의 역할로 View와 Model 사이의 의존성이 없다.
- Testability : View Model은 View에대해 전혀 모르며, 이것이 테스트하기 쉽게 해준다. View 또한 테스트 가능하지만 UIKit 의존이면 그러고 싶지 않게 원하게 될 것이다.
- Easy of use : 사실상 MVVM 코드가 더 길어 쓰기에는 MVC가 더 쉽고 좋은거 아니야 ? 라고 생각할 수 있지만, 맞다. 사용하기에는 MVC가 편리하고 간단하기 때문에 프로젝트의 규모가 작다면 이를 택할 수 있지만, 규모가 엄청나게 커진다면 유지보수와 코드의 의존성도 그만큼 높아지기 때문에 프로젝트의 규모와 사용성에 따라 아키텍쳐와 패턴을 선택하는 것이 옳다. (무조건 MVVM! MVC!는 위험한 생각..)

## 추가 질문

Q1. 왜 패턴들이 생기고 계층을 나눌까요?

각 코드의 역할을 나누어서 관리하면 의존성을 줄일 수 있고, 재사용성을 높일 수 있습니다. 이는 유지보수에 편리하고 개발 효율이 높아집니다. 또한 테스트하기 좋은 코드가 됩니다.

구체적인 설명 없이도 구조화된 패턴에 대해 사전 지식이 있다면 커뮤니케이션에 수월하고, 이미 검증된 구조이기 때문에 설계과정에 속도를 높일 수 있습니다.

Q2. 좋은 아키텍처의 특징은?

- 개체들간의 책임 분리(Distribution)를 균형있게 해야한다.
- 테스트 가능해야 한다. (Testability)
- 사용에 편해야 하고 (Ease of use) 유지보수 하기 쉬워야 한다.

Q2-1. 왜 분리해야 하나요?

우리가 하나의 큰 코드를 보고도 각 코드의 역할을 알 수 있으면 문제가 없을 것이다. 그렇지만, 코드가 커지면 커질수록 사람이라면 기억하지 못하고 헷갈리게 될 것이다. 이를 극복하는 방법은 “단일 책임 원칙”으로 개체들의 책임을 쪼개 가독성을 높이는 것이다.

Q2-2. 왜 테스트 가능해야 하나요? (참고 대답)

이미 유닛테스트에대한 중요성을 알고 있는 사람에게 던지는 질문이 아니라, 새 기능을 추가한 뒤 일때나, 클래스의 몇몇 복잡성을 리팩토링을 하기 위해서 테스트에 실패하는 사람들이 하는 의문이기도하다. 이것은 테스트가 런타임 내에서의 이슈를 찾는데 도와주며, 반대로 실유저에게 이슈가 발생한다면 그걸 고친 앱을 다시 실유저가 다시 사용하기까지 일주일씩이나 걸린다. 

Q2-3. 왜 사용하기 쉬워야 하나요?

가장 좋은 코드는 ? 바로 하나도 작성하지 않은 코드이다 (ㅋㅋㅋ) 따라서 적은 코드는 그만큼 버그가 적고 최대한 중복되지 않도록 코드를 재사용하며 작성하는 것이 좋을 것이다!

## 참고 자료

- [https://blog.canapio.com/43](https://blog.canapio.com/43)
- [https://medium.com/hcleedev/ios-개발-mvc-패턴과-uikit의-viewcontroller-3fdb52f6b4b8](https://medium.com/hcleedev/ios-%EA%B0%9C%EB%B0%9C-mvc-%ED%8C%A8%ED%84%B4%EA%B3%BC-uikit%EC%9D%98-viewcontroller-3fdb52f6b4b8)
- [https://medium.com/ios-os-x-development/ios-architecture-patterns-ecba4c38de52](https://medium.com/ios-os-x-development/ios-architecture-patterns-ecba4c38de52)

### MVVM를 공부하기 좋을 강의

- 스탠포드 대학 MVVM 강의 : [https://www.youtube.com/watch?v=4GjXq2Sr55Q](https://www.youtube.com/watch?v=4GjXq2Sr55Q)
- Raywendlich MVVM 강의 : [https://www.youtube.com/watch?v=sWx8TtRBOfk&t=413s](https://www.youtube.com/watch?v=sWx8TtRBOfk&t=413s)