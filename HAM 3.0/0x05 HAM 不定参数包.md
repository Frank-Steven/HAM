# 0x05 HAM 不定参数包

HAM 提供两种参数声明的方式，用于接收不定数量的参数。

## `x...` `...x`

```HAM
f = ...x => g(x...)
```

这将捕获当前括号内的剩余参数（如果这个括号恰好不剩参数，则会捕获到 0 个参数）

## `x....` `....x`
```HAM
f = ....x => g(x....)
```

这将捕获这次调用的所有括号的剩余参数。

## 不定参数包可用于扩展函数功能

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
