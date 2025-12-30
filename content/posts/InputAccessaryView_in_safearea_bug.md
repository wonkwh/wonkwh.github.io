+++
title = "inputAccessoryView SafeLayoutGuide fix"
date = 2019-11-28

[taxonomies]
tags = ["swift","UIKit", "keyboard"]
+++

keyboard 위에 추가 뷰를 만들려면 NotificationHandler로 직접 콘트롤해도 되지만 `inputAccessoryView` 를 override해서 view를 재정의해주면 간단하다 
하지만 iphone x 이상 safelayout 을 인식하지 못하는 문제가 있다 이를 해결하기 위한 코드

<!-- more -->

### inputAccessoryView 재정의

```swift
class MessageInputDemoController: UIViewController {
    lazy var messageInputView: MessageInputContainerView = {
        let view = MessageInputContainerView()
        view.frame = .init(x: 0, y: 0, width: self.view.frame.width, height: 103.0)
        return view
    }()

    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = Colors.background
    }

    override var inputAccessoryView: UIView? {
        return messageInputView
    }

    override var canBecomeFirstResponder: Bool {
        return true
    }
}
```

### SafeLayout 위에 올리기 위한 추가 코드 

```swift
public class MessageInputContainerView: UIView {
    ....
    public override func didMoveToWindow() {
        if let window = self.window {
            self.bottomAnchor.constraint(lessThanOrEqualToSystemSpacingBelow: window.safeAreaLayoutGuide.bottomAnchor,
                                         multiplier: 1.0).isActive = true
        }
    }
}
```

 - 위와 같이 `didMoteToWindow()`메소드에 추가 구현해주면 된다. 

### 참조
 - https://ahbou.org/post/165762292157/iphone-x-inputaccessoryview-fix
