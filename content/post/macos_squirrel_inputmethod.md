---
title: "macos上的神级输入法squirrel"
date: 2018-12-22T17:10:35+08:00
lastmod: 2018-12-22T17:10:35+08:00
draft: false
keywords: ["macos输入法","五笔","squirrel"]
description: "macos上的神级输入法"
tags: ["macos输入法","五笔","squirrel"]
categories: ["mac"]
author: "beyondkmp"

---
squirrel输入法真是极客最想要的输入法，然而大多数国产输入法，都在忙着收集用户的输入数据，却没有把精力投入到输入法的核心功能。

squirrel目前的优势：

1. 配置相当丰富，可定制性，可玩性都非常高
2. 输入速度是非常快
3. 五笔支持的非常好
4. 不收集用户数据，这是我最看重的。

<!--more-->

## 编译安装

目前如果要安装squirrel最新版本是要自己编译安装。具体的编译过程如下:

``` sh
#安装Xcode的工具,运行下面的命令

xcode-select --install

# 使用 [Homebrew](http://brew.sh/)安装相差依赖库:
# dev tools:
brew install cmake
brew install git
# libraries:
brew install boost

#下载源代码
git clone --recursive https://github.com/rime/squirrel.git

#编译依赖库
make deps

#编译squirrel
make

#安装squirrel
sudo make install
```

## 部署

1. 下载官网的鼠须管安装包
2. 安装完成后不重启，使用上面的编译安装后重启
3. 「偏好设置 - 键盘 - 输入法」中添加「鼠须管」，重启系统
4. 下载配置文件和扩充词典文件~/Library/Rime/目录下面，重新部署，部署后会生成新的文件到build目录里面。

## 配置

**配置输入法的的外观和特性的几个文件**:

* squirrel.custom.yaml ，自定义皮肤；
* default.custom.yaml ，设定备选词数量，定义输入方案；
* luna_pinyin_simp.custom.yaml ，定义扩充词库、加载符号库、模糊拼音。明月拼音·简化字的输入方案配置文件，明月拼音对应的文件就是 luna_pinyin.custom.yaml；
* installation.yaml，定义配置文件保存到 Dropbox 文件夹

>以上几个以 .custom.yaml 作为后缀的文件，意味着是以补丁的方式来实现个性化定制的，输入法后续升级不会覆盖这些文件。所以自定义的文件配置中起始部分都会有patch:的字段，每个配置文件中有且只需要一行这个代码段。

### 界面配置

界面的配置文件是squirrel.custom.yaml, 目前我最喜欢的界面如下，自己修改了一些，还有就是目前不能对单个style进行patch，试了很多配置都没有解决。索性直接自己完整的写个color theme

```yaml
patch:
  show_notifications_when: appropriate # 状态通知，适当，也可设为全开（always）全关（never）
  "app_options/com.runningwithcrayons.Alfred-3/ascii_mode": true

  style/color_scheme: beyondkmp             # 方案命名，不能有空格
  preset_color_schemes:
    beyondkmp:
      name: 幽能／Psionics
      author: 雨過之後、佛振

      horizontal: true                                   # 水平排列
      inline_preedit: true                               # 单行显示，false双行显示
      candidate_format: "%c\u2005%@ \u2005 \u2005"       # 用 1/6 em 空格 U+2005 来控制编号 %c 和候选词 %@ 前后的空间。

      corner_radius: 5                                   # 候选条圆角
      border_height: 6                                   # 窗口边界高度，大于圆角半径才生效
      border_width: 6                                    # 窗口边界宽度，大于圆角半径才生效
      font_face: "PingFangSC-Regular,HanaMinB"      # 候选词字体
      font_point: 18                                     # 候选字词大小
      label_font_face: "STHeitiSC-Light"                 # 候选词编号字体
      label_font_point: 16                               # 候选编号大小
      text_color: 0xc2c2c2                               # 拼音行文字颜色，24位色值，16进制，BGR顺序
      back_color: 0x444444                               # 候选条背景色
      border_color: 0xE0B693                             # 边框色
      label_color: 0x9e9e9e                              # 预选栏编号颜色
      candidate_text_color: 0xeeeeee                     # 预选项文字颜色
      comment_text_color: 0x808080                       # 拼音等提示文字颜色
      hilited_text_color: 0xeeeeee                       # 高亮拼音 (需要开启内嵌编码)
      hilited_back_color: 0x444444                       # 高亮背景色
      hilited_candidate_text_color: 0xfafafa             # 第一候选项文字颜色
      hilited_candidate_back_color: 0xd8bf00             # 第一候选项背景背景色
      hilited_candidate_label_color: 0xfafafa            # 第一候选项编号颜色
      hilited_comment_text_color: 0x444444               # 注解文字高亮

```

界面效果如下:

![界面](/imgs/squirrel_interface.png)

### 自定义配置

配置文件为`default.custom.yaml`,目前的配置如下:

目前主要用luna_pinyin_simp和wubi_pinyin两种输入法

```yaml
patch:
  menu:
    page_size: 6  # 候选词的长度
  schema_list:
    - schema: luna_pinyin_simp      # 朙月拼音 简化字
    - schema: wubi_pinyin           # 五笔拼音混合輸入

  key_binder/bindings:  # 以方括号换页
    - when: paging # [ -> 上一页
      accept: bracketleft
      send: Page_Up
    - when: has_menu #] -下一页
      accept: bracketright
      send: Page_Down

  ascii_composer/switch_key:
    Caps_Lock: noop
    Control_L: noop
    Control_R: noop
    Eisu_toggle: clear
    Shift_L: commit_code # 配置后，可以直接用shift上屏英文字母,就是打了一段编码后直接以这段编码上屏
    Shift_R: noop
```

### 五笔配置

```yaml
# Rime schema settings
# encoding: utf-8

patch:
  switches:
    - name: ascii_mode
      reset: 0
      states: ["中文", "西文"]
    - name: full_shape
      states: ["半角", "全角"]
    - name: simplification
      reset: 0
      states: ["汉字", "漢字"]
    - name: ascii_punct
      states: ["。，", "．，"]
    - name: show_emoji
      reset: 1
      states: [ "🈚️️\uFE0E", "🈶️️\uFE0F" ]

  simplifier:
      opencc_config: simp2trad.json  # 简入繁出

  engine/filters:
    - simplifier
    - uniquifier
    - simplifier@emoji_conversion

  emoji_conversion:
    opencc_config: emoji.json
    option_name: show_emoji
    tags: abc


  "speller/max_code_length": 4 #最长4码
  "speller/auto_select": false #顶字上屏
  "speller/auto_select_unique_candidate": false #无重码自动上屏

  "translator/dictionary": wubi86 #加载五笔词库
  "reverse_lookup/comment_format/@1": xform/^(\w+).*/$1/

#  符号快速输入和部分符号的快速上屏
  punctuator:
    import_preset: symbols
    symbols:
      "/fs": [½, ‰, ¼, ⅓, ⅔, ¾, ⅒ ]
      "/bq": [😂️, 😅️, ☺️, 😱️, 😭️, 😇️, 🙃️, 🤔️, 💊️, 💯️, 👍️, 🙈️, 💩️, 😈️ ]
      "/dn": [⌘, ⌥, ⇧, ⌃, ⎋, ⇪, , ⌫, ⌦, ↩︎, ⏎, ↑, ↓, ←, →, ↖, ↘, ⇟, ⇞]
      "/fh": [ ©, ®, ℗, ℠, ™, ℡, ⓘ, ♂, ♀, ☉, ☊, ☋, ☌, ☍, ☐, ☑︎, ☒, ☜, ☝, ☞, ☟, ✎, ✄, ♲, ♻, ⚐, ⚑, ⚠]
      "/xh": [ ＊, ×, ✱, ★, ☆, ✩, ✧, ❋, ❊, ❉, ❈, ❅, ✿, ✲]
    half_shape:
      "#": "#"
      "*": "*"
      "`": "`"
      "~": "~"
      "@": "@"
      "=": "="
      "/": ["/", "÷"]
      '\': "、"
      "_" : "──"
      "'": {pair: ["「", "」"]}
      "[": ["【", "["]
      "]": ["】", "]"]
      "$": ["¥", "$", "€", "£", "¢", "¤"]
      "<": ["《", "〈", "«", "<"]
      ">": ["》", "〉", "»", ">"]
  recognizer/patterns/punct: "^/([a-z]+|[0-9]0?)$"

# 使用自定义词典 custom_phrase.txt
  custom_phrase:
    dictionary: ""
    user_dict: custom_phrase
    db_class: stabledb
    enable_completion: false
    enable_sentence: false
    initial_quality: 1
  "engine/translators/@4": table_translator@custom_phrase
```


## 参考

1. [How to Rime with Squirrel](https://github.com/rime/squirrel/blob/master/INSTALL.md)
2. [「鼠须管」的调教笔记](https://scomper.me/gtd/-shu-xu-guan-de-diao-jiao-bi-ji)
3. [我的鼠须管配置](https://placeless.net/blog/my-rime-squirrel-config)
4. [安装及配置 Mac 上的 Rime 输入法——鼠鬚管 (Squirrel)](https://placeless.net/blog/my-rime-squirrel-config)
5. [RimeWithSchemata](https://github.com/rime/home/wiki/RimeWithSchemata)
