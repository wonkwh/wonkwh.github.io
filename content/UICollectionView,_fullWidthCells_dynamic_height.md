+++
title = "UICollectionView, full width cells, allow autolayout dynamic height"
date = 2020-02-27
category = "uikit"

[taxonomies]
tags = ["swift","uikit", "uicollectionview", "autolayout"]
+++

CollectionView 를 이용해 tableview와 유사한 ListView를 사용하면서 tableview의 dynamic height와 같이 autolayout 기반의 dynamic height를 구현하는 코드

<!-- more -->

### CollectionView 기반의 ListController

```swift
import UIKit

extension UIView {
    func insect(by insect: UIEdgeInsets) {
        guard let superview = superview else {
            return
        }

        translatesAutoresizingMaskIntoConstraints = false

        leadingAnchor.constraint(equalTo: superview.leadingAnchor, constant: insect.left).isActive = true
        trailingAnchor.constraint(equalTo: superview.trailingAnchor, constant: -insect.right).isActive = true
        topAnchor.constraint(equalTo: superview.topAnchor, constant: insect.top).isActive = true
        bottomAnchor.constraint(equalTo: superview.bottomAnchor, constant: -insect.bottom).isActive = true
    }
}
class ListCell: UICollectionViewCell {
    let titleLabel: UILabel = {
        let label = UILabel()
        label.text = "App Section Title"
        label.font = UIFont.preferredFont(forTextStyle: .body)
        label.numberOfLines = 0
        label.textColor = .black
        return label
    }()


    override init(frame: CGRect) {
        super.init(frame: frame)
        contentView.backgroundColor = .green
        contentView.addSubview(titleLabel)
        titleLabel.insect(by: .init(top: 5, left: 5, bottom: 5, right: 5))
    }

    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
}

let items = [
        "Lorem ipsum dolor sit amet.",
        "Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris. Lorem ipsum dolor sit amet, consectetur adipiscing elit.",
        "Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.",
        "Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt.",
        "Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.",
        "Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt.",
        "Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.",
        "Lorem ipsum. Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris."
]

class ListController: UIViewController {
    lazy var collectionView: UICollectionView = {
        let layout = UICollectionViewFlowLayout()
        layout.scrollDirection = .vertical
        //layout.itemSize = .init(width: self.view.frame.width, height: 80)
        layout.sectionInset = .init(top: 12, left: 0, bottom: 0, right: 0)
        let view = UICollectionView(frame: .zero, collectionViewLayout: layout)
        view.backgroundColor = .white
        view.register(ListCell.self, forCellWithReuseIdentifier: "ListCell")
        view.dataSource = self
        view.delegate = self
        view.alwaysBounceVertical = true
        view.translatesAutoresizingMaskIntoConstraints = false
        return view
    }()

    override func viewDidLoad() {
        super.viewDidLoad()
        //navigationController?.navigationBar.prefersLargeTitles = true
        navigationItem.title = "ListController"
        view.addSubview(collectionView)

        NSLayoutConstraint.activate([
            collectionView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor),
            collectionView.leadingAnchor.constraint(equalTo: view.safeAreaLayoutGuide.leadingAnchor),
            collectionView.trailingAnchor.constraint(equalTo: view.safeAreaLayoutGuide.trailingAnchor),
            collectionView.bottomAnchor.constraint(equalTo: view.safeAreaLayoutGuide.bottomAnchor)
        ])

    }
}

// MARK: - UICollectionViewDataSource
extension ListController: UICollectionViewDataSource {
    func collectionView(_ collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int {
        return 8
    }

    func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
        let cell = collectionView.dequeueReusableCell(withReuseIdentifier: "ListCell", for: indexPath) as! ListCell
        cell.titleLabel.text = items[indexPath.item]
        return cell
    }
}

extension ListController: UICollectionViewDelegateFlowLayout {
    func collectionView(_ collectionView: UICollectionView, layout collectionViewLayout: UICollectionViewLayout, sizeForItemAt indexPath: IndexPath) -> CGSize {
        return .init(width: self.view.frame.width, height: 80)
    }
}

```
 - `UICollectionViewDelegateFlowLayout`  의 sizeForItemAt method를 오버라이드하여 width를 full width로 설정하면 된다. 
 - 하지만 height를 위와 같이 80으로 고정하면 contents내용에 따라 동적으로 height를 구성하기 꽤 빡시다. 
 ![no_dynamic_height](https://user-images.githubusercontent.com/911787/75467624-493c1c80-59cf-11ea-8c3e-8c472f142319.png)

### Solution
- 참조 링크를 보면 꽤 다양한 방법들이 존재하나 가장 심플한 방법
- `UICollectionViewDelegateFlowLayout` 관련 코드를 삭제
```swift
    lazy var collectionView: UICollectionView = {
        let layout = UICollectionViewFlowLayout()
        ...
        view.delegate = self // 요 라인만 삭제
        ...
    }()

...
extension ListController: UICollectionViewDelegateFlowLayout {
    func collectionView(_ collectionView: UICollectionView, layout collectionViewLayout: UICollectionViewLayout, sizeForItemAt indexPath: IndexPath) -> CGSize {
        return .init(width: self.view.frame.width, height: 80)
    }
}
```
- 다음 코드를 추가
  - ListCell 에 width anchor를 추가한다.
    ```swift
    class ListCell: UICollectionViewCell {
        ...
      override init(frame: CGRect) {
          super.init(frame: frame)
          contentView.backgroundColor = .green
          contentView.addSubview(titleLabel)
          //하단 라인 추가
          titleLabel.widthAnchor.constraint(equalToConstant: UIScreen.main.bounds.size.width - 10).isActive = true
          titleLabel.insect(by: .init(top: 5, left: 5, bottom: 5, right: 5))
      }

    }
    ...
    ```

  - ListController 에 `itemSize` ,`estimatedItemSize` 를 추가
    ```swift
    lazy var collectionView: UICollectionView = {
        let layout = UICollectionViewFlowLayout()
        ...
        // 하단 두라인 추가

        layout.itemSize = UICollectionViewFlowLayout.automaticSize
        layout.estimatedItemSize = UICollectionViewFlowLayout.automaticSize

        ...
    }()
    ...
    ```
    ![dynamic_height](https://user-images.githubusercontent.com/911787/75467646-50632a80-59cf-11ea-9023-5f233f0ce146.png)


### 참조
 - https://stackoverflow.com/questions/44187881/uicollectionview-full-width-cells-allow-autolayout-dynamic-height