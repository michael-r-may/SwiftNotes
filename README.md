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

#### Strong vs Weak Outlets

Scott Berrevoets proposes three rules for deciding what decorator to choose for your interface builder outlets:

- ! needs a guarantee that the view exists, so always use strong to provide that guarantee
- If it's possible the view isn't part of the view hierarchy, use ? and appropriate optional-handling (optional binding/chaining) for safety. 
- If you don't need a view anymore after removing it from the view hierarchy, use weak so it gets removed from memory.

Original Article: [scottberrevoets.com](http://scottberrevoets.com/2016/03/21/outlets-strong-or-weak/)

#### Tuples

Tuples consist of zero or more types (typically String, Integer, Character and Bool, as well as other tuples). Tuples are also passed by value (copied), not by reference. You create them in this kind of a way

```
let person = (42, false, “Michael”)
let person: (Int, Bool, String) = (42, false, “Michael”)
let person = (age: 42, isTall: false, name: “Michael”)
```

You access the individual elements by either dot index format or via their name (if named).

```
print(person.age)
print(person.0)
```

If you plan to use the tuple in multiple (hah!) places, a typealias is your friend. This is especially useful for returning tuples from functions.

```
typealias Person = (age: Int, isTall: Bool, name: String)

func createUser() -> Person {
      ...
}
```

Did you know that “Void” is just a typealias for “()”, or the empty tuple. 

```
func doNothingA() -> Void { }
```

Does not return nothing, as you think, but instead returns a tuple with zero elements.

Original Article: [medium.com](https://medium.com/swift-programming/swift-tuple-328aecff50e7)

#### Always Prefix Extensions on Objective-C classes
>Swift extensions on Objective-C classes still need to be prefixed. You can use @objc(prefix_name) to keep the name pretty  in Swift and expose a prefixed version for the ObjC runtime."
[pspdfkit.com](https://pspdfkit.com/blog/2016/surprises-with-swift-extensions/)
