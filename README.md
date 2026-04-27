# ui_json
太乱了，整理一下自己用过的跨平台 ui 库实现的接口

--------------------------------------------------------

文档分成两部分。一部分是纯 代码开发 ui 的接口，这部分已经很成熟了。
另外一部分就是通过 json 配置文件生成 ui 的部分，因为原来要避免与 html dom 的属性冲突，和最早的 
lua 版本好像不一样（主要是属性值）。这部分要再整理清楚。

--------------------------------------------------------
因为历史的原来，ui 库实现有好多个版本。最早是纯 HTML+js 版本，那里还很讨厌 TS 所以直接用的 js ... 后来代码量大了 .. :) 确实不用 ts 很难维护。但同一个项目中同时有 ts 和 js 存在也是个麻烦，vscode 的提示会混乱甚至失效。再加上各种其他原因，近些年又想某些时候还是纯 js 加上注解更方便。不过纯 TS 在 bun/deno 环境下很好用，所以 ts 也是要保留的 :)

商业/工作中用得最成功的其实是 lua 版本，有 ios/android/qt 三个版本，但实现到最后发现设计得太细，非全职工作状态下很难细心去维护。所以近些年来一直打算改成以 json 接口为主，抛弃单个函数与功能一一对应的方式（原因暂时不说了，太长）。

不过不管是怎样的接口，UI 函数基本上是固定的，成功应用了好多年，也一直很成功。json 配置的比较乱，不算成功，主要原因是设计器太花时间，有时候名称冲突无法解决，只好先上一个自己不喜欢的属性名 ...

用了很久后觉得还是纯代码方式好 ... 纯代码模式又用了好多年，直到移植一个很复杂的 gui 时发现实在没有精力，还是应该结合市面上存在的 ui 设计器来做。所以这部分还要再整理重新设计下。

不管怎么说纯函数的接口一定要统一了，所以这个文档不得不写一下 ...

--------------------------------------------------------
1.纯函数接口



```
UI_CreateView(parentView)
```

```
View_SetOnClick(view,func)
```

```
View_SetRect(view, rect)
```

```
//vcl 兼容的顶对齐 //现在不应当再用这个函数
function UI_AlignTop(view:any) 
```

* 这个是早期模仿 delphi 的对齐模式，但在实际开发中发现一来其他语言/环境有点难以实现，更重要的是早期的 delphi 没有 padding 的概念，导致 ui 要放很多多余的空白控件。现在主要使用 js/html/dom 的与父节点的位置的关系，也就是直接对应 css 的 left/right/top/bottom 属性，而早期的 delphi/lazarus 实际上也可以完美地用 anchor 锚来实现。在最新的 godot 的 ui 这样的环境中也很方便一一对应。

不过 ui 差异越来越大，可能 ui 布局十有八九不太可能统一，所以这个并不要求所有环境都实现。大多数还能支持传统 ui 精确定位的环境中也用 锚 对齐函数来代替了(目前这样可以在 delphi/lazarus/godot/c#winform 中通用，单实例网页 app 其实也完美支持，不过 react/web 主流就肯定不是这样的了)。



--------------------------------------------------------

2.控件一般用对齐就能实现大部分情况了，并不需要布局。目前精简为只定义控件的位置信息就可以了，再加少量扩展属性。

当然实际上复杂的控件应该是有自己的更多属性的。

基本信息示例如下:

```
{
    "name": "DIV",
    "id": "VForm_t1",
    "top": "40",
    "alignTop": "parentView",
    "ui_marginTop": "40",
    "left": "111",
    "alignLeft": "parentView",
    "ui_marginLeft": "111",
    "right": "590",
    "alignRight": "",
    "ui_marginRight": "228",
    "bottom": "281",
    "alignBottom": "",
    "ui_marginBottom": "216",
    "children": [
        {
            "name": "DIV",
            "id": "uiPanel1",
            "top": "44",
            "alignTop": "parentView",
            "ui_marginTop": "44",
            "left": "193",
            "alignLeft": "parentView",
            "ui_marginLeft": "193",
            "right": "358",
            "alignRight": "",
            "ui_marginRight": "121",
            "bottom": "95",
            "alignBottom": "",
            "ui_marginBottom": "146",
            "text": "uiPanel1",
            "back_color": "#444444",
            "children": []
        },
        {
            "name": "DIV",
            "id": "btnOpen",
            "top": "28",
            "alignTop": "parentView",
            "ui_marginTop": "28",
            "left": "37",
            "alignLeft": "parentView",
            "ui_marginLeft": "37",
            "right": "113",
            "alignRight": "",
            "ui_marginRight": "366",
            "bottom": "63",
            "alignBottom": "",
            "ui_marginBottom": "178",
            "children": []
        }
    ]
}

```


实际上定义与父节点的 4 个方向是否对齐、对齐多少就可以了，这是长久经验提炼出来的，欢迎大家实践 :)

对于 web, delphi/lazarus, ios 原生, macos 原生 的支持都很好，实现也都很简单直接。对安卓自己扩展个简单控件后也很容易，
这些都是已经在实际工程中应用了多年的。

后来发现和 cocos3d, godot 的控件布局方式也很容易兼容。所以说实际上是不需要那种 web/gtk 上使用的所谓布局形式的。
退一步说，在这基础上面要实现那种排版式的自动布局也很容易，工作中就实现过安卓版本的。





--------------------------------------------------------
尽量更新主文件
https://github.com/clqsrc/ui_json/edit/main/README.md






