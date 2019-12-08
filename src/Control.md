# 参数操作
## 参数定义
一个参数类型包括参数位置、参数类型、参数值，参数有8种

官方的解说: 
```lua
Type of control parameter (string).
"DYN": Dynamics
"BRE": Bureshinesu
"BRI": Brightness
"CLE": Clearnes
"GEN": Gender factor
"PIT": Pitch
"PBS": Pitch Bend Sensitivity
"POR": Portamento timing
```

### 参数定义例子

```lua
local control = {}
control.posTick = 180
control.value = 30
control.type = "DYN"
```

## 通过时间点获取参数信息
只要能够获得到想要操作参数的时间点位置，就可以对参数进行信息获取、修改、添加与删除。如利用主函数在开头传输的三个时间点:

```lua
-- 获取音乐选区参数
beginPosTick = processParam.beginPosTick  -- 音乐选区起始点
endPosTick   = processParam.endPosTick    -- 音乐选区结束点
songPosTick  = processParam.songPosTick   -- 光标位置点
```

<p align="center" class="logo-img">
    <img src="static/img/control.png">
</p>

### 获取光标处的PIT值
```lua
local r, pitVal = VSGetControlAt("PIT", songPosTick)
```
> 第一次写插件时我经常漏掉前面的返回值，所以传到的总是1，在此提醒一下大家（雅马哈也太奇葩了，怎么不把返回值放在第二个返回参数）
## 参数的修改更新
使用`VSUpdateControlAt`传入类型，数值和参数位置就可以进行更新了。
```lua

VSUpdateControlAt("DYN", 20, songPosTick)
```
## 提示
很多时候对某一段的每个点的值进行进行计算和修改时，需要将所有的点的数值先提取出来，计算后，再放入循环中更新，而且在修改后，最后一个点最好恢复原值，不能直接在循环中边获取数值计算数值和更新数值，这是因为V会自动把后面没有设定参数值的部分自动更新为有设定参数值的最后一个点的值。
比如下面这种情形：
<p align="center" class="logo-img">
    <img src="static/img/control2.png">
</p>

先提取出PIT数值数组
```lua

 -- 获取原来此区间内的PIT（数组）
    local oriIdx = 0
    local PITArray = {}
    for i = beginPosTick, endPosTick do
        local r, PITVal
        r, PITVal = VSGetControlAt("PIT", i)
        PITArray[oriIdx] = PITVal
        oriIdx = oriIdx + 1
    end
    oriIdx = 0
```
## 通过游标对参数的操作
这种操作类似与音符的操作，不过在实际上比较少用到，所以不在这里赘述。详情前往阅读英文版
[The sequential access method API with Cursor](https://jobplugineng.now.sh/#/src/control#the-sequential-access-method-api-with-cursor)

## 官方案例

### ControlSample1.lua

```lua

function manifest()
    myManifest = {
        name = "获取控制参数信息",
        comment = "获取控制参数信息的JobPlugin",
        author = "雅马哈公司",
        pluginID = "{5F2176EE-2B4A-44ea-89B0-B7687085749F}",
        pluginVersion = "1.0.0.1",
        apiVersion = "3.0.0.1"
    }
    return myManifest
end

function main(processParam, envParam)
    -- 获取执行时传递的处理条件参数。
    local beginPosTick = processParam.beginPosTick    -- 选中的开始时刻（本地Tick）.
    local endPosTick = processParam.endPosTick    -- 选中的终止时刻（本地Tick）.
    local songPosTick = processParam.songPosTick    -- 当前光标所处位置（本地Tick）.
    local TickLength = endPosTick - beginPosTick -- 选中长度
    VSMessageBox(TickLength, 0)

    -- 获得此Lua脚本的效果信息
    local scriptDir = envParam.scriptDir    -- Lua脚本所在的目录路径（末尾包括定界符“ \”）
    local scriptName = envParam.scriptName    -- Lua脚本文件名。
    local tempDir = envParam.tempDir        -- 可以使用Lua插件的临时目录路径（末尾包括定界符“ \”）。

    local retCode -- 返回值 1 或 0 ？
    local controlVal = 0
    local controlMin = 128
    local controlMax = 0
    local controlAvg = 0

    -- 查找选定范围内参数的最大值（此例子是DYN）的最大/最小/平均值。
    for posTick = beginPosTick, endPosTick do
        retCode, controlVal = VSGetControlAt("DYN", posTick)
		VSMessageBox(controlVal, 0)
        if (controlVal < controlMin) then
            controlMin = controlVal
        end
        if (controlMax < controlVal) then
            controlMax = controlVal
        end
        controlAvg = controlAvg + controlVal
    end
    if (beginPosTick < endPosTick) then
        controlAvg = controlAvg / (endPosTick - beginPosTick)
    else
        controlAvg = -1
    end

    -- 显示结果.
    local msg
    msg = "在选择的范围内，此参数的,\n" ..
            "  最大値 = [" .. controlMax ..
            "]\n  最小値 = [" .. controlMin ..
            "]\n  平均値 = [" .. controlAvg ..
            "]\n"
    VSMessageBox(msg, 0)
    VSMessageBox(TickLength, 0)
    return 0
end
```

### ControlSample2.lua

```lua

-- ControlSample2.lua
function manifest()
    myManifest = {
        name          = "コントロールパラメータの取得/更新のサンプル",
        comment       = "コントロールパラメータの取得/更新のサンプルJobプラグイン",
        author        = "Yamaha Corporation",
        pluginID      = "{A00AF690-A52F-4f3a-AE29-2D6EAE732907}",
        pluginVersion = "1.0.0.1",
        apiVersion    = "3.0.0.1"
    }
    
    return myManifest
end


--
-- VOCALOID3 Jobプラグインスクリプトのエントリポイント.
--
function main(processParam, envParam)
	-- 実行時に渡された処理条件パラメータを取得します.
	local beginPosTick = processParam.beginPosTick	-- 選択範囲の始点時刻（ローカルTick）.
	local endPosTick   = processParam.endPosTick	-- 選択範囲の終点時刻（ローカルTick）.
	local songPosTick  = processParam.songPosTick	-- カレントソングポジション時刻（ローカルTick）.

	-- 実行時に渡された実行環境パラメータを取得します.
	local scriptDir  = envParam.scriptDir	-- Luaスクリプトが配置されているディレクトリパス（末尾にデリミタ "\" を含む）.
	local scriptName = envParam.scriptName	-- Luaスクリプトのファイル名.
	local tempDir    = envParam.tempDir		-- Luaプラグインが利用可能なテンポラリディレクトリパス（末尾にデリミタ "\" を含む）.


	local control = {}
	local controlList = {}
	local controlNum
	local retCode
	local idx


	-- ダイナミクスコントロールイベントを取得します.
	retCode = VSSeekToBeginControl("DYN")
	idx = 1
	retCode, control = VSGetNextControl("DYN")
	while (retCode == 1) do
		controlList[idx] = control
		retCode, control = VSGetNextControl("DYN")
		idx = idx + 1
	end
	controlNum = table.getn(controlList)

	-- 取得したダイナミクスコントロールイベントへゆらぎを付与します.
	local seed = os.time()
	math.randomseed(seed)
	for idx = 1, controlNum do
		local randControl = {}
		randControl = controlList[idx]
		randControl.value = randControl.value + math.random()*5.0
		retCode = VSUpdateControl(randControl)
	end
	
	-- 正常終了.
	return 0
end

```

### ControlSample3.lua

```lua

-- ControlSample3.lua
function manifest()
    myManifest = {
        name          = "コントロールパラメータの追加/削除のサンプル",
        comment       = "コントロールパラメータの追加/削除のサンプルJobプラグイン",
        author        = "Yamaha Corporation",
        pluginID      = "{98EBB5BE-A7D2-447b-80B8-79603BEA7B79}",
        pluginVersion = "1.0.0.1",
        apiVersion    = "3.0.0.1"
    }
    
    return myManifest
end


--
-- VOCALOID3 Jobプラグインスクリプトのエントリポイント.
--
function main(processParam, envParam)
	-- 実行時に渡された処理条件パラメータを取得します.
	local beginPosTick = processParam.beginPosTick	-- 選択範囲の始点時刻（ローカルTick）.
	local endPosTick   = processParam.endPosTick	-- 選択範囲の終点時刻（ローカルTick）.
	local songPosTick  = processParam.songPosTick	-- カレントソングポジション時刻（ローカルTick）.

	-- 実行時に渡された実行環境パラメータを取得します.
	local scriptDir  = envParam.scriptDir	-- Luaスクリプトが配置されているディレクトリパス（末尾にデリミタ "\" を含む）.
	local scriptName = envParam.scriptName	-- Luaスクリプトのファイル名.
	local tempDir    = envParam.tempDir		-- Luaプラグインが利用可能なテンポラリディレクトリパス（末尾にデリミタ "\" を含む）.


	local control = {}
	local controlList = {}
	local controlNum
	local retCode
	local idx


	-- ダイナミクスコントロールイベントを取得します.
	retCode = VSSeekToBeginControl("DYN")
	idx = 1
	retCode, control = VSGetNextControl("DYN")
	while (retCode == 1) do
		controlList[idx] = control
		retCode, control = VSGetNextControl("DYN")
		idx = idx + 1
	end
	controlNum = table.getn(controlList)

	-- 取得したダイナミクスコントロールイベントを全部削除します.
	for idx = 1, controlNum do
		retCode = VSRemoveControl(controlList[idx])
	end

	-- 取得したダイナミクスコントロールイベントを一定間隔で間引きます.
	-- 平均値のイベントを挿入することで,結果として間引きになります.
	local pos1 = 1
	local pos2
	for pos2 = 2, controlNum do
		local control1 = {}
		local control2 = {}
		control1 = controlList[pos1]
		control2 = controlList[pos2]
		
		local dt = control2.posTick - control1.posTick
		if (32 < dt) then
			local newControl = {}
			newControl.posTick = (control1.posTick + control2.posTick) / 2
			newControl.value   = (control1.value   + control2.value) / 2
			newControl.type    = "DYN"
			retCode = VSInsertControl(newControl)
			pos1 = pos2
		end
	end


	-- 正常終了.
	return 0
end

```