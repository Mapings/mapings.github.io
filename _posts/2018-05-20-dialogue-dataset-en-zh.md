# 剧本网

中国剧本网( http://www.juben.cn/ )是一个剧本网站，其内部有大量对话内容，例如：http://www.juben.cn/jbk/ckjb.aspx?ProductID=137450&Type=3 是爱情公寓的一个剧本样本。

<p align="center">
  <img src="/images/juben_example.png" width="350"/>
</p>

通过爬虫与适当的正则表达式处理，可以得到一定量的对话数据(raw utf-8 string for source and target)，这部分的单轮对话数据存储在笔者本地的硬盘上。

遗憾的是，这段爬虫的代码已经丢失了，如果需要对爬虫及正则表达式处理过程进程改动，则需要进行做这部分工作 :)

# OpenSubtitles 数据

很多论文使用OpenSubtitles数据来训练对话模型，比如 [A Neural Conversational Model](https://arxiv.org/abs/1506.05869). 

为了方便大家使用OpenSubtitle的数据，本着开源共享的精神，[opus](http://opus.nlpl.eu/) —— "the open parallel corpus" —— 团队帮助大家从 opensubtitles.org 字幕库做了实时的爬虫和一些数据预处理。他们提供：

```
OpenSubtitles - the opensubtitles.org corpus (OpenSubtitles.tar.gz - 1.9 GB)
OpenSubtitles2011, OpenSubtitles2012, OpenSubtitles2013
OpenSubtitles2016 - snapshot from 2016
OpenSubtitles2018 - new complete version
```

我们可以在OPUS网站直接搜索得到相应数据的链接，比如最新的版本的[OpenSubtitles2018](http://opus.nlpl.eu/OpenSubtitles2018.php)。

在这个链接总，可以找到我们需要的中文和英文字幕数据分别为：

http://opus.nlpl.eu/download.php?f=OpenSubtitles2018/en.tar.gz , 25GB（解压之后更大）

http://opus.nlpl.eu/download.php?f=OpenSubtitles2018/zh_cn.tar.gz , 1GB+

下载后的数据格式为xml格式, 例如：
```xml
<s id="141">
    <w id="141.1">-</w>
    <w id="141.2">黛安娜</w>
    <w id="141.3">！</w>
    <time id="T115E" value="00:10:46,000" />
  </s>
  <s id="142">
    <time id="T116S" value="00:10:57,780" />
    <w id="142.1">你</w>
    <w id="142.2">有</w>
    <w id="142.3">什么</w>
    <w id="142.4">？</w>
    <time id="T116E" value="00:10:58,650" />
  </s>
  <s id="143">
    <time id="T117S" value="00:10:59,120" />
    <w id="143.1">-</w>
    <w id="143.2">不</w>
    <w id="143.3">，</w>
    <w id="143.4">妈妈</w>
    <w id="143.5">。</w>
  </s>
  <s id="144">
    <w id="144.1">我</w>
    <w id="144.2">没有</w>
    <w id="144.3">什么</w>
    <w id="144.4">。</w>
  </s>
  <s id="145">
    <w id="145.1">只是</w>
    <w id="145.2">...</w>
  </s>
  <s id="146">
    <w id="146.1">...</w>
  </s>
  <s id="147">
    <w id="147.1">-</w>
    <w id="147.2">泰利</w>
    <w id="147.3">在</w>
    <w id="147.4">干什么</w>
    <w id="147.5">。</w>
    <time id="T117E" value="00:11:01,950" />
  </s>
```

## 预处理

预处理可以参考：https://github.com/AlJohri/OpenSubtitles :)

处理目标：得到分好词的source-target pairs, source 和 target 分别在两个文件里。

处理后共得到中文 31,137,513 对数据，英文？对数据, 噪声非常大。

处理后类似：
```
- 黛安娜 ！
你 有 什么 ？
- 不 ， 妈妈 。
我 没有 什么 。
只是 ...
...
- 泰利 在 干什么 。
```

## 拆分训练集和测试集

|数据名称|句子pair数量|年份|
|:-----:|:-----:|:------:|
|zh_cn_train|28,023,762|全部|
|zh_cn_test|3,113,751|全部|
|en_small_train|3,045,832|2017|
|en_small_test|338,237|2017|

# Cornell Movie--Dialogs Corpus

来源：http://www.cs.cornell.edu/~cristian/Cornell_Movie-Dialogs_Corpus.html

这个数据集很小，只是几十万的量级（不到1个millon）, 比OpenSubtitle的几十个millon的量级相比小很多，好处是它的数据中有角色信息。anyway，拿到数据后我做了一点简单的预处理，生成了一些对话的pair。

```python
# encoding=utf-8

f = open("movie_lines.txt", 'r')
source = open("en.source", "w")
target = open("en.target", "w")

lines = f.readlines()
last_charactor = None
last_sentence = None
sources = []
targets = []

for line in lines:
    s = line.strip()
    charactor = s.split(" +++$+++ ")[-2]
    sentence = s.split(" +++$+++ ")[-1]
    if charactor != last_charactor and \
            last_charactor is not None and \
            last_sentence is not None:
        sources.append(last_sentence)
        targets.append(sentence)
    last_charactor = charactor
    last_sentence = sentence

assert len(sources) == len(targets)
dialogue = zip(sources, targets)
for d in dialogue:
    source.write(d[0] + "\n")
    target.write(d[1] + "\n")
```

最终生成279,976对句子（每一行是一句话，未分词）。称为：en.source 和 en.target

# Redit
TODO
