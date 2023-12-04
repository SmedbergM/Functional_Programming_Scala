## Tail Recursion 

One common critique of using recursion is that it is inherently slow. 
Why is that?
The clue is in what is known as the call stack. 
Every time a function is called, some information about the call is put on the call stack -- we say some stack space is 
allocated. 
This information is kept there until all the computations within the function have finished, after which the stack is 
deallocated (the information about the function call is removed from the stack) and the computed value is returned. 
If a function calls another function, then the stack is allocated again, before deallocating the previous function's call. What is worse, we wait until the inner call is done, its stack frame is deallocated and its value returned to compute 
the result of the caller function.
It is especially significant for recursive functions, because the depth of call stack can be astronomical. 

Consider the example of computing the factorial.

```scala 3
def factorial(n: BigInt): BigInt =
  if (n <= 0) 1
  else n * factorial(n - 1)
```

Evaluating `factorial(3)` leads to the following events happening on the stack: 

```scala 3
factorial(3)
3 * factorial(2)
3 * 2 * factorial(1)
3 * 2 * 1 * factorial(0)
3 * 2 * 1 * 1 
3 * 2 * 1 
3 * 2 
6
```

Until we've reached the base case, we cannot start doing multiplication, and have to keep all the multipliers on the stack.
Calling `factorial` with big enough argument (`10000` on my computer) results in the stack overflow error, and no 
result is computed at all. 

Do not get discouraged! 
There is a well-known optimisation technique capable of mitigating this issue. 
It involves rewriting your recursive function into a tail-recursive form. 
In it, the recursive call should be the last thing the function does. 
For example, `factorial` can be rewritten like this: 

```scala 3
def factorial(n: BigInt): BigInt = 
  def go(n: BigInt, accumulator: BigInt): BigInt =
    if (n <= 0) accumulator
    else go(n - 1, n * accumulator)
  go(n, 1)
```

We add a new parameter `accumulator` to the recursive function where we keep track of the partially computed 
multiplication.
Notice, that the recursive call to `go` is the last thing which happens in the `else` branch of the `if` condition. 
Whatever value the recursive call returns, it's just returned by the caller. 
There is no reason to allocate any stack frames, because nothing is awaiting the result of the recursive call to enable
further computation. 
Smart enough compilers (and the compiler of Scala is one of them) is capable to optimize away the unnecessary stack 
allocations in this case. 
Go ahead and try to find `n` such that the tail recursive `factorial` results in the stack overflow. 
Unless something goes horribly wrong, you should not be able to find such `n`. 

By the way, do you remember the key searching function we implemented in the previous task?
Have you wondered how we got away not keeping track of a collection of boxes to look through?
The trick is that the stack replaces that collection. 
All the boxes to be considered are somewhere on the stack patiently waiting for their turn. 

Is there a way we can make that function tail recursive? 
Yes, of course, there is! 
Similarly to the `factorial` function, we can create a helper function `go` with an extra parameter `boxesToLookIn`
to keep track of the boxes to search the key in.
This way we can ensure that `go` is tail recursive, i.e. either returns a value or calls itself as its final step. 

```scala 3
def lookForKey(box: Box): Option[Key] =
  def go(item: Item, boxesToLookIn: Set[Item]): Option[Key] =
    item match
      case key: Key => Some(key)
      case Box(content) =>
        if content.isEmpty
        then
          if boxesToLookIn.isEmpty
          then None
          else go(boxesToLookIn.head, boxesToLookIn.tail)
        else
          go(content.head, boxesToLookIn ++ content.tail)
  go(box, Set.empty)
```

There is a way in Scala to ensure that your function is tail-recursive: `@tailrec` annotation from `scala.annotation.tailrec`. 
It checks if your implementation is tail-recursive, and triggers a compiler error if it is not. 
We recommend using this annotation to make sure that the compiler is capable to optimize your code even through its
future changes.

### Exercise 

Implement tail recursive functions for reversing a list and finding the sum of digits of a non-negative number. 
We annotated the helper functions with `@tailrec` so that the compiler could check the property for us.  