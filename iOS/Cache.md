# Cache

# 답변

> 자주 사용하거나 시간이 오래 걸리는 작업의 데이터를 저장하여 빠른 접근시간을 제공하는 메모리입니다.
> 

# 내용

## 캐싱?

> 캐시 영역에 데이터를 저장하는 것으로, 자주 사용되거나 시간이 오래 걸리는 작업의 결과를 저장하여 어플리케이션의 처리 속도를 높여준다.
> 
- 시간이 오래 걸리는 작업 ex) URL 주소로 이미지 가져오기

| Memory Cache | Disk Cache |
| --- | --- |
| 데이터를 RAM에 저장 | 데이터를 Disk에 저장 |
| 빠르지만 휘발성 | 느리지만 비휘발성 |

## NSCache (Foundation/Collections)

> 키-값 쌍을 임시로 저장하는데 사용하는 가변 컬렉션 (In-memory cache)
> 

```swift
class NSCache<KeyType, ObjectType> : NSObject where KeyType : AnyObject, ObjectType : AnyObject
```

- 다른 가변 컬렉션과 다른 점
    - **자동 삭제 정책**이 포함되어 있어 캐시가 시스템 메모리를 너무 많이 사용하지 않도록 한다.
    - 캐시를 직접 Lock하지 않고 다른 스레드에서 항목을 추가, 삭제 및 질의를 할 수 있다. → Thread safety
    - **[NSMutableDictionary](https://developer.apple.com/documentation/foundation/nsmutabledictionary)** 객체와는 달리 캐시는 캐시에 넣은 키 객체를 복사하지 않는다.
- 일반적으로 NSCache 객체를 사용하여 생성 비용이 많이 드는 일시적인 데이터가 포함된 객체를 임시로 저장한다. 이러한 객체를 재사용하면 그 값을 다시 계산할 필요가 없기 때문에 성능이 향상될 수 있다.
하지만 이런 객체는 어플리케이션에 중요한 것이 아니므로 메모리가 부족하면 폐기될 수 있다.
만약 폐기된 후 필요해지면 다시 계산해야한다.
- 폐기될 수 있는 구성요소를 가진 객체는 캐시 제거 동작을 개선하기 위해 **[NSDiscardableContent](https://developer.apple.com/documentation/foundation/nsdiscardablecontent)** 프로토콜을 채택할 수 있다. 자동 삭제 정책은 변경할 수 있지만, 기본적으로 캐시 내의 **NSDiscardableContent** 객체는 콘텐츠가 폐기되면 자동으로 삭제된다. **NSDiscardableContent** 객체가 캐시에 저장되면 캐시는 해당 객체를 삭제할 때 **[discardContentIfPossible()](https://developer.apple.com/documentation/foundation/nsdiscardablecontent/1408998-discardcontentifpossible)** 를 호출한다.

```swift
class ImageCache {
    static let imageCache = NSCache<NSString, UIImage>()
}

func imageDownload(_ url: URL) {
        // 캐시에 이미지 있을 때
        if let cachedImage = ImageCache.imageCache.object(forKey: url.absoluteString as NSString) {
            DispatchQueue.main.async { [weak self] in
                print("캐시된 데이터가 있습니다")
                self?.contentMode = .scaleAspectFit
                self?.image = cachedImage
            }
        } else {
            var request = URLRequest(url: url)
            request.httpMethod = "GET"

            URLSession.shared.dataTask(with: request) { data, response, error in
                guard let httpURLResponse = response as? HTTPURLResponse, httpURLResponse.statusCode == 200,
                      let data = data, error == nil,
                      let image = UIImage(data: data)
                else {
                    print("Download image fail : \(url)")
                    return
                }

                DispatchQueue.main.async() { [weak self] in
                    print("새로운 이미지를 받아왔습니다")

                    ImageCache.imageCache.setObject(image, forKey: url.absoluteString as NSString)

                    self?.contentMode = .scaleAspectFit
                    self?.image = image
                }
            }.resume()
        }
    }
```

## URLCache (Foundation/URL Loading System)

> URL 요청을 캐시된 응답 객체에 매핑하는 객체
> 

```swift
class URLCache : NSObject
```

- URL Loading System은 성능 향상과 네트워크 트래픽을 줄이기 위해 response를 메모리와 디스크에 캐싱하는데, URLCache 클래스는 네트워크 리소스의 응답을 캐싱하는데 사용된다.
- URLCache 클래스는 NSURLRequest 객체를 [CachedURLResponse](https://developer.apple.com/documentation/foundation/cachedurlresponse) 객체로 매핑하여 URL Load 요청에 대한 응답 캐싱을 구현한다.
- URLCache의 shared 프로퍼티를 사용하면 공유 캐시 인스턴스에 직접 접근할 수 있다.
- **in-memory** 캐시와 **on-disk** 캐시를 제공하며 크기를 조작할 수 있다.
    - on-disk 캐시는 시스템의 디스크 공간이 부족할 때 삭제될 수 있지만 앱이 실행되지 않을 때만 삭제된다.
- 캐시 데이터가 영구적으로 저장되는 경로도 제어할 수 있다.
- **Thread Safety**
    - iOS 8 이상 및 MacOS 10.10 이상에서는 스레드 세이프하다.
    - URLCache 인스턴스 메서드는 여러 실행 컨텍스트에서 동시에 안전하게 호출될 수 있지만,
    [cachedResponse(for:)](https://developer.apple.com/documentation/foundation/urlcache/1411817-cachedresponse) 메서드와 [storeCachedResponse(_:for:)](https://developer.apple.com/documentation/foundation/urlcache/1410340-storecachedresponse) 메서드처럼 같은 요청에 대한 응답 읽기 또는 쓰기를 시도할 때 피할 수 없는 **race condition**을 갖는다는 점에 유의해야 한다.
    - URLCache의 서브클래스는 오버라이드된 메서드를 스레드 세이프 방식으로 구현해야 한다.
- **서브클래싱**
    - URLCache는 있는 그대로 사용하는 것이 목적이지만 특정 요구가 있다면 서브클래스를 할 수 있다.
        - ex) 캐시되는 응답을 가리거나 보안 또는 다른 이유로 스토리지 메커니즘을 재구현
    - 오버라이딩할 때는 시스템이 task 파라미터를 가진 메서드를 그렇지 않은 메서드보다 선호한다는 점을 유의해야 한다.
    따라서 서브클래싱할 땐 task 기반 메서드를 오버라이드해야 한다.
        - 캐시에 응답 저장
        - 캐시에서 응답 가져오기
        - 캐시된 응답 삭제

```swift
func imageDownloadToUrlCache(_ url: URL) {
        let request = URLRequest(url: url)

        if URLCache.shared.cachedResponse(for: request) != nil {
            print("URLCache에 캐시된 데이터가 있습니다")
            DispatchQueue.global().async {
                if let data = URLCache.shared.cachedResponse(for: request)?.data, let image = UIImage(data: data) {
                    DispatchQueue.main.async {
                        self.image = image
                    }
                }
            }
        } else {
            print("URLCache에 데이터 다운")
            DispatchQueue.global().async {
                let dataTask = URLSession.shared.dataTask(with: url) { data, response, error in
                    if let data = data {
                        let cachedData = CachedURLResponse(response: response!, data: data)
                        URLCache.shared.storeCachedResponse(cachedData, for: request)
                        DispatchQueue.main.async {
                            self.image = UIImage(data: data)!
                        }
                    }
                }
                dataTask.resume()
            }
        }
    }
```

### URL 요청에 대한 캐시 정책 설정

- 각 **URLRequest 인스턴스**는 캐싱을 수행할지 여부와 방법을 나타내는 **CachePolicy 객체**를 포함하고 있다.
이 정책을 변경해서 **캐싱을 제어**할 수 있다.
- 편의를 위해 **URLSessionConfiguration**에는 **requestCachePolicy 속성**이 있다.
이 세션에서 생성된 모든 요청은 해당 캐시 정책을 상속한다.
- 현재 HTTP와 HTTPS 응답만 캐시되고, FTP와 파일 URL의 경우 원본 소스에 접근할 수 있는지 여부만 결정한다.

| Cache policy | Local cache | Originating source |
| --- | --- | --- |
| [NSURLRequest.CachePolicy.reloadIgnoringLocalCacheData](https://developer.apple.com/documentation/foundation/nsurlrequest/cachepolicy/reloadignoringlocalcachedata) | Ignored (무시) | Accessed exclusively (단독 접근) |
| [NSURLRequest.CachePolicy.returnCacheDataDontLoad](https://developer.apple.com/documentation/foundation/nsurlrequest/cachepolicy/returncachedatadontload) | Accessed exclusively (단독 접근) | Ignored (무시) |
| [NSURLRequest.CachePolicy.returnCacheDataElseLoad](https://developer.apple.com/documentation/foundation/nsurlrequest/cachepolicy/returncachedataelseload) | Tried first (먼저 시도) | Accessed only if needed (필요하다면 접근) |
| [NSURLRequest.CachePolicy.useProtocolCachePolicy](https://developer.apple.com/documentation/foundation/nsurlrequest/cachepolicy/useprotocolcachepolicy) | Depends on protocol | Depends on protocol  |
- HTTP와 HTTPS에 대해 `useProtocolCachePolicy`가 구현되는 방법에 대한 설명은 [NSURLRequest.CachePolicy](https://developer.apple.com/documentation/foundation/nsurlrequest/cachepolicy) 참조
- `useProtocolCachePolicy`는 **URLRequest** 객체의 기본값이다.
- `useProtocolCachePolicy`는 HTTPS 응답을 디스크에 캐시하므로 사용자 데이터를 보호하는데 바람직하지 않을 수 있다. 이를 프로그래밍 방식으로 캐싱 동작을 수동으로 변경할 수 있다.

### 캐시에 직접 접근

- 세션의 configuration 객체의 [urlCache](https://developer.apple.com/documentation/foundation/urlsessionconfiguration/1410148-urlcache) 속성을 사용하여 URLSession 객체에서 사용하는 캐시 객체를 가져오거나 설정할 수 있다.
- 어떤 요청에 대한 캐시된 응답을 찾으려면 캐시에서 [cachedResponse(for:)](https://developer.apple.com/documentation/foundation/urlcache/1411817-cachedresponse)를 호출한다.
캐시된 데이터가 있으면 [CachedURLResponse](https://developer.apple.com/documentation/foundation/cachedurlresponse) 객체를 반환하고 없으면 **nil**을 반환한다.
- 캐시에서 사용하는 리소스를 검사할 수 있다. [currentDiskUsage](https://developer.apple.com/documentation/foundation/urlcache/1407771-currentdiskusage)
 와 [diskCapacity](https://developer.apple.com/documentation/foundation/urlcache/1413505-diskcapacity) 속성은 캐시에서 사용되는 파일 시스템 리소스를 나타내며, [currentMemoryUsage](https://developer.apple.com/documentation/foundation/urlcache/1408199-currentmemoryusage)
 와 [memoryCapacity](https://developer.apple.com/documentation/foundation/urlcache/1409781-memorycapacity) 속성은 메모리 사용량을 나타낸다.
- [removeCachedResponse(for:)](https://developer.apple.com/documentation/foundation/urlcache/1415377-removecachedresponse)를 사용하여 개별 아이템의 캐시된 데이터를 제거할 수 있다. 또한 **지정된 날짜가 지난** 아이템을 지우는 [removeCachedResponses(since:)](https://developer.apple.com/documentation/foundation/urlcache/1415231-removecachedresponses) 또는 **전체 캐시를 삭제**하는 [removeAllCachedResponses()](https://developer.apple.com/documentation/foundation/urlcache/1417802-removeallcachedresponses)를 사용하여 많은 캐시된 아이템들을 동시에 지울 수 있다.

### 프로그래밍 방식으로 캐싱 관리

- [storeCachedResponse(_:for:)](https://developer.apple.com/documentation/foundation/urlcache/1410340-storecachedresponse) 메서드를 사용하여 새로운 CachedURLResponse 객체와 URLRequest 객체를 전달하여 프로그래밍 방식으로 캐시에 쓸 수 있다.
- 일반적으로 URLSessionTask 객체에서 처리하는 동안 응답의 캐싱을 관리한다. 응답별 캐싱을 관리하려면 [URLSessionDataDelegate](https://developer.apple.com/documentation/foundation/urlsessiondatadelegate) 프로토콜의 [urlSession(_:dataTask:willCacheResponse:completionHandler:)](https://developer.apple.com/documentation/foundation/urlsessiondatadelegate/1411612-urlsession) 메서드를 구현해야 한다.
이 메서드는 업로드 및 데이터 작업에만 호출되며, 백그라운드 또는 [ephemeral configuration](https://developer.apple.com/documentation/foundation/urlsessionconfiguration/1410529-ephemeral)의 세션에서는 호출되지 않는다.
- 이 delegate는 두 개의 파라미터(CachedURLResponse 객체와 completion handler)를 받는다.
다음 중 하나를 전달하여 completion handler를 직접 호출해야 한다.
    - 제공된 응답을 있는 그대로 캐시하기 위해 제공된 CachedURLResponse 객체
    - 캐싱을 방지하기 위한 nil
    - 새로 생성된 CachedURLResponse 객체로, 일반적으로 제공된 객체를 기반으로 하지만 수정된 [storagePolicy](https://developer.apple.com/documentation/foundation/cachedurlresponse/1412269-storagepolicy)와 [userInfo](https://developer.apple.com/documentation/foundation/cachedurlresponse/1411900-userinfo) 딕셔너리를 사용한다.

```swift
func urlSession(_ session: URLSession, dataTask: URLSessionDataTask,
                willCacheResponse proposedResponse: CachedURLResponse,
                completionHandler: @escaping (CachedURLResponse?) -> Void) {
    if proposedResponse.response.url?.scheme == "https" {
        let updatedResponse = CachedURLResponse(response: proposedResponse.response,
                                                data: proposedResponse.data,
                                                userInfo: proposedResponse.userInfo,
                                                storagePolicy: .allowedInMemoryOnly)
        completionHandler(updatedResponse)
    } else {
        completionHandler(proposedResponse)
    }
}
```

- HTTPS 요청에 대한 응답을 가로채고 in-memory 캐시에만 저장할 수 있도록 하는 urlSession 메서드의 구현

# 추가 질문

## Dictionary와 NSCache의 차이

**NSCache**에는 기기의 메모리가 부족할 경우 **자동으로 삭제**해주는 정책이 있기 때문에 **캐시로 사용**하기 적합하다. 

또한, **Thread safe**하기 때문에 여러 스레드에서 접근할 때 Lock을 걸어줄 필요가 없다.

그리고 **[NSMutableDictionary](https://developer.apple.com/documentation/foundation/nsmutabledictionary)** 객체와는 달리 캐시는 캐시에 넣은 key 객체를 복사하지 않는다..???

**NSCache**의 
[setObject(_:forKey)](https://developer.apple.com/documentation/foundation/nscache/1408223-setobject)

```swift
func setObject(
    _ obj: ObjectType,
    forKey key: KeyType
)
```

**NSMutableDictionary**의
[setObject(_:forKey)](https://developer.apple.com/documentation/foundation/nsmutabledictionary/1411616-setobject)

```swift
func setObject(
    _ anObject: Any,
    forKey aKey: NSCopying
)
```

- [NSCopying](https://developer.apple.com/documentation/foundation/nscopying) 프로토콜을 따르는 key는 [copy(with:)](https://developer.apple.com/documentation/foundation/nscopying/1410311-copy) 메서드를 사용하여 복사된다.
- NSCache에는 NSCacheKey라는 클래스를 통해 key 값에 대한 객체를 사용하고,
NS(Mutable)Dictionary는 저장하려는 key-value 엔트리를 _storage라는 내부 딕셔너리 자료구조에 저장을 하는데 이때 key를 복사해서 저장한다. 이 key는 NSCopying을 준수하는 NSObject이고 클래스이기 때문에 복사하는 것일까..? 그럼 왜 value는 따로 복사하지 않는지 의문인데… 아직 명확한 이유를 찾지 못 했다.

```swift
open func setObject(_ obj: ObjectType, forKey key: KeyType, cost g: Int) {
    let g = max(g, 0)
    let keyRef = NSCacheKey(key)
...
```

```swift
public required init(objects: UnsafePointer<AnyObject>!, forKeys keys: UnsafePointer<NSObject>!, count cnt: Int) {
    _storage = [NSObject : AnyObject](minimumCapacity: cnt)
    for idx in 0..<cnt {
        let key = keys[idx].copy()
        let value = objects[idx]
        _storage[key as! NSObject] = value
    }
}
...
```

- 아무튼 NSCache는 key에 대한 객체를 따로 생성하기 때문에 복사를 하지 않는다.

그럼 **Dictionary와 NSDictionary의 차이점**은..?

**Dictionary**는 **Swift의 구조체**이고 **NSDictionary**는 **Cocoa의 클래스**이다.

보다 자세한 차이점은 다음 시간에 알아보자..ㅎ

# 참고 자료

[https://developer.apple.com/documentation/foundation/nscache](https://developer.apple.com/documentation/foundation/nscache)

[https://developer.apple.com/documentation/foundation/urlcache](https://developer.apple.com/documentation/foundation/urlcache)

[https://levelup.gitconnected.com/image-caching-with-urlcache-4eca5afb543a](https://levelup.gitconnected.com/image-caching-with-urlcache-4eca5afb543a)

[https://developer.apple.com/documentation/foundation/url_loading_system/accessing_cached_data](https://developer.apple.com/documentation/foundation/url_loading_system/accessing_cached_data)

[https://developer.apple.com/documentation/foundation/nsurlrequest/cachepolicy](https://developer.apple.com/documentation/foundation/nsurlrequest/cachepolicy)

[https://developer.apple.com/documentation/foundation/urlrequest](https://developer.apple.com/documentation/foundation/urlrequest)

[https://developer.apple.com/documentation/foundation/nsmutabledictionary](https://developer.apple.com/documentation/foundation/nsmutabledictionary)

[https://developer.apple.com/documentation/foundation/nsdictionary](https://developer.apple.com/documentation/foundation/nsdictionary)

[https://developer.apple.com/documentation/foundation/nscopying](https://developer.apple.com/documentation/foundation/nscopying)

[https://developer.apple.com/documentation/objectivec/nsobject](https://developer.apple.com/documentation/objectivec/nsobject)

[https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/KeyValueCoding/BasicPrinciples.html#//apple_ref/doc/uid/20002170](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/KeyValueCoding/BasicPrinciples.html#//apple_ref/doc/uid/20002170)

[https://github.com/apple/swift-corelibs-foundation/blob/main/Sources/Foundation/NSCache.swift](https://github.com/apple/swift-corelibs-foundation/blob/main/Sources/Foundation/NSCache.swift)

[https://github.com/apple/swift-corelibs-foundation/blob/main/Sources/Foundation/NSDictionary.swift](https://github.com/apple/swift-corelibs-foundation/blob/main/Sources/Foundation/NSDictionary.swift)

[https://stackoverflow.com/questions/25554259/what-is-difference-between-nsdictionary-vs-dictionary-in-swift](https://stackoverflow.com/questions/25554259/what-is-difference-between-nsdictionary-vs-dictionary-in-swift)
