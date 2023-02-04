# flutter
* 谷歌开发, 开源免费, 跨平台(ios/android/mac/windows/linux/web), 高性能, 语法稳定
* jdk/android studio/flutter sdk/镜像/flutter插件
* 工程根目录下 flutter run [-d 运行平台/all]
    * flutter drivces 查看可用驱动
* android studio打开android目录
* 控制台快捷键: r 热加载, R 热重启, p 显示网格, o 切换ios/android预览, q 退出
* vscode: alt + shift + f 格式化

## 项目结构
* 建议使用android studio创建应用, 控制台flutter create xx创建的包名不是自定义的
* android/ios/linux/macos/windows/web 对应不同平台资源文件
* lib 代码存放路径 lib/main.dart 入口文件
* pubspec.yaml 依赖配置
* analysis_options.yaml 分析语法规范

## 入口
* 首先引入material, 在main调用runApp启动组件
* 一般使用MaterialApp作为根组件, 设置主页(home)等
* 使用Scaffold组件设置导航(appBar), 内容(body), 抽屉, 弹窗等

## 组件Weight
* ctrl点进去看构造方法看参数
* 使用key进行唯一标识
* 自定义组件
    * 继承StatelessWeight无状态组件, 实现build方法, 返回一个组件
    * 继承StatefulWeight有状态组件, 实现build方法, 返回一个组件

### Container
* 容器组件, 类似div

### Center
* 居中组件

### Text
* 文本组件

### Image
* 图片组件, 可以加载本地(asset)/远程(.network)图片
* 通过fit剪切图片
* 本地图片存放到images路径下, 以及2.0x/3.0x文件夹下存放, 在pubspec.yaml配置assets
    * 访问时直接asset('images/xx')

### SizeBox
* 占位组件

### Divider
* 分割线组件

### ClipOval
* 椭圆形状, 可以设置宽高定义成圆形

### Icon
* 图标组件, 内置图标
* 自定义图标(例如阿里巴巴图标库)
    * 下载图标的字体, 放入项目fonts文件夹
    * 在pubspec.yaml配置fonts来引入下载字体
        * 设置family名字, fonts.asset指定ttf文件目录
    * 创建IconData类, codePoint在json文件可以拿到(0x的16进制)
    * 使用时传入Icon组件

### ListView
* scrollDirection指定水平/垂直列表, 子container直接加高/宽无效, 会撑满, 可以在ListView外套container设置高/宽
* ListView.builder可以传入循环次数和循环产生item方法来渲染表格
* 可以配合ListTile实现列表效果
    * leading可以在项目前加组件(例如加图标), trailing可以在项目后加组件

### GridView
* 网格布局组件
* count: 设置主轴数量和子组件, 到数量换行
* extend: 设置子元素最大长度和子组件, 按最大长度试图铺满然后换行
* builder: 设置itemCount循环次数, gridDelegate样式(区分count/extend)和itemBuilder构建方法

### Padding
* 边框组件, 功能单一, 相较于container效率高

### Row/Column
* 水平/垂直组件, 子组件排列一行/一列, 默认铺满行/列
* 可以设置主/次轴排列方式控制间距和位置(相对于当前容器/外容器)
    * 外层容器可以设置double.infinity无穷大/double.mixFinite很大, 保证子元素居中

### Flex
* 弹性布局组件, Row/Column都继承自Flex
* 配合Expanded组件使用, flex属性控制占比, Row/Column也能使用
    * 普通组件+Expanded可以实现固定+自适应布局

### Stack
* 层叠布局组件
* 配合Position实现绝对定位
    * 相对于外部容器定位, 没有外部容器就相对于屏幕定位
    * 如果嵌套了Row, 那么需要指定宽度, 不能使用无限大, 可以使用FlutterMediaQuery取得屏幕大小
    
### Align
* center组件继承自Align, 控制位置的组件
* alignment除了可以传枚举, 还可以Alignment(num, num)

### AspectRatio
* 控制容器宽高比的组件

### Card
* 卡片组件

### CircleAvatar
* 圆形组件

### 按钮组件
* 大小可以设置容器大小

#### ElevateButton
* 突起按钮, onPressed点击事件, .icon命名构造函数带图片

#### TextButton
* 文本按钮

#### OutlinedButton
* 带边框按钮

#### IconButton
* 图标按钮

### Wrap
* 主轴放不下换行/列

### 有状态组件
* 能重新渲染的组件
* 通过setState更新数据并反应到画面(重新build组件)