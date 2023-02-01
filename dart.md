# Dart

## 概念
* google开发的语言, 用于flutter开发
* 安装sdk, dart --version测试
* vscode安装dart和code runner
* 后缀名是.dart, main函数是入口

## 函数

### 标准写法
```dart
void main() {
    print('test'); // 打印
}
```

### void的返回类型可以省略, 其他类型不建议省略
```dart
main() {
    print('test');
}
```

### 一个表达式的函数可以使用箭头函数, 包含隐式的return
```dart
main() => print('test');
```

### 参数类型不建议省略
```dart
int getName(int a) {
  a++;
  return a + 1;
}
```

### 可选参数用中括号括起来放最后
```dart
test(a, [age]) {
  print(a);
  if (age != null) {
    print(age);
  }
  return a;
}
```

### 可选参数可以设置默认参数
```dart
test(a, [int age = 77]) {
  print(a);
  print(age);
  return a;
}
```

### 命名参数用大括号括起来, 调用时key:val形式
```dart
main() {
  test2(
    a: 3,
    b: 6,
  );
}

test2({a = 1, b = 2}) {
  print(a);
  print(b);
}
```

### 立即执行方法
```dart
((){
  print(2);
})();
```

### 闭包: 常驻内存且不污染全局变量
```dart
main() {
  var f = test();
  print('....');
  print(f());
  print(f());
  print('...');
}
test() {
  int age = 29;
  return () => age++;
}
```

## 数据类型

### var
* 自动推断数据类型
* 如果初始化时赋值了, 之后赋值时不能使类型不一致
* 初始化没赋值, 可以修改类型

### String
* 字符串
* 可以单引号, 双引号, 三引号(跨行)
* 可以模板字符串$变量, 或者${表达式}

### bool
* true/false

### 数字
* int/double/num, 其中int/double继承了num抽象类, 注意都是类
* double可以赋值整形
* 除法和js不同
```dart
int a = 1;
int b = 2;
print(a/b); // 普通除法, 均为int也会返回double
print(a~/b); // ~/ 是整除
```

### List
* 数组, [1,2,3]形式, 下标0开始, 元素类型可以不同
* 定义时, 可以指定泛型<T>, 限制元素类型
```dart
List list = <int>[1,2];
```
* List.filled(length, item), 创建一个固定长度的集合, 不能add了, 只能下表去修改内容
    * 支持泛型, list<T>.filled
* 手动修改length可以达到截取效果
* reversed属性返回倒叙的Iterable, 可以toList
* addAll 拼接数组
* remove/indexOf 都是针对第一个匹配项处理
* fillRange 修改指定区间内容, insert/insertAll 插入元素到指定位置
* where 匹配函数索引
* any 有一个满足条件就返回true, every 全满足返回true

### Map
* 字典, 键值对形式, key可以为变量
* 可以new Map, 也可以大括号直接初始化
* map[key]可以取值赋值
* keys属性取得所有key
* addAll会进行同名覆盖

### Set
* 不重复的集合
* addAll放入List来去重

### is
* 进行类型判断
```dart
print(1 is int);
```

### const/final
* 常量修饰符, 修改会报错
* 可以省略数据类型, 相当于隐藏了一个var去推断数据类型
* final相较于const, 是惰性初始化, 第一次使用前才初始化, 可以配合方法去初始化, 而const使用方法初始化会报错
* 引用型(List/Map)可以修改内容

### 类型转换
* int/double.parse(val), 空字符串会异常, 可以isEmpty判断空字符串, 注意是属性而不是方法
* val.toString(), 数字可以用isNaN判断异常数字
* val as int 强制类型转换

## 逻辑控制

### ==
* 不只是比较值, 会区分类型

### ??=
* 如果为null, 赋值, 否则不处理
* a = a ?? b 的简写

### if/else/else if
* 和js一致

### switch/case/break/default
* 和js一致

### 三目运算符
* 和js一致

### try/catch
* 和js一致

### 循环
* 普通for循环 for (int i = 0; i < 2; i++)
* 数组in循环 for (int a in list) 取得元素
* 数组forEach, map
* while/do while和js一致
* break/continue都是跳出一层循环, 配合label语法跳出标记层循环

## 类
* 所有对象都继承自Object
* 方法内通过this.属性名/方法名去获取属性/调用方法
* 属性/方法默认是共有的, 使用下划线开头标记私有属性/方法且作为单文件才有效
* 构造函数和类名同名, 可以直接简写成 CLASSNAME(this.PROPNAME) 来为属性赋值
    * 命名构造函数: CLASSNAME.MYNAME(){} 调用时 new CLASSNAME.MYNAME(), 参数同样可以简写
* import PATH 来引用其他类
* 通过get修饰符实现计算属性, 注意没有(), 只有{}或者=>
* set修饰符有参数
* 初始化列表: 写在构造函数()之后, {}之前的赋值语句, 在执行构造函数前初始化
* new关键字可以省略

### 静态
* static关键字, 修饰属性/方法, 非实例化直接访问, 跟java一样
    * 静态方法不能调用非静态属性/方法
    * 非静态方法可以访问静态属性/方法
* 方法内访问属性: 建议静态属性直接写属性名, 非静态通过this.属性名访问(也可以直接属性名, 不推荐)

### 条件运算符
* 在调用方法/取属性前?判空, 对象?.属性形式

### 级联
* ..运算符, 类似于链式语法而不需要方法返回原本对象
```dart
main() {
  A a = new A();
  a..log()..say();
}
class A {
  log() {
    print('log');
  }
  say() {
    print('say');
  }
}
```

### 继承
* 和Java一样extends, 也是@override去复写
* 由于构造函数无法继承, 需要执行super传值到父类构造函数(可以使用初始化列表形式)
* 可以super.name 去调用命名构造函数

### 抽象
* 和java一样abstract去抽象类, 但是没有接口关键字, 使用普通类/抽象类(建议用抽象类)去定义接口
* 方法不写方法体就是抽象方法, 而不需要abstract关键字
* implements实现要去实现所有属性方法, 而继承只需要实现抽象方法
* 只是约束子类, 用实现, 想要约束+调用共通方法, 用实现

### 实现多个接口
* implements后用逗号分隔多个接口

### mixins混入
* 因为可以多实现, 不能多继承, 用混入模拟多继承
* with关键字, 让被混入的类拥有源头类的属性和方法
* 混入源头的类父类必须是Object, 且不能有构造函数
* 混入类可以被其他类混入
* 一个类可以继承一个类并混入其他类
* 同名属性/方法会被最后一个覆盖
* 使用is去判断类型, 混入源都会返回true

## 泛型
* 保持类型检查, 同时放宽类型限制
* 返回值用字母声明(常用T), 方法名和参数中间使用<T>来声明泛型
* 调用时, 也是在方法名和参数中间使用<int>来声明, 不传入泛型相当于不做校验
* 类也可以使用泛型, 数组就是例, 可以写参数前, 也可以像java一样写到返回值类型中
* 泛型类接收到的泛型可以传到泛型接口中
```dart
main() {
  print(test('2'));
  print(test2<String>('2'));
  A<int> a = A(3);
  print(a.name);
}
class A<T> {
  T name;
  A(this.name);
}
test(t) {
  print('test');
  return t;
}
T test2<T>(t) {
  print('test');
  return t;
}
```

## 库
* 自定义库 import 文件路径
* 系统库 import dart:xx
* 第三方库 下载后按文档引用
    * package.yaml文件中的dependencies中维护, 执行pub get去下载依赖
    * https://pub.dev/packages
* 重名的库通过as去重命名
* 通过show去局部引用 import NAME show FunctionName, 通过hide去去除
* 通过deferred去延迟导入
* part是切片, export导出

## 异步
* 和js一样的async/await
* async加在参数和方法体之间

## 空安全
* 默认是非空变量, 赋值给null会报错
* 使用类型后接?是可为空变量

### 类型断言
* 防止操作空出现空指针, 使用!, 类似于js的?, 但是空还是会抛出异常

## late
* 延迟初始化属性, 非构造函数初始化的属性
* 抽象接口属性也可以用该关键字

## require
* 命名参数中使用, 标记该命名参数不为空
* 配和?去定义可空属性, 非空的用require标记

## identical(a, b)
* 判断是否指向同一个引用
* 通过const去调用构造函数, 指向是同一个对象(创建一个类的多个对象, 内存中只保留一个对象)
* 同样的数字, 指向也是一样
* 通过const定义的数组, 元素相同则指向同一引用
* 类中使用final修饰属性, const修饰构造函数, 这样在实例化时才能使用const去调用构造函数
    * flutter不会重新渲染const组件