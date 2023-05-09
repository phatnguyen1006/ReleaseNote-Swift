# Swift 5.8

<details>
  <summary>Table of Contents</summary>
  <ol>
    <li>
      <a href="#lift-all-limitations-on-variables-in-result-builders">Lift all limitations on variables in result builders</a>
    </li>
    <li>
      <a href="#function-back-deployment">Function back deployment</a>
    </li>
    <li>
      <a href="#allow-implicit-self-for-weak-self-captures-after-self-is-unwrapped">Allow implicit self for weak self captures, after self is unwrapped</a>
      </li>
    <li>
      <a href="#concise-magic-file-names">Concise magic file names</a>
    </li>
    <li>
      <a href="#opening-existential-arguments-to-optional-parameters">Opening existential arguments to optional parameters</a>
    </li>
    <li>
      <a href="#collection-downcasts-in-cast-patterns-are-now-supported">Collection downcasts in cast patterns are now supported</a>
    </li>
  </ol>
</details>

## Lift all limitations on variables in result builders
> [SE-0373](https://github.com/apple/swift-evolution/blob/main/proposals/0373-vars-without-limits-in-result-builders.md) relaxes some of the restrictions on variables when used inside [result builder](https://www.hackingwithswift.com/swift/5.4/result-builders) ( available from Swift 5.4), allowing us to write code that would previously have been disallowed by the compiler.

> [SE-0373](https://github.com/apple/swift-evolution/blob/main/proposals/0373-vars-without-limits-in-result-builders.md) nới lỏng một số ràng buộc trong [result builder](https://www.hackingwithswift.com/swift/5.4/result-builders) ( available from Swift 5.4)

```swift
struct CounterView: View {
    var body: some View {
        @AppStorage("counter") var tapCount = 0
    
        Button("Count: \(tapCount)") {
            tapCount += 1
        }
    }
}

struct CounterChien {
    var a = 0
    
    func checkA(completion: @escaping () -> Void) {
        lazy var b = a
        print("this is b \(b)")
    }
    
    func B() {
        self.checkA {
            lazy var c = 0
            print("this is c \(c)")
        }
    }
}
```

## Function-back-deployment
> [SE-0376](https://github.com/apple/swift-evolution/blob/main/proposals/0376-function-back-deployment.md) adds a new `@backDeployed` attribute that makes it possible to allow new APIs to be used on older versions of frameworks. It works by writing the code for a function into your app’s binary then performing a runtime check: if your user is on a suitably new version of the operating system then the system’s own version of the function will be used, otherwise the version copied into your app’s binary will be used instead.

> [SE-0376](https://github.com/apple/swift-evolution/blob/main/proposals/0376-function-back-deployment.md) Cho phép khai báo các method chỉ available cho một số phiên bản trở về trước. Hoặc có thể dùng chung với available để tạo ra một tập hợp phiên bản từ A đến B có thể sử dụng method.

```swift
import SwiftUI

extension Text {
    @backDeployed(before: iOS 16.4, macOS 13.3, tvOS 16.4, watchOS 9.4)
    @available(iOS 14.0, macOS 11, tvOS 14.0, watchOS 7.0, *)
    public func monospaced(_ isActive: Bool) -> Text {
        fatalError("Implementation here")
    }
}
```

## Allow implicit self for weak self captures, after self is unwrapped
> [SE-0365](https://github.com/apple/swift-evolution/blob/main/proposals/0365-implicit-self-weak-capture.md) takes another step  towards letting us remove `self` from closures by allowing an implicit `self` in places where a `weak self` capture has been unwrapped.

> [SE-0365](https://github.com/apple/swift-evolution/blob/main/proposals/0365-implicit-self-weak-capture.md) Cho phép compiler tự suy self trong closure sau khi đã được unwrap.

```swift
import Foundation

class TimerController {
    var timer: Timer?
    var fireCount = 0
    
    init() {
        timer = Timer.scheduledTimer(withTimeInterval: 1, repeats: true) { [weak self] timer in
            guard let self else { return }
            print("Timer has fired \(fireCount) times")
            fireCount += 1
        }
    }
}
```

## Concise magic file names
> [SE-0274](https://github.com/apple/swift-evolution/blob/main/proposals/0274-magic-file.md) adjusts the `#file` magic identifier to use the format Module/Filename, e.g. MyApp/ContentView.swift. Previously, `#file` contained the whole path to the Swift file, e.g. `/Users/lap12345/Desktop/WhatsNewInSwift/WhatsNewInSwift/ContentView.swift`, which is unnecessarily long and also likely to contain things you’d rather not reveal.

> [SE-0274](https://github.com/apple/swift-evolution/blob/main/proposals/0274-magic-file.md) thay vì sử dụng `#filePath` sẽ trả ra đường dẫn tuyệt đối trong máy người dùng. Thì bây giờ có thể dùng `#file` để có đường dẫn với Root là module của file đó.

```swift
// New behavior, when enabled
print(#file)      // Module/Filename
    
// Old behavior, when needed
print(#filePath)  // /Users/lap12345/Desktop/WhatsNewInSwift/WhatsNewInSwift/ContentView.swift
```

## Opening existential arguments to optional parameters
> [SE-0375](https://github.com/apple/swift-evolution/blob/main/proposals/0375-opening-existential-optional.md) extends a Swift 5.7 feature that allowed us to call generic functions using a protocol, fixing a small but annoying inconsistency: Swift 5.7 would not allow this behavior with optionals, whereas Swift 5.8 does.

> [SE-0375](https://github.com/apple/swift-evolution/blob/main/proposals/0375-opening-existential-optional.md) cho phép sử dụng Generic kết hợp với optional. Trước đó chưa được hỗ trợ.

```swift
func double<T: Numeric>(_ number: T) -> T {
    number * 2
}
    
let first = 1
let second = 2.0
let third: Float = 3
    
let numbers: [any Numeric] = [first, second, third]
    
for number in numbers {
    print(double(number))
}
/*:
In Swift 5.8, that same parameter can now be optional, like this:
*/
func optionalDouble<T: Numeric>(_ number: T?) -> T {
    let numberToDouble = number ?? 0
    return  numberToDouble * 2
}
    
for number in numbers {
    print(optionalDouble(number))
}
```

## Collection downcasts in cast patterns are now supported
> This resolves another small but potentially annoying inconsistency in Swift where downcasting a collection – e.g. casting an array of `ClassA` to an array of another type that *inherits* from `ClassA` – would not be allowed in some circumstances.

> Cho phép ép kiểu cho một mảng hoặc một tập hợp. Trước Swift 5.8 sẽ gặp lỗi “Collection downcast in cast pattern is not implemented; use an explicit downcast to '[Type]' instead.”. Và phải sử dụng syntax `if let dogs = pets as? [Dog] { // jobs }`.

```swift
class Pet { }
class Dog: Pet {
    func bark() { print("Woof!") }
}
    
func bark(using pets: [Pet]) {
    switch pets {
    case let pets as [Dog]:
        for pet in pets {
            pet.bark()
        }
    default:
        print("No barking today.")
    }
}
```

