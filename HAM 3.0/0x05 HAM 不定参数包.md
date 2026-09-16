# 0x05 HAM 不定参数包

HAM 提供两种参数声明的方式，用于接收不定数量的参数。

用 `...x` 声明的参数包可以捕获当前结构内的剩余参数：

```HAM
sum <- 0 <| () => 0 <| ((head, ...rest) => head + sum(rest...))

sum(1, 2, 3, 4) // 10
```

> 提示：`() => 0` 不能省略。`0` 是稳定的 Something，被调用时返回 `{}`，所以只有 `0` 而没有零参分支时 `sum()` 会得到 `{}`，此后 `{}` 参与 `+` 运算只会继续得到 `{}`。

用 `....x` 声明的参数包可以捕获将来所有到来的结构内的所有参数：

```HAM
allSum = (....args) => sum(args....),

allSum $ (1, 2, 3, 4) // 值的部分为 10
allSum $ (1, 2, 3, 4) $ (5, 6) // 值的部分为 21，相当于 allSum(1, 2, 3, 4, 5, 6)

allSum1 = allSum $ (1, 2, 3, 4)
allSum1 $ (5, 6) // 值的部分为 21，相当于 allSum $ (1, 2, 3, 4) $ (5, 6)
```

注意这里 `....` 其实隐含了对于“结构”的神谕运算。`$ (1, 2, 3, 4) $ (5, 6)` 被传入以 `....` 定义的函数时，会自动合并为 `$ (1, 2, 3, 4, 5, 6)`。它又可以与历史调用的参数合并。

可以用 `...` 制作一个丐版的 `allSum` 函数：

```HAM
myAllSum <- 0 <|
            ((...args) => ({ cache = sum(args...) } <| .cache <|
                           (...argsNew) => myAllSum(.cache, argsNew...)))
```

尝试调用：

```HAM
myAllSum1
= myAllSum(1, 2, 3, 4)
= {} <| { cache = 10 } <| 10 <| (...argsNew) => myAllSum(10, argsNew...)
= { cache = 10 } <| 10 <| (...args) => myAllSum(10, args...)

myAllSum1(5, 6)
= myAllSum(10, 5, 6)
= { cache = 21 } <| 21 <| (...args) => myAllSum(21, args...)
```

不定参数包也可用于扩展函数功能：

```HAM
getArgc = ...args => {
  #curArgc <- 0 <| () => 0 <| (arg, ...rest) => #curArgc(rest...) + 1,
  argc = #curArgc(args...)
},

getAllArgc = ....args => {
  #curArgc <- 0 <| () => 0 <| (arg, ....rest) => #curArgc(rest....) + 1,
  allArgc = #curArgc(args....)
},

f = getArgc <| getAllArgc <| ....args => .argc % 3 + .allArgc % 3
```

## 参数包的类型

参数包在类型里的写法，是在类型前加 `...` 或 `....`：

```HAM
sum <- 0 <| () => 0 <| ((head, ...rest) => head + sum(rest...)),
typeof(sum) // { 0 } <~ () -> { 0 } <~ (Int, ...Int) -> Int

allSum = (....args) => sum(args....),
typeof(allSum) // (....Int) -> Int
```

`...Int` 表示当前结构内的剩余参数，每个都是 `Int`；`....Int` 表示将来所有结构内的所有参数，每个都是 `Int`。判定时 `...Int` 可以吸收任意多个后续参数：

```HAM
sum is () -> Int              // true   成分 () -> { 0 }
sum is (Int) -> Int           // true
sum is (Int, Int) -> Int      // true
sum is (Int, Int, Int) -> Int // true
```

`allSum` 每次调用后函数面还在（`allSum1 $ (5, 6)` 仍然有效），所以它的类型是递归的：

```HAM
typeof(allSum) = let { AllSum <- (....Int) -> (Int <~ AllSum) } in AllSum
```

每次调用返回的是“值面加函数面”，所以 `allSum $ (1, 2, 3, 4) $ (5, 6)` 的值部分是 `Int`，同时它还能继续接收参数。
