
#### 1. 自动存储类别（auto）

- **关键字**：auto（默认可省略，通常不显式书写）
- **存储期**：自动（函数调用时分配内存，函数返回时释放）
- **作用域**：块作用域（仅在声明它的代码块 `{}` 内可见）
- **链接**：无链接（仅当前块内有效，无法被其他文件/函数访问）
- **声明方式**：在代码块内声明，如 `int a;` 或 `auto int a;`
- **适用对象**：局部变量（函数内的临时变量）
- **生效范围**：当前代码块 `{}`
- **注意事项**：C11 起 `auto` 被重新用于类型推断（如 `auto x = 10;` 等价于 `int x = 10;`），与旧标准中作为存储类别说明符的含义不同。若在 C11 及以后标准中使用，需注意这一变化。

**程序用例**：

```c
#include <stdio.h>

void func() {
    auto int x = 10; // 自动存储类别，函数调用时分配，返回时释放
    printf("x = %d\n", x);
}

int main() {
    func(); // 输出 x = 10
    // x 在此处不可见（作用域为func函数的代码块）
    return 0;
}
```

---

#### 2. 寄存器存储类别（register）

- **关键字**：register
- **存储期**：自动（同 auto，函数调用时分配，返回时释放）
- **作用域**：块作用域
- **链接**：无链接
- **声明方式**：在代码块内用 register 声明，如 `register int i;`
- **适用对象**：局部变量（建议编译器存入 CPU 寄存器，提高访问速度，适用于循环计数器等高频访问变量）
- **生效范围**：当前代码块
- **注意事项**：register 变量不能取地址（`&` 操作符），编译器可能忽略 register 请求（仍存入内存）。

**程序用例**：

```c
#include <stdio.h>

void loop() {
    register int i;
    for (i = 0; i < 5; i++) {
        printf("i = %d\n", i);
    }
}

int main() {
    loop(); // 输出 0 1 2 3 4
    return 0;
}
```

---

#### 3. 静态存储类别（static）

静态存储类别分为内部链接（文件作用域）和无链接（块作用域）两种：

##### 3.1 静态（内部链接，文件作用域）

- **关键字**：static（在函数外声明）
- **存储期**：静态（程序启动时分配存储空间，程序结束时释放）
- **作用域**：文件作用域（全局可见，但仅当前源文件内有效）
- **链接**：内部链接（仅当前源文件可见，其他文件无法访问）
- **声明方式**：在所有函数外用 static 声明，如 `static int global_var;` 或 `static void func();`
- **适用对象**：全局变量、函数（限制作用域，避免命名冲突）
- **生效范围**：当前源文件

**程序用例**：

```c
// file1.c
#include <stdio.h>

static int file1_var = 100; // 仅file1.c可见的全局变量

static void file1_func() {
    printf("file1_func: file1_var = %d\n", file1_var);
}

void call_file1_func() {
    file1_func(); // 调用file1.c内的静态函数
}
```

```c
// file2.c
#include <stdio.h>

extern void call_file1_func(); // 声明file1.c的函数

int main() {
    call_file1_func(); // 输出 file1_func: file1_var = 100
    // 以下代码会报错（file1_var是static，file2.c无法访问）
    // printf("file2: file1_var = %d\n", file1_var);
    return 0;
}
```

##### 3.2 静态（无链接，块作用域）

- **关键字**：static（在代码块内声明）
- **存储期**：静态（程序启动时分配存储空间，程序结束时释放；但**初始化在程序执行流首次到达该声明时进行，且仅初始化一次**）
- **作用域**：块作用域（仅在声明它的代码块 `{}` 内可见）
- **链接**：无链接（仅当前块内有效）
- **声明方式**：在代码块内用 static 声明，如 `static int local_var;`
- **适用对象**：局部变量（函数内的变量，生命周期为整个程序，多次调用函数时保留值）
- **生效范围**：当前代码块（但生命周期是静态的，即程序运行期间一直存在）

**程序用例**：

```c
#include <stdio.h>

void counter() {
    static int count = 0; // 静态局部变量，首次调用时初始化，后续调用保留值
    count++;
    printf("count = %d\n", count);
}

int main() {
    counter(); // 输出 count = 1
    counter(); // 输出 count = 2
    counter(); // 输出 count = 3
    return 0;
}
```

---

#### 4. 外部存储类别（extern）

- **关键字**：extern
- **存储期**：静态（程序启动时分配，程序结束时释放）
- **作用域**：文件作用域（extern 声明本身的作用域是从声明处到文件末尾；但由于其引用的是具有**外部链接**的实体，因此该实体可被其他翻译单元通过 extern 声明访问）
- **链接**：外部链接（所有源文件可见）
- **声明方式**：在函数外用 extern 声明，如 `extern int global_var;`（用于引用其他文件定义的全局变量/函数）
- **适用对象**：全局变量、函数（跨文件共享）
- **生效范围**：所有源文件（通过外部链接实现跨文件访问）

**程序用例**：

```c
// file1.c
#include <stdio.h>

int shared_var = 50; // 全局变量，可在其他文件访问

void shared_func() {
    printf("shared_func: shared_var = %d\n", shared_var);
}
```

```c
// file2.c
#include <stdio.h>

extern int shared_var;          // 声明file1.c的全局变量
extern void shared_func();      // 声明file1.c的函数

int main() {
    shared_var = 100; // 修改全局变量
    shared_func();    // 输出 shared_func: shared_var = 100
    return 0;
}
```

---

#### 5. 函数参数作用域与 goto 语句

- **作用域**：函数参数的作用域是整个函数体，包括所有代码块和标签。
- **goto 语句**：goto 用于在函数内部进行无条件跳转，可以跳转到函数体内的任何标签（`label:`），但**不能跳过带有初始化器的变量声明进入该变量的作用域**。
- **适用对象**：函数参数（在函数调用时初始化，作用域覆盖整个函数体）。
- **生效范围**：函数体内（从函数入口到函数出口）。
- **注意事项**：
    - 函数参数在函数调用时已初始化完毕，goto 不可能跳过其初始化，跳转后仍可正常访问。
    - goto 不能跳过带有初始化器的变量声明（进入其作用域），例如：

```c
goto label;
int x = 10; // 错误：goto 跳过了 x 的初始化
label:
```

**程序用例**：

```c
#include <stdio.h>

void demo_goto(int param) {
    printf("函数开始，param = %d\n", param); // 使用函数参数

    if (param > 0) {
        goto positive_label; // 跳转到positive_label标签
    }

    printf("param <= 0，继续执行...\n");
    return;

positive_label:
    printf("param > 0，跳转到此处！\n");
    printf("param = %d（仍可访问函数参数）\n", param);
}

int main() {
    demo_goto(5);  // 输出：函数开始，param = 5；param > 0，跳转到此处！；param = 5（仍可访问函数参数）
    demo_goto(-3); // 输出：函数开始，param = -3；param <= 0，继续执行...
    return 0;
}
```

---

#### 6. 类型别名（typedef）—— 补充说明

typedef 不属于存储类别，但用于定义类型别名，简化代码：

```c
typedef int INT; // 定义INT为int的别名
INT a = 10;      // 等价于 int a = 10;
```

---

#### 7. 线程存储期（_Thread_local，C11 新增）—— 补充说明

C11 引入了第四种存储期——**线程存储期（thread storage duration）**，使用 `_Thread_local`（或 `<threads.h>` 中的 `thread_local` 宏）声明。

- **关键字**：`_Thread_local` / `thread_local`
- **存储期**：线程（每个线程拥有独立的变量副本，线程启动时创建，线程结束时销毁）
- **作用域**：取决于声明位置（文件作用域或块作用域）
- **链接**：可与 `static`（内部链接）或 `extern`（外部链接）结合使用
- **适用对象**：多线程程序中需要每个线程独立持有的变量

```c
#include <stdio.h>

_Thread_local int thread_var = 0; // 每个线程拥有独立的 thread_var 副本
```

---

### 存储类别总结表格

|存储类别|关键字|存储期|作用域|链接|适用对象|典型场景|
|---|---|---|---|---|---|---|
|自动（auto）|auto|自动|块作用域|无链接|局部变量|临时局部变量|
|寄存器（register）|register|自动|块作用域|无链接|局部变量（高频访问）|循环计数器等|
|静态（内部链接）|static|静态|文件作用域|内部链接|全局变量、函数|限制作用域，避免命名冲突|
|静态（无链接）|static|静态|块作用域|无链接|局部变量|函数内需保留值的变量（如计数器）|
|外部（extern）|extern|静态|文件作用域|外部链接|全局变量、函数|跨文件共享|
|函数参数|无|自动|函数体|无链接|函数参数|函数调用时传递的参数|
|线程（C11）|_Thread_local|线程|取决于声明位置|可配合static/extern|多线程变量|每个线程独立持有的变量|
