# platform_led_driver
platform开发——led驱动

### platform总线驱动
platform是一种虚拟总线，主要用来管理那些不需要时序通信的设备
基本结构图：
![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6fcf923c-2afb-40a5-935c-8783e3a12b22/Untitled.png)

### 核心数据类型之platform_device
```c
struct platform_device 
{
    const char    *name;    //匹配用的名字
    int        id;//设备id,用于在该总线上同名的设备进行编号，如果只有一个设备，则为-1
    struct device    dev;   //设备模块必须包含该结构体
    struct resource    *resource;//资源结构体 指向资源数组
    u32        num_resources;//资源的数量 资源数组的元素个数
    const struct platform_device_id    *id_entry;//设备八字
};
```
```c
struct platform_device_id
{
	char name[20];//匹配用名称
	kernel_ulong_t driver_data;//需要向驱动传输的其它数据
};
```
```c
struct resource 
{
	resource_size_t start;  //资源起始位置   
	resource_size_t end;   //资源结束位置
	const char *name;      
	unsigned long flags;   //区分资源是什么类型的
};
#define IORESOURCE_MEM        0x00000200
#define IORESOURCE_IRQ        0x00000400 
/*
flags 指资源类型，我们常用的是 IORESOURCE_MEM、IORESOURCE_IRQ  这两种。start 和 end 的含义会随着 flags而变更，如
a -- flags为IORESOURCE_MEM 时，start 、end 分别表示该platform_device占据的内存的开始地址和结束值；注意不同MEM的地址值不能重叠
b -- flags为 IORESOURCE_IRQ   时，start 、end 分别表示该platform_device使用的中断号的开始地址和结束值
*/
```
```c
/**
 *注册：把指定设备添加到内核中平台总线的设备列表，等待匹配,匹配成功则回调驱动中probe；
 */
int platform_device_register(struct platform_device *);
/**
 *注销：把指定设备从设备列表中删除，如果驱动已匹配则回调驱动方法和设备信息中的release；
 */
void platform_device_unregister(struct platform_device *);
```
```c
struct resource *platform_get_resource(struct platform_device *dev,unsigned int type, unsigned int num);
/*
	功能：获取设备资源
	参数：dev:平台驱动
		type:获取的资源类型
		num:对应类型资源的序号（如第0个MEM、第2个IRQ等，不是数组下标）
	返回值：成功：资源结构体首地址,失败:NULL
*/
```

#### 核心数据类型之platform_driver
```c
struct platform_driver 
{
    int (*probe)(struct platform_device *);//设备和驱动匹配成功之后调用该函数
    int (*remove)(struct platform_device *);//设备卸载了调用该函数
    
    void (*shutdown)(struct platform_device *);
    int (*suspend)(struct platform_device *, pm_message_t state);
    int (*resume)(struct platform_device *);
    struct device_driver driver;//内核里所有的驱动必须包含该结构体
    const struct platform_device_id *id_table;  //能够支持的设备八字数组，用到结构体数组，一般不指定大小，初始化时最后加{}表示数组结束
};
```
```c
int platform_driver_register(struct platform_driver*pdrv);
/*
	功能：注册平台设备驱动
	参数：pdrv:平台设备驱动结构体
	返回值：成功：0
	失败：错误码
*/
void platform_driver_unregister(struct platform_driver*pdrv);
```
