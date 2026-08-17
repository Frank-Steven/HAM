# 0x03 HAM Something 与集合运算

HAM 中的所有表达式类型现在你已经全部见过了，在 HAM 中，这些东西共同属于集合 `Anything`（或 `Any`），它包含了你在 HAM 中能见到的一切合法的表达式。

除了 `Anything` 之外，本节将重点介绍构成它的两个子集：`Something`（或 `Sth`）和 `Nothing`。

注：本节中，被代码块包裹的 `Something` 和 `Nothing` 是集合的名字，而未被代码块包裹的 "Something" 是表达式的名字。

## Nothing

`Nothing` 是一个仅含 `{}`（空组合）的集合，空组合既不能参与运算，又不能被调用，是 HAM 里看似最没用的存在。

然而，空组合的存在让 HAM 可以在函数调用不合法时返回一个空组合，表示“没有值”，而不是抛出异常。

空组合也是所有未定义表达式名称的默认初始值。

此外，空组合属于一切集合——`{} in Int` 为 `true`（`Empty` 除外）。`in` 判定的是“把 `x` 当作集合元素使用不会出现未定义行为”，而 `{}` 表示“没有值”，作为任何集合的退化成员被放行。这保证了 `x: Int = 3.14` 得到的 `{}` 依然满足 `Int` 标注：类型不匹配的结果是“没有值”，而不是类型错误。

这让函数扩展的语义和组合结构完全统一，也让函数的求值模型顺理成章。

## Something

`Something` 是 `Anything` 去掉 `{}` 的集合。Something 表示一个可以被使用的东西，可以参与运算，可以被调用。

### Something 的稳定性

Something 的稳定性是指它是否包含函数，稳定的 Something 被调用时，结构为空则返回自身，结构非空则返回 `{}`；不稳定的 Something 被调用后可能返回一个新 Something。

稳定的 Something，也称纯稳定的 Something，只能包含值和组合；纯不稳定的 Something，只能包含函数。Something 里的各元素之间应由 `<|` 运算符连接，遵循覆写原则。

不稳定的 Something 可以包含值、组合，必须包含函数。

### Something 标准式

Something 的标准式是指它被化简后的形式。可以被写成：

纯不稳定的 Something 标准式 = `函数 <| 函数 <| ... <| 函数`。

纯稳定的 Something 标准式 = `组合 <| 值`。

不稳定的 Something 标准式由纯稳定块与纯不稳定块交替组成，任一种块都可以开头：

`纯稳定 <| 纯不稳定 <| 纯稳定 <| ...` 或 `纯不稳定 <| 纯稳定 <| 纯不稳定 <| ...`。

## 集合的运算

有了 `Something` 后，HAM 的类型系统就可以被扩展为一个集合系统。

### 值与集合

`in` 运算符用来判断一个东西是否**可被用作某个集合中的元素**。

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

> 提示：`x in A` 可以理解为，把 `x` 用在任何需要 `A` 中元素的位置都不会出现未定义行为。具体规则见 `<~` 运算符的说明。

`notin` 运算符是 `in` 运算符的否定，`x notin A` 等价于 `!(x in A)`：

```HAM
1 notin LessThan3               // false
3 notin LessThan3               // true
2 <| x => x + 1 notin LessThan3 // false
```

### 集合与集合

#### 并集与交集

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

#### 集体的扩展

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
SomeSet1 = { 1, 2, 3 } <~ { 2, 3, 4 }                   // { 2, 3, 4 }
SomeSet2 = { 1, 2, 3 } <| { 2, 3, 4 }                   // { 2, 3, 4 }

SomeSet3 = { x: Int, y: Int } <~ { x: Float, z: Float } // { x: Float, y: Int, z: Float }
SomeSet4 = { x: Int, y: Int } <| { x: Float, z: Float } // { x: Float, z: Float }
```

> 提示：普通组合在 `<|` 运算符下遵循按键覆写原则。但集合是原子值，所以在 `<|` 运算符下遵循覆写原则，会被整体替代。注意区分 `{ x: Int, y: Int }` 和 `{ x = Int, y = Int }`，前者是组合集合表达式，后者是组合（值是集合）。

对于被 `<~` 运算符扩展的集合，判定 `x in A` 的规则是：

假设 `x` 和 `A` 可以被写为：

```HAM
x = ... <| fn2 <| fn1 <| val <| comb <| f1 <| f2 <| ...
A = ... <~ Fn2 <~ Fn1 <~ Val <~ Comb <~ F1 <~ F2 <~ ...
```

这是交替标准式的实例：以纯不稳定开头，全部稳定块合并为一段 `val <| comb`，函数块按相对位置分列两侧，任一侧可以为空。

则 `x in A` 当且仅当：

- `val in Val` 或 `A` 中没有 `Val`
- `comb in Comb` 或 `A` 中没有 `Comb`
- 对任意 `Fi`，存在 `fk`，使得 `fk in Fi`
- 对任意 `Fnj`，存在 `fnl`，使得 `fnl in Fnj`

这可以被总结成以下几点：

- **可多不可少**：`comb` 里的键可以多于 `Comb` 里的键，但不能少于 `Comb` 里的键；`Fn` 和 `F` 可以被多个 `fn` 和 `f` 对应，但每个 `Fn` 和 `F` 至少要有一个对应的 `fn` 和 `f`
- **不稳定块分开算**：`Fn` 跟 `fn` 对应，`F` 跟 `f` 对应，`Val` 跟 `val` 对应，`Comb` 跟 `comb` 对应，前后不混合

#### 补集与差集

`~` 运算符用来表示集合的**补集**。

`~A` 是**所有与 `A` 不相交的集合**的并集。两个集合是否不相交由神谕协助判定，判定时忽略空组合：既然 `{}` 属于一切集合，若把它计入，任何集合之间都不可能不相交，`~A` 将退化为空集。所谓不相交，指两个集合中除 `{}` 之外的成员互不重叠。

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

#### 子集关系

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

## 集合与类型

HAM 的类型系统是建立在集合系统之上的。可以用 `typeof` 函数来获取一个表达式的类型（即所属的集合）。

> 提示：`in` 运算符还可以理解为“类型检查”，`x in A` 等同于 `typeof(x) subseteq A`。

### 字面量的类型

由于 HAM 没有基本类型（因为没有基本集合），所以字面量的类型就是它自己：

```HAM
typeof(1)         // { 1 }
typeof(3.14)      // { 3.14 }
typeof("abc")     // { "abc" }
typeof({ x = 1 }) // { { x = 1 } }
typeof( _ + 1 )   // { x => x + 1 }
typeof(Int)       // Int
```

> HAM 格言：`万物即{万物}`

### 键的类型

如果一个键在定义的时候，没有标记它的类型，则它的类型就是它的值的类型：

```HAM
x = 1,
y = { x = 1 },
typeof(x) // { 1 }
typeof(y) // { { x = 1 } }

inc: Int -> Int = _ + 1,
z = inc(x), // 这里 x 的类型 { 1 } subseteq Int，所以可以被传入 inc 而不得到 {}
typeof(z)   // Int 因为 inc 的返回值类型是 Int
```

有类型标记时，该键会承载标记的类型信息：

```HAM
x: Int = 1,
y: { x: Int } = { x = 1 },
inc: Int -> Int = _ + 1,

typeof(x)   // Int
typeof(y)   // { x: Int }
typeof(inc) // Int -> Int
```

## 附录

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

### 内置集合一览

| 集合名      | 说明                       |
| ----------- | -------------------------- |
| `Empty`     | 空集，所有东西都不 `in` 它 |
| `Anything`  | 全体表达式                 |
| `Something` | 全体非 `{}` 表达式         |
| `Nothing`   | 仅含 `{}` 的集合           |
| `Int`       | 全体整数                   |
| `Float`     | 全体小数                   |
| `Char`      | 全体字符                   |
| `Bool`      | `true` 或 `false`          |
| `Function`  | 全体函数                   |
| `Set`       | 全体集合                   |
