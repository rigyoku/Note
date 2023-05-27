# Kotlin
* Android官方语言
* 可以编译成java, 也可以编译成js
* .kt后缀结尾
* kotlinc FILE -include-runtime -d XX.[class/jar] 将文件编译成class或者jar
    * -include-runtime包含了kotlin运行库, 不写-include-runtime直接编译成库
* 结尾分号可以省略
* idea中, 可以点击 tools -> kotlin -> show kotlin bytecode 查看编译后的字节码

## 包
* 和java一样package声明, import引入
* 会有默认的包导入

## 数据类型
* 类似ts语法, 使用 变量: 类型 格式来声明
* var 可读可写 val只读(注意只读变量不等价于常量)
* 常用类型: String/Char/Boolean/Int/Byte/Short/Double/Float/Long/Array/List/Set/Map
    * kotlin全都是引用类型, 编译成java后才存在基本类型
    * === 判断地址, == 判断值(等价于equals), 和js一样, 同内容的字符串引用一致
    * 通过.toInt()/.toShort()等方法转成数字
    * 高精度转低精度自动截位, 低精度不能隐式转高精度
    * 位运算符方法 shl左移/shr右移/and与/or或/xor异或/inv反向
* 可以类型推断, 可以省略一些类型声明
* 编译时常量使用const修饰, 不能写到局部变量(函数内运行时才能执行赋值), 需要配合val声明
* 可以使用模板字符串, 和flutter一样, 用$x显示变量, ${}包裹表达式即可

## 表达式(有返回值)
* if也是表达式, 返回代码块最后一行
* range表达式, 使用..表示范围, 可以配合in表示在范围内, 包括开始包括结束
* when表达式, 类似于java的switch/case/break, 条件可以逗号分隔, else相当于default

## 函数
* 入口也是main函数
* 使用fun关键字定义函数
* 默认public, 可以写private
* 返回值类型写在参数列表后 : 返回值类型
* 无返回用Unit关键字, 相当于其他语言void, 也可以省略
* 用vararg修饰可变长参数, 就是参数数量不固定, 可以理解为参数数组
    * 放在变量名之前
    * 函数内循环可变长参数取值
* 参数可以设置默认值, 写在返回类型之后
* 支持具名函数, 调用时可以根据参数名设置值, 声明函数时不需要额外处理
* TODO函数, 会抛出异常停止程序并显示对应字符串
* 函数名可以``括起来, 使用时也要括起来函数名, 可以用来区分关键字
* 函数的参数如果是函数形式, 可以声明对应函数的参数和返回值类型
    * 非变量形式的函数, 使用 ::函数名 取得函数引用来传入, 算是具名函数
* 变量形式声明函数, 等号右侧是大括号, 内部是 参数->具体逻辑 的形式, 函数类型中返回值也是->声明, 注意不是js的=>
    * :(入参)->返回值 可以省略, 由kotlin自行推断类型
* 匿名函数(lambda表达式)最后一行就是返回值
* 单个参数时it指向item
* 最后一个参数的函数的话, 调用时可以实参列表传前面的, 最后小括号外一个大括号写匿名函数
* inline关键字声明内联(写在fun之前), 有lambda表达式参数的方法用内联可以提高性能(转java后会把代码直接移动过去, 而不是对象声明形式)
* 函数返回函数时, 也是大括号包裹返回的函数内容

## 空安全
* 默认是不可为空类型, 可以为空类型要在类型后加?, 和flutter一样
* 可空类型调用方法要使用?., 为空则跳过当前语句, 当前语句返回null
* !!.去强制执行方法, 相当于断言不为空, 实际为空会抛异常
* 通过if去判断空分歧后, 内部代码块认为是非空
* ?:空合并操作符, 如果前面变量为空, 执行后续代码, 可以配合?.使用
* 先决条件函数 checkNotNull/requireNotNull/require函数, 空了抛异常

## 常用方法
* substring截取字符串, 默认包前不包后, 可以用..表达式, 可以用until关键字也是包前不包后, 源字符串未变更
* split按字符串分割
* replace替换, 可以传入正则和返回字符串的方法完成选中和替换逻辑, 替换后源字符串未变更
* forEach遍历字符串
* toIntOrNull安全类型转换, 例如防止double的字符串转int报错
* roundToInt四舍五入转成int(而直接toInt是截取)
* "%.xf".format 保留x位小数
* run, 固定this指向本身, 返回最后一行或者指定return
* applay, 固定this指向本身, 返回this本身
* let, 固定it指向本身, 返回最后一行或return
* with, 效果和run一致, 但是调用方式是with(xx){}或者with(xx, f)
* alse, it指向本身, 返回本身
* takeIf, true返回本身, false返回null, 配合?:使用
* takeUnless, 和takeIf相反

## 集合
* List集合
    * 通过listof初始化定长集合, mutableListOf初始化的是可变长集合
    * 使用getOrElse, 传入下标和生成默认值函数防止越界异常
    * getOrNull, 越界返回null, 配合?:
    * 可变长集合可以add/remove增减元素, 注意add指定索引可能导致越界
    * 定长集合可以通过toMutableList转成可变长集合, 也可以toList转回不可变
    * 可以通过 += / -= 来添加/移除(第一个匹配项)
    * removeIf来循环根据条件移除
    * 循环: for(x in list) / list.forEach{} / list.forEachIndexed{}
    * 通过(xx, xx)按顺序来结构, 转成java就是get(index), _可以占位不需要的索引
* Set集合
    * 不重复的List, 通过setOf创建, 通过.elementAt取值, 而不能直接下标取值
    * 同样可以用 elementAtOrElse/elementAtOrNull 防止越界
    * 同样用mutableSetOf创建可变长Set, 可以add/remove/+=/-=
    * 可以来回转换变长/不变长
    * 通过list.toSet()可以去重, 也可以通过list.distinct()去重
* Array数组
    * 数组可以通过arrayof()函数创建, 或者Array工厂函数去循环创建
    * 可以指定类型为IntArray/DoubleArray等(没有StringArray), 对应intArrayOf
    * 也可以 elementAtOrElse/elementAtOrNull
    * 可以和集合相互转换
* Map键值对
    * mapOf(k to v, k to v)创建, 泛型也是<k, v>形式
    * 也可以mapOf(Pair(k,v), Pair(k,v))创建
    * 可以[]取值, 可以get取值, 取不到值返回null
    * 可以 getOrDefault 设取不到的默认值, 注意不同于之前的lambda而是直接给默认值
    * 可以 getOrElse 通过lambda设默认值
    * 通过getValue取值可能会出异常
    * 通过 .forEach{k,v->} / .forEach{it->} / .forEach{(k,v)->} / for(m in map) 来遍历
    * 通过mutableMapOf声明可变长map, 可以put/remove/+=/-=
    * 取值可以 getOrDefault/getOrElse 设默认值
    * getOrPut能取到值返回值, 取不到执行put操作并返回值

## 类
* 也是class关键字
* 继承使用:, 而不是extends
    * 自定义异常可以继承IllegalArgumentException




## Android

### fragment
* 嵌套在Activity中, 多个activity可以是使用同一个fragment
* name指定class, layout指定view
* view中的context属性对应class, 对应Binding, 用XXBinding.inflate初始化

### 进程
* 不写process, 在主进程(包名)
* <service android:name=".Service" android:process=":test"></service>
* 冒号开头是私有进程, 会启动单独进程
* 直接写字符串, 全局进程, 需要权限
* 可以加activity/service之类上, 也可以加到application上控制整体进程, 在logcat中显示location:进程名, 也是Forked child process创建的 
* 在子进程创建不写process的进程, 会出现在主进程中
* 子进程会单独创建application