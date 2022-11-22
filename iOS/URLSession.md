## 답변

> 네트워크 통신과 관련된 작업들의 그룹을 관리하는 객체입니다. 앱과 서버 간에 데이터를 주고 받기위해 HTTP를 사용합니다.
> 

## 내용

[공식문서](https://developer.apple.com/documentation/foundation/urlsession)를 바탕으로 공부하였습니다.

### URLSession이란?

URL로 표시된 endpoint 에서 데이터를 다운로드 하고 데이터를 업로드 하기위해 API를 제공한다.

- endpoint : 서버에서 자원(resource)에 접근할 수 있도록 하는 URL
    ![Untitled](https://user-images.githubusercontent.com/37897873/203249362-8ab69778-7009-4d7d-814c-10389356472f.png)

  

    
앱이 running 되고 있지 않거나, 앱이 suspended 상태여도(새로 안 사실..) 백그라운드에서 다운로드 할 수 있다.

관련 `URLSessionDelegate` 와 `URLSessionTaskDelegate` 를 사용하여 권한이나 redirection, task completion 작업 같은 이벤트를 받을 수 있다.

> Note: 공식문서에 URLSession은 다른 클래스들과 여러 복잡하게 함께 사용되어 해당 레퍼런스만 읽고 명확하게 이해하기 어려울 수 있다고 합니다. 그래서 URL Loading System 내용이 담긴 [이 문서](https://developer.apple.com/documentation/foundation/url_loading_system)를 먼저 읽으라고 합니다.
> 

### URL Loading System

URL Loading System은 https 같은 표준 프로토콜이나 커스텀 프로토콜을 사용할 때, URL로 식별되는 리소스에 대한 접근을 제공한다.

Loading은 비동기적으로 수행되어서 앱이 응답성을 유지하고(응답할 것을 기대하고라고 이해하면 되려나), 들어오는 데이터나 에러를 처리할 수 있다.

우리는 URLSession 인스턴스를 사용해서 하나 이상의 URLSessionTask 인스턴스를 생성하고, 이를 통해 데이터를 fetch 하고 return 하거나, 파일 다운로드, 데이터나 파일을 원격에 업로드 할 수 있다.

자세하게 설정하기 위해선 `URLSessionConfiguration` 객체를 통해 캐시, 쿠키, 등 다양한 정보를 설정할 수 있다.

*좀 더 부가적인 설명은 [공식문서](https://developer.apple.com/documentation/foundation/url_loading_system) 참고!

### ****Types of URL Sessions****

주어진 URL 세션 내에서의 task들은 세션에 한번에 최대 동시 연결 수, 셀룰러 네트워크 할 수 있는 지 등 모두 세션 설정에서 공유합니다.

URLSession은 `shared` 싱글톤 세션을 갖고 있는데, 설정 객체를 갖고있지 않고 그저 기본 요청에 대해서만 지원한다. 커스터마이징 할 순 없지만, 요구사항이 제한적인 (일반적인) 케이스라면 좋은 선택
(아하, 정말 기본적인 설정만 지원하고, 상세한건 위에서 처럼 URLSessionConfiguration을 구현해야 하네..)

세션 인스턴스 생성시의 3가지 설정으로 구현할 수 있다.

<img width="506" alt="%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-11-16_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_11 55 47" src="https://user-images.githubusercontent.com/37897873/203249357-b7bab3e7-b8dc-4a33-9532-1eaf277d435b.png">


1. Default : shared 처럼 아주 기본적인 기능들을 제공하지만, 직접 설정할 수 있다는 차이점이 있다. 많은 데이터를 얻기위해서 delegate를 통해 설정할 수 있다!
2. Ephemeral : shared 세션과 이것도 유사하지만, 캐시나 쿠키, 인증정보들이 디스크에 남기지 않는다.
3. Background : content를 업로드나 다운로드를 앱이 running 상태가 아니여도 백그라운드에서 동작되게 수행할 수 있다.

### ****Types of URL Session Tasks****

URLSession은 작업을 관리하는 객체이고, 데이터를 올리거나 받을 때는 task 작업 단위로 이뤄져 있어, 각 세션 내에하는 일에 따라 여러개의 task를 정의할 수 있다.

그 중 URLSession API는 네가지 유형의 task를 정의 할 수 있습니다.

1. DataTask : NSData 객체를 이용해 데이터를 보내고 받는다. 서버에 대한 짧은 interactive 요청을 위한 것
2. UploadTask : DataTask와 유사하지만, 데이터(주로 파일 형식)을 전송하고, 앱이 실행되지 않는 동안 백그라운드 업로드를 지원한다.
3. DownloadTask : 파일 형식으로 데이터를 검색하고, 앱이 실행되지 않는 동안 백그라운드 다운로드를 지원한다.
4. WebSocket : RFC 6455에 정의된 WebSocket 프로토콜을 사용하여 TCP 및 TLS를 통해 메시지를 교환합니다.

(데이터 받아올 때 뿐만 아니라 그냥 dataTask만 썼는데.. 정확히는 정의된 task를 사용하자!)

### ****Protocol Support****

URLSession은 data, file, ftp, http, https의 URL Scheme을 지원하고

HTTP는 1.1/2/3 프로토콜을 지원!

### ****Thread Safety****

URLSession API는 Thread-safe 하다.

어떤 쓰레드에서든지 세션이나 task를 자유롭게 생성할 수 있다.

completion handler가 제공된 delegate 메소드를 호출하면, 올바른 delegate queue에서 작업이 자동적으로 할당된다.

- delegate queue: The operation queue provieded when this object was created (이 객체 URLSession이 생성될 때 제공되는 작업queue 구나)

### CompletionHandler로 결과 받기

- 주로 URLSession shared 객체로 dataTask를 생성하고  completionHandler로 `data`, `response`, `error` 를 받는다.

![Untitled 1](https://user-images.githubusercontent.com/37897873/203249355-f66474b2-fab4-47c9-a895-2818f58a5bda.png)

```swift
let dataTask = URLSession.shared.dataTask(with: url) { data, response, error in
	//code
}
```

### Delegate를 활용해 세부 설정
![Untitled 2](https://user-images.githubusercontent.com/37897873/203249348-fe80542c-df9c-42c9-8b92-ba2b64013343.png)


1. delegate를 사용하면 기본 shared가 아닌 URLSession 인스턴스를 생성해야 한다.

```swift
private lazy var session: URLSession = {
    let configuration = URLSessionConfiguration.default
    configuration.waitsForConnectivity = true
    return URLSession(configuration: .default,
                      delegate: self, delegateQueue: nil)
}()
```

2. dataTask를 시작시키고, 처리하기 위해 delegate 콜백을 활용해야 한다.
    - `urlSession(_:dataTask:didReceive:completionHandler:)` : response가 성공적인 HTTP 상태 코드를 가지고 있는지 검증하고 MIME 타입이 text/html인지 text/plain인지 확인한다. 만약 두 경우 모두 아니라면 task는 취소되며, 정상인 경우는 진행된다.
    - `urlSession(_:dataTask:didReceive)` : task를 통해 받은 각 Data 인스턴스를 receivedData
    라고 불리는 버퍼에 저장한다.
    - `urlSession(_:task:didCompleteWithError:)` : 전송 오류가 발생했는지 확인 후 오류가 없다면 receivedData 버퍼를 원하는 모델 타입에 맞게 디코딩해준다

<details>
<summary>공식문서 Delegate 예시코드 </summary>
<div markdown="1"> 

<br>
    
```swift
var receivedData: Data?

func startLoad() {
    loadButton.isEnabled = false
    let url = URL(string: "https://www.example.com/")!
    receivedData = Data()
    let task = session.dataTask(with: url)
    task.resume()
}

// delegate methods

func urlSession(_ session: URLSession, dataTask: URLSessionDataTask, didReceive response: URLResponse,
                completionHandler: @escaping (URLSession.ResponseDisposition) -> Void) {
    guard let response = response as? HTTPURLResponse,
        (200...299).contains(response.statusCode),
        let mimeType = response.mimeType,
        mimeType == "text/html" else {
        completionHandler(.cancel)
        return
    }
    completionHandler(.allow)
}

func urlSession(_ session: URLSession, dataTask: URLSessionDataTask, didReceive data: Data) {
    self.receivedData?.append(data)
}

func urlSession(_ session: URLSession, task: URLSessionTask, didCompleteWithError error: Error?) {
    DispatchQueue.main.async {
        self.loadButton.isEnabled = true
        if let error = error {
            handleClientError(error)
        } else if let receivedData = self.receivedData,
            let string = String(data: receivedData, encoding: .utf8) {
            self.webView.loadHTMLString(string, baseURL: task.currentRequest?.url)
        }
    }
}
```
</details>
	
	
	
## 질문

### Alamofire나 Moya 같은 것과 어떤 연관이 있나요?

더욱 사용하기 편리하게 구현되어 있을 뿐 내부 로직은 URLSession에 기반되어 wrapping 되어 있기 때문에 URLSession을 먼저 명확히 사용하고 외부 라이브러리를 사용해야 합니다.

### 그래서 URLSession은 어떤 스레드에서 동작하나요?

기본적으로 백그라운드 스레드에서 작업이 할당된다. 그렇기 때문에, URLSession으로 작업 처리하고 그 내부에UI 관련된 코드 짜면 에러나므로 main 스레드로 옮겨야 한다.

## 참고 자료

### 공식문서

- [https://developer.apple.com/documentation/foundation/urlsession](https://developer.apple.com/documentation/foundation/urlsession)
- [https://developer.apple.com/documentation/foundation/url_loading_system](https://developer.apple.com/documentation/foundation/url_loading_system)
- [https://developer.apple.com/documentation/foundation/urlsessionconfiguration](https://developer.apple.com/documentation/foundation/urlsessionconfiguration%20)
- [https://developer.apple.com/documentation/foundation/url_loading_system/fetching_website_data_into_memory](https://developer.apple.com/documentation/foundation/url_loading_system/fetching_website_data_into_memory)

### 참고

- [https://ios-development.tistory.com/651](https://ios-development.tistory.com/651)
- [https://jeonyeohun.tistory.com/357](https://jeonyeohun.tistory.com/357)
- endpoint : [https://velog.io/@kho5420/Web-API-그리고-EndPoint](https://velog.io/@kho5420/Web-API-%EA%B7%B8%EB%A6%AC%EA%B3%A0-EndPoint)
