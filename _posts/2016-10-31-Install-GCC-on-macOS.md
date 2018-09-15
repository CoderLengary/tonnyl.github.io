---
layout: post # needs to be post
title: macOS 安装 GCC 教程
summary: 简单几步就可以在 macOS 上成功安装GCC.
featured-img: work
categories: [Note]
---

网上的教程大多是通过安装 Xcode 实现安装 gcc 的，这种方式的花费有点太得不偿失，Xcode占据的空间是在太大，这对于128g用户简直就是噩梦。

还有就是通过homebrew安装，前提是你电脑上要安装了homebrew。

这里介绍一种简单的方法。

1. 到 https://sourceforge.net/projects/hpc/files/hpc/gcc/ 下载所需要的的gcc压缩包。

	![Download zip](https://i.loli.net/2018/03/27/5ab9d6094d953.jpg)

2. 下载完成后，通过 terminal 进入下载目录，即 `gcc-4.8-bin.tar.gz` 所在的目录。输入命令：

	```bash
	gunzip gcc-4.8-bin.tar.gz
	```

	解压 `gcc-4.8-bin.tar.gz` 压缩包，注意只需要解压一次，在同一目录下得到 `gcc-4.8-bin.tar` 文件。

3. 继续在 terminal 中输入命令：
	
	```bash
	sudo tar -xvf gcc-4.8-bin.tar
	```

	输入密码后，自动在当前目录下生成了usr目录。`cd /user/local/bin` 就可以看到所有的 `gcc 相关的命令了。当然你也可以将整个目录移动到你需要的目录。
	
	![Folder](https://i.loli.net/2018/03/27/5ab9d6a4671dd.jpg)

4. 添加 gcc 路径。关掉当前 terminal ，重新新建一个 terminal 。输入如下命令：`touch ～/.bash_profile` 新建一个 bash_profile 文件，使用 vi 或者其他编辑器写入如下代码：`export PATH="/Users/lizhaotailang/Gcc/usr/local/bin:$PATH"`, 当然这是我的路径，具体的路径根据你的实际情况而定。写入完成后关闭文件。为了使文件生效，还需要在 terminal 输入 `source ～／.bash_profile` 。

5. 查看是否生效。在 terminal 中输入：`echo $PATH` ,如果能够在输出中看到刚刚添加的路径就说明添加成功了。或者直接输入 gcc 命令，如果能够看到 gcc 的 clang 错误而不是找不到文件的话，就说明配置成功了。

	![Success](https://i.loli.net/2018/03/27/5ab9d72a352d2.jpg)

6. 然后就可以愉快的使用 gcc 了。