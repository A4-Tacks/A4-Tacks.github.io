# Keynav

这是一个鼠标模拟点击软件, 利用了多叉树划分区域并点击中心点的思想, 高效完成离散点击

相比于使用上下左右键控制鼠标的无障碍功能, [keynav] 在离散点击效率要高很多, 并且支持录制宏

相应的连续点击效率就会差很多, 例如用于玩扫雷数独等

离散点击的场景比如玩 gal, 操作 linuxqq 等部分 GUI 软件 (我曾半年电脑未接入关于鼠标的输入设备),
对于部分场景还是有更好用的工具的, 像是浏览器的 vimium, keynav 的效率就偏差了, 只是作为一个备选


平台
-------------------------------------------------------------------------------
[keynav] 主要是用于 linux 平台下的 X11 框架, 在 windows 平台有一个功能阉割的 [keynavish] 移植

而在 linux wayland 及其它平台似乎有其它实现, 不过我没有体验过


操作
-------------------------------------------------------------------------------
我个人习惯用四叉切分为主, 默认键位设置是二叉为主, 效率偏低且不太好按只是好上手

好像还支持六叉九叉, 但我认为人很难建立起这种复杂度的肌肉记忆以供高效点击


我的配置
-------------------------------------------------------------------------------
我使用 `d` `f` 来切分左上和右下, 使用 `j` `k` 来切分左下和右上, 这是可以直接输入的高频键

左键, 右键, 中键点击使用 `i` `o` `p`, 左右键点击并退出的版本是 空格 和 `v`

而需要在中心点附近小范围选中时利用 `s` `e` `r` 来缩小或扩展选区, 可视屏幕大小调整, 这是一个硬编码的像素值.
范围更大的 '选中范围初始化' 使用开始选择的 `Ctrl+;` 或者选中窗口的 `w` 来进行

鼠标滚轮参考接近 shell 操作风格, 使用 `b` `g` 进行 向上/向下 翻页, 不过习惯起来比较别扭,
这一块的键要么和滚轮方向打架要么和 shell 操作习惯打架, 我选择了前者.
我还通过 `[` `]` 来进行多倍速滚动, 浏览长网页和聊天记录时有用

宏录制默认键位是使用 `q` 录制且使用 `@` 调用, 这是来自 vim 键风格,
考虑到 keynav 的应用场景几乎都是在调用宏, 我将 `q` `@` 对调了一下, 使用 `q` 来调用宏

关于连续操作, 当退出了点击模式, 下次进入时可以用 `c` 将区域接近在鼠标区域,
又或者可以用 `a` 来回退选区状态

嗯… 虽然我主要使用四叉切分, 但当目标点在当前中心点坐标轴上, 直接二叉切分会比较方便,
我保留了 `u` `n` `h` `l` 来进行上下左右的切分

```configure
#daemonize

clear  # clear all default

ctrl+semicolon start
Escape end
x end
ctrl+bracketleft end  # ctrl+左方括号

h cut-left
n cut-down
u cut-up
l cut-right
d cut-left,cut-up
k cut-right,cut-up
j cut-left,cut-down
f cut-right,cut-down
shift+h move-left
shift+n move-down
shift+u move-up
shift+l move-right
shift+d move-left,move-up
shift+k move-right,move-up
shift+j move-left,move-down
shift+f move-right,move-down
# 在使用keynav时方便地按住ctrl
# Handy for holding ctrl while using keynav:
ctrl+h cut-left
ctrl+n cut-down
ctrl+u cut-up
ctrl+l cut-right
ctrl+d cut-left,cut-up
ctrl+k cut-right,cut-up
ctrl+j cut-left,cut-down
ctrl+f cut-right,cut-down

# 单击
space warp,click 1,end # 在中间左键单击并结束
t warp,click 2,end
v warp,click 3,end
# 单击 shift
shift+space warp,click 1 shift,end
shift+t warp,click 2 shift,end
shift+v warp,click 3 shift,end
# 单击 ctrl
ctrl+space warp,click 1 ctrl,end
ctrl+t warp,click 2 ctrl,end
ctrl+v warp,click 3 ctrl,end
# 单击 ctrl shift
ctrl+shift+space warp,click 1 ctrl shift,end
ctrl+shift+t warp,click 2 ctrl shift,end
ctrl+shift+v warp,click 3 ctrl shift,end
# 单击 alt
alt+space warp,click 1 alt,end
alt+t warp,click 2 alt,end
alt+v warp,click 3 alt,end

# 双击, (Return 键大小写敏感)
Return warp,doubleclick 1,end
shift+Return warp,doubleclick 1 shift,end
ctrl+Return warp,doubleclick 1 ctrl,end
ctrl+shift+Return warp,doubleclick 1 ctrl shift,end
alt+Return warp,doubleclick 1 alt,end

semicolon warp,end    # 移动鼠标并结束
shift+semicolon warp  # 移动鼠标但不结束
# 鼠标处点击而不是中心处
4 click 4 # 上滚轮
5 click 5 # 下滚轮
6 click 6 # 左滚轮
7 click 7 # 右滚轮
# 点击但不结束 shift
shift+4 click 4 shift
shift+5 click 5 shift
shift+6 click 6 shift
shift+7 click 7 shift
# 点击但不结束 ctrl
ctrl+4 click 4 ctrl
ctrl+5 click 5 ctrl
ctrl+6 click 6 ctrl
ctrl+7 click 7 ctrl
# 点击但不结束 alt
alt+4 click 4 alt
alt+5 click 5 alt
alt+6 click 6 alt
alt+7 click 7 alt

# 特殊点击, 用于处理一些keynav模拟不到位的情况, 可酌情扩展
8 warp,sh "xdotool mousedown 1; sleep 0.2; xdotool mouseup 1"
9 warp,sh "xdotool mousedown 2; sleep 0.2; xdotool mouseup 2"
0 warp,sh "xdotool mousedown 3; sleep 0.2; xdotool mouseup 3"


# 中心处的滚轮
b warp,click 4 # 上滚轮
g warp,click 5 # 下滚轮
shift+b warp,click 4 shift # 左滚轮
shift+g warp,click 5 shift # 右滚轮
# 中心处的滚轮 ctrl
ctrl+b warp,click 4 ctrl
ctrl+g warp,click 5 ctrl
ctrl+shift+b warp,click 4 shift ctrl
ctrl+shift+g warp,click 5 shift ctrl
# 中心处的滚轮 alt
alt+b warp,click 4 alt
alt+g warp,click 5 alt
alt+shift+b warp,click 4 shift alt
alt+shift+g warp,click 5 shift alt
# 中心处的滚轮 ctrl+alt
ctrl+alt+b warp,click 4 ctrl alt
ctrl+alt+g warp,click 5 ctrl alt
ctrl+alt+shift+b warp,click 4 shift ctrl alt
ctrl+alt+shift+g warp,click 5 shift ctrl alt

# 快速滚动
bracketleft  warp,click 4,click 4
bracketright warp,click 5,click 5
alt+bracketleft  warp,click 4,click 4,click 4
alt+bracketright warp,click 5,click 5,click 5

# 点击中心处但不结束
i warp,click 1
o warp,click 3
p warp,click 2
# ctrl
ctrl+i warp,click 1 ctrl
ctrl+o warp,click 3 ctrl
ctrl+p warp,click 2 ctrl
# shift
shift+i warp,click 1 shift
shift+o warp,click 3 shift
shift+p warp,click 2 shift
# ctrl shift
ctrl+shift+i warp,click 1 ctrl shift
ctrl+shift+o warp,click 3 ctrl shift
ctrl+shift+p warp,click 2 ctrl shift
# alt
alt+i warp,click 1 alt
alt+o warp,click 3 alt
alt+p warp,click 2 alt

# period: `.`
# 拖动
m drag 1
comma drag 3
period drag 2
# 拖动 ctrl
ctrl+m drag 1 ctrl
ctrl+comma drag 3 ctrl
ctrl+period drag 2 ctrl
# 拖动 shift
shift+m drag 1 shift
shift+comma drag 3 shift
shift+period drag 2 shift
# 拖动 ctrl shift
ctrl+shift+m drag 1 ctrl+shift
ctrl+shift+comma drag 3 ctrl+shift
ctrl+shift+period drag 2 ctrl+shift
# 拖动 alt
alt+m drag 1 alt
alt+comma drag 3 alt
alt+period drag 2 alt

# 缩放到当前窗口
# Zoom to the current window
w windowzoom
ctrl+w windowzoom
shift+w windowzoom

# 缩放到具有给定高度和宽度的光标位置
# Zoom to the cursor location with a given height and width
c cursorzoom 200 200
ctrl+c cursorzoom 200 200
# 用于扩大选区的一种方式
s warp,cursorzoom 200 200
shift+c warp,cursorzoom 200 200
ctrl+shift+c cursorzoom 200 200
# 横向|纵向扩展选区
e warp,cursorzoom 400 400
r warp,cursorzoom 800 800

# 箭头键也可以移动网格
# Arrow keys can move the grid as welll
Left cut-left
Down cut-down
Up cut-up
Right cut-right
shift+Left move-left
shift+Down move-down
shift+Up move-up
shift+Right move-right
ctrl+Left cut-left
ctrl+Down cut-down
ctrl+Up cut-up
ctrl+Right cut-right
ctrl+shift+Left move-left
ctrl+shift+Down move-down
ctrl+shift+Up move-up
ctrl+shift+Right move-right

# 记录keynav行动, q? @?
# Record keynav actions
shift+at record
q playback

# History
a history-back
```

这份配置应该放到 '家目录' 中, 像是 `~/.keynavrc`,
对于 [keynavish], 文件名建议用 `_keynavrc`

> * `<executable path>/keynavrc` (for portability)
> * `~/.keynavrc`
> * `~/keynavrc`
> * `~/.config/keynav/keynavrc`
> 
> Tildes (`~`) in paths are expanded to the value of `%HOME%` if it's set,
> with fallback to `%USERPROFILE%` (usually `C:\Users\<username>`).

> [!NOTE]
> 这篇的主要目的其实是把我 private 仓库的完整配置, 裁剪出一份比较方便引用的配置


[keynav]: https://github.com/jordansissel/keynav
[keynavish]: https://github.com/lesderid/keynavish
