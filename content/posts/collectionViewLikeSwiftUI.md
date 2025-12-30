+++
title = "ListController like SwifUI's List"
date = 2020-03-16

[taxonomies]
tags = ["swift","uikit", "uicollectionview"]
+++

`Micro` Lib를 이용하여 swiftUI의 `List`와 유사하게 collectionView를 구성
- original article 
  - [How-to-build-SwiftUI-style-UICollectionView-data-source-in-Swift](https://onmyway133.github.io/blog/How-to-build-SwiftUI-style-UICollectionView-data-source-in-Swift/)
<!-- more -->
## Implementation
 - iOS13 이상의 SwiftUI로 List를 구성하는 소스는 아래와 같이 직관적이다. 

```swift
var body: some View {
    List {
        ForEach(blogs) { blog in
            VStack {
                Text(blog.name)
            }
            .onTap {
                print("cell was tapped")
            }
        }
    }
}
```

- 전 Building Better View에서 수정했던 UI를 아래와 **Micro** lib 사용하면 아래와 같이 구현할 수 있다.
- Section Header, Footer View를 구현하기 위해 라이브러리 소스코드를 일부 수정하였다. 

```swift
import UIKit
import LBTATools
import Nuke

class CustomDataSource: DataSource {
    func collectionView(_ collectionView: UICollectionView, viewForSupplementaryElementOfKind kind: String, at indexPath: IndexPath) -> UICollectionReusableView {
        let footer = collectionView.dequeueReusableSupplementaryView(ofKind: kind, withReuseIdentifier: "footerId", for: indexPath)
        return footer
    }
}

class MicroListController: UIViewController {
    var dataSource: DataSource!

    lazy var collectionView: UICollectionView = {
        let layout = UICollectionViewFlowLayout()
        let view = UICollectionView(frame: .zero, collectionViewLayout: layout)
        view.backgroundColor = .white
        return view
    }()

    var results = [Result]()

    override func viewDidLoad() {
        super.viewDidLoad()

        collectionView.register(LoadingFooter.self, forSupplementaryViewOfKind: UICollectionView.elementKindSectionFooter, withReuseIdentifier: "footerId")
        view.addSubview(collectionView)
        collectionView.fillSuperviewSafeAreaLayoutGuide()

        //micro
        dataSource = CustomDataSource(collectionView: collectionView)
        collectionView.dataSource = dataSource
        collectionView.delegate = dataSource

        setDataSource()
        fetchData()
    }

    var isPaginating = false
    var isDonePaginating = false

    func setDataSource() {
        dataSource.state = forEach(results) { result in
            Cell<ListCell>() { context, cell in
                cell.configure(viewData: result)

                // initiate pagination
                if context.indexPath.item == self.results.count - 5 && !self.isPaginating {
                    print("fetch more data")

                    self.isPaginating = true

                    let urlString = "https://itunes.apple.com/search?term=\(self.searchTerm)&offset=\(self.results.count)&limit=20"
                    Service.shared.fetchGenericJSONData(urlString: urlString) { (searchResult: SearchResult?, err) in

                        if let err = err {
                            print("Failed to paginate data:", err)
                            return
                        }

                        if searchResult?.results.count == 0 {
                            self.isDonePaginating = true
                        }

                        sleep(2)

                        self.results += searchResult?.results ?? []
                        DispatchQueue.main.async {
                            self.setDataSource()
                        }
                        self.isPaginating = false
                    }
                }
            }
            .onSelect { context in
                print("cell at index \(context.indexPath.item) is selected")
            }
            .onSize { context -> CGSize in
                return .init(width: self.view.frame.width, height: 100.0)
            }
            .onFooterSize { context -> CGSize in
                let height: CGFloat = self.isDonePaginating ? 0 : 100
                return .init(width: self.view.frame.width, height: height)
            }
        }
    }


    fileprivate let searchTerm = "IZONE"

    fileprivate func fetchData() {
        let urlString = "https://itunes.apple.com/search?term=\(searchTerm)&offset=0&limit=20"
        Service.shared.fetchGenericJSONData(urlString: urlString) { (searchResult: SearchResult?, err) in

            if let err = err {
                print("Failed to paginate data:", err)
                return
            }

            self.results = searchResult?.results ?? []
            DispatchQueue.main.async {
                self.setDataSource()
            }
        }
    }
}


```
## fullSource
 - https://github.com/wonkwh/BlogShowcase/tree/feature/apply_micro
