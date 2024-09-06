## 钩子函数

钩子函数的核心是在原函数基础上做更多事，或拦截原函数做另外的事。

在实际应用中，函数hook可以用于很多场景，比如在调试和测试时，可以通过hook来打印函数的调用信息；在性能优化时，可以通过hook来统计函数的执行时间等。



### 特性模块

```c
// module1_api.h
#include <stdio.h>

// 1、由某个模块实现提供
// 原本的执行函数
int Module1_regFunc1(int para1, int para2);
```



```c
// module1.c
#include "module1_api.h"

int Module1_regFunc1(int para1, int para2) {
  printf("in IRF_regFunc1: %d %d\n", para1, para2);
  return para1 + para2;
}
```





### 钩子模块



```c
// hook_cfg_api.h
#include "module1_api.h"

// 2、hook模块注册
typedef int (*regFunc1Type)(int, int);

typedef struct {
  regFunc1Type regFunc1;
} ModuleRegFuncSet;
static ModuleRegFuncSet module1RegFuncSet;

// 3、 hook实现
#define CALL_FUNC(ret, ralcfg, method, defaultRet, ...) \
  (ret) = ((ralcfg).method == NULL) ? defaultRet : (ralcfg).method(__VA_ARGS__)

int Hook_regFunc1(int para1, int para2);
```





```c
// hook_cfg.c
#include "hook_cfg_api.h"

static ModuleRegFuncSet module1RegFuncSet = {
    .regFunc1 = Module1_regFunc1,
};

// 3、 hook实现
// 对 Module1_regFunc1 包了一层的hook函数
int Hook_regFunc1(int para1, int para2) {
  // 其他处理， hook的核心点
  int ret;
  printf("in Hook_regFunc1: %d %d\n", para1, para2);
  // 原函数regFunc1
  CALL_FUNC(ret, module1RegFuncSet, regFunc1, 0, para1, para2);
  return ret;
}
```



---



使用：`gcc -o main .\main.c .\module1.c .\hook_cfg.c;.\main`

```c
// main.c
#include "hook_cfg_api.h"
int main() {
    printf("in main: %d\n", Hook_regFunc1(1, 2));
    return 0;
}
```

具体来说，它通过hook的方式，对原本的Module1_regFunc1函数进行了包装，使得当调用Hook_regFunc1时，会先调用Hook_regFunc1函数进行处理，然后再调用Module1_regFunc1函数本身。

## 回调函数



回调函数，字面意思就是回过头去调用的函数。如下例中`LoginBeginProc`、`LoginPostProc`就是回调函数：

```c
#include <stdio.h>


void LoginBeginProc()
{
    printf("in LoginBeginProc\n");
}

void LoginPostProc()
{
    printf("in LoginPostProc\n");
}

int UsrLogin(int *pwd, int pwdLen)
{
	LoginBeginProc();
	printf("pwd check\n");
	LoginPostProc();
}

int main()
{
    int pwd[] = {1, 2, 3, 4, 5, 6};
    UsrLogin(pwd, sizeof(pwd));
    return 0;
}
```

