---
title: "循环调用接口下载文件并打包的脚本"   
author: "Kingram"  
date: 2022-12-16   
lastmod: 2022-12-16

tags: [  
    "python"
]
---
## 背景
循环调用http接口，每次返回一个zip压缩文件，下载到本地，然后打包这些压缩文件为一个包

## python版本
```python
import requests
import zipfile
import os


def download_and_zip_files(appid_list):
  zip_file = zipfile.ZipFile("compressed_files.zip", "w")
  
  for appid in appid_list:
    # http://192.168.49.221/api/v1/sts/3520a6a966e74648ad34eb22bd706e79/zip
    response = requests.get(f"http://192.168.49.221/api/v1/sts/{appid}/zip")
    zip_file.writestr(f"{appid}.zip", response.content)
  
  zip_file.close()

def read_appid_list_from_config_file():

    config_file_path = os.path.join(os.getcwd(), 'config.txt')
    with open(config_file_path, 'r', encoding='utf-8') as f:
        appid_list = []
        for line in f.readlines():
            appid_list.append(line.strip())
    return appid_list

appid_list = read_appid_list_from_config_file()
download_and_zip_files(appid_list)
```

## shell版本

```shell
#!/bin/bash

# Read the appid list from the config file.
appid_list=`cat config.txt`

# Loop through the list of appids and call the http interface for each one.
for appid in $appid_list
do
   curl --compressed -H "Accept-Encoding: application/zip"  "http://192.168.49.221/api/v1/sts/$appid/zip" -o "$appid.zip"
done

# Create the new package with all the zip files.
tar -zcvf package.tar.gz *.zip

# Remove the individual zip files.
rm *.zip
```

## config.txt
```
3520a6a966e74648ad34eb22bd706e79
05d1f9829f1e434386755c647427c684
```