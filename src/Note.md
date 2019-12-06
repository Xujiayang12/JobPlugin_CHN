# 音符操作
## 音符定义
### 基本型 VSLuaNote
用于简单的音符创建
```lua
local note = {}	
note.posTick	= 1920  -- 音符起始点
note.durTick	= 480   -- 音符时长
note.noteNum	= 69    -- 音符音高
note.velocity	= 64    -- 音符VEL值
note.lyric	= "wan" -- 音符歌词
note.phonemes	= "ua_n"-- 音符音标
note.phLock     = 1     -- 音标保护 1为开启，0为关闭，默认0 这个一般拆音插件都要开着的
```
### 拓展型 VSLuaNoteEx
常用于音符属性修改
```lua
local note = {}	
note.posTick	= 1920  -- 音符起始点
note.durTick	= 480   -- 音符时长
note.noteNum	= 69    -- 音符音高
note.velocity	= 64    -- 音符VEL值
note.lyric	= "wan" -- 音符歌词
note.phonemes	= "ua_n"-- 音符音标
note.phLock     = 1     -- 音标保护 1为开启，0为关闭，默认0 这个一般拆音插件都要开着的

note.bendDepth = 20  -- 弯音深度 0~100
note.bendLength = 20  -- 弯音长度 0~100
note.risePort = 0  -- Addition of portamento flag on line form 为布尔值0或1
note.fallPort = 0   -- Portamento additional flag in the descending form 为布尔值0或1
note.decay = 20  -- 延迟 0~100
note.accent = 0 -- 0~100
note.opening = 127  -- 0~100
note.vibratoLength = 0  -- 颤音长度
note.vibratoType = 0  -- 颤音类型 0：没有颤音 1-4：Normal 5-8：Extreme 9-12：Fast 13-16：Slight
```

> 写拆音插件时记得把`note.phLock`设为`1`，否则之后音符音标会变回去。
## 通过游标获取到音乐选区的音符
首先，VSSeekToBeginNote()，这会把游标放到这个序列的第一个音符之前
然后，可以通过VSGetNextNote()或VSGetNextNoteEx()，获取到游标之后的音符。稍后，通过反复调用VSGetNextNote()或VSGetNextNoteEx()，获取到序列的所有音符。

<p align="center" class="logo-img">
    <img src="static/img/note.png">
</p>

## 如何获取用户选定的音符
这里给出一个我认为比较难出错的例子
```lua
function main(processParam, envParam)
    local beginPosTick = processParam.beginPosTick
    local endPosTick = processParam.endPosTick
    local songPosTick = processParam.songPosTick
    local endStatus = 0

    local scriptDir = envParam.scriptDir
    local scriptName = envParam.scriptName
    local tempDir = envParam.tempDir

    local noteEx = {}
    local noteExList = {}
    local noteCount
    local retCode
    local idx

    VSSeekToBeginNote()
    idx = 1
    retCode, noteEx = VSGetNextNoteEx()
    while (retCode == 1) do
        noteExList[idx] = noteEx
        retCode, noteEx = VSGetNextNoteEx()
        idx = idx + 1
    end

    noteCount = table.getn(noteExList)
    if (noteCount == 0) then
        VSMessageBox("你需要选择一个音符", 0)
        return 0
    end

    for idx = 1, noteCount do
        local note = noteExList[idx]
        if (note.posTick >= beginPosTick and note.posTick + note.durTick <= endPosTick) then
           -- 选中区域的音符操作
        end
    end
    return endStatus
end
```

## 音符信息的更新(修改)

通过上面的音符遍历获取到选区的音符后，可以直接对音符属性进行修改然后进行更新
> - `VSGetNextNote()`对应使用`VSUpdateNote(VSLuaNote note)`
> - `VSGetNextNoteEx()`对应使用`VSUpdateNoteEx(VSLuaNoteEx note)`


```lua
function main(processParam, envParam)
    local beginPosTick = processParam.beginPosTick
    local endPosTick = processParam.endPosTick
    local songPosTick = processParam.songPosTick
    local endStatus = 0

    local scriptDir = envParam.scriptDir
    local scriptName = envParam.scriptName
    local tempDir = envParam.tempDir

    local noteEx = {}
    local noteExList = {}
    local noteCount
    local retCode
    local idx

    VSSeekToBeginNote()
    idx = 1
    retCode, noteEx = VSGetNextNoteEx()
    while (retCode == 1) do
        noteExList[idx] = noteEx
        retCode, noteEx = VSGetNextNoteEx()
        idx = idx + 1
    end

    noteCount = table.getn(noteExList)
    if (noteCount == 0) then
        VSMessageBox("你需要选择一个音符", 0)
        return 0
    end

    for idx = 1, noteCount do
        local note = noteExList[idx]
        if (note.posTick >= beginPosTick and note.posTick + note.durTick <= endPosTick) then
            note.lyric = "wo"
            VSUpdateNoteEx(note)
            -- 选区的音符歌词全改为wo
        end
    end
    return endStatus
end
```

## 音符的创建
可以在指定位置创建新的音符，尽量不可以使用遍历获取到的音符修改后进行音符创建，会出错。假如想在原音符修改后进行创建，需要对音符的table进行复制。
> - 基本型音符创建 VSInsertNote( VSLuaNote note )
> - 拓展型音符创建 VSInsertNoteEx( VSLuaNoteEx note )

### 例子1——从零创建
```lua
local note = {}	
note.posTick	= 1920
note.durTick	= 480
note.noteNum	= 69
note.velocity	= 64
note.lyric	= "wan"
note.phonemes	= "ua_n"
note.phLock     = 1

VSInsertNote(note)
```

### 例子2——复制已有创建
```lua
function copyNoteEx(note_ex_src)
    local note_ex_new = {}
    --Normal Note Properties
    note_ex_new.posTick = note_ex_src.posTick
    note_ex_new.durTick = note_ex_src.durTick
    note_ex_new.noteNum = note_ex_src.noteNum
    note_ex_new.velocity = note_ex_src.velocity
    note_ex_new.phonemes = note_ex_src.phonemes
    note_ex_new.lyric = note_ex_src.lyric
    --Extended Note Properties
    note_ex_new.bendDepth = note_ex_src.bendDepth
    note_ex_new.bendLength = note_ex_src.bendLength
    note_ex_new.risePort = note_ex_src.risePort
    note_ex_new.fallPort = note_ex_src.fallPort
    note_ex_new.decay = note_ex_src.decay
    note_ex_new.accent = note_ex_src.accent
    note_ex_new.opening = note_ex_src.opening
    note_ex_new.vibratoLength = note_ex_src.vibratoLength
    note_ex_new.vibratoType = note_ex_src.vibratoType
    return note_ex_new
end

function main(processParam, envParam)
    local beginPosTick = processParam.beginPosTick
    local endPosTick = processParam.endPosTick
    local songPosTick = processParam.songPosTick
    local endStatus = 0

    local scriptDir = envParam.scriptDir
    local scriptName = envParam.scriptName
    local tempDir = envParam.tempDir

    local noteEx = {}
    local noteExList = {}
    local noteCount
    local retCode
    local idx

    VSSeekToBeginNote()
    idx = 1
    retCode, noteEx = VSGetNextNoteEx()
    while (retCode == 1) do
        noteExList[idx] = noteEx
        retCode, noteEx = VSGetNextNoteEx()
        idx = idx + 1
    end

    noteCount = table.getn(noteExList)
    if (noteCount == 0) then
        VSMessageBox("你需要选择一个音符", 0)
        return 0
    end

    for idx = 1, noteCount do
        local note = noteExList[idx]
        if (note.posTick >= beginPosTick and note.posTick + note.durTick <= endPosTick) then
            local newNote = copyNoteEx(note)
            newNote.lyric = "x a"
            VSInsertNoteEx(newNote)
        end
    end
    return endStatus
end
```

## 音符删除
这个只有`VSRemoveNote(VSLuaNote note)`一个API，我猜传入拓展型的音符应该也可以删除，毕竟基本型的属性拓展型也有，能读取到信息删除就行了。

## 官方案例
### 案例1——NoteSample1

```lua
-- NoteSample1.lua
-- ノートの取得/更新のサンプル.
--

--
-- Copyright (C) 2011 Yamaha Corporation
--

--
-- プラグインマニフェスト関数.
--
function manifest()
    myManifest = {
        name          = "ノートの取得/更新のサンプル",
        comment       = "ノートの取得/更新のサンプルJobプラグイン",
        author        = "Yamaha Corporation",
        pluginID      = "{77E0B197-7E0D-46b5-A78E-FCCE63545372}",
        pluginVersion = "1.0.0.1",
        apiVersion    = "3.0.0.1"
    }
    
    return myManifest
end


--
-- VOCALOID3 Jobプラグインスクリプトのエントリポイント.
--
function main(processParam, envParam)
	-- 実行時に渡されたパラメータを取得します.
	local beginPosTick = processParam.beginPosTick	-- 選択範囲の始点時刻（ローカルTick）.
	local endPosTick   = processParam.endPosTick	-- 選択範囲の終点時刻（ローカルTick）.
	local songPosTick  = processParam.songPosTick	-- カレントソングポジション時刻（ローカルTick）.

	-- 実行時に渡された実行環境パラメータを取得します.
	local scriptDir  = envParam.scriptDir	-- Luaスクリプトが配置されているディレクトリパス（末尾にデリミタ "\" を含む）.
	local scriptName = envParam.scriptName	-- Luaスクリプトのファイル名.
	local tempDir    = envParam.tempDir		-- Luaプラグインが利用可能なテンポラリディレクトリパス（末尾にデリミタ "\" を含む）.


	local noteEx     = {}
	local noteExList = {}
	local noteCount
	local retCode
	local idx

	-- ノートを取得してノートイベント配列へ格納します.
	VSSeekToBeginNote()
	idx = 1
	retCode, noteEx = VSGetNextNoteEx()
	while (retCode == 1) do
		noteExList[idx] = noteEx
		retCode, noteEx = VSGetNextNoteEx()
		idx = idx + 1
	end

	-- 読み込んだノートの総数.
	noteCount = table.getn(noteExList)
	if (noteCount == 0) then
		VSMessageBox("読み込んだノートがありません.", 0)
		return 0
	end
	
	
	-- 取得したノートイベントを更新します.
	for idx = 1, noteCount do
		local updNoteEx = {}
		updNoteEx = noteExList[idx]
		
		updNoteEx.lyric    = "ら"
		updNoteEx.phonemes = "4 a"
		
		updNoteEx.bendDepth = 100
		updNoteEx.bendLength = 90
		updNoteEx.risePort = 1
		updNoteEx.fallPort = 1
		updNoteEx.decay = 70
		updNoteEx.accent = 80
		updNoteEx.opening = 20
		updNoteEx.vibratoLength = 100
		updNoteEx.vibratoType = 4
		
		retCode = VSUpdateNoteEx(updNoteEx);
		if (retCode ~= 1) then
			VSMessageBox("更新エラー発生!!", 0)
			return 1
		end
	end


	-- 取得したノートイベントの音程を移調してシーケンスの末尾へ追加します.
	local endPos1 = noteExList[noteCount].posTick + noteExList[noteCount].durTick + 1920
	local endPos2 = endPos1
	for idx = 1, noteCount do
		local newNoteEx = {}
		newNoteEx = noteExList[idx]
		
		endPos2 = endPos1 + newNoteEx.posTick
		
		newNoteEx.posTick = endPos2
		newNoteEx.noteNum = newNoteEx.noteNum + 2
		
		endPos2 = endPos2 + newNoteEx.durTick

		retCode = VSInsertNoteEx(newNoteEx);
		if (retCode ~= 1) then
			VSMessageBox("追加エラー発生!!", 0)
			return 1
		end
	end


	-- MusicalパートのplayTimeを更新します.
	local musicalPart = {}
	retCode, musicalPart = VSGetMusicalPart()
	musicalPart.playTime = endPos2
	retCode = VSUpdateMusicalPart(musicalPart)
	if (retCode ~= 1) then
		VSMessageBox("MusicalパートのplayTimeを更新できません!!", 0)
		return 1
	end


	-- 正常終了.
	return 0
end

```

### 案例2——NoteSample2

```lua

--
-- NoteSample2.lua
-- ノートの削除のサンプル.
--

--
-- Copyright (C) 2011 Yamaha Corporation
--

--
-- プラグインマニフェスト関数.
--
function manifest()
    myManifest = {
        name          = "ノートの削除のサンプル",
        comment       = "ノートの削除のサンプルJobプラグイン",
        author        = "Yamaha Corporation",
        pluginID      = "{6B749178-0913-47e0-97D5-4BD8DEAE620A}",
        pluginVersion = "1.0.0.1",
        apiVersion    = "3.0.0.1"
    }
    
    return myManifest
end


--
-- VOCALOID3 Jobプラグインスクリプトのエントリポイント.
--
function main(processParam, envParam)
	-- 実行時に渡されたパラメータを取得します.
	local beginPosTick = processParam.beginPosTick	-- 選択範囲の始点時刻（ローカルTick）.
	local endPosTick   = processParam.endPosTick	-- 選択範囲の終点時刻（ローカルTick）.
	local songPosTick  = processParam.songPosTick	-- カレントソングポジション時刻（ローカルTick）.

	-- 実行時に渡された実行環境パラメータを取得します.
	local scriptDir  = envParam.scriptDir	-- Luaスクリプトが配置されているディレクトリパス（末尾にデリミタ "\" を含む）.
	local scriptName = envParam.scriptName	-- Luaスクリプトのファイル名.
	local tempDir    = envParam.tempDir		-- Luaプラグインが利用可能なテンポラリディレクトリパス（末尾にデリミタ "\" を含む）.


	local note     = {}
	local noteList = {}
	local noteCount
	local retCode
	local idx

	-- ノートを取得してノートイベント配列へ格納します.
	VSSeekToBeginNote()
	idx = 1
	retCode, note = VSGetNextNote()
	while (retCode == 1) do
		noteList[idx] = note
		retCode, note = VSGetNextNote()
		idx = idx + 1
	end

	-- 読み込んだノートの総数.
	noteCount = table.getn(noteList)
	if (noteCount == 0) then
		VSMessageBox("読み込んだノートがありません.", 0)
		return 0
	end
	
	
	-- 選択範囲のノートイベントを削除します.
	for idx = 1, noteCount do
		local delNote = {}
		delNote = noteList[idx]
		
		if (beginPosTick <= delNote.posTick and delNote.posTick <= endPosTick) then
			retCode = VSRemoveNote(delNote);
			if (retCode ~= 1) then
				VSMessageBox("削除エラー発生!!", 0)
				return 1
			end
		end
	end


	-- 正常終了.
	return 0
end

```
