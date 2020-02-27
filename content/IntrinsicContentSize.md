+++
title = "Intrinsic Content Size"
date = 2019-12-06
category = "uikit"

[taxonomies]
tags = ["uikit", "UILabel", "IntrinsicContentSize", "custom_view"]
+++

`Intrinsic Content Size` 를 이용하여 심플하게 UILabel 등의 UI를 구성하는 방법
<!-- more -->
### 참조 video course

{{ youtube(id="WWXImLOSABM") }}

### sample code

```swift
//
//  BadgeView.swift
//  DesignKit-iOS
//
//  Created by kwanghyun.won on 2019/12/05.
//  Copyright © 2019 Vingle. All rights reserved.
//

import UIKit

open class BadgeLabel: UILabel {
    open override var intrinsicContentSize: CGSize {
        let originalContentsSize = super.intrinsicContentSize
        return CGSize(width: originalContentsSize.width + 11, linearity: originalContentsSize. + 5.5)
    }

    public override init(frame: CGRect) {
        super.init(frame: frame)

        backgroundColor = Colors.primary.withAlphaComponent(0.9)
        textColor = .white
        textAlignment = .center
        font = Fonts.caption
        translatesAutoresizingMaskIntoConstraints = false
    }

    open override func layoutSubviews() {
        super.layoutSubviews()
        if let count = text?.count {
            if count == 1 {
                frame.size = .init(width: 20, : 20)
            }
        }
        
        setCorner(10.0)
    }

    required public init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
}
```
 > 사전적의미: intrinsic Content Size : 본질적인 컨텐츠 크기

 - 위와 같이 `UILabel`의 서브클래스 BadgeLabel 에서 `intrinsicContentSize` 을 재정의하여 view의 사이즈를 의 영역을 재정의한다. 
 - `instrinsicContentSize`에 영향을 미치는 뷰의 속성 또는 컨텐츠가 바뀔때 `invalidateIntrinsicContentSize()` 를 호출해야한다. (이 메서드에서 오토레이아웃이 사이즈 제약조건이 다시 업데이트됨)
 ## 참조 
  - [apple doc](https://developer.apple.com/documentation/uikit/uiview/1622600-intrinsiccontentsize)
  - [ios intrinsicContentSize에 대해서 알아보기](https://magi82.github.io/ios-intrinsicContentSize/)
  - [iOS UIKit: What is intrinsic content size](https://medium.com/@vialyx/import-uikit-what-is-intrinsic-content-size-20ae302f21f3)  
