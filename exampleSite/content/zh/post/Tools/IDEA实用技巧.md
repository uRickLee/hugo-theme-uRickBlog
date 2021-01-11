---
title: IDEA实用技巧
date: 2020-01-01 23:15:04
image: https://gitee.com/uRick/oss/raw/master/blog/IDEA-IMAGE.png
description: 梳理总结工作中使用到IDEA工具技巧，后期遇到好玩的持续更新
feature: true
categories: 
 - Tools
tags:
  - 开发工具
---

![IDEA](https://gitee.com/uRick/oss/raw/master/blog/IDEA-IMAGE.png "哈哈")
## 1.1.1. 绿色版安装配置

1. 下载：https://www.jetbrains.com/idea/
2. 配置ipr.reg
> 将安装的IDEA关联起来，这里默认都是配置在C盘目录下的。
3. 激活方式：这个将就搜索技巧

```
REGEDIT4

[HKEY_CLASSES_ROOT\.ipr]
@="IntelliJIDEAProjectFile"

[HKEY_CLASSES_ROOT\IntelliJIDEAProjectFile]
@="IntelliJ IDEA Project File"

[HKEY_CLASSES_ROOT\IntelliJIDEAProjectFile\DefaultIcon]
@="F:\\Tools\\IntelliJ IDEA\\bin\\idea.exe,0"

[HKEY_CLASSES_ROOT\IntelliJIDEAProjectFile\shell]

[HKEY_CLASSES_ROOT\IntelliJIDEAProjectFile\shell\open]

[HKEY_CLASSES_ROOT\IntelliJIDEAProjectFile\shell\open\command]
@="\"F:\\Tools\\IntelliJ IDEA\\bin\\idea.exe" \"%1\""

```

4. idea.properties （默认配置在C盘User目录中）配置（**若使用默认配置可略**）

```
# 配置信息
# 指定配置文件路径
 idea.config.path=F:/Tools/IntelliJ IDEA/myConfig
# 系统配置路
 idea.system.path=F:/Tools/IntelliJ IDEA/myConfig/system
# 自定义插件路径
 idea.plugins.path=F:/Tools/IntelliJ IDEA/myConfig/plugins
# 指定平常使用日志的存储路
 idea.log.path=F:/Tools/IntelliJ IDEA/myConfig/log
# 最大文件存储大小（应该是指代码帮助信息存储，这个数值越大，对硬件要求更高，更耗性能）
idea.max.intellisense.filesize=2500
# IDEA 可以打开文件大小
idea.max.content.load.filesize=20000
# 控制台输出缓冲大小，如需要禁用则配置 idea.cycle.buffer.size=disabled
idea.cycle.buffer.size=1024
# 配置在IDE中运行进程时是否应使用特殊启动器。使用Launcher启用“软退出”和“线程转储”功能 不懂啥意思
idea.no.launcher=false
# 避免太长的classpath
idea.dynamic.classpath=false
# IDEA调试使用 通常禁止使用中抛出ProcessCanceledException
#idea.ProcessCanceledException=disabled
# 此属性有两个可选值“heavy”和“medium”，好像同弹出菜单有关系
idea.popup.weight=heavy
# 不可删除次属性，若删除可能会导致windowsd端IDEA使用下降
sun.java2d.d3d=false
#设置swing.bufferPerWindow = false来解决JDK6中的慢速滚动问题（参见IDEA-35883），但这可能会导致JDK8性能下降，因为它会禁用双缓冲，这是消除blit加速滚动撕裂所必需的
恢复帧缓冲区内容，无需通常重新绘制，即使EDT被阻止。
swing.bufferPerWindow=true
# 删除此属性可能会导致X Window下的编辑器性能下降
sun.java2d.pmoffscreen=false
# 在JBRE中启用HiDPI支持
sun.java2d.uiScale.enabled=true
# swing组件缩放调整
javax.swing.rebaseCssSizeMap=true
#在Mac OS X下访问剪贴板时避免长时间挂起的解决方法。
#ide.mac.useNativeClipboard=True
# 最大大小（千字节）IDEA将加载以显示过去的文件内容
#idea.max.vcs.loaded.size.kb=20480
# IDEA文件选择器查看目录内部以检测它们是否包含有效项目（使用相应的图标标记此类目录）。取消注释该选项可防止用户主目录之外的此行为。
#idea.chooser.lookup.for.project.dirs=false
# 实验选项，可以做很多事情来实现真正平滑的滚动，具体查看按照目录解释
#idea.true.smooth.scrolling=true
# IDEA可以复制库.jar文件以防止其锁定。默认情况下，此行为在Windows上启用，在其他平台上禁用。取消注释此属性以覆盖。
# idea.jars.nocopy=false
# 用于在调试模式下启动JVM的VM选项值。一些JRE以不同的方式定义它（Oracle VM中的-XXdebug）
idea.xdebug.key=-Xdebug
# 如果要接收有关安装的IDE或插件发生的致命错误的即时可视通知，请更改为“已启用”。
idea.fatal.error.notification=disabled
```

5. idea.exe.vmoptions配置
**-Xms128m**， 16 G 内存的机器可尝试设置为 -Xms512m
(设置初始的内存数，增加该值可以提高 Java 程序的启动速度。 )
**-Xmx750m**， 16 G 内存的机器可尝试设置为 -Xmx1500m
(设置最大内存数，提高该值，可以减少内存 Garage 收集的频率，提高程序性能)
**-XX:ReservedCodeCacheSize=240m**， 16G 内存的机器可尝试设置为**-XX:ReservedCodeCacheSize=500m**  (保留代码占用的内存容量)


### 1.1.2. 快捷键

#### 1.1.2.1. 从 Windows 过度到 Mac 必备快捷键对照表

##### 1.1.2.1.1. Mac 键盘符号说明

- `⌘` == `Command`
- `⇧` == `Shift`
- `⇪` == `Caps Lock`
- `⌥` == `Option`
- `⌃` == `Control`
- `↩` == `Return/Enter`
- `⌫` == `Delete`
- `⌦` == `向前删除键（Fn+Delete）`
- `↑` == `上箭头`
- `↓` == `下箭头`
- `←` == `左箭头`
- `→` == `右箭头`
- `⇞` == `Page Up（Fn+↑）`
- `⇟` == `Page Down（Fn+↓）`
- `Home` == `Fn + ←`
- `End` == `Fn + →`
- `⇥` == `右制表符（Tab键）`
- `⇤` == `左制表符（Shift+Tab）`
- `⎋` == `Escape (Esc)`
- `⏏` == `电源开关键`

##### 1.1.2.1.2. Ctrl

|Win 快捷键|Mac 快捷键|介绍|
|:---------|:---------|:---------|
|<kbd>Ctrl</kbd> + <kbd>F</kbd>|<kbd>Command</kbd> + <kbd>F</kbd>|在当前文件进行文本查找|
|<kbd>Ctrl</kbd> + <kbd>R</kbd>|<kbd>Command</kbd> + <kbd>R</kbd>|在当前文件进行文本替换|
|<kbd>Ctrl</kbd> + <kbd>Z</kbd>|<kbd>Command</kbd> + <kbd>Z</kbd>|撤销|
|<kbd>Ctrl</kbd> + <kbd>Y</kbd>|<kbd>Command</kbd> + <kbd>Delete</kbd>|删除光标所在行 或 删除选中的行|
|<kbd>Ctrl</kbd> + <kbd>D</kbd>|<kbd>Command</kbd> + <kbd>D</kbd>|复制光标所在行 或 复制选择内容，并把复制内容插入光标位置下面|
|<kbd>Ctrl</kbd> + <kbd>W</kbd>|<kbd>Option</kbd> + <kbd>方向键上</kbd>|递进式选择代码块。可选中光标所在的单词或段落，连续按会在原有选中的基础上再扩展选中范围|
|<kbd>Ctrl</kbd> + <kbd>E</kbd>|<kbd>Command</kbd> + <kbd>E</kbd>|显示最近打开的文件记录列表|
|<kbd>Ctrl</kbd> + <kbd>N</kbd>|<kbd>Command</kbd> + <kbd>O</kbd>|根据输入的 **类名** 查找类文件|
|<kbd>Ctrl</kbd> + <kbd>J</kbd>|<kbd>Command</kbd> + <kbd>J</kbd>|插入自定义动态代码模板|
|<kbd>Ctrl</kbd> + <kbd>P</kbd>|<kbd>Command</kbd> + <kbd>P</kbd>|方法参数提示显示|
|<kbd>Ctrl</kbd> + <kbd>U</kbd>|<kbd>Command</kbd> + <kbd>U</kbd>|前往当前光标所在的方法的父类的方法 / 接口定义|
|<kbd>Ctrl</kbd> + <kbd>B</kbd>|<kbd>Command</kbd> + <kbd>B</kbd>|进入光标所在的方法/变量的接口或是定义处，等效于 `Ctrl + 左键单击` |
|<kbd>Ctrl</kbd> + <kbd>/</kbd>|<kbd>Command</kbd> + <kbd>/</kbd>|注释光标所在行代码，会根据当前不同文件类型使用不同的注释符号|
|<kbd>Ctrl</kbd> + <kbd>F1</kbd>|<kbd>Command</kbd> + <kbd>F1</kbd>|在光标所在的错误代码处显示错误信息|
|<kbd>Ctrl</kbd> + <kbd>F11</kbd>|<kbd>Option</kbd> + <kbd>F3</kbd>|选中文件 / 文件夹，使用助记符设定 / 取消书签|
|<kbd>Ctrl</kbd> + <kbd>F12</kbd>|<kbd>Command</kbd> + <kbd>F12</kbd>|弹出当前文件结构层，可以在弹出的层上直接输入，进行筛选|
|<kbd>Ctrl</kbd> + <kbd>Space</kbd>|<kbd>Control</kbd> + <kbd>Space</kbd>|基础代码补全，默认在 Windows 系统上被输入法占用，需要进行修改，建议修改为 `Ctrl + 逗号`|
|<kbd>Ctrl</kbd> + <kbd>Delete</kbd>|<kbd>Option</kbd> + <kbd>Fn</kbd>+ Delete|删除光标后面的单词或是中文句|
|<kbd>Ctrl</kbd> + <kbd>BackSpace</kbd>|<kbd>Option</kbd> + <kbd>Delete</kbd>|删除光标前面的单词或是中文句|
|<kbd>Ctrl</kbd> + <kbd>1,2,3...9</kbd>|<kbd>Control</kbd> + <kbd>1,2,3...9</kbd>|定位到对应数值的书签位置|
|<kbd>Ctrl</kbd> + <kbd>加号</kbd>|<kbd>Command</kbd> + <kbd>加号</kbd>|展开代码|
|<kbd>Ctrl</kbd> + <kbd>减号</kbd>|<kbd>Command</kbd> + <kbd>减号</kbd>|折叠代码|
|<kbd>Ctrl</kbd> + <kbd>左键单击</kbd>|<kbd>Control</kbd> + <kbd>左键单击</kbd>|在打开的文件标题上，弹出该文件路径|
|<kbd>Ctrl</kbd> + <kbd>左方向键</kbd>|<kbd>Option</kbd> + <kbd>左方向键</kbd>|光标跳转到当前单词 / 中文句的左侧开头位置|
|<kbd>Ctrl</kbd> + <kbd>右方向键</kbd>|<kbd>Option</kbd> + <kbd>右方向键</kbd>|光标跳转到当前单词 / 中文句的右侧开头位置|
|<kbd>Ctrl</kbd> + <kbd>前方向键</kbd>|预设中没有该快捷键|等效于鼠标滚轮向前效果|
|<kbd>Ctrl</kbd> + <kbd>后方向键</kbd>|预设中没有该快捷键|等效于鼠标滚轮向后效果|

##### 1.1.2.1.3. Alt

|Win 快捷键|Mac 快捷键|介绍|
|:---------|:---------|:---------|
|<kbd>Alt</kbd> + <kbd>\`</kbd>|<kbd>Control</kbd> + <kbd>V</kbd>|显示版本控制常用操作菜单弹出层|
|<kbd>Alt</kbd> + <kbd>F1</kbd>|<kbd>Option</kbd> + <kbd>F1</kbd>|显示当前文件选择目标弹出层，弹出层中有很多目标可以进行选择|
|<kbd>Alt</kbd> + <kbd>F7</kbd>|<kbd>Option</kbd> + <kbd>F7</kbd>|查询所选对象/变量被引用|
|<kbd>Alt</kbd> + <kbd>Enter</kbd>|<kbd>Option</kbd> + <kbd>Enter</kbd>|IntelliJ IDEA 根据光标所在问题，提供快速修复选择，光标放在的位置不同提示的结果也不同|
|<kbd>Alt</kbd> + <kbd>Insert</kbd>|<kbd>Command</kbd> + <kbd>N</kbd>|代码自动生成，如生成对象的 set / get 方法，构造函数，toString() 等|
|<kbd>Alt</kbd> + <kbd>左方向键</kbd>|<kbd>Control</kbd> + <kbd>左方向键</kbd>|切换当前已打开的窗口中的子视图，比如Debug窗口中有Output、Debugger等子视图，用此快捷键就可以在子视图中切换|
|<kbd>Alt</kbd> + <kbd>右方向键</kbd>|<kbd>Control</kbd> + <kbd>右方向键</kbd>|切换当前已打开的窗口中的子视图，比如Debug窗口中有Output、Debugger等子视图，用此快捷键就可以在子视图中切换|
|<kbd>Alt</kbd> + <kbd>前方向键</kbd>|<kbd>Control</kbd> + <kbd>前方向键</kbd>|当前光标跳转到当前文件的前一个方法名位置|
|<kbd>Alt</kbd> + <kbd>后方向键</kbd>|<kbd>Control</kbd> + <kbd>后方向键</kbd>|当前光标跳转到当前文件的后一个方法名位置|
|<kbd>Alt</kbd> + <kbd>1,2,3...9</kbd>|<kbd>Command</kbd> + <kbd>1,2,3...9</kbd>|显示对应数值的选项卡，其中 1 是 Project 用得最多|

##### 1.1.2.1.4. Shift

|Win 快捷键|Mac 快捷键|介绍|
|:---------|:---------|:---------|
|<kbd>Shift</kbd> + <kbd>F11</kbd>|<kbd>Command + F3</kbd>|弹出书签显示层|
|<kbd>Shift</kbd> + <kbd>Tab</kbd>|<kbd>Shift + Tab</kbd>|取消缩进|
|<kbd>Shift</kbd> + <kbd>Enter</kbd>|<kbd>Shift + Enter</kbd>|开始新一行。光标所在行下空出一行，光标定位到新行位置|
|<kbd>Shift</kbd> + <kbd>左键单击</kbd>|<kbd>Shift + 左键单击</kbd>|在打开的文件名上按此快捷键，可以关闭当前打开文件|

##### 1.1.2.1.5. Ctrl + Alt

|Win 快捷键|Mac 快捷键|介绍|
|:---------|:---------|:---------|
|<kbd>Ctrl</kbd> + <kbd>Alt</kbd> + <kbd>L</kbd>|<kbd>Command</kbd> + <kbd>Option</kbd> + <kbd>L</kbd>|格式化代码，可以对当前文件和整个包目录使用|
|<kbd>Ctrl</kbd> + <kbd>Alt</kbd> + <kbd>O</kbd>|<kbd>Control</kbd> + <kbd>Option</kbd> + <kbd>O</kbd>|优化导入的类，可以对当前文件和整个包目录使用|
|<kbd>Ctrl</kbd> + <kbd>Alt</kbd> + <kbd>T</kbd>|<kbd>Command</kbd> + <kbd>Option</kbd> + <kbd>T</kbd>|对选中的代码弹出环绕选项弹出层|
|<kbd>Ctrl</kbd> + <kbd>Alt</kbd> + <kbd>S</kbd>|<kbd>Command</kbd> + <kbd>逗号</kbd>|打开 IntelliJ IDEA 系统设置|
|<kbd>Ctrl</kbd> + <kbd>Alt</kbd> + <kbd>Enter</kbd>|<kbd>Command</kbd> + <kbd>Option</kbd> + <kbd>Enter</kbd>|光标所在行上空出一行，光标定位到新行|
|<kbd>Ctrl</kbd> + <kbd>Alt</kbd> + <kbd>左方向键</kbd>|<kbd>Command</kbd> + <kbd>Option</kbd> + <kbd>左方向键</kbd>|退回到上一个操作的地方|
|<kbd>Ctrl</kbd> + <kbd>Alt</kbd> + <kbd>右方向键</kbd>|<kbd>Command</kbd> + <kbd>Option</kbd> + <kbd>右方向键</kbd>|前进到上一个操作的地方|

##### 1.1.2.1.6. Ctrl + Shift

|Win 快捷键|Mac 快捷键|介绍|
|:---------|:---------|:---------|
|<kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>F</kbd>|<kbd>Command</kbd> + <kbd>Shift</kbd> + <kbd>F</kbd>|根据输入内容查找整个项目 或 指定目录内文件|
|<kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>R</kbd>|<kbd>Command</kbd> + <kbd>Shift</kbd> + <kbd>R</kbd>|根据输入内容替换对应内容，范围为整个项目 或 指定目录内文件|
|<kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>J</kbd>|<kbd>Control</kbd> + <kbd>Shift</kbd> + <kbd>J</kbd>|自动将下一行合并到当前行末尾|
|<kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>Z</kbd>|<kbd>Command</kbd> + <kbd>Shift</kbd> + <kbd>Z</kbd>|取消撤销|
|<kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>W</kbd>|<kbd>Option</kbd> + <kbd>方向键下</kbd>|递进式取消选择代码块。可选中光标所在的单词或段落，连续按会在原有选中的基础上再扩展取消选中范围|
|<kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>N</kbd>|<kbd>Command</kbd> + <kbd>Shift</kbd> + <kbd>O</kbd>|通过文件名定位 / 打开文件 / 目录，打开目录需要在输入的内容后面多加一个正斜杠|
|<kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>U</kbd>|<kbd>Command</kbd> + <kbd>Shift</kbd> + <kbd>U</kbd>|对选中的代码进行大 / 小写轮流转换|
|<kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>T</kbd>|<kbd>Command</kbd> + <kbd>Shift</kbd> + <kbd>T</kbd>|对当前类生成单元测试类，如果已经存在的单元测试类则可以进行选择|
|<kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>C</kbd>|<kbd>Command</kbd> + <kbd>Shift</kbd> + <kbd>C</kbd>|复制当前文件磁盘路径到剪贴板|
|<kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>B</kbd>|<kbd>Control</kbd> + <kbd>Shift</kbd> + <kbd>B</kbd>|跳转到类型声明处|
|<kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>/</kbd>|<kbd>Command</kbd> + <kbd>Option</kbd> + <kbd>/</kbd>|代码块注释|
|<kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>\[</kbd>|<kbd>Command</kbd> + <kbd>Shift</kbd> + <kbd>\[</kbd>|选中从光标所在位置到它的顶部中括号位置|
|<kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>\]</kbd>|<kbd>Command</kbd> + <kbd>Shift</kbd> + <kbd>\]</kbd>|选中从光标所在位置到它的底部中括号位置|
|<kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>加号</kbd>|<kbd>Command</kbd> + <kbd>Shift</kbd> + <kbd>加号</kbd>|展开所有代码|
|<kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>减号</kbd>|<kbd>Command</kbd> + <kbd>Shift</kbd> + <kbd>减号</kbd>|折叠所有代码|
|<kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>F7</kbd>|<kbd>Command</kbd> + <kbd>Shift</kbd> + <kbd>F7</kbd>|高亮显示所有该选中文本，按Esc高亮消失|
|<kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>F12</kbd>|<kbd>Command</kbd> + <kbd>Shift</kbd> + <kbd>F12</kbd>|编辑器最大化|
|<kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>Enter</kbd>|<kbd>Command</kbd> + <kbd>Shift</kbd> + <kbd>Enter</kbd>|自动结束代码，行末自动添加分号|
|<kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>Backspace</kbd>|<kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>Backspace</kbd>|退回到上次修改的地方|
|<kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>1,2,3...9</kbd>|<kbd>Control</kbd> + <kbd>Shift</kbd> + <kbd>1,2,3...9</kbd>|快速添加指定数值的书签|
|<kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>左键单击</kbd>|<kbd>Command</kbd> + <kbd>Shift</kbd> + <kbd>左键单击</kbd>|把光标放在某个类变量上，按此快捷键可以直接定位到该类中|
|<kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>左方向键</kbd>|<kbd>Option</kbd> + <kbd>Shift</kbd> + <kbd>左方向键</kbd>|在代码文件上，光标跳转到当前单词 / 中文句的左侧开头位置，同时选中该单词 / 中文句|
|<kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>右方向键</kbd>|<kbd>Option</kbd> + <kbd>Shift</kbd> + <kbd>右方向键</kbd>|在代码文件上，光标跳转到当前单词 / 中文句的右侧开头位置，同时选中该单词 / 中文句|
|<kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>前方向键</kbd>|<kbd>Command</kbd> + <kbd>Shift</kbd> + <kbd>前方向键</kbd>|光标放在方法名上，将方法移动到上一个方法前面，调整方法排序|
|<kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>后方向键</kbd>|<kbd>Command</kbd> + <kbd>Shift</kbd> + <kbd>后方向键</kbd>|光标放在方法名上，将方法移动到下一个方法前面，调整方法排序|

##### 1.1.2.1.7. Alt + Shift

|Win 快捷键|Mac 快捷键|介绍|
|:---------|:---------|:---------|
|<kbd>Alt</kbd> + <kbd>Shift</kbd> + <kbd>N</kbd>|<kbd>Option</kbd> + <kbd>Shift</kbd> + <kbd>B</kbd>|选择 / 添加 task|
|<kbd>Alt</kbd> + <kbd>Shift</kbd> + <kbd>左键双击</kbd>|<kbd>Option</kbd> + <kbd>Shift</kbd> + <kbd>左键双击</kbd>|选择被双击的单词 / 中文句，按住不放，可以同时选择其他单词 / 中文句|
|<kbd>Alt</kbd> + <kbd>Shift</kbd> + <kbd>前方向键</kbd>|<kbd>Option</kbd> + <kbd>Shift</kbd> + <kbd>前方向键</kbd>|移动光标所在行向上移动|
|<kbd>Alt</kbd> + <kbd>Shift</kbd> + <kbd>后方向键</kbd>|<kbd>Option</kbd> + <kbd>Shift</kbd> + <kbd>后方向键</kbd>|移动光标所在行向下移动|

##### 1.1.2.1.8. Ctrl + Shift + Alt

|Win 快捷键|Mac 快捷键|介绍|
|:---------|:---------|:---------|
|<kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>Alt</kbd> + <kbd>V</kbd>|<kbd>Command</kbd> + <kbd>Shift</kbd> + <kbd>Option</kbd> + <kbd>V</kbd>|无格式黏贴|
|<kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>Alt</kbd> + <kbd>S</kbd>|<kbd>Command</kbd> + <kbd>;</kbd>|打开当前项目设置|

##### 1.1.2.1.9. 其他

|Win 快捷键|Mac 快捷键|介绍|
|:---------|:---------|:---------|
|<kbd>F2</kbd>|<kbd>F2</kbd>|跳转到下一个高亮错误 或 警告位置|
|<kbd>F4</kbd>|<kbd>F4</kbd>|编辑源|
|<kbd>F11</kbd>|<kbd>F3</kbd>|添加书签|
|<kbd>F12</kbd>|<kbd>F12</kbd>|回到前一个工具窗口|
|<kbd>Tab</kbd>|<kbd>Tab</kbd>|缩进|
|<kbd>ESC</kbd>|<kbd>ESC</kbd>|从工具窗口进入代码文件窗口|


### 1.1.3. postfix（系统默认）

```txt
# 定义非表达式
b!——> !b
# 定义方法参数
o.arg——>(o)
# 断言
value.assert——>assert value
# 强转类型
o.cast——>((强转类型)o)
# if非判断
b.else ——> if (!b) {}
# if判断
b.if ——> if (b) {}
# 抽取变量
b.feild——> 
private boolean b; 
    void m(boolean b) {
        this.b = b;
}
# 根据数组值生成遍历foreach方法
int[] values = {1, 2, 3};
values.for
或者
values.iter
——>
  int[] values = {1, 2, 3};
    for (int value : values) {
    }
# 根据for循环遍历值生成遍历方法i++
 int foo = 100;
 foo.fori ——>
   int foo = 100;
    for (int i = 0; i < foo; i++) {
    }
# 根据for循环遍历值生成遍历方法i--
 int foo = 100;
 foo.forr ——>
  int foo = 100;
    for (int i = foo; i > 0; i--) {
    }
# 类型判断生成三目表达式（强制转换）
o.inst——> o instanceof  ? (() o) : null;
o.instanceof——> o instanceof  ? (() o) : null;
# 生成lamba表达式
foo().lambda——>() -> foo()
# if判断对象不可以为空
o.nn或o.notnull——> if (o != null){ }
# if判断为空
o.null——>if (o = null){ }
# boolean 非
 m(b.not)——> m(!b)
# 同步代码块
o.synchronized——>synchronized (o) { }
# 抛出异常对象
 new RuntimeException("error").throw——>throw new RuntimeException("error");
# 生成异常try获取块
m().try——>
try {
    m();
  } catch(CheckedException e) {
    e.printStackTrace();
  }
# 创建异常，流自动释放（支持jdk7）
getStream().twr——>
 try (AutoCloseable stream = getStream()) {
    } catch (Exception e) {
    }
# 类型判断
o instanceof String.var——>boolean foo = o instanceof String;
# 生成while
x.while——>while (x) {}
```

### 1.1.4. live template
> idea 模板，快捷生成代码，常用技巧
1. 添加mavn依赖

```xml
dep——>
<dependency>
   <groupId>$GROUP$</groupId>
   <artifactId>$ARTIFACT$</artifactId>
   <version>$VERSION$</version>
</dependency>
pl——>
<plugin>
   <groupId>$GROUP$</groupId>
   <artifactId>$ARTIFACT$</artifactId>
   <version>$VERSION$</version>
</plugin>
repo——>
<repository>
  <id>$ID$</id>
  <name>$NAME$</name>
  <url>$URL$</url>
</repository>
```
2. xml中特殊字符转译
```xml
CD——>
<![CDATA[
$SELECTION$
]]>
```
3.创建sql
```txt
col：
$col$ $type$ $null$$END$
ins:
insert into $table$ ($columns$) values ($END$);
sel:
select * from $table$$END$;
selc:
select count(*) from $table$ $alias$ where $alias$.$END$;
selw:
select * from $table$ $alias$ where $alias$.$END$;
tab:
create table $table$ (
  $col$ $type$ $null$$END$
);
upd:
update $table_name$ set $col$ = $value$ where $END$;
```

4.其他
```txt
prsf:private static final
psf:public static final 
psfi:public static final int 
psfs:public static final String 
St:String 
thr:throw new 
serr:System.err.println($END$);
souf:System.out.printf("$END$");
sout:System.out.println($END$);
soutm:System.out.println("$CLASS_NAME$.$METHOD_NAME$");
soutp:System.out.println($FORMAT$);
soutv:System.out.println("$EXPR_COPY$ = " + $EXPR$);
```

```txt
geti:
public static $CLASS_NAME$ getInstance() {
  return $VALUE$;
}
ifn:
if ($VAR$ == null) {
$END$
}
inn:
if ($VAR$ != null) {
$END$
}
inst:
if ($EXPR$ instanceof $TYPE$) {
  $TYPE$ $VAR1$ = ($TYPE$)$EXPR$;
  $END$
}
lazy:
if ($VAR$ == null) {
  $VAR$ = new $TYPE$($END$);
}
mn：
$VAR$ = Math.min($VAR$, $END$);
mx：
$VAR$ = Math.max($VAR$, $END$);
psvm：
public static void main(String[] args){
  $END$
}

lst:
$ARRAY$[$ARRAY$.length - 1]
```

### 1.1.5. live template 与postfix 的区别
> Live Templates 可以自定义，而 Postfix Completion 不可以。 同时，有些操作二者都提供了模板， 据说Postfix Templates 较 Live Templates 能快 0.01 秒


### 1.1.6. 文档注释预定义变量
```
${PACKAGE_NAME} - the name of the target package where the new class or interface will be created.
${PROJECT_NAME} - the name of the current project.
${FILE_NAME} - the name of the PHP file that will be created.
${NAME} - the name of the new file which you specify in the New File dialog box during the file creation.
${USER} - the login name of the current user.
${DATE} - the current system date.
${TIME} - the current system time.
${YEAR} - the current year.
${MONTH} - the current month.
${DAY} - the current day of the month.
${HOUR} - the current hour.
${MINUTE} - the current minute.
${PRODUCT_NAME} - the name of the IDE in which the file will be created.
${MONTH_NAME_SHORT} - the first 3 letters of the month name. Example: Jan, Feb, etc.
${MONTH_NAME_FULL} - full name of a month. Example: January, February, etc.
```

### 1.1.7. 版本控制

![IDEA-Version-Control](https://gitee.com/uRick/oss/raw/master/blog/IDEA-Version-Control.png)
1. 很多人认为 IntelliJ IDEA 自带了 SVN 或是 Git 等版本控制工具，认为只要安装了 IntelliJ IDEA 就可以完全使用版本控制应有的功能。这完全是一种错误的解读， IntelliJ IDEA 是自带对这些版本控制工具的插件支持，但是该装什么版本控制客户端还是要照样装的。
2. IntelliJ IDEA 对版本控制的支持是以插件化的方式来实现的。旗舰版默认支持目前主流的版本控制软件： CVS、 Subversion（SVN）、 Git、 Mercurial、Perforce、TFS。又因为目前太多人使用 Github 进行协同或是项目版本管理，所以 IntelliJ IDEA 同时自带了 Github 插件，方便 Checkout 和管理你的Github 项目。
3. 在实际开发中，发现在 IDEA 中使用 SVN 的经历不算愉快，经常会遇到很多问题，比如紧急情况下 IDEA 无法更新、提交等。所以这里，谈下在 IDEA中使用 Git。

### 1.1.10. 必备插件
1. IdeaVim
> 通过键盘编辑，但是跟IDEA兼容不是很好.
> [https://plugins.jetbrains.com/plugin/164-ideavim](https://plugins.jetbrains.com/plugin/164-ideavim)
2. AceJump
> 脱离鼠标，快速跳转
> [https://plugins.jetbrains.com/plugin/7086-acejump](https://plugins.jetbrains.com/plugin/7086-acejump)
3. Lombok
> Lombok插件，提供getter/setter支持
> [https://plugins.jetbrains.com/plugin/6317-lombok-plugin](https://plugins.jetbrains.com/plugin/6317-lombok-plugin)
4. MybatisX 源码
> Mybatis plus 支持插件，当然其他mybatis环境也可以使用很方便
> [https://mp.baomidou.com/#/mybatisx-idea-plugin](https://mp.baomidou.com/#/mybatisx-idea-plugin)
> 源码 [https://gitee.com/baomidou/MybatisX](https://gitee.com/baomidou/MybatisX)
5. FindBugs 
> bug检查工具
> [https://plugins.jetbrains.com/plugin/3847-findbugs-idea](https://plugins.jetbrains.com/plugin/3847-findbugs-idea)
6. Key Promoter X
> 一个快捷键提示工具，它能够很好的辅助工作提高开发效率
> [https://plugins.jetbrains.com/plugin/index?xmlId=Key%20Promoter%20X](https://plugins.jetbrains.com/plugin/index?xmlId=Key%20Promoter%20X)
7. mutil-markdown
> 多功能markdown管理工具，虽然是收费但是可以破解，破解详见[https://www.jianshu.com/p/a0550f81cbd1](https://www.jianshu.com/p/a0550f81cbd1)

- 修改LisenceAgent.java源文件重新编译替换集合，修改内容如下：

```java
getLicenseExpires()// 删除方法体，直接返回过期时间字符串
getLicenseCode() // 设置最后一样return 返回true
isValidLicense() // 删除方法体，设置 return true;
isValidActivation() // 删除方法体,设置 return true;
getLicenseType() // 删除方法体，设置return "License" 或 return "license";
getLicenseExpiringIn() // 删除方法体，设置 return 36000;(单位是天)
isActivationExpired() // 删除方法体,设置return false;
getActivatedOn// 激活日期，设置  return "你的激活日期";
```

*注意：* 编译时注意包名要一样

## 1.2. IDEA 中SpringBoot项目无法开启Dashboard问题
## 1.3. 配置启动器
1. 首先找到项目中.idea文件下的workspace.xml开打
2. 接下来找到<component name="RunDashboard">

```xml
  <component name="RunDashboard">
    <option name="ruleStates">
      <list>
        <RuleState>
          <option name="name" value="ConfigurationTypeDashboardGroupingRule" />
        </RuleState>
        <RuleState>
          <option name="name" value="StatusDashboardGroupingRule" />
        </RuleState>
      </list>
    </option>
  </component>
```

3. 加入如下代码：

```xml
 <component name="RunDashboard">
 <!--配置RunDashboard 启动器-->
    <option name="configurationTypes">
      <set>
        <option value="SpringBootApplicationConfigurationType" />
      </set>
    </option>
    <option name="ruleStates">
      <list>
        <RuleState>
          <option name="name" value="ConfigurationTypeDashboardGroupingRule" />
        </RuleState>
        <RuleState>
          <option name="name" value="StatusDashboardGroupingRule" />
        </RuleState>
      </list>
    </option>
  </component>
```


## 1.4. IDEA在Win10 情况下外接高分屏幕时，当扩展屏直接拔出，没有将IDEA窗口拽会主屏幕，则无法打开解决

1. 项目下workspace.xml 找到ProjectFrameBounds 并删除 \<option name="x" value="173" /> 标签即可
