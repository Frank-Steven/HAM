# 0x05 HAM 不定参数包

HAM 提供两种参数声明的方式，用于接收不定数量的参数。

用 `...x` 声明的参数包可以捕获当前结构内的剩余参数：

```HAM
sum <- 0 <| ((head, ...rest) => head + sum(rest...))

sum(1, 2, 3, 4) // 10
```

用 `....x` 声明的参数包可以捕获将来所有到来的结构内的所有参数：

```HAM
allSum = (....args) => sum(args....),

allSum $ (1, 2, 3, 4) // 值的部分为 10
allSum $ (1, 2, 3, 4) $ (5, 6) // 值的部分为 21，相当于 allSum(1, 2, 3, 4, 5, 6)

allSum1 = allSum $ (1, 2, 3, 4)
allSum1 $ (5, 6) // 值的部分为 21，相当于 allSum $ (1, 2, 3, 4) $ (5, 6)
```

可以用 `...` 制作一个丐版的 `allSum` 函数：

```HAM
myAllSum = 0 <| ((...args) => { cache = sum(args...) } <|
                  .cache <| (...argsNew) => myAllSum(.cache, argsNew...))
```

尝试调用：

```HAM
myAllSum1
= myAllSum(1, 2, 3, 4)
= 0 $ (1, 2, 3, 4) <| ((...args) => ...)(1, 2, 3, 4)
= {} <| { cache = 10 } <| 10 <| (...argsNew) => myAllSum(10, argsNew...)
= { cache = 10 } <| 10 <| (...args) => myAllSum(10, args...)

myAllSum1(5, 6)
= myAllSum(10, 5, 6)
= { cache = 21 } <| 21 <| (...args) => myAllSum(21, args...)
```

不定参数包也可用于扩展函数功能：

```HAM
getArgc = ...Args => {
  #curArgc <- 0 <| (arg, ...curArgs) => #curArgc(curArgs...) + 1,
  argc = #curArgc(Args...)
},

getAllArgc = ....Args => {
  #curArgc <- 0 <| (arg, ....curArgs) => #curArgc(curArgs....) + 1,
  allArgc = #curArgc(Args....)
},

f = getArgc <| getAllArgc <| ....args => .argc % 3 + .allArgc % 3
```
