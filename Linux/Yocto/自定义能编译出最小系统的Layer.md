# 自定义能编译出最小系统的Layer

假设已经在当前会话窗口中进行过环境设置，即执行过**MACHINE=imx6qvab820 source via-setup-release.sh -b build-vab820**，保证一些命令能正常使用

### 1. 进入到sources目录，并创建自定义的layer
> cd sources  
> yocto-layer create my  

**在创建layer过程中，按照设置向导的提示对layer进行设置：**  
Please enter the layer priority you'd like to use for the layer: [default: 6] 6  
Would you like to have an example recipe created? (y/n) [default: n] y  
Please enter the name you'd like to use for your example recipe: [default: example] my 
Would you like to have an example bbappend file created? (y/n) [default: n] n  

New layer created in meta-my.

Don't forget to add it to your BBLAYERS (for details see meta-my\README).

layer创建完成后，在sources目录自动创建meta-my目录，该目录中包含了基本的配方文件，源代码文件及配置文件等。

### 2. 适当修改配方文件以编译Linux最小系统
**删除示例recipe的源代码目录：**  
> rm -rf sources/meta-my/repices-example/example/my-0.1

**修改示例recipe的配方文件my_0.1.bb内容：**  
> SUMMARY = "Simple helloworld application"  
> SECTION = "examples"  
> LICENSE = "MIT"  
> LIC_FILES_CHKSUM = "file://${COMMON_LICENSE_DIR}/MIT;md5=0835ade698e0bcf8506ecda2f7b4f302"
> 
> inherit core-image  
> 
> IMAGE_FEATURES += " splash"  
> 
> PACKAGE_ARCH = "${MACHINE_ARCH}"  

其中inherit core-image，表示继承core-image.bbclass，其中已经设置好了一些基本配置变量及相关的功能函数，编译my recipe时，就能编译出基本的Linux系统。

### 3. 使能新建的layer
**在via-setup-release.sh中使能meta-my层:**  
> echo "BBLAYERS += \" \${BSPDIR}/sources/meta-my \"" >> $BUILD_DIR/conf/bblayers.conf

在via-setup-releash.sh中使能meta-my层后，在初始化环境后，会在build目录的conf/bblayer.bb中生成新增的内容

### 4. 初始化环境
在yocto根目录执行如下命令来初始化环境：
> MACHINE=imx6qvab820 source via-setup-releash.sh -b build-mytest

### 5. 编译Linux最小系统
环境初始化完成后，会自动进入build-mytest目录，在该目录编译my recipe
> bitbake my

编译完成后，uboot、kernel和根文件系统将会生成在如下目录中：  
build-mytest/tmp/deploy/images/imx6qvab820
