---
layout: post
title: GnuPG 的三个使用场景
excerpt: "使用 GPG 工具来验证下载内容、给 GitHub commit 签名、加密 Facebook 的邮件通知"
categories: articles
tags: [Unix-like]
comments: true
share: true
---

以前写过一篇文章，详细介绍了 GnuPG，以及如何使用 GPG 工具给邮件签名和加密。考虑到实际上你的朋友们并不会和你一样使用 GPG，这些基本就成了屠龙之技。

好在 GPG 还是有其他用处的。

## 验证下载内容

不管是政策法规原因，还是运营商出国带宽原因，反正现在要去国外的软件官网下点东西，是越来越难了。如果去国内第三方下载，怎么保证下到的文件没有经过篡改呢？毕竟，之前闹得沸沸扬扬的 XcodeGhost 事件就是前车之鉴啊。

有的网站提供 MD5 验证。通过对比本地使用 `md5sum` 计算得到的校验和与官网提供的校验和，来验证文件完整性。但是 MD5 可以构造冲突，发动所谓碰撞攻击 *collision attack*，也就是说，被篡改的文件明明和原文件不一样，但是可以构造出相同的 MD5 校验和。

又有网站提供 SHA256 验证。使用方法差不多，抵御 collision attack 的能力比 MD5 要强。如果是在 TLS/SSL 加密过的官网（你得好好确认）获得的 SHA256 校验和的话，那么这种方法还是具有一定可靠性的。

为什么要强调 TLS/SSL 呢？因为没有签名的网页给出的校验和本身有可能被篡改啊。某些第三方的镜像，同时还镜像了官方的未签名的校验和，那就简直搞笑了：第三方如果真的篡改了文件，只要相应修改镜像内的校验和即可，用户拿着它在本地校验纯属浪费时间。

所以为了验证文件完整性，一方面需要保证哈希算法的 collision resistance，另一方面还要保证官方给出的校验和正确送达你手中。防不胜防啊。

第二种需求就是 GPG 的典型应用场景——邮件签名嘛。官方公开一份用自己的私钥签名了的文件，其中包含了正确的 SHA256 校验和，无论用户在哪里得到了它，都可以验证内容是否由官方发出，是否被篡改。只要确保拥有官方的公钥，无论在哪里获得的软件，无论在哪里获得的校验和，都可以放心地用 `sha256sum` 验证对比后使用。

怎么获得对方公钥呢？你得知道对方的 key ID，然后从公钥服务器上获取。

```shell
gpg --keyserver hkp://keys.gnupg.net --recv-key 7D8D0BF6
```

`7D8D0BF6` 是 [Kali Linux](https://www.kali.org) 发布软件用的钥匙，其文档中有详细的校验步骤，本文就是参考文档写成。

话说回来，你还是得知道对方的 key ID。不过 key ID 很短，比较容易获得和记忆，而且钥匙一旦获得，会保存在本地，下次使用就不需要再次获取了。

查看 fingerprint，确认公钥正确导入：

```shell
gpg --fingerprint 7D8D0BF6
```

输出差不多是这样：

```shell
pub   4096R/7D8D0BF6 2012-03-05 [expires: 2018-02-02]
      Key fingerprint = 44C6 513A 8E4F B3D3 0875  F758 ED44 4FF0 7D8D 0BF6
uid                  Kali Linux Repository <devel@kali.org>
sub   4096R/FC0D0DCB 2012-03-05 [expires: 2018-02-02]
```

和软件一起下载到的 `SHA256SUMS` 和 `SHA256SUMS.gpg`，前者是校验和，后者是对前者的签名，用公钥对签名进行验证。

```shell
gpg --verify SHA256SUMS.gpg SHA256SUMS
```

确保看到 *Good signature* 的提示，否则就是验证失败了。

官方给出的校验和没问题，接下来就看与本地算出的校验和是不是一致了。

你当然可以算出后进行肉眼比对，但是下面的命令更方便直观：

```shell
grep kali-linux-2016.2-amd64.iso SHA256SUMS | shasum -a 256 -c
```

如果不出意外，结果差不多是这样：

```shell
kali-linux-2016.2-amd64.iso: OK
```

看到 *OK*，就是验证成功了。

第一次导入公钥成功后，以后每次验证只需要两条命令，算是比较方便的了。

ubuntu 也使用了同样的验证方式。CentOS 验证方式略有不同，它提供了一个同时包含校验和与签名的 *sha256sum.txt.asc*，可以这样校验签名：

```shell
gpg --verify sha256sum.txt.asc
```

如果在 Mac 上装了 GPG Suite 的话，双击 `.asc` 文件就可以完成校验，导入公钥都省了。

把 `.asc` 和 `.iso` 放在同一目录下，可以用以下命令验证校验和：

```shell
sha256sum -c sha256sum.txt.asc 2>&1 | grep OK
```

## 为 GitHub 每一次提交签名

仍然是保证提交完整性的一个应用场景。先在 GitHub 账户中上传你的公钥，随后对每次提交，在本地用私钥签名，当 `git push` 到 GitHub 后，服务器会验证这次提交是不是由你完成的。

服务器端，在 GitHub 的 **Settings** -> **SSH and GPG keys** 里，添加你的 GPG 公钥。

在本地，首先告知 `git` 钥匙信息。用 `gpg --list-secret-keys --keyid-format LONG` 命令，查询私钥 ID。

```shell
$ gpg --list-secret-keys --keyid-format LONG
/Users/hubot/.gnupg/secring.gpg
------------------------------------
sec   4096R/3AA5C34371567BD2 2016-03-10 [expires: 2017-03-10]
uid                          Hubot 
ssb   4096R/42B317FD4BA89E7A 2016-03-10
```

像上面那样， `3AA5C34371567BD2` 就是私钥的 key ID。

告知 `git`，

```shell
git config --global user.signingkey 3AA5C34371567BD2
```

如果用的不是 GPG Suite，还要设置环境变量，

```shell
echo 'export GPG_TTY=$(tty)' >> ~/.bash_profile
```

GPG 钥匙对应的邮件地址必须和 `git` 提交的邮件地址一致，否则看[这里](https://help.github.com/articles/associating-an-email-with-your-gpg-key/)。

提交的时候，加参数 `-S` 可签名。

```shell
git commit -S -m 'your commit message'
```

如果要默认签名，可以设置

```shell
git config commit.gpgsign true
```

如果希望本机所有提交都签名，给上面的命令加 `—global` 参数即可。

## 加密 Facebook 邮件通知

Facebook 为用户提供上传 GPG 公钥服务。点击**设置**，选择**安全与登录**，在页面最下方上传。上传后，既可以打开加密邮件通知，也可以在个人页面公开，以便和你的 geek 朋友们通信。设置完成后，需要点击邮件链接确认。注意，如果打开加密，账户找回通知也是加密发送的。要是哪天重装系统把私钥给弄丢了，赶紧关掉加密或者换掉公钥吧。



## 参考资料

* [Verifying Your Downloaded Kali Image](https://docs.kali.org/introduction/download-official-kali-linux-images)
* [How to Verify you are Getting CentOS Linux Images, ISOs, or Packages](https://wiki.centos.org/Download/Verify)
* [Signing commits with GPG](https://help.github.com/articles/signing-commits-with-gpg/)