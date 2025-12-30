+++
title = "fluid interface effect"
date = 2019-11-27

[taxonomies]
tags = ["swift","UIKit", "animation", "wwdc"]
+++

wwdc2018 세션중 하나인 [Designing Fluid Interfaces](https://developer.apple.com/videos/play/wwdc2018/803/) 를 직접 구현한 opensource중 button 구현에 대해 정리
- [fluid-interfaces](https://github.com/nathangitter/fluid-interfaces)
<!-- more -->

### ios app중 계산기앱의 버튼과 같은 효과

- Button구현의 풀소스는 다음과 같다 
```swift
//
//  Button.swift
//  Example
//
//  Created by kwanghyun.won on 2019/11/07.
//  Copyright © 2019 vingle. All rights reserved.
//

import UIKit

open class Button: UIButton {
    private struct Const {
        static let borderWidth: CGFloat = 1
        static let cornerRadius: CGFloat = 2
    }

    open override var intrinsicContentSize: CGSize {
        var origSize = super.intrinsicContentSize
        origSize.height = self.size == .large ? origSize.height + 4.0 : origSize.height + 2.0
        //dump(origSize)
        return origSize
    }

    open override func tintColorDidChange() {
        super.tintColorDidChange()
        update()
    }

    open var style: ButtonStyle = .flat {
        didSet {
            if style != oldValue {
                update()
            }
        }
    }

    open var size: ButtonSize = .large {
        didSet {
            if size != oldValue {
                update()
            }
        }
    }

    open override var isHighlighted: Bool {
        didSet {
            if isHighlighted != oldValue {
                update()
            }
        }
    }

    open override var isEnabled: Bool {
        didSet {
            if isEnabled != oldValue {
                update()
            }
        }
    }

    private var animator = UIViewPropertyAnimator()
    private let activationFeedBackGenerator = UIImpactFeedbackGenerator(style: .light)

    public init(style: ButtonStyle = .flat, size: ButtonSize = .large) {
        self.style = style
        self.size = size
        super.init(frame: .zero)
        initialize()
    }
    
    public override init(frame: CGRect) {
        super.init(frame: frame)
        initialize()
    }

    public required init?(coder aDecoder: NSCoder) {
        super.init(coder: aDecoder)
        initialize()
    }

    open override func layoutSubviews() {
        super.layoutSubviews()
        layer.cornerRadius = Const.cornerRadius
        layer.borderWidth = style == .ghost ? Const.borderWidth : 0.0
    }

    func initialize() {
        update()
        NotificationCenter.default.addObserver(self, selector: #selector(didContentSizeCategoryDidChange),
                                               name: UIContentSizeCategory.didChangeNotification, object: nil)
        addTarget(self, action: #selector(didTouchDown), for: [.touchDown, .touchDragEnter])
        addTarget(self, action: #selector(didTouchUp), for: [.touchUpInside, .touchDragExit, .touchCancel])
    }

    private func update() {
        setTitleColor(style.titleColor, for: .normal)
        setTitleColor(style.hightlightTitleColor, for: .highlighted)
        setTitleColor(style.disableTitleColor, for: .disabled)

        self.backgroundColor = isEnabled ? style.backgroundColor : style.disableBackgroundColor

        layer.borderColor = isEnabled ? style.titleColor.cgColor : style.disableTitleColor.cgColor

        titleLabel?.font = size.titleFont
        let lineHeight: CGFloat = (size == .large) ? 22.0 : 19.0
        titleLabel?.setLineHeight(lineHeight: lineHeight)

        contentEdgeInsets = size.contentEdgeInsets
    }

}

// MARK: - event
extension Button {
    @objc private func didContentSizeCategoryDidChange() {
        update()
    }

    @objc private func didTouchDown() {
        animator.stopAnimation(true)
        backgroundColor = style.hightlightBackgroundColor
        layer.borderColor = Colors.Button.borderHighlighted.cgColor
        activationFeedBackGenerator.impactOccurred()
    }

    @objc private func didTouchUp() {
        animator = UIViewPropertyAnimator(duration: 0.5, curve: .easeOut, animations: {
            self.backgroundColor = self.style.backgroundColor
            self.layer.borderColor = Colors.Button.border.cgColor
        })
        animator.startAnimation()
    }
}

``` 

- 위소스중 계산기 버튼과 같은 효과를 주기위한 효과는 

```swift 
private var animator = UIViewPropertyAnimator()
///...

    @objc private func didTouchDown() {
        animator.stopAnimation(true)
        backgroundColor = style.hightlightBackgroundColor
        //..
    }

    @objc private func didTouchUp() {
        animator = UIViewPropertyAnimator(duration: 0.5, curve: .easeOut, animations: {
            self.backgroundColor = self.style.backgroundColor
            self.layer.borderColor = Colors.Button.border.cgColor
        })
        animator.startAnimation()
    }

```

## flashlight button 과 같은 확대,축소 애니메이션과 진동 feedback

    위 버튼 구현 소스에는 확대,축소 애니메이션은 구현되어 있지 않다 

- 진동 피드백 효과
```swift

    private let activationFeedBackGenerator = UIImpactFeedbackGenerator(style: 
    .light)
    //...
    @objc private func didTouchDown() {
        activationFeedBackGenerator.impactOccurred()
    }
```

- 확대,축소 효과를 주려면 아래 코드를 추가하면 된다. 
```swift
 private func animate() {
        let timingParameters = UISpringTimingParameters(damping: 0.4, response: 0.2)
        let animator = UIViewPropertyAnimator(duration: 0, timingParameters: timingParameters)
        animator.addAnimations {
            self.transform = CGAffineTransform(scaleX: 1, y: 1)
            self.backgroundColor = self.isOn ? self.onColor : self.offColor
        }
        animator.isInterruptible = true
        animator.startAnimation()
    }
```
