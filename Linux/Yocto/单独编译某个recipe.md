# 单独编译某个recipe

当需要单独编译某个recipe时，可以执行如下命令：

### 1. 进入build目录
> cd build-xxx

### 2. 重新编译
> bitbake -c cleansstate linux-imx  
> bitbake linux-imx

其中，linux-imx表示某个recipe名称，对应路径为sources/meta-xxx-bsp/meta-xxx-arm/recipes-kernel/linux/  
  
该路径包含内容如下：  
linux-imx (目录，存放源代码，如dts和patch文件等)  
linux-imx_4.1.15.bbappend (配方文件）  
