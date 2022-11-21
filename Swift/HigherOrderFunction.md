# 고차함수(Higher-Order function)?


# 답변

> **함수**를 **매개변수**로 갖거나 **반환**하는 함수를 고차함수라고 한다.
> 

# 내용

- 스위프트는 함수를 [일급 객체](https://ko.wikipedia.org/wiki/%EC%9D%BC%EA%B8%89_%EA%B0%9D%EC%B2%B4)로 취급하기 때문에 함수를 전달인자나 반환 타입으로 사용할 수 있다.
- 스위프트에서 대표적인 고차함수로는 Map, Filter, Reduce 가 있다.
    - `Sequence`, `Collection` 프로토콜을 준수하는 타입은 모두 사용 가능
- 반복문을 사용한 코드보다 간결한 코드를 작성할 수 있다.

## Map

> 컨테이너 내부 데이터를 변형시켜 새로운 컨테이너에 담아서 반환한다.
> 

```swift
let strNums = ["1", "2", "3", "4", "5"]

let intNums = strNums.map { Int($0)! }

print(intNums) // [1, 2, 3, 4, 5]

let doubledNums = intNums.map { $0 * 2 }

print(doubledNums) // [2, 4, 6, 8, 10]
```

- 클로저 표현을 간략화했기 때문에 코드가 간결해졌다.

```swift
let intNums0 = strNums.map({ (strNum: String) -> Int in
  return Int(strNum)!
})

// 매개변수와 반환 타입 생략
let intNums1 = strNums.map({ return Int($0)! })

// return 키워드 생략
let intNums2 = strNums.map({ Int($0)! })

// 후행 클로저 사용
let intNums3 = strNums.map { Int($0)! }
```

- [`Sequence`에 정의된 Map](https://github.com/apple/swift/blob/103b4a89c225915214d36ba350900132ef335161/stdlib/public/core/Sequence.swift#L653)

```swift
@inlinable
  public func map<T>(
    _ transform: (Element) throws -> T
  ) rethrows -> [T] {
    let initialCapacity: Int = underestimatedCount
    var result: ContiguousArray<T> = ContiguousArray<T>()
    result.reserveCapacity(initialCapacity)

    var iterator: Self.Iterator = self.makeIterator()

    // Add elements up to the initial capacity without checking for regrowth.
    for _ in 0..<initialCapacity {
      result.append(try transform(iterator.next()!))
    }
    // Add remaining elements, if any.
    while let element: Self.Element = iterator.next() {
      result.append(try transform(element))
    }
    return Array(result)
  }
```

- [`Collection`에 정의된 Map](https://github.com/apple/swift/blob/103b4a89c225915214d36ba350900132ef335161/stdlib/public/core/Collection.swift#L1174)

```swift
@inlinable
  public func map<T>(
    _ transform: (Element) throws -> T
  ) rethrows -> [T] {
    // TODO: swift-3-indexing-model - review the following
    let n: Int = self.count
    if n == 0 {
      return []
    }

    var result: ContiguousArray<T> = ContiguousArray<T>()
    result.reserveCapacity(n)

    var i: Self.Index = self.startIndex

    for _ in 0..<n {
      result.append(try transform(self[i]))
      formIndex(after: &i)
    }

    _expectEnd(of: self, is: i)
    return Array(result)
  }
```

## CompactMap

> 컨테이너 내부 요소를 변형시키는데, **nil이 아닌 요소**만 새로운 컨테이너에 담아 반환한다.
> 

```swift
var strNums = ["1", "2", "#", "4", "5"]

let mapped = strNums.map { Int($0) }
let compactMapped = strNums.compactMap { Int($0) }

print(mapped) // [Optional(1), Optional(2), nil, Optional(4), Optional(5)]
print(compactMapped) // [1, 2, 4, 5]
```

- [`SequenceAlgorithms`에 정의된 **CompactMap**](https://github.com/apple/swift/blob/103b4a89c225915214d36ba350900132ef335161/stdlib/public/core/SequenceAlgorithms.swift#L768)

```swift
@inlinable // protocol-only
  @inline(__always)
  public func _compactMap<ElementOfResult>(
    _ transform: (Element) throws -> ElementOfResult?
  ) rethrows -> [ElementOfResult] {
    var result: [ElementOfResult] = []
    for element in self {
      if let newElement = try transform(element) {
        result.append(newElement)
      }
    }
    return result
  }
```

- `if-let` **옵셔널 바인딩**으로 옵셔널 값을 추출하는 것을 확인할 수 있다.
    - 상단 예제 코드에서 Map 결과는 옵셔널이지만 CompactMap은 아닌 이유이기도 하다.

## FlatMap

> **Sequence** 내부 요소의 요소들을 새로운 1차원 배열에 담는다.
> 
- 2차원 배열이라면 내부 요소는 1차원 배열이고, 그 요소들을 1차원 배열에 담기 때문에 2차원 → 1차원 배열로 펴지게 된다.
- 3차원 배열이라면 내부 요소는 2차원 배열이고, 그 요소들은 1차원 배열이기 때문에 1차원 배열을 1차원 배열에 추가하여 결과는 2차원 배열이 된다.

```swift
let arr2d = [[1, 2], [3, 4, 5], [6, 7, 8, 9]]

let arr = arr2d.flatMap { $0 }
print(arr) // [1, 2, 3, 4, 5, 6, 7, 8, 9]

let arr3d = [[[11, 22, 33], [444, 555], [6666]]]

let flatMappedArr2d = arr3d.flatMap { $0 }
let flatMappedArr1d = arr3d.flatMap { $0 }.flatMap { $0 }

print(flatMappedArr2d) // [[11, 22, 33], [444, 555], [6666]]
print(flatMappedArr1d) // [11, 22, 33, 444, 555, 6666]

let arr1 = ["abc", "def"]

let arr2 = arr1.flatMap { $0 }

print(arr2) // ["a", "b", "c", "d", "e", "f"]
```

<img src=https://user-images.githubusercontent.com/31722496/203112256-51899ff7-bd84-4671-a0ae-3b33f5c8ed3a.png width=40%>

- arr2는 [String.Element] 즉, [Character]이다.
    - String 배열이면 내부 요소 String의 요소는 Character
    
	<img src=https://user-images.githubusercontent.com/31722496/203112272-c4fd21fe-10d3-4633-af23-fc0ed4429773.png width=40%>
    


- [`SequenceAlgorithms`에 정의된 **FlatMap**](https://github.com/apple/swift/blob/103b4a89c225915214d36ba350900132ef335161/stdlib/public/core/SequenceAlgorithms.swift#L729)

```swift
@inlinable
  public func flatMap<SegmentOfResult: Sequence>(
    _ transform: (Element) throws -> SegmentOfResult
  ) rethrows -> [SegmentOfResult.Element] {
    var result: [SegmentOfResult.Element] = []
    for element in self {
      result.append(contentsOf: try transform(element))
    }
    return result
  }
```

- 그럼 **String**은 어떻게 될까?

```swift
let str = "Swift"

let flatMapped = str.flatMap { $0 }
let compactMapped = str.compactMap { $0 }
let mapped = str.map { $0 }

print(flatMapped) // ["S", "w", "i", "f", "t"]
print(compactMapped) // ["S", "w", "i", "f", "t"]
print(mapped) // ["S", "w", "i", "f", "t"]
```

- 이 때 `'flatMap' is deprecated: Please use compactMap(_:) for the case where closure returns an optional value` 라는 경고 문구가 발생한다.
    - **flatMap**대신 **compactMap**을 사용하라고 한다.
- 이 경고문은 옵셔널 타입에 적용할 때도 발생한다.

```swift
let nums: [Int?] = [1, 2, 3, nil, 5]

print(nums.flatMap { $0 }) // [1, 2, 3]
print(nums.compactMap { $0 }) // [1, 2, 3]
```

- 공식 문서에 이렇게 정의된 있는 흔적이 있다.
    - `func flatMap<ElementOfResult>(_ transform: (Self.Element) throws -> ElementOfResult?) rethrows -> [ElementOfResult]`
- Swift 4.1에서 **compactMap**이 나오면서 deprecate됐다.
    - [https://github.com/apple/swift-evolution/blob/main/proposals/0187-introduce-filtermap.md](https://github.com/apple/swift-evolution/blob/main/proposals/0187-introduce-filtermap.md)

## Filter

> 컨테이너 내부 요소를 걸러내어 새로운 컨테이너에 담아서 반환한다.
> 

```swift
let oddNums = intNums.filter { $0 % 2 != 0 }

print(oddNums) // [1, 3, 5]
```

- filter의 매개변수로 전달되는 함수의 반환타입은 `Bool`
    - true인 요소를 새로운 컨테이너에 담는다.

- 클로저 간략화

```swift
let oddNums0 = intNums.filter({ (num: Int) -> Bool in
  return num % 2 != 0
})

let oddNums1 = intNums.filter({ return $0 % 2 != 0 })

let oddNums2 = intNums.filter({ $0 % 2 != 0 })

let oddNums3 = intNums.filter { $0 % 2 != 0 }
```

- [`Sequence`에 정의된 Filter](https://github.com/apple/swift/blob/103b4a89c225915214d36ba350900132ef335161/stdlib/public/core/Sequence.swift#L693)

```swift
@inlinable
  public __consuming func filter(
    _ isIncluded: (Element) throws -> Bool
  ) rethrows -> [Element] {
    return try _filter(isIncluded)
  }

  @_transparent
  public func _filter(
    _ isIncluded: (Element) throws -> Bool
  ) rethrows -> [Element] {

    var result: ContiguousArray<Self.Element> = ContiguousArray<Element>()

    var iterator: Self.Iterator = self.makeIterator()

    while let element: Self.Element = iterator.next() {
      if try isIncluded(element) {
        result.append(element)
      }
    }

    return Array(result)
  }
```

## Reduce

> 컨테이너 내부 요소들을 하나로 결합하여 반환한다.
> 
- 초기값을 설정하고 연산자를 지정한다.

```swift
let sum = intNums.reduce(0, +)

print(sum) // 15
```

- 클로저 간략화

```swift
let sum0 = intNums.reduce(0, { (now: Int, next: Int) -> Int in
  return now + next
})

let sum1 = intNums.reduce(0, { (now, next) in
  return now + next
})

let sum2 = intNums.reduce(0) { return $0 + $1 }

let sum3 = intNums.reduce(0) { $0 + $1 }

let sum4 = intNums.reduce(0, +)
```

- [`SequenceAlgorithms`에 정의된 Reduce](https://github.com/apple/swift/blob/103b4a89c225915214d36ba350900132ef335161/stdlib/public/core/SequenceAlgorithms.swift#L582)

```swift
@inlinable
  public func reduce<Result>(
    _ initialResult: Result,
    _ nextPartialResult:
      (_ partialResult: Result, Element) throws -> Result
  ) rethrows -> Result {
    var accumulator: Result = initialResult
    for element: Self.Element in self {
      accumulator = try nextPartialResult(accumulator, element)
    }
    return accumulator
  }
```

## 연계 사용 (Chaining)

```swift
let result = strNums.map { Int($0)! }.filter { $0 % 2 != 0 }.reduce(0, +)

print(result) // 9
```

## ForEach

> `Sequence`의 각 요소마다 주어진 클로저를 수행한다.
> 

```swift
let strNums = ["1", "2", "3", "4", "5"]

strNums.forEach { print($0) }

/*
1
2
3
4
5
*/
```

- **For-in** 과 다른 점
    - **For-in**은 반복문이고 **ForEach**는 각 요소마다 클로저를 수행하는 고차함수다.
        - 따라서 `break`, `continue`는 **For-in** 에서 잘 동작하지만 **ForEach**에서는 에러가 발생한다.
            - **ForEach**안에 반복문이 있다면 해당 반복문안에서 사용 가능
        - `return` 같은 경우에도 **For-in**은 현재 반복문을 포함하는 함수를 종료하지만, **ForEach**는 고차함수이므로 현재 클로저를 종료하고 다음 요소에 대한 클로저를 수행한다.

- 클로저 간략화

```swift
strNums.forEach({ (num: String) -> Void in
  print(num)
})

strNums.forEach { num in
  print(num)
}

strNums.forEach { print($0) }
```

- [`Sequence` 에 정의된 ForEach](https://github.com/apple/swift/blob/103b4a89c225915214d36ba350900132ef335161/stdlib/public/core/Sequence.swift#L756)

```swift
@_semantics("sequence.forEach")
  @inlinable
  public func forEach(
    _ body: (Element) throws -> Void
  ) rethrows {
    for element in self {
      try body(element)
    }
  }
```

## Sort(by:) & Sorted(by:)

> `by: 매개변수`로 들어오는 조건에 맞게 요소를 정렬한다.
**Sort**는 기존 배열을 정렬하고, **Sorted**는 새로운 배열에 정렬하여 반환한다.
> 
- **Sort**하려는 배열은 `let`이 아닌 `var`여야 가능하다.

```swift
var strNums = ["1", "2", "3", "4", "5"]

let sortedStrNums = strNums.sorted(by: >)
print(strNums) // ["1", "2", "3", "4", "5"]
print(sortedStrNums) // ["5", "4", "3", "2", "1"]

strNums.sort(by: >)
print(strNums) // ["5", "4", "3", "2", "1"]
```

- [**Sort**에 정의된 **Sort**와 **Sorted**](https://github.com/apple/swift/blob/103b4a89c225915214d36ba350900132ef335161/stdlib/public/core/Sort.swift#L14)

```swift
@inlinable
  public mutating func sort(
    by areInIncreasingOrder: (Element, Element) throws -> Bool
  ) rethrows {
    let didSortUnsafeBuffer: Void? =
      try withContiguousMutableStorageIfAvailable { buffer in
        try buffer._stableSortImpl(by: areInIncreasingOrder)
      }
    if didSortUnsafeBuffer == nil {
      // Fallback since we can't use an unsafe buffer: sort into an outside
      // array, then copy elements back in.
      let sortedElements = try sorted(by: areInIncreasingOrder)
      for (i, j) in zip(indices, sortedElements.indices) {
        self[i] = sortedElements[j]
      }
    }
  }

@inlinable
  public func sorted(
    by areInIncreasingOrder:
      (Element, Element) throws -> Bool
  ) rethrows -> [Element] {
    var result = ContiguousArray(self)
    try result.sort(by: areInIncreasingOrder)
    return Array(result)
  }
```

- **Sort**는 기존 배열을 정렬하기 때문에 `mutating` 수식어가 있다.
- **Sort**는 시간 복잡도가 $O(NlogN)$ 이며, **안정 정렬**이다. → **Tim Sort** 사용

- 클로저 간략화

```swift
var strNums = ["1", "2", "3", "4", "5"]

strNums.sort(by: { (a: String, b: String) -> Bool in
  return a > b
}) // ["5", "4", "3", "2", "1"]

strNums.sort(by: { a, b in
  return a < b
}) // ["1", "2", "3", "4", "5"]

strNums.sort(by: { $0 > $1 }) // ["5", "4", "3", "2", "1"]
print(strNums) 

strNums.sort { $0 < $1 } // ["1", "2", "3", "4", "5"]
print(strNums) 

strNums.sort(by: >) // ["5", "4", "3", "2", "1"]

strNums.sort() // ["1", "2", "3", "4", "5"]
```

# 추가 질문

## 반복문과 고차함수의 성능차이가 있을까?

같은 동작을 할 때 반복문 안에서 하는 것이 함수를 여러번 호출하는 것보다 빠를 것이라 예상했지만

[https://medium.com/skoumal-studio/performance-of-built-in-higher-order-function-vs-for-in-loop-in-swift-166fa41b545f](https://medium.com/skoumal-studio/performance-of-built-in-higher-order-function-vs-for-in-loop-in-swift-166fa41b545f)

이 글에선 map, filter는  For-in loop보다 빨랐고

reduce, flatMap, chaining은 반대였다.

최적화 관련된 것인지.. 원인은 추후에 더 조사해봐야겠다.

## 함수형 프로그래밍?

> 함수형 프로그래밍은 함수를 프로그램의 어떤 값이나 상태를 변화시키는 것이 아닌 **수학적 함수의 계산**으로 취급하여 함수의 응용을 강조한 **선언형 프로그래밍** 패러다임 중 하나이다.
> 
- 순수 함수
    - 동일한 input에 대해 항상 같은 output을 반환하기 때문에 side-effect가 없다.
    - 함수 외부에 영향을 끼치지 않기 때문에 thread-safe하고 병렬 처리에 유리하다.
- 선언형 프로그래밍 ←→ 명령형 프로그래밍
    - ‘어떻게’보다는 ‘무엇’을 할 것인지에 포커스
- 함수를 일급 객체로 취급
    - 일급 객체?
        - 전달인자로 전달 가능
        - 동적 프로퍼티 할당 가능
        - 변수나 데이터 구조에 담을 수 있음
        - 반환 값으로 사용 가능
        - 할당할 때 사용된 이름과 관계없이 고유한 객체로 구별 가능

# 참고 자료

[http://www.yes24.com/Product/Goods/78907450](http://www.yes24.com/Product/Goods/78907450)

[https://yagom.net/courses/swift-basic/lessons/고차함수/](https://yagom.net/courses/swift-basic/lessons/%ea%b3%a0%ec%b0%a8%ed%95%a8%ec%88%98/)

[https://sujinnaljin.medium.com/swift-map-파헤치기-4b057834d310](https://sujinnaljin.medium.com/swift-map-%ED%8C%8C%ED%97%A4%EC%B9%98%EA%B8%B0-4b057834d310)

[https://levelup.gitconnected.com/higher-order-functions-in-swift-35861620ad1](https://levelup.gitconnected.com/higher-order-functions-in-swift-35861620ad1)

[https://developer.apple.com/documentation/swift/sequence/flatmap(_:)-jo2y](https://developer.apple.com/documentation/swift/sequence/flatmap(_:)-jo2y)

[https://jinshine.github.io/2018/12/14/Swift/22.고차함수(2) - map, flatMap, compactMap/](https://jinshine.github.io/2018/12/14/Swift/22.%EA%B3%A0%EC%B0%A8%ED%95%A8%EC%88%98(2)%20-%20map,%20flatMap,%20compactMap/)

[https://ko.wikipedia.org/wiki/함수형_프로그래밍](https://ko.wikipedia.org/wiki/%ED%95%A8%EC%88%98%ED%98%95_%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D)

[https://mangkyu.tistory.com/111](https://mangkyu.tistory.com/111)

[https://hyunndyblog.tistory.com/163](https://hyunndyblog.tistory.com/163)
