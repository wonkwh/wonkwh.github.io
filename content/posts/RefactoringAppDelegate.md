+++
title = "Refactoring Massive App Delegate"
date = 2021-02-01

[taxonomies]
tags = ["swift", "design_pattern", "refactoring"]
+++

## Refactoring Massive App Delegate

실제 production app 을 개발하다 보면 `AppDelegate` 클래스가 쉽게 비대해지고 파일수가 어마어마하게 늘어나는 경우가 비일비재하다. 이를 refactoring 하는 방법
<!-- more -->

### Mediator Design Pattern

[refactoring-massive-app-delegate](https://www.vadimbulavin.com/refactoring-massive-app-delegate/) 에 보면 다양한 디자인 패턴으로 appDelegate를 refactoring 하는 법이 나와있다 그중 mediator design pattern 으로 실제 활용한 예

### AppLifecycle Mediator

- 먼저 AppLifeCycle mediator 를 정의한다.
- 아래 소스는 `appDelegate.m` 이 objective-c 로 구현되어 있기에 `NSObject`를 상속받고 `objc` propertyObserver를 붙였다.

```swift
import Foundation

// MARK: - AppLifecycleListener
protocol AppLifecycleListener {
  func onAppWillEnterForeground()
  func onAppDidEnterBackground()
  func onAppDidFinishLaunching()
}

extension AppLifecycleListener {
  func onAppWillEnterForeground() {}
  func onAppDidEnterBackground() {}
}

// MARK: - Mediator
@objc
class AppLifecycleMediator: NSObject {
  private let listeners: [AppLifecycleListener]
  
  init(listeners: [AppLifecycleListener]) {
    self.listeners = listeners
    super.init()
    subscribe()
  }
  
  deinit {
    NotificationCenter.default.removeObserver(self)
  }
  
  private func subscribe() {
    NotificationCenter.default.addObserver(self, selector: #selector(onAppWillEnterForeground), name: UIApplication.willEnterForegroundNotification, object: nil)
    NotificationCenter.default.addObserver(self, selector: #selector(onAppDidEnterBackground), name: UIApplication.didEnterBackgroundNotification, object: nil)
    NotificationCenter.default.addObserver(self, selector: #selector(onAppDidFinishLaunching), name: UIApplication.didFinishLaunchingNotification, object: nil)
  }
  
  @objc private func onAppWillEnterForeground() {
    listeners.forEach { $0.onAppWillEnterForeground() }
  }
  
  @objc private func onAppDidEnterBackground() {
    listeners.forEach { $0.onAppDidEnterBackground() }
  }
  
  @objc private func onAppDidFinishLaunching() {
    listeners.forEach { $0.onAppDidFinishLaunching() }
  }
}
```

### AppListener class 구현

- 이제 `AppLifecycleListener` 를 상속하여 클래스를 구현한다.
- appDelegate 에서 구현된 각각의 서비스 구현들을 여기에 Extract 하여 구현
- 예를 들면 기존에 AppDelegate에서 구현되어있던 location permission 관련 feature 들은 아래와 같이 `AppLocationListener` 구현한다.  

 ```swift
import Foundation
import CoreLocation
import LoggingKit

class AppLocationListener: NSObject, AppLifecycleListener {
  let locationManager = CLLocationManager()
  
  fileprivate func permissionState(_ status: CLAuthorizationStatus) {
    LogService.shared.debug("LocationPermission: \(status.rawValue)", logCategory: \.debug)
    switch status {
    case .authorizedAlways, .authorizedWhenInUse:
      locationManager.startUpdatingLocation()
    case .restricted, .notDetermined:
      locationManager.requestWhenInUseAuthorization()
    case .denied:
      stopLocation()
    @unknown default:
      print("GPS: Default")
    }
  }
  
  func stopLocation() {
    locationManager.stopUpdatingHeading()
    locationManager.stopUpdatingLocation()
    locationManager.delegate = nil
  }
  
  func onAppDidFinishLaunching() {
    locationManager.delegate = self
    locationManager.desiredAccuracy = kCLLocationAccuracyBest
    let status: CLAuthorizationStatus
    if #available(iOS 14.0, *) {
      status = locationManager.authorizationStatus
    } else {
      status = CLLocationManager.authorizationStatus()
    }
    permissionState(status)
  }
}

extension AppLocationListener: CLLocationManagerDelegate {
  func locationManager(_ manager: CLLocationManager, didChangeAuthorization status: CLAuthorizationStatus) {
    self.permissionState(status)
  }
  
  func locationManager(_ manager: CLLocationManager, didFailWithError error: Error) {
    locationManager.stopUpdatingLocation()
  }
  
  func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
    let newLocation = locations[0]
    let geocoder = CLGeocoder()
    geocoder.reverseGeocodeLocation(newLocation) { (playmarks, error) in
      if let _ = error {
        self.stopLocation()
        return
      }
      
      if let city = playmarks?.first?.locality {
        cityDefault.saveLocationLng("\(newLocation.coordinate.longitude)")
        cityDefault.saveLocationLat("\(newLocation.coordinate.latitude)")
        cityDefault.saveLocationCity(city)
      }
    }
    stopLocation()
  }
}

```

- 그리고 `AppLifecycleMediator` 에 listener  에 추가

```swift
extension AppLifecycleMediator {
  @objc
  static func makeDefaultMediator() -> AppLifecycleMediator {
    let locationListener = AppLocationListener()
    let debugListener = AppDebugListener()

    return AppLifecycleMediator(listeners: [debugListener, locationListener])
  }
}

```

- 3rdParty service 들, debug feature 등 listener 로 추가하여 refactoring 해주면 된다. 

### add mediator to AppDelegate

- 이제 mediator class 를 AppDelegate 에 등록한다.
- 다음 코드 한줄만 appDelegate 에 추가하면 된다.

```swift
@UIApplicationMain

class AppDelegate: UIResponder, UIApplicationDelegate {

   var window: UIWindow?

   let mediator = AppLifecycleMediator.makeDefaultMediator() // 여기에 추가

   func application(_ application: UIApplication,                                           didFinishLaunchingWithOptions launchOptions:                              [UIApplicationLaunchOptionsKey: Any]?) -> Bool {

      return true

   }
}
```

- objective-c 구현에 붙인다면 조금 애매한데 `willFinishLaunchingWithOptions` 메소드에 다음과 같이 추가하였다. 

```objc
@interface AppDelegate ()<CLLocationManagerDelegate, UNUserNotificationCenterDelegate, FIRMessagingDelegate, AppsFlyerLibDelegate>
{
  AppLifecycleMediator *_mediator; // 여기에 선언하고 
}


...
- (BOOL)application:(UIApplication *)application willFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
  _mediator = [AppLifecycleMediator makeDefaultMediator]; // 구현추가.
  return YES;
}

```

## Reference

- [refactoring-massive-app-delegate](https://www.vadimbulavin.com/refactoring-massive-app-delegate/)
- [Architecting an Analytics Service for iOS Apps – Andreas Lüdemann (andreaslydemann.com)](https://andreaslydemann.com/architecting-an-analytics-service-for-ios-apps/)
