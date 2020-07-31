---
layout: post
date: 2019-11-20 20:55:07
title: 利用python自动发布jekyll博客post
comments: true
categories: [python]
tags: [python, jekyll]
---

笔记文件夹和jekyll项目不在一起，所以通过python脚本
- 给md笔记添加frontmatter
- 复制图片到\{\{ site.url  \}\}/img/，并修改图片路径
- 输出“日期-title”格式的md文件到_post文件夹

<!-- more -->

## 使用方法

在自己的笔记文件夹打开命令行，运行命令

`python auto_post.py 利用python自动发布jekyll博客post.md  -o ~/Work/z-swei.github.io`

![]( {{ site.url }}/img/001.png)



## 代码`auto_post.py`

{% raw %}

```python
import os
import argparse
import time
import datetime
import shutil
import re

def timestamp2time(timestamp):
    time_struct = time.localtime(timestamp)
    return time.strftime("%Y-%m-%d-",time_struct)

def post(args):
    '''
    args:   filename: str
            -t --title: str
            -c --comments: bool    
    '''
    file_name, title, comments, out = args.file_name, args.title, args.comments, args.out
    # 读取文件 
    with open(args.file_name,encoding='utf-8') as f:
        contents = f.readlines()
        ctime = timestamp2time(os.path.getctime(file_name))
    
    # 编写新post文件内容
    new_contents = []
    # 添加Front Matter 
    new_contents.append('---\n')
    new_contents.append('layout: post\n')
    
    # 添加文章标题
    if title:
        new_contents.append('title: '+ title +'\n')
    else:
        new_contents.append('title: '+ file_name[:-3] +'\n')
        
    # 添加评论区
    if comments:
        new_contents.append('comments: true' +'\n')
    
    # 添加文章分类categories
    if(contents[0][:10]=='categories'):
        new_contents.append(contents[0])
        contents.pop(0)
    
    # 添加文章标签tags
    if(contents[0][:4]=='tags'):
        new_contents.append(contents[0])
        contents.pop(0)
        
    new_contents.append('---\n')
    
    # 整合内容
    new_contents += contents
    
    with open(os.path.join(os.path.join(out,'_posts'),ctime+file_name.replace(' ','-')),'w',encoding='utf-8') as f:
        for line in new_contents:
            f.write(line.replace("img/","{{ site.url }}/img/"))
            imgs = re.findall("\]\(.*img/.*\)", line)
            if(len(imgs)>0):
                for img in imgs:
                    shutil.copy(img[2:-1].strip(),os.path.join(out,'img/'))

if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("file_name")
    parser.add_argument("-t", "--title", help="input title of this post")
    parser.add_argument("-c", "--comments",  action="store_true", help="add comments:true")
    parser.add_argument("-o","--out", default="", help="output dir")
    args = parser.parse_args()
    post(args)
```{% endraw %}
