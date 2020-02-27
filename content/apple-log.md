+++
title = "usage-official-swift-log"
date = 2020-01-06
category = "swift"

[taxonomies]
tags = ["swift", "log", "spm"]
+++

Apple의 official swift-log package를 이용하여 os unified logging를 사용하는 방법
<!-- more -->

### Configuration
- https://github.com/apple/swift-log
- https://github.com/chrisaljoudi/swift-log-oslog

swift-log의 oslog backend 패키지를 Package.swift에 추가

```swift
dependencies: [
        .package(url: "https://github.com/chrisaljoudi/swift-log-oslog.git", from: "0.2.1")
    ],
targets: [
    .target(
        name: "VExtensionUtility",
        dependencies: [
            "LoggingOSLog"
        ]
    ),
```

### generate Log class
- 다음과 같은 Wrapping class를 생성하면 조금더 편리하게 사용할 수 있다. 


```swift
//
//  Log.swift
//
//  Created by kwanghyun.won` on 2020/01/06.
//

import Logging
import LoggingOSLog

/// Example:
///     Log.setup(subsystem: "com.Vingle.DesignKitDemo", level: .debug)
///     Log.info("setup Log Complete!")
public final class Log {

    public static var `default`: Logger = Logger(label: "com.Vingle.app", factory: LoggingOSLog.init)

    public class func setup(subsystem: String, level: Logger.Level = .debug) {
        self.default = Logger(label: subsystem, factory: LoggingOSLog.init)
        self.default.logLevel = level
    }
}

public extension Log {
    static func trace(_ message: Logger.Message, functionName: String = #function, fileName: String = #file) {
        let className = fileName.extractClassName()
        self.default.trace("\(className):\(functionName) \(message)")
    }

    static func debug(_ message: Logger.Message, functionName: String = #function, fileName: String = #file) {
        let className = fileName.extractClassName()
        self.default.debug("\(className):\(functionName) \(message)")
    }

    static func info(_ message: Logger.Message, functionName: String = #function, fileName: String = #file) {
        let className = fileName.extractClassName()
        self.default.info("\(className):\(functionName) \(message)")
    }

    static func notice(_ message: Logger.Message, functionName: String = #function, fileName: String = #file) {
        let className = fileName.extractClassName()
        self.default.notice("\(className):\(functionName) \(message)")
    }

    static func warning(_ message: Logger.Message, functionName: String = #function, fileName: String = #file) {
        let className = fileName.extractClassName()
        self.default.warning("\(className):\(functionName) \(message)")
    }

    static func error(_ message: Logger.Message, functionName: String = #function, fileName: String = #file) {
        let className = fileName.extractClassName()
        self.default.error("\(className):\(functionName) \(message)")
    }

    static func critical(_ message: Logger.Message, functionName: String = #function, fileName: String = #file) {
        let className = fileName.extractClassName()
        self.default.critical("\(className):\(functionName) \(message)")
    }
}
```

### usage 

- `AppDelegate.swift` 에 Log setting 
> Log.setup(subsystem: "com.Vingle.DesignKitDemo", level: .debug)
- 그 이후 자유롭게 Log 사용 
> Log.debug(" debug message \(variable)")

### Link 
- [migrating-to-unified-logging-console-and-instruments](https://www.raywenderlich.com/605079-migrating-to-unified-logging-console-and-instruments) 
    - raywenderlich 의 os-log tutorial 
