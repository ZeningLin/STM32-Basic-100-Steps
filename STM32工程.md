# STM32工程

## 工程结构树
- CMSIS：内核驱动程序，官方提供
- Lib：内部功能的基本函数库，官方提供，**根据项目需求自行增减**
- Startup：单片机启动程序，汇编语言编写，官方提供
- User：用户程序，包括`main.c`；其中还含有一库文件`stm32f10x_it.c`，不需要修改
  - `stm32f10x_it.c`是针对stm32f10x系列单片机的错误处理文件
- Basic：用户自定义内部功能驱动程序，如`delay.c`；还包含`sys.c`文件，官方提供不需要修改
- Hardware：用户自定义外部硬件驱动程序

## MDK配置
- 在`Preprocessor Symbols`的`Define`中填入了`USE_STDPERIPH_DRIVER` `STM32F10X_MD`，相当于在整个工程中进行了下列宏定义：
  ```C
  #define USE_STDPERIPH_DRIVER
  #define STM32F10X_MD  // 芯片容量，此处用了中等容量(64K~256K)
                        // 大容量芯片(>256K)则为HD
  ```
  该操作触发了固件库中的一些条件编译内容使之符合芯片配置要求