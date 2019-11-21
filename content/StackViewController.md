+++
title = "UIStackViewController"
date = 2019-11-25
category = "UIKit"

[taxonomies]
tags = ["swift","uikit", "uistackview", "uiscrollview","child_viewcontroller"]
+++

UITableView, UICollectionView를 쓰지 않고 간단히 UIScrollView에 UIStackView를 addView해서 사용하는 법
<!-- more -->
```swift
class StackViewController: UIViewController {
    private let scrollView = UIScrollView()
    private let stackView = UIStackView()

    override func viewDidLoad() {
        super.viewDidLoad()
        view.addSubview(scrollView)
        scrollView.addSubview(stackView)
        setupConstraints()
        stackView.axis = .vertical
    }

    private func setupConstraints() {
        scrollView.translatesAutoresizingMaskIntoConstraints = false
        stackView.translatesAutoresizingMaskIntoConstraints = false

        NSLayoutConstraint.activate([
            scrollView.leadingAnchor.constraint(equalTo: view.safeAreaLayoutGuide.leadingAnchor),
            scrollView.trailingAnchor.constraint(equalTo: view.safeAreaLayoutGuide.trailingAnchor),
            scrollView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor),
            scrollView.bottomAnchor.constraint(equalTo: view.safeAreaLayoutGuide.bottomAnchor),

            stackView.leadingAnchor.constraint(equalTo: scrollView.leadingAnchor),
            stackView.trailingAnchor.constraint(equalTo: scrollView.trailingAnchor),
            stackView.topAnchor.constraint(equalTo: scrollView.topAnchor),
            stackView.bottomAnchor.constraint(equalTo: scrollView.bottomAnchor),
            stackView.widthAnchor.constraint(equalTo: view.safeAreaLayoutGuide.widthAnchor)
        ])
    }
}

extension StackViewController {
    func add(_ child: UIViewController, size: CGSize) {
        addChild(child)
        stackView.addArrangedSubview(child.view)
        child.didMove(toParent: self)

        child.view.translatesAutoresizingMaskIntoConstraints = false
        NSLayoutConstraint.activate([
            child.view.heightAnchor.constraint(equalToConstant: size.height),
            child.view.widthAnchor.constraint(equalToConstant:size.width)
        ])
    }

    func remove(_ child: UIViewController) {
        guard child.parent != nil else {
            return
        }

        child.willMove(toParent: nil)
        stackView.removeArrangedSubview(child.view)
        child.view.removeFromSuperview()
        child.removeFromParent()
    }
}
```

### 위 `StackViewController`를 이용하여 height이 500인 3개의 child view controller를 추가

```swift
class ViewController: StackViewController {

    override func viewDidLoad() {
        super.viewDidLoad()

        let vc1 = UIViewController()
        vc1.view.backgroundColor = UIColor.orange

        let vc2 = UIViewController()
        vc2.view.backgroundColor = UIColor.purple

        let vc3 = UIViewController()
        vc3.view.backgroundColor = UIColor.blue

        add(vc1, size: CGSize(width: UIScreen.main.bounds.width, height: 500))
        add(vc2, size: CGSize(width: UIScreen.main.bounds.width, height: 500))
        add(vc3, size: CGSize(width: UIScreen.main.bounds.width, height: 500))
    }
}
```