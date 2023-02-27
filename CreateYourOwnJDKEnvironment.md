# How to Create Your Own JDK Environment

## 0. Overview
JVM, short for Java Virtual Machine, may be the most important infrasturcture of all time. Not only java, but also many other programming language such as kotlin use JDK as their compile tool. Therefore, all developers who wanna improve their skills to the next level should read JDK, understand JDK, then try to debug it or alternate it! That is the purpose of this article, making a good start point for developers to build their own JDK environment.

## 1. What it is & isn't
What it is
* A guide to build JDK from source code;
* A solution to problems potentially happend when compiling the source code of openjdk.

What it isn't
* A guide to install JDK to your computer;
* A guide similar to official illustration `building.md` in openjdk github repo.

## 2. Who does this guide for
I wrote this article not for myself, although only me focus on this repo. I hope it will be helpful for:

* Java developers who wanna improve themselves;
* Researchers who are interested in compiler design;
* Students who don't have the experience to get a programming language from source code

## 3. My Computer Introduction
This guide is all re-implemented many times on my computer. And I can make sure it is executable.

About My Computer:

| Component                | Description                                                  |
| ------------------------ | ------------------------------------------------------------ |
| Name                     | Apple MacBook Pro                                            |
| CPU                      | M1                                                           |
| RAM                      | 16 GB                                                        |
| Operating system         | macOS 13.2.1 (Ventura)                                       |
| Xcode Command Line Tools | 2396                                                         |
| Clang Compiler Version   | Apple clang version 14.0.0                                   |
| Boot JDK Version         | openjdk version "17.0.5" 2022-10-18 LTS<br/>OpenJDK Runtime Environment Zulu17.38+21-CA (build 17.0.5+8-LTS)<br/>OpenJDK 64-Bit Server VM Zulu17.38+21-CA (build 17.0.5+8-LTS, mixed mode, sharing) |

So it is very suitable for developers who use new apple silicon macs. We should meet many problems if we use a uncommon architecture for PC.

## 4. Pre-requiries
1. boot jdk, it's best to download the last previous version before jdk which you want to compile.

    for example, you should download jdk 16 as your boot jdk if you want to research jdk 17;

2. Xcode, you should install it for getting all tools although you may be not interested in Apple mobile develop at all;

3. At least 5GB is required;

## 5. Where can I get openjdk source code?
The first difficulty is located at the very beginning. You will find that it is not easy to get source code. Although an official repo is published on GitHub by openjdk group and there are many distributions developed by different companies, you'll find that most of them only provide binary installers. You can find source code in official website but they, openjdk group, doesn't consider compatiability at all. Things get worse when you use Mac on Apple silicon, next generation arm PC SoC. 

Thankfully, I got one which is compatiable with Apple Silicon and shares source code to every developers. It is `JetbrainsRuntime`, a foundation openjdk used by every ides and tools of jetbrains such as Intellij, Pycharm and Clion. You can clone it from its Github repo [HERE](https://github.com/JetBrains/JetBrainsRuntime). It has supported Apple Silicon natively since Apple released its new self-developed SoC. At the meanwhile, it adds a lot of features for rendering text, images, and other contents in oder to improve users' experience. Thus, it will be a better choice if you're interested in those extra features.

## 6. Following `build.md`, and..., get your first error!

You enter into the repo folder after you clone it from github, read `build.md` provided by Jetbrains, execute your first command `sh ./configure`, and then... get lots of error logs!

Don't be upset when you meet error at the beginning. Almost all developers can't build their own jdk at first time without any error. The most common reason for this error may be:

* Some essiential tools aren't installed:

    *  `autoconf`: `brew install autoconf`;
    
    * `command line tool`: install xcode in [developer.apple.com](developer.apple.com);
    
    * `bootstrap jdk`: install at least `n-1` version jdk from mainstream developers such as azul;

* Your computer chooses default command line tool rather than Xcode's command line tools

    * First, use `xcode-select -p` to check which command line tool you selected, the default one is at `/Library/Developer/CommandLineTools`;

    * Second, change your command line tool by using `sudo xcode-select -s /Applications/Xcode.app/Contents/Developer`. password is required;
    
    * Third, check your command line tool again.

Execute `sh ./configure` again, and enjoy the joy you configed your openjdk building environment.

## 7. Make your own iamges, and,,, lots of errors!

Following the official `building.md`, you execute `make images` in order to get your first jdk binary. And you get more errors! Unfortunately, building jdk is much bigger and more difficult task than building golang or other programming languages.

You maybe see lots of familiar errors which say that a function called `sprintf` will be deprecated in the future. And your building process terminates because of them. However, they are, in fact, warnings but not errors.

You can solve it very easily, but must reexecute the first step: make your configure. Add `--disable-warnings-as-errors` to `sh ./configure` then your configure won't see warnings as errors. Although it is not recommended in practice, it is the only way you can get your own openjdk on your M1 Macs.

As matter of fact, it is good to apply a strict mode in building such a big and complicated virtual machine due to robust. It can make your project more stable, less errors.

## 8. Get your own openjdk!
Finally, you build it and get your own openjdk! Your jdk path is `<JetbrainsRuntimePath>/build/macosx-aarch64-server-release/images/jdk`.

## 9. At last...
Last but not least, I have to say only building your own jdk but not editting it is meaningless. You have to make your editting in order to understand how jdk and java work and improve your skills.