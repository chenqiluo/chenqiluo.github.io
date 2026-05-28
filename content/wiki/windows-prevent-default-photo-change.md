---
title: "Windows防止图片默认修改"
date: 2026-05-28
draft: false
tags: ["Windows", "注册表", "默认应用", "照片App", "系统设置"]
categories: ["技术笔记"]
description: "通过注册表加 NoOpenWith 标记或关闭安全软件默认程序保护，彻底防止 Windows 照片 App 反复抢占图片默认打开方式。"
showToc: true
TocOpen: true
---

## 方案 A — 用注册表让"照片"App 不再跳出来抢

如果你的问题是系统自带的"照片"App 反复抢占，可以给它加个隐藏标记让它闭嘴：

1. **Win + R** → 输入 `regedit` 打开注册表编辑器
2. 导航到（路径中版本号会变，先展开看实际名称）：

```text
HKEY_CURRENT_USER\SOFTWARE\Classes\Local Settings\Software\Microsoft\Windows\CurrentVersion\AppModel\Repository\Packages\
```

3. 找到 `Microsoft.Windows.Photos_xxxx__8wekyb3d8bbwe\App\Capabilities\FileAssociations`
4. 记下右栏里 `.jpg` 对应的那个 **ProgID** 值（长得像 `AppXxxxxxxxx...`）
5. 导航到：

```text
HKEY_CURRENT_USER\SOFTWARE\Classes\【刚才那个ProgID】
```

6. 右键右侧空白处 → **新建 → 字符串值** → 命名为 `NoOpenWith`（值留空）
7. 退出注册表 → 重新设一次默认应用

> ⚠️ 操作注册表前建议先点 **文件 → 导出** 备份一下。

---

## 方案 B — 检查是不是安全软件/电脑管家在作妖

如果你装了 **360安全卫士 / 腾讯电脑管家 / 火绒** 等：

1. 进它们的 **"安全设置" → "默认程序保护" / "防篡改"** 类功能
2. **关掉**它"帮你锁定默认浏览器/看图软件"的选项（反而在帮倒忙）
