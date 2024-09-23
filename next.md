# next
* 基于15.0.0版本的next

## 01

### init
* 为什么是next而不是react
    * react官方一直在推的框架
    * 和react差异
        * 全栈: 内置api支持
        * 服务端组件: 服务端渲染, 更好的seo(更容易被百度到)
            * 默认都是服务端组件, 切换成客户端组件使用'use client'声明
            * 客户端组件可以当作传统react组件
        * 路由: 内置基于目录的路由功能
* 为什么要做monorepo, 为什么用Turborepo
    * monorepo就是把多个项目放在一个代码库里进行管理
        * 优点: 共享代码和依赖, 一致性, 快速部署
        * 缺点: 安全性问题, repo过大
    * Turborepo和next都是vercel的产品
* 初始化
    ```sh
        npx create-turbo@latest
        # 建议全局安装
        npm i turbo -g

        # 禁用检测功能, 屏蔽error log
        npx next telemetry disable
    ```
* debug
    * vscode配置launch.json来debug服务端组件
    ```json
        {
            "name": "Launch Program",
            "program": "node_modules/turbo/bin/turbo",
            "args": ["dev"],
            "request": "launch",
            "skipFiles": [
                "<node_internals>/**"
            ],
            "type": "node"
        }
    ```
    * [安装tool来debug客户端组件](https://chromewebstore.google.com/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi?hl=en&pli=1)

### turbo
* 默认开启缓存, 文件不变时会hit-cache
* 目录结构
    * apps: 工程代码
        * 采用next框架
            * ts + app_router
            * 未安装tailwind
    * node_modules: 所有工程的依赖
    * packages: 共享库
        * package.json
            * name: 包名, 用于import时匹配
            * exports: 为暴露的文件路径起别名
            * imports: 为文件路径起别名, 可以用通配符, 可以用于引入的路径
    * package.json: 
        * workspaces: 指定目录下, 带有package.json的文件夹作为包
    * turbo.json: 配置task
        * 通过turbo命令可以直接调用
* 配置
    * 全局
        * ui: tui能交互, stream只能输出
        * globalDependencies: 参数文件如果产生变化时miss-cache
        * globalEnv: 环境变量参数变化时miss-cache
        * globalPassThroughEnv: 除了env/globalEnv, 可以在代码中使用的非.env文件声明环境变量
    * task
        * 直接命名是针对所有子工程的script, 可以通过packName#scriptName针对指定工程
        * cache: 可以设false禁用缓存
        * env: 可用环境变量, 变化时miss-cache, 支持*通配符和!排除
        * passThroughEnv: 可用环境变量, 变化时不影响hit-cache
        * input: 输入的内容, 可以通过!屏蔽指定文件对hit-cache的影响
        * output: 要缓存的输出内容的目录
            * 指定目录后, 在hit-cache后会恢复该目录
        * dependsOn: 依赖项, 会先执行依赖
            * 可以用 packName#scriptName 指定依赖
            * ^表示递归依赖来执行该任务
            * 直接写表示同级的任务
        * outputLogs: 日志等级
        * persistent: 持久任务, 防止被依赖, 默认接受输入
            * interactive: 接收输入
* 运行参数
    * --workspace=xx 安装包时指定安装在哪个子工程
        ```sh
            # 实际只会存在一个, 目的是为了避免幽灵依赖
            npm i axios --workspace=docs --workspace=web 
        ```
    * --filter 只在对应工程执行script
    * --force 强制执行
    * --dry 查看调试log

### env
* 存放在根目录, 通过后缀区分环境, 对应NODE_ENV的值
    * 所有环境共用.env
    * 只能区分3个环境
        * development
        * test
        * production
    * .local优先级更高, 且被gitignore了
        * 工程启动时会显示使用的环境变量, 优先级从前到后
            ```log
                - Environments: .env.development.local, .env.local, .env.development, .env
            ```
* 通过process.env来使用
    * 默认只能被服务端使用, 添加NEXT_PUBLIC_前缀可以被客户端使用
    * 可以运行时修改环境变量的值
* 变量的值可以换行(双引号扩起来)
* 变量的值可以通过$引用其他变量的值(取得优先级最高的值, 而非当前文件值)
    * 运行时修改环境变量的值不会影响$的取值
* 不建议使用next.config加环境变量

### next.config
* distDir: 打包文件保存目录
* basePath: 配置url的根路径, 必须斜线开头
    * 图片的src需要对应调整
* images: 图片加载的配置, 配合cdn, 比如可以接入aws的cloudfront
* output: 打包配置
    * standalone, 打包时自动加入node_module依赖, 可以直接启动server.js
* headers: 针对指定请求设置响应头, 常添加一些安全性相关, 比如跨域, iframe, xss相关头
    * 可以用:xx作为占位符, *做通配符
    * 可以用has/miss判断请求的k/v
    * 可以配置locale屏蔽国际化的url
* redirects: 重定向请求
    * 支持通配符, 查询, 国际化
    * 注意basePath, 不想自动拼接时设false
    * permanent false307 true308
* rewrites: 重写请求, 作用于客户端
    * 外部映射可以避免跨域问题
    * 支持通配符, 查询, 国际化
    * 注意basePath, 不想自动拼接时设false
* serverActions: 修改bodySizeLimit限制请求大小(默认1m)

## 02

### react hook
* react(18.3.1/19-beta)内置的hook有下面17个
* 严格模式dev环境hook操作会连续执行2次来排查错误
* 规则
    * use开头
    * 写在顶层(不能在循环,if,事件处理等等里面)
    * 只在函数组件和自定义hook里使用hook(不在普通函数使用)

#### 状态
* useState(让函数组件拥有状态)
    * 参数是状态的初始值
        * 参数可以是无参函数, 返回值作为初始值
            * 注意不要直接调用函数
        * 初始化值只生效一次, 可以用于记录第一次传入的props(后续变更无法察觉)
    * 返回状态当前值和更改状态方法(setState)的数组, 常用解构取值
    * setState可以直接写新状态, 也可以写函数接受当前状态返回新状态
        * 更新状态是放入队列然后异步更新, 直接取拿不到最新值, 可以在setState的函数参数的参数里拿到
            * 尽可能不用flushSync
        * 新旧状态一致(Object.is)时不会重新渲染 ~~比较引用,所以push数组再返回原数组无效~~
    * 渲染阶段(顶层代码)只能 ***有条件的*** 调用当前组件setState, 不能调用其他组件setState ~~避免死循环,避免产生副作用~~
    * 定义时候规范
        * 尽可能少的状态数量 ~~相关的状态合并成一个, 能从其他状态计算得出(拼接,筛选)就不要单独定义, 防止改漏了~~
        * 尽可能浅的状态结构 ~~过深将导致更新state要写很多的...,扁平化~~
* useReducer(状态管理的聚合)
    * 参数
        * 参数1: 是一个方法, 方法参数是(当前)state和action, 根据action来更新state并返回更新后的state
        * 参数2: 初始值, 或者计算初始值的方法
        * 参数3: 可选参数, 有值则作为参数2方法的参数计算初始值
    * 返回当前状态和更新状态的方法(dispatch)的数组, 常用解构取值
        * dispatch用于触发参数1的方法, 传入action
    * 特性和setState一致, 注意不要写异步请求 ~~渲染时执行,需要是纯函数~~
    * 针对于state的改修逻辑很多时才建议使用reducer, 逻辑分离单独调试会更方便
    * [use-immer库](https://github.com/immerjs/use-immer): 对useState的包装, 封装了对象的变更操作 ~~屏蔽了数据不可变性,容易带坏小朋友~~
* useOptimistic(乐观更新UI)
    * 乐观就是先把ui给更新了, 但是异步操作没结束(db还没插进去)呢, 等异步结束之后再更新ui
        * 乐观更新适用于成功率极高, 可以撤回的场合 ~~抢票就不行,先提示成功,再出失败就气死了~~
    * 参数
        * 参数1: 初始值
        * 参数2: 是一个方法, 方法参数是(当前)state和待更新的值, 返回乐观state(临时副本)
            * 注意是临时副本, 原state并未变化, 所以是可撤回的, 同时异步操作正确结束时也可以更新正确的值
    * 返回临时副本和更新状态的方法(addOptimistic)
        * addOptimistic的参数就是参数2的待更新的值

#### 上下文
* useContext(数据共享)
    * createContext: 创建上下文, 参数提供默认值 ~~使用默认值没什么意义,全局变量一样的效果~~
    * useContext: 参数传入context, 返回(最近的父级provider)值 ~~避免层层props传递的麻烦~~
    * context.provider: 覆盖默认值, 可以多层覆盖(包括多provider)
    * 下层修改数据: 虽然setState也可以, 常用dispatch
    * 使用事项
        * 传值方式适合场景
            * 短传递用props
            * 单个孙子层使用的话用children参数 ~~比如dialog框周围那些~~
            * 多层都要使用, 使用的值可以切换的时候用context
        * 按照业务拆分context, 管理和使用更方便, 使用时再拼接成对应业务的provider组件
        * 使用memo缓存中间层, 不会影响孙子拿到最新context重新渲染


#### 副作用
* useEffect(模拟生命周期)
    * 副作用: 除了本身要做的事, 还产生了额外的效果 ~~比如页面初始化渲染,触发查api/操作dom~~
    * 参数1为依赖发生变化时(Object.is)要触发的函数, 参数2为依赖数组
        * 参数1
            * 渲染后执行 ~~少用,影响效率,后续章节展开~~
            * 不能是异步的
            * 使用的外部变量要添加依赖, 配置了lint的话会提示
                * [eslint-plugin-react-hooks](https://www.npmjs.com/package/eslint-plugin-react-hooks-rc)
                * 用函数式setState隐藏state依赖
            * 返回值为函数, 在组件卸载时触发
                * 刷新时, 先执行返回函数, 再执行参数1
                * 清理计时器, abort请求, 移除事件监听
        * 参数2
            * 不传时, 每次渲染都会触发
            * 传递空数组时, 只有初始化会执行
            * 数组有值, 内容变化时触发
    * 常用于初始化时候发请求, 操作dom
        * 真正唯一的操作应该放在文件头上 ~~比如通过storage记录用户是否第一次访问~~
* useLayoutEffect(不可见的渲染)
    * 重绘前触发的useEffect, 会影响性能
        * 虽然重新渲染, 但是用户看不出来
        * 依赖渲染出来的内容更新ui时才会用到
* useInsertionEffect(css-in-js)
    * 为了在dom上添加style标签导入css
    * 参数和useEffect相同, 执行顺序为 useInsertionEffect > useLayoutEffect > useEffect
    * refs无效, setup/cleanup同时触发

#### ref
* useRef(保存不用于渲染的值)
    * 用于保存不需要渲染的值 ~~改变值不会重新渲染,区别state~~
        * 相较于普通局部变量, 渲染不会重置
        * 相较于state, 不会影响页面渲染
        * 相较于全局变量, 每个组件独立
    * 参数是初始值, 返回的对象永远不变, 只会变更current
        * 渲染期间不要用ref的current
    * 为dom的ref属性赋值, 可以通过current拿到dom
        * ref属性可以接收函数参数, 手动保存dom
        * 可以上层创建ref, forwardRef传递给子组件的dom
            * 基本组件才可以这么做, 复杂组件别做, 不然行为不好预测
* useImperativeHandle(子组件为ref.current赋值)
    * 配合forwardRef
        * 不再直接把ref放入dom, 避免暴露整个dom, 而是暴露自定义的方法
    * 参数1是ref, 参数2是方法, 参数3是参数2方法的依赖数组
        * 参数2的方法无参, 返回值作为ref的current

#### 性能
* useMemo(缓存结果)
    * 参数1是无参函数, 参数2是依赖数组
    * 返回参数1计算后的值, 依赖不变则返回值不变
    * 使用场景: 计算量极大, 配合memo, 自定义hook, 或者作为别人的依赖
* useCallback(缓存函数)
    * 参数1是要缓存的函数, 参数2是依赖数组
    * 返回一个函数, 用于保证渲染时, 依赖不变则函数不变, 不然每次都是一个新的
    * 使用场景: 配合memo, 自定义hook, 或者作为别人的依赖
* useDeferredValue(包装state)
    * 参数1是要延迟更新的变量, 参数2是可选的初始值
    * 使用该值的ui更新会变成非阻塞的, 配合memo降低卡顿 ~~适用于更新很慢的组件,告诉react先不同管我,我后面自己慢慢同步~~
        * 后台渲染, 所以非阻塞
            * 后台渲染过程中更新值, 会放弃这次渲染来使用新值计算
            * 放弃的后台渲染不会触发effect
        * 新ui出现时间取决于react渲染速度, 也就是机器性能
    * 只是ui层面的节流, 不能减少网络请求次数
* useTransition(包装setState)
    * 没有参数, 返回数组, 第一项是状态是否更新完, 第二项是一个函数startTransition
    * startTransition参数是无参方法
        * 参数方法会立即执行
        * 参数必须直接调用setState ~~不能套setTimeout,要套就套外面~~
    * transition过程中被其他地方setState, 会跳过transition直接渲染state


#### 不常用
* useId(唯一id)
    * 无参, 返回一个唯一的id
        * 重新渲染不影响id的值
        * 调用多次返回不同id
    * 组件内使用可以自己拼字符串, 相较于多次useId可读性更高一点
* 自定义hook
    * 遵守hook规则, 封装状态和逻辑
        * 和共通方法对比, 维护了状态用于更新ui
        * 和组件对比, 可以暴露修改状态的方法
* useDebugValue(debug)
    * 自定义hook时, 在tool里显示值
    * 参数1是要显示的值, 参数2是对参数1的格式化方法
* useSyncExternalStore(使用外部状态)
    * 自定义hook时, 取得外部状态
        * react内部的状态用state, 外部的状态使用该hook, 比如其他库, 浏览器api
    * 参数1是一个方法, 接收callback并在状态变更时调用, 参数2取得数据的方法, 参数3是服务端数据初始值
    * 返回值是当前状态
* useActionState(form+state)
    * 参数1是一个方法, 接收上次state和表单数据并返回最新state, 参数2是初始化state, 参数3是url ~~js加载前就submit, 做重定向~~
    * 返回值是数组, 第一项是state, 第二项是formAction

### rule
* 多用children
    * 中间刷新不影响children
* 数据
    * 局部数据不要提升到全局
    * 按照业务拆分context
    * 通过setState减少依赖
* 纯粹性
    * 尽可能保证组件是纯粹的, 组件只做渲染
        * 渲染阶段不做有副作用的事
            * ~~只要管好自己这摊事,只影响自己渲染, 不去更改外部数据和dom~~
    * 减少useEffect的使用 
        * ~~classname和 document.getElementById('test')?.classList.add('xxx') ~~
        * ~~setState减少子组件渲染次数~~
* 幂等性
    * 要保证每次调用组件得到的结果是一致的
        * react无法保证组件执行顺序
        * 非幂等不要在渲染阶段做, 可以放到副作用里 ~~取当前时间,随机数~~
        * 不要在组件文件‘藏’数据 ~~组件文件内,在组件外定义数据~~
* 不可变性
    * 不要在组件内部改动非局部数据, 保证数据不可变和可追溯性, 避免副作用 ~~子组件直接push数组~~
        * props: 使用父组件回调
        * state: 使用setState
        * context: 使用对应dispatch
    * hook也不应该修改参数和返回值 ~~可能导致缓存错误~~
    * jsx表达式可能在渲染前执行, 所以也不要在jsx使用过之后改变数据 ~~没遇到过~~

## 03

### app router
* app router在next13开始使用, 文件存放app目录下, 基于文件系统的路由
* app下每个文件路径对应url的路径 
    * 特殊路径比如路由组和动态路由后续再介绍
    * 并非所有路径都能访问
        * api下有route的文件夹
        * 其他有page的非私有文件夹
            * _开头就是私有文件夹
* 所有文件默认都是服务端组件
### 基础页面
* page
    * 用于显示页面(后缀是js,jsx,tsx都行, 后续的其他路由结尾同理, 省略后缀)
    * 直接返回一个默认的函数组件即可显示
    * 可以获取searchParams(kv)
    * 客户端组件通过 usePathname 获取当前路由
### 切换路由
* 通过Link组件切换
    * 渲染成a标签, 可以传href和children, 可以带锚点
        * 可以通过props禁用锚点
    * 可见时预加载link的内容 ~~代码分割+按需加载代码~~
        * 可以通过prefetch禁用
        * 打包才能看效果
* useRouter
    * 客户端使用, 无参, 返回路由对象
        * 注意从 next/navigation 引入
    * history相关
        * push: 参数为url, 向history加一条
            * 可以禁用锚点
        * replace: 参数为url, 替换当前路由
            * 可以禁用锚点
        * back: 无参, 从history倒退
        * forward: 无参, 从history向前
        * 不建议手动操作history, 避免混淆push和pushState
    * refresh
        * 刷新服务端组件, 不影响客户端组件
        * 清除预加载的缓存
            * f5也会失效
    * prefetch
        * 参数为url, 手动预加载
        * 同样只有打包才可用
    * back/forward默认记录滚动位置
* redirect
    * 服务端使用, 参数为重定向的url, 类型可以为replace/push
    * 不能放try里
### 共享UI
* laytout
    * 切换路由时, layout会进行保留, 不会重新渲染
        * Link组件会保留, 通过a标签切不行
        * 直接使用usePathname会重新渲染, 可以单独起一个客户端组件去做然后引入layout
    * 也是函数组件, 接收children
    * 根布局必须存在, 且包含html和body元素
        * 默认的根布局还提供了metadata配置
            * 其他layout也可以改metadata
        * 且只有根路由能有html和body元素
    * 拿不到searchParams
* template
    * 切换路由时, template会新建
        * 所以可以用于重置状态
    * 也是函数组件, 接收children
        * 在layout和page之间
    * 拿不到searchParams, 可以拿到usePathname
### 加载效果
* 路由级别
    * loading
        * 在template和page之间
* 组件级别
    * 通过Suspense动态加载, 通过fallback配置loading效果
### 错误页
* error
    * 用于处理意外的错误, 必须是客户端组件
    * 可以接受error和reset参数
        * 打包后拿不到完整error
        * reset用于重新渲染
    * 可以多层处理, 从下向上处理
### 404
* not-found
    * app目录下的全局拦截
### 路由组
* 把一组路由放在一个文件夹下归类, 这个文件夹不会出现在url中
* 小括号括起来
    * 不同的组可以有不同的layout
    * 组下面的page不能导致路由冲突
* 可以嵌套
### 动态路由
* 中括号括起来
* 文件夹名就是key, 通过props.params.key取值
* layout和page能拿到, template拿不到
* generateStaticParams
    * 暴露该方法, 返回params数组, 这样就会在build阶段做缓存
    * 暴露dynamicParams为false可以禁用未定义的参数, 返回404
* 支持多级嵌套
* 支持扩展运算符
    * 多层只能写在最后
    * 可以用双中括号加扩展运算符表示可选参数
### 平行路由
* layout里显示多个内容 ~~具名插槽~~
    * 如果有template, 每个page都会带上
* @开头的文件夹
    * 可以写layout
* 插槽内也可以有路由结构, 和当前url匹配
    * 不匹配时, 取决于触发方式
        * 通过link迁移时, 插槽会保持初始路由
        * f5时, 会渲染default
* 在平行路由层级可以开始写下列文件, 会优先使用下级的
    * default
    * loading
    * error
* 在layout/template可以用useSelectedLayoutSegment传插槽名, 拿到该插槽当前路由
### 拦截路由
* 在当前路由下通过(.)/(..)/(...)去拦截, 和直接访问该url达到内容不一致的效果
    * 不是按文件夹算, 是按照实际路由算
    * ...是app目录
    * 可以嵌套
### api
* route
    * 一般放在api下, ts/js
        * 不放也行, tsx也行
        * 但是不能和page同级 ~~都是根目录,会冲突~~
    * 暴露 POST(create)/GET(read)/PUT(update)/PATH(delete)
        * 也支持 PATH(局部更新)/HEAD(没body)/OPTIONS(查询支持的方法)
        * 不能返回default
    * 可以从参数拿到请求数据(header,cookie,fordata之类), 返回NextResponse对象 ~~cookie的一些方法~~
        * 表单数据为异步, 配合post请求
        * req.nextUrl.searchParams.get('key')获取参数
        * 暴露dynamic为force-static会导致拿不到参数
        * 返回值也可以写code和header
        ~~fetch('http://localhost:3000/aaa/1', {method: 'post'})~~
        ~~'Access-Control-Allow-Origin': '*'~~
    * 可以配合路由组,动态路由
        * 获取params
    * 可以重定向
### 中间件
* middleware
    * 根目录下的全局拦截(注意不是app目录), 在匹配到page前进行处理
    * 一些拦截处理, 比如认证,log,响应处理等轻任务
    * 暴露middleware方法写处理逻辑, 暴露config用于指定配置
        * middleware方法:
            * 接收request参数
                * 由于全局唯一, 不同的路由逻辑自己写判断去做路径匹配
                * 通过NextResponse.next额外处理并放行请求, 或者直接响应请求, 或者重定向
                    * 重定向用NextResponse.redirect, 且不能是相对路径, 可以在url上clone去改pathname
        * config.matcher: 哪些路由要进入处理逻辑
            * 可以是字符串, 数组, 使用has/missing逻辑的对象, 正则
            * 不配就是全局有效
    * 只支持Edge的运行时, 某些api不好用(比如eval)
### 国际化
* 不同语言做到不同的动态路由下
* 中间件自动重定向

## 04
* data相关

### server data
* api
    * 异步调用fetch, 并异步转成json来使用
    * 纯静态页面会在build时调用fetch来预构建 ~~避免每次都调用api,适用于不怎么变的数据,比如邮编,电话区号~~
        * fetch设置no-sotre来禁用构建时调用
            * 只会隔离那行之后的代码, 之前的还是会执行并且访问时再次执行
        * 可以在组件内执行unstable_noStore方法来禁用构建时调用
            * 也是代码级别
            * 能catch到异常
        * 暴露dynamic为force-dynamic也可以达到禁用效果
        * 一个文件级别, 两个代码级别
* db
    * 纯静态页面去查db(mysql,prisma)也会预构建
        * mysql
            ```sh
            npm i mysql2
            ```
            * createConnection配置连接
            * connect开始连接
            * query进行查询
        * prisma
            ```sh
            npm i prisma
            npx prisma init
            # 修改配置文件, 指定参数
            npx prisma db pull
            npm i @prisma/client
            npx prisma generate
            ```
            * 通过PrismaClient实例取值
        * unstable_noStore和dynamic同样可用
* 利用并发提前渲染
    * 利用fetch的缓存特性, 提前去触发子组件的fetch
        * 因为函数组件的return才是渲染的地方, 写在await后会在父组件异步结束后再渲染子组件
    * 配合promise并行
* 数据安全
    * 通过server-only库来防止客户端使用
        ```sh
        npm i server-only
        ```
        * 引入不会, 引入并实际使用才会抛异常, 上层传入也不会
    * 为数据打污点, 防止流入客户端
        * react的实验性api, 需要在配置文件开启
        * experimental_taintObjectReference, 接收errMsg和对象, 无返回值
            * 只会处理对象的引用, 对象的值不会处理
        * experimental_taintUniqueValue, 接收errMsg, obj和obj.val, 无返回值

### client data
* 从api取数据时, 可以用effect在初始化时候fetch
* 三方库来包装请求
    ```
        # 先降一下版本, 都是18.2的react依赖
        # "19.0.0-rc-f994737d14-20240522" => 18.2.0
        # 15.0.0-rc.0 => 14.1.4
        # 添加rewrite避免跨域, 调整api的代码让重复请求效果不一致
    ```
    * [swr](https://swr.vercel.app/zh-CN/docs/getting-started)
        * 通过useSWR去请求, 提供缓存, 重发等功能
            * 参数1(key)为的值传入参数2的方法, 参数2(可选)是实际发请求的方法, 参数3(可选)是配置项
                * 参数1: 
                    * 缓存的key, 判断相同使用值而非引用 ~~可以预请求加速~~
                    * 广义false或者通过方法抛异常都不会触发这个请求
                        * key使用上一个请求拿到的数据来触发异常 ~~可以用于等待依赖项~~
                * fetcher: 可以省略, 写到配置项里, 返回promise对象或者value
            * 返回值是个对象
                * data: 查询的数据, 没查完是undefined
                * error: fetcher抛的错, 没错是undefined
                * isLoading: 是否在没缓存情况下请求
                * isValidating: 是否重新验证
                    * 区别于loading, key变了就重新验证, 但是命中缓存就不会loading
                * mutate: 更改缓存的方法, 参数1是新数据或者方法(返回值作为新数据), 参数2是配置, 返回值是新数据或者新数据的promise
                    * revalidate: 可以禁用默认的重发效果
                    * optimisticData: 乐观更新ui
        * SWRConfig进行配置共有, 类似context
            * 多层使用时下层最优先
            * 可以通过函数形式拿到上层配置
                * 值为对象时, 不是整体覆盖, 可以做merge
            * provider为缓存对象, 不提供则使用上层的
            * useSWRConfig可以获取该配置
                * 可以通过cache拿到缓存对象
                * 通过mutate更改缓存和局部的mutate效果一致
        * 参数3的配置项:
            * onSuccess: 拿到data的回调
            * refreshInterval: 自动刷新间隔 ~~最小2s~~
                * revalidateOnFocus: (浏览器页面级别)焦点变更时重发
                * revalidateOnReconnect: 断网重连后重发
                * revalidateIfStale: 有缓存也重发
                * focusThrottleInterval: 焦点变更截流
                * dedupingInterval: 请求截流
            * onError: 异常回调 ~~可以写到全局~~
                * shouldRetryOnError: 错误时重试
                * errorRetryInterval: 第一次重试间隔
                * errorRetryCount: 错误时重试的最大次数
            * loadingTimeout: loading超时
                * onLoadingSlow: loading超时回调
            * fallback: kv形式的默认值
        * ~~还提供了无效滚动和取websocket的hook, 用的不多, 有需要自己看文档就行~~
    * [react-query](https://tanstack.com/query/latest/docs/framework/react/installation)
        * 和swr差不多, 也是key+fetcher+options形式的hook
            * 外面的provider是必须的

### server action
* 运行在服务端的函数
    * 和普通函数区别在于, 运行在服务端, 调用在客户端
        * 类似api, 但是并不是以url形式调用, 而是直接传递 ~~更安全~~
        * 也要注意认证过期问题 ~~放置很长时间再提交, 过期了重定向到登录页~~
            * 过期了做重定向或者抛error的处理
                * 事件/副作用调用会被catch, form的action会进入error页
    * 使用方式
        * 直接import或者传入
        * 传入form的action,响应事件或者副作用
            * 本质是网络请求, 返回promise ~~即使未定义成async~~
        * 服务端组件非export声明时, 如果传递给form, 函数内不能省略use server
            * 即使文件级别声明了use server
        * hidden可以传一些额外参数
            * disabled不会提交
        * 默认post提交, 不会切换路由
            * 会重新build
    * form
        * 直接传到action没法做处理loading和返回值
            * post过程state不会更新到ui
                * 配合useOptimistic处理loading
                * 配合useActionState处理loading
        * action想额外加参数可以用bind
            * 不能把方法作为参数
        * 除了submit提交
            * button的formAction属性也可以提交触发action
            * 通过dom.form.requestSubmit提交也能触发action

## 05
* cache

### react cache
* 参数为一个方法, 方法的返回值作为缓存内容
* 返回值为包装后的方法
* 用于避免重复执行耗时操作
    * 约等于 使用context取值 + 判断没值再执行
* 调用时如果参数一样, 则不会重新执行cache参数的方法, 会直接返回值
    * key判断引用(object.is)
    * 同key时, 因为方法不会执行, 第一次如果抛异常, 后续也会使用异常的缓存
* 每次执行包装都会返回新的包装结果
    * 包装放在组件外(避免重复包装)
    * 使用要放在组件内
* 配合服务端组件使用, 用于创建组件外的缓存
    * 区别useMemo: 客户端, 单个组件
    * cache: 服务端, 跨组件
* 影响缓存
    * refresh会清缓存
    * server action不会清缓存
        * 但是action内容执行特殊操作会清, 比如
    * 跳到新路由/f5会清缓存
        * 注意跳转后layout和template不会自动更新服务端数据, 所以用的还是旧数据
        * 操作history不会清缓存

### next cache
~~ 之前课程介绍fetch的缓存效果, 比如在上层组件提前去执行子组件fetch做加速渲染, 通过参数和配置禁用缓存等功能. 这节课会详细说一下fetch的缓存和原理. ~~

#### 请求层(next-fetch)
* 服务器组件渲染期间, 针对请求的缓存
    * 参数一致则不会再发出fetch请求
    * 可以传递signal来禁用
        * 用于中断请求
    * dev环境初次取值可以看出请求缓存的效果
* 源码解析
    * 被patch-fetch包装
    * 将fetch属性添加到cachedFetch上, 并赋值给fetch
    * cachedFetch
        * 配置了 *signal* 直接走普通fetch
        * 根据配置生成缓存key
            * 指定url且配置为空(即get请求), 固定字符串的缓存key
            * 不是get和head, 或者keepalive的请求, 直接走普通fetch
            * 否则根据配置项生成缓存key
        * 取缓存
            * 根据url取react.cache, 取不出来走普通fetch, 并把fetch的promise记录到缓存中
            * 通过key没取到缓存, 走普通fetch, 并把fetch的promise记录到缓存中
            * 有缓存则直接使用缓存的value来注册then回调并返回
            * clone一个response, 因为默认只能读一次
            ```js
            a = await fetch('https://exam.dlufl.edu.cn/info/1056/1700.htm')
            a.body.values()
            ```
    * dev只有初次进入next-fetch, 后续就会走process/pre_execution, 所以后续看不到缓存效果

#### 数据层(store)
* 针对数据缓存
    * 可以理解为服务端的swr
        * 如果有缓存, 先显示缓存, 不显示Suspense
        * 缓存过期时, 下次访问时在后台取新数据, 成功后缓存更新为新数据
            * 触发重验证的那次请求, 看不到最新值
    * build(静态)和访问期间(动态)都能触发
    * 配置项
        * cache: 默认开启缓存, 设置no-store会禁用
        * next:
            * revalidate: 数据有效期
                * 单位是秒
                * false永不过期
                * 动态路由不设置相当于0
            * tags: 用于清除缓存的key
    * 手动清除缓存
        * revalidatePath 路由级别缓存全部清除
            * 包括该路由上面的layout/template
        * revalidateTag 单个请求级别
            * 传入任意tag都可以
        * 触发api需要手动刷新(f5/router.refresh), action自动刷新
        * 只能在serverAction/api里执行
    * 非fetch的方法可以用unstable_cache包裹

### 缓存小节
* 路由级别也可以配revalidate, 影响page的重构
    * 不会影响fetch的缓存
* 客户端级别有layout的和history的缓存, 以及preload
* 使用不频繁的页面, 可以把Link的prefetch禁用
* 在服务端组件中
    * 方法的缓存: react.cache/unstable_cache
    * 利用请求级别缓存, 在父组件异步发起子组件的请求做预加载加速渲染
    * 对于数据几乎不变的页面, 直接做成静态的在build时预渲染, 动态路由通过提前指定参数的形式也在build阶段优化
    * 比如db的数据每天有batch来更新, 可以把这些数据的查询利用数据缓存做控制, 平时使用缓存, 在batch结束后触发一个api来清缓存数据做同步
    * 时效性要求不高的场合, 比如查看数据走势, 设置数据有效期, 减少频繁分析的性能损耗
    * 做数据修改的form, 对应action清缓存更新页面数据


## Next的渲染模式
* 静态渲染: 无动态内容 / force-static
    * 会先生成一版html, 在.next/server/app下
* SSG: 动态路由的静态参数
    * 根据参数生成对应html
* 动态渲染: 使用运行时数据 / unstable_noStore / force-dynamic / revalidate = 0

## 06

### 渲染的补充
* ppr: 动态组件的流式渲染
    * 修改config
    * 暴露属性
    * suspense
* 服务端组件
    * 异步的服务端组件要套在suspense里
    * 优点
        * 服务端发出请求, 服务器性能肯定比用户pc更好
        * 渲染后交给客户端, 减少js传输量, 渲染好的内容更好被爬虫检索, 预览效果也更好
        * 缓存在服务端, 减少客户端内存使用
* 客户端组件
    * 通过use client定义, 该组件下方所有子组件都自动变成客户端组件
        * 基于自动转换的特性可以做包装
        * 服务端和客户端组件可以传递数据
        * 不能在客户端组件内直接引用服务端组件
            * 先渲染服务端再渲染客户端, 所以客户端下的服务端没法渲染
                * 服务端先生产内容, 客户端决定摆放位置
            * 可以把服务端组件的jsx向下给客户端组件
                * 实际关联的是服务端组件而非客户端组件
    * 客户端组件不会影响静态渲染
        * 但是渲染的内容只是未携带动态内容的模板
    * 优点
        * 浏览器api
        * 支持交互
* 基于上述优点, 尽可能使用服务端组件, 只在局部使用客户端组件
* 数据共享
    * 服务端fetch/cache
    * 客户端swr/context

### auth
* 登录表单
    * 复习一下serverAction/useActionState/抽hook/memo
    * 离程序核心越远拦截越好
* 认证处理
    * 可以把用户信息存db里, 然后把加密后的key返回给浏览器
        * redis 高效+时效性 / Bloom Filter 提高验证效率
        * cookie 自动携带, 浏览器有大小限制
        * jwt 跨域+时效性, 需要自己带
    * 再次访问时进行校验
        * 中间件
        * 组件内根据用户权限做不同处理
            * 组件间共享缓存
        * serveraction/api也可以做校验

### [next auth](https://authjs.dev/getting-started/installation)
* 集成3方登录
    * 新建一个文件夹, 配置package
        * 和turbo(react19/next15)兼容性并不好, 一大堆bug
    * npx auth secret创建env
    * 初始化NextAuth, 配置对应provider
    * 创建route处理登录页
        * ts的路径别名
    * 验证处理
    * 创建signin/signout入口
        * 重定向参数
    * 服务端/客户端/中间件处理
        * src
    * 会自动刷过期时间


### 其他补充
* lint
    * 代码格式(换行,空格,符号,对齐)
    * 未使用的变量
        * 引用
        * 接收的返回值
* ts
    * 类型尽量清楚写出出来, 避免any和as
    * 类型生成
    * 类型采用大写
* js
    * 额外的方法包装
    * 三元 || ??
    * return undefined
    * 解构取值
* 框架
    * 逻辑抽到hook, 组件只负责渲染
    * effect
    * react19 use / context
    * swr.useSWRMutation
    * next out


