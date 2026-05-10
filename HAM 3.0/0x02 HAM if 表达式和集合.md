# 0x03 HAM if 表达式和集合

if 表达式可以帮助我们简单地创建分段函数（分支结构），其形式如下

```HAM
a = if cond 100            // 100（cond 为 true）/ {}（cond 为 false）
b = if cond 100 else 10    // 100（cond 为 true）/ 10（cond 为 false）
```

if 表达式应该视为一种分段函数的简化写法

集合可以视为 HAM 的类型系统的底层，也可以用于逻辑运算，在声明组合或声明参数列表时，可以用 `:` 来快速标记所属集合

```HAM
x:Int = 1,
f = (x:Int) => x + 1
```

用集合表达式来表达高级集合

```HAM
f:Int->Int = x:Int => x + 1,
comb:{ x:Int, y:Int } = { x = 1, y:Int = 2 }
```

集合表达式也可以赋给键

```HAM
Num = Int | Float
Function = Int->Int
```

