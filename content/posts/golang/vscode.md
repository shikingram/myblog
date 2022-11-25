{
  "title":"从Goland转到Vscode的感受",
  "tags":[
    "development tools"
  ],
  "date":"2022-11-25",
  "lastmod":"2022-11-25",
  "draft":"false",
  "author":"kingram"
}

# 为什么从Goland转到Vscode
主要原因是电脑性能太差

mac是19年丐版，跑了goland和docker基本就满内存了，8g不能满足日常开销

![1](/img/mac/1.jpg)

goland跑起来占用内存太大了

另外一个原因就是他是收费的，虽然用了循环试用，但是这总让我很不爽

以前也试过vscode，但是那时候没折腾好

前段时间工作不忙正好再试试折腾一下，习惯之后，真香~

# vscode的优势
## 插件
最好玩的就是很多插件了，技术相关的和不相关的

什么韭菜盒子，实时更新股票基金虚拟币的行情啊

什么虎扑摸鱼助手

![2](/img/mac/2.jpg)

好家伙，边写代码边逛步行街

还能看文字直播

各种格式的文件美化格式化插件

比如以前json yaml文件都需要用网站工具格式化

使用插件后直接ctl+n创建文件粘贴

然后使用快捷键直接格式化

还有一些数据库插件之类很多好玩的插件待发掘

## 内存占用
vscode内存占用比goland小太多了

同时开启几个页面也不在话下

## 快捷键
所有goland的操作，在vscode都能找到替代

我自己也总结了一些，贴在这里

- 折叠所有函数  cmd+k cmd+0
- 展开所有函数 cmd+k cmd+j
- 切换主题  cmd+k +t
- 切换标签页  cmd + alt + 左右
- 前进返回  alt + 左右
- 搜索ctl +f
- 替换 ctl + h
- 整行移动 alt + 上下
- 翻页 alt+pgup/pgdn
- 页首页尾 alt+home/end
- 侧栏可见   ctl +b
- 打开新文件 ctl +n 
- 打开文件 ctl +o
- 打开上一个   ctl+shift+tab
- 打开下一个 ctl+tab
- 格式化json alt+shift+f

## 写其他语言
前段时间写exlier，使用goland装个插件就能支持

有时写js或者html代码，直接在vscode打开方便很多

切换vscode后，前期可能艰难几天，但是熟悉后的感受是可玩性比goland高很多