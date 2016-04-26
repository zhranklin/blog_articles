# 打造一个现代化的vim编辑器: neovim + nyaovim + youcompleteme + airline
## 写在前面
对于vim不熟悉的小朋友们, 我提前解释一下:
- `<leader>`如果没有被修改过, 就是\, 反斜杠
- `<C-S>`的前缀C代表ctrl键所以是ctrl+s, `<S-SPACE>`, 前缀S代表shift, 所以是shift+space(不过我不确定neovim识不识别这个组合键)

## 简介
vim是一个非常折腾人的插件, 从使用vim到neovim, 感觉折腾这个玩意儿是非常非常费时的, 不过, 折腾也是一种乐趣咯。我的配置文件是以前(使用windows的时候)在百度上找到的一个gvim+插件+配置文件的打包, 而且针对不同平台做了处理, 所以在电脑上装的各个系统都能用, 不过现在找不到原作者了。当我开始使用mac的时候, 因为意识到不太可能用gvim了, 所以把很大部分的代码(包括用不到的设置, gvim相关的代码, 各种if语句)都删掉了, 添加了自己的配置。

## neovim
Bram Moolenaar 在写 Vim 时还是 90 年代初，至今已经 20 多年 过去了。其中，不仅包含了大量的遗留代码，而且程序的维护、Bug 的 修复、以及新特性的添加都变得越来越困难。为了解决这些问题，Neovim 项目应运而生。Neo 即“新”之意，它是 Vim 在这个新时代的重生。

根据 Neovim 的自述说明，在总体上，它将达到下列目的：

- 通过简化维护以改进 Bug 修复及特性添加的速度；
- 分派各个开发人员的工作；
- 实现新的、现代化的用户界面，而不必修改核心源代码；
- 利用新的、基于协同进程的新插件架构改善扩展性，并支持使用任何语言 编写插件

以上介绍来自[linuxtoy](https://linuxtoy.org/archives/neovim.html)

此外, 在最近的版本中, 还有非常值得注意的几点:

- 实现了嵌入式终端模拟器, 可以跟各种REPL插件说再见了
- 使用远程API(好像是socket), 不光能使用各种语言编写插件, 而且可以很方便的编写GUI版本, 甚至嵌入至IDE中
- 由于内部优化了事件监听器还是什么的, 代码粘贴的时候, 可以自动识别, 不像vim里一样需要:set paste, 不然会出现蜜汁缩进和括号对
- 直接支持剪贴板, 不需要重新编译


## neovim基本配置
### neovim安装
neovim已经被打包到了linux各个发行版上, 可以直接用对应的包管理工具安装, windows和安卓(什么鬼...)也可以。直接参照wiki上的[Installing-Neovim](https://github.com/neovim/neovim/wiki/Installing-Neovim)

mac上直接用brew安装:

```bash
brew install neovim/neovim/neovim
```

不过在这里还没结束, 很多vim插件都是用python写的, 为了对这些插件进行支持, 还需要安装python上的neovim模块, 用pip安装:

### python模块
```bash
pip install neovim
```

在我的mac上安装的时候, 似乎有权限问题, 所以还得加上sudo -H

呃, 如果机器上没有pip, 就安装一个, 同理, 用homebrew, apt-get之类的包管理工具即可。

最后, 使用nvim命令即可打开neovim编程器:

`nvim`

### cscope/ctags
还有一些插件需要事先安装ctags和cscope(用包管理器就可以了)

### 从vim迁移
由于neovim支持vim的所有特性, 从vim迁移到neovim是一件很容易的事情, wiki上给出了一个方案:

```bash
mkdir -p ${XDG_CONFIG_HOME:=$HOME/.config}
ln -s ~/.vim $XDG_CONFIG_HOME/nvim
ln -s ~/.vimrc $XDG_CONFIG_HOME/nvim/init.vim
```

做了一个简单的链接工作, 所以可以看出~/.vimrc和~/.config/nvim/init.vim, ~/.vim和~/.config/nvim是一一对应的。所以以前没有安装过vim的朋友, 直接把配置文件.vimrc的代码放进init.vim即可。由于neovim的某些指令在vim中是不能使用的, 所以可使用has('nvim')来判断当前使用的版本:

```
if has('nvim')
  ...
endif
```

当然了neovim对vim的偶尔还是会有不兼容的情况, 不过一般情况下直接使用nvim, 和原来的外观应该没有区别。

### 剪贴板集成
:h nvim_clipboard可以查看相关帮助, 只要在路径下找到下面的命令, 剪贴板就能自动启用:

- xclip
- xsel (newer alternative to xclip)
- pbcopy/pbpaste (only for Mac OS X)
- lemonade (useful for SSH machine)

mac自带pbcopy和pbpaste, 所以不需要另外安装, 配置完毕后, 在操作前面加上"+前缀就可以对剪贴板操作了, 如`"+p ` `"+dd`之类的。

### 兼容输入法
在vim下使用中文输入法是一件非常蛋疼的事情, 如果需要输入中文, 在普通模式和插入模式下, 得不停地切换输入法, 不过还是有解决办法的。linux用户直接安装fcitx.vim插件即可:

```vim
Plug 'fcitx.vim'
```

而mac则相对麻烦一些:

#### mac下vim兼容输入法
安装[fcitx-remote-for-osx](https://github.com/CodeFalling/fcitx-remote-for-osx)

```bash
brew install fcitx-remote-for-osx --with-input-method=baidu-pinyin
```
--with-input-method=baidu-pinyin参数表示是为百度拼音安装的, 这个插件支持若干个输入法:

```bash
--with-input-method=
  Select input method: baidu-pinyin(default), baidu-wubi, sogou-pinyin, qq-wubi, squirrel-rime, osx-pinyin, osx-wubi
```
根据自己系统输入法选择对应的参数即可。

然后在vim上安装osx上的fcitx.vim--[fcitx-vim-osx](https://github.com/CodeFalling/fcitx-vim-osx):

`Plug 'CodeFalling/fcitx-vim-osx'`

### 使用嵌入式终端模拟器
个人觉得这是neovim杀手级特性之一, 尤其是使用gui的时候, 下面有一个嵌入式的终端简直方便。

使用这个命令:

```vim
:e term://bash
```

或者

```vim
:e term://zsh
```

或者

```vim
:e term://python
```

都是可以直接打开一个内置终端的, 只要在term://后面加上命令即可。当然了从上面可以看出, term://...是被当做文件来编辑的, 所以也可以:

```vim
:vs term://scala
```

就可以垂直的在左边打开scala REPL了。刚开始是插入模式, 可以直接与终端交互, 如果要退出插入模式就得用快捷键`<C-\><C-n>`了, 因为终端有时也要识别<esc>的嘛。当然了, 如果觉得一次按两个键太麻烦, 可以做一个map(只有neovim下才有tnormap):

 ```vim
if has('nvim')
    tnoremap <Esc> <C-\><C-n>
endif 
 ```

这样就能像平时一样按esc退出编辑了。这里我添加了一个快捷键:

```vim
nmap t<Enter> :bo sp term://zsh\|resize 5<CR>i
```

在普通模式下只要按t回车, 就会在底部出现一个5行大小的终端模拟器。平时不用zsh的话, 把zsh换成bash就好了。

一般来说, 如果使用REPL的话, 还能用vim进行代码高亮, 这是直接用终端不太容易做到的, 比如:bo sp term://scala\|resize 5<CR>i之后, 可以使用scala的代码高亮:

```vim
:set filetype=scala
```

python同理。

**tips**: 直接输入:set ft<TAB>就会变成:set filetype

## 插件
### vim-plug
#### vim-plug安装
[github](https://github.com/junegunn/vim-plug)上给了一个最简单的方式:

```bash
curl -fLo ~/.config/nvim/autoload/plug.vim --create-dirs \
    https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
```

#### vim-plug使用
我这里一共安装了30多个插件, 一个一个去下, 再覆盖是不可能的...所以github上有若干个插件包管理工具, 目前来说, 最快的应该就是vim-plug了, 可以同时下载不同插件。在.vimrc(init.vim)文件中, 按这样的格式声明插件即可, 单引号里面是github上的项目名(带斜杠), 或者vim.org上的名字(不带斜杠, 就默认为vim-scripts/...):

```vim
call plug#begin('~/.vim/plugged')
Plug '...'
Plug '...'
...
call plug#end()
```

保存完毕重启之后, :PlugInstall即可安装所有插件(参见后面附上的init.vim文件), 使用:PlugUpdate可以对插件进行更新, 所有的插件都会放在plugged/目录下。个别插件好像会安装不了, 手动进入plugged, 自行git clone就可以了(但是不要删掉Plug '...'那一行, 不然插件不会启用)。

### 代码补全/辅助工具
- YouCompleteMe 一个非常强大的工具, 安装见后面介绍
- auto-pairs 括号自动补全, 功能其实比想象的要多一些, 详见github

#### nerdcommenter
可以使用快捷键插入注释, 默认快捷键如下:

```vim
<Leader>ci 以每行一个 /* */ 注释选中行(选中区域所在行)，再输入则取消注释
<Leader>cm 以一个 /* */ 注释选中行(选中区域所在行)，再输入则称重复注释
<Leader>cc 以每行一个 /* */ 注释选中行或区域，再输入则称重复注释
<Leader>cu 取消选中区域(行)的注释，选中区域(行)内至少有一个 /* */
<Leader>ca 在/*...*/与//这两种注释方式中切换（其它语言可能不一样了）
<Leader>cA 行尾注释
```

以下是github上的说明:

**[count]<leader>cc |NERDComComment|**<br>
Comment out the current line or text selected in visual mode.

**[count]<leader>cn |NERDComNestedComment|**<br>
Same as <leader>cc but forces nesting.

**[count]<leader>c<space> |NERDComToggleComment|**<br>
Toggles the comment state of the selected line(s). If the topmost selected line is commented, all selected lines are uncommented and vice versa.

**[count]<leader>cm |NERDComMinimalComment|**<br>
Comments the given lines using only one set of multipart delimiters.

**[count]<leader>ci |NERDComInvertComment|**<br>
Toggles the comment state of the selected line(s) individually.

**[count]<leader>cs |NERDComSexyComment|**<br>
Comments out the selected lines "sexily"

**[count]<leader>cy |NERDComYankComment|**<br>
Same as <leader>cc except that the commented line(s) are yanked first.

**<leader>c$ |NERDComEOLComment|**<br>
Comments the current line from the cursor to the end of line.

**<leader>cA |NERDComAppendComment|**<br>
Adds comment delimiters to the end of line and goes into insert mode between them.

**|NERDComInsertComment|**<br>
Adds comment delimiters at the current cursor position and inserts between. Disabled by default.

**<leader>ca |NERDComAltDelim|**<br>
Switches to the alternative set of delimiters.

**[count]<leader>cl**<br>
**[count]<leader>cb |NERDComAlignedComment|**<br>
Same as |NERDComComment| except that the delimiters are aligned down the left side (<leader>cl) or both sides (<leader>cb).

**[count]<leader>cu |NERDComUncommentLine|**<br>
Uncomments the selected line(s).


#### vim-surround
surround可以用来修改括号, 方括号, 标签等等包围在两边的元素, 下面是[github](https://github.com/tpope/vim-surround)上面的说明:

It's easiest to explain with examples.  Press `cs"'` inside

    "Hello world!"

to change it to

    'Hello world!'

Now press `cs'<q>` to change it to

    <q>Hello world!</q>

To go full circle, press `cst"` to get

    "Hello world!"

To remove the delimiters entirely, press `ds"`.

    Hello world!

Now with the cursor on "Hello", press `ysiw]` (`iw` is a text object).

    [Hello] world!

Let's make that braces and add some space (use `}` instead of `{` for no
space): `cs]{`

    { Hello } world!

Now wrap the entire line in parentheses with `yssb` or `yss)`.

    ({ Hello } world!)

Revert to the original text: `ds{ds)`

    Hello world!

Emphasize hello: `ysiw<em>`

    <em>Hello</em> world!

Finally, let's try out visual mode. Press a capital V (for linewise
visual mode) followed by `S<p class="important">`.

    <p class="important">
      <em>Hello</em> world!
    </p>

This plugin is very powerful for HTML and XML editing, a niche which
currently seems underfilled in Vim land.  (As opposed to HTML/XML
*inserting*, for which many plugins are available).  Adding, changing,
and removing pairs of tags simultaneously is a breeze.

The `.` command will work with `ds`, `cs`, and `yss` if you install
[repeat.vim](https://github.com/tpope/vim-repeat).

总结一下就是surround的命令有三种, cs, ds, 以及ys 分别是修改, 删除, 添加周围的东西:
- cs + 原符号 + 新符号
- ds + 原符号, 删除周围的元素
- ys + 范围(可以是文本对象iw之类的) + 新符号, 添加
- 可视模式模式下, S + 新符号, 也可以添加

对于修改和添加, 如果新符号为对称的左边边部分(如左括号等), 则会在中间添加空格, 否则不添加空格。

#### EasyAlign
一个用于代码对齐的神器! 下面是[github](https://github.com/junegunn/vim-easy-align)上的gif截图:

![](https://raw.githubusercontent.com/junegunn/i/master/easy-align/equals.gif)

首先在init.vim文件中添加下面的代码

```vim
" Start interactive EasyAlign in visual mode (e.g. vipga)
xmap ga <Plug>(EasyAlign)

" Start interactive EasyAlign for a motion/text object (e.g. gaip)
nmap ga <Plug>(EasyAlign)
```

指令示例:

- `vipga=`
    - `v`isual-select `i`nner `p`aragraph
    - Start EasyAlign command (`ga`)
    - Align around `=`
- `gaip=`
    - Start EasyAlign command (`ga`) for `i`nner `p`aragraph
    - Align around `=`

##### 预定义的规则

该插件预定义了一些字符, 用于表示对齐的规则: `<Space>`, `=`, `:`, `.`, `|`, `&`, `#`, and `,`.

##### `=`

![](https://raw.githubusercontent.com/junegunn/i/master/easy-align/equals.gif)

- `=` Around the 1st occurrences
- `2=` Around the 2nd occurrences
- `*=` Around all occurrences
- `**=` Left/Right alternating alignment around all occurrences
- `<Enter>` Switching between left/right/center alignment modes

##### `<Space>`

![](https://raw.githubusercontent.com/junegunn/i/master/easy-align/spaces.gif)

- `<Space>` Around the 1st occurrences of whitespaces
- `2<Space>` Around the 2nd occurrences
- `-<Space>` Around the last occurrences
- `<Enter><Enter>2<Space>` Center-alignment around the 2nd occurrences

##### `,`

![](https://raw.githubusercontent.com/junegunn/i/master/easy-align/commas.gif)

- The predefined comma-rule places a comma right next to the preceding token
  without margin (`{'stick_to_left': 1, 'left_margin': 0}`)
- You can change it with `<Right>` arrow

##### 使用正则表达式

![](https://raw.githubusercontent.com/junegunn/i/master/easy-align/regex.gif)

You can use an arbitrary regular expression by
- pressing `<Ctrl-X>` in interactive mode
- or using `:EasyAlign /REGEX/` command in visual mode or in normal mode with
  a range (e.g. `:%`)

##### 也可以使用:EasyAlign命令

![](https://raw.githubusercontent.com/junegunn/i/master/easy-align/modes.gif)

This demo shows how you can start interative mode with visual selection or use
non-interactive `:EasyAlign` command.

##### 对齐单元格
这个在写markdown的时候很方便

![](https://raw.githubusercontent.com/junegunn/i/master/easy-align/tables.gif)

Check out various alignment options and "live interative mode".

##### 能够识别部分语法

![](https://raw.githubusercontent.com/junegunn/i/master/easy-align/yaml.gif)

Delimiters in strings and comments are ignored by default.

##### 块可视模式

![](https://raw.githubusercontent.com/junegunn/i/master/easy-align/blockwise-visual.gif)

You can limit the scope with blockwise-visual mode.

##### 使用流程

<img src="https://raw.githubusercontent.com/junegunn/i/master/easy-align/usage.png" width="469">

这个插件还支持各种复杂的指令, 详情就参见github吧。

### 外观类插件
- indentLine 在缩进的时候能显示竖线, 还能设定颜色, 如果需要修改的话参见github
- syntastic 在文件保存的时候能自动进行语法检查, 并标出(配置比较复杂, 我只是简单的设置了一下)
- numbers 有点丑但是很实用的插件, 就是在普通模式下, 会显示相对行号, 做跨行指令的时候就比较容易了
- vim-gitgutter 能在侧边标出git修改情况

#### solarized
这是一个非常流行的主题, 有浅色和深色的版本, 可以设置:

```vim
set background=light
```

如果使用这个的话, 在后面介绍的nyaovim里面会出现一些问题, 需要改几行代码(后面再说)。

#### gruvbox 
差不多就是solarized的"暖屏"版, 个人更喜欢这个, 也可以设置深色和浅色。

不管使用哪个主题, 都要记得把另一个的`Plug '...'`代码给注释掉。上面两个主题都有更深入的参数设置, 参见github

#### airline
一个非常好用的编辑器底栏, 以优雅的方式显示当前文件的各种信息, 模式啊目录啊git增删改的行数(需要安装前面说的gitguter)等等, 还有顶部有一表情的形式显示的文件缓冲区, 可以用快捷键直接编辑相应的文件(我以我的就是t<number>了):

```vim
nmap t1 <Plug>AirlineSelectTab1
nmap t2 <Plug>AirlineSelectTab2
nmap t3 <Plug>AirlineSelectTab3
nmap t4 <Plug>AirlineSelectTab4
nmap t5 <Plug>AirlineSelectTab5
nmap t6 <Plug>AirlineSelectTab6
nmap t7 <Plug>AirlineSelectTab7
nmap t8 <Plug>AirlineSelectTab8
nmap t9 <Plug>AirlineSelectTab9
nmap t[ <Plug>AirlineSelectPrevTab
nmap t] <Plug>AirlineSelectNextTab
```

还有底部显示所用的字符我按照github上给的样例进行了设置(见最后的配置文件)。

为了符合当前使用的主题的配色, 还需要安装vim-airline-themes。

### 常用工具/文件导航类插件
- repeat.vim 配合surround使用, 可以用.来重复上一个surround命令
- ccvext.vim 用来连接ctags和cscope, 不太会用
- nerdtree 查看当前路径下的目录树, 在我的配置中用<F4>可以呼出
- ctrlp.vim 查找文件的神器, 顾名思义ctrl+p可以呼出, 只要输入零散的字符就可以匹配(比如lne可以匹配**L**ISCE**N**S**E**)
- SrcExpl 按<F8>呼入呼出, 会出来一个小窗口, 显示光标下函数/类等等的源代码, 第一次使用的时候会用ctags生成标签
- tagbar 显示当前代码的结构, 普通模式下tb呼出
taglist.vim 与tagbar差不多, 普通模式下tl呼出
- dash.vim mac下用于跳转到dash的工具, 不过我没怎么用

#### Mark
一个很好用的标记工具, 可以将不同的词标成不同的颜色, 比如标记在变量名上, 就很方便。

可以直接用<leader>m或者在可选模式下键入<leader>m , 或者用<leader>r然后键入正则表达式, 标记所有匹配的字符串, 以及使用:Mark 命令匹配正则表达式也是可以的, 用:MarkClear用来清除所有标记。

### 语言相关的插件
#### c/c++
c.vim, 一个写c/c++的神器, 内置了各种模板代码, 用的话还是参考一下github吧, 当前版本为5.18, 不过作者自己维护的版本出了最新版6.0, 在neovim下与youcompleteme不兼容, 所以还是选择了vim-scripts/c.vim, 参考:h csupport-usage-vim 或者一个简单的快捷键介绍~/.vim/plugged/c.vim/doc/c-hotkeys.pdf(虽然这个插件并不止这些东西, 但是已经很实用了)

还是列出来吧:

```shell
-- Help ---------------------------------------------------------------

   \hm       show manual for word under the cursor (n,i)
   \hp       show plugin help                      (n,i)

-- Comments -----------------------------------------------------------

[n]\cl       end-of-line comment                 (n,v,i)
[n]\cj       adjust end-of-line comment(s)       (n,v,i)
   \cs       set end-of-line comment column      (n)
[n]\c*       code -> comment /* */               (n,v)
[n]\cc       code -> comment //                  (n,v)
[n]\co       comment -> code                     (n,v)
   \cfr      frame comment                       (n,i)
   \cfu      function comment                    (n,i)
   \cme      method description                  (n,i)
   \ccl      class description                   (n,i)
   \cfdi     file description (implementation)   (n,i)
   \cfdh     file description (header)           (n,i)
   \ccs      C/C++-file section  (tab. compl.)   (n,i)
   \chs      H-file section      (tab. compl.)   (n,i)
   \ckc      keyword comment     (tab. compl.)   (n,i)
   \csc      special comment     (tab. compl.)   (n,i)
   \cd       date                                (n,v,i)
   \ct       date \& time                        (n,v,i)
[n]\cx       toggle comments: C <--> C++         (n,v,i)

-- Statements ---------------------------------------------------------

   \sd       do { } while                        (n,v,i)
   \sf       for                                 (n,i)
   \sfo      for { }                             (n,v,i)
   \si       if                                  (n,i)
   \sif      if { }                              (n,v,i)
   \sie      if else                             (n,v,i)
   \sife     if { } else { }                     (n,v,i)
   \se       else { }                            (n,v,i)
   \sw       while                               (n,i)
   \swh      while { }                           (n,v,i)
   \ss       switch                              (n,v,i)
   \sc       case                                (n,i)
   \s{ \sb   { }                                 (n,v,i)

-- Preprocessor -------------------------------------------------------

   \ps       choose a standard library include   (n,i)
   \pc       choose a C99 include                (n,i)
   \p<       #include <>                         (n,i)
   \p"       #include ""                         (n,i)
   \pd       #define                             (n,i)
   \pu       #undef                              (n,i)
   \pif      #if  #endif                         (n,v,i)
   \pie      #if  #else #endif                   (n,v,i)
   \pid      #ifdef #else #endif                 (n,v,i)
   \pin      #ifndef #else #endif                (n,v,i)
   \pind     #ifndef #def #endif                 (n,v,i)
   \pi0      #if 0 #endif                        (n,v,i)
   \pr0      remove #if 0 #endif                 (n,i)
   \pe       #error                              (n,i)
   \pl       #line                               (n,i)
   \pp       #pragma                             (n,i)

-- Idioms -------------------------------------------------------------

   \if       function                            (n,v,i)
   \isf      static function                     (n,v,i)
   \im       main()                              (n,v,i)
[n]\i0       for( x=0; x<n; x+=1 )               (n,v,i)
[n]\in       for( x=n-1; x>=0; x-=1 )            (n,v,i)
   \ie       enum   + typedef                    (n,i)
   \is       struct + typedef                    (n,i)
   \iu       union  + typedef                    (n,i)
   \ip       printf()                            (n,i)
   \isc      scanf()                             (n,i)
   \ica      p=calloc()                          (n,i)
   \ima      p=malloc()                          (n,i)
   \ire      p=realloc()                         (n,i)
   \isi      sizeof()                            (n,v,i)
   \ias      assert()                            (n,v)
   \ii       open input file                     (n,i)
   \io       open output file                    (n,i)
   \ifs      fscanf                              (n,i)
   \ifp      fprintf                             (n,i)

-- Snippets -----------------------------------------------------------

   \nr       read code snippet                   (n,i)
   \nw       write code snippet                  (n,v,i)
   \ne       edit code snippet                   (n,i)
[n]\nf       pick up function prototype          (n,v,i)
[n]\np       pick up function prototype          (n,v,i)
[n]\nm       pick up method prototype            (n,v,i)
   \ni       insert prototype(s)                 (n,i)
   \nc       clear  prototype(s)                 (n,i)
   \ns       show   prototype(s)                 (n,i)
   \ntl      edit local templates                (n,i)
   \ntg      edit global templates               (n,i)
   \ntr      rebuild templates                   (n,i)

-- C++ ----------------------------------------------------------------

   \+co      cout  <<  << endl;                  (n,i)
   \+"       << ""                               (n,i)
   \+c       class                               (n,i)
   \+ps      #include <...> STL                  (n,i)
   \+pc      #include <c..> C                    (n,i)
   \+cn      class (using new)                   (n,i)
   \+ci      class implementation                (n,i)
   \+cni     class (using new) implementation    (n,i)
   \+mi      method implementation               (n,i)
   \+ai      accessor implementation             (n,i)

   \+tc      template class                      (n,i)
   \+tcn     template class (using new)          (n,i)
   \+tci     template class implementation       (n,i)
   \+tcni    template class (using new) impl.    (n,i)
   \+tmi     template method implementation      (n,i)
   \+tai     template accessor implementation    (n,i)

   \+tf      template function                   (n,i)
   \+ec      error class                         (n,i)
   \+tr      try ... catch                       (n,v,i)
   \+ca      catch                               (n,v,i)
   \+c.      catch(...)                          (n,v,i)

-- Run ----------------------------------------------------------------

  \rc       save and compile                    (n,i)
  \rl       link                                (n,i)
  \rr       run                                 (n,i)
  \ra       set comand line arguments           (n,i)
  \rm       run make                            (n,i)
  \rmc      run 'make clean'                    (n,i)
  \rcm      choose makefile                     (n,i)
  \rme      executable to run                   (n,i)
  \rma      cmd. line arg. for make             (n,i)
  \rp       run splint                          (n,i)
  \rpa      cmd. line arg. for splint           (n,i)
  \rk       run CodeCheck (TM)                  (n,i)
  \rka      cmd. line arg. for CodeCheck (TM)   (n,i)
  \rd       run indent                          (n,v,i)
[n]\rh       hardcopy buffer                     (n,v,i)
  \rs       show plugin settings                (n,i)
  \rx       set xterm size                      (n, only Linux/UNIX & GUI)
  \ro       change output destination           (n,i)

```

它还有自动编译链接的功能, 不过我没用过

#### java
javacomplete2是javacomplete的升级版, 可以配合YouCompleteMe使用如果在maven/gradle项目下面还能自动找到项目相关的补全(似乎还有重构的功能?没研究过)。

#### html/css/javascript
emmet-vim, emmet太有名了, 开发前端的都知道, 详见github吧。

#### python
REPL直接使用前面说的嵌入式终端即可, 补全在YouCompleteMe里面有(还包含了代码跳转之类的功能)。

#### scala/clojure
用的人不多, 我就不说了...

## YouCompleteMe
### 简介/使用
代码补全神器, 对所有的代码支持简单(基于标识符)的补全, 对c/c++/c#/javascript/typescript/python/rust/go有良好的支持, 也能和现有的代码补全整合。前面这些语言还支持跳转到goto(跳转到各种地方, 定义, 父类, 说明文档[包括javadoc]等等), fix重构等等功能。其他语言如果使用ctags生成了标签, 也是能直接支持的, 也支持补全文件路径。

使用的时候如果没有弹出补全列表, 可以用`<c-space>`呼出, 可以上下键或者直接用`<tab>/<s-tab>`切换。(选中之后直接打后面的代码就可以了不要按回车...)

有很多插件已经做了相关适配, 虽然没有这么多功能, 但是补全还是支持的。

要使用ycm的各种功能, 还需要绑定子命令的快捷键(参见[sub-command](https://github.com/Valloric/YouCompleteMe#ycmcompleter-subcommands)), 触发子命令的方式为:YcmCompleter 子命令 回车, 所以绑定快捷键的话, 例如:

```vim
nnoremap <leader>jd :YcmCompleter GoTo
```

就可以使用<leader>jd跳转到定义了

**tips**: 跳转过程中`<c-o>/<c-i>`用来"前进"/"后退", 平时也可以用(常识吧..)

### 安装
github上有详细的安装过程。

我们用的是neovim, 所以忽略他说的版本问题, 使用vim-plug完成安装后, cd到~/.vim/plugged/YouCompleteMe进行编译

不过在编译之前需要安装cmake和python头文件(mac下homebrew版python已经有头文件了), 这两个依赖的安装参照github上的说明。

完毕之后, 如果系统里同时有python2和3的话..他会默认使用python2编译, 使用python2编译的版本在不全的时候如果出现了非ascii就会有崩溃, 所以最好在python3的环境下编译, 这件事情折腾了我很久很久很久...不过有pyenv这个工具非常方便的实现这个功能。github上面有, 自己装吧。

装完之后(zsh用户请进入bash执行):

```bash
CONFIGURE_OPTS="--enable-shared" pyenv install 3.5.0
pyenv global 3.5.0
```

CONFIGURE_OPTS="...", 表示编译的时候加的参数, 一定要加上, 不然就没有头文件了。

后面的命令会把当前的python环境转成python3, 执行python --version就知道效果了

然后执行`./install.py`, 一定要记得根据需要加上参数(比如对c/c++的支持), 可以用`./install.py --help`查看有哪些选项。

如果有c的话会下载一个很大的clang.zip..耐心等待就好了(在win下要有mv支持, osx下要有x-code-command-line-tools, linux下要有c/c++编译环境, 不然装不了)。

不过我在用pyenv指定的环境下编译出来的ycm, 一打开neovim就会闪退, 完全不像python的作风..后来一看系统出来的信息似乎是python动态库访问了非法内存(真的好无语...), 所以是python的bug么...无奈之下, 试一试用系统的python(pyenv的python是从python.org下载的, 跟系统自带(我忘了是自带还是用homebrew装的了)的应该不一样), `where python3`发现python3的目录是`/Library/Frameworks/Python.framework/Versions/3.5/`, 于是我把pyenv下的python删了, 换成这个版本

```bash
cd ~/.pyenv/versions
rm -rf 3.5.0
cp /Library/Frameworks/Python.framework/Versions/3.5 3.5.0
```

然后进入3.5.0/bin/, 把所有xxx3或xxx-3(或者3.5)的文件都复制成xxx, 然后重新编译一次ycmd, 成功!

**注意**: 安装完之后, 要执行一下:UpdateRemotePlugins, 很多远程插件都要执行这个命令的。

### 配置
安装完之后, 还需要一些简单的配置, ycm\_python\_binary\_path和ycm\_server\_python\_interpreter分别指定了python补全的版本和运行ycmd使用的解释器, 由于刚在是在python3环境下编译ycmd的, 所以后者必须是python3。

```bash
let g:ycm_python_binary_path = '/Library/Frameworks/Python.framework/Versions/3.5/bin/python3'
let g:ycm_server_python_interpreter = '/Library/Frameworks/Python.framework/Versions/3.5/bin/python3'
```


编辑cpp文件的时候, 还需要专门配置一个文件.ycm\_extra\_conf.py, 可以放在项目根目录, 或者放在用户主目录, 参考[See YCM's own .ycm\_extra\_conf.py](https://github.com/Valloric/ycmd/blob/master/cpp/ycm/.ycm_extra_conf.py), 根据需要在自己修改相关的编译参数

实际编辑的时候, 会发现ycm并不能找到std的库, faq里面有说明, 需要把相关的库添加到.ycm\_extra\_conf.py的flags中, `echo | clang -v -E -x c++ -`, 查看`#include  search starts here:`后面的路径, 为每一个路径添加一个`-isystem`参数(添加到flags数组里), 比如我执行`echo | clang -v -E -x c++ -`的结果是:

```bash
Apple LLVM version 7.3.0 (clang-703.0.29)
Target: x86_64-apple-darwin15.4.0
Thread model: posix
InstalledDir: /Library/Developer/CommandLineTools/usr/bin
 "/Library/Developer/CommandLineTools/usr/bin/clang" -cc1 -triple x86_64-apple-macosx10.11.0 -Wdeprecated-objc-isa-usage -Werror=deprecated-objc-isa-usage -E -disable-free -disable-llvm-verifier -main-file-name - -mrelocation-model pic -pic-level 2 -mthread-model posix -mdisable-fp-elim -masm-verbose -munwind-tables -target-cpu core2 -target-linker-version 264.3.101 -v -dwarf-column-info -resource-dir /Library/Developer/CommandLineTools/usr/bin/../lib/clang/7.3.0 -stdlib=libc++ -fdeprecated-macro -fdebug-compilation-dir /Users/Zhranklin -ferror-limit 19 -fmessage-length 157 -stack-protector 1 -fblocks -fobjc-runtime=macosx-10.11.0 -fencode-extended-block-signature -fcxx-exceptions -fexceptions -fmax-type-align=16 -fdiagnostics-show-option -fcolor-diagnostics -o - -x c++ -
clang -cc1 version 7.3.0 (clang-703.0.29) default target x86_64-apple-darwin15.4.0
ignoring nonexistent directory "/usr/include/c++/v1"
#include "..." search starts here:
#include <...> search starts here:
 /Library/Developer/CommandLineTools/usr/bin/../include/c++/v1
 /usr/local/include
 /Library/Developer/CommandLineTools/usr/bin/../lib/clang/7.3.0/include
 /Library/Developer/CommandLineTools/usr/include
 /usr/include
 /System/Library/Frameworks (framework directory)
 /Library/Frameworks (framework directory)
End of search list.
# 1 "<stdin>"
# 1 "<built-in>" 1
# 1 "<built-in>" 3
# 336 "<built-in>" 3
# 1 "<command line>" 1
# 1 "<built-in>" 2
# 1 "<stdin>" 2
```

那么我的.ycm\_extra\_conf.py文件就要添加:

```python
flags=[
  ...
  ...
  '-isystem',
  '/Library/Developer/CommandLineTools/usr/bin/../include/c++/v1',
  '-isystem',
  '/usr/local/include',
  '-isystem',
  '/Library/Developer/CommandLineTools/usr/bin/../lib/clang/7.3.0/include',
  '-isystem',
  '/Library/Developer/CommandLineTools/usr/include',
  '-isystem',
  '/usr/include',
  '-isystem',
  '/System/Library/Frameworks',
  '-isystem',
  '/Library/Frameworks',
]
```

## nyaovim
nyaovim是这篇文章非常重要的一部分, 鉴于neovim非常好的扩展性, 已经有很多的neovim GUI被开发出来了, nyaovim是一款我认为比较实用的GUI, nyaovim使用electron开发的, 安装什么的比较方便, 而且很好扩展, 在github上搜索nyaovim就能看到不少nyaovim插件:

- 内置浏览器
- 工具栏提示
- markdown预览

还有发弹幕, 播放mp3什么的(汗...)

对我来说, 最常用的就是markdown预览了, markdown编辑器有很多, 有些也能支持vi语法, 但是, 他们不可能有插件吧..所以这么个功能还是非常有好处的。

### 安装
nyaovim的安装很简单:

```bash
npm install -g nyaovim
```

呃, 不会连npm都没有吧, 装一个:

```bash
brew install nyaovim
```

linux各大发行版, 方式类似。

我没有记错的话, 安装完nyaovim也是要运行一下:UpdateRemotePlugins的

在命令行中输入:

```bash
nyaovim <文件名>
```

就会弹出electron的GUI界面了, 里面就是熟悉的neovim!

由于是用electron写的, 自己进行界面布局调整很方便(前提是熟练使用html/css/js), nyaovim相关的配置在`~/.config/nyaovim/nyaovimrc.html`文件中(详见后面的代码清单), 可以设置字体, 颜色之类的, 还有要使用插件, 则必须在里面配置。事先已经安装好了markdown preview(也就是Plug 'rhysd/nyaovim-markdown-preview')了, 所以只需要在nyaovimrc.html里面配置一下就好了。给个软件截图:

![](http://7xt2lb.com2.z0.glb.clouddn.com/屏幕快照%202016-04-27%20上午2.14.11.png)


## 代码清单
有很多东西如快捷键设置等等在前面都没有提及, 还请参看后面的.vimrc

### ~/.config/nyaovim/nyaovimrc.html
```html
<dom-module id="nyaovim-app">
  <template>
    <style>
      /* CSS configurations here */
      .horizontal {
        display: flex;
        width: 100%;
        height: 100%;
      }
      neovim-editor {
        width: 100%;
        height: 100%;
      }
    </style>
    <!-- Component tags here -->
    <div class="horizontal">
        <neovim-editor 
          id="nyaovim-editor"
          argv$="[[argv]]"
          font-size="18"
          font="monofur,monofur for Powerline"
          line-height="1.2"
        ></neovim-editor>
        <!-- Put component -->
        <markdown-preview editor="[[editor]]"></markdown-preview>
    </div>
  </template>
</dom-module>
```

### ~/.vimrc
```vim
call plug#begin('~/.vim/plugged')

" 自动补全/代码辅助
Plug 'Valloric/YouCompleteMe'
Plug 'jiangmiao/auto-pairs' "自动括号匹配
Plug 'scrooloose/nerdcommenter'
Plug 'tpope/vim-surround'
Plug 'junegunn/vim-easy-align'

" 外观
Plug 'Yggdroot/indentLine'
Plug 'scrooloose/syntastic'
Plug 'myusuf3/numbers.vim'
Plug 'vim-airline/vim-airline'
Plug 'vim-airline/vim-airline-themes'
Plug 'morhetz/gruvbox'
Plug 'airblade/vim-gitgutter'
" Plug 'altercation/vim-colors-solarized'

" 常用工具
Plug 'Mark--Karkat'
Plug 'repeat.vim'
Plug 'ccvext.vim'
Plug 'CodeFalling/fcitx-vim-osx'
Plug 'rhysd/nyaovim-markdown-preview'

" 文件导航
Plug 'scrooloose/nerdtree'
Plug 'ctrlpvim/ctrlp.vim'
Plug 'wesleyche/SrcExpl'
Plug 'majutsushi/tagbar'
Plug 'taglist.vim'
Plug 'rizzatti/dash.vim'

" c/c++
Plug 'c.vim' "说明一下版本问题

" java
Plug 'artur-shaik/vim-javacomplete2'

" html/css/javascript
Plug 'mattn/emmet-vim'

" scala
Plug 'ensime/ensime-vim'
Plug 'derekwyatt/vim-scala'
"Plug 'ktvoelker/sbt-vim'
"Plug 'megaannum/vimside'

" clojure
Plug 'guns/vim-clojure-static'
Plug 'vim-fireplace'
Plug 'tpope/vim-pathogen'
Plug 'tpope/vim-salve'
Plug 'tpope/vim-projectionist'
Plug 'tpope/vim-dispatch'
Plug 'kien/rainbow_parentheses.vim' "显示彩色的括号, 在lisp家族语言中使用很方便

call plug#end()

"inoremap <silent><expr> <Tab> pumvisible() ? "\<C-n>" : deoplete#mappings#manual_complete()

set hlsearch        "高亮搜索
set incsearch       "在输入要搜索的文字时，实时匹配
syntax on
set mouse=a                    " 在任何模式下启用鼠标
set backspace=2                " 设置退格键可用
au BufReadPost * if line("'\"") > 1 && line("'\"") <= line("$") | exe "normal! g'\"" | endif
autocmd BufWritePost *.scala :EnTypeCheck
autocmd FileType java setlocal omnifunc=javacomplete#Complete
let $NVIM_TUI_ENABLE_TRUE_COLOR=1

" =============================================================================
"                          << 以下为用户自定义配置 >>
" =============================================================================

" 以下为要安装或更新的插件，不同仓库都有（具体书写规范请参考帮助）

" -----------------------------------------------------------------------------
"  < 编码配置 >
" -----------------------------------------------------------------------------
" 注：使用utf-8格式后，软件与程序源码、文件路径不能有中文，否则报错 文件格式，默认 ffs=dos,unix
set encoding=utf-8                                    "设置gvim内部编码，默认不更改
set fileencoding=utf-8                                "设置当前文件编码，可以更改，如：gbk（同cp936）
set fileencodings=ucs-bom,utf-8,gbk,cp936,latin-1     "设置支持打开的文件的编码
set fileformat=unix                                   "设置新（当前）文件的<EOL>格式，可以更改，如：dos（windows系统常用）
set fileformats=unix,dos,mac                          "给出文件的<EOL>格式类型

" -----------------------------------------------------------------------------
"  < 编写文件时的配置 >
" -----------------------------------------------------------------------------
filetype on                                           "启用文件类型侦测
filetype plugin on                                    "针对不同的文件类型加载对应的插件
filetype plugin indent on                             "启用缩进
set smartindent                                       "启用智能对齐方式
set expandtab                                         "将Tab键转换为空格
set tabstop=2                                         "设置Tab键的宽度，可以更改，如：宽度为2
set shiftwidth=2                                      "换行时自动缩进宽度，可更改（宽度同tabstop）
set smarttab                                          "指定按一次backspace就删除shiftwidth宽度
set foldenable                                        "启用折叠
set foldmethod=indent                                 "indent 折叠方式
" set foldmethod=marker                               "marker 折叠方式
set ignorecase                                        "搜索模式里忽略大小写
set smartcase                                         "如果搜索模式包含大写字符，不使用 'ignorecase' 选项，只有在输入搜索模式并且打开 'ignorecase' 选项时才会使用
" set noincsearch                                     "在输入要搜索的文字时，取消实时匹配
set autoread                                          "当文件在外部被修改，自动更新该文件



" 常规模式下用空格键来开关光标行所在折叠（注：zR 展开所有折叠，zM 关闭所有折叠）
nnoremap <space> @=((foldclosed(line('.')) < 0) ? 'zc' : 'zo')<CR>

" 常规模式下输入 cS 清除行尾空格,输入cM 清除行尾^M 符号
nmap cS :%s/\s\+$//g<CR>:noh<CR>
nmap cM :%s/\r$//g<CR>:noh<CR>

" 快捷键<leader>o 来用系统自带软件打开当前文件, 只适用于mac
nmap <leader>o :!open %<cr>
imap <leader>o :!open %<cr>

" 快捷键ii用来在常规模式下直接进入普通模式
imap ii <Esc>

tnoremap <Esc> <C-\><C-n> 
nmap t<Enter> :bo sp term://zsh\|resize 5<CR>i

" 启用每行超过80列的字符提示（字体变蓝并加下划线）
"au BufWinEnter * let w:m2=matchadd('Underlined', '\%>' . 80 . 'v.\+', -1)

" -----------------------------------------------------------------------------
"  < 界面配置 >
" -----------------------------------------------------------------------------
set number                                            "显示行号
set laststatus=2                                      "启用状态栏信息
" set cmdheight=2                                       "设置命令行的高度为2，默认为1
set cursorline                                        "突出显示当前行
set nowrap                                            "设置不自动换行
"set shortmess=atI                                     "去掉欢迎界面

" 设置代码配色方案
set nu
set t_Co=256
" let g:solarized_termcolors=256
set background=dark
" colorscheme solarized
colorscheme gruvbox


" -----------------------------------------------------------------------------
"  < 其它配置 >
" -----------------------------------------------------------------------------
set writebackup                             "保存文件前建立备份，保存成功后删除该备份
set nobackup                                "设置无备份文件
"set noswapfile                              "设置无临时文件
" set vb t_vb=                                "关闭提示音

" =============================================================================
"                          << 以下为常用插件配置 >>
" =============================================================================

" -----------------------------------------------------------------------------
"  < auto-pairs 插件配置 >
" -----------------------------------------------------------------------------
" 用于括号与引号自动补全

" -----------------------------------------------------------------------------
"  < ccvext.vim 插件配置 >
" -----------------------------------------------------------------------------
" 用于对指定文件自动生成tags与cscope文件并连接
" 如果是Windows系统, 则生成的文件在源文件所在盘符根目录的.symbs目录下(如: X:\.symbs\)
" 如果是Linux系统, 则生成的文件在~/.symbs/目录下
" 具体用法可参考www.vim.org中此插件的说明
" <Leader>sy 自动生成tags与cscope文件并连接
" <Leader>sc 连接已存在的tags与cscope文件

" -----------------------------------------------------------------------------
"  < ctrlp.vim 插件配置 >
" -----------------------------------------------------------------------------
" 一个全路径模糊文件，缓冲区，最近最多使用，... 检索插件；详细帮助见 :h ctrlp
" 常规模式下输入：Ctrl + p 调用插件

" -----------------------------------------------------------------------------
"  < EasyAlign 插件配置 >
" -----------------------------------------------------------------------------
" 一个强大的工具, 用来做代码对齐
xmap ga <Plug>(EasyAlign)
nmap ga <Plug>(EasyAlign)

" -----------------------------------------------------------------------------
"  < emmet-vim（前身为Zen coding） 插件配置 >
" -----------------------------------------------------------------------------
" HTML/CSS代码快速编写神器，详细帮助见 :h emmet.txt

" -----------------------------------------------------------------------------
"  < indentLine 插件配置 >
" -----------------------------------------------------------------------------
" 用于显示对齐线，与 indent_guides 在显示方式上不同，根据自己喜好选择了
" 在终端上会有屏幕刷新的问题，这个问题能解决有更好了
" 开启/关闭对齐线
nmap <leader>il :IndentLinesToggle<CR>

" 设置终端对齐线颜色，如果不喜欢可以将其注释掉采用默认颜色
let g:indentLine_color_term = 239

" -----------------------------------------------------------------------------
"  < Mark--Karkat（也就是 Mark） 插件配置 >
" -----------------------------------------------------------------------------
" 给不同的单词高亮，表明不同的变量时很有用，详细帮助见 :h mark.txt

" 可用t<k,j,h,l>切换到上下左右的窗口中去
noremap tk <c-w>k
noremap tj <c-w>j
noremap th <c-w>h
noremap tl <c-w>l

" -----------------------------------------------------------------------------
"  < markdown_preview(NyaoVim)插件配置 >
" -----------------------------------------------------------------------------
let g:markdown_preview_eager=1

" -----------------------------------------------------------------------------
"  < nerdcommenter 插件配置 >
" -----------------------------------------------------------------------------
" 我主要用于C/C++代码注释(其它的也行)
" 以下为插件默认快捷键，其中的说明是以C/C++为例的，其它语言类似
" <Leader>ci 以每行一个 /* */ 注释选中行(选中区域所在行)，再输入则取消注释
" <Leader>cm 以一个 /* */ 注释选中行(选中区域所在行)，再输入则称重复注释
" <Leader>cc 以每行一个 /* */ 注释选中行或区域，再输入则称重复注释
" <Leader>cu 取消选中区域(行)的注释，选中区域(行)内至少有一个 /* */
" <Leader>ca 在/*...*/与//这两种注释方式中切换（其它语言可能不一样了）
" <Leader>cA 行尾注释
let NERDSpaceDelims = 1                     "在左注释符之后，右注释符之前留有空格

" -----------------------------------------------------------------------------
"  < nerdtree 插件配置 >
" -----------------------------------------------------------------------------
" 有目录村结构的文件浏览插件

" 常规模式下输入 F2 调用插件
nmap <F4> :NERDTreeToggle<CR>

" -----------------------------------------------------------------------------
"  < airline 插件配置 >
" -----------------------------------------------------------------------------
" 状态栏插件，更好的状态栏效果
let g:airline#extensions#tabline#enabled = 1
if !exists('g:airline_symbols')
  let g:airline_symbols = {}
endif
let g:airline_left_sep = ''
let g:airline_left_alt_sep = ''
let g:airline_right_sep = ''
let g:airline_right_alt_sep = ''
let g:airline_symbols.branch = ''
let g:airline_symbols.readonly = ''
let g:airline_symbols.linenr = ''
let g:airline_symbols.crypt = '🔒'
let g:airline_symbols.paste = 'ρ'
" let g:airline_symbols.paste = 'Þ'
" let g:airline_symbols.paste = '∥'
let g:airline_symbols.notexists = '∄'
let g:airline_symbols.whitespace = 'Ξ'

let g:airline#extensions#tabline#buffer_idx_mode = 1
nmap t1 <Plug>AirlineSelectTab1
nmap t2 <Plug>AirlineSelectTab2
nmap t3 <Plug>AirlineSelectTab3
nmap t4 <Plug>AirlineSelectTab4
nmap t5 <Plug>AirlineSelectTab5
nmap t6 <Plug>AirlineSelectTab6
nmap t7 <Plug>AirlineSelectTab7
nmap t8 <Plug>AirlineSelectTab8
nmap t9 <Plug>AirlineSelectTab9
nmap t[ <Plug>AirlineSelectPrevTab
nmap t] <Plug>AirlineSelectNextTab

" -----------------------------------------------------------------------------
"  < repeat 插件配置 >
" -----------------------------------------------------------------------------
" 主要用"."命令来重复上次插件使用的命令

" -----------------------------------------------------------------------------
"  < SrcExpl 插件配置 >
" -----------------------------------------------------------------------------
nmap <F8> :SrcExplToggle<CR>

" -----------------------------------------------------------------------------
"  < surround 插件配置 >
" -----------------------------------------------------------------------------
" 快速给单词/句子两边增加符号（包括html标签），缺点是不能用"."来重复命令
" 不过 repeat 插件可以解决这个问题，详细帮助见 :h surround.txt

" -----------------------------------------------------------------------------
"  < Syntastic 插件配置 >
" -----------------------------------------------------------------------------
" 用于保存文件时查检语法
let g:syntastic_cpp_compiler = "g++"
let g:syntastic_cpp_compiler_options = ' -std=c++11'

" -----------------------------------------------------------------------------
"  < Tagbar 插件配置 >
" -----------------------------------------------------------------------------
" 相对 TagList 能更好的支持面向对象

" 常规模式下输入 tb 调用插件，如果有打开 TagList 窗口则先将其关闭
nmap tb :TlistClose<CR>:TagbarToggle<CR>

let g:tagbar_width=30                       "设置窗口宽度
" let g:tagbar_left=1                         "在左侧窗口中显示

" -----------------------------------------------------------------------------
"  < TagList 插件配置 >
" -----------------------------------------------------------------------------
" 高效地浏览源码, 其功能就像vc中的workpace
" 那里面列出了当前文件中的所有宏,全局变量, 函数名等

" 常规模式下输入 tl 调用插件，如果有打开 Tagbar 窗口则先将其关闭
nmap tl :TagbarClose<CR>:Tlist<CR>

let Tlist_Show_One_File=1                   "只显示当前文件的tags
" let Tlist_Enable_Fold_Column=0              "使taglist插件不显示左边的折叠行
let Tlist_Exit_OnlyWindow=1                 "如果Taglist窗口是最后一个窗口则退出Vim
let Tlist_File_Fold_Auto_Close=1            "自动折叠
let Tlist_WinWidth=30                       "设置窗口宽度
let Tlist_Use_Right_Window=1                "在右侧窗口中显示

" -----------------------------------------------------------------------------
"  < YouCompleteMe 插件配置 >
" -----------------------------------------------------------------------------
"let g:ycm_key_invoke_completion = '<space>'
let g:ycm_confirm_extra_conf=0
"let g:ycm_python_binary_path = '/usr/bin/python'
let g:ycm_python_binary_path = '/Library/Frameworks/Python.framework/Versions/3.5/bin/python3'
let g:ycm_server_python_interpreter = '/Library/Frameworks/Python.framework/Versions/3.5/bin/python3'
nnoremap <leader>jd :YcmCompleter GoTo<CR>

" -----------------------------------------------------------------------------
"  < cscope 工具配置 >
" -----------------------------------------------------------------------------
" 用Cscope自己的话说 - "你可以把它当做是超过频的ctags"
if has("cscope")
    "设定可以使用 quickfix 窗口来查看 cscope 结果
    set cscopequickfix=s-,c-,d-,i-,t-,e-
    "使支持用 Ctrl+]  和 Ctrl+t 快捷键在代码间跳转
    set cscopetag
    "如果你想反向搜索顺序设置为1
    set csto=0
    "在当前目录中添加任何数据库
    if filereadable("cscope.out")
        cs add cscope.out
    "否则添加数据库环境中所指出的
    elseif $CSCOPE_DB != ""
        cs add $CSCOPE_DB
    endif
    set cscopeverbose
    "快捷键设置
    nmap <C-\>s :cs find s <C-R>=expand("<cword>")<CR><CR>
    nmap <C-\>g :cs find g <C-R>=expand("<cword>")<CR><CR>
    nmap <C-\>c :cs find c <C-R>=expand("<cword>")<CR><CR>
    nmap <C-\>t :cs find t <C-R>=expand("<cword>")<CR><CR>
    nmap <C-\>e :cs find e <C-R>=expand("<cword>")<CR><CR>
    nmap <C-\>f :cs find f <C-R>=expand("<cfile>")<CR><CR>
    nmap <C-\>i :cs find i ^<C-R>=expand("<cfile>")<CR>$<CR>
    nmap <C-\>d :cs find d <C-R>=expand("<cword>")<CR><CR>
endif

" -----------------------------------------------------------------------------
"  < ctags 工具配置 >
" -----------------------------------------------------------------------------
" 对浏览代码非常的方便,可以在函数,变量之间跳转等
set tags=./tags;                            "向上级目录递归查找tags文件（好像只有在Windows下才有用）


" =============================================================================
"                          << 以下为常用自动命令配置 >>
" =============================================================================

" 自动切换目录为当前编辑文件所在目录
au BufRead,BufNewFile,BufEnter \@!(term://)* cd %:p:h


```
