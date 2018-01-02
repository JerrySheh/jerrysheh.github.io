---
title: 使用Python遍历文件
categories: Python
tags: Python
abbrlink: c78ec954
date: 2017-10-29 12:22:01
---


下载了很多歌曲，有些是`.mp3`格式的，有些是`.flac`格式，还有的是`.wav`、`.ape`各种各样。我们知道，`.flac`和`.ape`是无损格式，所以我想保留这两种格式的文件，删掉`.wav`、`.mp3`格式的音乐。

可以用 Python 自动化处理。


<!-- more -->


---

# 代码

首先我的歌曲全都是放在`D:\CloudMusic`这个目录下，而这个目录下还有很多子目录，现在要做的就是把`D:\CloudMusic`和它的子目录下所有`.wav`、`.mp3`格式的音乐找出来。

```Python
import os

music_dir = "D:\\CloudMusic"

for root, dirs, files in os.walk(music_dir):
    for file in files:
        target = os.path.join(root, file)
        if file.endswith(".mp3") or file.endswith(".wav"):
            print(target)
```

先看`os.walk()`的用法：

`os.walk(top[, topdown=True[, onerror=None[, followlinks=False]]])`

`top` 参数让 `os.walk()` 根目录下每一个文件夹（包括根目录）产生一个3-元组 (dirpath, dirnames, filenames)【文件夹路径, 文件夹名字, 文件名】

因此，`for root, dirs, files in os.walk(music_dir):`中的`root`、`dirs`、`files`分别表示文件夹路径, 文件夹名字, 文件名。

---

```Python
>>>for root, dirs, files in os.walk(music_dir):
>>>    print(root)

```

输出的是`CloudMusic`目录下的所有子文件夹路径：
```
D:\CloudMusic
D:\CloudMusic\Cache
D:\CloudMusic\IU
D:\CloudMusic\IU\IU - Last.Fantasy
D:\CloudMusic\IU\IU - Real 2010 FLAC
D:\CloudMusic\IU\IU - 꽃갈피(花书签)
D:\CloudMusic\IU\IU chat-shire
D:\CloudMusic\IU\IU-Can You Hear Me
D:\CloudMusic\IU\Palette
D:\CloudMusic\MV
```

```Python
>>>for root, dirs, files in os.walk(music_dir):
>>>    print(dirs)

```

输出的是`CloudMusic`目录下的所有子文件夹名字(放在一个list里)：
```
['Cache', 'IU', 'MV']
[]
['IU - Last.Fantasy', 'IU - Real 2010 FLAC', 'IU - 꽃갈피(花书签)', 'IU chat-shire', 'IU-Can You Hear Me', 'Palette']
[]
[]
[]
[]
[]
[]
[]
```

第一个空list表示 Cache 里没有子文件夹了， 第二个空list表示 IU - Last.Fantasy 里没有子文件夹了，以此类推。


```Python
>>>for root, dirs, files in os.walk(music_dir):
>>>    print(files)

```


输出的是`CloudMusic`目录下(包括子文件夹)的所有文件名字(放在一个list里)：
```
['01 Welcome To New York.flac', '02 Blank Space.flac', '03 Style.flac', '04 - Mean.flac', '05 All You Had To Do Was Stay.flac', '06 Shake It Off.flac', '07 I Wish You Would.flac', '08 Bad Blood.flac', '09 Wildest Dreams.flac', '10 How You Get The Girl.flac', '11 This Love.flac', '12 I Know Places.flac', '13 Clean.flac', '14 Wonderland.flac', '15 You Are In Love.flac', '16 New Romantics.flac', '17 I Know Places - Voice Memos.flac', '18 I Wish You Would - Voice Memos.flac', '19 Blank Space - Voice Memos.flac', 'AlbumArtSmall.jpg', 'C Allstar - 天梯.flac', 'Chen、Punch - Everytime.ape', 'Davichi - 这份爱.ape', 'desktop.ini', 'Folder.jpg', 'G.E.M.邓紫棋 - 喜欢你.flac', 'Gummy - You Are My Everything(English Ver.).flac', 'Gummy - You Are My Everything.flac', 'K.Will - 说干什么呢.flac', 'LYn - With You.flac', 'M.C. The Max - 为你化成风.flac', 'Mad Clown、金娜英 - 再次见到你.flac', 'SG WANNABE - 让我们相爱.flac', '____.flac', '光良 - 第一次.flac', '其实我介意.flac', '孙燕姿 - 雨天.flac', '小幸运.flac', '曲婉婷 - Drenched.flac', '李克勤 - 友情岁月.flac', '林宥嘉-想自由.flac', '林宥嘉-浪费.flac', '林宥嘉-说谎.APE', '梁静茹 - 爱久见人心.flac', '荒木毬菜 - 小雨と君.wav', '许美静 - 倾城.wav', '许美静-遗憾.WAV', '逃跑计划 - 夜空中最亮的星.flac', '金俊秀 - How Can I Love You.flac', '陈奕迅 - 富士山下.flac', '陈奕迅 - 最佳损友.flac', '陈奕迅 - 陪你度过漫长岁月.flac']['IU - 至少有那天 【视频from：微博@德米安】.mp3', 'IU - 너의 의미(你的意义).mp4', '[onlyU字幕组][MV] IU아이유–Through the Night 夜信[1080P精效中字].mp4', '（中字720P）《至少有那天》饭拍：wj_淮.mp4']
['01. 비밀.flac', '02. 잠자는 숲 속의 왕자 (feat. 윤상).flac', '03. 별을 찾는 아이 (feat. 김광진).flac', '04. 너랑 나.flac', '05. 벽지무늬.flac', '06. 삼촌 (feat. 이적).flac', '07. 사랑니.flac', "08. Everything's Alright (feat. 김현철).flac", '09. Last Fantasy.flac', '10. Teacher (feat. Ra.D).flac', '11. 길 잃은 강아지.flac', '12. 4AM.flac', "13. 라망 (L'amant).flac"]
['01. 이게 아닌데.flac', '02. 느리게 하는 일.flac', '03. 좋은 날.flac', '04. 첫 이별 그날 밤.flac', '05. 혼자 있는 방.flac', '06. 미리 메리 크리스마스.flac', '07. 좋은 날 (Inst.).flac', 'cover.jpg']
['01. 나의 옛날이야기.flac', '02. 꽃（花）.flac', '03. 삐에로는 우릴 보고 웃지（小丑在对着我们微笑）.flac', '04. 사랑이 지나가면（当爱已逝去）.flac', '05. 너의 의미 (Feat. 김창완)（你的意义）.flac', '06. 여름밤의 꿈（仲夏夜之梦）.flac', '07. 꿍따리 샤바라 (Feat. 클론)（Kungtari Shabara）.flac', 'AlbumArtSmall.jpg', 'Folder.jpg']
['Red Queen.flac', 'Zezé.flac', '무릎(膝盖).flac', '새 신발(新鞋).flac', '스물셋(二十三).flac', '안경(眼镜).flac', '푸르던(曾经蔚蓝).flac']
['01 Beautiful Dancer.wav', '02 Truth.wav', '03 Fairytale.wav', '04 Voice-mail.wav', '05 New World.wav', '06 The Age Of The Cathedrals.wav', 'IU Can You Hear Me cover pics.jpg']
['01 這一刻.flac', '02 Palette.flac', '03 這種結局.flac', '04 爱情很好.flac', '05 Jam Jam.flac', '06 Black Out.flac', '07 終止符.flac', '08 夜信.flac', '09 愛情就那樣.flac', '10 致名字.flac']
[]
```

用`os.remove()`删除

```Python
import os

music_dir = "D:\\CloudMusic"

for root, dirs, files in os.walk(music_dir):
    for file in files:
        target = os.path.join(root, file)
        if file.endswith(".mp3") or file.endswith(".wav"):
            print("deleting " + target)
            os.remove(target)
```

`target`这个变量主要是把文件夹路径和文件名拼接起来，构成一个完整的文件路径，比如：
```
D:\CloudMusic\IU\IU-Can You Hear Me\06 The Age Of The Cathedrals.wav
```

后面的就是判断后缀名是否为 `.mp3` 和 `.wav` ， 是的话就删除之。


---

# 知识点

写这篇记录的目录主要还是学习一个知识点：

输入：
```Python
L = [[1,2,3],[4,5,6],[7,8,9],[10,11,12]]
for i in L:
    print(i)
```

输出：
```
[1, 2, 3]
[4, 5, 6]
[7, 8, 9]
[10, 11, 12]
```

输入：
```Python
L = [[[1,11,111],2,3],[[4,44,444],5,6],[[7,77,777],8,9]]
for i,j,k in L:
    print(i)
```

输出：
```
[1, 11, 111]
[4, 44, 444]
[7, 77, 777]
```

输入：
```Python
L = [[[1,11,111],2,3],[[4,44,444],5,6],[[7,77,777],8,9]]
for i,j,k in L:
    for ii in i:
        print(ii)
```

输出：
```
1
11
111
4
44
444
7
77
777
```
