# 静态项目脚手架

## 前置条件

1. node 使用12以上的 lts 版本

```shell
nvm install 12 --lts
```

## 安装脚手架

```shell
cd static-pack-cli
npm install
npm link
```

## 运行

### 开发模式运行

```shell
npm run start
```

### 生产环境编译

```shell
npm run build
```

> 注意：
>
> 给`测试环境`编译的代码**不可以**发布到`生产环境`，发布`生产环境`之前必须要==**重新编译**==，详细原因参考配置文件`promptDomainName`的说明。

### 自动提取国际化资源

一般我们在开发的时候，尤其是写 html 文件的时候，需要写大量的文本。如果每一个都去 locale 文件中添加的话，效率比较低。可以直接使用 `<Locale></Locale>` 来包裹固定文字，然后等开发完成后，再执行命令，一次性把这些文本自动提取到国际化资源中去。

```bash
npm run extract
```

例如，在页面中这样写：`<Locale>这是一段测试文本</Locale>`，然后执行 `yarn extract` 命令之后，就被替换成

```html
<Locale>index-html-d83dfb7107e13a65296452bdb57a6994644a528e72af9eacf0f55b417480852a</Locale>
```

同时，资源文件中会自动增加：

```json
{
    "index-html-d83dfb7107e13a65296452bdb57a6994644a528e72af9eacf0f55b417480852a": "这是一段测试文本"
}
```

### 导出增量的国际化资源

项目中可能已经存在很多国际化的资源了，当我们增量开发了一些功能时，这些未翻译的中文资源会和已经翻译的资源混合在一起，抽取这些增量的待翻译资源可能是一个很麻烦的事。如果把整个大文件丢给翻译人员，他（她）也很难受，要从数以千计的资源中找到中文资源很麻烦，也容易遗漏。

所以，我们开发了一个命令，可以把所有未翻译的资源都提取到一个文件中，这样翻译起来就非常方便了。翻译完成后，还可以自动导入到系统中（参见下一节）

```bash
npm run export
```

导出后的文件保存在 `./locales/未翻译.csv` 中，可以用 Excel 打开，打开后是类似这样的，

| Key                                          | zh-cn        | en-us        |
| -------------------------------------------- | ------------ | ------------ |
| index-html-00573500895ddacc8331e55455fe6687c | 工单管理系统 | 工单管理系统 |
| index-html-029dfc63f052d8e2899996a87d532a7c1 | 下载案例集   | 下载案例集   |
| index-html-00573500895ddacc8331e55455fe6687c | 阿里云代理   | 阿里云代理   |

1. 第一列 `Key` 是国际化资源对应的 key，在翻译过程中不能修改；
2. 第二列是中文列，主要用来参考的，不用翻译，在导入的时候也会自动忽略掉；
3. 从第三列开始是系统中支持的其他语言，这些是需要翻译的

### 导入国际化资源

在 Excel 中把上述待翻译的资源都翻译完成后，点击 `文件`，`另存为`，保存为 `逗号分隔值(.csv)` 文件类型。然后把文件拷贝到项目的 `./locales/已翻译.csv` 下，然后执行命令自动导入。

```bash
npm run import
```

导入成功后，中文以外的其它语言对应的文件会自动更新。

## 配置 package.json

如果当前项目还没有`package.json`，需要先初始化一下，然后一路回车即可。

```shell
cd example-project
npm init
```

打开`package.json`，添加下面这一段代码：

```json
"scripts": {
    "start": "npx static-pack-cli --dev",
    "build": "npx static-pack-cli",
    "extract": "npx static-pack-cli --extract",
    "export": "npx static-pack-cli --export",
    "import": "npx static-pack-cli --import",
    "all": "concurrently \"npx static-pack-cli --extract\" \"npx static-pack-cli\""
},
"devDependencies": {
    "webpack": "4.43.0",
    "webpack-cli": "3.3.11",
    "webpack-dev-server": "3.11.0"
}
```

安装webpack等依赖包，引入脚手架

```shell
npm install
npm link static-pack-cli
```

之后就可以执行编译、运行、提取国际化资源等命令。

## 添加配置文件

在主项目下创建`static-pack-cli.config.js`，参照下面代码导出一个`json`对象。注意：实例代码是一个最小化的配置，更多参数设置，可以参考说明文档，而且默认值需要酌情修改，不要完全照搬实例代码。

```js
module.exports = {
    distDir: path.resolve('./dist'),
    ignore: ['package.json', '.*', 'partial/**/*', '*.sublime-project', 'README.md'],
    gitignore: true,
    promptDomainName: true,
};
```

更多配置文件说明，请参考下面的表格。

| 名称                    | 类型            | 默认值                              | 说明                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| ----------------------- | --------------- | ----------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `distDir`               | `String`        | `path.resolve("./dist")`            | 项目编译后的输出目录，一般是 web 项目对应的 build 目录下的`dist`目录，例如：`path.resolve("../website_build/dist")`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| `ignore`                | `Array<String>` | 参见前面的代码实例                  | 设置需要排除在`distDir`之外的文件(或目录)列表。编译过程默认会拷贝 web 项目中的所有文件到`distDir`，但有些目录是需要排除在外的，比如`node_modules`等。此目录可以排除这些目录或文件。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| `gitignore`             | `Boolean`       | `true`                              | 是否把`.gitignore`文件中的内容自动合并到`ignore`数组中，默认是`true`，即会自动忽略`.gitignore`中指定的文件。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| `templateHtmlFolders`   | `Array<String>` | `undefined`                         | 指定作为模板文件动态编译的目录列表，默认会编译所有子目录下的`html`文件，但如果有些目录不需要动态编译，可以通过设置这个属性，只编译某些子目录。<br />_动态编译是什么意思？_<br />是指通过`gulp-file-include`插件提供的动态插入另外一个`partial` html 模块文件的特性。如果多个页面之间有一些`html`代码是重复的，可以抽离到一个独立的 html 模块文件中，然后通过`@@include('./partial-file.html')`指令动态引入。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| `partialFolderDir`      | `String`        | `partial`                           | 用来存放`partial`复用模块文件的目录名                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| `promptDomainName`      | `Boolean`       | `true`                              | 是否在编译过程中提示输入主域名，默认为`true`。<br /><br />_输入主域名有什么用？_<br />由于脚手架默认支持`CDN`，即会把静态文件引用路径前面自动添加`cdn`的域名，例如`//www-img-cdn.com`，因此要求输入主域名来设置绝对路径。<br />_为什么不能使用`/`相对路径？_<br />因为我们把不同种类的静态文件映射到不同的域名下，这样可以加浏览器加载静态资源的速度。在没有启用`http2`的情况下（或这不支持的浏览器中），一个域名下只能有`6`个左右的并发下载数，如果页面中静态文件比较多的情况下，就免不了产生排队，页面加载时间会被拖慢。如果把 js+css、图片、视频、字体等不同类型的资源分散到不同的子域名下，这样每种类型的资源就分别得到`6`个并发量，所以加载速度会得到提升。<br /> _为什么要提示手动输入？_<br />因为要把域名编译到`html`或`css`文件中，不能用`js`动态生成。一般我们修改了网站内容的话，会先发布到测试环境测试一下，没问题后才能发布生产环境。而由于`测试环境`和`生产环境`下的`cdn域名`是不同的，如果错把`测试环境`的代码直接发布到生产环境，就会导致生产环境下跑到测试环境下加载静态资源，而测试服务器是非常不稳定的，必然会造成线上故障，所以这一点切记切记！<br />而为什么默认需要手动输入呢？因为这样会比较方便，每次编译过程中直接输入`正确的`域名即可，否则的话，我们每次都需要修改配置文件，比较麻烦，更严重的是，如果我们忘了编译生产环境之前修改域名，就会自动编译错的域名，而我们也不会得到提示，可能发现不了问题。 |
| `defaultDomainName`     | `String`        | `domain.com`                          | 如果`promptDomainName`设置为`false`时，此属性设置`CDN`默认的主域名。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| `staticAssetsDnsPrefix` | `String`        | `//www-static-cdn`                  | 脚本类静态资源的`CDN`域名的前缀，与`defaultDomainName`两个值一块构成了完整的`cdn`域名。`www`代表了`官网`，所以如果是某个业务项目需要启用`cdn`支持的话，最好结构保持一致，例如：sg-static-cdn、bi-static-cdn。如果一个项目有多个站点需要 cdn 的话，可以再增加一级，例如：cem-static-cdn、www-cem-static-cdn                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| `imgAssetsDnsPrefix`    | `String`        | `//www-img-cdn`                     | 图片类静态资源的`CDN`域名的前缀，与`defaultDomainName`两个值一块构成了完整的`cdn`域名。`www`代表了`官网`，所以如果是某个业务项目需要启用`cdn`支持的话，最好结构保持一致，例如：sg-img-cdn、bi-img-cdn。如果一个项目有多个站点需要 cdn 的话，可以再增加一级，例如：cem-img-cdn、www-cem-img-cdn                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| `imageFileExtensions`   | `String`        | `jpg,jpeg,gif,png,bmp,svg,webp,ico` | 用来识别为图片类资源的文件后缀名，多个值之间用`,`分隔。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| `mediaAssetsDnsPrefix`  | `String`        | `//www-media-cdn`                   | 媒体类静态资源的`CDN`域名的前缀，与`defaultDomainName`两个值一块构成了完整的`cdn`域名。`www`代表了`官网`，所以如果是某个业务项目需要启用`cdn`支持的话，最好结构保持一致，例如：sg-media-cdn、bi-media-cdn。如果一个项目有多个站点需要 cdn 的话，可以再增加一级，例如：cem-media-cdn、www-cem-media-cdn                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| `videoFileExtensions`   | `String`        | `mp3,wav,mp4,wmv,mov,pfk`           | 用来识别为视频类资源的文件后缀名，多个值之间用`,`分隔。`videoFileExtensions`与`fontFileExtensions`都属于`media`类的静态资源。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| `fontFileExtensions`    | `String`        | `ttf,ttc,woff,woff2,eot`            | 用来识别为 web 字体资源的文件后缀名，多个值之间用`,`分隔。`videoFileExtensions`与`fontFileExtensions`都属于`media`类的静态资源。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| `enableSubCdn`          | `Boolean`       | `true`                              | 是否启用子 CDN 域名，如果启用的话，会根据静态资源的分类，使用不同的 CDN 子域名。如果设置为`false`，则不会对引用的静态资源地址做任何修改，仍然会从主域名进行加载。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |

## .env 配置文件

脚手架还支持通过`.env`(及`.env.development`、`.env.production`)设置一些配置内容，自己酌情设置。

| 名称 | 默认值 | 说明                                           |
| ---- | ------ | ---------------------------------------------- |
| HOST | 9100   | 在开发模式下，`webpack-dev-server`使用的端口号 |
|      |        |                                                |
|      |        |                                                |

# 使用示例

## Html

```html
<div>
    <!-- 访问多语言占位符，（方式一） -->
    <Locale>a-b</Locale>
</div>
<div>
    <!-- 访问多语言占位符，（方式二） -->
    $a-b$
</div>
```

```html
<!-- 1. 可以使用if语句 -->
<!-- 2. 当前语言的占位符 -->
<% if ($LANG$ === "en-us") { %>
<!-- 3. 访问多语言占位符，（方式一） -->
<Locale>a.b</Locale>
<%} else {%>
<!-- 4. 访问多语言占位符，（方式二） -->
$a.b$
<% } %>
<!-- 5. 访问json数据（全局+页面级别），pageData.xxx -->
<!-- 全局数据放在json目录下，全部文件自动合并；页面级别数据，存放在与html文件同名的json文件中 -->
<% if (pageData.users.length > 1) { %>
<h2>
    <!-- 6. 输出一个值 -->
    <%= pageData.appName %>
</h2>
<!-- 7. 可以使用lodash -->
<link href="cmps/bootstrap/dist/css/bootstrap.min.css" rel="stylesheet" />
<link href="css/style.css" rel="stylesheet" />
<% _.forEach(pageData.users, function(user) { %>
<!-- 8. 进行html编码 -->
<li><%- user.name %></li>
<% }); %>
<!-- 9. 使用for循环 -->
<% for (let user of pageData.users) {%>
<li><%- user.name %></li>
<% } %>
<!-- 10. 复用一段局部header.html文件，文件存放在partial目录下 -->
<%= partials.header %>
<!-- 更多格式： -->
<!-- https://lodash.com/docs/4.17.15#template -->
<!-- https://github.com/tj/ejs -->
```

## JS

```js
// 使用$a.b.c$来访问多语言资源
let title = '$labels.title$';
// 使用$LANG$访问当前语言
let currentLanguage = '$LANG$';
// output: en-us
```

## CSS

```css
.my-class {
    /* 与js访问多语言资源类似，但几个特殊符号需要转义($-.) */
    height: \$page\.admin\.user\.container_height\$;
    /* 输出： color: 60px; */
}
/* 使用 \$LANG\$ 插入当前语言，可以针对某种语言写特定样式 */
.my-class\$LANG\$ {
    /* 输出： .my-class-en-us */
    display: none;
}
```

## 其它

-   可以借助动态模板，实现一些“动态”渲染，把一些数组数据存放在 json 中，进行动态循环渲染，提高代码可维护性，
    -   可以通过 pageData 来访问。
    -   数据由两部分组成，全局数据+页面级数据，全局数据放在 json 目录下，全部文件自动进行合并；页面级别数据，存放在与 html 文件同名的 json 文件中，
    -   在 json 文件中可以使用多语言占位符。
-   会在每个页面里自动添加 babel-polyfill；
-   可以创建/index_head.js 中添加文件，会自动添加到所有页面中；
-   可以创建/index_css.js，里面 import 多个 css 文件，这些 css 会自动添加到每个页面；
