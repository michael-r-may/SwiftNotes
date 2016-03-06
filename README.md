# SwiftNotes
A collection of notes, tips and tricks that I want to remember when working with Ruby

####Assertions and Preconditions
There is a great blog post by [Mike Ash](https://www.mikeash.com/pyblog/friday-qa-2016-03-04-swift-asserts.html) going into detail.

Use `assert` when you are happy to have the assert removed from optimised builds, and use `precondition` when you want it to remain (you can use unchecked builds to get rid of them too, but I don't know why you would do that).

```Swift
assert(x >= 0, "x can't be negative here")
precondition(x >= 0, "x can't be negative here")
```

You can use the failure style versions to remove he precondition and always have them fail.

```Swift
assertFail("Something went very wrong")
preconditionFail("Some else went very wrong")
```

Note that there are also two more parameters to both these methods that have default arguments - file and line. 

```Swift
public func assert(
      @autoclosure condition: () -> Bool,
      @autoclosure _ message: () -> String = String(),
      file: StaticString = #file, line: UInt = #line
    )
```

####Fatal Errors
Use `fatalError` to signal a failure and halt the program. This can never be compiled out.

####Autoclosure
Directly from Mike Ash's article (see above):

> The @autoclosure argument can be applied to an argument of function type which takes no parameters. At the call site, the caller provides an expression for that argument. This expression is then implicitly wrapped in a function, and that function is passed in as the parameter.

This is useful for situations where we do not wish to evaluate an argument unless we have to. For example (again from Mike Ash) without the autoclosure the b argument is going to always be evaluated to its Bool result. This is inefficient and  potentially crashy (in this example) because it does not conform to expected behaviour of the and binary operator (lazy evaluation).

```Swift
func &&(a: Bool, b: Bool) -> Bool {
        if a {
            if b {
                return true
            }
        }
        return false
    }
}

func &&(a: Bool, @autoclosure b: () -> Bool) -> Bool {
        if a {
            if b() {
                return true
            }
        }
        return false
    }
}
```
