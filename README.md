# Swift Notes
A collection of notes, tips and tricks that I want to remember when working with Ruby

####Assertions and Preconditions
There is a great blog post by [Mike Ash](https://www.mikeash.com/pyblog/friday-qa-2016-03-04-swift-asserts.html) going into detail as well as [another](https://www.mikeash.com/pyblog/friday-qa-2013-05-03-proper-use-of-asserts.html) about why you should use them, and how, and that they should never be compiled out.

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
      file: StaticString = #file, 
      line: UInt = #line
    )
```

####Fatal Errors
Use `fatalError` to signal a failure and halt the program. This can never be compiled out.

All of the arguments to the assert and fatalError calls are autoclosured.

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


####Lazy
Use lazy to delay computation until it's needed. 

For example,

```
class Avatar {
  static let defaultSmallSize = CGSize(width: 64, height: 64)

  lazy var smallImage: UIImage = self.largeImage.resizedTo(Avatar.defaultSmallSize)
  var largeImage: UIImage

  init(largeImage: UIImage) {
    self.largeImage = largeImage
  }
}
```

#####Lazy Closures
It works with closures

```
lazy var smallImage: UIImage = {
    let size = CGSize(
      width: min(Avatar.defaultSmallSize.width, self.largeImage.size.width),
      height: min(Avatar.defaultSmallSize.height, self.largeImage.size.height)
    )
    return self.largeImage.resizedTo(size)
  }()
```

And you can even reference self safely because the closure can never be executed until self exists.

You can’t create lazy let instance properties in Swift to provide constants that would only be computed if accessed. Let constants declared at global scope or declared as a type property (using static let, not as instance properties) are automatically lazy (and thread-safe.

#####Lazy Sequences

You can apply delayed computation to sequences too, which can be a powerful saving when they are computationally intense calculated sequences.

```
func increment(x: Int) -> Int {
  print("Computing next value of \(x)")
  return x+1
}

let array = Array(0..<1000)
let incArray = array.lazy.map(increment)
print("Result:")
print(incArray[0], incArray[4])
```

Original source for lazy information. http://alisoftware.github.io/swift/2016/02/28/being-lazy
