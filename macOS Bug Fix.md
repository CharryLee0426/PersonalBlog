# macOS Bug Fix
## 0. Brief Introduction
Although there are many new features in macOS 12 and I love them very much such as live text and system translation, I have to say there are also too manu bugs that Apple really needs to fix them.

## 1. Safari
第一条该骂的就是，Safari 浏览器，那个浏览器实在是拉的离谱。更改到官方推荐的新特性以后会特别卡顿，被迫选回未升级的配置。希望能在下几个版本中修复这些 Bug。

## 2. Simplified Chinese IME
中文输入法，在开机大概一周之后就会莫名其妙的卡顿，我从官网上已经得知貌似这就是一个 Bug，按照客服的说法，关机后按着 command + option + p + r 开机，这样的话就会解决这个问题。我可以实验一下，如果没好的话，直接开骂好吧。我发现一条有用的解决方法，可以暂时性的解决问题。重启以后会暂时解决卡顿问题，那么重启简体中文输入法相关进程应该会解决问题，所以在终端中输入下面命令杀掉和简体中文输入法相关的进程。

```bash
pkill -f SCIM.app
```

## 3. Terminal
我又遇到了一个 macOS 的恶性 bug，这个 bug 是关于终端（Terminal）的。复现过程是：将一个项目做大量行的改动，然后提交 commit，commit 的 -m 的信息要写中文，这时就会触发第一次终端崩溃。然后，点击“重新打开”可以再次进入，但是输入一些字符后再次崩溃。再次打开时就会意外退出。之后重新启动也无效，而且重新启动会被卡住，毫无反应。最后必须借助长按电源键才能强行关闭，再次打开也无效。最后解决办法是，在反复重新打开时有可能会进入终端一段时间（依旧无响应），趁这段时间，长按图片将终端强行退出，此时会弹出和不做任何反应退出的退出界面不一样的提示界面，再该界面重新打开，即可再次打开终端。之后关闭所有程序，再次重启，即可解决问题。macOS 12 的 bug 真的好多，不想吐槽了。升到最后一个系统以后果断弃升，稳定为先！

## 4. TracePad Not Work
触摸板在工作一段时间后三指和四指手势会失效，但是其他手势可以正常工作。按照 Apple Support 论坛的“达摩院小智丈”说法，主要是 iTerm2 的问题，原回答如下：
> *经过这段时间的使用，发现可能与第三方软件 iTerm2 有关系。卸载后的一周时间 ，没有再出现同样的问题。还有就问题而言其实不是触摸板失效了，是调度中心与启动台失效了，在失效的时候，就算是鼠标去点击调度中心和启动台也是没有响应的。因为恰好对应触摸板的三指四指操作，所以我误以为是触摸板的问题。可能因为 iTerm2 导致 Dock 进程卡死了，其他第三方的软件也可能导致此类情况的发生。请各位知悉。*

按照 ta 的说法，这个问题可以通过强制结束 Dock 进程来重启 Dock 进程来解决，进入 Terminal 中执行以下命令来解决。

```bash
killall Dock
```

***To be continued...***