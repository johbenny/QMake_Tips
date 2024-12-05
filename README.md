# QMake

[qmake Languag](https://doc.qt.io/qt-6/qmake-manual.html)

- QMake 自动包含 MOC 和 UIC 特性

- QMake 可易理解为字符串形式的脚本语言，`按行解析`

### 数据类型：变量 ，列表 （都是字符串类型）

---
- 变量赋值
```qmake
DEST = "Program Files" # 变量赋值

HEADERS = mainwindow.h paintwidget.h # 列表赋值

# 允许 “\” 换行， 列表 append 追加
SOURCES += main.cpp mainwindow.cpp \ 
          paintwidget.cpp 

SOURCES -= main.cpp # 删除列表中的元素
```

- 取值
```qmake
MY_VAR = $$SOURCES # 自定义变量，变量展开
MY_VAR = $${MY_VAR}_$${TEMPLATE} # 字符串拼接
DESTDIR = $$(PWD) # 读取环境变量 $$(...)
message(Qt version: $$[QT_VERSION]) # 读取 QMake 的属性 $$[...]
```
### 条件运算

Scope 用于执行判断语句的结果

```qmake
<condition> {
    <command or definition>
    ...
}
```
1. 变量条件运算

理解为变量是否存在

```qmake
win32 {
    SOURCES += paintwidget_win.cpp
} 
# 非
!win32 {
    SOURCES -= paintwidget_win.cpp
}
# 或
win32|macx {
    HEADERS += debugging.h
}
# 字符串匹配
win32-* {
    # Matches every mkspec starting with "win32-"
    SOURCES += win32_specific.cpp
}
# 配置是否存在
CONFIG += opengl
opengl {
    TARGET = application-gl
} else {
    TARGET = application
}
```
2. 变量值条件运算
```cmake
# 判断变量值，且指定变量值的互斥条件（存在 debug, 且 debug 和 release 不能并存）
CONFIG(debug, debug|release) {
        HEADERS += debugging.h
    }
# 判断变量值，无互斥条件
CONFIG(debug) {
    HEADERS += debugging.h
}
# 与
macx:CONFIG(debug, debug|release) {
    HEADERS += debugging.h
}
```
3. if - else
```qmake
if(win32|macos):CONFIG(debug, debug|release) {
    # Do something on Windows and macOS,
    # but only for the debug configuration.
}

win32:xml {
    message(Building for Windows)
    SOURCES += xmlhandler_win.cpp
} else:xml {                     # else if
    SOURCES += xmlhandler.cpp
} else {
    message("Unknown configuration")
}
```
4. 单行条件判断
```qmake
win32:DEFINES += USE_MY_STUFF # if win32
```

## 函数 (Replace Functions / Test Functions)
- [Replace Functions](https://doc.qt.io/qt-6/qmake-function-reference.html)
```qmake
# 定义函数 headersAndSources
defineReplace(headersAndSources) {
    variable = $$1          # 引用第一个参数
    names = $$eval($$variable) # 局部变量，外部不可见
    headers =
    sources =

    for(name, names) { # for-each 循环
        header = $${name}.h
        exists($$header) {
            headers += $$header
        }
        source = $${name}.cpp
        exists($$source) {
            sources += $$source
        }
    }
    return($$headers $$sources) # 返回值
}
```
- [Test Functions](https://doc.qt.io/qt-6/qmake-test-function-reference.html)

可用于条件判断

```qmake
defineTest(allFiles) {
    files = $$ARGS # 引用所有参数为列表

    for(file, files) {
        !exists($$file) {
            return(false)
        }
    }
    return(true)
}
```
## 项目构建

qmake 通过一些预置的变量(空值)来管理项目

[Building Common Project Types](https://doc.qt.io/qt-6/qmake-common-projects.html)

```qmake
TEMPLATE = app # 指定目标类型
DESTDIR  = c:/helloapp # 指定生成目标存储位置
HEADERS += hello.h # 指定目标头文件
SOURCES += hello.cpp #指定目标源文件
SOURCES += main.cpp 
DEFINES += USE_MY_STUFF # 添加预处理宏定义
CONFIG  += release # 设置配置信息
```

**[TEMPLATE](https://doc.qt.io/qt-6/qmake-variable-reference.html#template)**

TEMPLATE 用于划分目标配置就作用域，并指定目标类型

**子项目配置**

[SUBDIRS - handling dependencies](https://wiki.qt.io/SUBDIRS_-_handling_dependencies)


**链接库**

配置 `LIBS` 
- -l ： 链接指定名称的库
- -L ： 指定库文件的搜索路径
- -framework ：  macOS 上链接特定的框架
- -F ： macOS 上指定框架的搜索路径

`特： `
```qmake
# 直接链接特定库文件
LIBS += /path/to/library/libMyStaticLibrary.a
```
  

**构建类型**

配置 [`CONFIG`](https://doc.qt.io/qt-6/qmake-variable-reference.html#config) 实现构建类型

```qmake
CONFIG += debug_and_release # 同时构建 debug 和 release 版本

# 根据构建版本设置目标名称
CONFIG(debug, debug|release) {
    TARGET = debug_binary
} else {
    TARGET = release_binary
}
```

## 构建 Qt 项目

QMake 对 Qt 项目有特殊支持，无需手动链接 Qt 库， QMake 自动处理 Qt 库依赖

```qmake
CONFIG += qt # 启动 qmake 对 Qt 的支持
QT += network xml # 包含 Qt 库组件
TARGET = MyProject
TEMPLATE = app

SOURCES += main.cpp \
           mywidget.cpp
HEADERS  += mywidget.h
```