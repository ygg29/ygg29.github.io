---
title: 【译文】swift中的泛型编程
date: 2018-06-23 12:40:38
category: 
	- 文章翻译
tags: 
	- 泛型
	- swift
---

- [swift](https://segmentfault.com/t/swift/blogs)
- [原文（Generics in Swift）](http://austinzheng.com/2015/09/29/swift-generics-pt-2/)

本文中，笔者旨在对于Swift中的泛型编程进行一个综合性描述。读者可以查看[上一篇](http://austinzheng.com/2015/01/02/swift-generics-pt-1/) 系列中的描述来看之前笔者的论文。泛型编程是编程方式的一种，主要是一些譬如类、结构体以及枚举这样的复杂类型以及函数可以使用类型参数进行定义(type parameters)。类型参数可以看做是真实类型的占位符，当泛型或者函数被使用时才会被真实类型替换。

<!-- more -->

在Swift中，对于泛型编程最直观的表现当属Array这个类型。在Objective-C中，Array的示例NSArray可以包含任意类型的对象。但是，Swift中的Array类型需要在声明时即指定它们包含的元素类型，，譬如Array<Int>, Array<UIView>。Array与Int都是一种数据类型，泛型机制允许这两种类型有机组合起来从而能够传递更多的额外的信息。

备注：在Swift中惯用的声明Array包含了Foo类型的用法是`[Foo]`这种语法，而本文中使用`Array<Foo>`写法旨在帮助理解以及强调Array是一个通用泛型。

### Why generics (为什么使用泛型)？

这里列举出几个静态类型语言中使用泛型编程的原因：

- 类型安全：类似于Array这样的容器类型，可以给出它们存放的具体的元素的类型，从而告知编译器可以插入容器中的对象的类型以及从容器中返回的对象的类型。这种机制作用于所有可以被当做Array参数的类型，并且也作用于类型之间的关系。
- 减少冗余代码：有些情况下需要对多个数据类型进行相同的操作，可以用一个泛型方程来代替多个不同类型参数或者返回值的重复的方程。这样可以避免潜在的代码错误等等。
- 灵活的依赖库：第三方库可以暴露一些接口，从而避免使用这些接口的开发者被强制使用或者类型或者返回值为一个固定的类型。它们可以使用泛型进行更加抽象地编程，举例来说，泛型会允许一个接口来接受不仅仅是一个Array参数，而是一个Collection参数。

## Generic entities (泛型实体)

Swift中的泛型编程主要表现在以下两个不同的情况下：当定义某个类型或者定义某个函数时。泛型类型的特性以及`<`与`>`这两个关键字往往意味着某个类型或者函数是泛型。

### Generic types (泛型类型)

Swift中主要的三个用户可自定义的类型可以被当做泛型，下面就以Result枚举类型为例，该类型中存放了表征成功的Success以及表征失败的Failure：

```swift
enum Result<T, U> {
  case Success(T)
  case Failure(U)
}
```

在Result类型之后有两个类型参数： T以及U。这些类型参数将会在创建实例时被替换为真实的类型：

```swift
let aSuccess : Result<Int, String> = .Success(123)
let aFailure : Result<Int, String> = .Failure("temperature too high")
```

泛型类型的类型参数可以被用于以下的几个方面：

- 作为属性的类型
- 作为枚举体中的关联类型
- 作为某个方法的返回值或者参数
- 作为构造器的参数类型
- 作为另一个泛型的类型参数，譬如`Array`

泛型可以在以下两种方式中被初始化：

- 在创建新的实例时，类型参数被用户给出的真实的数据类型替换。
- 类型被推导得出，通过调用初始化器或者静态方法来创建某个实例。

```swift
struct Queue<T> { /* ... */ }

// First way to instantiate
let a1 : Queue<Int> = Queue()
let a2 : Queue<Int> = Queue.staticFactoryMethod() // or just .staticFactoryMethod()

// Second way to instantiate
let b1 = Queue<Int>()
let b2 = Queue<Int>.staticFactoryMethod()
```

注意，像T这样的类型参数在类型定义时出现，无论这个类型何时被调用，这些类型参数都会被替换为真实的类型。举例而言，`Array<String>`或者`Array<UIWindow>`，便是以固定类型初始化的数组。而如果是像`Array<T>`这样的，它便一直处于由其他类型或者函数作为类型参数定义的上下文中。

### Generic functions (泛型函数)

函数、方法、属性、下标以及初始化器都可以当做泛型进行处理，它们自身可以成为泛型或者存在于某个泛型类型的上下文中：

```swift
// Given an item, return an array with that item repeated n times
func duplicate<T>(item: T, numberOfTimes n: Int) -> [T] {
  var buffer : [T] = []
  for _ in 0 ..< n {
    buffer.append(item)
  }
  return buffer
}
```

同样的，类型参数是定义在`<`与`>`紧跟在函数名之后。它主要可以用于以下几个方面：

- 作为某个函数的参数
- 作为函数的返回值
- 作为另一个泛型类型的类型参数，譬如`T`可以作为`Array<T?>`中的一部分

当然，如果所有的类型参数都是未用状态编译器会不太友好。而泛型方法可以同时定义在泛型类型与非泛型类型上：

```swift
extension Result {
  // Transform the value of the 'Result' using one of two mapping functions
  func transform<V>(left: T -> V, right: U -> V) -> V {
    switch self {
    case .Success(let value): return left(value)
    case .Failure(let value): return right(value)
    }
  }
}
```

上述中的`transform`方法就是泛型方法，存放于泛型类型`Result`中。除了由Result中定义的类型参数T、U，泛型方法本身也定义了一个泛型参数V。当调用一个泛型方程时，不一定需要指明清楚传入的类型参数。编译器的类型推导接口会自动根据参数与返回类型推导出相关信息：

```swift
// Example use of the 'duplicate:numberOfTimes:' function defined earlier.
// T is inferred to be Int.
let a = duplicate(52, numberOfTimes: 10)
```

实际上，尝试去清楚地设置参数类型还会引发错误：

```swift
// Does not compile
// Error: "Cannot explicitly specialize a generic function"
let b = duplicate<String>("foo", numberOfTimes: 5)
```

## Associated types (关联类型)

Swift中的Protocol不可以使用类型参数定义泛型，不过Protocol可以使用`typealias`关键字来定义一些关联类型：

```swift
// A protocol for things that can accept food.
protocol FoodEatingType {
  typealias Food

  var isSatiated : Bool { get }
  func feed(food: Food)
}
```

在这个实例中，Food是定义在FoodEatingType协议中的关联类型，而某个协议中的关联类型数目可以根据实际的需求数量定义。

关联类型，类似于类型参数，都是一种占位符。而后面如果某个类需要实现这个协议，则需要确定`Food`的具体类型，是Hay还是Rabbit。具体的做法就是继承并且实现协议的属性与方法，并在实现时再判断应该用哪些真实的数据类型去替换这些关联类型：

```swift
class Koala : FoodEatingType {
  var foodLevel = 0
  var isSatiated : Bool { return foodLevel < 10 }

  // Koalas are notoriously picky eaters
  func feed(food: Eucalyptus) {
    // ...
    if !isSatiated {
      foodLevel += 1
    }
  }
}
```

对于`Koala`这个具体的实现者，关联类型Food被定义为了`Eucalyptus`，换言之，也就是`Koala.Food`被定义为了`Eucalyptus`。如果某个类型实现了多个协议，那么对于每个协议中的关联类型也都必须实现。而如果某个继承的类型也是个泛型，也可以使用类型参数去帮助确定关联类型：

```swift
// Gourmand Wolf is a picky eater and will only eat his or her favorite food.
// Individual wolves may prefer different foods, though.
class GourmandWolf<FoodType> : FoodEatingType {
  var isSatiated : Bool { return false }

  func feed(food: FoodType) {
    // ...
  }
}

let meela = GourmandWolf<Rabbit>()
let rabbit = Rabbit()
meela.feed(rabbit)
```

在上述代码中, `GourmandWolf<Goose>.Food` 即 `Goose`, 而 `GourmandWolf<Sheep>.Food` 即 `Sheep`。顺便说一句，协议中的关联类型虽然属于泛型，但也可以为其添加一些约束或者父类。譬如我们定义的某个协议中为Heap添加了一些列的操作，并且要保证所有的Heap的Key是可比较的，即必须是Comparable的子类或者实现，那么可以添加如下约束：

```swift
// Types that conform represent simple max-heaps which use their elements as keys
protocol MaxHeapType {
  // Elements must support comparison ops; e.g. 'a is greater than b'.
  typealias Element : Comparable

  func insert(x: Element)
  func findMax() -> Element?
  func deleteMax() -> Bool
}
```

## Type constraints (类型约束)

截至目前，我们提及的泛型，诸如T以及U可以用任意类型来替换。而标准库中的Array类型即是这样一种无任何约束的典型，可以由其类型参数来决断。譬如下面一个例子中，需要编写一个函数，输入一个数组而获取数组中最大的那个值并且返回：

```swift
// Doesn't compile.
func findLargestInArray<T>(array: Array<T>) -> T? {
  if array.isEmpty { return nil }
  var soFar : T = array[0]
  for i in 1..<array.count {
    soFar = array[i] > soFar ? array[i] : soFar
  }
  return soFar
}
```

不过这样毫无限制的泛型对于编译器是非常不友好的，如上述代码中需要进行一个比较，即`array[i] > soFar`，而我们只知道`array[i]`是类型T，并且`soFar`也是类型T。但是编译器根本不知道这个类型T能否进行比较。譬如我们创建一个空的结构体`Foo`，并且把它当做类型参数传了进来，那么我们压根不知道`>`这个比较运算符能否起作用。

在上文对于Protocol的讨论中，我们已经尝试使用`:`为某个关联类型设置一些约束，而在刚才的例子中，如果传入的String类型是可以正常编译的，但是一旦传入的是NSView类型，则不能正常编译了。而我们可以以如下方式添加约束：

```swift
// Note that <T> became <T : Comparable>, meaning that whatever type fills in
// 'T' must conform to Comparable.
func findLargestInArray<T : Comparable>(array: Array<T>) -> T? {
  if array.isEmpty { return nil }
  var soFar : T = array[0]
  for i in 1..<array.count {
    soFar = array[i] > soFar ? array[i] : soFar
  }
  return soFar
}

// Example usage:
let arrayToTest = [1, 2, 3, 100, 12]
// We're calling 'findLargestInArray()' with T = Int.
if let result = findLargestInArray(arrayToTest) {
  print("the largest element in the array is \(result)")
} else {
  print("the array was empty...")
}
// prints: "the largest element in the array is 100"
```

这是因为，在某种意义上，存在着一种悖论，越限制类型参数与添加约束，用户越能方便地使用这些参数。不加任何限制地类型参数往往只能使用在简单地交换或者从集合中添加或者删除某些元素。

### 简单约束

```swift
func example<T, U : Equatable, V : Hashable>(foo: T, bar: U, baz: V) {
  // ...
}
```

如果需要对泛型进行细粒度的控制，首先来讨论下在哪些地方可以进行控制处理：

- 类似于U,V这样的类型参数，即上文中的普通的泛型参数
- 类型参数的关联类型，即上文中协议里的关联类型。

Swift中提供了以下三个类型的约束：

- `T : SomeProtocol`：类型T必须遵从协议`SomeProtocol`。需要注意，使用`protocol<Foo,Bar>`。
- `T==U`：类型参数T必须是类型参数或者关联类型U。
- `T:SomeClass`:T必须是一个类，更加具体而言，T必须是SomeClass的一个实例或者它的子类。

## Putting it all together

上文中已经介绍了基本的泛型的用法和约束，而对于泛型类型的签名可以综合使用如下：

- 声明类型参数，如果愿意的话，最好每个类型参数都要声明遵循某个协议。
- 使用`where`关键字。
- 使用逗号分割声明约束。

```swift
protocol Foo {
  typealias Key
  typealias Element
}

protocol Bar {
  typealias RawGeneratorType
}

func example<T : Foo, U, V where V : Foo, V : Bar, T.Key == V.RawGeneratorType, U == V.Element>
  (arg1: T, arg2: U, arg3: V) -> U {
  // ...
}
```

不要惊慌，我们会一步一步地介绍这些泛型的用法。在`where`关键字之前，我们声明了三个类型参数：T，U以及V。其中T必须遵循Foo协议。而在where关键字之后，我们声明了四个约束：

- V必须遵循Foo协议。
- V也必须实现了Bar协议。
- 由于T实现了Foo协议，T有一个关联类型为`T.Key`。V还有另一个关联类型，V.RawGeneratorType。这两个类型必须是相同的：T.Key == V.RawGeneratorType。
- 因为V实现了协议Foo，V包含一个关联类型V.Element。这个类型必须与U一致，U == V.Element。

综上所述，无论何时使用`example()`函数时，必须要选择合适的T、U以及V类型。

## Constrained extensions

Swift 2中新近提出了约束扩展的概念，一个更强大的能够使用泛型的语法特性。在Swift中，扩展允许向任意类型，即使尚未定义的类型中添加方法。同样允许向某个协议中添加默认的实现方法。同样的，这样的基于约束的扩展可以方便某些泛型的用法：

- 对于像Array这样的泛型，可以在类型参数符合某个特定的约束的情况下添加某个方法。
- 对于像CollectionType这样包含关联类型的协议，可以当某个关联类型符合某个约束时添加默认的实现方法。

### Syntax and limitations (语法与限制)

基于约束的扩展语法如下所示：

```swift
// Methods in this extension are only available to Arrays whose elements are
// both hashable and comparable.
extension Array where Element : Hashable, Element : Comparable {
  // ...
}
```

where关键字跟随在类型参数之后，而后跟着以逗号分割的一系列泛型类型和参数。不过这其中的约束中并不能限定为非泛型，即：

```swift
// An extension on Array<Int>.
// Error: "Same-type requirement makes generic parameter 'Element' non-generic"
extension Array where Element == Int {
  // ...
}
```

同时，也不能进行协议的传递：

```swift
protocol MyProtocol { /* ... */ }

// Only Arrays with comparable elements conform to MyProtocol.
// Error: "Extension of type 'Array' with constraints cannot have an inheritance clause"
extension Array : MyProtocol where Element : Comparable {
  // ...
}
```

