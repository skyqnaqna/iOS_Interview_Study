# Collection과 Sequence

# 답변

> Sequence는 순차적이고 반복적인 접근을 제공하는 프로토콜이며,
Collection은 인덱싱된 첨자로 접근할 수 있는 Sequence입니다.
> 

# 내용

## Sequence

> Sequence는 요소들의 목록으로 **무한하거나 유한**하며
반복적인 접근이 가능하지만 한 번 이상 반복될지는 장담할 수 없습니다.
> 

```swift
for element in sequence {
    if ... some condition { break }
}

for element in sequence {
    // No defined behavior
}
```

- for-in 반복문으로 Sequence 타입의 요소를 반복 접근할 수 있습니다.
- 하지만 위 코드에서 첫 번째 반복문에서 중단되고 두 번째 반복문에서 다시 처음부터 시작할지 중단된 부분부터 시작할지 보장되지 않습니다.

### Iterator Protocol

> Sequence는 반복 과정을 추적하고 한 번에 하나의 요소를 반환하는 반복자를 생성하여 요소에 대한 접근을 제공합니다.
for-in Loop를 사용할 때 Swift는 Sequence나 Collection의 반복자를 내부적으로 사용합니다.
> 

```swift
public protocol IteratorProtocol<Element> {
	associatedtype Element

	mutating func next() -> Element?
}
```

- next() 메서드는 연관된 시퀀스에서 한 단계 전진하고 시퀀스가 끝나면 nil을 반환합니다.

```swift
struct Countdown: Sequence {
  let start: Int

  func makeIterator() -> CountdownIterator {
    return CountdownIterator(self)
  }
}
```

- Countdown이라는 Sequence는 start가 0이 될 때까지 반복할 수 있도록 구현할겁니다.
- makeIterator()는 커스텀 타입인 CountdownIterator를 반환하고 있습니다.

```swift
struct CountdownIterator: IteratorProtocol {
  let countdown: Countdown
  var times = 0

  init(_ countdown: Countdown) {
    self.countdown = countdown
  }

  mutating func next() -> Int? {
    let nextNumber = countdown.start - times
    guard nextNumber > 0 else { return nil }

    times += 1
    return nextNumber
  }
}
```

- CountdownIterator는 Countdown 시퀀스와 값이 반환된 횟수를 추적합니다.
- CountdownIterator 인스턴스에서 next() 메서드가 호출될 때마다 새로운 nextNumber를 계산하고
0인지 확인하여 다음 숫자를 반환하거나 nil을 반환합니다. → 시퀀스의 끝에 도달하여 반환을 마치는 것
- 만약 0인지 확인하는 과정이 없다면?? **무한 반복**하게 될 것입니다.

```swift
let five = Countdown(start: 5)

for count in five {
  print(count)
}

5
4
3
2
1
```

- next() 메서드를 반복적으로 호출하면 시퀀스의 모든 요소가 순서대로 반환됩니다.

```swift
let five = Countdown(start: 5)

var fiveIterator = five.makeIterator()

while let num = fiveIterator.next() {
  print(num)
}

5
4
3
2
1
```

- 만약 iterator의 다른 복사본이 next() 메서드를 호출하여 진행된 경우 이 메서드를 호출하면 안 됩니다.
- 시퀀스는 O(1) 시간에 iterator를 제공해야 한다.
- 시퀀스는 요소 엑세스에 대한 요구사항이 없으므로 시퀀스를 통과하는 루틴은 O(n)이어야 한다.

```swift
let five = Countdown(start: 5)

var a = five.makeIterator()
var b = five.makeIterator()

while let num = a.next() {
  if num == 3 { break }
  print("a:", num)
}

while let num = a.next() {
  print("a:", num)

  _ = b.next()
}

while let num = b.next() {
  print("b:", num)
}

a: 5
a: 4

a: 2
a: 1

b: 3
b: 2
b: 1
```

### Sequence Protocol

```swift
public protocol Sequence<Element> {

	// 시퀀스의 요소를 나타내는 타입입니다.
	associatedtype Element
	// 시퀀스의 반복 인터페이스를 제공하고 반복 상태를 캡슐화하는 타입입니다.
	associatedtype Iterator: IteratorProtocol where Iterator.Element == Element

	__consuming func makeIterator() -> Iterator

	var underestimatedCount: Int { get }

	func _customContainsEquatableElement(
    _ element: Element
  ) -> Bool?

	__consuming func _copyToContiguousArray() -> ContiguousArray<Element>

	__consuming func _copyContents(
    initializing ptr: UnsafeMutableBufferPointer<Element>
  ) -> (Iterator,UnsafeMutableBufferPointer<Element>.Index)

	func withContiguousStorageIfAvailable<R>(
    _ body: (_ buffer: UnsafeBufferPointer<Element>) throws -> R
  ) rethrows -> R?
}
...
```

## Collection

> Collection은 Sequence 프로토콜을 준수하며 Sequence가 가진 문제점을 해결한 프로토콜입니다.
요소를 인덱싱된 첨자로 접근할 수 있습니다.
> 
- 모든 Collection은 **유한한 요소**로 구성되어 있습니다.
- 원하는 만큼 반복할 수 있습니다.
- 컬렉션은 표준 라이브러리 전체에서 광범위하게 사용됩니다. (Arrays, Dictionaries 등)
- 컬렉션의 endIndex 속성을 제외한 유효한 인덱스를 사용한 첨자를 통해 컬렉션 요소에 접근할 수 있습니다.

```swift
let text = "Sequence and Collection"

print(text.first) // Optional("S")

let firstChar = text[text.startIndex]

print(firstChar) // S
```

### IndexingIterator

- IndexingIterator는 **인덱스를 사용하여 컬렉션에서 반복**하는 타입입니다.
- 이 타입은 자신의 컬렉션을 선언하지 않는(?) 컬렉션에 대한 **기본 반복자**입니다.
- 컬렉션의 인덱스를 사용하여 각 값을 단계별로 지정하여 **반복자 역할**을 합니다.
- 표준 라이브러리에 있는 대부분의 컬렉션은 IndexingIterator를 반복자로 사용합니다.

```swift
@frozen
public struct IndexingIterator<Elements: Collection> {
  @usableFromInline
  internal let _elements: Elements
  @usableFromInline
  internal var _position: Elements.Index

  /// Creates an iterator over the given collection.
  @inlinable
  @inline(__always)
  public /// @testable
  init(_elements: Elements) {
    self._elements = _elements
    self._position = _elements.startIndex
  }

  /// Creates an iterator over the given collection.
  @inlinable
  @inline(__always)
  public /// @testable
  init(_elements: Elements, _position: Elements.Index) {
    self._elements = _elements
    self._position = _position
  }
}
```

- 기본적으로 사용자가 만드는 커스텀 컬렉션은 IndexingIterator 인스턴스를 반환하는 makeIterator() 메서드를 상속하므로 자신의 인스턴스를 선언할 필요가 없습니다.
- 커스텀 컬렉션을 만들 땐, 시작 및 끝 인덱스와 요소에 접근하기 위한 첨자와 같은 컬렉션 프로토콜의 최소 요구사항만 추가하면 됩니다.
이러한 요소를 정의하면 상속된 makeIterator() 메서드는 시퀀스 프로토콜의 요구 사항을 충족합니다.

```swift
struct Pair<Element>: Collection {
  let elements: (Element, Element)

  init(_ first: Element, _ second: Element) {
    self.elements = (first, second)
  }

  var startIndex: Int { return 0 }
  var endIndex: Int { return 2 }

  subscript(index: Int) -> Element {
    switch index {
    case 0: return elements.0
    case 1: return elements.1
    default: fatalError("Index out of bounds.")
    }
  }

  func index(after i: Int) -> Int {
    precondition(i < endIndex, "Can't advance beyond endIndex")
    return i + 1
  }
}

let point = Pair(20, 23)

for element in point {
  print(element)
}
// 20
// 23

print(point[0], point[1]) // 20 23
```

- makeIterator() 메서드나 Iterator 관련 타입을 정의하지 않았기 때문에 기본 Iterator인 IndexingIterator를 사용합니다. 이 예제는 좌표 값을 가지는 인스턴스를 생성하여 for-in loop를 통해 반복하는 것을 보여줍니다.

```swift
extension IndexingIterator: IteratorProtocol, Sequence {
  public typealias Element = Elements.Element
  public typealias Iterator = IndexingIterator<Elements>
  public typealias SubSequence = AnySequence<Element>

	@inlinable
  @inline(__always)
  public mutating func next() -> Elements.Element? {
    if _position == _elements.endIndex { return nil }
    let element = _elements[_position]
    _elements.formIndex(after: &_position)
    return element
  }
}
```

- 다음 요소로 이동하여 반환하거나 없으면 nil을 반환합니다.
- 이 메서드를 반복적으로 호출하면  기본 시퀀스의 모든 요소가 순서대로 반환됩니다.
모든 요소를 반환하고 호출하면 nil을 반환합니다.
- _position이 _elements의 endIndex인지 먼저 확인하는 것을 볼 수 있습니다.

### Collection Protocol

```swift
public protocol Collection<Element>: Sequence {
	override associatedtype Element

	// 콜렉션에서 위치를 나타내는 타입입니다.
	// 유효한 인덱스는 모든 요소의 위치와 첨자로 사용할 수 없는 끝을 지난 위치로 구성됩니다.
	associatedtype Index: Comparable

	// 비어 있지 않은 컬렉션에서 첫 번째 요소의 위치
	var startIndex: Index { get }

	var endIndex: Index { get }

	// 컬렉션의 반복 인터페이스를 제공하고 반복 상태를 캡슐화하는 타입입니다.
	// 기본적으로 컬렉션에 연관된 Iterator 타입으로 IndexingIterator를 제공하여
	// 시퀀스 프로토콜을 준수합니다.
	associatedtype Iterator = IndexingIterator<Self>

	// 이 컬렉션 요소의 인접 하위 범위를 나타내는 컬렉션입니다.
	// 서브시퀀스는 오리지널 컬렉션과 인덱스를 공유합니다.
	// 기본 서브시퀀스 타입은 Slice 입니다.
	associatedtype SubSequence: Collection = Slice<Self>
  where SubSequence.Index == Index,
        Element == SubSequence.Element,
        SubSequence.SubSequence == SubSequence

	@_borrowed
  subscript(position: Index) -> Element { get }
	subscript(bounds: Range<Index>) -> SubSequence { get }

	// 컬렉션을 서브스크립트하는데 유효한 인덱스를 오름차순으로 나타내는 타입입니다.
	associatedtype Indices: Collection = DefaultIndices<Self>
    where Indices.Element == Index, 
          Indices.Index == Index,
          Indices.SubSequence == Indices

	var indices: Indices { get }

	// 컬렉션이 비어 있는지 여부를 나타내는 부울 값
	// 컬렉션이 비어 있는지 확인할 때 `count` 속성이 0인지 확인하는 것 대신 
	// 'isEmpty` 속성을 사용하세요.
	// 'RandomAccessCollection'을 준수하지 않는 컬렉션은 'count' 속성이
	// 컬렉션 요소를 반복합니다.
	// 복잡도: O(1)
	var isEmpty: Bool { get }

	// 컬렉션 요소의 개수
	// 복잡도: O(1) -> 'RandomAccessCollection'을 준수하는 경우. 그렇지 않으면 O(n)
	var count: Int { get }

	func _customIndexOfEquatableElement(_ element: Element) -> Index??

	func _customLastIndexOfEquatableElement(_ element: Element) -> Index??

	func index(_ i: Index, offsetBy distance: Int) -> Index

	func index(
    _ i: Index, offsetBy distance: Int, limitedBy limit: Index
  ) -> Index?

	func distance(from start: Index, to end: Index) -> Int

	func _failEarlyRangeCheck(_ index: Index, bounds: Range<Index>)
	
  func _failEarlyRangeCheck(_ index: Index, bounds: ClosedRange<Index>)

	func _failEarlyRangeCheck(_ range: Range<Index>, bounds: Range<Index>)

	func index(after i: Index) -> Index

	func formIndex(after i: inout Index)
}
```

**endIndex**

```swift
var endIndex: Index { get }
```

- 컬렉션의 끝을 지난 위치 즉, 마지막 유효 첨자보다 1 큰 위치 (마지막 요소 다음 위치)
- 컬렉션의 마지막 요소를 포함하는 범위가 필요하면 반열림 범위 연산자 ’..<’ 를 사용합니다.
- '..<' 연산자는 상한을 포함하지 않으므로 항상 endIndex와 사용하는 것이 안전합니다.
- 만약 컬렉션이 비어 있다면 endIndex는 startIndex와 같습니다.

```swift
let numbers = [1, 2, 3, 4, 5]
if let idx = numbers.firstIndex(of: 3) {
  print(numbers[idx ..< numbers.endIndex]) // [3, 4, 5]
}
```

**subscript - position**

```swift
subscript(position: Index) -> Element { get }
```

- 특정 위치의 요소에 접근합니다.
- 컬렉션의 끝 인덱스를 제외한 모든 유효한 인덱스로 컬렉션을 서브스크립트할 수 있습니다.
- 끝 인덱스는 마지막 요소를 지나는 위치를 참조하기 때문에
- position 파라미터: 접근할 요소의 위치입니다.
- position은 endIndex 속성과 다른 유효한 인덱스여야 합니다.
- 복잡도: O(1)

```swift
let numbers = [1, 2, 3, 4, 5]

print(numbers[2]) // 3
```

**subscript - bounds**

```swift
subscript(bounds: Range<Index>) -> SubSequence { get }
```

- 컬렉션 요소의 인접 하위 범위에 접근합니다.
- 배열에서 'PartialRangeFrom' 범위 표현식을 사용하면 범위 표현식의 시작부터 배열의 끝까지 하위 범위에 접근합니다.
- 접근된 슬라이스는 원래 컬렉션과 동일한 요소에 대해 동일한 인덱스를 사용합니다.
- 인덱스가 특정 값에서 시작한다고 가정하는 대신 항상 슬라이스의 startIndex 속성을 사용합니다.
- 슬라이스 범위를 벗어나는 인덱스를 사용하여 접근하려고 하면 해당 인덱스가 원래 컬렉션에서 유효하더라도 런타임 오류가 발생할 수 있습니다.
- bounds 파라미터: 컬렉션의 인덱스 범위입니다.
- 범위의 경계는 컬렉션의 유효한 인덱스여야 합니다.
- 복잡도: O(1)

```swift
let numbers = [1, 2, 3, 4, 5]

let numbersSlice = numbers[2..<5]
print(numbersSlice) // [3, 4, 5]

let idx = numbersSlice.firstIndex(of: 3)! // 2
print(numbers[idx]) // 3

print(numbersSlice.startIndex) // 2
print(numbersSlice[2]) // 3
print(numbersSlice[0]) // Fatal error: Index out of bounds
```

**indices**

```swift
var indices: Indices { get }
```

- 컬렉션을 서브스크립트하는데 유효한 인덱스는 오름차순입니다.
- 켈렉션의 인덱스 속성은 컬렉션 자체에 대한 강력한 참조를 보유할 수 있으므로 컬렉션이 고유하지 않게 참조됩니다. (???)
- 인덱스를 반복하는 동안 컬렉션을 수정하면 강력한 참조로 인해 예기치 않은 컬렉션 복사본이 생성될 수 있습니다. (???)
- 예기치 않은 복사를 방지하려면 'index(after:)'메서드를 'startIndex'로 시작하여 대신 인덱스를 생성하세요.

```swift
let numbers = [1, 2, 3, 4, 5]

var nums = numbers
var i = nums.startIndex
while i != nums.endIndex {
  nums[i] *= 10
  i = nums.index(after: i)
}

print(nums == [10, 20, 30, 40, 50]) // true
```

## 결론

시퀀스의 **무한한 요소**를 가질 수 있는 문제점을 컬렉션에서 **유한한 요소**를 가지도록 해결한다.

인덱싱된 위치에 접근 가능하며 **시작**과 **끝** 인덱스를 가진다.

유한한 요소를 가지며 시작과 끝이 있기 때문에 **얼마든지 반복**할 수 있다. → 시퀀스의 한 번 이상의 반복을 보장하지 못하는 문제를 해결


# 추가 질문

## Associated Type(연관된 타입)?

프로토콜을 정의할 때 정의 부분에 사용되는 타입에 대한 placeholder를 제공합니다.

해당 프로토콜이 채택되기 전까지 실제 타입이 결정되지 않습니다. → 프로토콜 정의 부분에서 사용하는 제너릭

```swift
protocol Container {
    associatedtype Item
    mutating func append(_ item: Item)
    var count: Int { get }
    subscript(i: Int) -> Item { get }
}
```

- Container 프로토콜은 Item이라는 연관된 타입을 선언하였고, 
append(_:)메서드와 count 프로퍼티, subscript를 정의합니다.

```swift
struct IntStack: Container {
    // original IntStack implementation
    var items: [Int] = []
    mutating func push(_ item: Int) {
        items.append(item)
    }
    mutating func pop() -> Int {
        return items.removeLast()
    }
    // conformance to the Container protocol
    typealias Item = Int
    mutating func append(_ item: Int) {
        self.push(item)
    }
    var count: Int {
        return items.count
    }
    subscript(i: Int) -> Int {
        return items[i]
    }
}
```

- Container를 준수하는 IntStack은 Item을 Int타입으로 지정했습니다.
- `typealias Item = Int` 는 Container 프로토콜의 추상적 타입인 Item에서 구체적인 타입인 Int로 변환합니다.
- 위 예제에서는 append(_:) 메서드의 파라미터 타입과 서브스크립트의 반환 타입에서 Item 타입을 유추할 수 있기 때문에 `typealias Item = Int` 라인을 지워도 동작에는 이상이 없습니다.

- Container 프로토콜을 준수하는 제너릭 Stack 타입도 만들 수 있습니다.

```swift
struct Stack<Element>: Container {
    // original Stack<Element> implementation
    var items: [Element] = []
    mutating func push(_ item: Element) {
        items.append(item)
    }
    mutating func pop() -> Element {
        return items.removeLast()
    }
    // conformance to the Container protocol
    mutating func append(_ item: Element) {
        self.push(item)
    }
    var count: Int {
        return items.count
    }
    subscript(i: Int) -> Element {
        return items[i]
    }
}
```

- Element 타입이 append(_:) 메서드의 파라미터와 서브스크립트 반환 타입으로 사용되기 때문에 Item으로 사용하는 타입이라고 유추할 수 있습니다.

### 연관된 타입에 제약 추가

제약조건을 충족하는 타입을 요구하기 위해 프로토콜의 연관된 타입에 제약을 추가할 수 있습니다.

```swift
protocol Container {
    associatedtype Item: Equatable
    mutating func append(_ item: Item)
    var count: Int { get }
    subscript(i: Int) -> Item { get }
}
```

- Container를 준수하려면 Item 타입은 Equatable 프로토콜을 준수해야 합니다.

### 연관된 타입의 제약조건에서 프로토콜 사용

프로토콜은 자신의 요구사항의 일부로 나타낼 수 있습니다. 

suffix(_:) 메서드는 컨테이너 끝에서 size개수만큼의 요소를 저장한 Suffix 타입의 인스턴스를 반환합니다.

```swift
protocol SuffixableContainer: Container {
    associatedtype Suffix: SuffixableContainer where Suffix.Item == Item
    func suffix(_ size: Int) -> Suffix
}
```

- Suffix라는 연관 타입은 SuffixableContainer 프로토콜을 준수해야 하고, Item 타입은 컨테이너의 Item 타입과 동일해야 합니다.

```swift
extension Stack: SuffixableContainer {
    func suffix(_ size: Int) -> Stack {
        var result = Stack()
        for index in (count-size)..<count {
            result.append(self[index])
        }
        return result
    }
    // Inferred that Suffix is Stack.
}
var stackOfInts = Stack<Int>()
stackOfInts.append(10)
stackOfInts.append(20)
stackOfInts.append(30)
let suffix = stackOfInts.suffix(2)
// suffix contains 20 and 30
```

- Stack 타입의 SuffixableContainer 프로토콜 확장
- Stack에 대한 Suffix 연관된 타입이 Stack이므로 반환되는 것도 Stack입니다.
- 하지만 컨테이너의 Item 타입이 같으면 되기 때문에 Stack이 아닌 다른 타입을 Suffix 타입으로 가질 수 있습니다.

```swift
extension IntStack: SuffixableContainer {
    func suffix(_ size: Int) -> Stack<Int> {
        var result = Stack<Int>()
        for index in (count-size)..<count {
            result.append(self[index])
        }
        return result
    }
    // Inferred that Suffix is Stack<Int>.
}
```

- IntStack의 Suffix 타입은 Stack<Int>입니다.

### 제너릭 Where 절이 있는 연관된 타입

연관된 타입에 제너릭 where 절을 포함할 수 있습니다.

예를 들어 Sequence 프로토콜처럼 반복자를 포함하는 Container를 만든다면 다음과 같습니다.

```swift
protocol Container {
    associatedtype Item
    mutating func append(_ item: Item)
    var count: Int { get }
    subscript(i: Int) -> Item { get }

    associatedtype Iterator: IteratorProtocol where Iterator.Element == Item
    func makeIterator() -> Iterator
}
```

- Iterator의 제너릭 where 절은 반복자의 타입에 상관없이 컨테이너의 요소와 동일한 타입의 요소를 탐색해야 한다는 것을 요구합니다.
- 다른 프로토콜을 상속하는 프로토콜의 경우 프로토콜 선언에 제너릭 where 절을 사용하여 상속된 연관 타입에 제약조건을 추가합니다.

```swift
protocol ComparableContainer: Container where Item: Comparable { }
```

- 위 코드는 Item이 Comparable을 준수하도록 요구하는 comparableContainer 프로토콜을 선언합니다.

## Collection을 상속하는 Collection들…?

<img src=https://user-images.githubusercontent.com/31722496/211301920-eb1f7a08-af5b-41fd-acfb-fa103b626f9e.png width=50%>

- BidirectionalCollection : 양방향(앞, 뒤) 이동을 지원하는 컬렉션
- LazyCollectionProtocol : map과 filter같이 사용 빈도가 높은 연산이 lazy하게 구현된 컬렉션
- MutableCollection : 서브스크립트 할당을 지원하는 컬렉션
- RandomAccessCollection : 효율적인 Random-Access 인덱스 순회를 제공하는 컬렉션
- RangeReplaceableCollection : 임의의 하위 범위 요소를 다른 컬렉션의 요소로 교체할 수 있는 컬렉션
- StringProtocol : 문자열을 문자 모음으로 나타낼 수 있는 타입

# 참고 자료

[https://academy.realm.io/kr/posts/try-swift-soroush-khanlou-sequence-collection/](https://academy.realm.io/kr/posts/try-swift-soroush-khanlou-sequence-collection/)

[https://developer.apple.com/documentation/swift/sequence](https://developer.apple.com/documentation/swift/sequence)

[https://developer.apple.com/documentation/swift/collection](https://developer.apple.com/documentation/swift/collection)

[https://github.com/apple/swift/blob/main/stdlib/public/core/Sequence.swift](https://github.com/apple/swift/blob/main/stdlib/public/core/Sequence.swift)
