---
layout:     post
title:      "金融数据智能分析平台"
subtitle:   "一个简单的数据展示和模型集成web平台"
date:       2017-07-26
author:     "Diane"
header-img: "img/post-bg-sightview.jpg"
tags:

---

## 前言

[感冒依旧没有好...](#build) 

总结一下实习期间搭建的一个金融数据智能分析平台

<p id = "build"></p>
---
## 正文
<script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=default"></script>
<blockquote>1.	服务器端：</blockquote>
<li>flask http://flask.pocoo.org/ </li>
	轻量级的Python Web框架
<li>pandas http://pandas.pydata.org/ </li>
    Python的数据结构和数据分析工具包，提供数据处理的Wrangling的功能
<li>sklearn http://scikit-learn.org/ </li>
    非常流行的Python机器学习包，依赖于numpy，scipy和matplotlib

<blockquote>2.	客户端:</blockquote>
<li>jquery</li>
	reactjs http://facebook.github.io/react/ 
	facebook开发的js UI框架，基于组件（component）而非mvc
<li>d3js https://d3js.org/ </li>
    数据驱动的DOM操纵库，可以创建丰富的数据可视化呈现。
<li>echarts http://www.oschina.net/p/echarts </li>
    百度开发的数据可视化库，基于canvas技术，功能丰富。实为中国开源项目的翘楚。
<li>bootstrap http://getbootstrap.com/ </li>
    twitter开发的前端框架，非常流行。
<li>jquery datatables  http://www.datatables.net/ </li>
    非常实用的基于jquery的表格控件
<li>bootstrap fielinput https://github.com/kartik-v/bootstrap-fileinput </li>
    HTML5文件上传控件
<li>papaparse https://github.com/mholt/PapaParse </li>
    CSV文件的JS解析
<li>requirejs http://www.requirejs.org/ </li>
    JS 依赖管理
<li>select2  https://select2.github.io/ </li>
    基于jquery的select控件

<blockquote>3.	开发构建工具</blockquote>
<li>nodejs https://nodejs.org/en/ </li>
<li>babel https://babeljs.io/ </li>
<p>javascript的编译器，支持把ES6的代码转换成浏览器可执行的代码，这里主要是为了支持reactjs使用的jsx的编译。运行financial_dataplatform非常简单，下载github上的code后，建议安装anaconda2，所有的Python2.7依赖就都准备好了，进入financial_dataplatform/package目录，运行：python main.py
以下步骤在只有js源文件而没有lib文件夹情况下才需要进行，而github中含有所有需要的文件，所以下述命令都不需要进行。</p>
<p>在只有js文件夹的情况下，因为react的js需要编译，需要运行如下的命令用babel进行js的转码才能运行，具体命令如下：</p>
    ## install node first</br>
    ## cd package/static</br>
    npm install -g babel-cli</br>
    npm install babel-preset-es2015 --save</br>
    npm install babel-preset-react --save</br>
    babel --presets es2015,react --watch js/ --out-dir lib/</br>
<p>大家也可以参考package/static/package.json了解需要的依赖。有时间需要集成一个更简单的build脚本来做这些事情。生成的JS文件在lib目录下。当修改了js／文件夹下的js文件之后也需要执行上一条babel命令完成转码，然后在浏览器中键入 localhost:5000启动客户端。</p>

<blockquote>4.	接下来介绍两部分功能：</blockquote>
<p>数据上传用到了file input控件，数据表用了datatable控件。为了方便CSV文件直接存贮在本地文件系统中。后台用pandas对csv文件进行处理。前台用Rest API读取csv文件，然后用papaparse解析后，展现在数据表中。更好的做法是在后台用Python对CSV文件作解析。注意这里对上传的CSV文件有严格的要求，必须有首行的header，末尾不能有空行。</p>
<p>有了数据后，就可以开始做分析了。首先我们看看可视化的分析。点击菜单Analysis／Visualization。可视化这一块的主要工作是从CSV的表结构数据，根据数据绑定，变形到echart的数据结构。因为echart并没有一个统一的数据模型，所以每一个类型的图表都需要有对应的数据变形的逻辑 。（代码 package/static/js/visualization ）现在主要的做了Pie，Bar，Line，Treemap，Scatter， Area这几种chart。现在用下来感觉echart优缺点都很明显，他提供的辅助功能很好，可以方便的增加辅助线，note，存贮为图形等。但是由于缺乏统一的数据模型扩展起来比较麻烦，接下来试用一下plotly，当然highchart是非常成熟的图表库。</p>
<p>除了基于可视化的分析功能，还有机器学习的功能。分类的算法可以使用KNN，Bayes和SVM。聚类算法现在实现了Kmeans。回归实现了线性回归和逻辑回归。</p>

<blockquote>5.	接下来可以进一步实现的功能：</blockquote>
<li>数据源</li>
现在的数据源只有CSV文件和数据库，可以考虑更多的数据源支持，例如数据仓库，REST调用，流等等。
<li>数据模型</li>
现在的数据模型比较简单，就是pandas的dataframe或者一个简单的cvs的表结构。可以考虑引入数据库。另外还需要增加对层级数据（hierachical）的支持
<li>数据变形</li>
数据变形是数据分析的必要准备工作。业内有很多专注于数据准备的产品，例如paxata,trifacta。现有的平台没有任何的数据变形和准备的功能，其实pandas有非常丰富的data wrangling的功能，我希望能在这之上包装一个data wrangling的DSL，可以让用户快速的进行数据准备。
<li>可视化库</li>
Baidu的echart是非常优秀的可视化库，可是用于数据探索时，还不够好。希望能有一套类似ggplot的前端可视化库来使用。另外地图功能和层级化的图表也是数据分析常见的功能。
<li>仪表盘功能</li>
这个没有仪表盘功能，这个功能是数据分析软件的标配。pyxley似乎是个不错的选择，也和dataplay的架构一致（python，reactjs）。
<li>机器学习和预测</li>
现在实现了最简单的一些机器学习和深度学习的算法，我觉得方向应该是面向用户，变得更简单，用户只给出简单的选项，例如要预测的目标属性，和用于预测的属性，然后自动的选择算法。另外需要更方便的对算法进行扩展。未来可以进一步拓展GPU的支持。


## 后记

下一篇介绍系统的数据库分析平台搭建方式。
