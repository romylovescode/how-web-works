### https://zhuanlan.zhihu.com/p/35478792

SCSS简明上手指南
Hank
Hank
UinIO.com 电子技术实验室
来自专栏 · 成都IT圈
54 人赞同了该文章
：是成熟、稳定、强大的CSS预处理器，截止到目前为止已经发展有10年，前当最新release版本为3.5.5。而SCSS是Sass3版本当中引入的新语法特性，完全兼容CSS3的同时继承了Sass强大的动态功能。本文翻译自Sass Guide和Sass Syntactically Awesome StyleSheets两篇官方文档，讲解了现代化前端开发当中经常使用的SCSS语法特性，便于开发小组的同学快速上手。知乎文章目前存在字数限制，完整带书签版本请进入笔者的电子技术博客 UinIO.com：

SCSS 3.5.5 简明上手指南
​www.uinio.com/Web/Scss/


基于Gulp完成前端自动化的年代，出于快速上手以及npm安装方便的考虑，开发团队一直沿用Less和gulp-less作为CSS预处理工具，但是Sass提供了更加丰富的动态语法特征，因此逐步淘汰基于Gulp的beaver前端项目脚手架以后，新项目全部基于Webpack的node-sass和sass-loader作为预处理工具。Sass和Less的详细比较可以参考sass-vs-less和Sass与Less比拼两篇文章，里面对两者的优劣做了非常详实的比较。

由于笔者个人精力有限，翻译与撰写过程之中难免存有疏漏之处，SASS作为前端日常开发当中，言简意赅书写CSS的良好伴侣，笔者专门在自己的Github上开辟 一个英文官方文档的翻译项目，如果阅读过程中发现存有缪误之处，欢迎大家向笔者提交pull request。

uinika/scss-reference-chinese
​github.com/uinika/scss-reference-chinese

安装
如果正在使用的是Linux发行版，需要通过apt安装Ruby以及build-essential。

➜  sudo apt-get install ruby-dev
➜  gem install sass
命令行当中可以通过sass input.scss output.css编译Scss文件，也可以添加--watch选项监听Scss文件的变化：

➜  sass --watch input.scss:output.css
如果多个Scss文件放置在一个目录当中，也可以对整个文件目录进行监听。

➜  sass --watch app/sass:public/stylesheets
通过-i或者--interactive可以运行一个交互式的SassScript命令行。

➜  sass -i
>> 5px + 8px
13px
通过sass --help获取更多关于SASS在命令行的使用方法。
编码规则
SASS首先会检查代码文件的Unicode BOM（byte order mark），然后是@charset声明，再然后是底层运行Ruby的字符串编码，如果这些都未进行设置，将会默认以UTF-8编码输出CSS文件。

@charset 'utf-8';

#app {
  background: url('../assets/背景图片.png');
}
建议代码开头位置显式定义@charset "encoding-name";，让SASS能够按照给定的字符集编译输出。
特性概览
CSS书写代码规模较大的Web应用时，容易造成选择器、层叠的复杂度过高，因此推荐通过SASS预处理器进行CSS的开发，SASS提供的变量、嵌套、混合、继承等特性，让CSS的书写更加有趣与程式化。

变量
变量用来存储需要在CSS中复用的信息，例如颜色和字体。SASS通过$符号去声明一个变量。

$font-stack:    Helvetica, sans-serif;
$primary-color: #333;

body {
  font: 100% $font-stack;
  color: $primary-color;
}
上面例子中变量$font-stack和$primary-color的值将会替换所有引用他们的位置。

body {
  font: 100% Helvetica, sans-serif;
  color: #333; }
嵌套
SASS允许开发人员以嵌套的方式使用CSS，但是过度的使用嵌套会让产生的CSS难以维护，因此是一种不好的实践，下面的例子表达了一个典型的网站导航样式：

nav {
  ul {
    margin: 0;
    padding: 0;
    list-style: none;
  }

  li { display: inline-block; }

  a {
    display: block;
    padding: 6px 12px;
    text-decoration: none;
  }
}
大家注意上面代码中的ul、li、a选择器都被嵌套在nav选择器当中使用，这是一种书写更高可读性CSS的良好方式，编译后产生的CSS代码如下：

nav ul {
  margin: 0;
  padding: 0;
  list-style: none; }
nav li {
  display: inline-block; }
nav a {
  display: block;
  padding: 6px 12px;
  text-decoration: none; }
引入
SASS能够将代码分割为多个片段，并以underscore风格的下划线作为其命名前缀（_partial.scss），SASS会通过这些下划线来辨别哪些文件是SASS片段，并且不让片段内容直接生成为CSS文件，从而只是在使用@import指令的位置被导入。CSS原生的@import会通过额外的HTTP请求获取引入的样式片段，而SASS的@import则会直接将这些引入的片段合并至当前CSS文件，并且不会产生新的HTTP请求。下面例子中的代码，将会在base.scss文件当中引入_reset.scss片断。

// _reset.scss
html, body, ul, ol {
  margin:  0;
  padding: 0;
}

// base.scss
@import 'reset';
body {
  font: 100% Helvetica, sans-serif;
  background-color: #efefef;
}
SASS中引入片断时，可以缺省使用文件扩展名，因此上面代码中直接通过@import 'reset'引入，编译后生成的代码如下：

html, body, ul, ol {
  margin: 0;
  padding: 0; }

body {
  font: 100% Helvetica, sans-serif;
  background-color: #efefef; }
SASS片断使用下划线前缀命名，主要用于SASS命令行工具watch指定目录源码的场景；如果使用Webpack等打包工具则毋须顾虑该问题，CSS样式将会通过Webpack加载器，按照ES6风格的import或Webpack插件extract-text-webpack-plugin进行打包和模块化。
混合
混合（Mixin）用来分组那些需要在页面中复用的CSS声明，开发人员可以通过向Mixin传递变量参数来让代码更加灵活，该特性在添加浏览器兼容性前缀的时候非常有用，SASS目前使用@mixin name指令来进行混合操作。

@mixin border-radius($radius) {
          border-radius: $radius;
      -ms-border-radius: $radius;
     -moz-border-radius: $radius;
  -webkit-border-radius: $radius;
}

.box {
  @include border-radius(10px);
}
上面的代码建立了一个名为border-radius的Mixin，并传递了一个变量$radius作为参数，然后在后续代码中通过@include border-radius(10px)使用该Mixin，最终编译的结果如下：

.box {
  border-radius: 10px;
  -ms-border-radius: 10px;
  -moz-border-radius: 10px;
  -webkit-border-radius: 10px; }
继承
继承是SASS中非常重要的一个特性，可以通过@extend指令在选择器之间复用CSS属性，并且不会产生冗余的代码，下面例子将会通过SASS提供的继承机制建立一系列样式：

// 这段代码不会被输出到最终生成的CSS文件，因为它没有被任何代码所继承。
%other-styles {
  display: flex;
  flex-wrap: wrap;
}

// 下面代码会正常输出到生成的CSS文件，因为它被其接下来的代码所继承。
%message-common {
  border: 1px solid #ccc;
  padding: 10px;
  color: #333;
}

.message {
  @extend %message-common;
}

.success {
  @extend %message-common;
  border-color: green;
}

.error {
  @extend %message-common;
  border-color: red;
}

.warning {
  @extend %message-common;
  border-color: yellow;
}
上面代码将.message中的CSS属性应用到了.success、.error、.warning上面，魔法将会发生在最终生成的CSS当中。这种方式能够避免在HTML元素上书写多个class选择器，最终生成的CSS样式是下面这样的：

.message, .success, .error, .warning {
  border: 1px solid #ccc;
  padding: 10px;
  color: #333; }

.success {
  border-color: green; }

.error {
  border-color: red; }

.warning {
  border-color: yellow; }
操作符
SASS提供了标准的算术运算符，例如+、-、*、/、%。在接下来的例子里，我们尝试在aside和article选择器当中对宽度进行简单的计算。

.container { width: 100%; }

article[role="main"] {
  float: left;
  width: 600px / 960px * 100%;
}

aside[role="complementary"] {
  float: right;
  width: 300px / 960px * 100%;
}
上面代码以960px为基准建立了简单的流式网格布局，SASS提供的算术运算符让开发人员可以更容易的将像素值转换为百分比，最终生成的CSS样式如下所示：

.container {
  width: 100%; }

article[role="main"] {
  float: left;
  width: 62.5%; }

aside[role="complementary"] {
  float: right;
  width: 31.25%; }
CSS扩展
SASS当中有两种可用的语法，一种是Scss（即SASS化的CSS），这是一种类CSS的语法，以.scss作为源码文件后缀。另一种是Sass，提供了比较简明的CSS书写方式，通过缩进而非括号来标识嵌套的选择器，以及使用换行而非分号来分隔各个属性，并使用.sass作为代码文件的后缀。

Sass和Scss两种代码风格可以通过sass-convert命令行工具进行相互转换：

➜  sass-convert test.scss test.sass

➜  sass-convert test.sass test.scss
日常开发环境下，出于前端开发的使用习惯，更多会选择Scss语法，接下来的内容也将会以Scss风格为主。
嵌套规则
Scss允许CSS规则嵌套使用，父子规则将会呈现包含选择器的关系，例如：

/*===== SCSS =====*/
#main p {
  color: #00ff00;
  width: 97%;

  .redbox {
    background-color: #ff0000;
    color: #000000;
  }
}

/*===== CSS =====*/
#main p {
  color: #00ff00;
  width: 97%; }
  #main p .redbox {
    background-color: #ff0000;
    color: #000000; }
这样可以避免重复的使用父级选择器，从而达到简化CSS代码结构的目的，例如：

/*===== SCSS =====*/
#main {
  width: 97%;

  p, div {
    font-size: 2em;
    a { font-weight: bold; }
  }

  pre { font-size: 3em; }
}

/*===== CSS =====*/
#main {
  width: 97%; }
  #main p, #main div {
    font-size: 2em; }
    #main p a, #main div a {
      font-weight: bold; }
  #main pre {
    font-size: 3em; }
引用父级选择器&
Scss使用$关键字在CSS规则中引用父级选择器，例如在嵌套使用伪类选择器的场景下：

/*===== SCSS =====*/
a {
  font-weight: bold;
  text-decoration: none;
  &:hover { text-decoration: underline; }
  body.firefox & { font-weight: normal; }
}

/*===== CSS =====*/
a {
  font-weight: bold;
  text-decoration: none; }
  a:hover {
    text-decoration: underline; }
  body.firefox a {
    font-weight: normal; }
无论CSS规则嵌套的深度怎样，关键字&都会使用父级选择器级联替换全部其出现的位置：

/*===== SCSS =====*/
#main {
  color: black;
  a {
    font-weight: bold;
    &:hover { color: red; }
  }
}

/*===== CSS =====*/
#main {
  color: black; }
  #main a {
    font-weight: bold; }
    #main a:hover {
      color: red; }
&必须出现在复合选择器开头的位置，后面再连接自定义的后缀，例如：

/*===== SCSS =====*/
#main {
  color: black;
  &-sidebar { border: 1px solid; }
}

/*===== CSS =====*/
#main {
  color: black; }
  #main-sidebar {
    border: 1px solid; }
如果在父级选择器不存在的场景使用&，Scss预处理器会报出错误信息。
嵌套属性
CSS许多属性都位于相同的命名空间（例如font-family、font-size、font-weight都位于font命名空间下），Scss当中只需要编写命名空间一次，后续嵌套的子属性都将会位于该命名空间之下，请看下面的代码：

/*===== SCSS =====*/
.demo {
  // 命令空间后带有冒号:
  font: {
    family: fantasy;
    size: 30em;
    weight: bold;
  }
}

/*===== CSS =====*/
.demo {
  font-family: fantasy;
  font-size: 30em;
  font-weight: bold; }
命令空间上可以直接书写CSS简写属性，但是日常开发中建议直接书写CSS简写属性，尽量保持CSS语法的一致性。

.demo {
  font: 20px/24px fantasy {
    weight: bold;
  }
}

.demo {
  font: 20px/24px fantasy;
    font-weight: bold;
}
占位符选择器%
占位符选择器与id或者class选择器的用法类似，只是#和.需要替换成为%，占位符选择器必须通过@extend指令进行调用。

关于占位符的讲解，请具体参考@extend章节。
注释
SASS支持标准CSS的多行注释/* */和单行注释//，其中，多行注释会完整输出到编译后的CSS文件，而单行注释则不会。

/* 多行注释
 * 会原样输出到CSS文件 */
body { color: black; }

// 单行注释不会出现在输出的CSS当中
a { color: green; }
编译成为CSS的结果如下：

@charset "UTF-8";
/* 多行注释
 * 会原样输出到CSS文件 */
body {
  color: black; }

a {
  color: green; }
插值语句#{$variable}可以应用在多行注释当中。

$version: "3.5.5";
/* 这段CSS是通过SASS #{$version}生成的。*/
编译成为CSS的结果：

@charset "UTF-8";
/* 这段CSS是通过SASS 3.5.5生成的。*/
当注释中出现中文时，SASS会自动在生成的CSS文件头部添加@charset "UTF-8";。
SassScript
SassScript是Sass提供的一系列CSS语法扩展，允许在任意CSS属性上书写变量、计算以及函数。SassScript还可以通过interpolation插值生成选择器和属性名，这在编写mixins时非常有用。

变量$
SassScript通过$关键字声明和使用一个变量。

/*===== SCSS =====*/
$width: 8rem;  // 声明变量

#main {
  width: $width;  // 使用变量
}

/*===== CSS =====*/
#main {
  width: 8rem; }
SassScript变量支持块级作用域，嵌套规则当中声明的变量都是局部变量，可以通过!global将局部变量声明为全局变量。

/*===== SCSS =====*/
#menu {
  $width: 5rem !global;
  width: $width;
}

#sidebar {
  width: $width;
}

/*===== CSS =====*/
#menu {
  width: 5rem; }

#sidebar {
  width: 5rem; }
数据类型
SassScript支持8种数据类型，

数值number：1.2,13,10px
颜色color：blue、#04a3f9、rgba(255, 0, 0, 0.5)
布尔值bollean：true、false
空值null：null
字符串string：有或没有引号，"menu"、'sidebar'、navbar
数组list：由空格或逗号分隔，1.5em 1em 0 2em, Helvetica, Arial, sans-serif
对象map：(key1: value1、key2: value2)
函数function。
Scss支持所有其它类型的CSS属性值，例如Unicode字符或者!important声明。Scss并没有对这些类型进行特殊处理，仅仅将其视为没有使用引号的字符串。
String
CSS支持两种类型的字符串：

使用单双引号的："Lucida Grande"， 'http://sass-lang.com';
没有使用引号的：sans-serif，bold;
SassScript能够自动识别这2种字符串，Scss当中使用了哪种类型字符串，就会在生成的CSS文件原样输出该类型字符串。

但是这里存在一个意外，当使用#{}插值语法的时候，使用引号的字符串会失去引号，这样可以便于mixin中使用选择器的名称。

/*===== SCSS =====*/
@mixin firefox-message($selector) {
  body.firefox #{$selector}:before {
    content: "Hi, Firefox users!";
  }
}

@include firefox-message(".header");

/*===== CSS =====*/
body.firefox .header:before {
  content: "Hi, Firefox users!"; }
List
SASS当中可以通过List来表达CSS声明的值，例如margin: 10px 15px 0 0或font-face: Helvetica, Arial, sans-serif。SASS中的List是一系列由空格或者逗号分隔的值，实际上单个值也可以作为一个List，即只拥有一项元素的List。

List本身的作用不多，但是通过SassScript的List函数可以让它发挥更大的作用。nth函数可以操作List中的每一项元素，join函数能够合并多个List在一起，append函数能够向List添加一项元素，而@each指令可以为List当中每项元素添加样式。

除了包含简单的值，List还可以包含其它的List，例如拥有2项元素的List1px 2px, 5px 6px包含了1px 2px和5px 6px两个List，如果内部的List和外部List拥有相同的分隔符，那么必须使用圆括号()标识内部List的起始和结束位置，例如(1px 2px) (5px 6px)。

当List被转换为普通CSS的时候，SASS不会添加其它圆括号，这意味(1px 2px) (5px 6px)和1px 2px 5px 6px变成CSS的时候结果是相同的。但是无论如何，它们在SASS里是不相同的，(1px 2px) (5px 6px)是一个拥有2个List元素的List，1px 2px 5px 6px则是拥有4个普通元素的List。

List里同样可以不包含任何元素，这样的List会被表现为()（同样可以认为其是一个空的map），它们并不能直接输出为CSS。例如SASS编译font-family: ()时将会发生错误。如果List包含一个空的List()或者其本身为空值null，例如对于1px 2px () 3px或1px 2px null 3px当中的null和()将会在输出的CSS当中被移除。

逗号分隔的List尾部可能还跟着一个逗号，这在一些特殊的情况下比较有用，例如展现一个只有1个元素的List，(1,)就是一个包含了元素1的List，而(1 2 3,)是一个逗号分隔的List，这个List又包含着一个空格分隔的List，空格分隔的List拥有着1、2、3三个元素。

List也可以通过方括号[]编写，方括号编写的List用来在CSS Grid布局中表达行的名称，但是它同样也可以用于纯SASS代码，同样也可以通过逗号或者空格进行分隔。
Map
Map提供了key与value之间的关联关系，并且能够通过key找到相应的value。他们可以容易的搜集value到一个被命名的分组，然后动态的操作这些分组。它们在CSS中没有直接并行的概念，尽管在句法上与媒体查询表达式相似：

$map: (key1: value1, key2: value2, key3: value3);
不同与前面提到的List，Map必须总是被圆括号()包裹，并且必须使用逗号,进行分隔。Map中的key与value可以是任意的SassScript对象，一个Map可以只拥有1个给定的key和对应的value（可以是一个List），但是一个给定的value可能会关联到多个key。

和List一样，Map大部分情况下也需要通过SassScript函数对其进行操作。其中map-get函数负责查询Map中的value，map-merge函数添加value至Map，指令@each可以被用来为Map中的每个key和value添加样式。Map中键值对的顺序总是与其被建立的时候相同。

List能够使用的位置，Map也能正常使用。当通过一个List函数进行使用的时候，Map被视为拥有键值对的List。例如：(key1: value1, key2: value2)在List函数当中将被视为一个嵌套的List（即key1 value1, key2 value2）。但是，值得注意的是，List并不能相反的被视为一个Map，尽管空的List将会抛出错误。()即可以被视为一个没有key和value的Map，也可以视为没有元素项的List.

注意Map的key可以是任意的Sass数据类型（即使是Map），并且声明Map的语法允许任意的SassScript表达式，该表达式将会被解析以决定key的值。

Map并不能被转换为普通CSS，在CSS函数中使用它作为变量和参数的值将会导致错误。这种情况下，建议使用inspect($value)函数为Map的调试生成一个有用的字符串输出。

Color
任何CSS颜色表达式都返回一个SassScript色彩值，这包括大量被命名的颜色，它们与未引用的字符串是不可区分的。

在压缩输出模式下，Sass将输出颜色的最小的CSS表示。例如，在压缩模式下，#FF0000将输出为红色red，而#FFEBCD将会输出为白杏色blanchedalmond。

用户使用被命名的颜色时，经常会遇到一个问题：在。。。SASS喜欢在其他输出模式中键入相同的输出格式。。。之后，一个颜色会被插值到一个选择器，代码压缩时这将会变成不合法的语法。要避免这个问题，在被命名颜色用于构建选择器的时候，总是需要为它们添加引号。

First-Class Function
get-function($function-name)将会返回一个函数引用，该函数能够被传递到call($function, $args...)并得到调用。First-Class函数不能直接用于CSS输出，任何这样的尝试都将会引发错误。

运算
SassScript所有数据类型都支持==或!=逻辑运算，但是每种数据类型还拥有自己的运算方式。

数值运算
数值类型支持常见的数值运算+、-、*、/、%，并在必要时在不同的绝对数值单位之间转换。SASS数学函数会在计算过程中保留单位，这意味不同单位不能在一起进行运算（例如px和em），两个相同单位的数值相乘后可能会造成运算结果的单位重复，例如：10px * 10px == 100px * px，显然这里px * px是一个无效单位，此时SASS会由于使用非法单位而报错。

关系运算符<、 >、<=、>=同样支持数值类型，而相等运算符==, !=则可以被SASS所有类型所支持。
除法与标识符/
CSS允许在属性值中使用/作为分隔数字的手段，SassScript为CSS属性语法提供了一个扩展，允许/作为除法运算符来使用。如果SassScript当中2个数值通过/进行分隔，那么它们的计算结果将会出现在生成的CSS当中。

/被解释为除法的情况主要有以下3种：

如果值及其部分被保存到一个变量，或者通过一个函数返回。
值被圆括号()包裹，除非这些圆括号在一个List的外面并且该地，
如果指定值被用于其它算术表达式。
例如：

/*===== SCSS =====*/
p {
  font: 10px/8px;                // 普通CSS，未进行除法运算。
  $width: 1000px;
  width: $width/2;               // 使用一个变量，进行了除法运算。
  width: round(1.5)/2;           // 使用一个函数，进行了除法运算。
  height: (500px/2);             // 使用圆括号, 进行了除法运算。
  margin-left: 5px + 8px/2px;    // 使用了加号+, 进行了除法运算。
  font: (italic bold 10px/8px);  // 在List当中，圆括号不会进行计算。
}

/*===== CSS =====*/
p {
  font: 10px/8px;
  width: 500px;
  height: 250px;
  margin-left: 9px; }
如果需要在普通CSS当中使用变量，可以通过#{}去插入它们，例如：

/*===== SCSS =====*/
p {
  $font-size: 12px;
  $line-height: 30px;
  font: #{$font-size}/#{$line-height};
}

/*===== CSS =====*/
p {
  font: 12px/30px; }
减法、负数与标识符-
中划线-在CSS和SASS中有许多不同的意义，可以表达一个减法运算符（5px - 3px），一个负数的开始（-23px），一元负值运算符（-$var），或者一个标识符的组成部分（font-weight），大部分情况下何时何地使用是比较清晰的，但也存在一些例外情况，为了避免这些例外需要遵循如下原则：

进行减法运算的时候，符号两则尽量保留空格。
作为负值标识或一元负值运算符时，需要在数值或变量前包含一个空格。
如果是在空格分隔的List中，就用括号将一元负值运算符括起来，就像10px (-$var)一样。
不同含义的表达遵循如下的优先次序：

表达式中使用字母时，a-1表示一个没有引号的字符串，其值为"a-1"。但是唯一的例外在于单位的使用，比如5px-3px与5px - 3px是等效的，其执行结果为2px。
两个数字之间没有空格意味着这是一个减法，所以1-2与1 - 2作用相同。
将-放置在数字开头表示这是一个负数，所以1 -2是一个包含了1和-2两个元素的List。
两个数字之间不考虑空格表示这是减法，所以1 -$var和1 - $var相同。
放置在一个值的前面表示这是一元负值运算符，该运算符会返回当前数值的负值。
➜  hank ✗ sass -i
>> a-1
"a-1"
>> 5px-3px
2px
>> 1 -2
(1 -2)
>> 1-2
-1
颜色运算
SASS中所有的数学运算都分段的支持颜色值，即分别对红、绿、蓝依次进行计算。

/*===== SCSS =====*/
p {
  color: #010203 + #040506;
}
在执行完01 + 04 = 05，02 + 05 = 07，03 + 06 = 09运算后编译为下面的CSS：

/*===== CSS =====*/
p {
  color: #050709; }
通常来说，使用颜色函数比使用颜色运算能更加有效的达到相同目的。颜色运算同样也可以作用于数值与颜色值，并且依然是分段进行的，例如：

p {
  color: #010203 * 2;
}
进行01 * 2 = 02，02 * 2 = 04，03 * 2 = 06运算后编译为下面CSS：

p {
  color: #020406; }
注意，带有alpha通道的颜色（使用rgba或hsla函数创建的颜色）必须具有相同的alpha值单位，以便与其执行颜色运算。

/*===== SCSS =====*/
p {
  color: rgba(255, 0, 0, 0.75) + rgba(0, 255, 0, 0.75);
}

/*===== CSS =====*/
p {
  color: rgba(255, 255, 0, 0.75); }
颜色的alpha通道可以通过opacify和transparentize函数进行修改，例如：

/*===== SCSS =====*/
$translucent-red: rgba(255, 0, 0, 0.5);
p {
  color: opacify($translucent-red, 0.3);  // 增加alpha通道值
  background-color: transparentize($translucent-red, 0.25);  // 减少alpha通道值
}

/*===== CSS =====*/
p {
  color: rgba(255, 0, 0, 0.8);
  background-color: rgba(255, 0, 0, 0.25); }
IE的filter滤镜要求所有颜色都包括alpha层，并且严格采用#AABBCCDD格式，SASS提供了ie_hex_str函数能够更加简便的对颜色值进行转换，例如：

/*===== SCSS =====*/
$translucent-red: rgba(255, 0, 0, 0.5);
$green: #00ff00;
div {
  filter: progid:DXImageTransform.Microsoft.gradient(enabled='false', startColorstr='#{ie-hex-str($green)}', endColorstr='#{ie-hex-str($translucent-red)}');
}

/*===== CSS =====*/
div {
  filter: progid:DXImageTransform.Microsoft.gradient(enabled='false', startColorstr='#FF00FF00', endColorstr='#80FF0000'); }
字符串运算
SassScript使用+运算符执行字符串连接操作。

/*===== SCSS =====*/
p {
  cursor: e + -resize;
}

/*===== CSS =====*/
p {
  cursor: e-resize; }
注意，如果有引号的字符串被添加到没有引号的字符串（即带引号的字符串在+的左侧），生成的字符串将会是有引号的。同理，如果将没有引号的字符串添加到有引号的字符串（即没有引号的字符串在+的左侧），则结果为没有引号的字符串。

/*===== SCSS =====*/
p:before {
  content: 'Foo ' + Bar;         // 有引号字符串 + 无引号字符串 = 有引号字符串
  font-family: sans- + 'serif';  // 无引号字符串 + 有引号字符串 = 无引号字符串
}

/*===== CSS =====*/
p:before {
  content: "Foo Bar";
  font-family: sans-serif; }
默认情况下，临近的两个CSS属性值可以通过空格进行连接。

/*===== SCSS =====*/
p {
  margin: 3px + 4px auto;
}

/*===== CSS =====*/
p {
  margin: 7px auto; }
字符串里可以通过#{}插入动态的值，就像下面这样：

/*===== SCSS =====*/
p:before {
  content: "I ate #{5 + 10} pies!";
}

/*===== CSS =====*/
p:before {
  content: "I ate 15 pies!"; }
如果字符串插值#{}中的变量是空值，则插值表达式的结果将被视为空字符串。

/*===== SCSS =====*/
$value: null;  // 空值
p:before {
  content: "I ate #{$value} pies!";
}

/*===== CSS =====*/
p:before {
  content: "I ate  pies!"; }
布尔运算
SassScript当中布尔类型的值支持与and、或or、非not运算。

/*===== SCSS =====*/
$year: 2018;
$moth: 8;
$day: 6;
$name: Hank;
#app {
  @if ($year > 2010 and $moth==8 or $day==6 not $name==Hank) {
    color: blue;
  } @else {
    color: red;
  }
}

/*===== CSS =====*/
#app {
  color: blue; }
List运算
数组不支持任何特定的计算方式，只能通过list函数进行操作，下面代码是一个使用了hsl($hue, $saturation, $lightness)函数的例子：

/*===== SCSS =====*/
a {
  color: hsl(120deg, 100%, 50%)
}

/*===== CSS =====*/
a {
  color: lime; }
圆括号
SASS中可以通过圆括号{}来调整运算的优先级。

/*===== SCSS =====*/
p {
  width: 1em + (2em * 3);
}

/*===== CSS =====*/
p {
  width: 7em; }
函数
SassScript内置了许多有用的函数，可以通过普通的CSS语句进行调用。

/*===== SCSS =====*/
p {
  color: hsl(0, 100%, 30%);
}

/*===== CSS =====*/
p {
  color: #990000; }
关键词参数
Sass函数还允许通过关键词参数(即带有键名的参数)进行调用，上面的例子也可以改写为下面代码：

/*===== SCSS =====*/
p {
  color: hsl($hue: 0, $saturation: 100%, $lightness: 30%);
}

/*===== CSS =====*/
p {
  color: #990000; }
虽然这不是很简洁，但可以使样式表更加容易阅读。同时还能够让函数提供更灵活的接口，提供多个参数的同时，并不会变得难以调用。

命名参数能够以任何顺序进行传递，而带有默认值的参数可以省略掉。如果命名参数是变量名，那么下划线与破折号可以互换使用。

可以通过Sass::Script::Functions查看SASS函数及其参数名称的完整清单，也可以通过Ruby进行自定义。

插值#{}
开发人员可以通过插值（interpolation）#{}在选择器和属性名称中使用SassScript变量。

/*===== SCSS =====*/
$name: uinika;
$attr: border;
p.#{$name} {
  #{$attr}-color: blue;
}

/*===== CSS =====*/
p.uinika {
  border-color: blue; }
还可以使用#{}将SassScript放入属性值，大多数情况下这种方式并不优于变量，但是使用#{}意味着其附近的任意操作都将被视为普通CSS，例如：

/*===== SCSS =====*/
p {
  $font-size: 12px;
  $line-height: 30px;
  font: #{$font-size}/#{$line-height};
}

/*===== CSS =====*/
p {
  font: 12px/30px; }
SassScript中的&
如同使用选择器一样，SassScript中的&引用着当前的父选择器列表（一个由逗号分隔List作为元素再通过空格进行分隔的List）。

.navbar.text .sidebar.word,
.menu.item {
  $selector: &;
}
上面代码中$selector的值为((".navbar.text" ".sidebar.word"), ".menu.item")，这里的复合选择器使用了引号"标识它是一个字符串，但是真实情况它可能是没有引号的。即使其父选择器并没有包含逗号和空格，&依然会拥有两层嵌套，因此它能够被持续的访问。

当父级选择器列表不存在的时候，&运算符的值为null，使用mixin当中可以通过该特性判断父选择器列表是否存在。

@mixin does-parent-exist {
  @if & {
    &:hover {
      color: red;
    }
  }
  @else {
    a {
      color: red;
    }
  }
}
变量默认值!default
可以在变量后添加!default声明，如果变量已被赋值就不会再被重新进行赋值，如果变量未被赋值，则会被赋予新值。

/*===== SCSS =====*/
$content: "First content";
$content: "Second content?" !default;
$new_content: "First time reference" !default;
#main {
  content: $content;
  new-content: $new_content;
}

/*===== CSS =====*/
#main {
  content: "First content";
  new-content: "First time reference"; }
如果变量的值是null，则会被!default视为没有赋值。

/*===== SCSS =====*/
$content: null;
$content: "Non-null content" !default;
#main {
  content: $content;
}

/*===== CSS =====*/
#main {
  content: "Non-null content"; }
@规则与指令
SASS支持所有CSS3的@-Rules规则，以及SASS特有的#Directive指令。

@import / #import
SASS拓展了CSS的@import允许其导入.scss或.sass文件，导入的文件将合并、编译到一个CSS文件，文件中的变量和mixin都可以在导入的主文件当中使用。

SASS会基于当前目录查找其它文件，以及Rack、Rails或者Merb下的SASS文件目录。可以通过:load_paths或--load-path选项指定额外的搜索目录。

@import通过指定路径以及文件名来引入SASS文件，但是在下面4种情况当中，@import仅被编译为普通的CSS语句：

如果文件拓展名为.css。
如果文件名以http://开头。
如果文件名是url()
如果@import包含媒体查询语句。
/*===== SCSS =====*/
@import "app.css";
@import "pp" screen;
@import "https://uinika.github.io/app";
@import url(app);

/*===== CSS =====*/
@import url(app.css);
@import "pp" screen;
@import "https://uinika.github.io/app";
@import url(app);
除上述四种情况外，文件拓展名为.scss或.sass的情况都会进行预编译处理，即使文件拓展名缺省依然能够正确的进行导入。

// 下面2种导入方式等效
@import 'base';
@import 'base.scss;
SASS允许在一个@import语句内同时导入多个文件。

@import 'base', 'reset', 'app';
@import语句内可以有限制的使用#{}插值，即不能动态的导入基于变量的SASS文件，只能用于标准CSS的@import url("");导入方式。

/*===== SCSS =====*/
$family: unquote("Droid+Sans");
@import url("http://fonts.googleapis.com/css?family=#{$family}");

/*===== CSS =====*/
@import url("http://fonts.googleapis.com/css?family=Droid+Sans");
Partial
如果你引入一个SCSS或Sass文件，但并不希望它们编译成CSS文件，那么可以在文件名开头添加一个下划线_，这样Sass将不会把该文件编译成CSS文件，然后引入文件的时候不需要添加下划线。例如：你拥有一个_colors.scss，但是Sass并不会创建_colors.css文件，然后，不过仍然可以通过@import "colors";语句引入_colors.scss文件。

注意，不能在同一目录中包含相同名称的Partial和非Partial；例如：_colors.scss不能与colors.scss在相同目录下并存。
嵌套的@import
大多数情况下，在代码的顶层使用@import导入是最为常用，但是我们依然可以在CSS规则和@media规则中使用；其效用如同基本的@import，仍然会去包含引入文件的内容。

/*===== SCSS =====*/
// 文件demo.scss包含如下代码
.demo {
  color: red;
}

#app {
  @import "demo"; // 在文件app.scss中引入demo.scss
}

/*===== CSS =====*/
#app .demo {
  color: red;
}
指令只能用于文档的基本级别，在一个嵌套上下文当中被@import引入的文件里，诸如@mixin或者@charset是不被允许的。
不能在mixin或者控制指令当中嵌套使用@import。
@media / #media
Sass中@media的用法与CSS相同，但是允许在CSS规则中嵌套使用。编译时@media将被编译到文件最外层，包含嵌套的父级选择器。

/*===== SCSS =====*/
.sidebar {
  width: 300px;
  @media screen and (orientation: landscape) {
    width: 500px;
  }
}

/*===== CSS =====*/
.sidebar {
  width: 300px;
}

@media screen and (orientation: landscape) {
  .sidebar {
    width: 500px;
  }
}
@media的查询条件可以嵌套使用，编译后SASS会自动添加and关键字。

/*===== SCSS =====*/
@media screen {
  .sidebar {
    @media (orientation: landscape) {
      width: 500px;
    }
  }
}

/*===== CSS =====*/
@media screen and (orientation: landscape) {
  .sidebar {
    width: 500px;
  }
}
@media的查询条件当中也可以使用SassScript。

/*===== SCSS =====*/
$media: screen;
$feature: -webkit-min-device-pixel-ratio;
$value: 1.5;
@media #{$media} and ($feature: $value) {
  .sidebar {
    width: 500px;
  }
}

/*===== CSS =====*/
@media screen and (-webkit-min-device-pixel-ratio: 1.5) {
  .sidebar {
    width: 500px;
  }
}
@extend / #extend
现实的Web页面开发场景当中，当一个class属性拥有其它多个class所具备样式的时候，通常的做法是在HTML上书写全部这些class，如同下面的代码：

<div class="error seriousError">
  Oh no! You've been hacked!
</div>

<style>
.error {
  border: 1px #f00;
  background-color: #fdd;
}
.seriousError {
  border-width: 3px;
}
</style>
这样的代码并不利于维护，并且会带来非语义相关的样式到HTML上。

使用SASS提供的@extend可以在不同CSS选择器之间继承样式，从而完美避免上述问题。

/*===== SCSS =====*/
.error {
  border: 1px #f00;
  background-color: #fdd;
}
.seriousError {
  @extend .error;
  border-width: 3px;
}

/*===== CSS =====*/
.error, .seriousError {
  border: 1px #f00;
  background-color: #fdd;
}
.seriousError {
  border-width: 3px;
}
上面代码中.error的样式会自动应用到.seriousError，作用相当于同时使用了这两个class上的样式。

通过.error定义的其它样式规则，同样会在.seriousError上工作，比如下面这样定义样式后。

.error.danger {
  background-image: url("/image/warning.png");
}
答主在成都的 IT 行业工作近十余年，经常会在自己的电子技术博客 UinIO.com 当中分享一些产业与技术相关的文章，赠人玫瑰，手有余香，大家的【点赞、收藏、加关注】将会是我持续写作的最大动力。
UinIO.com
