+++
title = "StackViewControllable"
date = 2019-11-18
categories = ["UIKit"]

[taxonomies]
tags = ["swift","uikit", "uistackview", "uiscrollview","child_viewcontroller","protocol"]

+++

상속대신 프로토콜을 이용하여 StackViewController 구현방법
<!-- more -->

### protocol implementation

```swift
protocol StackViewControllerProtocol: UIViewController {
    var scrollView: UIScrollView { get set }
    var stackView: UIStackView { get set }
}

extension StackViewControllerProtocol {
    func add(_ child: UIViewController, size: CGSize) {
        addChild(child)
        stackView.addArrangedSubview(child.view)
        child.didMove(toParent: self)
        child.view.translatesAutoresizingMaskIntoConstraints = false
        NSLayoutConstraint.activate([
            child.view.heightAnchor.constraint(equalToConstant: size.height),
            child.view.widthAnchor.constraint(equalToConstant: size.width)
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
    func setUpStackAndScrollView() {
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
```
### protocol composition

```swift
class ViewController: UIViewController, StackViewControllerProtocol {
    var scrollView: UIScrollView = UIScrollView()
    var stackView: UIStackView  = UIStackView()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        setUpStackAndScrollView()
        
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