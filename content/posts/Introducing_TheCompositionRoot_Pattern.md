+++
title = "Introducing the Composition Root Pattern in a Swift Codebase"
date = 2024-11-15

[taxonomies]
tags = ["swift","uikit", "composition_root", "dependancy_injection", "protocol", "design_pattern"]
+++

`CompositionRoot` 를 SwiftCodeBase 에 적용하기
<!-- more -->
- 원글: https://simonbs.dev/posts/introducing-the-composition-root-pattern-in-a-swift-codebase/

## Introduction to Dependency Injection

- 2022년 초, 저는 Swift에서 의존성 주입에 대한 다양한 접근 방식을 탐색하다가 [Mark Seemann](https://blog.ploeh.dk/)이 [.NET의 의존성 주입](https://www.amazon.com/gp/product/1935182501)에서 소개한 Composition Root 패턴을 접하게 되었습니다. 그 이후로 여러 Swift 코드베이스에 Composition Root 패턴을 도입했고 매우 유용하다는 것을 알게 되었습니다.
- 이 글에서는 Swift 코드베이스에 컴포지션 루트를 도입하고 iOS 프로젝트에서 사용하는 방법을 살펴보겠습니다. 먼저 의존성 주입에 대한 간략한 개요를 살펴보겠습니다.


- 의존성 주입은 객체가 직접 종속성을 생성하지 않고 외부 소스에서 종속 객체를 객체에 전달하는 디자인 패턴입니다.
- Consider the following `MovieService` type that loads movies and stores them in a `MovieRepository`.

	```swift
	  struct MovieRepository {
	    func add(_ movie: Movie) {
	    ...
	    }
	  }
	  
	  struct MovieService {
	    let repository = MovieRepository()
	  
	    func loadMovies() {
	    ...
	    }
	  }
	  ```

- `MovieService` 는 초기화 시 생성되는 `MovieRepository`에 대한 종속성을 가집니다. 이 두 유형 간의 긴밀한 결합은 코드를 단위 테스트할 때 문제를 일으킬 수 있습니다. 예를 들어, `MovieRepository`는 영화를 디스크에 저장할 수 있는데, 이는 단위 테스트에서 바람직하지 않을 수 있습니다.
- 다음과 같이 코드를 수정하여 컴포넌트를 분리할 수 있습니다:
	
- 코드베이스를 세 가지 변경했습니다:
	- 1. `영화 저장소` 프로토콜을 추가했습니다.
	- 2. 이전 `MovieRepository` 구현의 이름을 `DiskMovieRepository`로 변경하여 이제 `MovieRepository` 프로토콜을 준수합니다.
	- 3. 이제 `MovieService`는 구체적인 구현이 아닌 `MovieRepository` 프로토콜에 의존합니다.
- 이러한 변경 사항을 적용하면 아래와 같이 `MovieService`를 사용할 수 있습니다:
- 
 ```swift
  let repository = DiskMovieRepository()
  let service = MovieService(repository: repository)
  ```
- 이제 리포지토리는 생성자를 통해 서비스에 주입되고 있습니다. 이렇게 하면 `MovieService`에 어떤 종속성이 있는지 명확히 알 수 있고 다양한 요구 사항을 수용하기 위해 리포지토리 구현을 쉽게 바꿀 수 있습니다. 예를 들어 단위 테스트를 실행할 때 동영상을 메모리에 저장하는 저장소를 사용하고 싶을 수 있습니다.
- 이 글에서는 이니셜라이저를 통해 종속성을 주입하는 이니셜라이저 주입에 초점을 맞추고 있습니다. 메서드 주입과 프로퍼티 주입이라는 다른 유형의 의존성 주입도 있다는 점에 주목할 필요가 있습니다.

## Building the Dependency Graph

- 이니셜라이저를 통해 종속성을 주입할 때는 기본적으로 각 노드가 종속성을 나타내는 객체 그래프를 만듭니다. 이전 예시를 확장하여 이를 설명해 보겠습니다.
  
  ```swift
  class WatchlistViewController: UIViewController {
	init(userService: UserService, movieService: MovieService) {
	...
	}
  }
  
  struct UserService {
	let networkClient: NetworkClient
  }
  
  struct MovieService {
	let repository: MovieRepository
	let networkClient: NetworkClient
  }
  ```
	  
	
  - `WatchlistViewController` 는 두 개의 종속성을 가지고 있고, 그 종속성에는 다시 자체 종속성이 있기 때문에 여기서는 객체 그래프를 다루고 있습니다. 아래 그림과 같이 그래프를 그릴 수 있습니다.
  ![](https://simonbs.dev/images/posts/introducing-the-composition-root-pattern-in-a-swift-codebase/dependency-graph.png)
  
  - 이니셜라이저를 통해 종속성을 주입해야 하므로 이러한 종속성을 생성하기 위해 외부 소스에 의존하지만 외부 소스에도 종속성이 있을 수 있으므로 종속성 그래프가 커지고 질문이 생깁니다:
  
> How do we construct this graph and where in our codebase do we do it?
> 이 그래프를 어떻게 구성하고 코드베이스의 어디에서 구성해야 할까요?
> 
  
  - [많은](https://github.com/Swinject/Swinject) [오픈 소스](https://github.com/square/Cleanse) [Swift](https://github.com/uber/needle) [프레임워크](https://github.com/hmlongco/Factory) [그 주소](https://github.com/pointfreeco/swift-dependencies) [이러한 질문](https://github.com/scribd/Weaver) 등이 있습니다. 
  - 이러한 프레임워크 중 일부는 컴파일 타임에 의존성을 제공하는 것을 잊어버리면 런타임 오류가 발생할 수 있는 변형된 의존성 주입을 사용합니다. 
  - 실제로 SwiftUI의 `@EnvironmentObject`도 동일한 동작을 보일 수 있습니다.
	  
  다음 SwiftUI view 를 보면:
	  
  ```swift
  struct ContentView: View {
  @EnvironmentObject var movieService: MovieService
  
  var body: some View {
	Text("\(movieService.count) movies")
  }
  }
  ```
	  
- `ContentView`는 뷰의 조상 중 하나에  `environmentObject()` 뷰 수정자를 사용하여 삽입해야 하는 `MovieService`에 대한 종속성을 가지고 있습니다. 그러나 이를 잊어버리면 런타임에 다음과 같은 오류가 발생합니다:
  
> Fatal error: No ObservableObject of type MovieService found. A View.environmentObject(_:) for MovieService may be missing as an ancestor of this view

-   이러한 런타임 오류와 그에 따른 앱 충돌은 잘못 설계된 의존성 주입 전략의 결과이므로 피해야 합니다. 
-   대신 런타임 오류보다 컴파일 타임 오류를 우선시하는 방식으로 코드를 설계하는 것을 목표로 해야 합니다.
-   또한 종속성 주입 전략을 선택하는 것은 아키텍처적 결정이며, 코드베이스에서 외부 프레임워크에 대한 강력한 종속성을 생성할 수 있으므로 이를 위해 타사 프레임워크에 의존하지 않는 것을 선호합니다.

## The Composition Root Pattern Enters the Room
  
 -  [마크 시만](https://blog.ploeh.dk/)은 자신의 저서 [.NET의 의존성 주입](https://www.amazon.com/gp/product/1935182501)에서 객체 그래프를 어디에 구성할지에 대한 질문에 답하기 위해 컴포지션 루트를 사용할 것을 제안합니다. 책 제목에 '.NET'이 들어간다고 해서 겁먹지 마세요. 이 패턴을 Swift 😊에 적용하기 위한 것이니까요.
 - 컴포지션 루트는 컴포넌트에 주입되는 모든 종속성을 생성하는 역할을 합니다. 이를 위해 'CompositionRoot'라는 열거형을 만들고 각 종속성을 정적 프로퍼티로 나열하면 됩니다. 객체에 종속성이 있을 때마다 해당 프로퍼티를 참조할 수 있습니다.
  
  ```swift
  enum CompositionRoot {
  static var watchlistViewController: WatchlistViewController {
    WatchlistViewController(
      userService: userService,
      movieService: movieService
    )
  }
  
  private static var userService: UserService {
    UserService(networkClient: networkClient)
  }
  
  private static var movieService: MovieService {
    MovieService(
      repository: movieRepository, 
      networkClient: networkClient
    )
  }
  
  private static var movieRepository: MovieRepository {
    DiskMovieRepository()
  }
  
  private static var networkClient: NetworkClient {
    NetworkClient()
  }
  }
  ```
  
  - ``CompositionRoot`` 열거형은 엔트리 포인트가 열거형 자체인 객체 그래프를 표현합니다.
  -   이 컴포지션 루트는 매우 작기 때문에 프로덕션에서 컴포지션 루트를 사용하는 예시를 보고 싶다면 [Shape](https://shape.dk/)에서 개발한 [앱에서 이 컴포지션 루트 보기](https://github.com/shapehq/tartelet/blob/main/Tartelet/Sources/CompositionRoot.swift#L29)를 참고하세요.
  -   이제 `CompositionRoot` 를 배치하고 객체 그래프가 어떻게 구성되는지 지정했으므로 이제 '컴포지션 루트' 유형을 사용하여 객체의 인스턴스를 생성하고 종속성을 연결할 수 있습니다.
### Using the Composition Root in an iOS App
  
  - [Seemann](https://blog.ploeh.dk/2011/07/28/CompositionRoot/)에 따르면 객체 그래프는 애플리케이션의 진입점에 최대한 가깝게 구성해야 합니다. 이는 이니셜라이저를 통해 종속성을 제공하기 위해 외부 소스에 의존하는 객체의 당연한 결과입니다.
  -  iOS 앱에서는 `AppDelegate`와 씬 델리게이트를 애플리케이션의 엔트리 포인트로 간주할 수 있습니다. 즉, 각 장면 델리게이트가 그 자체로 엔트리 포인트이기 때문에 하나의 엔트리 포인트가 없을 수도 있습니다.
  -  `WatchlistViewController`의 인스턴스를 표시하는 장면이 필요하다고 가정해 봅시다. 대부분의 경우 씬의 루트 뷰 컨트롤러는 UINavigationController의 인스턴스이므로 `CompositionRoot`를 수정하여 `UINavigationController`의 인스턴스를 노출합니다.
  
  ```swift
  enum CompositionRoot {
  static var rootViewController: UIViewController {
    UINavigationController(
      rootViewController: watchlistViewController
    )
  }
  
  ...
  }
  ```
  

  - `컴포지션 루트`가 루트 뷰 컨트롤러를 노출하면 이제 남은 작업은 씬 델리게이트에서 `rootViewController` 프로퍼티를 참조하는 것뿐입니다.
  
  ```swift
  class SceneDelegate: UIResponder, UIWindowSceneDelegate {
  func scene(
    _ scene: UIScene, 
    willConnectTo session: UISceneSession, 
    options connectionOptions: UIScene.ConnectionOptions
  ) {
    let windowScene = scene as! UIWindowScene
    window = UIWindow(windowScene: windowScene)
    window?.rootViewController = CompositionRoot.rootViewController
    window?.makeKeyAndVisible()
  }
  }
  ```

## Sharing State Between Objects
  
  - `컴포지션 루트`는 종속성을 참조할 때마다 새로운 인스턴스를 생성합니다. 
  -   아래 코드를 살펴보면 `userService` 및 `movieService` 프로퍼티가 반환하는 서비스에 각각 고유한 `NetworkClient` 인스턴스가 있음을 알 수 있습니다. 
  -  이는 `네트워크 클라이언트`가 접근 시 새 인스턴스를 생성하는 게터로만 구현되어 있기 때문입니다.
  
  ```swift
  enum CompositionRoot {
  ...
  
  private static var userService: UserService {
    UserService(networkClient: networkClient)
  }
  
  private static var movieService: MovieService {
    MovieService(
      repository: movieRepository, 
      networkClient: networkClient
    )
  }
  
  private static var networkClient: NetworkClient {
    NetworkClient()
  }
  
  ...
  }
  ```
  
  - 이 동작은 대부분의 경우 객체를 일시적으로 사용할 수 있으므로 바람직합니다. 필요할 때 생성하고 가능한 한 빨리 폐기하면 됩니다. 
  - 그러나 두 종속성 간에 상태를 공유해야 할 때는 이 접근 방식이 작동하지 않습니다.
  - 네트워크 요청을 큐에 대기시키는 사용자 정의 로직이 있고 객체가 `NetworkClient`의 인스턴스를 공유해야 하는 시나리오를 상상해 봅시다. 
  - 이 경우 `CompositionRoot`의 `networkClient` 속성을 변경하여 인스턴스를 생성하고 저장하여 속성을 참조할 때마다 동일한 인스턴스를 반환하도록 할 것입니다. 다행히도 이 작업은 `networkClient`를 상수로 만드는 것만큼이나 간단합니다.
  
  ```swift
  enum CompositionRoot {
  ...
  
  private static let networkClient = NetworkClient()
  
  ...
  }
  ```
 
## 커지는 컴포지션 루트 처리하기
  
- 컴포지션 루트 패턴의 적응에 대해 논의할 때 사람들이 주로 우려하는 것 중 하나는 시간이 지남에 따라 `CompositionRoot` 열거형이 너무 커질 수 있다는 것입니다. 
- 이는 타당한 우려이며, 이를 해결하기 위해 이를 별도의 파일로 분할하여 각 파일마다 `CompositionRoot`에 extension를 추가할 수 있습니다.
- 일반적으로 이러한 확장은 객체 그래프에서 앱의 특정 도메인이나 기능과 관련된 부분을 다룹니다.
- 예를 들어, 앱의 영화 도메인과 관련된 모든 객체를 포함하는 `Movies` 열거형을 추가하는 extension를 `CompositionRoot`에 생성할 수 있습니다.
  
  ```swift
  /// 📄 CompositionRoot+Movies.swift
  extension CompositionRoot {
  enum Movies {
    func moviesService(networkClient: NetworkClient) {
      MovieService(
        repository: movieRepository, 
        networkClient: networkClient
      )
    }
    
    private static var movieRepository: MovieRepository {
      DiskMovieRepository()
    }
  }
  }
  ```
  
  - 오브젝트는 여전히 `CompositionRoot`를 직접 사용해야 하므로 `MovieService`를 노출하고 `NetworkClient`의 공유 인스턴스를 사용하도록 합니다.
  ```swift
  /// 📄 CompositionRoot.swift
  enum CompositionRoot {
  ...
  
  var movieService: MovieService {
    Movies.movieService(networkClient: networkClient)
  }
  
  private static let networkClient = NetworkClient()
  }
  ```

## 마무리 생각
  
  - 2022년 초부터 저는 대부분의 부업 프로젝트에 컴포지션 루트 패턴을 도입했고, [Shape](https://shape.dk/)에서 일하고 있는 몇몇 프로젝트에도 도입했습니다.
  - 의존성 주입과 컴포지션 루트 패턴을 도입하면 컴포넌트가 덜 긴밀하게 결합되어 단위 테스트가 더 쉬워집니다. 예를 들어, `MovieService`를 단위 테스트하려면 `MovieRepository`의 모의 구현을 생성하고 `MovieService`를 초기화할 때 이를 전달하기만 하면 됩니다.
  - 또한, [Medium]에서 설명한 전략(https://medium.engineering/evolution-of-the-medium-ios-app-architecture-8b6090f4508e)을 사용하여 컴포넌트를 자체 Swift 패키지로 옮기면 호스트 앱을 매우 얇게 만들 수 있습니다. 
  - 제 프로젝트에 컴포지션 루트 패턴을 도입한 후, 메인 타깃에는 보통 앱 델리게이트, 씬 델리게이트, 컴포지션 루트만 거의 포함되지 않는다는 것을 알게 되었습니다.
  - 메인 타깃에 코드가 거의 포함되어 있지 않다는 것은 프로젝트에 _Dev Apps_ 를 도입하기 위한 훌륭한 설정이 있다는 것을 의미합니다. 개발자 앱(https://medium.com/airbnb-engineering/designing-for-productivity-in-a-large-scale-ios-application-9376a430a0bf)은 개발자가 개발 중에 특정 기능을 반복할 때 사용하는 앱입니다. 
  - 이러한 앱은 앱의 특정 부분으로 실행되고 메인 타겟보다 적은 코드를 포함하므로 컴파일 시간이 단축되므로 개발 중 반복 작업을 더 빠르게 수행할 수 있습니다. 
  - 개발 앱 빌드는 [XcodeGen](https://github.com/yonaskolb/XcodeGen)을 사용하여 Xcode 프로젝트에 새 대상을 추가하고 앱을 구성하는 컴포지션 루트를 추가하는 것만으로 간단해집니다.
  - 이 포스팅을 읽고 Swift 코드베이스에 컴포지션 루트를 도입하는 것이 궁금하다면, 제가 일하면서 작업하고 최근에 오픈소스화한 앱인 [Tartelet 앱](https://github.com/shapehq/tartelet)의 [CompositionRoot.swift](https://github.com/shapehq/tartelet/blob/main/Tartelet/Sources/CompositionRoot.swift) 파일을 살펴볼 수 있습니다.

## 추가 레퍼런스 
- [Swift Connection 2023 - Simon B. Støvring. - Achieving Loose Coupling with Pure Dependency Injection - YouTube](https://www.youtube.com/watch?v=bmIW1skJQFo)
	- [GitHub - shapehq/CoffeeShopsExample: Example project showing how Dependency Injection along with the Composition Root pattern can be introduced in a SwiftUI codebase.](https://github.com/shapehq/CoffeeShopsExample)
	- 위 글은 쓴 저자의 #dependancyInjection , #compositionRoot 에 대한 컨퍼런스 와 해당 자료...
- https://www.youtube.com/watch?v=lx3-KDRORyo&list=PLQ_uTU3EyV1TsXYHGaB11Y4-owo5sHQwY
	- #dependancyInjection  에 대한 youtube playlist
