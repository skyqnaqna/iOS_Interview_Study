# Copy On Write가 무엇인가요?

# 답변

> Copy-On-Write는 복사 동작을 할 때, 실제 원본이나 복사본이 수정되기 전까지 복사 하지 않고 원본 리스소 공유하다가 수정이 일어날 때 복사하는 작업을 한다.
Swift에선 Collection Type(Array, Dictionary,Set)을 복사해서 사용할 때 일어난다.
> 

# 내용

배열 바로 복사하지 않고, 첫 수정이 일어날 때 복사가 일어나서 메모리에 할당됨

실제 수정이 이뤄질 때 복사를 하고, 그전엔 **참조를 통해 불필요한 복사**를 줄이자!

![Untitled](https://user-images.githubusercontent.com/37897873/214742487-0ea310fa-31b8-45e4-89c8-9e27e7688006.png)


1. 복사본에 값을 추가하면 COW 일어나서 주소값 변경됨
2. 테스트 해보니 2,4,6,8 이렇게 capacity가 증가된다. 7을 append하면 배열의 크기가 6으로 되는데, 현재 capacity = 6이라서 초과되지 않았으므로 동일한 주소사용함.
3. 3을 하나 더 추가하므로써 capacity를 초과했으므로 새로운 배열을 생성해서 값을 추가한다.
    1. 현재 capacity = 6
    2. 배열의 크기 = 7

![Untitled2](https://user-images.githubusercontent.com/37897873/214742483-9dcb33d6-9761-48b1-80ec-3c946111d105.png)

### 공식문서 속 COW

Value type은 copy on write 최적화를 사용한다.

> 표준 라이브러리의 모든 가변 크기 컬렉션과 마찬가지로 배열은 copy-on-write 최적화를 사용합니다. 어레이의 여러 복사본은 복사본 중 하나를 수정할 때까지 동일한 스토리지를 공유합니다. 이 경우 수정 중인 어레이는 스토리지를 고유하게 소유한 복제본으로 교체한 다음 제자리에서 수정합니다. 때때로 복사량을 줄일 수 있는 최적화가 적용됩니다.

</aside>

### array 동작방식

1. 초기에 적당한 크기의 공간을 할당해둔 다음
2. 해당 공간이 다 차면 원래 capacity 의 배수만큼 새로운 곳에 공간을 할당하고
3. 새로운 영역으로 값을 복사하고 있었던 거군

# 추가 질문

### array 복사하면 다른 값이 나오는데, string에서는 다른 주소값 나오는 이유?

<img width="522" alt="3" src="https://user-images.githubusercontent.com/37897873/214742478-ed2b7146-458f-4c52-a77e-d30c4aa91cdf.png">

- unsafe pointer를 인자로 받는 함수에 **`array`를 전달**하는 것은 **첫번째 어레이 인자의 주소를 전달**하는 것입니다. 따라서 array들이 같은 저장공간을 사용한다면 동일한 값이 나오게 됩니다.
- unsafe pointer를 인자로 받는 함수에 **`string`을 전달**하는 것은 **string을 나타내는 임시 UTF-8 의 pointer를 전달**하는 것입니다. 따라서 이 주소는 같은 string일지라도 매 호출마다 달라질 수 있습니다.

> string은 임시 UTF-8 포인터를 전달해서 매호출마다 달라진다(?)
> 

### capacity란?

![4](https://user-images.githubusercontent.com/37897873/214742473-ba82c23c-46af-4625-b13a-5ca1e481a84b.png)

- capacity : 새 저장소를 할당하지 않고 배열에 포함할 수 있는 총 요소수
- 배열에 요소 추가시 예약된 용량을 초과하면 더 큰 메모리 할당하고, 요소를 할당한 새 메모리에 할당
- 배열의 크기가 capacity 이하이면 현재 메모리를 그대로 사용한다.
- 새로운 저장소는 이전 저장소 크기의 2배
- `reserveCapacity(_:)` : 해당 메소드로 용량을 예약하고 재할당을 방지할 수 있다.
    - 초과할당을 제한된 숫자로 막을 수 있따.

![5](https://user-images.githubusercontent.com/37897873/214742469-2e0c500d-23d1-44ed-a887-e9921dde3070.png)

# 참고 자료

### cow

- [https://babbab2.tistory.com/18](https://babbab2.tistory.com/18)
- [https://sihyungyou.github.io/iOS-copy-on-write/](https://sihyungyou.github.io/iOS-copy-on-write/)
- [https://sujinnaljin.medium.com/swift-value-type의-메모리-주소-7d4c3de73b11](https://sujinnaljin.medium.com/swift-value-type%EC%9D%98-%EB%A9%94%EB%AA%A8%EB%A6%AC-%EC%A3%BC%EC%86%8C-7d4c3de73b11)
- [https://jeong9216.tistory.com/474](https://jeong9216.tistory.com/474)

### capacity

- [https://zeddios.tistory.com/117](https://zeddios.tistory.com/117)
- [https://sujinnaljin.medium.com/swift-array-의-capacity-9c3a99d2c31f](https://sujinnaljin.medium.com/swift-array-%EC%9D%98-capacity-9c3a99d2c31f)

### 공식문서 array

- [https://developer.apple.com/documentation/swift/array](https://developer.apple.com/documentation/swift/array)