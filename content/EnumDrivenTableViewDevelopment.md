+++
title = "UEnum-Driven TableView Development"
date = 2020-11-09
category = "uikit"

[taxonomies]
tags = ["swift","uikit", "uicollectionview", "uitableview", "enum", "raywenderlich"]
+++

# Enum-Driven TableView Development
- [Link](https://www.raywenderlich.com/5542-enum-driven-tableview-development)
- 위의 글을 참조하여 itunes search list에 적용한다. 

- original source 
```swift
class ListController: UICollectionViewController, UICollectionViewDelegateFlowLayout {

    fileprivate let cellId = "cellId"
    fileprivate let footerId = "footerId"

    override func viewDidLoad() {
        super.viewDidLoad()
        collectionView.backgroundColor = .white

        collectionView.register(ListCell.self, forCellWithReuseIdentifier: cellId)
        collectionView.register(LoadingFooter.self, forSupplementaryViewOfKind: UICollectionView.elementKindSectionFooter, withReuseIdentifier: footerId)

        fetchData()
    }

    var results = [Result]()

    fileprivate let searchTerm = "BTS"

    fileprivate func fetchData() {
        let urlString = "https://itunes.apple.com/search?term=\(searchTerm)&offset=0&limit=20"
        Service.shared.fetchGenericJSONData(urlString: urlString) { (searchResult: SearchResult?, err) in

            if let err = err {
                print("Failed to paginate data:", err)
                return
            }

            self.results = searchResult?.results ?? []
            DispatchQueue.main.async {
                self.collectionView.reloadData()
            }
        }
    }

    override func collectionView(_ collectionView: UICollectionView, viewForSupplementaryElementOfKind kind: String, at indexPath: IndexPath) -> UICollectionReusableView {
        let footer = collectionView.dequeueReusableSupplementaryView(ofKind: kind, withReuseIdentifier: footerId, for: indexPath)
        return footer
    }

    func collectionView(_ collectionView: UICollectionView, layout collectionViewLayout: UICollectionViewLayout, referenceSizeForFooterInSection section: Int) -> CGSize {
        let height: CGFloat = isDonePaginating ? 0 : 100
        return .init(width: view.frame.width, height: height)
    }

    override func collectionView(_ collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int {
        return results.count
    }

    var isPaginating = false
    var isDonePaginating = false

    override func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
        let cell = collectionView.dequeueReusableCell(withReuseIdentifier: cellId, for: indexPath) as! ListCell

        let track = results[indexPath.item]
        cell.configure(viewData: track)
        // initiate pagination
        if indexPath.item == results.count - 1 && !isPaginating {
            print("fetch more data")

            isPaginating = true

            let urlString = "https://itunes.apple.com/search?term=\(searchTerm)&offset=\(results.count)&limit=20"
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
                    self.collectionView.reloadData()
                }
                self.isPaginating = false
            }
        }

        return cell
    }

    func collectionView(_ collectionView: UICollectionView, layout collectionViewLayout: UICollectionViewLayout, sizeForItemAt indexPath: IndexPath) -> CGSize {
        return .init(width: view.frame.width, height: 100)
    }

}

```

## Different States
- 잘 디자인된 리스트는 다음의 4가지 state를 가진다
	- Loading: The app is busy fetching new data.
	- Error: A service call or another operation has failed.
	- Empty: The service call has returned no data.
	- Populated: The app has retrieved data to display.

## state enum을 이용하여 refactoring
- enum 추가 
```swift
    case loading
    case paging([Result])
    case populated([Result])
    case empty
    case error(Error)

    var currentResults: [Result] {
        switch self {
        case .paging(let results):
            return results
        case .populated(let results):
            return results
        default:
            return []
        }
    }

```

- property observer를 적용한 state variable 추가 
- 다른곳에 사용한 collectionView.reloadData() 는 모두 지운다.
```swift
    var state = State.loading {
        didSet {
            DispatchQueue.main.async {
                self.collectionView.reloadData()
            }
        }
    }

```

- `results` 전역 변수를 지우고 `state.currentResults` 로 변경한다. 
- `isPaginating`, `isDonePaginating` 전역변수도 지우고 state 변경으로 refactoring

```swift
    fileprivate func loadPage() {
        state = .paging(state.currentResults)
        let urlString = "https://itunes.apple.com/search?term=\(searchTerm)&offset=\(state.currentResults.count)&limit=20"
        Service.shared.fetchGenericJSONData(urlString: urlString) { (searchResult: SearchResult?, err) in
            if let err = err {
                self.state = .error(err)
                return
            }

            if searchResult?.results.count == 0 {
                self.state = .empty
                return
            }

            sleep(2)

            var allResults = self.state.currentResults
            allResults += searchResult?.results ?? []
            self.state = .populated(allResults)
        }
    }
```

## final source 
 - [here](https://github.com/wonkwh/BlogShowcase/blob/develop/BlogShowcase/BlogShowcase/showcase/EnumListController.swift)

## next 
 - swift composale architecture 를 이용하여 변경 
