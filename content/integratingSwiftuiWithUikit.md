+++
title = "integrating swiftui with uikit"
date = 2020-9-14
category = "swift", "swiftui

[taxonomies]
tags = ["swift", "swiftui"]
+++

# integrating swiftui with uikit
- [Link](https://www.avanderlee.com/swiftui/integrating-swiftui-with-uikit/)

## Presenting a SwiftUI view in a UIKit view controller

- UIViewController에서 SwiftUI로 만들어진 View를 present하려면 `UIHostingController`를 이용하여 아래와 같이 사용하면 된다.

```swift
if #available(iOS 13.0, *) {
    presentSwiftUIView()
} else {
    // Fallback on earlier versions
}

func presentSwiftUIView() {
    let swiftUIView = SwiftUIView()
    let hostingController = UIHostingController(rootView: swiftUIView)
    present(hostingController, animated: true, completion: nil)
}
```

## Adding a SwiftUI view to a UIKit view
- 위와 유사하게 present하는것이 아니라 child view로 추가시키려면 아래와 같이 추가 

```swift
func addSwiftUIView() {
    let swiftUIView = SwiftUIView()
    let hostingController = UIHostingController(rootView: swiftUIView)

    /// Add as a child of the current view controller.
    addChild(hostingController)

    /// Add the SwiftUI view to the view controller view hierarchy.
    view.addSubview(hostingController.view)

    /// Setup the constraints to update the SwiftUI view boundaries.
    hostingController.view.translatesAutoresizingMaskIntoConstraints = false
    let constraints = [
        hostingController.view.topAnchor.constraint(equalTo: view.topAnchor),
        hostingController.view.leftAnchor.constraint(equalTo: view.leftAnchor),
        view.bottomAnchor.constraint(equalTo: hostingController.view.bottomAnchor),
        view.rightAnchor.constraint(equalTo: hostingController.view.rightAnchor)
    ]

    NSLayoutConstraint.activate(constraints)

    /// Notify the hosting controller that it has been moved to the current view controller.
    hostingController.didMove(toParent: self)
}
```

## swiftUI view를 추가하는  UIViewController Extension 구현
```swift
extension UIViewController {

    /// Add a SwiftUI `View` as a child of the input `UIView`.
    /// - Parameters:
    ///   - swiftUIView: The SwiftUI `View` to add as a child.
    ///   - view: The `UIView` instance to which the view should be added.
    func addSubSwiftUIView<Content>(_ swiftUIView: Content, to view: UIView) where Content : View {
        let hostingController = UIHostingController(rootView: swiftUIView)

        /// Add as a child of the current view controller.
        addChild(hostingController)

        /// Add the SwiftUI view to the view controller view hierarchy.
        view.addSubview(hostingController.view)

        /// Setup the contraints to update the SwiftUI view boundaries.
        hostingController.view.translatesAutoresizingMaskIntoConstraints = false
        let constraints = [
            hostingController.view.topAnchor.constraint(equalTo: view.topAnchor),
            hostingController.view.leftAnchor.constraint(equalTo: view.leftAnchor),
            view.bottomAnchor.constraint(equalTo: hostingController.view.bottomAnchor),
            view.rightAnchor.constraint(equalTo: hostingController.view.rightAnchor)
        ]

        NSLayoutConstraint.activate(constraints)

        /// Notify the hosting controller that it has been moved to the current view controller.
        hostingController.didMove(toParent: self)
    }
}
```

위와 같이 추가하고 `UIViewController` 에서 아래와 같이 사용하면 된다
```swift
func addSwiftUIView() {
    let swiftUIView = SwiftUIView()
    addSubSwiftUIView(swiftUIView, to: view)
}
```

----
tags : #swiftui, #uikit