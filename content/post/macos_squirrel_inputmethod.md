---
title: "macos上的神级输入法squirrel"
date: 2018-12-22T17:10:35+0800
lastmod: 2019-11-27T17:22:07+0800
draft: false
keywords: ["macos输入法","五笔","squirrel","rime"]
description: "macos上的神级输入法"
tags: ["macos输入法","五笔","squirrel","rime"]
categories: ["mac"]
author: "beyondkmp"

---
squirrel输入法是一个比较极客范的输入法，定制化与可玩性非常高, 由此带来的配置就比较复杂，而且没有界面，所以把常用配置记录下来方便直接使用。

squirrel目前的优势：

* 配置相当丰富, 可定制性高
* 输入速度快，兼容性好，兼容的系统版本非常多
* 五笔支持的非常好
* 不收集用户数据，这是我最看重的。

<!--more-->

## 安装

### 稳定版下载

官方最近终于更新版本，之前一直14年版本，有点太老了。官方已经更新到[0.14.0 (2019-06-23)](https://bintray.com/rime/squirrel/release), 可以直接官方下载安装。

### 最新版本下载

如果想尝试最新功能，可以下载squirrel测试版本 <https://bintray.com/rime/squirrel/testing/0.14.0%2Bgit08ed4f4>。最近新加了`vim_mode`功能，我就下载了最新版本。

### 编译安装

想自己修改一些功能的，或者使用最最新的功能，可以自己编译安装。具体的编译方法如下:

``` bash
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

cd squirrel

#编译依赖库
make deps

#编译squirrel
make

#安装squirrel
sudo make install
```


## 配置

目前要定制化自己的特殊配置，都要以`.custom.yaml`结尾，这样在输入法升级或者重新部署的时候都不会直接覆盖这些文件。 这些配置文件里面都要以`patch`开头，以打补丁的方式来实现个性化定制。

* squirrel.custom.yaml: 自定义皮肤, 鼠须管外观,地 ascii 模式;
* default.custom.yaml: 设定备选词数量，定义输入方案, 快捷键的设置;
* luna_pinyin_simp.custom.yaml: 定义扩充词库，加载符号库、模糊拼音;
* wubi_pinyin.custom.yaml: 定义扩充词库、加载符号库;
* installation.yaml: 同步和备份

### squirrel设置

界面的配置squirrel.custom.yaml, 目前主要设置了通知方式、皮肤和某些app的默认输入方式。

这里要说一下`vim_mode`, 在使用vim快捷方式app中有用，按esc键会自动把中文模式切换成英文模式，而不用先按shift切换英文再按esc.

```yaml
patch:
  show_notifications_when: appropriate # 状态通知，适当，也可设为全开（always）全关（never）
  style:
    color_scheme: psionics
    horizontal: true                                   # 水平排列
    inline_preedit: true                               # 单行显示，false双行显示
    candidate_format: "%c\u2005%@ \u2005 \u2005"       # 用 1/6 em 空格 U+2005 来控制编号 %c 和候选词 %@ 前后的空间。
    corner_radius: 5                                   # 候选条圆角
    border_height: 6                                   # 窗口边界高度，大于圆角半径才生效
    border_width: 6                                    # 窗口边界宽度，大于圆角半径才生效
    font_face: "PingFangSC-Regular,HanaMinB"           # 候选词字体
    font_point: 18                                     # 候选字词大小
    label_font_face: "STHeitiSC-Light"                 # 候选词编号字体
    label_font_point: 16                               # 候选编号大小

  # 默认使用ascii输入，不用中文输入
  # app的名字可以在/Applications/的app目录中的Info.plist里面查找
  app_options:
    com.apple.dt.Xcode:
      ascii_mode: true
    com.runningwithcrayons.Alfred:
      ascii_mode: true
    com.googlecode.iterm2:
      ascii_mode: true
      vim_mode: true                                   # vim mode, 按esc键会自动切换成英文
    com.microsoft.VSCode:
      ascii_mode: true
      vim_mode: true
    com.apple.finder:
      ascii_mode: true
    com.apple.appstore:
      ascii_mode: true
    com.apple.systempreferences:
      ascii_mode: true
    com.apple.keychainaccess:
      ascii_mode: true
    # com.apple.Safari:
    #   ascii_mode: true

  us_keyboard_layout: true
```

效果如下:

![界面](/imgs/squirrel_interface.png)

### 全局配置

配置文件为`default.custom.yaml`。 全局配置里面，可以定义输入方案、候选词数量和快捷键的设置。

1. 输入方案: 朙月拼音简化字和五笔拼音混合输入
2. 候选词设置为6个
3. 候选词换页的按键修改为: `[` 和 `]`
4. 修改shift_L为切所中英文(个人习惯)

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
    - when: has_menu # ] -> 下一页
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
4. [安装及配置 Mac 上的 Rime 输入法——鼠鬚管 (Squirrel)](https://www.dreamxu.com/install-config-squirrel/)
5. [RimeWithSchemata](https://github.com/rime/home/wiki/RimeWithSchemata)
6. [鼠须管输入法的新配色](https://scomper.me/gtd/shu-xu-guan-shu-ru-fa-de-xin-pei-se)
