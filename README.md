<h1 align = center>批量合成bilibili的M4S缓存文件为MP4格式</h1>
<p>
<font size = 4>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;众所周知，B站是一个学习网站，我常常在B站看一些教程视频，但是有时候没有网络就看不了了，于是我打算把教程都缓存下来看，不得不说，B站的缓存速度还是相当快的，缓存好以后，当我找到缓存目录的时候，发现缓存的文件音画是分离的，我尝试了用格式工厂的视频混流功能，虽然能够实现m4s文件合成为mp4文件，但是不能做到批量合成，需要一个个拖入音频和视频M4S文件，于是我就用python写了一个批量合成m4s文件的工具，通过循环调用ffmpeg中的命令合成MP4文件。</font>
</p>

<p>
首先需要安装ffmpeg<p>
    ffmpeg下载地址：<a href="https://ffmpeg.zeranoe.com/builds/">https://ffmpeg.zeranoe.com/builds</a></p>
解压好下载的压缩包后，再将bin目录加入Path环境变量中
按Win+R 运行 输入cmd 在弹出的框框中输入 ffmpeg ，如果没有出现既不是内部或外部命令之类的话就是安装成功了
</p>

<p>
先将手机上的缓存目录复制到电脑上
一般B站的缓存目录位于：/Android/data/tv.danmaku.bili/download
    <br><br>
再打开工具将缓存目录的路径复制到工具中敲回车，工具将自动扫描下载目录，并将生成的mp4文件放在工具的同级的output目录下。 
</p>

已编译的EXE可执行文件：[https://github.com/bloodarea/ConvertM4StoMP4/releases](https://github.com/bloodarea/ConvertM4StoMP4/releases)

展示：
<img src ="http://a1.qpic.cn/psc?/V11YWwIn2YELvg/m*TTJoI3x9iCCJx4ECa9pHQmTxlrUoE*rlXCgQo3c8y2x2NqRCDJrc.k.PHqJUQehyAhOcS72fSIFG3Ag.dOAQ!!/b&ek=1&kp=1&pt=0&bo=1AP8AdQD*AEDEDU!&tl=1&su=0199983535&tm=1590285600&sce=0-12-12&rf=2-9" />
<img src ="http://a1.qpic.cn/psc?/V11YWwIn2YELvg/m*TTJoI3x9iCCJx4ECa9pCMUnLFRn6dfvMUnDgzzMNYfleakt8.u9sooXf.He3OxfI1WWwRAg*zY4HalL5N*Hg!!/b&ek=1&kp=1&pt=0&bo=SANMAUgDTAEDEDU!&tl=1&su=0220054255&tm=1590285600&sce=0-12-12&rf=2-9" />

源码：
```python
import os
import json
import re
import time

# 读取json文件
def readJson(fileName):
    f = open(fileName, encoding='utf-8')
    setting = json.load(f)
    title = setting['page_data']['download_subtitle']
    return title

# 获取文件列表
def getFileList(file_dir):
    #定义三个列表
    Title = []
    VideoPath = []
    AudioPath = []
    #遍历文件目录
    for root, dirs, files in os.walk(file_dir):
        if ('entry.json' in files):
            Tname = str(root) + '\\entry.json'
            Tname = readJson(Tname)
            Title.append(Tname)
        if ('video.m4s' in files and 'audio.m4s' in files):
            Vpath = str(root) + '\\video.m4s'
            Apath = str(root) + '\\audio.m4s'
            VideoPath.append(Vpath)
            AudioPath.append(Apath)
        if('0.blv' in files):
            Title.pop()
    return [Title, VideoPath, AudioPath]

# 输出mp4文件
def getMP4(title, video_path, audio_path):
    # 生成输出目录
    if not os.path.exists("./output"):
        os.mkdir("./output")
    #循环生成MP4文件
    for i in title:
        #规范文件名
        cop = re.compile("[^\u4e00-\u9fa5^a-z^A-Z^0-9]") # 匹配不是中文、大小写、数字的其他字符
        reName = i
        reName = cop.sub('', reName)  # 将标题中匹配到的字符替换成空字符
        #开始生成MP4文件
        if not os.path.exists("./output/" + reName + ".mp4"):
            os.system(
                "ffmpeg -i " + video_path[title.index(i)] + " -i " + audio_path[title.index(i)] + " -codec copy ./output/" + reName + ".mp4")
            print("正在合成...")
            print("标题：" + i)
            print("视频源：" + video_path[title.index(i)])
            print("音频源：" + audio_path[title.index(i)])
            time.sleep(1)

print("欢迎使用批量合成M4S工具")
fileDir = str(input("请输入含M4S文件的目录:"))
f = getFileList(fileDir)
getMP4(f[0], f[1], f[2])
print("合成完毕")

```
