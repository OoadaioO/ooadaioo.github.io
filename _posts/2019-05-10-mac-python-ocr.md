---
layout: post
title: 【笔记】mac下使用python 进行ocr识别
categories: 编程
tags: 建站
author: adai
---

* content
{:toc}


#步骤
1. 安装tesseract库 brew install tesseract
2. pip安装python的tesseract接口 pip install pytesseract
3. 下载中文数据集 [tessdata github](https://github.com/tesseract-ocr/tessdata)  <font color='#ff0000'>chi_sim.traineddata</font>放置到tesseract安装目录下share/tessdata e.g:/usr/local/Cellar/tesseract/3.05.01/share/tessdata


#使用tesseract
```python
import pytesseract
from PIL import Image

# open image
image = Image.open('test.jpg')
code = pytesseract.image_to_string(image, lang='chi_sim')
print(code)
```
