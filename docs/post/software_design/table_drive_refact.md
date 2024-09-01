

## 问题场景

考虑如下一个场景，一个函数需要对不同的硬件平台做特殊处理，实现代码如下：

```c
typedef enum {
	PLATFORM_TYPE1,
	PLATFORM_TYPE2,
	PLATFORM_TYPE3,
	PLATFORM_BUTT
} Platform;

void Process (Platform platform, void *args)
{
    switch (platform)
    {
        case PLATFORM_TYPE1:
            /* code */
            break;
        case PLATFORM_TYPE2:
            /* code */
            break;
        case PLATFORM_TYPE3:
            /* code */
        break;
        default:
            break;
    }
}
```

看着貌似没啥问题，可是一个较大的项目可能这样需要对不同的硬件平台做特殊处理的函数有很多个，如果需要新增硬件平台的话，可维护性和可扩展性太差，比如以下弊端：

- 修改遗漏导致出现问题
- 修改工作量太大
- 超大函数，代码膨胀

这个问题场景可通过**表驱动重构**方法完美解决，一般来说需要五个要素：

- tag定义
- tag入参结构体（如果各平台需要的参数相同，则定义一个公共结构体即可）
- tag处理函数（如果各平台部分处理逻辑相同，可提取重复作为公共函数）
- 配置表定义
- 表配置框架

## tag定义

如果新增平台，需要在此处新增对应的tag定义。

```c
// 扩展点1 - 类型枚举
typedef enum {
	PLATFORM_TYPE1,
	PLATFORM_TYPE2,
	PLATFORM_TYPE3,
	PLATFORM_BUTT
} Platform;
```

## tag入参结构体

如果新增平台，需要在此处新增对应的tag入参结构体。

考虑到各个平台处理需要的参数可能不同，所以各个平台需要自己新增一个入参结构体，tag处理函数根据此类型进行逻辑处理，这一点也是依赖倒置原则（细节应当依赖于抽象）的体现。

```c
// 扩展点2 - 类型对应的入参结构体
typedef struct {
	int para1;
    int para2;
} ArgsForType1;
```

## tag处理函数

如果新增平台，需要在此处新增对应的tag处理函数。

每个tag处理函数仅仅只负责本平台的逻辑处理，体现了单一职责原则。

```c
// 扩展点3 - 类型对应的处理函数
void ProcessForPlatformType1 (void *args)
{
    // 入参转换
    ArgsForType1* argsForType1 = (ArgsForType1 *)args;
    // 逻辑处理
    printf("in ProcessForPlatformType1: para1:%d, para1:%d\n", argsForType1->para1, argsForType1->para2);
}
void ProcessForPlatformType2 (void *args)
{
    // 入参转换
    // 逻辑处理
}
void ProcessForPlatformType3 (void *args)
{
    // 入参转换
    // 逻辑处理
}
```

## 配置表定义

如果需要新增一个平台，需要在配置表中新增一行配置。

```c
// 配置表元素定义
typedef struct {
    Platform platform;
    void (*pFunc)(void *);
} TableCfgInfo;

// 扩展点4 - 配置表注册
TableCfgInfo tableCfgInfo[] = {
    { PLATFORM_TYPE1, ProcessForPlatformType1 },
    { PLATFORM_TYPE2, ProcessForPlatformType2 },
    { PLATFORM_TYPE3, ProcessForPlatformType3 },
};
```

## 表配置框架

```c
void Process (Platform platform, void *args)
{
    for (int loop = 0; loop < sizeof(tableCfgInfo) / sizeof(TableCfgInfo); ++loop) {
        if (tableCfgInfo[loop].platform == platform) {
            tableCfgInfo[loop].pFunc(args);
            break;
        }
    }
}
```

最后，如果说有n个对各平台差异化处理的函数，那么通过**表驱动重构**的方法，将修改点从**线性级复杂度**4*n降低为**常数级复杂度**4，大大减轻了打杂员工的心智负担。