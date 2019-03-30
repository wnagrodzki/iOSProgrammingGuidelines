# iOSProgrammingGuidelines

## Swift

### Type vs instance methods

If a method is independent of instance state it should be declared as type method, preferably by using `static` keyword (or `class` in case when the method is designed to be overridden in a subclass).
> This is to avoid methods giving a false impression that they are dependent on the instance state while in fact they are not.

### Controlling subclassing behavior

Subclassing behavior should be designed with maximal hermetization principle in mind - choosing most restrictive access level possible.

Declare class `final` unless you explicitly design it to be inheritable. For non final class declare all instance members as `final` and type members as `static` except for those you explicitly design for overriding. 

When designing a module prefer `public` for non final class unless you explicitly design it to be inheritable by external clients. For `open` class prefer `public` for members except for those you explicitly design for overriding by external clients.

> Every point that can be overridden increases complexity, thus it should always be a conscious choice to add it.

### Assertions, preconditions and fatal errors

Use `precondition(_:_:file:line:)` for internal sanity checks by default. If you do not want to impact performance of shipping code apply `assert(_:_:file:line:)` instead.

> This is to avoid internal sanity checks being removed in optimized builds. Using preconditions to enforce valid data and state causes your application to terminate more predictably if an invalid state occurs, and helps make the problem easier to debug. Stopping execution as soon as an invalid state is detected also helps limit the damage caused by that invalid state.

Utilize `fatalError(_:file:line:)` in places where application's state and logic are not evaluated at all, for example: unimplemented method stubs.

### Avoid force unwrapping

Use `guard` and `preconditionFailure(_:file:line:)` instead of force unwrapping to clarify your intent.

```swift
guard let value = optionalValue else { preconditionFailure("reason why value must be present") }
```

> This is to avoid situations in which reason for force unwrapping is undocumented.

### Place noop comment in functions left empty by intention

```swift
func didDeliver(parcel: Parcel) {
    // noop
}
```
> This is to make clear function is empty by intention and not by mistake

### Declarations

Do not declare type if compiler is able to infer it.

```swift
let string = "a string"
let double = 1.0
let int = 1
let array = [1, 2, 3]
let dictionary = [1: "one", 2: "two"]
```

> This is to minimize clutter.

Keep type on the right hand side of assignment operator (this rule does not apply for closures, declare the type wherever it helps readability).

```swift
let height = CGFloat(44)
let duration = NSTimeInterval(0.25)
let view = UIView()
var array = [Int]()
var dictionary = [Int: String]()

let accumulator: (Int, Int) -> Int = { $0 + $1 }
```

> Since not all the types can be declared by a literal and type on the left hand side of assignment operator (e.g. `let height: CGFloat = 20`), let's keep all the types on the right side.

Declare constants within appropriate type.

```swift
class PhysicalObject {
    /// N * m^2 * kg^-2
    static let gravitationalConstant = 6.667e-11
}
```

> This is to increase discoverability and avoid global namespace pollution.

Gather IBOutlets in one group above all the other properties and keep them private.

```swift
@IBOutlet private weak var view: UIView!
```

> Outlets are implementation detail and should be kept private. Gathering them in one grup above all the other properties is old convention.

### Avoid specifying access control modifiers on extension level

Access control modifiers should not be specified on extension level:

```swift
private extension MyType {
    func method() {
        // code using helper()
    }
    func helper() { ... }
}
```

But on member level:

```swift
extension MyType {
    fileprivate func method() {
        // code using helper()
    }
    private func helper() { ... }
}
```

> This is to introduce distinction between members that should be visible from different types in the same file and those which should not. (`private extension` has the same result as `fileprivate extension` where all members end up being `fileprivate`).

### Avoid double negation

Don't not avoid double negation.
```swift
if !unavailable {
    //...
}
```
> This is to avoid confusion.

Use "positive" names for variables and properties.

```swift
if available {
    //...
}
```
```swift
return available ? operationWhenAvailable() : nil
```
> This is to decrease mental tax often caused by "negative" properties. It will also help keeping "main" application flow on the left hand side of colon when conditional operator `?:` is used.

Do not use negative conditions with guard.
```swift
guard !array.isEmpty else { return }
```

> This is to restrain overusing of `guard` statement where `if` would suffice. 
> `if array.isEmpty { return }`

### Golden (or Happy) Path

Nesting `if` statements should be avoided.

```swift
func buy(soda id: SodaID, with money: [Coin]) throws -> (Soda, [Coin]) {
    if let soda = soda(for: id) {
        if value(of: money) >= soda.price {
            if let change = change(from: money, minus: soda.price) {
                return (soda, change)
            }
            else {
                throw PurchaseError.noChange
            }
        }
        else {
            throw PurchaseError.insufficientFunds
        }
    }
    else {
        throw PurchaseError.outOfStock
    }
}
```

All error conditions should be handled at the beginning of the function leaving "the working code" on the first indentation level.

```swift
func buy(soda id: SodaID, with money: [Coin]) throws -> (Soda, [Coin]) {
    guard let soda = soda(for: id) else {
        throw PurchaseError.outOfStock
    }
    
    if value(of: money) < soda.price {
        throw PurchaseError.insufficientFunds
    }
    
    guard let change = change(from: money, minus: soda.price) else {
        throw PurchaseError.noChange
    }
    
    return (soda, change)
}
```
> It is easier to understand error conditions written in linear manner than as a nested structure.

### Use structured data

It is common to misuse primitive type like in the function below. Any string can be passed as the  argument. It is too general provided that application expects email to have certain format.

```swift
func showUser(withEmail: String) { }
```

Define a new type to make the API more strict, provide validation and convenience methods/properties if needed.

```swift
struct Email: RawRepresentable {
    
    let username: String
    let host: String
    
    var rawValue: String {
        return username + "@" + host
    }
    
    init?(rawValue: String) {
        // parsing and validation code
    }
}

func showUser(with: Email) { }
```
> This is to increase source code reliability by building it on data you can trust.

### Extending object's lifetime

If you are breaking reference cycle for an object bind it to a strong reference for the duration of the closure’s execution.

```swift
operation.onComplete { [weak self] result in
  guard let self = self else { return }
  let model = self.updateModel(result)
  self.updateUI(model)
}
```

> This is to avoid deallocation of weakly referenced object during closure execution.
> Use `strongSelf` instead of `self` for local variable name until [SR-6156]( https://bugs.swift.org/browse/SR-6156) is fixed.

### Accessing Singleton

Do not access singleton instance directly via static method - `Singleton.sharedInstance()`. Decouple from it with dependency injection.

> If a singleton is used directly there is no way to perform unit tests in isolation from it. Moreover, it may introduce unexpected shared state between unit tests if they are executed in one run.
