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
    * === 判断地址, == 判断值, 和js一样, 每次定义变量引用都会不同
    * 通过.toInt()/.toShort()等方法转成数字
    * 高精度转低精度自动截位, 低精度不能隐式转高精度
    * 位运算符方法 shl左移/shr右移/and与/or或/xor异或/inv反向
    * 数组可以通过arrayof()函数创建, 或者Array工厂函数去循环创建, 下标取值, 从0开始, 长度固定
* 可以类型推断, 可以省略一些类型声明
* 编译时常量使用const修饰, 不能写到局部变量(函数内运行时才能执行赋值), 需要配合val声明
* 可以使用模板字符串, 和flutter一样, 用$x显示变量, ${}包裹表达式即可

## 表达式(有返回值)
* if也是表达式, 返回代码块最后一行
* range表达式, 使用..表示范围, 可以配合in表示在范围内, 包括开始包括结束
* when表达式, 类似于java的switch/case/break, 条件可以逗号分隔, else相当于default
* run表达式, 固定this指向, 返回最后一行或者指定return
* applay表达式, 固定this指向, 返回this本身

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

* lambda匿名函数, 注意不是js的=>, 而是->
    * 函数类型中返回值也是->