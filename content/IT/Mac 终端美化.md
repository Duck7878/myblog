---
title: "Mac 终端美化"
date: 2026-10-30
description: ""
hiddenInHomeList: false
categories:
  - IT
tags:
  - IT
---

# Mac 终端美化

Review: 2026-10-30 17:21:00

# 查看当前使用的 Shell

```bash
echo $SHELL
```

# 查看已安装的 Shell

```bash
cat /etc/shells
```

# 使用 Homebrew 安装/更新 Zsh

```bash
brew install zsh
```

切换默认 Shell：

```bash
chsh -s /bin/zsh
```

重启终端即可使用 Zsh。

---

Oh My Zsh

执行官方安装脚本：

```bash
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

如果失败，可以先使用 Git 下载：

```bash
git clone git://github.com/robbyrussell/oh-my-zsh.git ~/.oh-my-zsh
```

然后复制默认配置文件：

```bash
cp ~/.oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc
```

---

主题配置

打开配置文件：

```bash
vim ~/.zshrc
```

配置项：

```bash
ZSH_THEME="主题名称"
```

Oh My Zsh 提供大量主题可供选择。

如果设置为：

```bash
ZSH_THEME="random"
```

则每次打开终端都会随机使用一种主题。

修改完成后加载配置：

```bash
source ~/.zshrc
```

---

插件安装

# 自动提示插件（zsh-autosuggestions）

```bash
git clone git://github.com/zsh-users/zsh-autosuggestions \
$ZSH_CUSTOM/plugins/zsh-autosuggestions
```

# 语法高亮插件（zsh-syntax-highlighting）

```bash
git clone git://github.com/zsh-users/zsh-syntax-highlighting \
$ZSH_CUSTOM/plugins/zsh-syntax-highlighting
```

如果 `git://github.com` 无法连接，可以改成：

```bash
https://github.com
```

下载完成后，在 `~/.zshrc` 中配置：

```bash
plugins=(
    zsh-autosuggestions
    zsh-syntax-highlighting
)
```

重新加载配置：

```bash
source ~/.zshrc
```

---

解决每次打开终端提示大量内容

编辑配置文件：

```bash
vi ~/.zshrc
```

在顶部加入：

```bash
ZSH_DISABLE_COMPFIX=true
```

保存后执行：

```bash
source ~/.zshrc
```

‍
