---
title: 解决Xcode开发组件失效的命令
tags:
  - xcode
date: 2016-07-30 23:26:17
categories: xcode
---
find ~/Library/Application\ Support/Developer/Shared/Xcode/Plug-ins -name Info.plist -maxdepth 3 | xargs -I{} defaults write {} DVTPlugInCompatibilityUUIDs -array-add `defaults read /Applications/Xcode.app/Contents/Info DVTPlugInCompatibilityUUID`
