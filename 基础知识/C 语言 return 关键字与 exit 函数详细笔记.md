### return 关键字

---

#### 1. return 基本使用

- **语法**：`return expression;`
- **作用**：将表达式的值返回给调用函数，并立即退出当前函数。

```c
int add(int a, int b) {
    return a + b;
}
```

---

#### 2. 无返回值的使用（void 函数）

- 如果函数返回类型是 `void`，可以直接写 `return;` 用于提前退出函数，也可以省略不写（函数末尾自动返回）。

```c
void printHello() {
    printf("Hello\n");
    return; // 可省略，此处用于提前退出
}
```

---

#### 3. return 中的表达式求值

- `return` 后面的表达式会先被求值，然后返回结果。

```c
int square(int x) {
    return x * x; // 先计算 x * x，再返回结果
}
```

---

#### 4. 函数返回类型与 return 值类型的关系

- 函数的返回值类型**应与** return 语句返回的值类型一致，否则会发生**隐式类型转换**，可能导致精度丢失并产生编译警告。

```c
double getDouble() {
    return 5.5; // 正确，类型匹配
}

int getInt() {
    return 5.5; // 编译警告：double 隐式转换为 int，截断为 5
}
```

---

#### 5. 返回指针

- `return` 可以用于返回指针类型，但必须保证指针指向的内存在函数返回后仍然有效。
- **不能返回局部变量（auto）的地址**，因为函数返回后局部变量立即失效。
- C 语言**不能直接返回数组类型**，若需返回数组，可将数组包装在结构体中返回，或返回指向数组的指针（需确保内存有效）。

```c
int* getPointer() {
    static int value = 10;
    return &value; // 返回静态变量的地址是安全的（静态变量生命周期为整个程序）
}

// 错误示例：返回局部变量的地址
int* badPointer() {
    int local = 10;
    return &local; // 危险！函数返回后 local 已失效，指针成为悬空指针
}
```

---

#### 6. 返回结构体

- C 语言**允许直接返回结构体**（按值拷贝），这是与数组的重要区别。
- 如果结构体较大，按值拷贝会有性能开销，此时可以考虑返回结构体指针（配合 `malloc` 或 `static`）。

```c
typedef struct {
    int x;
    int y;
} Point;

// 返回结构体（按值拷贝）
Point createPoint(int x, int y) {
    Point p = {x, y};
    return p; // 合法：C 语言支持返回结构体
}

// 返回结构体指针（避免大结构体的拷贝开销）
Point* createPointDynamic(int x, int y) {
    Point* p = (Point*)malloc(sizeof(Point));
    p->x = x;
    p->y = y;
    return p; // 调用者需要负责 free()
}
```

---

#### 7. 多次 return 与执行路径

- `return` 语句一旦执行，就会**立即退出函数**，不会执行后续代码。
- 函数中可以有多个 `return`，但应确保每种执行路径都能到达 `return`，否则可能导致未定义行为。

```c
int findMax(int a, int b) {
    if (a > b)
        return a;
    else
        return b;
}
```

---

#### 8. return 在 switch-case 中的使用

- `return` 可以出现在 `switch-case` 内部，根据不同条件返回不同的值，这在实际开发中非常常见。
- 由于 `return` 会立即退出函数，因此 `case` 中不需要 `break`。

```c
const char* getDayName(int day) {
    switch (day) {
        case 1: return "Monday";
        case 2: return "Tuesday";
        case 3: return "Wednesday";
        case 4: return "Thursday";
        case 5: return "Friday";
        case 6: return "Saturday";
        case 7: return "Sunday";
        default: return "Unknown";
    }
}
```

---

#### 9. return 与三目运算符结合

- 三目运算符 `? :` 常与 `return` 配合使用，简化条件返回的逻辑，等价于 `if-else` 两个 `return`。

```c
// 使用三目运算符
int abs(int x) {
    return x >= 0 ? x : -x;
}

// 等价于 if-else 写法
int abs_if(int x) {
    if (x >= 0)
        return x;
    else
        return -x;
}
```

---

#### 10. return 中的逗号运算符与副作用

- 在 `return` 语句中可以使用**逗号运算符** `,`：逗号运算符会从左到右依次求值，整个表达式的值为**最右边表达式的值**。

```c
int getNumber() {
    return printf("Hello"), 5;
    // 先执行 printf("Hello")（打印 "Hello"）
    // 然后整个逗号表达式的值为 5，返回 5
}
```

- **注意**：在 `return` 中调用有副作用的函数（如 `printf`、`malloc` 等）虽然合法，但会降低代码可读性，不推荐在实际项目中使用。

---

#### 11. return 在递归中的使用

- 在递归函数中，`return` 用于返回递归调用的结果，逐层传递直到最外层。

```c
int factorial(int n) {
    if (n <= 1)
        return 1;        // 递归终止条件
    else
        return n * factorial(n - 1); // 返回递归调用的结果
}
```

---

#### 12. 缺少 return 的情况

- 如果函数的返回类型不是 `void`，而在某些执行路径上没有 `return` 语句，调用者读取返回值时将导致**未定义行为**（可能得到垃圾值）。

```c
int getValue(int x) {
    if (x > 0)
        return x;
    // 当 x <= 0 时没有 return，调用者得到的是未定义的值
}
```

- 编译器通常会对此发出警告，但不会阻止编译。建议开启 `-Wall` 编译选项以捕获此类问题。

---

#### 13. return 在 main 函数与非 main 函数中的语义

- **在 main 函数中**：`return` 的返回值作为程序的**退出状态码**传递给操作系统。
    - `return 0`：表示程序正常结束
    - `return 非零值`：表示程序异常结束（具体含义由程序定义）
- **在非 main 函数中**：`return` 的返回值仅代表函数的计算结果或状态码，具体含义由函数设计者定义，与程序退出状态无关。

```c
// main 函数中：返回值是程序退出状态码
int main() {
    int result = divide(10, 0);
    if (result == -1)
        return 1; // 程序异常退出
    return 0;     // 程序正常退出
}

// 非 main 函数中：返回值是函数的计算结果
int divide(int a, int b) {
    if (b == 0)
        return -1; // 表示除法失败（自定义约定）
    return a / b;
}
```

---

#### 14. return 与 bool 类型（C99 起）

C99 标准引入了 `<stdbool.h>` 头文件，提供了 `bool`、`true`、`false` 三个宏：

- `bool` 等价于 `_Bool`（C99 内置的布尔类型）
- `true` 等价于 `1`
- `false` 等价于 `0`

在函数中使用 `return` 返回 `bool` 值时，通常用于表示**成功（true）或失败（false）**的判断结果。

```c
#include <stdio.h>
#include <stdbool.h>

// 判断一个数是否为偶数
bool isEven(int n) {
    return n % 2 == 0; // 条件成立返回 true(1)，否则返回 false(0)
}

// 检查除数是否合法
bool isValidDivisor(int divisor) {
    if (divisor == 0)
        return false; // 除数为 0，返回失败
    return true;      // 除数合法，返回成功
}

int main() {
    if (isEven(4)) {
        printf("4 是偶数\n");
    }

    if (!isValidDivisor(0)) {
        printf("除数不能为 0\n");
    }

    // bool 本质是整数，可以直接打印
    printf("isEven(4) = %d\n", isEven(4));   // 输出 1
    printf("isEven(3) = %d\n", isEven(3));   // 输出 0

    return 0;
}
```

**注意事项**：

- **本质是整数**：`bool` 底层是 `_Bool` 类型（本质上是一个无符号整数），`true` 就是 `1`，`false` 就是 `0`。任何非零值赋给 `bool` 变量时都会被转换为 `1`。

```c
bool a = 100;  // a 的值为 1（非零值被截断为 1）
bool b = -1;   // b 的值为 1
bool c = 0;    // c 的值为 0
```

- **C99 之前没有 bool 类型**：在 C89/C90 标准中，通常用 `int` 返回 `1`（成功）或 `0`（失败），或者自定义宏：

```c
// C99 之前的常见做法
#define TRUE  1
#define FALSE 0

int isPositive(int n) {
    return n > 0 ? TRUE : FALSE;
}
```

- **与标准库函数的区别**：许多 C 标准库函数（如 `isalpha()`、`isdigit()` 等）返回 `int` 而非 `bool`，其中非零值表示"真"，`0` 表示"假"。这是因为这些函数在 C99 之前就已定义，早于 `bool` 类型的引入。

---

### exit() 函数

---

#### 15. 基本概念与函数原型

- `exit()` 是 C 标准库函数，用于**从程序任何位置**终止程序执行。
- 头文件：`<stdlib.h>`

```c
void exit(int status);
```

---

#### 16. 参数含义

- `status`：整数，表示程序退出的状态码。
    - `0` 或 `EXIT_SUCCESS`：表示程序正常退出
    - 非 0 值或 `EXIT_FAILURE`：表示程序异常退出

---

#### 17. exit() 的执行流程

调用 `exit()` 后，会按以下顺序执行清理操作：

1. **调用 atexit() 注册的函数**（按注册的逆序，即后注册的先调用）
2. **刷新并关闭所有打开的文件流**（如 `stdout`、`stderr` 等）
3. **删除由 `tmpfile()` 创建的临时文件**
4. **终止进程**，将状态码返回给操作系统

```c
#include <stdio.h>
#include <stdlib.h>

int main() {
    printf("程序即将退出...\n");
    exit(0); // 正常退出，会执行上述清理操作
    printf("这行代码不会被执行\n");
}
```

---

#### 18. exit() 与 return 在 main 函数中的对比

在 main 函数中，`return` 和 `exit()` 的效果**几乎相同**——两者都会调用 atexit() 注册的函数、刷新并关闭所有文件流。

|对比项|`return` (在 main 中)|`exit()`|
|---|---|---|
|调用 atexit 注册的函数|✅|✅|
|刷新/关闭文件流|✅|✅|
|调用位置|仅限函数末尾或提前退出|程序任何位置|
|执行机制|由运行时启动代码自动调用 exit|直接触发终止流程|
|适用场景|main 函数正常/提前退出|深层嵌套函数中需要立即终止程序|

---

#### 19. exit()、_Exit()、_exit() 三者对比

|函数|头文件|atexit 注册的函数|文件流关闭/缓冲刷新|临时文件删除|
|---|---|---|---|---|
|`exit()`|`<stdlib.h>`|✅ 调用|✅|✅|
|`_Exit()` (C99)|`<stdlib.h>`|❌ 不调用|✅|❌|
|`_exit()` (POSIX)|`<unistd.h>`|❌ 不调用|❌|❌|

- `_Exit()` 是 C99 标准函数，跳过 atexit 和信号处理，但会关闭文件流。
- `_exit()` 是 POSIX 系统调用，**不做任何清理**，直接终止进程，适合在 `fork()` 后的子进程中使用。

```c
#include <stdlib.h>

int main() {
    // 需要完整清理：使用 exit()
    exit(0);

    // 跳过 atexit，但关闭文件流：使用 _Exit()
    _Exit(0);
}
```

---

#### 20. atexit() 注册退出处理函数

- `atexit()` 用于注册在 `exit()` 被调用时（或 main 函数 return 时）自动执行的清理函数。
- 注册顺序为栈结构（LIFO），后注册的先执行。
- 最多可注册 `ATEXIT_MAX` 个函数（具体值由实现定义，通常至少 32 个）。
- **限制**：注册的函数不能带参数；注册的函数内部**不能再调用 `exit()`**，否则会导致无限递归（未定义行为）。

```c
#include <stdio.h>
#include <stdlib.h>

void cleanup1() {
    printf("清理工作 1\n");
    // exit(0); // 错误！会导致无限递归
}

void cleanup2() {
    printf("清理工作 2\n");
}

int main() {
    atexit(cleanup1); // 先注册
    atexit(cleanup2); // 后注册

    printf("程序运行中...\n");
    return 0;
    // 输出顺序：
    // 程序运行中...
    // 清理工作 2    （后注册的先执行）
    // 清理工作 1    （先注册的后执行）
}
```

---

#### 21. abort() 函数——异常终止

- `abort()` 用于**异常终止**程序，与 `exit()` 的关键区别：
    - **不调用** atexit() 注册的函数
    - **不刷新**标准 I/O 缓冲区（未刷新的输出会丢失）
    - 通常会生成**核心转储（core dump）**，便于调试
    - 触发 `SIGABRT` 信号
- 适用场景：检测到不可恢复的严重错误（如断言失败）。

```c
#include <stdio.h>
#include <stdlib.h>

int main() {
    printf("即将异常终止"); // 注意：没有 \n，可能不会输出（缓冲区未刷新）
    abort(); // 异常终止，生成 core dump
}
```

---

#### 22. exit() 在信号处理函数中的安全性

- 在信号处理函数（signal handler）中调用 `exit()` 是**不安全的**，因为 `exit()` 不是**异步信号安全（async-signal-safe）**的函数。
- 在信号处理函数中应使用 `_exit()` 或 `_Exit()` 代替。

```c
#include <signal.h>
#include <stdlib.h>
#include <unistd.h>

void signal_handler(int sig) {
    // exit(0);   // 不安全！exit() 不是异步信号安全的
    _exit(0);     // 安全：直接终止进程，不做清理
}

int main() {
    signal(SIGINT, signal_handler); // 注册 Ctrl+C 信号处理
    while (1) {} // 等待信号
    return 0;
}
```

---

#### 23. EXIT_SUCCESS 与 EXIT_FAILURE 宏

- 定义在 `<stdlib.h>` 中，用于提高代码可移植性：
    - `EXIT_SUCCESS`：表示程序正常退出（通常为 0）
    - `EXIT_FAILURE`：表示程序异常退出（通常为 1，但具体值因系统而异）

```c
#include <stdio.h>
#include <stdlib.h>

int main() {
    int error_condition = 1;

    if (error_condition) {
        printf("发生错误，程序将退出。\n");
        exit(EXIT_FAILURE); // 因失败而退出
    }

    printf("程序继续执行...\n");
    return EXIT_SUCCESS;
}
```

---

### 总结对比表

|终止方式|头文件|atexit 函数|文件流清理|缓冲刷新|核心转储|适用场景|
|---|---|---|---|---|---|---|
|`return` (main)|—|✅|✅|✅|❌|main 函数正常/提前退出|
|`exit()`|`<stdlib.h>`|✅|✅|✅|❌|任意位置正常终止|
|`_Exit()`|`<stdlib.h>`|❌|✅|✅|❌|跳过 atexit 的快速终止|
|`_exit()`|`<unistd.h>`|❌|❌|❌|❌|fork 后子进程/信号处理|
|`abort()`|`<stdlib.h>`|❌|❌|❌|✅|严重错误，需调试|
