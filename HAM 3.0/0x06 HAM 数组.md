# 0x06 HAM 数组

数组是内置函数，有大量的语法糖。

## 数组的声明

数组以 `Array` 函数创建：

```HAM
arr1 = Array(),
arr2 = Array(1, 2, 3)
```

可以用中括号来简化，上面的代码等价于：

```HAM
arr1 = [],
arr2 = [1, 2, 3]
```

## 数组的类型集合

在对应类型后加 `[]` 表示该类型的数组集合，这是**数组集合表达式**：

```HAM
arr3: Int[]  = [2, 3, 4] // [2, 3, 4]
arr4: Int[5] = [1]       // [1, {}, {}, {}, {}]
arr5: Int[2] = [1, 2, 3] // {}
```

当然也可以表示高维数组：

```HAM
arr6: Int[][] = [[1], [2, 3]] // 犬牙交错数组，每一项的长度可以不同
```

> 提示：`Int[][]` 其实就是 `(Int[])[]`

数组长度与标注不一致时，若长度不足用 `{}` 补齐，长度超出则视为与标注不匹配，结果为 `{}`。

注意数组字面量的类型也满足格言：`typeof([1, 2, 3])` 就是 `{ [1, 2, 3] }`。

## 数组的操作

注意：以下所有用键来访问的 API 都有一套 `Array` 组合的函数式调用方法。

### 引用元素

可以用 `at` 函数来获取数组对应下标的元素：

```HAM
arr2.at(0) // 1
```

可以用中括号来简化，上面的代码等价于：

```HAM
arr2[0] // 1
```

用 `Array` 组合的函数式调用方法：

```HAM
Array.at(arr2, 0) // 1
```

或使用 `|>` 运算符：

```HAM
arr2 |> Array.at(0) // 1
```

### 修改元素并覆写

可以用 `modify` 函数来修改并覆写数组的元素：

```HAM
arr3 = arr3.modify(1, 0) // arr3 变为 [2, 0, 4]
```

也可以覆写由中括号拿到的引用：

```HAM
arr3[1] = 2 // arr3 变为 [2, 2, 4]
```

### 过滤器

可以用 `filter` 函数来过滤数组中符合条件的元素：

```HAM
arr2.filter(x => x > 1) // [2, 3]
```

HAM 默认为数组重载了 `|` 运算符，用于简化代码：

```HAM
arr2 | x => x > 1 // [2, 3]
```

也可以用 `_` 语法糖来简化单参函数：

```HAM
arr2 | _ > 1 // [2, 3]
```

还可以用 `|>` 运算符把管道式调用串起来，例如先过滤再映射：

```HAM
arr2 |> Array.filter(_ > 1) |> Array.map(_ * 2) // [4, 6]
```

### 长度

可以用 `length` 函数获取数组的长度：

```HAM
arr1.length() // 0
arr2.length() // 3
```

### 拼接

HAM 默认为数组重载了 `+` 运算符，用于数组的拼接：

```HAM
arr2 + [4, 5] // [1, 2, 3, 4, 5]
```

当然也可以用 `concat` 函数来拼接两个数组：

```HAM
Array.concat(arr2, [4, 5]) // [1, 2, 3, 4, 5]
```

### 映射

可以用 `map` 函数对数组中的每个元素应用一个函数，返回映射后的新数组：

```HAM
arr2.map(x => x * 2) // [2, 4, 6]
```

如果 `map` 的函数返回数组，可以用 `flatMap` 把结果压平一层，相当于 `map` 后再 `flatten`：

```HAM
arr2.flatMap([_, -_]) // [1, -1, 2, -2, 3, -3]
```

`flatten` 可以把数组的数组压平一层：

```HAM
arr6.flatten() // [1, 2, 3]
```

### 折叠

可以用 `fold` 函数把数组归约为一个值，需要给定初始值和一个二元函数，从左到右依次把元素合入累加器：

```HAM
arr2.fold(0, (acc, x) => acc + x) // 6
```

### 切片

可以用 `take`、`drop`、`slice` 获取数组的一部分，都返回新数组：

```HAM
arr2.take(2)     // [1, 2]
arr2.drop(2)     // [3]
arr2.slice(1, 3) // [2, 3]
```

`slice(from, to)` 是 `[from, to)` 半开区间；`take`、`drop` 超出长度时截到末尾，`slice` 超出范围返回 `{}`。

### 反转

可以用 `reverse` 函数反转数组，返回新数组：

```HAM
arr2.reverse() // [3, 2, 1]
```

### 查找

可以用 `indexOf` 获取元素第一次出现的下标，用 `contains` 判断数组是否包含元素：

```HAM
arr2.indexOf(2) // 1
arr2.indexOf(9) // {}
arr2.contains(2) // true
```

`indexOf` 找不到时返回 `{}`。

## 附录

### 不可变 API 一览表

| API                                 | 说明                     |
| ----------------------------------- | ------------------------ |
| `Array.at(array, index)`            | 获取数组对应下标的元素   |
| `Array.modify(array, index, value)` | 修改并覆写数组的元素     |
| `Array.filter(array, predicate)`    | 过滤数组中符合条件的元素 |
| `Array.length(array)`               | 获取数组的长度           |
| `Array.concat(...arraies)`          | 拼接多个数组             |
| `Array.map(array, f)`               | 每个元素经 `f` 映射      |
| `Array.flatten(array)`              | 扁平化数组               |
| `Array.flatMap(array, f)`           | 映射后扁平化             |
| `Array.fold(array, init, f)`        | 从左到右折叠             |
| `Array.take(array, n)`              | 取前 `n` 个元素          |
| `Array.drop(array, n)`              | 去掉前 `n` 个元素        |
| `Array.slice(array, from, to)`      | 切片                     |
| `Array.reverse(array)`              | 反转                     |
| `Array.indexOf(array, x)`           | 第一个 `x` 的下标        |
| `Array.contains(array, x)`          | 是否包含 `x`             |
