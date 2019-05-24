---
layout: post
title: 【笔记】tesseract
categories: 编程
tags: 后端 AI
author: adai
---

* content
{:toc}


# tesseract 普通训练
![流程图]({{site.url}}/assets/2019-05-24/tesseract_training.svg)

* **step 1 : 获取tif文件** 

方式一、图片转换成tif ,设置DPI

```shell
#调整图片dpi,生成tif图片
$convert '*.jpg' -density 300 image_%d.tif
```

方式二、jTessBoxEditor 合并成多个tif文件

训练用tif文件命名格式 :  **[lang].[fontname].exp[num].tif**
**lang :** 语言名称
**fontname :** 字体名称
**num :** 字体分组编号

* **step 2 : 生成box文件**

```shell

$tesseract chi_sim.Arial.exp1.tif chi_sim.Arial.exp1 makebox batch.nochop

```

* **step 3 : 矫正box文件数据** 
先用**step 4**验证一遍tif数据准确性

jTessBoxEditor 修改 **\*.tif** **\*.box**准确性 

* **step 4 : 生成tr文件** 

```shell

tesseract chi_sim.Arial.exp1.tif chi_sim.Arial.exp1 box.train

```

* **step 5 : 生成 unicharset文件**

```shell

## 生成uncharset文件
##Tesseract’s unicharset file contains information on each symbol (unichar) the Tesseract OCR engine is trained to recognize.Currently, generating the unicharset file is done in two steps using these commands: unicharset_extractor and set_unicharset_properties.NOTE: The unicharset file must be regenerated whenever inttemp, normproto and pffmtable are generated (i.e. they must all be recreated when the box file is changed) as they have to be in sync.

$unicharset_extractor chi_sim.Arial.exp1.box chi_sim.Arial.exp2.box chi_sim.Arial.exp3.box chi_sim.Arial.exp4.box

## 设置属性
$set_unicharset_properties -U input_unicharset -O output_unicharset --script_dir=training/langdata

```

* **step 6 : 定义 font_properties 文件**
fontname 必须与**step1** 命名的fontname一致

```shell
[fontname][italic][bold][fixed][serif][fraktur]
```

* **step 7 : 聚类**
三个步骤shapeclustering 、 mftraining 和 cntraining

```shell
    #gen shaptable
    $shapeclustering -F font_properties -U chi_sim.unicharset *.tr
    #gen inttemp
    $mftraining -F font_properties -U unicharset -O chi_sim.unicharset *.tr
    # ten normproto
    $cntraining *.tr
```

* **step 8: 合并文件**

生成的五个文件shapetable，normproto，inttemp，pffmtable，unicharset 用**\[lang\]**为前缀重命名

```shell
combine_tessdata chi_sim.
```
