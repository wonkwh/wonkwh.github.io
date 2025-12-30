+++
title = "Building Better View"
date = 2020-03-15
category = "uikit"

[taxonomies]
tags = ["swift","uikit", "viewModel"]
+++

Model, ViewModel 을 view로 보여주어야 할떄 `viewData protocol` 을 이용하여 일관적으로 view를 configure 하는 방법

- original article 
  - [Building better view](https://www.fabisevi.ch/2019/12/26/building-better-views-part-i/)
<!-- more -->

## 개요
1. A UIView instance. This is your standard view that you’ll be displaying in an app. It can be a regular class, or a custom subclass as you need.
2. A ViewData protocol. This is what’s going to keep track of the data that needs to be displayed in your view. Most commonly this will be a slice of a model, used specifically for rendering the view.
3. A configure(viewData: ViewData) function. This is what’s going to map your View to your ViewData.


![Diagram](https://fabisevi.ch//assets/img/view_data_diagram.png)

## 일반적인 구현
### API
-  `itunes api` 를 이용하여 music track을 보여주는 예제 
-  IU의 음악을 검색하려면 다음과 같은 rest api 
   -   `https://itunes.apple.com/search?term=IU&offset=0&limit=20`
   -   깔끔히 보려면 요사이트를 이용하면 좋다 
       -   https://jsonformatter.org/json-pretty-print
   -   결과는  아래와 같다 

```json
{
  "resultCount": 20,
  "results": [
    {
      "wrapperType": "track",
      "kind": "song",
      "artistId": 409076743,
      "collectionId": 1438614238,
      "trackId": 1438614348,
      "artistName": "IU",
      "collectionName": "Bbibbi - Single",
      "trackName": "Bbibbi",
      "collectionCensoredName": "Bbibbi - Single",
      "trackCensoredName": "Bbibbi",
      "artistViewUrl": "https://music.apple.com/us/artist/iu/409076743?uo=4",
      "collectionViewUrl": "https://music.apple.com/us/album/bbibbi/1438614238?i=1438614348&uo=4",
      "trackViewUrl": "https://music.apple.com/us/album/bbibbi/1438614238?i=1438614348&uo=4",
      "previewUrl": "https://audio-ssl.itunes.apple.com/itunes-assets/AudioPreview128/v4/12/d9/6b/12d96b32-66b2-359c-b4d9-9d595b585a0d/mzaf_6786449235518924873.plus.aac.p.m4a",
      "artworkUrl30": "https://is5-ssl.mzstatic.com/image/thumb/Music118/v4/43/cf/eb/43cfeb10-6da1-d6dd-1cc5-e1f461947052/source/30x30bb.jpg",
      "artworkUrl60": "https://is5-ssl.mzstatic.com/image/thumb/Music118/v4/43/cf/eb/43cfeb10-6da1-d6dd-1cc5-e1f461947052/source/60x60bb.jpg",
      "artworkUrl100": "https://is5-ssl.mzstatic.com/image/thumb/Music118/v4/43/cf/eb/43cfeb10-6da1-d6dd-1cc5-e1f461947052/source/100x100bb.jpg",
      "collectionPrice": 1.29,
      "trackPrice": 1.29,
      "releaseDate": "2018-10-10T12:00:00Z",
      "collectionExplicitness": "notExplicit",
      "trackExplicitness": "notExplicit",
      "discCount": 1,
      "discNumber": 1,
      "trackCount": 1,
      "trackNumber": 1,
      "trackTimeMillis": 194425,
      "country": "USA",
      "currency": "USD",
      "primaryGenreName": "K-Pop",
      "isStreamable": true
    },
    ...
```
### Model
- 필요한 필드만 빼서 만든 model object


```swift
struct SearchResult: Decodable {
    let resultCount: Int
    let results: [Result]
}

struct Result: Decodable {
    let trackId: Int
    let trackName: String
    let primaryGenreName: String
    var averageUserRating: Float?
    var screenshotUrls: [String]?
    let artworkUrl100: String // app icon
    var formattedPrice: String?
    var description: String?
    var releaseNotes: String?
    var artistName: String?
    var collectionName: String?
}

``` 
### view 구현
- track list에 보여줄 cell view

```swift
class ListCell: UICollectionViewCell {
    let imageView = AspectFitImageView(image: nil, cornerRadius: 16)
    let nameLabel = UILabel(text: "TrackName", font: .boldSystemFont(ofSize: 18))
    let subtitleLabel = UILabel(text: "Subtitle Label", font: .systemFont(ofSize: 16), numberOfLines: 2)

    override init(frame: CGRect) {
        super.init(frame: frame)

        hstack(
            imageView.withWidth(80),
            stack(nameLabel, subtitleLabel, spacing: 4),
            spacing: 16
        ).withMargins(.allSides(16))
    }

    required init?(coder aDecoder: NSCoder) {
        fatalError()
    }
}
```

-  populate data 

```swift
    override func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
        let cell = collectionView.dequeueReusableCell(withReuseIdentifier: cellId, for: indexPath) as! ListCell

        let track = results[indexPath.item]

        cell.nameLabel.text = track.trackName
        if let url = URL(string: track.artworkUrl100) {
            Nuke.loadImage(with: url, into: cell.imageView)
        }

        cell.subtitleLabel.text = "\(track.artistName ?? "") • \(track.collectionName ?? "")"
```
-  full source code는 다음 repo 를 보며 된다. 
   -  https://github.com/wonkwh/BlogShowcase/tree/master/BlogShowcase/BlogShowcase

## viewData protocol
- 실제 cell에 보여줄 데이터 프로토콜을 정의하고 Model object의 extension으로 구현한다.
```swift 
protocol ResultViewData {
    var title: String { get }
    var subtitle: String { get }
    var albumCoverURL: URL? { get }
}


extension Result: ResultViewData {
    var title: String {
        return trackName
    }

    var subtitle: String {
        return "\(artistName ?? "") • \(collectionName ?? "")"
    }

    var albumCoverURL: URL? {
        return URL(string: artworkUrl100)
    }
}
```
- view cell에 extension 으로 `func configure(viewData: ..)` 를 구현

```swift
extension ListCell {
    func configure(viewData: ResultViewData) {
        self.nameLabel.text = viewData.title
        self.subtitleLabel.text = viewData.subtitle
        if let url = viewData.albumCoverURL {
            Nuke.loadImage(with: url, into: self.imageView)
        }
    }
}

```
- populate data

```swift
    override func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
        let cell = collectionView.dequeueReusableCell(withReuseIdentifier: cellId, for: indexPath) as! ListCell

        let track = results[indexPath.item]
        cell.configure(viewData: track)

```

위와 같은 형태로 일관성있게 view model -> view 로 구성할수 있다. 

## 참조
 - https://www.fabisevi.ch/2019/12/26/building-better-views-part-i/
 - https://fabisevi.ch//2019/12/26/building-better-views-part-ii