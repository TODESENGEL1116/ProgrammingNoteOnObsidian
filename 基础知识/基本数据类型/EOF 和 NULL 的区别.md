### 一句话先分清

**EOF 是“整数值”，用来标记「读不到数据了」；NULL 是“空指针”，用来标记「这个指针谁都不指」。** 一个是 `int`，一个是指针——把它们混用，就是 C 语言里最经典的几类 bug 来源。

---

### 一、EOF

**定义**（`<stdio.h>`）：

```c
#define EOF (-1)      /* 标准只要求它是负值的 int，通常是 -1 */
```

- 类型：**int**（这点极其重要）。
- 语义：表示输入流结束或读取出错。它**不是**文件里的一个真实字符。
- 谁返回它：`fgetc()` / `getc()` / `getchar()`、`fscanf()`/`scanf()`（失败或提前遇文件尾）、`fread` 不返回 EOF（它返回实际读到的元素个数，靠 `feof`/`ferror` 判断）。

**经典坑 #1：用 `char` 接 `getchar()`**

```c
char c = getchar();   // 错！0xFF 会被提升成 -1，把合法字符误判成 EOF
if (c == EOF) ...
```

正确写法：

```c
int c;
while ((c = getchar()) != EOF) {
    putchar(c);
}
```

**经典坑 #2：用 `feof()` 当循环条件**

```c
while (!feof(fp)) {          // 错：feof 只在"读过一次且发现到末尾"后才为真 → 最后一行重复一次
    fscanf(fp, "%d", &x);
}
```

正确写法：**先看读取结果，再判断原因**。

```c
while (fscanf(fp, "%d", &x) == 1) { ... }     // 按返回值
// 循环结束后若需区分：
if (feof(fp))  /* 正常读完 */;
if (ferror(fp)) /* 真的出错了 */;
```

**经典坑 #3：把 EOF 当字符写进文件** —— `fprintf(fp, "%d", EOF)` 写进去的是两个字符 `'-'` `'1'`，跟流结束毫无关系。

---

### 二、NULL

**定义**（`<stddef.h>` / `<stdio.h>` 等）：

```c
#define NULL ((void *)0)   /* C 里常见；C++ 里必须写成 0 / nullptr */
```

- 类型：**指针类型**（空指针常量），值为 0，但语义上"不指向任何有效对象"。
- 谁返回它：`fopen()` 失败、`malloc()/calloc()/realloc()` 失败、`strchr()` 没找到、链表/树遍历到头、`fgets()` 读到文件尾或出错。

**基本用法：**

```c
FILE *fp = fopen("a.txt", "r");
if (fp == NULL) { perror("open"); return 1; }   // 一定要检查

int *p = malloc(n * sizeof *p);
if (!p) { /* 内存不足 */ }                      // !p 等价于 p == NULL

char *q = strchr(s, 'x');
if (q != NULL) ...                              // 找到了
```

**经典坑 #4：解引用 NULL** → 段错误（SIGSEGV）。凡是指针使用前先判空，尤其是函数参数和返回值。

**经典坑 #5：free 后不置 NULL** → 悬垂指针，二次 free 或未定义行为。

```c
free(p); p = NULL;   // 好习惯（对局部即将失效的指针意义不大，但对长生命周期的有用）
```

**经典坑 #6：`NULL` 和 `'\0'` 混为一谈。**

|写法|类型/含义|
|---|---|
|`NULL`|空**指针**|
|`0`|整数零（也是空指针常量）|
|`'\0'`|**字符**零，字符串结束符，等价于整数 0|
|`"0"`|字符串，含两个字符 `'0'` 和 `'\0'`|
|`false`|布尔假|

```c
char *s = NULL;    // 指针为空
char t[] = "";     // t[0] == '\0'，指针不为空，只是指向空串
strlen(s);         // 崩！
strlen(t);         // 0，正常
```

---

### 三、两者最容易撞车的场景

**`fgetc` 系列返回的是 int（可能是 EOF），而 `fgets` 返回的是指针（失败/到末尾返回 NULL）——千万别记反。**

```c
int c;
while ((c = fgetc(fp)) != EOF) { ... }     // EOF 是 int

char buf[256];
while (fgets(buf, sizeof buf, fp) != NULL) { ... }  // NULL 是指针
```

同理：`fopen` 返回 `FILE*`，失败是 **NULL**，不是 EOF；`fclose` 成功返回 0，失败返回 **EOF**（这里是 EOF，不是 NULL）。

---

### 四、速查对比表

||EOF|NULL|
|---|---|---|
|本质|宏，负 int（通常 -1）|宏，空指针常量（0 / `(void*)0`）|
|头文件|`<stdio.h>`|`<stddef.h>`、`<stdio.h>` 等|
|表示什么|流结束 / 读取失败|指针无效 / 分配失败 / 没找到|
|典型来源|`getchar`、`fgetc`、`scanf` 族|`fopen`、`malloc`、`strchr`、`fgets`|
|判断方式|`== EOF`|`== NULL` 或 `!p`|
|常见误用|用 `char` 接收、用 `feof` 控循环|解引用、free 后不置空、和 `'\0'` 混淆|

---

### 五、C++ 补充

- `NULL` 在 C++ 中通常是 `0`（不能是 `(void*)0`），所以存在重载歧义问题，推荐用 **`nullptr`**（类型 `std::nullptr_t`）。
- EOF 仍是 -1，不变。
- 现代 C++ 更推荐用 `std::ifstream` + 流状态（`while (cin >> x)`、`.eof()`、`.fail()`），语义类似但更安全。

---

### 六、记忆口诀

> **读到的是值，值不对给 EOF；拿到的是地址，地址无效给 NULL。** `getchar` 看 EOF，`fgets/fopen/malloc` 看 NULL；`char` 别接 `getchar`，`feof` 别当循环条件。

---

要不要我出几道易混淆的选择题，帮你把 EOF 和 NULL 的边界再巩固一下？