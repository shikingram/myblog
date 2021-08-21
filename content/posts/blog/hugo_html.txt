{
  "title":"hugo渲染html标签",
  "tags":[
    "blog"
  ],
  "date":"2021-08-21",
  "lastmod":"2021-08-21",
  "draft":"false",
  "author":"kingram"
}

Hugo使用 Blackfriday渲染Markdown文件。从Hugo 0.60版本开始，默认的Markdown渲染器已更改为goldmark。在goldmark中，默认设置是不呈现原始HTML标签。

## 解决方案

在config.toml中添加如下配置
```
[blackfriday]
extensionsmask = ["noIntraEmphasis"]
fractions = false

[markup.goldmark.renderer]
unsafe = true
```

效果：

```
<font color=#800000>酒紅色</font>酒紅色
<font color=#FF00FF>粉紅色</font>粉紅色
<font color=#800080>紫色</font>紫色	
<font color=#808080>灰色</font>灰色
```
<font color=#800000>酒紅色</font>酒紅色
<font color=#FF00FF>粉紅色</font>粉紅色
<font color=#800080>紫色</font>紫色	
<font color=#808080>灰色</font>灰色