# 解决鼠标中建在putty中不起作用的问题

Logitech MX Master 没有问题，但310就不行,查阅[ubuntu操作手册](https://wiki.ubuntu.com/X/Config/Input#Example:%20Disabling%20middle-mouse%20button%20paste%20on%20a%20scrollwheel%20mouse)

取消鼠标中键粘贴功能
点中键的习惯一时还改不了，用代码或文本编辑器的时候，一不小心上面就多了不少粘贴的代码文字，这就比较麻烦了，结果运行不了。费了不少时间才找到替代的解决办法 (来源: ubuntu wiki),可以把点击鼠标中键替换为左键或者右键，上下滚动鼠标的功能保留（找到一些方法是屏蔽鼠标中键，上下滚动也不能用，那也麻烦），跟windows没有完全一致，不过比原来感觉好多了，这里把方法简化一下：      
第一步：查询鼠标设备  

```
$ xinput list | grep 'id='
```

得到类似结果    

```
"Virtual core pointer" id=0 [XPointer]
"Virtual core keyboard" id=1 [XKeyboard]
"AT Translated Set 2 keyboard" id=2 [XExtensionKeyboard]
"Macintosh mouse button emulation" id=3 [XExtensionPointer]
"PIXART USB OPTICAL MOUSE" id=4 [XExtensionPointer]
``

第二部：查询鼠标按键参数表（可以省去）

```
xinput get-button-map "Logitech M310/M310t"

xinput get-button-map "Logitech MX Master"

xinput get-button-map "PIXART USB OPTICAL MOUSE"

```

得到结果
1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 10 
其实就是左键1，中键2，右键3，其他键全部傻乎乎的试了一下，没什么用

第三部：替换中键点击功能   

$ xinput set-button-map "PIXART USB OPTICAL MOUSE" 1 1 3

把中键点击替换为左键相同功能，或者改成右键，用" 1 3 3"就可以，中间的鼠标名称记得用第一查询出来的替换掉
