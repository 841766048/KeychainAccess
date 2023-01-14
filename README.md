# KeychainAccess

[![Build Status](https://travis-ci.com/kishikawakatsumi/KeychainAccess.svg?branch=master)](https://travis-ci.com/kishikawakatsumi/KeychainAccess)
[![Carthage compatible](https://img.shields.io/badge/Carthage-compatible-4BC51D.svg?style=flat)](https://github.com/Carthage/Carthage)
[![SPM supported](https://img.shields.io/badge/SPM-supported-DE5C43.svg?style=flat)](https://swift.org/package-manager)
[![Version](https://img.shields.io/cocoapods/v/KeychainAccess.svg)](http://cocoadocs.org/docsets/KeychainAccess)
[![Platform](https://img.shields.io/cocoapods/p/KeychainAccess.svg)](http://cocoadocs.org/docsets/KeychainAccess)

KeychainAccess 是适用于 iOS 和 OS X 的 Keychain 的简单 Swift 包装器。使使用 Keychain API 变得极其简单，并且在 Swift 中使用起来更容易接受。

<img src="https://raw.githubusercontent.com/kishikawakatsumi/KeychainAccess/master/Screenshots/01.png" width="320px" />
<img src="https://raw.githubusercontent.com/kishikawakatsumi/KeychainAccess/master/Screenshots/02.png" width="320px" />
<img src="https://raw.githubusercontent.com/kishikawakatsumi/KeychainAccess/master/Screenshots/03.png" width="320px" />

## :bulb: 特征

- 简单的界面
- 支持访问组
- [支持辅助功能](#accessibility)
- [支持iCloud分享g](#icloud_sharing)
- **[支持 TouchID 和钥匙串集成 (iOS 8+)](#touch_id_integration)**
- **[支持共享网络凭证 (iOS 8+)](#shared_web_credentials)**
- [适用于 iOS 和 macOS](#requirements)
- [支持 watchOS 和 tvOS](#requirements)
- **[支持Mac Catalyst](#requirements)**
- **[兼容 Swift 3、4 和 5](#requirements)**

## :book: 用法

##### :eyes: 另请参阅:  
- [:link: iOS Example Project](https://github.com/kishikawakatsumi/KeychainAccess/tree/master/Examples/Example-iOS)

### :key: 基础用法

#### 保存应用密码

```swift
let keychain = Keychain(service: "com.example.github-token")
keychain["kishikawakatsumi"] = "01234567-89ab-cdef-0123-456789abcdef"
```

#### 保存互联网密码

```swift
let keychain = Keychain(server: "https://github.com", protocolType: .https)
keychain["kishikawakatsumi"] = "01234567-89ab-cdef-0123-456789abcdef"
```

### :key: 实例化

#### 为应用程序密码创建钥匙串

```swift
let keychain = Keychain(service: "com.example.github-token")
```

```swift
let keychain = Keychain(service: "com.example.github-token", accessGroup: "12ABCD3E4F.shared")
```

#### 为互联网密码创建钥匙串

```swift
let keychain = Keychain(server: "https://github.com", protocolType: .https)
```

```swift
let keychain = Keychain(server: "https://github.com", protocolType: .https, authenticationType: .htmlForm)
```

### :key: 添加item

#### 下标方式添加

##### value 是 String 类型

```swift
keychain["kishikawakatsumi"] = "01234567-89ab-cdef-0123-456789abcdef"
```

```swift
keychain[string: "kishikawakatsumi"] = "01234567-89ab-cdef-0123-456789abcdef"
```

##### value 是 NSData 类型

```swift
keychain[data: "secret"] = NSData(contentsOfFile: "secret.bin")
```

#### Set 方式

```swift
keychain.set("01234567-89ab-cdef-0123-456789abcdef", key: "kishikawakatsumi")
```

#### 错误处理

```swift
do {
    try keychain.set("01234567-89ab-cdef-0123-456789abcdef", key: "kishikawakatsumi")
}
catch let error {
    print(error)
}
```

### :key: 获取 item

#### 下标方式获取

##### String 类型 (如果值是NSData，尝试转换为字符串)

```swift
let token = keychain["kishikawakatsumi"]
```

```swift
let token = keychain[string: "kishikawakatsumi"]
```

##### NSData 类型

```swift
let secretData = keychain[data: "secret"]
```

#### get 方法

##### 获取字符串类型数据

```swift
let token = try? keychain.get("kishikawakatsumi")
```

```swift
let token = try? keychain.getString("kishikawakatsumi")
```

##### 获取NSData类型数据

```swift
let data = try? keychain.getData("kishikawakatsumi")
```

### :key: 删除 item

#### 下标方式

```swift
keychain["kishikawakatsumi"] = nil
```

#### remove 方法

```swift
do {
    try keychain.remove("kishikawakatsumi")
} catch let error {
    print("error: \(error)")
}
```

### :key: 设置标签(Label)和注释(Comment)

```swift
let keychain = Keychain(server: "https://github.com", protocolType: .https)
do {
    try keychain
        .label("github.com (kishikawakatsumi)")
        .comment("github access token")
        .set("01234567-89ab-cdef-0123-456789abcdef", key: "kishikawakatsumi")
} catch let error {
    print("error: \(error)")
}
```

### :key: 获取其他属性

#### 持久引用

```swift
let keychain = Keychain()
let persistentRef = keychain[attributes: "kishikawakatsumi"]?.persistentRef
...
```

#### 创建日期

```swift
let keychain = Keychain()
let creationDate = keychain[attributes: "kishikawakatsumi"]?.creationDate
...
```

#### 所有属性

```swift
let keychain = Keychain()
do {
    let attributes = try keychain.get("kishikawakatsumi") { $0 }
    print(attributes?.comment)
    print(attributes?.label)
    print(attributes?.creator)
    ...
} catch let error {
    print("error: \(error)")
}
```

##### 下标方式

```swift
let keychain = Keychain()
if let attributes = keychain[attributes: "kishikawakatsumi"] {
    print(attributes.comment)
    print(attributes.label)
    print(attributes.creator)
}
```

### :key: 配置（辅助功能、共享、iCloud 同步）

**提供流畅的接口**

```swift
let keychain = Keychain(service: "com.example.github-token")
    .label("github.com (kishikawakatsumi)")
    .synchronizable(true)
    .accessibility(.afterFirstUnlock)
```

#### <a name="accessibility"> 辅助功能

##### 默认可访问性匹配后台应用程序 (=kSecAttrAccessibleAfterFirstUnlock)

```swift
let keychain = Keychain(service: "com.example.github-token")
```

##### 对于后台应用

###### 创建实例

```swift
let keychain = Keychain(service: "com.example.github-token")
    .accessibility(.afterFirstUnlock)

keychain["kishikawakatsumi"] = "01234567-89ab-cdef-0123-456789abcdef"
```

###### 一次

```swift
let keychain = Keychain(service: "com.example.github-token")

do {
    try keychain
        .accessibility(.afterFirstUnlock)
        .set("01234567-89ab-cdef-0123-456789abcdef", key: "kishikawakatsumi")
} catch let error {
    print("error: \(error)")
}
```

##### 对于前台应用

###### 创建实例

```swift
let keychain = Keychain(service: "com.example.github-token")
    .accessibility(.whenUnlocked)

keychain["kishikawakatsumi"] = "01234567-89ab-cdef-0123-456789abcdef"
```

###### 一次

```swift
let keychain = Keychain(service: "com.example.github-token")

do {
    try keychain
        .accessibility(.whenUnlocked)
        .set("01234567-89ab-cdef-0123-456789abcdef", key: "kishikawakatsumi")
} catch let error {
    print("error: \(error)")
}
```

#### :couple: 共享钥匙串项目

```swift
let keychain = Keychain(service: "com.example.github-token", accessGroup: "12ABCD3E4F.shared")
```

#### <a name="icloud_sharing"> :arrows_counterclockwise: 将钥匙串项目与 iCloud 同步

###### 创建实例

```swift
let keychain = Keychain(service: "com.example.github-token")
    .synchronizable(true)

keychain["kishikawakatsumi"] = "01234567-89ab-cdef-0123-456789abcdef"
```

###### 一次

```swift
let keychain = Keychain(service: "com.example.github-token")

do {
    try keychain
        .synchronizable(true)
        .set("01234567-89ab-cdef-0123-456789abcdef", key: "kishikawakatsumi")
} catch let error {
    print("error: \(error)")
}
```

### <a name="touch_id_integration"> :cyclone: 触控 ID（面容 ID）集成

**任何需要身份验证的操作都必须在后台线程中运行。.**  
**如果在主线程中运行，UI 线程将锁定系统以尝试显示身份验证对话框。**


**要使用面容 ID，请将NSFaceIDUsageDescription密钥添加到您的Info.plist**

#### :closed_lock_with_key: 添加 Touch ID（Face ID）保护项目
如果要存储受 Touch ID 保护的钥匙串项目，请指定`accessibility`和 `authenticationPolicy` 属性。

```swift
let keychain = Keychain(service: "com.example.github-token")

DispatchQueue.global().async {
    do {
        // Should be the secret invalidated when passcode is removed? If not then use `.WhenUnlocked`
        try keychain
            .accessibility(.whenPasscodeSetThisDeviceOnly, authenticationPolicy: [.biometryAny])
            .set("01234567-89ab-cdef-0123-456789abcdef", key: "kishikawakatsumi")
    } catch let error {
        // Error handling if needed...
    }
}
```

#### :closed_lock_with_key: 更新受 Touch ID（Face ID）保护的项目

添加时的方法相同。

**添加时的方法相同**
** 如果您尝试添加的项目可能已经存在并受到保护，请不要在主线程中运行。 因为更新受保护的项目需要身份验证。.**

此外，您希望在更新时显示自定义身份验证提示消息，请指定一个`authenticationPrompt`属性。
如果该项目不受保护，则该`authenticationPrompt`参数将被忽略。



```swift
let keychain = Keychain(service: "com.example.github-token")

DispatchQueue.global().async {
    do {
        // Should be the secret invalidated when passcode is removed? If not then use `.WhenUnlocked`
        try keychain
            .accessibility(.whenPasscodeSetThisDeviceOnly, authenticationPolicy: [.biometryAny])
            .authenticationPrompt("Authenticate to update your access token")
            .set("01234567-89ab-cdef-0123-456789abcdef", key: "kishikawakatsumi")
    } catch let error {
        // Error handling if needed...
    }
}
```

#### :closed_lock_with_key: 获取 Touch ID（Face ID）保护的物品

与获得普通物品时的方式相同。如果您尝试获取的项目受到保护，它将自动显示 Touch ID 或密码验证。
如果要显示自定义身份验证提示消息，请指定一个`authenticationPrompt`属性。如果该项目不受保护，则该`authenticationPrompt`参数将被忽略。

```swift
let keychain = Keychain(service: "com.example.github-token")

DispatchQueue.global().async {
    do {
        let password = try keychain
            .authenticationPrompt("Authenticate to login to server")
            .get("kishikawakatsumi")

        print("password: \(password)")
    } catch let error {
        // Error handling if needed...
    }
}
```

#### :closed_lock_with_key: 移除受 Touch ID（Face ID）保护的项目

与删除普通项目时的方式相同。、
删除钥匙串项时无法显示 Touch ID 或密码身份验证。

```swift
let keychain = Keychain(service: "com.example.github-token")

do {
    try keychain.remove("kishikawakatsumi")
} catch let error {
    // Error handling if needed...
}
```

### <a name="shared_web_credentials"> :key: 共享网络凭证

> 共享 Web 凭据是一个编程接口，它使本机 iOS 应用程序能够与其网站对应项共享凭据。例如，用户可以在 Safari 中登录网站，输入用户名和密码，然后使用 iCloud Keychain 保存这些凭据。稍后，用户可能会运行来自同一开发人员的本机应用程序，而不是应用程序要求用户重新输入用户名和密码，共享 Web 凭据允许它访问之前在 Safari 中输入的凭据。用户还可以在应用程序中创建新帐户、更新密码或删除她的帐户。这些更改随后会被保存并由 Safari 使用。
<https://developer.apple.com/library/ios/documentation/Security/Reference/SharedWebCredentialsRef/>


```swift
let keychain = Keychain(server: "https://www.kishikawakatsumi.com", protocolType: .HTTPS)

let username = "kishikawakatsumi@mac.com"

// First, check the credential in the app's Keychain
if let password = try? keychain.get(username) {
    // If found password in the Keychain,
    // then log into the server
} else {
    // If not found password in the Keychain,
    // try to read from Shared Web Credentials
    keychain.getSharedPassword(username) { (password, error) -> () in
        if password != nil {
            // If found password in the Shared Web Credentials,
            // then log into the server
            // and save the password to the Keychain

            keychain[username] = password
        } else {
            // If not found password either in the Keychain also Shared Web Credentials,
            // prompt for username and password

            // Log into server

            // If the login is successful,
            // save the credentials to both the Keychain and the Shared Web Credentials.

            keychain[username] = inputPassword
            keychain.setSharedPassword(inputPassword, account: username)
        }
    }
}
```

#### 请求所有关联域的凭据

```swift
Keychain.requestSharedWebCredential { (credentials, error) -> () in

}
```

#### 生成强随机密码

生成与 Safari 自动填充 (xxx-xxx-xxx-xxx) 使用的格式相同的强随机密码。
    
```swift
let password = Keychain.generatePassword() // => Nhu-GKm-s3n-pMx
```

#### 如何设置共享 Web 凭据

> 1.将 com.apple.developer.associated-domains 权利添加到您的应用程序。此权利必须包括您要与之共享凭据的所有域。
>
> 2. 将 apple-app-site-association 文件添加到您的网站。此文件必须包含站点要与之共享凭据的所有应用程序的应用程序标识符，并且必须正确签名。

> 3. 安装该应用程序后，系统会下载并验证其每个关联域的站点关联文件。如果验证成功，则该应用程序与该域相关联。

**更多详细信息:**  
<https://developer.apple.com/library/ios/documentation/Security/Reference/SharedWebCredentialsRef/>

### :mag: 调试

#### 如果打印钥匙串对象，则显示所有存储的项目

```swift
let keychain = Keychain(server: "https://github.com", protocolType: .https)
print("\(keychain)")
```

```
=>
[
  [authenticationType: default, key: kishikawakatsumi, server: github.com, class: internetPassword, protocol: https]
  [authenticationType: default, key: hirohamada, server: github.com, class: internetPassword, protocol: https]
  [authenticationType: default, key: honeylemon, server: github.com, class: internetPassword, protocol: https]
]
```

#### 获取所有存储的密钥

```swift
let keychain = Keychain(server: "https://github.com", protocolType: .https)

let keys = keychain.allKeys()
for key in keys {
  print("key: \(key)")
}
```

```
=>
key: kishikawakatsumi
key: hirohamada
key: honeylemon
```

#### 获取所有存储的item

```swift
let keychain = Keychain(server: "https://github.com", protocolType: .https)

let items = keychain.allItems()
for item in items {
  print("item: \(item)")
}
```

```
=>
item: [authenticationType: Default, key: kishikawakatsumi, server: github.com, class: InternetPassword, protocol: https]
item: [authenticationType: Default, key: hirohamada, server: github.com, class: InternetPassword, protocol: https]
item: [authenticationType: Default, key: honeylemon, server: github.com, class: InternetPassword, protocol: https]
```

## 钥匙串共享功能

如果您遇到以下错误，您需要添加一个`Keychain.entitlements`.

```
OSStatus error:[-34018] Internal error when a required entitlement isn't present, client has neither application-identifier nor keychain-access-groups entitlements.
```

<img alt="Screen Shot 2019-10-27 at 8 08 50" src="https://user-images.githubusercontent.com/40610/67627108-1a7f2f80-f891-11e9-97bc-7f7313cb63d1.png" width="500">

<img src="https://user-images.githubusercontent.com/40610/67627072-333b1580-f890-11e9-9feb-bf507abc2724.png" width="500" />

## Requirements

|            | OS                                                         | Swift              |
|------------|------------------------------------------------------------|--------------------|
| **v1.1.x** | iOS 7+, macOS 10.9+                                        | 1.1                |
| **v1.2.x** | iOS 7+, macOS 10.9+                                        | 1.2                |
| **v2.0.x** | iOS 7+, macOS 10.9+, watchOS 2+                            | 2.0                |
| **v2.1.x** | iOS 7+, macOS 10.9+, watchOS 2+                            | 2.0                |
| **v2.2.x** | iOS 8+, macOS 10.9+, watchOS 2+, tvOS 9+                   | 2.0, 2.1           |
| **v2.3.x** | iOS 8+, macOS 10.9+, watchOS 2+, tvOS 9+                   | 2.0, 2.1, 2.2      |
| **v2.4.x** | iOS 8+, macOS 10.9+, watchOS 2+, tvOS 9+                   | 2.2, 2.3           |
| **v3.0.x** | iOS 8+, macOS 10.9+, watchOS 2+, tvOS 9+                   | 3.x                |
| **v3.1.x** | iOS 8+, macOS 10.9+, watchOS 2+, tvOS 9+                   | 4.0, 4.1, 4.2      |
| **v3.2.x** | iOS 8+, macOS 10.9+, watchOS 2+, tvOS 9+                   | 4.0, 4.1, 4.2, 5.0 |
| **v4.0.x** | iOS 8+, macOS 10.9+, watchOS 2+, tvOS 9+                   | 4.0, 4.1, 4.2, 5.1 |
| **v4.1.x** | iOS 8+, macOS 10.9+, watchOS 3+, tvOS 9+, Mac Catalyst 13+ | 4.0, 4.1, 4.2, 5.1 |

## 安装

### CocoaPods

KeychainAccess is available through [CocoaPods](http://cocoapods.org). To install
it, simply add the following lines to your Podfile:

```ruby
use_frameworks!
pod 'KeychainAccess'
```

### Carthage
KeychainAccess 可通过[Carthage](https://github.com/Carthage/Carthage)获得。
要安装它，只需将以下行添加到您的 Cartfile 中

`github "kishikawakatsumi/KeychainAccess"`

### Swift Package Manager
KeychainAccess 也可以通过[Swift Package Manager](https://github.com/apple/swift-package-manager/)获得。

#### Xcode

Select `File > Add Packages... > Add Package Dependency...`,  

<img src="https://user-images.githubusercontent.com/40610/67627000-2833b580-f88f-11e9-89ef-18819b1a6c67.png" width="800px" />

#### 命令行界面
首先，创建`Package.swift`它的包声明包括：

```swift
// swift-tools-version:5.0
import PackageDescription

let package = Package(
    name: "MyLibrary",
    products: [
        .library(name: "MyLibrary", targets: ["MyLibrary"]),
    ],
    dependencies: [
        .package(url: "https://github.com/kishikawakatsumi/KeychainAccess.git", from: "3.0.0"),
    ],
    targets: [
        .target(name: "MyLibrary", dependencies: ["KeychainAccess"]),
    ]
)
```

然后，输入

```shell
$ swift build
```

### 手动添加到您的项目

1. 添加 `Lib/KeychainAccess.xcodeproj`到您的项目
2. 链接`KeychainAccess.framework`到你的目标
3. 添加 `Copy Files Build Phase`以将框架包含到您的应用程序包中

请参阅[iOS 示例项目](https://github.com/kishikawakatsumi/KeychainAccess/tree/master/Examples/Example-iOS)作为参考。

<img src="https://raw.githubusercontent.com/kishikawakatsumi/KeychainAccess/master/Screenshots/Installation.png" width="800px" />

## Author

kishikawa katsumi, kishikawakatsumi@mac.com

## License

KeychainAccess is available under the MIT license. See the LICENSE file for more info.
