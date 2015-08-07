title: "解决Ant编译打包错误: 非法字符: '\ufeff'"
date: 2014-12-01 09:18:32
tags: [Android]
---
这个问题其实是UTF-8编码格式的Byte Order Mark问题。解决很简单，只要去掉BOM头就行了。

安装了NotePad++或者EditPlus的朋友可以直接另存一下UTF-8无BOM格式编码即可。

因为本人使用的是Mac，所以就直接在vim上操作了，主要命令是下面几个：

- 去掉BOM标记

```
:set nobomb
```

- 加上BOM标记

```
:set bomb
```

- 查询当前UTF-8编码的文件是否有BOM标记

```
:set bomb?
```

- 以16进制模式打开文件

```
:%!xxd 
"删除文件开头的EF  BB  BF
```

- 将以16进制格式打开的文件返回文本模式编辑

```
:%!xxd -r
``` 