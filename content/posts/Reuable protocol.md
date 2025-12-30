+++
title = "Reuable Protocol"
date = 2019-11-22

[taxonomies]
tags = ["swift","UIKit", "uitableview", "protocol"]
+++

`Reusable` protocol 을 이용하여 cell register , dequeueReusableCell 
<!-- more -->

### protocol implementation

```swift
import UIKit

public protocol Reusable {
    static var identifier: String { get set }
}

public extension UITableView {
    
    /// Reusable 프로토콜을 구현한 TableViewCell 의 register
    ///
    /// For Example:
    /// ```
    /// tableView.register(cellType: TableViewCell.self)
    /// ```
    ///
    /// - Parameter cellType: Reusable type cell
    func register<T: UITableViewCell>(cellType: T.Type) where T: Reusable {
        register(cellType.self, forCellReuseIdentifier: cellType.identifier)
    }
    
    ///  Reusable 프로토콜을 구현한 TableViewCell 을 register후 reuse
    ///
    ///  For Example:
    ///  ```
    ///  let cell = tableView.dequeueReusableCell(for: indexPath, cellType: TableViewCell.self)
    ///  ```
    ///
    /// - Parameters:
    ///   - indexPath: IndexPath
    ///   - cellType: Reusable TableCell type
    func dequeueReusableCell<T: UITableViewCell>(for indexPath: IndexPath,
                                                 cellType: T.Type = T.self) -> T where T: Reusable {
        guard let cell = self.dequeueReusableCell(withIdentifier: cellType.identifier, for: indexPath) as? T else {
            fatalError("Failed to dequeue a cell \(cellType.identifier) matching type \(cellType.self).")
        }
        
        return cell
    }
}
```
