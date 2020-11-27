---
layout: post
title: Pass密码管理工具
---

pass（全称 password-store）是一个简单的并遵照Unix哲学的密码管理软件。 在 pass 中每一个密码都存放在一个 gpg 加密的文件中，这些文件对应着该密码相关的资源标识。 pass 需要 gpg 配合进行密码文件的加密和解密，所以在了解 pass 的同时还需要了解 gpg 的简单使用。 使用 pass 之后，完全可以实现一站一复杂密码的需求，而自己需要做到的仅仅是 **保存好 pass 使用的 gpg 私钥文件，以及记住该私钥的加密密码**。

## gpg 基本命令

[GPG入门教程](http://www.ruanyifeng.com/blog/2013/07/gpg.html)

```bash
# 生成密钥
gpg --key-gen
# 密钥越长越好；用户标识需要和 pass 中的用户id对应

# 列出密钥
gpg --list-keys

# 删除密钥
gpg --delete-key [用户ID]

# 导出公钥
gpg --armor --output public-key.txt --export [用户ID]

# 导出私钥
gpg --armor --output private-key.txt --export-secret-keys [用户ID]

# 导入私钥
gpg --import [密钥文件]
```

## Pass 命令

[Pass Command](http://git.zx2c4.com/password-store/about/) pass 将所有密码文件保存在 ~/.password-store 文件夹下，并且 pass 支持通过 git 同步管理该文件夹，pass git 命令的使用同 git 使用相同。

```bash
# 初始化，这里的gpg-id对应gpg中的用户id
pass init xxx@gmail.com

# 列出所有密码
pass
pass ls

# 查询
pass find .com
pass search .com

# 显示密码
pass Email/gmail.com

# 插入密码（需要指定密码）
# -m 指定使用多行密码
pass insert [-m] Email/126.com

# 生成并保存密码
# -n 生成字母数字密码
# -c 将生成的密码拷贝到剪贴板
# 最后的数字指定密码长度
pass generate [-n] [-c] Email/gmail.com 12

# 删除密码
pass remove Email/gmail.com

# 一些 pass git 相关命令
pass git init
pass git remote add origin guodong.co:pass-store
pass git push -u --all
```

## 总结

pass 通过 gpg 中生成的密钥来加密密码文件，置于如何确定 pass 使用 gpg 中的哪个密钥文件，取决于 pass init 时指定的 gpg-id 对应于 gpg 中的哪一个uid。 通常情况下，将 pass init 中的 gpg-id 设置为 gpg 密钥信息中的邮箱即可。
