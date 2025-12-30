+++
title = "Sign in with Apple"
date = 2020-12-05

[taxonomies]
tags = ["swift", "apple_signin"]
+++

Apple SignIn feature를 `DelegateProxy` 를 이용하여 Rx Extension 구현
<!-- more -->

## rxswift extension

```swift

import AuthenticationServices
import RxCocoa
import RxSwift
import UIKit

@available(iOS 13.0, *)
extension ASAuthorizationController: HasDelegate {
    public typealias Delegate = ASAuthorizationControllerDelegate
}

@available(iOS 13.0, *)
class ASAuthorizationControllerProxy: DelegateProxy<ASAuthorizationController, ASAuthorizationControllerDelegate>,
    DelegateProxyType,
    ASAuthorizationControllerDelegate,
ASAuthorizationControllerPresentationContextProviding {

    var presentationWindow: UIWindow = UIWindow()

    public init(controller: ASAuthorizationController) {
        super.init(parentObject: controller, delegateProxy: ASAuthorizationControllerProxy.self)
    }

    // MARK: - DelegateProxyType
    public static func registerKnownImplementations() {
        register { ASAuthorizationControllerProxy(controller: $0) }
    }

    // MARK: - Proxy Subject
    internal lazy var didComplete = PublishSubject<ASAuthorization>()

    // MARK: - ASAuthorizationControllerDelegate
    func authorizationController(controller: ASAuthorizationController, didCompleteWithAuthorization authorization: ASAuthorization) {
        didComplete.onNext(authorization)
        didComplete.onCompleted()
    }

    func authorizationController(controller: ASAuthorizationController, didCompleteWithError error: Error) {
        didComplete.onCompleted()
    }

    // MARK: - ASAuthorizationControllerPresentationContextProviding
    func presentationAnchor(for controller: ASAuthorizationController) -> ASPresentationAnchor {
        return presentationWindow
    }

    // MARK: - Completed
    deinit {
        self.didComplete.onCompleted()
    }
}



@available(iOS 13.0, *)
extension Reactive where Base: ASAuthorizationAppleIDProvider {
    public func login(scope: [ASAuthorization.Scope]? = nil) -> Observable<ASAuthorization> {
        let request = base.createRequest()
        request.requestedScopes = scope

        let controller = ASAuthorizationController(authorizationRequests: [request])

        let proxy = ASAuthorizationControllerProxy.proxy(for: controller)

        controller.presentationContextProvider = proxy
        controller.performRequests()

        return proxy.didComplete
    }
}

@available(iOS 13.0, *)
extension Reactive where Base: ASAuthorizationAppleIDButton {
    public func loginOnTap(scope: [ASAuthorization.Scope]? = nil) -> Observable<ASAuthorization> {
        return controlEvent(.touchUpInside)
            .flatMap {
                ASAuthorizationAppleIDProvider().rx.login(scope: scope)
        }
    }

    public func login(scope: [ASAuthorization.Scope]? = nil) -> Observable<ASAuthorization> {
        return ASAuthorizationAppleIDProvider().rx.login(scope: scope)
    }
}
```

- 아래와 같이 사용
```swift
 if #available(iOS 13.0, *) {
            appleSignInButton.rx
                .loginOnTap(scope: [.fullName, .email])
                .map { $0.credential as? ASAuthorizationAppleIDCredential }
                .filterNil()
                .subscribe(onNext: { [weak self] credential in
                    guard let `self` = self else { return }
                    FireBaseAnalytics.shared.sendEvent(name: "login",
                                                       params: ["method":"apple"])

                    CommonSignInRoutines.onTouchConnectWithApple(self)
                    AppleAuthenticationService.shared.signInPublisher.onNext(credential)

                }, onError: { error in
                    logger.error("#### error \(error.localizedDescription)", context: LogCategory.appleLogin)
                }).disposed(by: self.disposeBag)
        }
```

## resource
- https://sarunw.com/posts/sign-in-with-apple-1/
- https://medium.com/@twih1203/애플-로그인-구현하기-sign-in-with-apple-1-5db7606867
-
