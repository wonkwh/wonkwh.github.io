+++
title = "Swinject Tutorial for iOS"
date = 2024-02-22
categories = ["uikit"]

[taxonomies]
tags = ["swift","uikit", "dependancy_injection", "protocol", "design_pattern"]

+++


- 원문: https://www.kodeco.com/17-swinject-tutorial-for-ios-getting-started
- 이 튜토리얼에서는 Swift로 작성된 의존성 주입 프레임워크인 Swinject를 통해 의존성 주입(DI)의 개념에 대해 알아본다
<!-- more -->
- 비트코인의 현재 가격을 표시하는 [BitcoinAdventure](https://github.com/wonkwh/BlogShowcase/tree/develop/BitcoinAdventurer) 라는 작은 iOS 애플리케이션을 개선하는 방식으로 진행
- 튜토리얼을 진행하면서 앱을 리팩터링하고 그 과정에서 단위 테스트를 추가.

## Dependancy Injection

- 종속성 주입 (Dependancy Injection) 은 코드 자체 대신 다른 객체에서 종속성을 제공하도록 코드를 구성하는 접근 방식..
- 이렇게 코드를 정리하면 테스트 및 리팩토링할 수 있는 느슨하게 결합된(loosely-coupled) 구성 요소로 구성된 코드베이스가 생성.
- 타사 라이브러리 없이도 의존성 주입을 구현할 수 있지만, 널리 사용되는 패턴인 의존성 주입(DI) 컨테이너 (DI Container) 를 사용 하는 Swinject 를 사용
- 이러한 유형의 패턴은 코드 복잡성이 증가하더라도 종속성 해결을 단순하게 유지합니다.

## 종속성 주입이 필요한 이유

- 의존성 주입은 제어의 역전(**Inversion of Control**) 이라는 원칙에 의존.
- 주요 개념은 일부 종속성이 필요한 코드가 자체적으로 종속성을 생성하지 않고 이러한 종속성을 제공하는 제어를 더 높은 추상화로 위임
- 이러한 종속성은 일반적으로 객체의 이니셜라이저로 전달
- 이는 일반적인 계단식 객체 생성의 반대되는 접근 방식, 즉 반전된 접근 방식
- 객체 A가 객체 B를 생성하고, 객체 C를 생성하는 식
- 실용적인 관점에서 볼 때, 제어 반전의 주요 이점은 코드 변경이 격리된 상태로 유지
- 의존성 주입 컨테이너는 객체에 대한 종속성을 제공하는 방법을 알고 있는 객체를 제공함으로써 제어의 역전 원칙을 지원
- 컨테이너에 필요한 객체를 요청하기만 하면 됨

## 시작하기

- 앱이 실행되면 화면에 비트코인의 현재 가격이 표시
- 새로 고침을 탭하면 최신 데이터를 검색하기 위해 HTTP 요청
- 코인베이스 API는 약 30초마다 새로운 비트코인 가격을 제공.
- 현재 오류 가 발생
  - _Swinject: Resolution failed. Expected registration:_
- 모든 네트워킹 및 구문 분석 로직은 `BitcoinViewController.swift`
- 현재 코드 상태로는 뷰 레이어가 기본 로직 및 종속성과 매우 밀접하게 연결
- UIViewController 수명 주기와 독립적으로 로직을 테스트하기 어렵다

## DI와 커플링 

- `BitcoinViewController.swift` 의 코드는 크게 세 가지를 담당
  - Networking, Parsing, Formatting

### Networking and Parsing

- 대부분 network는 다음메소드에서 발생

```swift
  private func requestPrice()  {
    let bitcoin = Coinbase.bitcoin.path
    
    // 1. Make URL request
    guard let url = URL(string: bitcoin) else { return }
    var request = URLRequest(url: url)
    request.cachePolicy = .reloadIgnoringCacheData
    
    // 2. Make networking request
    let task = URLSession.shared.dataTask(with: request) { data, _, error in
      
      // 3. Check for errors
      if let error = error {
        print("Error received requesting Bitcoin price: \(error.localizedDescription)")
        return
      }
      
      // 4. Parse the returned information
      let decoder = JSONDecoder()

      guard let data = data,
            let response = try? decoder.decode(PriceResponse.self,
                                               from: data) else { return }
      
      print("Price returned: \(response.data.amount)")
      
      // 5. Update the UI with the parsed PriceResponse
      DispatchQueue.main.async { [weak self] in
        self?.updateLabel(price: response.data)
      }
    }
```

- json model 은 다음과 같다.

```json
{
  "data": {
    "base": "BTC",
    "currency": "USD",
    "amount": "15840.01"
  }
}
```

### Formatting

```swift
private func updateLabel(price: Price) {
  guard let dollars = price.components().dollars,
        let cents = price.components().cents,
        let dollarAmount = standardFormatter.number(from: dollars) else { return }
  
  primary.text = dollarsDisplayFormatter.string(from: dollarAmount)
  partial.text = ".\(cents)"
}
```

## Extracting Dependencies

### Extracting Networking Logic

- `Dependencies` folder 생성
- `HttpNetworking.swift`
  - 원문 article의 소스를 swift5, result type 사용하여 수정하였다.
  - Result type 사용법은 [The Result Type | Swift by Sundell](https://www.swiftbysundell.com/basics/result/) 요기를 참조하어 수정함.

```swift
import Foundation

enum CoinBaseError: Swift.Error {
  case networkFailure(Swift.Error)
  case invalidUrl
  case invalidData
}

public extension URLSession {
  func dataTask(
    with url: URLRequest,
    handler: @escaping (Result<Data, Swift.Error>) -> Void
  ) -> URLSessionDataTask {
    dataTask(with: url) { data, _, error in
      if let error = error {
        handler(.failure(error))
      } else {
        handler(.success(data ?? Data()))
      }
    }
  }
}

protocol Networking {
  typealias CompletionHandler = (Result<Data, CoinBaseError>) -> Void
  func request(from: Endpoint, completion: @escaping CompletionHandler)
}

struct HTTPNetworking: Networking {
  func request(from: Endpoint, completion: @escaping CompletionHandler) {
    guard let url = URL(string: from.path) else { return }
    let request = createRequest(from: url)
    let task = createDataTask(from: request, completion: completion)
    task.resume()
  }

  private func createRequest(from url: URL) -> URLRequest {
    var request = URLRequest(url: url)
    request.cachePolicy = .reloadIgnoringCacheData
    return request
  }

  private func createDataTask(
    from request: URLRequest,
    completion: @escaping CompletionHandler
  ) -> URLSessionDataTask {
    return URLSession.shared.dataTask(with: request) { result in
      switch result {
      case .success(let data):
        completion(.success(data))
      case .failure(let error):
        completion(.failure(.networkFailure(error)))
      }
    }
  }
}

```

- `BitcoinViewController.swift` 의 requestPrice() 을 다음과 같이 수정

```swift

  let networking = HTTPNetworking()
  ...
  
  private func requestPrice()  {
    networking.request(from: Coinbase.bitcoin) { result in
      switch result {
      case .success(let data):
        // 4. Parse the returned information
        let decoder = JSONDecoder()
        guard let response = try? decoder.decode(PriceResponse.self, from: data) else { return }
        print("Price returned: \(response.data.amount)")

        // 5. Update the UI with the parsed PriceResponse
        DispatchQueue.main.async { [weak self] in
          self?.updateLabel(price: response.data)
        }
      case .failure(let error):
        debugPrint(error.localizedDescription)
      }
    }
  }
```

### Extracting Parsing Logic

- `BitcoinPriceFetcher.swift` 파일을 생성

```swift

import Foundation

protocol PriceFetcher {
  func fetch(response: @escaping (PriceResponse?) -> Void)
}

struct BitcoinPriceFetcher: PriceFetcher {
  let networking: Networking

  // 1. Initialize the fetcher with a networking object
  init(networking: Networking) {
    self.networking = networking
  }

  // 2. Fetch data, returning a PriceResponse object if successful
  func fetch(response: @escaping (PriceResponse?) -> Void) {
    networking.request(from: Coinbase.bitcoin) { result in
      switch result {
      case .success(let data):
        // Parse data into a model object.
        let decoded = self.decodeJSON(type: PriceResponse.self, from: data)
        if let decoded = decoded {
          print("Price returned: \(decoded.data.amount)")
        }
        response(decoded)

      case .failure(let error):
        debugPrint(error.localizedDescription)
        response(nil)
      }
    }
  }

  // 3. Decode JSON into an object of type 'T'
  private func decodeJSON<T: Decodable>(type: T.Type, from: Data?) -> T? {
    let decoder = JSONDecoder()
    guard let data = from,
          let response = try? decoder.decode(type.self, from: data) else { return nil }

    return response
  }
}

```

- `BitcoinViewController.swift` 를 다음과 같이 수정

```swift
  let fetcher = BitcoinPriceFetcher(networking: HTTPNetworking())
  ...
   private func requestPrice() {
    fetcher.fetch { response in
      guard let response = response else { return }

      DispatchQueue.main.async { [weak self] in
        self?.updateLabel(price: response.data)
      }
    }
  }
```

- `PriveFetcher` protocol은  HTTP 요청이 아니더라도 모든 데이터소스에서 발생가능
- 이는 UnitTest의 중요한 특징이 될 것

> part 1 끝, part 2로... 
----
- tag: #swift, #uikit, #dependancy_injection, #protocol, #design_pattern
