---
layout: post
title: Xcode Server Integration
date: '2015-05-19 14:57:07'
permalink: "/xcode-server-integration/"
tags:
- xcode
---

Apple 在 Developer Center 里提供了 OS X Server 的 DMG 文件免费下载，此举措也在鼓励开发者使用 OS X Server 对自己的项目进行持续集成。相比 [Jenkins CI](https://jenkins-ci.org)，Xcode Server 的 UI 就好太多太多了，对于我这种颜控当然是没有什么抵抗力的。

网上对 Xcode Server 的资料不少，但是都停留在比较老的版本上，所以对于最新的版本支持并不好，有一些 Pre-Integration Script 和 Post-Integration Script 都不工作。

接下来让我们进入主题，假设我们有一个叫 [ditto](https://itunes.apple.com/cn/app/ditto-shi-pin-you-zhi-duan/id979919086?l=en&mt=8) 的 iOS/Cocoa 项目要进行持续集成。

基本的步骤大致参考了 Cee 的这篇 [Blog](https://blog.cee.moe/ios-integration/)，并根据自己的记忆进行了补充:

1. 安装 OS X Server，并启用 Xcode Server，设置可以创建 Bot 的权限
2. 对 Xcode Server 执行 Build 的用户 \_xcsbuildd 添加一对 SSH Key：
  - Login：sudo -u _xcsbuildd /bin/bash
  - Generate SSH Key：ssh-keygen -t rsa -C "$your_email"
  - Show your public key：cat /var/_xcsbuildd/.ssh/id_rsa.pub
  - Put the key to GitHub or GitCafe
  - SSH your server to accept the fingerprint：ssh -T git@git.domain.com
3. 配置 Workspace，选择需要创建 Bot 的 Build Scheme 并设置为 Shared
4. 选择 Product 下的 Create Bot，同时设置相关的集成参数

如果你的项目没有使用其他第三方库管理工具或者 git submodule 的话，那么你的项目的 Bot 已经设置完毕了，如果有使用这些东西的话，需要设置一下 Pre-Integration Script。

下面是我的使用的 Script，仅给出了使用了 CocoaPods 的：

```
export LC_ALL="en_US.UTF-8"
cd ditto # 这里换成你的项目名字
# 如果没有使用 git submodule 请删除下面这行的内容
git submodule update --init --recursive
# 如果没有使用 CocoaPods，请删除下面这块的内容
if [ ! -e "$HOME/.cocoapods/repos" ]
then
pod setup
fi
pod install 
```

需要注意的是，如果你没有安装 CocoaPods，那么请执行 `sudo gem install cocoapods`。我不确定 rbenv 下的 CocoaPods 能否在 _xcsbuildd 用户下执行，所以我这里使用的是系统的 gem。

通常关于持续集成的文章到这里就结束了，但是我们还遗漏了一个非常重要的部分，那就是 Post-Integration Script。

这里我首先给大家推荐一个非常棒的 [Repo](https://github.com/tibo/XcodeServer)。作者对 Xcode Server 进行了非常深入的研究，我的 Script 也是基于他的启发。

假设我们有一个每天执行的 Bot，我们想把每天执行 Bot 生成的可执行文件上传到 Dropbox 分享给团队里的成员使用，同时我们希望 Bot 将持续集成的结果不仅仅通过邮件的方式发送给某些成员，还想在 Slack 频道上通知大家。

```
PRODUCT_NAME="ditto" # 这里填写你的项目的名字
EXECUTABLE_NAME="ditto" # 这里填写你的项目的名字，如果你的可执行文件和项目名不一致的话，该成可执行文件的名字
# Remember to Add Administrator Group to DROPBOX_DIR and Dropbox Folder first!
DROPBOX_DIR="/Users/Yifei/Dropbox/ditto" # 这里换成你的 Dropbox 的文件夹
# Copy Archive to
ARCHIVE="${XCS_ARCHIVE}"
TARGET="/tmp/${PRODUCT_NAME}_${XCS_INTEGRATION_NUMBER}.xcarchive"
/bin/cp -Rp "${ARCHIVE}" "${TARGET}"
/bin/cp -Rp "${TARGET}/Products/Applications/${EXECUTABLE_NAME}.app" "${DROPBOX_DIR}/${PRODUCT_NAME}_${XCS_INTEGRATION_NUMBER}.app"
/bin/ln -Fs "${DROPBOX_DIR}/${PRODUCT_NAME}_${XCS_INTEGRATION_NUMBER}.app" "${DROPBOX_DIR}/${PRODUCT_NAME}_latest.app"
# Clean Up
/bin/rm -rf "${TARGET}"
# Push Notification to Slack
PAYLOAD="{\"channel\": \"#your-channel\", \"username\": \"Xcode-Server\", \"text\": \"${XCS_BOT_NAME} finished with status: ${XCS_INTEGRATION_RESULT}\", \"icon_emoji\": \":ghost:\"}"
/usr/bin/curl -X POST --data-urlencode "payload=${PAYLOAD}" "YOUR_HOOK_URL"
```

这样，每次持续集成完毕，我们都会将 Archive 中的可执行文件提取出来，存放到我们的 Dropbox，并创建一个 `${PRODUCT_NAME}_latest.app` 的软链接指向最新的 Build 文件。

通过在 Pre-Integration Script 和 Post-Integration Script 中添加 `set`，我们可以得到更多的环境变量信息，也可以利用这些资源编写出属于自己的持续集成脚本，希望我这篇文章能够对你有所启发。

另外如果你对 Xcode Server 集成过程有兴趣的话，不妨读一读 [Under the Hood of Xcode Server](http://honzadvorsky.com/blog/2015/5/4/under-the-hood-of-xcode-server)