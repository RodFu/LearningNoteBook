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

## 3. Workspace for building packges

Yocto里面会为每个要编译的包分配编译时的工作空间，路径为：**build-xxx/tmp/work/imx6qxxx-poky-linux-gnueabi**

比如，uboot的工作空间为：**build-xxx/tmp/work/imx6qxxx-poky-linux-gnueabi/u-boot-imx/2017.03-r0**

编译uboot时需要执行的任务在文件**build-xxx/tmp/work/imx6qxxxpoky-linux-gnueabi/u-boot-imx/2017.03-r0/temp/log.task_order**中可以查看：

```bash
do_cleansstate (2872): log.do_cleansstate.2872
do_fetch (2957): log.do_fetch.2957
do_unpack (2963): log.do_unpack.2963
do_prepare_recipe_sysroot (2964): log.do_prepare_recipe_sysroot.2964
do_patch (2986): log.do_patch.2986
do_populate_lic (3051): log.do_populate_lic.3051
do_configure (3050): log.do_configure.3050
do_compile (3065): log.do_compile.3065
do_create_extlinux_config (7989): log.do_create_extlinux_config.7989
do_install (7993): log.do_install.7993
do_deploy (7996): log.do_deploy.7996
do_populate_sysroot (8046): log.do_populate_sysroot.8046
do_package (8045): log.do_package.8045
do_packagedata (8124): log.do_packagedata.8124
do_package_qa (8151): log.do_package_qa.8151
do_package_write_rpm (8152): log.do_package_write_rpm.8152
do_fetch (9913): log.do_fetch.9913
do_unpack (9919): log.do_unpack.9919
do_prepare_recipe_sysroot (9920): log.do_prepare_recipe_sysroot.9920
do_patch (9942): log.do_patch.9942
do_populate_lic (10007): log.do_populate_lic.10007
do_configure (10006): log.do_configure.10006
do_compile (10023): log.do_compile.10023
do_fetch (14441): log.do_fetch.14441
do_unpack (14447): log.do_unpack.14447
do_prepare_recipe_sysroot (14448): log.do_prepare_recipe_sysroot.14448
do_patch (14470): log.do_patch.14470
do_configure (14534): log.do_configure.14534
do_populate_lic (14535): log.do_populate_lic.14535
do_compile (14560): log.do_compile.14560
do_create_extlinux_config (19385): log.do_create_extlinux_config.19385
do_deploy (19390): log.do_deploy.19390
do_install (19389): log.do_install.19389
do_package (19441): log.do_package.19441
do_populate_sysroot (19442): log.do_populate_sysroot.19442
do_packagedata (19521): log.do_packagedata.19521
do_package_qa (19548): log.do_package_qa.19548
do_package_write_rpm (19549): log.do_package_write_rpm.19549
do_fetch (19755): log.do_fetch.19755
do_unpack (19761): log.do_unpack.19761
do_prepare_recipe_sysroot (19762): log.do_prepare_recipe_sysroot.19762
do_patch (19784): log.do_patch.19784
do_configure (19848): log.do_configure.19848
do_populate_lic (19849): log.do_populate_lic.19849
do_compile (19881): log.do_compile.19881
do_create_extlinux_config (24696): log.do_create_extlinux_config.24696
do_deploy (24703): log.do_deploy.24703
do_install (24700): log.do_install.24700
do_populate_sysroot (24753): log.do_populate_sysroot.24753
do_package (24752): log.do_package.24752
do_packagedata (24832): log.do_packagedata.24832
do_package_qa (24859): log.do_package_qa.24859
do_package_write_rpm (24860): log.do_package_write_rpm.24860
do_fetch (25250): log.do_fetch.25250
do_unpack (25256): log.do_unpack.25256
do_prepare_recipe_sysroot (25257): log.do_prepare_recipe_sysroot.25257
do_patch (25279): log.do_patch.25279
do_configure (25343): log.do_configure.25343
do_populate_lic (25344): log.do_populate_lic.25344
do_compile (25362): log.do_compile.25362
do_create_extlinux_config (30173): log.do_create_extlinux_config.30173
do_deploy (30180): log.do_deploy.30180
do_install (30177): log.do_install.30177
do_populate_sysroot (30230): log.do_populate_sysroot.30230
do_package (30229): log.do_package.30229
do_packagedata (30309): log.do_packagedata.30309
do_package_write_rpm (30336): log.do_package_write_rpm.30336
do_package_qa (30337): log.do_package_qa.30337
do_fetch (30565): log.do_fetch.30565
do_unpack (30571): log.do_unpack.30571
do_prepare_recipe_sysroot (30572): log.do_prepare_recipe_sysroot.30572
do_patch (30594): log.do_patch.30594
do_configure (30659): log.do_configure.30659
do_populate_lic (30660): log.do_populate_lic.30660
do_compile (30676): log.do_compile.30676
do_create_extlinux_config (3318): log.do_create_extlinux_config.3318
do_deploy (3325): log.do_deploy.3325
do_install (3322): log.do_install.3322
do_package (3374): log.do_package.3374
do_populate_sysroot (3375): log.do_populate_sysroot.3375
do_packagedata (3454): log.do_packagedata.3454
do_package_qa (3481): log.do_package_qa.3481
do_package_write_rpm (3482): log.do_package_write_rpm.3482
do_fetch (3692): log.do_fetch.3692
do_unpack (3698): log.do_unpack.3698
do_prepare_recipe_sysroot (3699): log.do_prepare_recipe_sysroot.3699
do_patch (3721): log.do_patch.3721
do_populate_lic (3786): log.do_populate_lic.3786
do_configure (3785): log.do_configure.3785
do_compile (3811): log.do_compile.3811
do_create_extlinux_config (8610): log.do_create_extlinux_config.8610
do_deploy (8617): log.do_deploy.8617
do_install (8614): log.do_install.8614
do_package (8666): log.do_package.8666
do_populate_sysroot (8667): log.do_populate_sysroot.8667
do_packagedata (8746): log.do_packagedata.8746
do_package_qa (8773): log.do_package_qa.8773
do_package_write_rpm (8774): log.do_package_write_rpm.8774
```



