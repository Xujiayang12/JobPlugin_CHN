# 插件的基本结构
## 语法
插件的编写语言是Lua，所以插件可以使用Lua语言的所有特性（但是基本包有点少啊）。所以你要编写插件，得先学Lua，最快入门前往此处[Lua菜鸟教程](https://www.runoob.com/lua/lua-tutorial.html)

## 主函数
插件脚本必须始终具有名为“main”的函数。此函数作为插件的主函数。此外，此函数的结尾也是整个插件的结束点。运行时，插件主函数会被传入两个参数，具体见例子
> 主函数运行到最后必须有返回码，假如返回0则应用插件的更改，非0则放弃更改

## 插件信息函数
名为“manifest”的函数为插件提供插件名称，介绍，作者，版本号等信息，和主函数一样缺一不可。
> VOCALOID编辑器不能同时添加两个ID相同的插件否则会报错。所以插件新插件时记得改一下ID

## 例子

```lua
function manifest()
    myManifest = {
        name          = "Do Nothing", -- 插件名
        comment       = "Sample Plugin", -- 插件简介
        author        = "Yamaha Corporation", -- 插件作者
        pluginID      = "{EDC4AAE9-91DF-4f33-ADCA-68A6567A1E0B}", -- 识别插件的唯一ID
                      -- 注意：VOCALOID编辑器不能同时添加两个ID相同的插件否则会报错。
        pluginVersion = "1.0.0.1", -- 插件版本
        apiVersion    = "3.0.0.1" -- 插件API版本，不要改动
    }
    
    return myManifest
end

-- 插件主函数
function main(processParam, envParam)
    -- 获取音乐选区参数
    beginPosTick = processParam.beginPosTick  -- 音乐选区起始点
    endPosTick   = processParam.endPosTick    -- 音乐选区结束点
    songPosTick  = processParam.songPosTick   -- 光标位置点

    -- 获取插件运行信息
    scriptDir  = envParam.scriptDir   -- 插件存放目录
    scriptName = envParam.scriptName  -- 插件文件名
    tempDir    = envParam.tempDir     -- 插件运行时临时可用的目录

    -- 这里时插件内容

    -- 运行最后返回0，正常结束，返回1代表不正常结束，工程也不会发生改动
    return 0
end
```

## 命名约定与数据类型
插件脚本可以使用所有Lua语言提供的语言特性，包括数组、table（表）、nil值等。为了看辨别并避免与这些名称冲突，API的命名遵循以下约定。
 - API名称前缀 ——
插件API名称前缀为“VS”。
 - Lua表名前缀 ——
Lua表的表名（也就是table数据类型）前缀为“VS”。
 - 变量名 ——
Lua表字段名的变量名，以小写字母开头。
> 插件的VSInt32、VSFloat、VSCString都与Lua里面的number、string相兼容，所以不张开讲。
> 有一点，VSBool在插件里面直接用`0`和`1`就行了

## 如何定义结构和获取参数值
### 例子1 通过table(表)类型来定义一个音符
```lua
local note = {}	
note.posTick	= 1920
note.durTick	= 480
note.noteNum	= 69
note.velocity	= 64
note.lyric	= “wan”
note.phonemes	= "ua_n"
```
### 例子2 获取参数值
```lua
local retCode, DYN_value = VSGetControlAt("DYN", posTick)
 -- 获取DYN参数值，retCode时获取状态值，假如获取成功返回1，失败返回0。
```
> 第一次写插件时我经常漏掉前面的返回值，所以传到的总是1，在此提醒一下大家（雅马哈也太奇葩了，怎么不把返回值放在第二个返回参数）

## 插件编写的编码
编码必须为UTF-8，否则报错