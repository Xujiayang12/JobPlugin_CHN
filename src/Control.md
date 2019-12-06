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
