# imx6 DDR配置过程

## 1. DDR配置选择

> uboot目录/configs/mx6qvab820_defconfig文件中，通过CONFIG_SYS_EXTRA_OPTIONS来包含配置文件vab820.cfg，vab820.cfg中会根据宏CONFIG_USE_PLUGIN来决定是使用plugin.S还是该文件中自身包含的配置内容

## 2. plugin.S作用

> DDR的初始化动作是在plugin.S中处理，plugin.S会根据SOC类型及DDR内存大小来include不同的DDR配置文件，配置文件路径：/board/via/vab820/中

## 3. DDR配置文件的来源

> 配置文件中来源于《IMX6DQSDL DDR3 Script Aid V0.11》文档生成的RealView.inc文件的内容



> 注：调试过程中，如果DDR配置不正确，uboot有时无法运行，或kernel随机panic，或Android起来后，USB鼠标无法正常使用。

