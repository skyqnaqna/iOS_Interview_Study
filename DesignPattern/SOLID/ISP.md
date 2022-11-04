# Interface Segregation Principle 인터페이스 분리 원칙

> **하나의 일반적인 인터페이스보다는, 여러 개의 구체적인 인터페이스가 낫다.**  
→ 인터페이스 단일 책임  
→ Animal 인터페이스보다는 Barkable, Walkable, Eatable 인터페이스가 낫다.  
→ 사용하지 않는 인터페이스에 변경이 발생하더라도 영향을 받지 않도록 하는 것
>
> **사용하지 않을 인터페이스는 구현하지 않는다.**  
→ 응집성과 연관

## 적용방법

1. 클래스 상속을 통한 인터페이스 분리 → 추상 클래스나 인터페이스를 사용하라
2. 상속 대신 위임 사용 → 퍼사드 패턴 (Facade Parttern)

## Ex1

<img src=https://user-images.githubusercontent.com/31722496/199670821-41b32f99-abe5-4e6e-aff6-2e17c1962e89.png width=200px>

- 이 상태로 Bus나 Taxi 같은 클래스를 만들면 Vehicle의 모든 메소드를 구현한다.
- 만약 CarAudio 기능이 없는 차량이라면??
    - Vehicle을 더 작은 인터페이스로 분할한다.

<img src=https://user-images.githubusercontent.com/31722496/199671013-d0b6214f-1613-4126-8eef-a92d869399e6.png width=50%>

## Ex2

<img src=https://user-images.githubusercontent.com/31722496/199674383-44d66423-740f-44b8-9f33-550e38fb5895.png width=50%>

<details>
<summary>Code</summary>
<div markdown="1">       

```swift
// 방문 사이트가 있다.
protocol LandingSiteHaving {
    var landingSite: String { get }
}

// LandingSiteHaving 객체에 착륙할 수 있다.
protocol Landing {
    func land(on: LandingSiteHaving) -> String
}

// 페이로드가 있다.
protocol PayloadHaving {
    var payload: String { get }
}

// 차량에서 페이로드를 가져올 수 있다 (예. Canadaarm을 통해).
protocol PayloadFetching {
    func fetchPayload(vehicle: PayloadHaving) -> String
}

final class InternationalSpaceStation: PayloadFetching {

    // ⚠️ 우주 정거장은 SpaceXCRS8의 착륙 능력에 대해 전혀 모른다.
    func fetchPayload(vehicle: PayloadHaving) -> String {
        return "Deployed \(vehicle.payload) at April 10, 2016, 11:23 UTC"
    }
}

// 바지선 - 착륙 지점이 있다 (well, you get the idea).
final class OfCourseIStillLoveYouBarge: LandingSiteHaving {
    let landingSite = "a barge on the Atlantic Ocean"
}

// 페이로드가 있고 착륙 지점이 있는 곳에 착륙할 수 있다.
// 매우 제한된 우주 비행체라는 것을 안다.
final class SpaceXCRS8: Landing, PayloadHaving {

    let payload = "BEAM and some Cube Sats"

    // ⚠️ CRS8 은 착륙지 정보만 알고 있다.
    func land(on: LandingSiteHaving) -> String {
        return "Landed on \(on.landingSite) at April 8, 2016 20:52 UTC"
    }
}

let crs8 = SpaceXCRS8()
let barge = OfCourseIStillLoveYouBarge()
let spaceStation = InternationalSpaceStation()

spaceStation.fetchPayload(vehicle: crs8)
crs8.land(on: barge)
```
</div>
</details>

- `SpaceXCRS8` : 우주 비행체 [https://en.wikipedia.org/wiki/SpaceX_CRS-8](https://en.wikipedia.org/wiki/SpaceX_CRS-8)
    - 착륙 기능과 화물 탑재 기능을 가진다.
- `InternationalSpaceStation` : 국제 우주 정거장(ISS)
    - 화물을 가져오는 기능을 가진다.
- `OfCourseIStillLoveYouBarge` : 재사용 가능한 궤도 발사체를 위한 부유식 착륙 플랫폼으로 무인 수상 차량이다. [https://en.wikipedia.org/wiki/Autonomous_spaceport_drone_ship](https://en.wikipedia.org/wiki/Autonomous_spaceport_drone_ship)
    - 착륙 가능한 영역(지점)을 가진다.

## 추가 질문

### 퍼사드 패턴?

- 여러 개의 인터페이스를 통합하는 한 개의 인터페이스를 제공한다.
- 서브시스템이 너무 많고 사용하기가 복잡한 문제를 해결하기 위해 단순한 인터페이스를 중간에 넣음으로써 서브시스템을 쉽게 사용할 수 있다.
- 복잡성이 더 간단한 방법으로 포장되어 있기 때문에 복잡한 API를 더 쉽게 읽고, 이해하고, 사용할 수 있게 해준다.
- 해당 API의 내부에 대한 의존성을 줄임으로, 최종 결과가 동일한 작업을 수행하는 한 자유롭게 변경할 수 있습니다.

<img src=https://user-images.githubusercontent.com/31722496/199674628-f543b403-527d-4050-9c68-45c01e118d28.png width=70%>

- 애플의 플랫폼에는 퍼사드 패턴의 많은 예가 있다.
- 애플의 주요 프레임워크의 구조 자체가 바로 이 패턴을 기반으로 한다
    - [다윈 시스템](https://ko.wikipedia.org/wiki/%EB%8B%A4%EC%9C%88_(%EC%9A%B4%EC%98%81_%EC%B2%B4%EC%A0%9C))은 운영 체제의 핵심 구성 요소이다.
    - 그 위에 `Core Foundation`과 `Foundation`이 있다.
    - 그 위에는 `Core Audio`, `Core Video`, `Core Text`, `Metal`, `OpenGL` 등이 있습니다.
    - 그 위에는 `AVFoundation`, `MetalKit`, `SceneKit`, `SpriteKit`, `Text Kit` 등이 있습니다.

	<img src=https://user-images.githubusercontent.com/31722496/199674798-90da484d-a16f-472b-978c-cbb9f04ffce2.png>

- Kit layer에 도달하더라도, 애플은 여전히 우리의 삶을 더 쉽게 만들기 위해 퍼사드를 만듭니다.
- 대부분의 `UIKit` 컨트롤은 그리 어렵지 않습니다.
- 필요하다면 `UIButton`을 처음부터 만드는 것을 상상할 수 있습니다. 하지만 `UITextView`는?
- 그것은 심각한 작업입니다
    - 여러 줄에 걸쳐 스크롤 텍스트를 렌더링할 수 있고, 사용자가 편집할 수 있고(포맷 추가 포함), 사용자가 선택하고, 복사하고, 붙여넣을 수 있도록 하는 등
- 이것들은 우리가 당연하게 여기는 모든 것이며, `UITextView`의 존재는 `Text Kit`를 직접 만지는 대신 간단히 노출된 API에 집중할 수 있게 해준다.
- 퍼사드의 또 다른 미묘한 예는 `UIImage`입니다. 디스크에서 비트맵 기반(JPEG 또는 PNG)이미지를 로드하게 한 다음 화면에 렌더링할 수 있습니다. 그러나 또한 벡터 기반(PDF) 파일에서 벡터 삽화를 로드하도록 할 수도 있습니다. 벡터 아트는 해상도에 의존하지 않으며, 이것은 `UIImage`가 자동으로 적응하여 기기의 retina 스케일을 고려하여 사용하는 크기에 상관없이 올바른 해상도로 그려지도록 한다는 것을 의미합니다. **→ 이미지를 로드하고 사용하기 위한 통합 인터페이스를 제공한다는 것**
아마도 내가 모든 애플 파사드 중에서 가장 좋아하는 것은 iOS 7 이전에 많이 사용되었지만 그 이후로 거의 잊혀진 것: `UIColor`이다. 오늘날 우리 모두는 그것을 regular type으로 사용하지만, 그것은 사실 퍼사드이고 많은 사람들이 그것의 static property `scrollViewTexturedBackgroundColor`를 ****사용하던 iOS 7 이전에 일반적으로 사용되었다. 속성 자체를 Swift에서 항상 사용할 수 없음에도 불구하고 Interface Builder의 색상 선택 드롭다운에서 여전히 볼 수 있습니다.
- 이 속성을 사용하는 iOS 7 이전에는 솔리드 컬러보다는 일종의 스크래치 메탈 텍스처를 렌더링했으며 스크롤 View 뒤에서 너무 많이 축소하거나 스크롤했다는 것을 나타내기 위해 사용되었습니다. 물론 애플은 모든 뷰에 추가 `backgroundImage` 속성을 추가하여 API를 복잡하게 만들고 싶지 않았기 때문에 대신 필요한 것에 따라 이미지나 색상을 저장할 수 있는 `UIColor` 퍼사드를 만들었습니다.
스크롤 뷰 텍스처 속성을 사용할 수 없지만 `UIColor`는 여전히 `patternImage` initializer를 사용하여 이미지로 사용할 수 있습니다. 예를 들어 뷰의 배경으로 타일을 지정할 작은 체크무늬 이미지가 있는 경우 `UIColor`만 사용하여 다음과 같이 작업을 수행할 수 있는 작업은 다음과 같이 하십시오.
    
    ```swift
    if let checkers = UIImage(named: "checkers") {
       view.backgroundColor = UIColor(patternImage: checkers)
    }
    ```
    
    **Convenience initializers**
    
- 전용 퍼사드를 사용하는 것보다 복잡한 기능을 포장하는 가장 좋은 방법은 편의성 이니셜라이저를 사용하는 것입니다. 퍼사드의 목표는
    1. API를 사용하기 쉽게 만들거나 
    2. 내부로 손을 대는 기능을 제한하거나 제거하는 것입니다. 
- 편의성 이니셜라이저의 경우 두 가지 중 첫 번째를 쉽게 수행할 수 있습니다.
이것의 좋은 예는 macOS의 `NSTextField` 구성 요소이다. 정적 텍스트에는 `UILabel`을, 편집 가능한 텍스트에는 `UITextField`를 사용하는 데 익숙하기 전에 `UIKit` 코드를 작성한 적이 있지만, macOS에는 그런 구분이 존재하지 않아  `NSTextField`가 두 가지 책임을 모두 처리합니다. 예를 들어 파인더를 사용하면 레이블을 클릭하여 디렉터리 이름을 바꿀 수 있으므로 macOS에서 여러 종류의 레이블을 즉시 편집할 수 있으므로 유용합니다.
- macOS 10.12 (시에라)는 `NSTextField`를 사용하기 훨씬 더 쉽게 만들었으며 개발자들이 신경 쓰지 않는 모든 종류의 내부 초기화기를 숨겼다.
macOS 시에라 이전에는 이런 종류의 코드가 일반적이었다
    
    ```swift
    let label = NSTextField(frame: .zero)
    label.textColor = NSColor.labelColor()
    label.font = NSFont.systemFontOfSize(0)
    label.bezeled = false
    label.bordered = false
    label.drawsBackground = false
    label.editable = false
    label.selectable = false
    label.stringValue = "Hello, world!"
    ```
    
- 시에라 이후에 이렇게 쓴다
    
    ```swift
    let label = NSTextField(labelWithString: "Hello, world!")
    ```
    
- 직접 조작하려면 동일한 속성이 모두 노출되지만, 새로운 편의 이니셜라이저가 거의 모든 작업을 제거합니다.

> 퍼사드 패턴은 복잡하거나 불쾌한 작업을 더 똑똑하고 간단한 인터페이스로 포장할 수 있게 해줍니다.
그렇게 함으로써 개발자들은 우리의 API를 더 쉽게 사용할 수 있을 뿐만 아니라, 우리의 구현의 세부 사항이 숨겨져 있기 때문에 미래에 마음대로 리팩토링할 수 있습니다.
> 

[https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/CocoaFundamentals/CocoaDesignPatterns/CocoaDesignPatterns.html#//apple_ref/doc/uid/TP40002974-CH6-SW47](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/CocoaFundamentals/CocoaDesignPatterns/CocoaDesignPatterns.html#//apple_ref/doc/uid/TP40002974-CH6-SW47)