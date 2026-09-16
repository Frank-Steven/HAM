# 0x03 HAM Something 与集合运算

HAM 中的所有表达式类型现在你已经全部见过了，在 HAM 中，这些东西共同属于集合 `Anything`（或 `Any`），它包含了你在 HAM 中能见到的一切合法的表达式。

除了 `Anything` 之外，本节将重点介绍构成它的两个子集：`Something`（或 `Sth`）和 `Nothing`。

注：本节中，被代码块包裹的 `Something` 和 `Nothing` 是集合的名字，而未被代码块包裹的 "Something" 是表达式的名字。

## Nothing

`Nothing` 是一个仅含 `{}`（空组合）的集合，空组合既不能参与运算，又不能被调用，是 HAM 里看似最没用的存在。

然而，空组合的存在让 HAM 可以在函数调用不合法时返回一个空组合，表示“没有值”，而不是抛出异常。

空组合也是所有未定义表达式名称的默认初始值。

空组合也是 `<|` 运算的恒等元，`{} <| x = x`，`x <| {} = x`。空组合应当作组合使用。

注意“结构” `()` 不能在 Something 中出现。准确来说，“结构”是一个神谕，不能参与 `<|` 运算，只能参与 `$` 算符。

这让函数扩展的语义和组合结构完全统一，也让函数的求值模型顺理成章。

## Something

`Something` 是 `Anything` 去掉 `{}` 的集合。Something 表示一个可以被使用的东西，可以参与运算，可以被调用。

### Something 的稳定性

Something 的稳定性是指它是否包含函数。

**稳定**的 Something 参与 `$` 运算时永远返回一个空组合 `{}`。而不稳定的 Something 被调用后可能返回一个新 Something。

稳定的 Something，也称**纯稳定**的 Something，只能包含值和组合；**纯不稳定**的 Something，只能包含函数。Something 里的各元素之间应由 `<|` 运算符连接，遵循覆写原则。

**不稳定**的 Something 可以包含值、组合，必须包含函数。

### Something 标准式

Something 的标准式是指它被化简后的形式。可以被写成：

纯不稳定的 Something 标准式是函数按出现顺序 `<|` 起来的形式，至少有一个函数：`函数 <| 函数 <| ... <| 函数`。

纯稳定的 Something 标准式是组合与值按顺序 `<|` 起来的形式。二者至少出现其一：

- `非空组合 <| 值`
- `值`
- `非空组合`

而 Something 标准式则是组合、值、纯不稳定的 Something 块三者按顺序 `<|` 起来的形式。三者至少出现其中之一：

- `非空组合 <| 值 <| 纯不稳定的 Something 标准式`
- `非空组合 <| 值`
- `非空组合 <| 纯不稳定的 Something 标准式`
- `值   <| 纯不稳定的 Something 标准式`
- `纯不稳定的 Something 标准式`
- `非空组合`
- `值`

## 集合的运算

有了 `Something` 后，HAM 的类型系统就可以被扩展为一个集合系统。

### 值与集合

`is` 运算符用来判断一个东西是否**可被用作某个集合中的元素**。

与数学上 $\in$ 的定义不同，`x is A` 并不要求 `x` 是 `A` 的成员，而是要求把 `x` 当作 `A` 中的元素使用不会出现未定义行为。

```HAM
LessThan3 = { 0, 1, 2 },
1 is LessThan3                     // true
3 is LessThan3                     // false
2 <| x => x + 1 is LessThan3       // true
comb is { x: Int, y: Int }         // true
comb is { x: Int, y: Float }       // false
comb is { x: Int, y: Int, z: Int } // false
comb is { x: Int }                 // true
```

> 提示：上述代码不是真正的 HAM 代码，因为表达式出现在了组合的顶层，而在真正的 HAM 代码中，表达式应当出现在组合的键值中。

注意：`typeof({})` 是 `Nothing`（仅含 `{}` 的集合），于是 `{} is A` 就是 `Nothing subseteq A`，也就是判断 `{}` 是否属于 `A`。由于空组合属于一切非空集合，这对任意非 `Empty` 的 `A` 都成立。

如果 `A` 不是集合，则 `x is A` 为 `{}`，因为 `A isnt Set` 对于 `is` 来说是非法的。

`is` 运算符的优先级低于 `<|` 运算符。

`isnt` 运算符是 `is` 运算符的否定，`x isnt A` 等价于 `!(x is A)`：

```HAM
1 isnt LessThan3               // false
3 isnt LessThan3               // true
2 <| x => x + 1 isnt LessThan3 // false
```

### 相等

`==` 运算符用来判断两个东西的**值**是否相等，`!=` 是它的否定。

它只作用在值面上，组合面与函数面不参与。这是因为它的定义域仅限于值面：

```HAM
1 == 1                           // true
1 == 2                           // false
1 <| { x = 1 } == 1              // true，只看值面
1 <| { x = 1 } == 1 <| { y = 2 } // true，组合面不同也不影响
```

集合也是值，所以集合也可以判定相等：

```HAM
{ 0, 1, 2 } == { 2, 0, 1 } // true
Int == Int                 // true
Int == Float               // false
Bool == { true, false }    // true
```

然而，有一些特殊的集合无法判定相等：

```HAM
A = {...| (x: Int) => isPrimeA(x) },
B = {...| (x: Int) => isPrimeB(x) },
A == B // {}
```

`{}` 没有值面，但两个 `{}` 相等：

```HAM
{} == {} // true
{} == 1  // {}
```

除此之外，没有值面的东西不在 `==` 的定义域里，结果是 `{}`：

```HAM
{ x = 1 } == { x = 1 } // {}
(x => x) == (x => x)   // {}
```

函数不是值，而组合的相等需要递归到它里面的值才能判定，所以 `==` 不处理它们。

`==` 与 `is` 是两个独立的判定，一个看值相不相等，一个看有没有可用的成分：

```HAM
1 <| { x = 1 } == 2     // false，值面是 1
1 <| { x = 1 } is { 1 } // true，成分里有 { 1 }
```

### 集合与集合

#### 并集与交集

`|` 运算符用来表示集合的**并集**：

```HAM
LessThan5 = LessThan3 | {3} | {4}, // { 0, 1, 2, 3, 4 }
Num = Int | Float
```

`&` 运算符用来表示集合的**交集**：

```HAM
LessThan3 & LessThan5 // { 0, 1, 2 }
Num & Int // Int
```

#### 集合的扩展

`<~` 运算符（蝌蚪运算符）用来表示集合的**扩展**，集合的扩展规则与其它表达式使用的 `<|` 扩展规则类似：

```HAM
NumDeltaXY = Num <~ { x: Int, y: Int }
1                                       is NumDeltaXY // false
{ x = 1, y = 2 }                        is NumDeltaXY // false
1 <| { x = 1, y = 2 }                   is NumDeltaXY // true
1 <| { x = 1 }                          is NumDeltaXY // false
1 <| { x = 1, y = 2, z = 3 } <| x => -x is NumDeltaXY // true

NumDeltaFii = Num <~ (Int -> Int)
1                      is NumDeltaFii // false
(x: Int) => x + 1      is NumDeltaFii // false
1 <| (x: Int) => x + 1 is NumDeltaFii // true
1 <| (x: Int) => "abc" is NumDeltaFii // false

FiiDeltaNum = (Int -> Int) <~ Num
1 <| (x: Int) => x + 1 is FiiDeltaNum // false
(x: Int) => x + 1 <| 1 is FiiDeltaNum // true
```

注意与 `<|` 运算符的区别，`<~` 运算符是集合的扩展运算符，而 `<|` 运算符是表达式的扩展运算符。集合在表达式中作为值存在，所以会遵循覆写原则：

```HAM
SomeSet1 = { 1, 2, 3 } <~ { 2, 3, 4 }                   // { 2, 3, 4 }
SomeSet2 = { 1, 2, 3 } <| { 2, 3, 4 }                   // { 2, 3, 4 }

SomeSet3 = { x: Int, y: Int } <~ { x: Float, z: Float } // { x: Float, y: Int, z: Float }
SomeSet4 = { x: Int, y: Int } <| { x: Float, z: Float } // { x: Float, z: Float }
```

> 提示：普通组合在 `<|` 运算符下遵循按键覆写原则。但集合是原子值，所以在 `<|` 运算符下遵循覆写原则，会被整体替代。注意区分 `{ x: Int, y: Int }` 和 `{ x = Int, y = Int }`，前者是组合集合表达式，后者是组合（值是集合）。

#### 补集与差集

`~` 运算符用来表示集合的**补集**。补集作用在成分上，`~A` 就是 `Anything - A`：一个东西只要有一个成分不属于 `A`，它就属于 `~A`。

```HAM
1 is ~LessThan3              // false
1 <| { x = 2 } is LessThan3  // true
1 <| { x = 2 } is ~LessThan3 // true
x => x + 1     is ~LessThan3 // true
~LessThan3     is ~LessThan3 // true
```

> 注意：`is` 不是严格的集合论属于。`x is ~A` 与 `x is A` 可以同时成立。例如 `1 <| { x = 2 }` 同时属于 `LessThan3`（值部分 `1` 属于 `{1}`）和 `~LessThan3`（组合部分 `{ x = 2 }` 属于 `{ x: Int }`）。因此 `x is ~A` **不意味着** `x isnt A`，`isnt` 只是 `is` 的否定。

> 提示：这并不影响 `A & ~A` 是空集。`x is A & ~A` 要求**同一个成分**同时属于两边，而一个成分不会既属于 `A` 又不属于 `A`，所以它恒为假。

`-` 运算符用来表示集合的**差集**。

```HAM
LessThan5 - LessThan3 // { 3, 4 }
Num - Int // Float
```

#### 子集关系

`subseteq` 运算符用来判断集合的**子集**关系：

```HAM
LessThan3 subseteq LessThan3 // true
LessThan3 subseteq LessThan5 // true
Num subseteq NumDeltaXY      // false
```

`subseteq` 是**成分级**的，与 `is` 用的是同一个判定。一个被 `<~` 连接起来的集合，它的每一段都是一个**成分**，例如 `NumDeltaXY = Num <~ { x: Int, y: Int }` 的成分是 `Num` 与 `{ x: Int, y: Int }`。于是：

```HAM
X subseteq Y  当且仅当  X 自身或它的某个成分 C 满足 C subseteq Y
```

所以只要某个成分是另一个集合的子集，整个集合就是另一个集合的子集：

```HAM
NumDeltaXY subseteq Num // true，因为成分里的 Num subseteq Num
```

> 提示：`Num subseteq NumDeltaXY` 为假，是因为反过来不成立：`Num` 是原子集合，没有成分可分，而它的数值不属于需要一个组合面的 `NumDeltaXY`。

`subset` 运算符用来判断集合的**真子集**关系：

```HAM
LessThan3 subset LessThan3 // false
LessThan3 subset LessThan5 // true
Num subset NumDeltaXY      // false
```

#### 复合集合的判定

`|`、`&`、`-`、`~` 的结果也可以被 `is` 判定，但只有并集能写成布尔组合：

| 写法            | 判定                                                                |
| --------------- | ------------------------------------------------------------------- |
| `x is A \| B`   | `x is A \|\| x is B`                                                |
| `x is A & B`    | 存在**同一个**成分 `C`，使 `C subseteq A` 且 `C subseteq B`         |
| `x is A - B`    | 存在**同一个**成分 `C`，使 `C subseteq A` 且 `C` 不 `subseteq B`    |
| `x is ~A`       | 存在**同一个**成分 `C`，使 `C` 不 `subseteq A`                      |

`1 <| { x = 2 }` 是最小的例子：它 `is LessThan3`（值部分），也 `is ~LessThan3`（组合部分），但 `is LessThan3 & ~LessThan3` 为假——两次判定用的是不同的成分，而 `&` 要求同一个成分同时落在两边。

## 集合与类型

HAM 的类型系统是建立在集合系统之上的。可以用 `typeof` 函数来获取一个表达式的类型（即所属的集合）。

### 类型判定

HAM 中的 `is` 运算符可以用于类型检查。`x is A` 等同于 `typeof(x) subseteq A`，其中的 `subseteq` 是成分级的。

对原子集合（`Int`、`{ 1 }`、`{ x: Int }` 等），`typeof(x) subseteq A` 就是普通的包含；对 `<~` 链，它可以展开成逐面的形式。假设 `x` 和 `A` 已被化简为：

```HAM
x = comb <| val <| f_1 <| f_2 <| ... <| f_n
A = Comb <~ Val <~ F_1 <~ F_2 <~ ... <~ F_m
```

则 `x is A` 当且仅当：

- 对值面，满足其一：
    - `val` 与 `Val` 都存在，且 `val is Val`
    - `val` 存在而 `Val` 不存在
- 对组合面，满足其一：
    - `comb` 与 `Comb` 都存在，且 `comb is Comb`
    - `comb` 存在而 `Comb` 不存在
- 对函数面，满足其一：
    - `m` 大于 $0$，且对任意 `F_i`，存在严格递增的数列 `a`, 使得 `f_a_i is F_i`
    - `m` 为 $0$

### 字面量的类型

由于 HAM 没有基本类型（因为没有基本集合），所以字面量的类型就是它自己（注意函数内容的丢失）：

```HAM
typeof(1)         // { 1 }
typeof(3.14)      // { 3.14 }
typeof('a')       // { 'a' }
typeof("abc")     // { "abc" }
typeof({ x = 1 }) // { { x = 1 } }
typeof(`_ + 1`)   // { (Int | Float) -> Int | Float }
typeof(Int)       // { Int }
typeof({})        // Nothing
```

字符字面量用单引号（如 `'a'`），字符串字面量用双引号（如 `"abc"`）。字符串是字符的数组：`String = Char[]`。

> HAM 格言：`万物即{万物}`

### 键的类型

如果一个键在定义的时候，没有标记它的类型，则它的类型就是它的值的类型：

```HAM
x = 1,
y = { x = 1 },
Num = Int | Float
typeof(x)   // { 1 }
typeof(y)   // { { x = 1 } }
typeof(Num) // { Int | Float }

inc = (x: Int) -> Int => x + 1,
z = inc(x),      // 2    因为 x 的类型 { 1 } subseteq Int，所以可以被传入 inc 而不得到 {}
typeof(z)        // Int  因为 inc 的返回值类型是 Int
typeof(z <| inc) // Int <~ Int -> Int
```

用 `as` 关键字可以约束类型：

```HAM
x = 1                as Int,
y = { x = 1 }        as { x: Int },
z = { x = 1, y = 2 } as { y: Int }, // { y = 2 } 这里面 x 被 as 吃了
inc = `_ + 1`        as Int -> Int,
sth = 1 <| { x = 1 } as Int         // 1         这里面 { x = 1 } 被 as 吃了

typeof(x)   // Int
typeof(y)   // { x: Int }
typeof(z)   // { y: Int }
typeof(inc) // Int -> Int
typeof(sth) // Int
```

如果使用了 `<|` 运算符，则它的类型会使用 `<~` 运算符构建：

```HAM
typeof(1 <| { x = 1 })         // { x: { 1 } } <~ { 1 }
typeof(1 <| (x: Int) => x + 1) // { 1 } <~ Int -> Int
typeof(1 <| { x = 2 } <| 2)    // { x: { 2 } } <~ { 2 }

typeof(1 <| (x: Int) => { a = x + 1 } <| .a)             // { 1 } <~ Int -> { a: Int }
                                                         // 其中 .a 为 {}
typeof(1 <| (x: Int) => { a = x + 1 } <| (x: Int) => .a) // { 1 } <~ Int -> { a: Int } <~ Int
                                                         // 其中 .a 是前面函数返回组合中 a = x + 1 的投影
```

## 附录

### 集合运算符一览

| 运算符     | 说明                                 |
| ---------- | ------------------------------------ |
| `is`       | 判断一个表达式是否属于某个集合       |
| `isnt`     | 判断一个表达式是否不属于某个集合     |
| `\|`       | 两集合的并集                         |
| `&`        | 两集合的交集                         |
| `~`        | `Anything - A`，即集合的补集         |
| `-`        | 两集合的差集                         |
| `subseteq` | 判断一个集合是否是另一个集合的子集   |
| `subset`   | 判断一个集合是否是另一个集合的真子集 |
| `<~`       | 集合的扩展                           |

### 内置集合一览

| 集合名      | 说明                       |
| ----------- | -------------------------- |
| `Empty`     | 空集，所有东西都不 `is` 它 |
| `Anything`  | 全体表达式                 |
| `Something` | 全体非 `{}` 表达式         |
| `Nothing`   | 仅含 `{}` 的集合           |
| `Int`       | 全体整数                   |
| `Float`     | 全体小数                   |
| `Char`      | 全体字符                   |
| `String`    | 全体字符串（即 `Char[]`）  |
| `Bool`      | `true` 或 `false`          |
| `Function`  | 全体函数                   |
| `Set`       | 全体集合                   |
