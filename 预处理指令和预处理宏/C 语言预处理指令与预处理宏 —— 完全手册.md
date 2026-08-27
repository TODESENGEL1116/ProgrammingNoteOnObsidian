## 一、预处理阶段概述

C 语言源代码在编译之前，会先经过 **预处理器（Preprocessor）** 处理。预处理器根据以 `#` 开头的特殊指令完成文本替换、文件包含、条件编译等工作，处理后的结果再交给编译器。

```
源文件(.c) ──→ 预处理器 ──→ 展开后的文本 ──→ 编译器 ──→ 目标文件(.o/.obj) ──→ 链接器 ──→ 可执行文件
```

---

## 二、全部预处理指令一览

| 指令          | 引入标准              | 功能简述                          |
| ----------- | ----------------- | ----------------------------- |
| `#include`  | C89               | 包含头文件                         |
| `#define`   | C89               | 定义宏（对象宏/函数宏）                  |
| `#undef`    | C89               | 取消宏定义                         |
| `#if`       | C89               | 条件编译（表达式求值）                   |
| `#ifdef`    | C89               | 条件编译（宏是否定义）                   |
| `#ifndef`   | C89               | 条件编译（宏是否未定义）                  |
| `#elif`     | C89               | else-if 条件分支                  |
| `#elifdef`  | **C23**           | else-if-def 条件分支              |
| `#elifndef` | **C23**           | else-if-ndef 条件分支             |
| `#else`     | C89               | 条件编译的 else 分支                 |
| `#endif`    | C89               | 结束条件编译                        |
| `#error`    | C89               | 编译期报错，中止编译                    |
| `#warning`  | 非标准（GCC/Clang 扩展） | 编译期警告，不中止编译                   |
| `#pragma`   | C89               | 编译器特定指令                       |
| `#line`     | C89               | 修改 `__LINE__` / `__FILE__` 的值 |
| `#` (空指令)   | C89               | 空预处理指令，什么都不做                  |
| `_Pragma`   | C99               | `#pragma` 的运算符形式，可在宏中使用       |

---

## 三、各指令详解与完整代码示例

---

### 3.1 `#include` — 文件包含

```c
// 形式1：在系统标准目录中搜索
#include <stdio.h>

// 形式2：先在当前目录搜索，再到系统目录搜索
#include "myheader.h"

// 形式3：宏展开后的文件名（较少使用）
#define MY_HEADER "config.h"
#include MY_HEADER
```

**要点：**

- `< >` 用于标准库头文件
- `" "` 用于项目自定义头文件
- 可以嵌套包含（头文件中再 include 其他头文件）

---

### 3.2 `#define` — 宏定义

#### 3.2.1 对象宏（Object-like Macro）

```c
#define PI 3.14159265358979
#define MAX_BUFFER_SIZE 1024
#define VERSION "1.0.0"
#define NEWLINE '\n'

// 空宏 —— 用作标记/标志位
#define USE_FEATURE_X

// 多行宏（用反斜杠续行）
#define LONG_CONSTANT \
    "This is a very long string " \
    "that spans multiple lines."
```

#### 3.2.2 函数宏（Function-like Macro）

```c
// 基本函数宏
#define SQUARE(x)      ((x) * (x))
#define MAX(a, b)      ((a) > (b) ? (a) : (b))
#define MIN(a, b)      ((a) < (b) ? (a) : (b))
#define ABS(x)         ((x) < 0 ? -(x) : (x))
#define SWAP(a, b, T)  do { T tmp = (a); (a) = (b); (b) = tmp; } while(0)

// 使用示例
int main(void) {
    int a = 5, b = 10;
    printf("MAX = %d\n", MAX(a, b));    // 10
    printf("SQUARE(3+1) = %d\n", SQUARE(3+1));  // 16 (注意括号的重要性!)
    
    SWAP(a, b, int);
    printf("a=%d, b=%d\n", a, b);       // a=10, b=5
    return 0;
}
```

> ⚠️ **宏的陷阱**：宏只是文本替换，`SQUARE(3+1)` 若定义为 `(x)*(x)` 则变成 `3+1*3+1=7`，所以参数和整体都要加括号。

#### 3.2.3 `#` 字符串化运算符（Stringification）

```c
#define TO_STRING(x)    #x
#define PRINT_VAR(var)  printf(#var " = %d\n", var)
#define LOG(msg)        printf("[LOG] " #msg "\n")

int main(void) {
    printf("%s\n", TO_STRING(Hello World));   // 输出: Hello World
    int count = 42;
    PRINT_VAR(count);                          // 输出: count = 42
    LOG(Server started);                       // 输出: [LOG] Server started
    return 0;
}
```

`#` 将宏参数转化为字符串字面量。

#### 3.2.4 `##` 令牌粘贴运算符（Token Pasting）

```c
#define CONCAT(a, b)       a##b
#define MAKE_VAR(name, n)  name##n
#define CLASS_FUNC(cls, fn) cls##_##fn

int main(void) {
    int MAKE_VAR(my, 1) = 100;    // 等价于: int my1 = 100;
    int MAKE_VAR(my, 2) = 200;    // 等价于: int my2 = 200;
    
    printf("my1 = %d\n", my1);    // 100
    printf("my2 = %d\n", my2);    // 200
    
    // 拼接函数名
    int CONCAT(cal, c)(int a, int b) { return a + b; }  // 定义 calc 函数
    printf("calc(3,4) = %d\n", calc(3, 4));              // 7
    return 0;
}
```

#### 3.2.5 可变参数宏（Variadic Macro，C99）

```c
#define LOG_INFO(fmt, ...)  printf("[INFO] " fmt "\n", ##__VA_ARGS__)
#define DEBUG_PRINT(...)    fprintf(stderr, "DEBUG: " __VA_ARGS__)

// C23 正式引入 __VA_OPT__（GCC/Clang 早已支持）
#define MY_LOG(fmt, ...)  printf(fmt __VA_OPT__(,) __VA_ARGS__)

int main(void) {
    LOG_INFO("Server started on port %d", 8080);
    LOG_INFO("No extra args");        // 利用 ## 消除前面的逗号
    DEBUG_PRINT("x=%d, y=%d\n", 10, 20);
    
    MY_LOG("Hello\n");               // __VA_OPT__(,) 当无参数时不展开逗号
    MY_LOG("Value: %d\n", 42);
    return 0;
}
```

> `##__VA_ARGS__` 是 GCC 扩展（消除尾随逗号），C23 标准用 `__VA_OPT__` 正式解决此问题。

#### 3.2.6 do-while(0) 技巧

```c
// 安全的宏封装多语句
#define SAFE_FREE(ptr) do {     \
    free(ptr);                  \
    (ptr) = NULL;               \
} while(0)

#define LOG_AND_RETURN(err) do { \
    fprintf(stderr, "Error %d at %s:%d\n", err, __FILE__, __LINE__); \
    return err;                  \
} while(0)

void example(void) {
    int *p = malloc(sizeof(int) * 10);
    // ...
    SAFE_FREE(p);    // 可以安全地用在 if/else 语句中
    
    if (1)
        SAFE_FREE(p);  // 不会出现悬挂 else 问题
    else
        printf("won't happen\n");
}
```

---

### 3.3 `#undef` — 取消宏定义

```c
#define BUFFER_SIZE 256

// 使用 BUFFER_SIZE...
char buf[BUFFER_SIZE];

#undef BUFFER_SIZE    // 取消定义

// 之后可以重新定义
#define BUFFER_SIZE 512
char buf2[BUFFER_SIZE];
```

**常见场景：** 限制宏的作用范围，避免宏名冲突。

---

### 3.4 条件编译指令组

#### 3.4.1 `#if` / `#elif` / `#else` / `#endif`

```c
#define OPT_LEVEL 2

#if OPT_LEVEL == 0
    #define LOG_MSG "No optimization"
#elif OPT_LEVEL == 1
    #define LOG_MSG "Basic optimization"
#elif OPT_LEVEL == 2
    #define LOG_MSG "Full optimization"
#else
    #define LOG_MSG "Unknown level"
#endif

// #if 中可以使用的运算符：
//   算术: +, -, *, /, %
//   比较: ==, !=, <, >, <=, >=
//   逻辑: &&, ||, !
//   位运算: &, |, ^, ~, <<, >>
//   defined() 操作符

#if defined(_WIN32) && !defined(_WIN64)
    #define PLATFORM "Windows 32-bit"
#elif defined(_WIN64)
    #define PLATFORM "Windows 64-bit"
#elif defined(__linux__)
    #define PLATFORM "Linux"
#elif defined(__APPLE__)
    #define PLATFORM "macOS"
#else
    #define PLATFORM "Unknown"
#endif
```

#### 3.4.2 `#ifdef` / `#ifndef`

```c
#define DEBUG

#ifdef DEBUG
    printf("Debug mode is ON\n");
#endif

#ifndef RELEASE
    printf("This is NOT a release build\n");
#endif

// 经典用法：头文件保护宏（Include Guard）
// ========== myheader.h ==========
#ifndef MYHEADER_H
#define MYHEADER_H

void my_function(void);
extern int global_var;

#endif /* MYHEADER_H */
```

#### 3.4.3 `defined()` 操作符

```c
// defined() 只能用于 #if 和 #elif 中
// 两种写法等价：defined(MACRO) 和 defined MACRO

#if defined(__GNUC__) && !defined(__clang__)
    // GCC 但不是 Clang（Clang 也定义了 __GNUC__）
    #pragma GCC optimize("O3")
#elif defined(__clang__)
    #pragma clang optimize on
#endif

// 检查多个宏的组合
#if defined(FEATURE_A) || defined(FEATURE_B)
    void feature_init(void);
#endif
```

#### 3.4.4 `#elifdef` / `#elifndef`（C23 新增）

```c
// C23 之前需要写：
#if defined(__GNUC__)
    #define COMPILER "GCC"
#elif defined(__clang__)
    #define COMPILER "Clang"
#elif defined(_MSC_VER)
    #define COMPILER "MSVC"
#endif

// C23 简化写法：
#if defined(__GNUC__)
    #define COMPILER "GCC"
#elifdef __clang__       // 等价于 #elif defined(__clang__)
    #define COMPILER "Clang"
#elifndef _MSC_VER       // 等价于 #elif !defined(_MSC_VER)
    #define COMPILER "Other"
#endif
```

---

### 3.5 `#error` — 编译期错误

```c
// 当条件不满足时，中止编译并输出错误消息
#if !defined(__STDC__)
    #error "This code requires a standard C compiler!"
#endif

#if __STDC_VERSION__ < 199901L
    #error "C99 or later is required to compile this file."
#endif

#if UINTPTR_MAX < 0xFFFFFFFFFFFFFFFFULL
    #error "This library requires a 64-bit platform."
#endif

#if defined(_WIN32) && defined(__linux__)
    #error "Conflicting platform macros detected!"
#endif
```

---

### 3.6 `#warning`（非标准，GCC/Clang 扩展）

```c
#ifdef USE_DEPRECATED_API
    #warning "You are using deprecated API. Please migrate to the new API."
#endif

#if __STDC_VERSION__ < 201112L
    #warning "Consider upgrading to C11 for better thread safety."
#endif
```

---

### 3.7 `#pragma` — 编译器特定指令

#### 3.7.1 头文件保护

```c
// 等效于 #ifndef include guard，大多数现代编译器支持
#pragma once

// ========== myheader.h ==========
#pragma once
void my_function(void);
```

#### 3.7.2 结构体内存对齐（GCC/Clang/MSVC 通用）

```c
#pragma pack(push, 1)     // 保存当前对齐，设为 1 字节对齐
struct PackedData {
    char  a;     // 1 byte
    int   b;     // 4 bytes
    short c;     // 2 bytes
};                // sizeof = 7（而非默认的 12）
#pragma pack(pop)         // 恢复之前的对齐

// 或直接指定
#pragma pack(4)
struct Align4Data {
    char  a;
    int   b;
};
#pragma pack()            // 恢复默认对齐
```

#### 3.7.3 GCC 特有 pragma

```c
// 控制编译优化
#pragma GCC optimize("O3")
#pragma GCC optimize("unroll-loops")

// 控制警告
#pragma GCC diagnostic push
#pragma GCC diagnostic ignored "-Wunused-variable"
int unused_var = 42;        // 不会产生警告
#pragma GCC diagnostic pop   // 恢复之前的诊断状态

// 指定代码段/数据段
#pragma GCC section text=".mycode"
void my_special_func(void) { /* ... */ }
#pragma GCC section text    // 恢复默认

// 编译器选项推入/弹出
#pragma GCC push_options
#pragma GCC optimize("O0")
void debug_func(void) { /* 不优化 */ }
#pragma GCC pop_options
```

#### 3.7.4 MSVC 特有 pragma

```c
// 链接库
#pragma comment(lib, "user32.lib")
#pragma comment(lib, "ws2_32.lib")

// 编译优化
#pragma optimize("", off)    // 关闭优化
void critical_section(void) { /* ... */ }
#pragma optimize("", on)     // 恢复优化

// 控制警告级别
#pragma warning(push)
#pragma warning(disable: 4996)   // 禁用 "deprecated" 警告
strcpy(dest, src);
#pragma warning(pop)

// 运行时检查
#pragma runtime_checks("sc", off)

// 数据段
#pragma data_seg(".shared")
int shared_var = 0;
#pragma data_seg()

// 自动初始化/终止函数
#pragma section(".CRT$XCU", read)
__declspec(allocate(".CRT$XCU")) void (*my_init)(void) = init_func;
```

#### 3.7.5 Clang 特有 pragma

```c
#pragma clang diagnostic push
#pragma clang diagnostic ignored "-Wformat"
printf("%d", some_value);
#pragma clang diagnostic pop

// 循环优化提示
#pragma clang loop unroll(enable)
#pragma clang loop vectorize(enable)
for (int i = 0; i < n; i++) { /* ... */ }
```

---

### 3.8 `#line` — 修改行号与文件名

```c
// 修改后续代码的 __LINE__ 和 __FILE__ 值
#line 100 "generated_code.c"
// 此处 __LINE__ = 100, __FILE__ = "generated_code.c"

#line 200
// 此处 __LINE__ = 200, __FILE__ 仍为 "generated_code.c"

// 典型场景：代码生成器、yacc/bison 生成的 C 文件
```

---

### 3.9 `_Pragma` 运算符（C99）

```c
// _Pragma 是运算符，不是预处理指令
// _Pragma("xxx") 等价于 #pragma xxx
// 关键优势：可以在宏中使用！

#define STRINGIFY(x) #x
#define DO_PRAGMA(x) _Pragma(STRINGIFY(x))

// 在宏中使用 pragma
#define DISABLE_WARNING(w) \
    DO_PRAGMA(GCC diagnostic push) \
    DO_PRAGMA(GCC diagnostic ignored #w)

#define RESTORE_WARNING() \
    DO_PRAGMA(GCC diagnostic pop)

// 使用
DISABLE_WARNING(-Wunused-variable)
int x;   // 不报警告
RESTORE_WARNING()
```

---

### 3.10 `#` 空指令

```c
// 单独的 # 号是空指令，什么都不做
#

// 有些代码生成器会在条件编译中使用空指令占位
#if DEBUG
    #define LOG(x) printf x
#else
    #
#endif
```

---

## 四、全部标准预定义宏

### 4.1 基础预定义宏（C89/C99 起）

|宏名|引入|类型|说明|
|---|---|---|---|
|`__FILE__`|C89|`const char*`|当前源文件路径名|
|`__LINE__`|C89|`int`|当前代码行号|
|`__DATE__`|C89|`const char*`|编译日期，格式 `"Mmm dd yyyy"`|
|`__TIME__`|C89|`const char*`|编译时间，格式 `"hh:mm:ss"`|
|`__STDC__`|C89|`int`|遵循 ANSI/ISO C 标准时为 `1`|
|`__STDC_VERSION__`|C99|`long`|C 标准版本号|
|`__STDC_HOSTED__`|C99|`int`|宿主实现为 `1`，自由实现为 `0`|
|`__func__`|C99|`const char*`|当前函数名（预定义标识符，非严格宏）|
|`__STDC_ISO_10646__`|C99|`long`|wchar_t 采用 ISO 10646 编码时的年月值|
|`__STDC_IEC_559__`|C99|`int`|浮点数遵循 IEC 60559 (IEEE 754) 时为 `1`|
|`__STDC_IEC_559_COMPLEX__`|C99|`int`|复数遵循 IEC 60559 时为 `1`|

### 4.2 C11 新增预定义宏

|宏名|说明|
|---|---|
|`__STDC_NO_ATOMICS__`|不支持 `<stdatomic.h>` 时定义为 `1`|
|`__STDC_NO_COMPLEX__`|不支持复数类型时定义为 `1`|
|`__STDC_NO_THREADS__`|不支持 `<threads.h>` 时定义为 `1`|
|`__STDC_NO_VLA__`|不支持变长数组 (VLA) 时定义为 `1`|
|`__STDC_ANALYZABLE__`|支持 Annex L（可分析性）时定义为 `1`|

### 4.3 `__STDC_VERSION__` 版本值对照表

|值|对应标准|
|---|---|
|_(未定义)_|C89/C90|
|`199409L`|C95 (C90 Amendment 1)|
|`199901L`|C99|
|`201112L`|C11|
|`201710L`|C17/C18|
|`202311L`|C23|

### 4.4 常见编译器/平台扩展宏（非标准但广泛使用）

```c
// ============= 编译器检测 =============
#ifdef __GNUC__
    printf("GCC version: %d.%d.%d\n",
           __GNUC__, __GNUC_MINOR__, __GNUC_PATCHLEVEL__);
#endif

#ifdef __clang__
    printf("Clang version: %d.%d.%d\n",
           __clang_major__, __clang_minor__, __clang_patchlevel__);
#endif

#ifdef _MSC_VER
    printf("MSVC version: %d\n", _MSC_VER);     // 如 1936 = VS2022
    printf("MSVC full version: %d\n", _MSC_FULL_VER);
#endif

// ============= 操作系统检测 =============
#if defined(_WIN32) || defined(_WIN64)
    #define OS_NAME "Windows"
#elif defined(__linux__)
    #define OS_NAME "Linux"
#elif defined(__APPLE__) && defined(__MACH__)
    #define OS_NAME "macOS"
#elif defined(__FreeBSD__)
    #define OS_NAME "FreeBSD"
#elif defined(__unix__)
    #define OS_NAME "Unix"
#else
    #define OS_NAME "Unknown"
#endif

// ============= 架构检测 =============
#if defined(__x86_64__) || defined(_M_X64)
    #define ARCH "x86_64"
#elif defined(__i386__) || defined(_M_IX86)
    #define ARCH "x86"
#elif defined(__aarch64__) || defined(_M_ARM64)
    #define ARCH "ARM64"
#elif defined(__arm__) || defined(_M_ARM)
    #define ARCH "ARM"
#elif defined(__riscv)
    #define ARCH "RISC-V"
#endif

// ============= 其他常用扩展宏 =============
// __COUNTER__        — 从 0 开始每次使用递增 1（GCC/Clang/MSVC）
// __TIMESTAMP__      — 文件最后修改时间（GCC/Clang）
// __FUNCTION__       — 当前函数名（GCC/MSVC，__func__ 的非标准前身）
// __PRETTY_FUNCTION__— 带签名的函数名（GCC/Clang）
// __BASE_FILE__      — 最顶层源文件名（GCC）
// __INCLUDE_LEVEL__  — 头文件嵌套包含层级（GCC）
```

---

## 五、综合实战示例

### 5.1 跨平台日志宏系统

```c
// ========== logger.h ==========
#pragma once
#include <stdio.h>
#include <time.h>

// ---- 日志级别定义 ----
#define LOG_LEVEL_NONE  0
#define LOG_LEVEL_ERROR 1
#define LOG_LEVEL_WARN  2
#define LOG_LEVEL_INFO  3
#define LOG_LEVEL_DEBUG 4

// ---- 默认日志级别 ----
#ifndef LOG_LEVEL
    #ifdef NDEBUG
        #define LOG_LEVEL LOG_LEVEL_ERROR
    #else
        #define LOG_LEVEL LOG_LEVEL_DEBUG
    #endif
#endif

// ---- 颜色支持（终端）----
#if defined(__linux__) || defined(__APPLE__)
    #define COLOR_RED     "\033[31m"
    #define COLOR_YELLOW  "\033[33m"
    #define COLOR_GREEN   "\033[32m"
    #define COLOR_BLUE    "\033[34m"
    #define COLOR_RESET   "\033[0m"
    #define HAS_COLOR 1
#else
    #define COLOR_RED     ""
    #define COLOR_YELLOW  ""
    #define COLOR_GREEN   ""
    #define COLOR_BLUE    ""
    #define COLOR_RESET   ""
    #define HAS_COLOR 0
#endif

// ---- 核心日志宏 ----
#define _LOG_PREFIX(level, color) \
    fprintf(stderr, color "[%s] %s:%d (%s): " COLOR_RESET, \
            level, __FILE__, __LINE__, __func__)

#if LOG_LEVEL >= LOG_LEVEL_DEBUG
    #define LOG_DEBUG(fmt, ...) \
        do { _LOG_PREFIX("DEBUG", COLOR_BLUE); \
             fprintf(stderr, fmt "\n" __VA_OPT__(,) __VA_ARGS__); } while(0)
#else
    #define LOG_DEBUG(fmt, ...) ((void)0)
#endif

#if LOG_LEVEL >= LOG_LEVEL_INFO
    #define LOG_INFO(fmt, ...) \
        do { _LOG_PREFIX("INFO", COLOR_GREEN); \
             fprintf(stderr, fmt "\n" __VA_OPT__(,) __VA_ARGS__); } while(0)
#else
    #define LOG_INFO(fmt, ...) ((void)0)
#endif

#if LOG_LEVEL >= LOG_LEVEL_WARN
    #define LOG_WARN(fmt, ...) \
        do { _LOG_PREFIX("WARN", COLOR_YELLOW); \
             fprintf(stderr, fmt "\n" __VA_OPT__(,) __VA_ARGS__); } while(0)
#else
    #define LOG_WARN(fmt, ...) ((void)0)
#endif

#if LOG_LEVEL >= LOG_LEVEL_ERROR
    #define LOG_ERROR(fmt, ...) \
        do { _LOG_PREFIX("ERROR", COLOR_RED); \
             fprintf(stderr, fmt "\n" __VA_OPT__(,) __VA_ARGS__); } while(0)
#else
    #define LOG_ERROR(fmt, ...) ((void)0)
#endif
```

### 5.2 断言宏系统

```c
// ========== myassert.h ==========
#pragma once
#include <stdio.h>
#include <stdlib.h>

#ifdef NDEBUG
    // Release 模式：断言被移除
    #define MY_ASSERT(expr)          ((void)0)
    #define MY_ASSERT_MSG(expr, ...) ((void)0)
#else
    // Debug 模式
    #define MY_ASSERT(expr) \
        do { \
            if (!(expr)) { \
                fprintf(stderr, \
                    "Assertion failed: %s\n" \
                    "  File: %s\n  Line: %d\n  Func: %s\n", \
                    #expr, __FILE__, __LINE__, __func__); \
                abort(); \
            } \
        } while(0)

    #define MY_ASSERT_MSG(expr, fmt, ...) \
        do { \
            if (!(expr)) { \
                fprintf(stderr, \
                    "Assertion failed: %s\n" \
                    "  Message: " fmt "\n" \
                    "  File: %s\n  Line: %d\n  Func: %s\n", \
                    #expr, __VA_ARGS__, \
                    __FILE__, __LINE__, __func__); \
                abort(); \
            } \
        } while(0)
#endif

// 使用示例
void divide(int a, int b) {
    MY_ASSERT(b != 0);
    MY_ASSERT_MSG(a >= 0, "Expected non-negative a, got %d", a);
    printf("%d / %d = %d\n", a, b, a / b);
}
```

### 5.3 自动生成唯一变量名

```c
// 利用 __COUNTER__ 和 ## 生成唯一标识符
#define CONCAT_IMPL(a, b) a##b
#define CONCAT(a, b)      CONCAT_IMPL(a, b)
#define UNIQUE_VAR(prefix) CONCAT(prefix##_, __COUNTER__)

int main(void) {
    int UNIQUE_VAR(tmp) = 1;   // int tmp_0 = 1;
    int UNIQUE_VAR(tmp) = 2;   // int tmp_1 = 2;
    int UNIQUE_VAR(tmp) = 3;   // int tmp_2 = 3;
    return 0;
}
```

### 5.4 完整的跨平台兼容性头文件

```c
// ========== platform.h ==========
#pragma once

// ---- 编译器检测 ----
#if defined(__clang__)
    #define COMPILER_CLANG  1
    #define COMPILER_NAME   "Clang"
#elif defined(__GNUC__)
    #define COMPILER_GCC    1
    #define COMPILER_NAME   "GCC"
#elif defined(_MSC_VER)
    #define COMPILER_MSVC   1
    #define COMPILER_NAME   "MSVC"
#else
    #define COMPILER_UNKNOWN 1
    #define COMPILER_NAME    "Unknown"
#endif

// ---- C 标准版本检测 ----
#if defined(__STDC_VERSION__)
    #if __STDC_VERSION__ >= 202311L
        #define C_STANDARD 23
    #elif __STDC_VERSION__ >= 201710L
        #define C_STANDARD 17
    #elif __STDC_VERSION__ >= 201112L
        #define C_STANDARD 11
    #elif __STDC_VERSION__ >= 199901L
        #define C_STANDARD 99
    #else
        #define C_STANDARD 90
    #endif
#else
    #define C_STANDARD 89
#endif

// ---- 操作系统检测 ----
#if defined(_WIN32) || defined(_WIN64)
    #define OS_WINDOWS  1
    #define OS_NAME     "Windows"
    #ifdef _WIN64
        #define OS_BITS 64
    #else
        #define OS_BITS 32
    #endif
#elif defined(__APPLE__)
    #define OS_MACOS    1
    #define OS_NAME     "macOS"
    #define OS_BITS     64
#elif defined(__linux__)
    #define OS_LINUX    1
    #define OS_NAME     "Linux"
    #if defined(__x86_64__) || defined(__aarch64__)
        #define OS_BITS 64
    #else
        #define OS_BITS 32
    #endif
#else
    #define OS_UNKNOWN  1
    #define OS_NAME     "Unknown"
    #define OS_BITS     0
#endif

// ---- 编译器特有属性封装 ----
#ifdef COMPILER_GCC
    #define LIKELY(x)   __builtin_expect(!!(x), 1)
    #define UNLIKELY(x) __builtin_expect(!!(x), 0)
    #define UNUSED      __attribute__((unused))
    #define DEPRECATED  __attribute__((deprecated))
    #define NORETURN    __attribute__((noreturn))
    #define PACKED      __attribute__((packed))
    #define ALIGNED(n)  __attribute__((aligned(n)))
    #define WEAK        __attribute__((weak))
#elif defined(COMPILER_CLANG)
    #define LIKELY(x)   __builtin_expect(!!(x), 1)
    #define UNLIKELY(x) __builtin_expect(!!(x), 0)
    #define UNUSED      __attribute__((unused))
    #define DEPRECATED  __attribute__((deprecated))
    #define NORETURN    __attribute__((noreturn))
    #define PACKED      __attribute__((packed))
    #define ALIGNED(n)  __attribute__((aligned(n)))
    #define WEAK        __attribute__((weak))
#elif defined(COMPILER_MSVC)
    #define LIKELY(x)   (x)
    #define UNLIKELY(x) (x)
    #define UNUSED
    #define DEPRECATED  __declspec(deprecated)
    #define NORETURN    __declspec(noreturn)
    #define PACKED
    #define ALIGNED(n)  __declspec(align(n))
    #define WEAK
#else
    #define LIKELY(x)   (x)
    #define UNLIKELY(x) (x)
    #define UNUSED
    #define DEPRECATED
    #define NORETURN
    #define PACKED
    #define ALIGNED(n)
    #define WEAK
#endif

// ---- 使用示例 ----
#include <stdio.h>

int main(void) {
    printf("Compiler: %s\n", COMPILER_NAME);
    printf("C Standard: C%d\n", C_STANDARD);
    printf("OS: %s (%d-bit)\n", OS_NAME, OS_BITS);
    
    struct PACKED MyData {
        char a;
        int  b;
    };
    printf("sizeof(MyData) = %zu\n", sizeof(struct MyData));
    
    int x = 42;
    if (LIKELY(x > 0)) {
        printf("Positive!\n");
    }
    
    return 0;
}
```

---

## 六、预处理运算符速查表

|运算符|名称|用法|说明|
|---|---|---|---|
|`#`|字符串化|`#param`|将宏参数转换为字符串字面量|
|`##`|令牌粘贴|`a##b`|将两个标记拼接为一个|
|`defined`|定义检测|`defined(MACRO)`|用于 `#if`/`#elif`，宏已定义返回 1|
|`_Pragma`|Pragma 运算符|`_Pragma("...")`|C99，可在宏中使用的 `#pragma`|
|`__VA_ARGS__`|可变参数|`...` 展开|C99，代表可变参数部分|
|`__VA_OPT__`|可选分隔符|`__VA_OPT__(,)`|C23，有可变参数时展开其内容|

---

## 七、最佳实践与常见陷阱

|编号|建议|
|---|---|
|1|函数宏的参数和整体表达式**必须加括号** `((x)*(x))`|
|2|多语句宏用 `do { ... } while(0)` 包裹|
|3|优先使用 `inline` 函数替代函数宏（有类型检查）|
|4|头文件**必须**使用 include guard 或 `#pragma once`|
|5|宏名**全部大写**，与函数/变量区分|
|6|避免在宏中使用有**副作用的参数**（如 `MAX(i++, j++)`）|
|7|用 `#undef` 限制宏的作用范围|
|8|条件编译中优先用 `#if defined()` 而非 `#ifdef`（更灵活）|
|9|跨平台代码用**统一的抽象层宏**封装差异|
|10|`#error` 应尽早检测不兼容的配置，快速失败|

---

> **总结**：C 语言一共有 **15 种标准预处理指令**（C23 新增 `#elifdef`/`#elifndef` 后为 **17 种**），**1 个运算符形式** `_Pragma`，**6 种预处理运算符**（`#`、`##`、`defined`、`_Pragma`、`__VA_ARGS__`、`__VA_OPT__`），以及约 **20+ 个标准预定义宏**。掌握它们可以写出更灵活、更健壮、更易移植的 C 代码。