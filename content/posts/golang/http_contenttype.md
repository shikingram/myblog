---
title: "ChatGPT助我解决http接口返回zip问题"   
author: "Kingram"  
date: 2022-12-14   
lastmod: 2022-12-14

tags: [  
    "golang",
    "http"
]
---

## 背景
需求要求接口返回一个zip压缩包，且接口内部不要生成zip包，也就是在内存流中处理完成，不要写到文件中

## chatGPT怎么说 
通常解决方法都是使用application/octet-stream 把文件流以二进制的形式返回，但是这种返回文件名不可定义

问问看chatGPT怎么说

**K**: http application/octet-stream 怎么使用

**C**: HTTP application/octet-stream是一种通用的二进制数据流格式。它可以用来表示任何类型的文件，包括视频、音频、文档、程序等。使用这种格式来传输文件需要使用特定的软件，例如一款下载工具或浏览器插件。

---

**K**: http 返回gzip 使用什么Content-Type

**C**: 如果HTTP响应使用gzip压缩，则通常使用Content-Type头部指定压缩前的原始数据类型。例如，如果压缩前的数据是HTML文档，则Content-Type可能被设置为：
```
Content-Type: text/html
```

如果压缩前的数据是图像，则Content-Type可能被设置为：
```
Content-Type: image/jpeg
```
总之，Content-Type用于指定压缩前数据的类型，以便接收方程序能够正确地解析和处理数据。

---

**K**: 那如果是文件夹呢

**C**: HTTP协议本身不支持传输文件夹。如果需要在网络上传输一个文件夹，通常需要将其打包成一个压缩文件，例如zip、tar或gz文件，然后使用HTTP进行传输。在这种情况下，Content-Type应设置为指定压缩文件的类型，例如：
```
Content-Type: application/zip
```
另外，还需要在HTTP响应头中设置Content-Disposition头来指定文件名，以便浏览器能够自动将文件下载到本地。例如：
```
Content-Disposition: attachment; filename="folder.zip"
```
这样，接收方就可以使用软件解压缩文件来访问文件夹中的内容。

## example 

so 最终解决方案就是把文件压缩到response流中，最后以content-type 为zip的形式返回，example 如下：

``` golang
package util

import (
	"archive/zip"
	"io"
	"os"
	"path/filepath"
)

func Zip(src string, w io.Writer, includeSrc bool) error {
	archive := zip.NewWriter(w)
	defer archive.Close()

	filepath.Walk(src, func(path string, info os.FileInfo, err error) error {
		if err != nil {
			return err
		}
		var name string
		if includeSrc {
			name, err = filepath.Rel(filepath.Dir(src), path)
			if err != nil {
				return err
			}
		} else {
			if path == src {
				return nil
			}
			name, err = filepath.Rel(src, path)
			if err != nil {
				return err
			}
		}

		name = filepath.ToSlash(name)
		if info.IsDir() {
			name += "/"
		}

		writer, err := archive.Create(name)
		if err != nil {
			return err
		}

		if !info.IsDir() {
			file, err := os.Open(path)
			if err != nil {
				return err
			}
			defer file.Close()
			_, err = io.Copy(writer, file)
		}
		return err
	})

	return nil
}

```

http中函数这么写

```golang
func Zip(context *gin.Context) {
    var b bytes.Buffer
	w := gzip.NewWriter(&b)
    util.Zip("xxx",w,true)
    _ = w.Flush()
	defer w.Close()

    filename := time.Now().Format("20221214140150") + ".zip"
    context.Writer.Header().Set("Content-Encoding", "gzip")
	context.Writer.Header().Set("Content-Type", "application/zip")
	context.Writer.Header().Set("Content-Disposition", "attachment; filename="+filename)
	context.Data(http.StatusOK, "gzip", b.Bytes())
}
```

这样就完成了内存中的压缩操作，而且给http返回的文件也可自定义为时间戳，chatGPT真香。