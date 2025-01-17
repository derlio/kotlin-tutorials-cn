## `if`/`else`

`if`/`else` works the same way as in Python, but it's `else if` instead of `elif`, the conditions are enclosed in parentheses, and the bodies are enclosed in curly braces:

```kotlin
val age = 42
if (age < 10) {
    println("You're too young to watch this movie")
} else if (age < 13) {
    println("You can watch this movie with a parent")
} else {
    println("You can watch this movie")
}
```

The curly braces around a body can be omitted if the body is a oneliner. This is discouraged unless the body goes on the same line as the condition, because it makes it easy to make this mistake, especially when one is used to Python:

```kotlin
if (age < 10)
    println("You're too young to watch this movie")
    println("You should go home") // Mistake - this is not a part of the if body!
```

Without the curly braces, only the first line is a part of the body. Indentation in Kotlin matters only for human readers, so the second print is outside the if and will always be executed.

An if/else statement is also an expression, meaning that a ternary conditional (which looks like `result = true_body if condition else false_body` in Python) looks like this in Kotlin:

```kotlin
val result = if (condition) trueBody else falseBody
```

When using if/else as an expression, the `else` part is mandatory (but there can also be `else if` parts). If the body that ends up being evaluated contains more than one line, it's the result of the last line that becomes the result of the `if`/`else`.


## 比较

Structural equality comparisons are done with `==` and `!=`, like in Python, but it's up to each class to define what that means, by [覆盖](inheritance.html#覆盖) [`equals()`](classes.html#继承的内置函数) (which will be called on the left operand with the right operand as the parameter) and `hashCode()`. Most built-in collection types implement deep equality checks for these operators and functions. Reference comparisons - checking if two variables refer to the same object (the same as `is` in Python) - are done with `===` and `!==`.

Boolean expressions are formed with `&&` for logical AND, `||` for logical OR, and `!` for logical NOT. As in Python, `&&` and `||` are short-circuiting: they only evaluate the right-hand side if it's necessary to determine the outcome. Beware that the keywords `and` and `or` also exist, but they only perform _bitwise_ operations on integral values, and they do not short-circuit.

There are no automatic conversions to boolean and thus no concept of truthy and falsy: checks for zero, empty, or null must be done explicitly with `==` or `!=`. Most collection types have an `isEmpty()` and an `isNotEmpty()` function.


## `when`

We're not going to cover the [`when` expression](https://www.kotlincn.net/docs/reference/control-flow.html#when-表达式) in depth here since it doesn't have a close equivalent in Python, but check it out - it's pretty nifty, as it lets you compare one expression against many kinds of expressions in a very compact way (but it's not a full functional-programming-style pattern matcher). For example:

```kotlin
val x = 42
when (x) {
    0 -> println("zero")
    in 1..9 -> println("single digit")
    else -> println("multiple digits")
}
```




---

[← 上一节：字符串](strings.html) | [下一节：集合 →](collections.html)


---

*本资料英文原文的作者是 [Aasmund Eldhuset](https://eldhuset.net/)；其所有权属于[可汗学院（Khan Academy）](https://www.khanacademy.org/)，授权许可为 [CC BY-NC-SA 3.0 US（署名-非商业-相同方式共享）](https://creativecommons.org/licenses/by-nc-sa/3.0/us/)。请注意，这并不是可汗学院官方产品的一部分。中文版由[灰蓝天际](https://hltj.me/)译，遵循相同授权方式。*