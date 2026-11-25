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

要注意的是一般来说这里的尺寸单位应当是逻辑单位，而不是 px ，不过在 web 中根据映射实际上可以等于 px 即使是高清屏下。但在
lazarus 下保存时是会是原始 px 值的，所以保存时要除以放大系数；在 C# winform 里面似乎是逻辑单位，具体大家仔细看下。另外
这个数据的放大系统一般是 2 ，但实际上有可能是浮点数的，所以还是用浮点来实现比较好。稍好的手机上似乎至少是 3.5 ，在 4k 14" 
 PC 显示器上应该也是超过 2 了的。

对于 web, delphi/lazarus, ios 原生, macos 原生 的支持都很好，实现也都很简单直接。对安卓自己扩展个简单控件后也很容易，
这些都是已经在实际工程中应用了多年的。

后来发现和 cocos3d, godot 的控件布局方式也很容易兼容。所以说实际上是不需要那种 web/gtk 上使用的所谓布局形式的。
退一步说，在这基础上面要实现那种排版式的自动布局也很容易，工作中就实现过安卓版本的。


--------------------------------------------------------
以前都是想在各个平台上全部实现以上的对齐算法，总体来讲基本上困难不大，不过随着用到的环境越来越多实际上已经没有时间去逐个完成，
因此在环境所在系统中的 ui 再抽象一层还是必须的。比如 C# winfom, rust slint 这样的也没有必须再实现，直接用原生设计器就好了。
所以可以考虑原来从 json 文件整体读取全部界面然后再取单个控件的模式再捡回来，其实是很容易做到兼容的，因为就当做 load_from_json()
到 uijson 对象的工作已经完成了就行，然后各个 form 甚至是 godot ui, cocos 3d 这样的都可以这样认为界面类是 uijson 的实例就可以了。

不过这里从 uijson 中取对象的定义非常混乱，还没有统一，是用 getnode(), get(), 还是 get_view() 没有定。因为控件未必是树组织的，所以
getnode 不见得就好，同时控件未必是 view, 似乎 get 最合适，但又太过简单（目前 rust 版本是用的 get；最早的 lua 版本忘记是什么了，
似乎也不太合理）。以后还是要统一。可能尽量按最早 lua 版本的为基准

```

//假想，其实目前没有一个这样定义
func UI_LoadFromJson()

//假想，其实目前没有一个这样定义
func UI_GetID()
func UI_GetFromID()
func UI_Get_ID()


//原来 lua 版本的如下
function CreateFromJson(parent, fn)
    local json_src = GetFileJsonSrc(fn)
    local uijson = Create_ui_json()
    local dom_form = uijson_JsonToUIControl(parent, json_src, uijson)
    uijson.root_view = dom_form
    return uijson
end

//C# godot 版本倒确实是有 getid 这样的命名，似乎也挺清晰，特别是和具体类关联的时候
T GetId<T>
public static T GetId<T>(Node root, string nodeName) where T : Node
Button GetIdButton(Node root, string nodeName)

//--
意外 :)
看到以前的 java 版本有这个一个

public class ui_json {

    //根据控件的 id 取得控件
    public View GetView(String _id)
    {
        return this.nodes.get(_id);
    }//
...

似乎确实更合理 :)

```

又找到一个 C 语言配合 java 的 ndk 版本，这个更完整，以后再和前面的合并整理吧。

文件名为 c_ui_json.h

```

#ifndef _C_UI_JSON_H_
#define _C_UI_JSON_H_

//统一的 c 语言访问 view 接口 //ios 版应当与这个是同一文件
//.c 文件里中可以有 ndk 的文件，但 .h 文件里不能有

#ifdef __cplusplus
extern "C" {
#endif

//---------------------------------------------------------------

//#include <jni.h>

#include "base_c.h"
#include "lstring.h"


//全部用 chandle 或者 jobject,jclass 这样的太容易出错了。人为给它们加一个类型，以便编译器验证
//需要和 java/ios 交互的类型才需要这样做。所以看到参数是结构体的(而不是结构体指针)就知道这是 ui 控件
struct st_ui_json{
    chandle handle;


    //---------------------------------------------------------------
    //以下为方便性的函数指针

    //创建一个视图
    chandle (*CreateView)(chandle parentView);

    //从 json 中创建
    //ui_json CreateFromJson(chandle parentView, const char * fn);

    //取一个视图
    chandle (*GetView)(struct st_ui_json uijson, const char * id);

    chandle (*GetParentView)(chandle view);

    //取根视图
    //chandle ui_json_GetRootView(struct st_ui_json uijson);
    chandle (*GetRootView)(struct st_ui_json uijson);

    //取根 json
    //chandle ui_json_GetRootJson(struct st_ui_json uijson);
    chandle (*GetRootJson)(struct st_ui_json uijson);


    //对齐各个控件
    //void ui_json_AlignControl(struct st_ui_json uijson);
    void (*AlignControl)(struct st_ui_json uijson);

};
typedef struct st_ui_json cui_json;


//绑定函数
void ui_json_BindFunctions(cui_json * uijson);


//创建一个视图
chandle CreateView(chandle parentView);

chandle UI_CreateViewClass(chandle parentView, const char * ui_class);

//从 json 中创建
//cui_json CreateFromJson_(chandle parentView, const char * fn);
cui_json UI_CreateFromJson(chandle parentView, const char * fn);

//一般的操作都得到的是短期局部引用，要得到永久可用的指针，在 java 下必须用这个转换
//比如将 getview 的结果赋值给 this 成员变量时
//要注意的是目前测试情况看 ios 不加这个转换也可以，但是最好是加上去
chandle long_ref(chandle obj);

//取操作系统语言//"cn", "en"
lstring * GetOSLanguage(mempool * mem);

//未证实的函数 //检测一个引用是否已经回收
//int check_ref(JNIEnv* env, jobject obj);
int check_ref(chandle obj);

//取根视图
chandle ui_json_GetRootView(cui_json uijson);

//取根 json
chandle ui_json_GetRootJson(cui_json uijson);


//对齐各个控件
void ui_json_AlignControl(cui_json uijson);

//对齐子控件 //手工修改了控件有没睡的时候才使用，一般不要用
void ui_json_AlignControlSub(cui_json uijson, chandle view);

//----  虚拟 div 的操作
void ui_json_UI_Div_SetTxtColor(cui_json uijson, const char * id, const char * hexColor);

void ui_json_UI_Div_SetTxt(cui_json uijson, const char * id, const char * txt);

void ui_json_UI_Div_SetImage(cui_json uijson, const char * id, const char * image);

//关闭软件键盘，iphone6s 肯定是需要的，安卓待测试
void ui_json_UI_Div_CloseKeyboard(cui_json uijson);

//通过 对象 设置标题 //为了支持动态创建，确实还是需要这样的另一个版本的函数的
void ui_json_UI_Div_SetTxt_forObj(cui_json uijson, chandle view, const char * txt);

//设置语言包
void ui_json_UI_SetLanguage(cui_json uijson, const char * lng);

void ui_json_UI_Div_SetTxtColor_forObj(cui_json uijson, chandle view, const char * hexColor);

void ui_json_UI_Div_SetImage_forObj(cui_json uijson, chandle view, const char * fn_image);

//字符串居左边
void ui_json_UI_Div_SetTxtAlignLeft(cui_json uijson, chandle view);

//取 edit 控件字符串
lstring * ui_json_UI_GetEditText(chandle view, mempool * mem);

//设置 edit 控件字符串
lstring * ui_json_UI_SetEditText(chandle view, const char * text);

//设置 edit 为密码框
lstring * ui_json_UI_SetEditPass(chandle view, int as_pass);

//设置圆角
void ui_json_UI_Div_SetRadius_forObj(cui_json uijson, chandle view, const char * radius);

//ndk 版本，找不到函数声明的话不会报错，但在运行时直接崩溃！！！
lstring * java_UI_LG(const char * key, const char * cn_ref, mempool * mem);

//----  虚拟 div 的操作 _end;

//设置单实例定时器，目前只是给 lua 用的
void ui_json_UI_SetOnTime(cint64 func_c);

//---------------------------------------------------------------

#ifdef __cplusplus
}
#endif



#endif

```

--------------------------------------------------------
**应该放弃的接口**

1.主要是 view 大小在 html 环境下不一定能取到

```

//新代码最高用这个指定高度的版本
function UI_AlignTop_(handle: VViewHandle, height: number): void {
    handle.style.left = "0px";
    handle.style.top = "0px";
    handle.style.width = "100%";
    handle.style.height = `${height}px`;
}//

/**
 * UI_AlignTop：贴父顶部，横向铺满，**保留自身原有高度**
 * @param handle 视图句柄
 */
function UI_AlignTop(handle: VViewHandle): void {

    // 强制触发一次布局，减少append后马上读得到0的概率
    //handle.getBoundingClientRect();

    const h = handle.offsetHeight;  // offsetHeight 在布局未完成时可能会是 0 ,所以理论上 delphi 的这种 api 是有
    //缺陷的，最好是用 UI_AlignTop_() 这种指定高度的版本

    handle.style.left = "0px";
    handle.style.top = "0px";
    handle.style.width = "100%";
    handle.style.height = `${h}px`;
}//

```





--------------------------------------------------------
尽量更新主文件
https://github.com/clqsrc/ui_json/edit/main/README.md






