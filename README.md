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

### Avoid force unwrapping

Use `guard` and `fatalError()` instead of force unwrapping to clarify your intent.

```swift
guard let value = optionalValue else { fatalError("reason why value must be present") }
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
