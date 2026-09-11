
## 1. 转换总览

C 语言中的类型转换分为两大类：

|类别|触发方式|安全性|
|---|---|---|
|**隐式转换**（Implicit）|编译器自动执行，无需程序员干预|通常安全，但可能丢失精度|
|**显式转换**（Explicit）|程序员通过**强制类型转换**（cast）手动指定|程序员承担后果|

隐式转换又可分为三种场景：

|场景|触发时机|别名|
|---|---|---|
|**整型提升**（Integer Promotion）|表达式求值、函数参数传递|小整数 → `int`|
|**算术转换**（Arithmetic Conversion）|二元运算符两侧类型不同时|统一为公共类型|
|**赋值转换**（Assignment Conversion）|赋值运算符 `=` 右侧赋给左侧时|右侧类型 → 左侧类型|

### 完整转换链路总览

```
                        ┌──────────────────────────────┐
                        │     所有隐式转换触发点         │
                        └──────────────────────────────┘
                             │              │              │
                  ┌──────────┘              │              └──────────┐
                  ▼                         ▼                         ▼
            整型提升                   算术转换                   赋值转换
        (char/short→int)        (二元运算符两侧类型统一)      (右侧→左侧类型)
                  │                         │                         │
                  ▼                         ▼                         ▼
           触发场景：                 触发场景：                 触发场景：
           · 算术运算                 · 二元 +-*/%              · 赋值 =
           · 比较运算                 · 三元 ?:                · 函数返回值
           · 位运算                   · 算术转换                · 函数参数
           · 函数参数                                              （原型匹配）
           （可变参数）
                  │                         │                         │
                  └────────────┬────────────┘                         │
                               ▼                                      │
                        执行实际运算/赋值                                │
                               │                                      │
                               ▼                                      ▼
                     若赋值给变量 → 赋值转换 ←─────────────────────────┘
                     （结果转换为左侧变量类型，可能截断）
```

---

## 2. 整型提升（Integer Promotion）

### 2.1 规则

当整数类型的**秩（rank）低于 `int`** 时，在以下场景中会被自动提升为 `int`（如果 `int` 能完整表示原类型的所有值）或 `unsigned int`：

|原始类型|提升为|条件|
|---|---|---|
|`char`、`signed char`、`unsigned char`|`int`|`int` 能完整表示原类型（几乎总是成立）|
|`short`、`unsigned short`|`int`|同上|
|`_Bool`|`int`|—|
|位域（bit-field）|`int` 或 `unsigned int`|取决于位域类型和宽度|
|`enum` 类型|`int`|若 `int` 能表示该枚举的完整范围|

> ⚠️ **`char` 的符号性由编译器/平台决定**。无论哪种情况都提升为 `int`，但提升后的**数值**会不同：
> 
> ```c
> char c = 200;
> // x86 GCC 默认 char = signed char：c = -56，提升后 int 值 = -56
> // ARM GCC 默认 char = unsigned char：c = 200，提升后 int 值 = 200
> ```
> 
> 这意味着依赖 `char` 具体数值的代码可能在不同平台上表现不同。需要确定符号性时，应显式使用 `signed char` 或 `unsigned char`。

### 2.2 触发场景

- **算术运算符**：`+`、`-`、`*`、`/`、`%`
- **位运算符**：`<<`、`>>`、`&`、`^`、`|`、`~`
- **比较运算符**：`<`、`<=`、`>`、`>=`、`==`、`!=`
- **条件表达式**：`? :`
- **一元运算符**：`~`（按位取反）
- **可变参数函数**：`printf`、`scanf` 等（`...` 之后的参数）

### 2.3 典型示例

```c
#include <stdio.h>

int main(void) {
    // 示例 1：无符号字符相加
    unsigned char a = 200;
    unsigned char b = 200;

    // a 和 b 都被提升为 int（有符号），值为 200
    // 200 + 200 = 400，在 int 范围内，不会溢出
    unsigned char c = a + b;  // 400 赋给 unsigned char 时截断为 400 % 256 = 144

    printf("a + b = %d\n", a + b);  // 输出: 400（提升为 int 后计算）
    printf("c = %u\n", c);          // 输出: 144（截断后赋值）

    // 示例 2：按位取反的"意外"
    unsigned char x = 255;
    printf("%d\n", ~x);    // 输出: -256（不是 0！）
    // 解释：x 被提升为 int（值为 255 = 0x000000FF）
    //       ~0x000000FF =  0xFFFFFF00 = -256（补码）

    // 示例 3：char 符号性影响提升结果
    signed char sc = -1;
    unsigned char uc = 255;
    printf("signed:   %d\n", sc);  // 输出: -1
    printf("unsigned: %d\n", uc);  // 输出: 255
    // 两者位模式相同（0xFF），但提升后数值截然不同

    return 0;
}
```

---

## 3. 算术转换（Usual Arithmetic Conversion）

### 3.1 规则

当二元运算符的两个操作数类型不同时，编译器会将它们转换为**公共类型**后再运算。转换遵循以下优先级链（从低到高）：

```
bool/char/short → int → unsigned int → long → unsigned long → long long → unsigned long long → float → double → long double
```

**完整决策流程**（按顺序判断，命中即停）：

1. 若任一操作数为 `long double`，另一个转换为 `long double`。
2. 若任一操作数为 `double`，另一个转换为 `double`。
3. 若任一操作数为 `float`，另一个转换为 `float`。
4. **整数情况**（两个操作数都是整数类型）：
    - 先执行**整型提升**（见第 2 节）。
    - 若提升后两类型相同，运算即以此类型进行。
    - 若符号性不同（一个有符号、一个无符号）：
        - 若无符号类型的秩 ≥ 有符号类型的秩，有符号的转换为无符号的。
        - 若有符号类型能表示无符号类型的所有值，无符号的转换为有符号的。
        - 否则，两者都转换为有符号类型对应的无符号类型。
    - 若符号性相同，秩较低的转换为秩较高的。

### 3.2 类型秩（Rank）顺序

```
_Bool < char/signed char/unsigned char < short < int < long < long long
```

同一"家族"中（如 `int` 和 `unsigned int`），秩相同。

### 3.3 典型示例

```c
#include <stdio.h>

int main(void) {
    // 示例 1：整数 + 浮点数 → 整数转换为浮点数
    int a = 5;
    double b = 2.0;
    printf("%f\n", a / b);      // 输出: 2.500000（a 被提升为 double）

    // 示例 2：int + long → int 转换为 long
    int c = 10;
    long d = 20L;
    // c 被转换为 long 后运算

    // 示例 3：unsigned int + long → 取决于平台
    unsigned int e = 4000000000U;
    long f = -1L;
    // 64 位 Linux（long = 8 字节）：long 能表示 unsigned int 的所有值
    //   → unsigned int 转换为 long，结果为 long
    // 32 位 Windows（long = 4 字节）：long 不能表示 unsigned int 的所有值
    //   → 两者都转换为 unsigned long
    printf("%ld\n", e + f);     // 结果依赖平台！

    // 示例 4：int + unsigned int → int 转换为 unsigned int
    int g = -1;
    unsigned int h = 1;
    printf("%u\n", g + h);     // 输出: 0
    // -1 转为 unsigned int = 4294967295，+1 溢出回绕为 0

    return 0;
}
```

### 3.4 有符号与无符号混合运算的陷阱

这是 C 语言中**最常见的类型转换 bug 来源**：

```c
// 陷阱 1：比较时的隐式转换
int a = -1;
unsigned int b = 1;

if (a < b) {
    printf("a < b\n");
} else {
    printf("a >= b\n");   // 输出这个！
}
// 解释：a（-1）被转换为 unsigned int（4294967295），4294967295 > 1

// 陷阱 2：size_t 与 int 的比较
int len = -1;
size_t size = 0;
if (len < size) {
    printf("less\n");
} else {
    printf("greater\n");  // 输出这个！len 被转换为 size_t（一个巨大的正数）
}

// 陷阱 3：循环边界
int n = -1;
for (int i = 0; i < n + 1; i++) {
    // n + 1 = 0，循环不执行——看起来正确
    // 但如果 n = -1 且与 size_t 比较：
    //   (size_t)n = 一个巨大的数，循环会执行到溢出
}
```

> **最佳实践**：尽量避免有符号与无符号类型混合运算。若必须比较，先显式转换到同一类型。

---

## 4. 赋值转换

### 4.1 规则

赋值运算符 `=` 右侧的值会被转换为左侧变量的类型。这种转换**可能导致精度丢失或截断**，编译器通常会发出警告（`-Wconversion`）。

### 4.2 各类赋值转换

#### 浮点 → 整数

```c
double d = 3.99;
int i = d;          // i = 3（截断小数部分，向零取整）

float f = -2.7f;
int j = f;          // j = -2（向零取整，不是四舍五入）

// 四舍五入的正确做法：
int k = (int)(d + 0.5);       // d > 0 时：k = 4
int m = (int)(d + (d > 0 ? 0.5 : -0.5));  // 通用写法
```

> 若浮点值超出目标整数类型的表示范围，结果是**未定义行为**。

#### 整数 → 浮点

```c
int n = 100000001;
float f = n;        // f 可能 = 100000000.0（float 精度不足，丢失了末尾的 1）
double d = n;       // d = 100000001.0（double 精度足够）

// int64_t → float 会丢失大量精度
int64_t big = 9007199254740993LL;  // 2^53 + 1
float fb = (float)big;            // fb = 9007199254740992.0（丢失了 +1）
double db = (double)big;          // db = 9007199254740992.0（double 精度 52 位尾数，也丢失了 +1）
```

#### 大整数 → 小整数

```c
int big = 300;
char c = big;       // c = 44（300 % 256 = 44，截断高位字节）

unsigned short us = 65535;
short s = us;       // s = -1（位模式不变，解释方式改变）
```

#### 宽浮点 → 窄浮点

```c
double d = 3.141592653589793;
float f = d;        // f ≈ 3.1415927（精度从 15~17 位降到 6~7 位）
```

---

## 5. 三元运算符 `?:` 的类型转换

三元运算符 `?:` 的第二和第三个操作数之间也会发生**算术转换**，统一为公共类型后作为整个表达式的结果类型。

```c
#include <stdio.h>

int main(void) {
    // 示例 1：int 与 double → 结果为 double
    int a = 1;
    double b = 2.5;
    double r1 = a > 0 ? a : b;   // a（int）被转换为 double，r1 = 1.0

    // 示例 2：有符号与无符号 → 结果为 unsigned int
    int x = -1;
    unsigned int y = 1;
    unsigned int r2 = x > 0 ? x : y;
    // x 被转换为 unsigned int（4294967295），4294967295 > 1，所以取 x
    // r2 = 4294967295

    // 示例 3：赋给不同类型时的二次转换
    int r3 = x > 0 ? x : y;
    // 三元表达式结果类型为 unsigned int，值为 4294967295
    // 赋给 int 时再次截断，r3 = -1

    printf("r1 = %f\n", r1);     // 输出: 1.000000
    printf("r2 = %u\n", r2);     // 输出: 4294967295
    printf("r3 = %d\n", r3);     // 输出: -1

    return 0;
}
```

> 三元运算符的类型转换规则与二元算术运算符完全一致——先整型提升，再按优先级链统一。

---

## 6. 强制类型转换（Cast）

### 6.1 语法

```c
(目标类型) 表达式
```

### 6.2 典型用途

```c
// 用途 1：整数除法得到浮点结果
int a = 7, b = 2;
double result = (double)a / b;   // 3.5（a 先转为 double，再触发算术转换）
// 若写成 a / b，结果是 3（整数除法）

// 用途 2：显式截断
double pi = 3.14159;
int truncated = (int)pi;         // 3

// 用途 3：消除编译器警告
unsigned char byte = 0xFF;
int value = (int)byte;           // 明确告知编译器"我知道在做什么"

// 用途 4：指针类型转换
int x = 42;
void *p = &x;
int *ip = (int *)p;              // void* → int*（C 标准保证安全）
```

### 6.3 强制转换不改变原变量

```c
int i = 10;
double d = (double)i;   // i 仍然是 int，值为 10
                         // (double)i 产生一个临时的 double 值 10.0
printf("%d\n", i);       // 输出: 10（未改变）
```

### 6.4 指针转换：安全 vs 危险

#### 安全转换（C 标准保证）

```c
// void* ↔ 任何对象指针：完全合法
int x = 42;
void *p = &x;            // int* → void*（甚至可以省略强制转换）
int *ip = (int *)p;      // void* → int*
printf("%d\n", *ip);     // 输出: 42（安全，类型一致）

// malloc 返回值
int *arr = (int *)malloc(10 * sizeof(int));  // void* → int*
```

#### 危险转换（违反严格别名规则）

```c
// 类型双关（type punning）：用一种类型的指针解引用另一种类型的数据
int x = 42;
float *fp = (float *)&x;     // 编译通过，但解引用是未定义行为！
printf("%f\n", *fp);         // ⚠️ 违反严格别名规则，结果不可预测

// 安全的类型双关方式：
int x = 42;
float f;
memcpy(&f, &x, sizeof(f));   // 通过 memcpy 安全地重新解释位模式
printf("%f\n", f);           // 安全
```

> **严格别名规则（Strict Aliasing）**：C 标准规定，通过某种类型的指针访问另一种类型的数据是**未定义行为**，唯一的例外是 `char*`（可以用 `char*` 访问任何类型）。

---

## 7. 函数参数传递时的转换

### 7.1 默认参数提升（Default Argument Promotion）

调用**可变参数函数**（如 `printf`、`scanf`）时，`...` 之后的参数会自动提升：

|原始类型|提升为|
|---|---|
|`float`|`double`|
|`char`、`short`、`unsigned char`、`unsigned short`、`_Bool`|`int`（或 `unsigned int`）|

```c
// 这就是为什么 printf 中 %f 可以接受 float
float f = 3.14f;
printf("%f\n", f);   // f 被提升为 double，%f 接受 double，正确
```

### 7.2 原型匹配转换

调用有**函数原型声明**的函数时，实参会转换为形参的声明类型：

```c
void func(double x);

int main(void) {
    func(5);       // 5（int）被转换为 5.0（double）
    func(3.14f);   // 3.14f（float）被转换为 3.14（double）
    return 0;
}
```

### 7.3 `printf`/`scanf` 格式符与类型不匹配

这是实际编程中**最高频的类型转换 bug**。`printf` 是可变参数函数，编译器**无法检查**格式符与实参类型是否匹配，不匹配时会导致**未定义行为**：

```c
#include <stdio.h>
#include <string.h>

int main(void) {
    // 陷阱 1：size_t 用 %d 输出
    size_t len = strlen("hello");
    printf("%d\n", len);
    // ⚠️ 64 位系统上 size_t 是 8 字节，%d 期望 4 字节的 int
    //    结果：输出错误值，甚至崩溃

    // 陷阱 2：int 用 %f 输出
    int x = 42;
    printf("%f\n", x);
    // ⚠️ %f 期望 double（8 字节），但传入的是 int（4 字节）
    //    结果：输出 0.000000 或垃圾值（未定义行为）

    // 陷阱 3：long 用 %d 输出
    long l = 42L;
    printf("%d\n", l);
    // ⚠️ 64 位系统上 long 是 8 字节，%d 期望 4 字节
    //    结果：输出错误值

    // 陷阱 4：unsigned 用 %d 输出
    unsigned u = 4000000000U;
    printf("%d\n", u);
    // ⚠️ 输出: -294967296（4000000000 的补码被解释为有符号数）

    // ===== 正确写法 =====
    printf("%zu\n", len);     // %zu 对应 size_t
    printf("%d\n", x);        // %d 对应 int
    printf("%ld\n", l);       // %ld 对应 long
    printf("%u\n", u);        // %u 对应 unsigned

    return 0;
}
```

**格式符与类型对照速查**：

|类型|`printf` 格式符|`scanf` 格式符|
|---|---|---|
|`int`|`%d` 或 `%i`|`%d` 或 `%i`|
|`unsigned int`|`%u`|`%u`|
|`long`|`%ld`|`%ld`|
|`unsigned long`|`%lu`|`%lu`|
|`long long`|`%lld`|`%lld`|
|`unsigned long long`|`%llu`|`%llu`|
|`size_t`|`%zu`|`%zu`|
|`ptrdiff_t`|`%td`|`%td`|
|`float`（实际传 `double`）|`%f`|`%f`（注意传 `&f`）|
|`double`|`%f` 或 `%lf`|`%lf`|
|`char`|`%c`|`%c`|
|`char *`|`%s`|`%s`|
|指针|`%p`|`%p`|

> **编译器保护**：GCC/Clang 的 `-Wall` 或 `-Wformat` 选项会检查 `printf`/`scanf` 的格式符匹配，**务必开启**。

---

## 8. 常见陷阱与最佳实践

### 8.1 整数除法陷阱

```c
// 错误：两个整数相除，结果为整数
double ratio = 1 / 3;          // ratio = 0.0（1/3 = 0，再转为 double）

// 正确：至少一个操作数为浮点数
double ratio = 1.0 / 3;        // ratio = 0.333333...
double ratio = (double)1 / 3;  // ratio = 0.333333...
```

### 8.2 有符号溢出 vs 无符号溢出

```c
// 无符号溢出：定义良好的回绕行为
unsigned char u = 255;
u = u + 1;    // u = 0（256 % 256 = 0），完全合法

// 有符号溢出：未定义行为！
signed char s = 127;
s += 1;       // ⚠️ 未定义行为！
// 注意：s + 1 中 s 先被整型提升为 int（值为 127），127 + 1 = 128 在 int 范围内不会溢出
// 但赋值回 signed char 时，128 超出 signed char 范围（-128~127），触发未定义行为

// 安全写法：先检查再运算
if (s < 127) { s += 1; }
```

### 8.3 `size_t` 与 `int` 混用

```c
int arr[10] = {0};
int len = -1;

// 危险！len 被转换为 size_t（一个巨大的正数）
for (int i = 0; i < (int)(sizeof(arr) / sizeof(arr[0])) + len; i++) {
    // 可能永远不会执行，也可能越界
}

// 安全写法：统一类型
size_t size = sizeof(arr) / sizeof(arr[0]);
for (size_t i = 0; i < size; i++) { ... }
```

### 8.4 取模运算的符号问题

```c
// C99 规定：整数除法向零截断，取模结果的符号与被除数相同
printf("%d\n",  7 %  3);   //  1
printf("%d\n", -7 %  3);   // -1（不是 2！）
printf("%d\n",  7 % -3);   //  1
printf("%d\n", -7 % -3);   // -1

// C99 之前，取模行为是实现定义的，不同编译器可能给出不同结果
```

### 8.5 浮点比较陷阱

```c
double a = 0.1 + 0.2;
double b = 0.3;

if (a == b) {
    printf("相等\n");
} else {
    printf("不相等\n");  // 输出这个！0.1 + 0.2 = 0.30000000000000004
}

// 正确做法：使用 epsilon 比较
#include <math.h>
if (fabs(a - b) < 1e-9) {
    printf("近似相等\n");
}
```

---

## 9. 转换速查表

### 9.1 安全转换（不丢失信息）

| 源类型                              | 目标类型                     | 说明                                 |
| -------------------------------- | ------------------------ | ---------------------------------- |
| `char`/`short`                   | `int`、`long`、`long long` | 整型提升，值不变                           |
| `unsigned char`/`unsigned short` | `int`、`long`、`long long` | 整型提升，值不变                           |
| `int`                            | `long`、`long long`       | 秩提升，值不变                            |
| `float`                          | `double`、`long double`   | 精度提升，值不变（或更精确）                     |
| 小范围整数                            | 足够大的浮点类型                 | 通常安全，但 `int64_t` → `float` 会丢失大量精度 |

### 9.2 危险转换（可能丢失信息）

|源类型|目标类型|风险|示例|
|---|---|---|---|
|浮点 → 整数|小数部分被截断|`3.99` → `3`||
|大整数 → 小整数|高位字节被截断|`300` → `char` = `44`||
|有符号 → 无符号|负数变为大正数|`-1` → `unsigned int` = `4294967295`||
|无符号 → 有符号|大正数可能变为负数|`65535` → `short` = `-1`||
|`double` → `float`|精度降低|15~17 位 → 6~7 位||
|`long double` → `double`|精度降低|平台相关||
|`int64_t` → `float`|精度严重丢失|float 仅 6~7 位有效数字||

### 9.3 转换规则口诀

> **整型提升先行，算术转换随后；**  
> **赋值截断无声，强制转换自负。**

---

## 10. 编译器警告选项

|选项|作用|
|---|---|
|`-Wall`|开启常见警告（包含部分转换警告和 `printf` 格式检查）|
|`-Wconversion`|警告所有隐式转换可能导致值改变的情况|
|`-Wsign-conversion`|警告有符号/无符号之间的隐式转换|
|`-Wfloat-conversion`|警告浮点到整数的隐式转换|
|`-Wformat`|检查 `printf`/`scanf` 格式符与参数类型是否匹配|
|`-Wstrict-prototypes`|警告函数参数类型不匹配|

```bash
# 推荐的编译命令
gcc -Wall -Wconversion -Wsign-conversion -Wformat=2 -Werror=conversion main.c -o main
```

---

## 11. 实战综合案例

```c
#include <stdio.h>
#include <string.h>
#include <stdint.h>

int main(void) {
    // === 案例 1：整型提升的连锁效应 ===
    uint8_t a = 200, b = 100;
    uint8_t sum = a + b;  // a、b 提升为 int → 300 → 截断为 44
    printf("sum = %u (期望 44)\n", sum);

    // === 案例 2：有符号/无符号比较 ===
    int offset = -1;
    size_t length = 100;
    // if (offset < length)  // ⚠️ offset 被转为 size_t，变成巨大正数！
    if (offset < 0 || (size_t)offset < length) {  // 安全写法
        printf("offset 合法\n");
    }

    // === 案例 3：整数除法陷阱 ===
    int total = 7, count = 2;
    // double avg = total / count;       // avg = 3.0（错误！）
    double avg = (double)total / count;  // avg = 3.5（正确）
    printf("avg = %.1f\n", avg);

    // === 案例 4：printf 类型匹配 ===
    size_t len = strlen("Hello");
    long big = 1234567890L;
    printf("len = %zu, big = %ld\n", len, big);  // 正确

    // === 案例 5：位运算中的提升 ===
    uint8_t flags = 0x01;
    uint8_t mask = ~flags;  // flags 提升为 int → ~0x00000001 = 0xFFFFFFFE
                             // 截断回 uint8_t → 0xFE = 254
    printf("mask = %u (期望 254)\n", mask);

    return 0;
}
```
