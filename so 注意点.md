
项目引用 .h
.cpp include  .h 进行调用
.so 实现了 .h 



现状
用 jniLibs 方式自动导入 so 编译会报 .cpp:<LineNum>: error: undefined reference to 'xxx'
通过配置 externalNativeBuild.cmake.arguments "-DANDROID_ALLOW_UNDEFINED_SYMBOLS=TRUE" 可以忽略报错编译成功，apk 也已经确实包含了对应的 .so
但仍旧报 dlopen failed: cannot locate symbol "libusb_set_option" referenced by 但运行报错



所以目前 CMakeLists.txt  必须通过 IMPORTED 方式导入 .so