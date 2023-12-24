---
title: 怎么给同一台电脑配置不同的Git账号
date: 2023-05-10 21:32:50
tags:
---

最近搞了台新电脑，忽然发现有针对项目区分不同的 Git 地址和 Git 账号的需求。再跟 GPT 大哥几经周旋之后，得到了能用的答案。记录一下。

---

首先我把某个账号的项目放到了 ～/code/A 文件夹里。之后把另外一个账号的的项目放到了 ～/code/B 里面。这得配置需要分别配置不同的ssh密钥

1.  **生成 SSH 密钥对**： 在终端中运行以下命令，分别为 GitHub 和内网 Git 创建 SSH 密钥对。确保为每个密钥对选择一个不同的文件名。

```
ssh-keygen -t rsa -b 4096 -C "your_email@example.com" -f ~/.ssh/id_rsa_github 
ssh-keygen -t rsa -b 4096 -C "your_email@example.com" -f ~/.ssh/id_rsa_intranet_git
```

2.  **添加私钥到 SSH agent**： 首先确保 ssh-agent 在后台运行，在把两个私钥添加进 SSH agent。

```
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa_github ssh-add ~/.ssh/id_rsa_intranet_git
```

3. **ssh配置文件**

```
vim  ~/.ssh/config

## 打开 vim 之后编译文件内容
# GitHub
Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_rsa_github
  IdentitiesOnly yes

# Intranet Git
Host [hostname]
  HostName [hostname]
  User git
  IdentityFile ~/.ssh/id_rsa_intranet_git
  IdentitiesOnly yes
```

4. **在对应的的git服务器里添加对应的ssh密钥**

```
cat ~/.ssh/id_rsa_github.pub
cat ~/.ssh/id_rsa_intranet_git.pub
```

5. **验证链接**

```
ssh -T git@github.com 
ssh -T git@[hostname]
```

6. **之后因为git要用的昵称和邮箱也不一样，所以昵称和邮箱也得按文件夹区分开。导航到包含多个项目的父目录。**

```
 cd /path/to/your/parent_directory
```

7. **在父目录中创建一个名为 `.gitconfig` 的文件。使用文本编辑器打开 `.gitconfig` 文件并添加以下内容。将 `Your Name` 和 `your_email@example.com` 替换为您的用户名和电子邮件。**

```
vim .gitconfig

## 在文件中添加以下内容
[user]     
name = Your Name     
email = your_email@example.com
```

8. **对于父目录中的每个项目，您需要更新它们的 Git 配置以包含该父目录的 `.gitconfig` 文件。执行以下命令：**

```
cd /path/to/your/project 
git config --local include.path /path/to/your/parent_directory/.gitconfig
```

9. **之后配置全局git邮箱和账号**
    
```
git config --global user.name "Global Name"
git config --global user.email "global_email@example.com"
```

10. **在总.gitconfig(~/.gitconfig)文件下增加路由配置**

```
[includeIf "gitdir:/path/to/your/parent_directory/"]
    path = /path/to/your/parent_directory/.gitconfig
```

11. **检查是否生效**

```
git config --get user.name
git config --get user.email
```

如果在工作文件夹的项目里跟其他地方的用户名和邮箱不一致，则配置成功
