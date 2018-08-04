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
