# iOSProgrammingGuidelines

## Swift

### Type vs instance methods

If a method is independent of instance state it should be declared as type method, preferably by using `static` keyword (or `class` in case when the method is designed to be overridden in a subclass).
> This is to avoid methods giving a false impression that they are dependent on the instance state while in fact they are not.
