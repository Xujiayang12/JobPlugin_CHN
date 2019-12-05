# 音符操作
## 音符定义
### 基本型 VSLuaNote
基本型用于简单的音符创建
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
拓展型常用于音符属性修改
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

## 通过游标获取到音乐选区的音符
首先，VSSeekToBeginNote()，这会把游标放到这个序列的第一个音符之前
然后，可以通过VSGetNextNote()或VSGetNextNoteEx()，获取到游标之后的音符。稍后，通过反复调用VSGetNextNote()或VSGetNextNoteEx()，获取到序列的所以音符。

<p align="center" class="logo-img">
    <img src="/static/img/note.png">
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