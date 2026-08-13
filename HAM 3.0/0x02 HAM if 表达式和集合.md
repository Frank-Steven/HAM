# 0x02 HAM if 表达式和集合

## if 表达式

if 表达式可以帮助我们简单地创建分段函数（分支结构），其形式如下：

```HAM
a = if (cond) 100            // 100（cond 为 true）/ {}（cond 为 false）
b = if (cond) 100 else 10    // 100（cond 为 true）/ 10（cond 为 false）
```

if 表达式应该视为一种分段函数的简化写法。

## 集合

集合可以视为 HAM 的类型系统的底层。集合用来表示表达式的定义域。

### 集合的声明

HAM 中有许多内置的集合，例如 `Int`、`Float`、`Bool` 等。

在声明表达式或者声明参数列表时，可以用 `:` 来快速标记所属集合：

```HAM
x: Int = 1,
f = (x: Int) => x + 1
```

如果声明的表达式与集合不匹配，则自动变为一个空组合 `{}`，例如：

```HAM
someFloat: Int = 3.14 // x = {}
```

除内置集合外，还可以用**集合表达式**来表达高级集合。

想要表示包含函数的集合，就需要用到**函数集合表达式**。

```HAM
f: Int -> Int = x: Int => x + 1, // 单参时，括号可以省略
g: (Int, Int) -> Int = (x: Int, y: Int) => x + y, // 多参时，括号不可省略
h: () -> Int = () => 1 // 无参时，括号不可省略
```

想要表示包含组合的集合，就需要用到**组合集合表达式**：

```HAM
comb: { x: Int, y: Int } = { x = 1, y: Int = 2 }
```

集合表达式也可以赋给键：

```HAM
FunctionI2I = Int -> Int
```

当然，还可以用**列举法**表示集合：

```HAM
LessThan3 = { 0, 1, 2 }
```

也可以用**条件法**表示集合：

```HAM
Even = {...| x: Int => x % 2 == 0 }
Odd = {...|_ % 2 == 1 } // 用 `_` 语法糖简化单参函数
FirstQuadrant = {...| (e: {x: Num, y: Num}) => e.x > 0 && e.y > 0 }
```

`...|` 后的函数必须是一个返回 `Bool` 的单参函数，具体而言，该函数属于 `Anything -> Bool`。任何表达式 `x` 属于集合 `{...| f}`，当且仅当 `f(x)` 为 `true`。

### 集合的运算

`in` 运算符用来判断一个东西是否**属于**某个集合：

```HAM
1 in LessThan3                     // true
3 in LessThan3                     // false
2 <| x => x + 1 in LessThan3       // true
comb in { x: Int, y: Int }         // true
comb in { x: Int, y: Float }       // false
comb in { x: Int, y: Int, z: Int } // false
comb in { x: Int }                 // true
```

`in` 运算符的优先级低于 `<|` 运算符。

> 提示：`x in A` 可以理解为，把 `x` 用在任何需要 `A` 中元素的位置都不会有问题、能满足要求。具体规则见 `<~` 运算符的说明。

`notin` 运算符是 `in` 运算符的否定，`x notin A` 等价于 `!(x in A)`：

```HAM
1 notin LessThan3               // false
3 notin LessThan3               // true
2 <| x => x + 1 notin LessThan3 // false
```

`|` 运算符用来表示集合的**并集**：

```HAM
LessThan5 = LessThan3 | {3} | {4}, // { 0, 1, 2, 3, 4 }
Num = Int | Float | Double
```

`&` 运算符用来表示集合的**交集**：

```HAM
LessThan3 & LessThan5 // { 0, 1, 2 }
Num & Int // Int
```

`<~` 运算符用来表示集合的**扩展**，集合的扩展规则与其它表达式使用的 `<|` 扩展规则类似：

```HAM
NumDeltaXY = Num <~ { x: Int, y: Int }
1                                       in NumDeltaXY // false
{ x = 1, y = 2 }                        in NumDeltaXY // false
1 <| { x = 1, y = 2 }                   in NumDeltaXY // true
1 <| { x = 1 }                          in NumDeltaXY // false
1 <| { x = 1, y = 2, z = 3 } <| x => -x in NumDeltaXY // true

NumDeltaFii = Num <~ (Int -> Int)
1                      in NumDeltaFii // false
(x: Int) => x + 1      in NumDeltaFii // false
1 <| (x: Int) => x + 1 in NumDeltaFii // true
1 <| (x: Int) => "abc" in NumDeltaFii // false

FiiDeltaNum = (Int -> Int) <~ Num
1 <| (x: Int) => x + 1 in FiiDeltaNum // false
(x: Int) => x + 1 <| 1 in FiiDeltaNum // true
```

注意与 `<|` 运算符的区别，`<~` 运算符是集合的扩展运算符，而 `<|` 运算符是表达式的扩展运算符。集合在表达式中作为值存在，所以会遵循覆写原则：

```HAM
SomeSet1 = { 1, 2, 3 } <~ { 2, 3, 4 } // { 2, 3, 4 }
SomeSet2 = { x: Int, y: Int } <~ { x: Float, z: Float } // { x: Float, y: Int, z: Float }
SomeSet3 = { 1, 2, 3 } <| { 2, 3, 4 } // { 2, 3, 4 }
SomeSet4 = { x: Int, y: Int } <| { x: Float, z: Float } // { x: Float, z: Float }
```

> 提示：普通组合在 `<|` 运算符下遵循按键覆写原则。但集合是原子值，所以在 `<|` 运算符下遵循覆写原则，会被整体替代。注意区分 `{ x: Int, y: Int }` 和 `{ x = Int, y = Int }`，前者是组合集合表达式，后者是组合（值是集合）。

对于被 `<~` 运算符扩展的集合，判定 `x in A` 的规则是：

假设 `x` 和 `A` 可以被写为：

```HAM
x = ... <| fn2 <| fn1 <| val <| comb <| f1 <| f2 <| ...
A = ... <~ Fn2 <~ Fn1 <~ Val <~ Comb <~ F1 <~ F2 <~ ...
```

则 `x in A` 当且仅当：

- `val in Val` 或 `A` 中没有 `Val`
- `comb in Comb` 或 `A` 中没有 `Comb`
- 对任意 `Fi`，存在 `fk`，使得 `fk in Fi`
- 对任意 `Fnj`，存在 `fnl`，使得 `fnl in Fnj`

这可以被总结成以下几点：

- **可多不可少**：`comb` 里的键可以多于 `Comb` 里的键，但不能少于 `Comb` 里的键；`Fn` 和 `F` 可以被多个 `fn` 和 `f` 对应，但每个 `Fn` 和 `F` 至少要有一个对应的 `fn` 和 `f`
- **前后分开算**：`Fn` 跟 `fn` 对应，`F` 跟 `f` 对应，`Val` 跟 `val` 对应，`Comb` 跟 `comb` 对应，前后不混合

`~` 运算符用来表示集合的**补集**。

`~A` 是**所有与 `A` 不相交的集合**的并集。两个集合是否不相交由神谕协助判定。

```HAM
1 in ~LessThan3              // false
1 <| { x = 2 } in LessThan3  // true
1 <| { x = 2 } in ~LessThan3 // true
x => x + 1     in ~LessThan3 // true
~LessThan3     in ~LessThan3 // true
```

> 注意：`~A` 不是严格的集合论补集。`x in ~A` 与 `x in A` 可以同时成立。例如 `1 <| { x = 2 }` 同时属于 `LessThan3`（值部分 `1` 属于 `{1}`）和 `~LessThan3`（组合部分 `{ x = 2 }` 属于 `{ x: Int }`）。因此 `x in ~A` **不意味着** `x notin A`，`notin` 只是 `in` 的否定。

`-` 运算符用来表示集合的**差集**。

与传统上 `A & ~B` 不同的是，HAM 的差集的计算方式是，把 `A` 拆成与 `B` 不相交的集合，然后取它们的并。

```HAM
LessThan5 - LessThan3 // { 3, 4 }
Num - Int // Float | Double
```

`subseteq` 运算符用来判断集合的**子集**关系：

```HAM
LessThan3 subseteq LessThan3 // true
LessThan3 subseteq LessThan5 // true
Num subseteq NumDeltaXY      // false
```

`subset` 运算符用来判断集合的**真子集**关系：

```HAM
LessThan3 subset LessThan3 // false
LessThan3 subset LessThan5 // true
Num subset NumDeltaXY      // false
```

## 附录

### 内置集合一览

### 集合运算符一览

| 运算符     | 说明                                 |
| ---------- | ------------------------------------ |
| `in`       | 判断一个表达式是否属于某个集合       |
| `notin`    | 判断一个表达式是否不属于某个集合     |
| `\|`       | 两集合的并集                         |
| `&`        | 两集合的交集                         |
| `~`        | 与集合不相交的集合的并集             |
| `-`        | 两集合的差集                         |
| `subseteq` | 判断一个集合是否是另一个集合的子集   |
| `subset`   | 判断一个集合是否是另一个集合的真子集 |
| `<~`       | 集合的扩展                           |
