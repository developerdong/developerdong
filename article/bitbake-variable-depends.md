# BitBake中DEPENDS变量的作用

`DEPENDS`变量只是提供了包的来源，
实际是否用到还得要靠`debian/control`文件中的`Build-Depends`字段去定义。

例如，如果`Build-Depends`字段包含了`libxxx-dev`这个包，
而这个包是私有的，或者在apt中找不到，
那么就可以通过编写bb文件的方式来提供，
然后在`DEPENDS`字段中包含这个包。
反过来，如果在`DEPENDS`变量中包含了`libxxx-dev`，
但是`Build-Depends`字段中没有这个包，
那么编译环境中是不会自动安装这个包的，
编译时还是会出错。

总的来说，要想编译成功，
除了要在`Build-Depends`中声明依赖，
还需要在`DEPENDS`中提供来源。
