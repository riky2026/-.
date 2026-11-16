# 取景 · FrameGuide

原生 iPhone 相机源码，最低 iOS 17.0，首版拍摄界面固定竖屏。使用 SwiftUI、AVFoundation、Vision、Core Motion、Core Image 和 PhotoKit；无需服务端、API Key、账号或购买授权，不设置拍摄/构图次数上限。画面分析和照片处理都在设备本地进行，只申请相机及相册「仅添加」权限。

相机方向、取景范围、风景引导及权限流程修复已通过[最新 macOS CI](https://github.com/riky2026/-./actions/runs/43885558553)：Apple SwiftPM 30/30，原生 JPEG 像素测试和工程结构检查通过，模拟器 Debug、设备 Release 均编译成功。已生成未签名 IPA；模拟器交互、签名安装及 iOS 26 真机相机行为尚未验证。

## 获取源码

仓库支持两种布局：

- 完整源码仓库：正常克隆/下载后，在含 `FrameGuide.xcodeproj` 的目录操作。
- 源码包发布仓库：下载根目录的 **FrameGuide-Source.zip** 并完整解压。ZIP 内直接是工程根目录，没有多余外层文件夹。后文所有 `Core/`、`FrameGuide/`、`scripts/`、`docs/` 和 `.github/` 路径都在解压后的目录中；仅含 ZIP 的网页仓库不直接提供这些文档页面。阅读 ZIP 内的 `docs/validation.md` 和 `docs/device-checklist.md` 了解验证证据及验收步骤。

源码包包含完整原生工程、Core 与测试、资源、批准的设计文档、脚本和工作流，不包含原始 IPA、分析材料、Git 历史、个人签名或构建缓存。

## 功能入口

首次启动先显示本地拍摄用途，点「继续」才请求相机权限；拒绝后可前往设置或仅重新检查权限。允许相机后进入竖向 3:4 取景器。底部切换自动/人像/风景，轻点主体对焦及选择主体；顶部按钮控制辅助线、闪光灯和设置，底部右侧切换前后镜头。设置内可选智能/居中/左侧三分/右侧三分、构图引导及就绪触感。

引导在容差内稳定至少 0.6 秒才就绪；主体丢失超过 0.8 秒移除旧引导。快门不依赖构图就绪。前置为镜像预览及镜像照片。快门记录取景范围；预览、最近缩略图及导出均在原生照片像素内裁剪，无法确认对应范围时提示重试。风景无显著区域时可点选主体或按水平提示调整；需以真机核对取景范围和像素尺寸，不承诺所有机型相同分辨率。

拍后选择原片/暖调/清透/黑白，保存到照片或通过系统分享面板分享当前效果。左下角只回看本次会话最近拍摄照片；不会读取整个相册，退出应用不保留会话照片或设置。请先保存需要留存的照片。

## Windows 能做什么

Windows 可以阅读和修改源码、执行 Python 静态检查、生成源码 ZIP，以及从获授权的 GitHub Actions macOS runner 下载构建日志和未签名产物。Windows 本机无法运行 Xcode、Apple iOS SDK、actool 或 iOS 模拟器；Windows 静态检查不等同于 iOS 编译成功。

在完整源码或解压后的根目录运行：

```powershell
python scripts/validate_project.py
powershell -NoProfile -ExecutionPolicy Bypass -File scripts/package_source.ps1
```

输出 `build/FrameGuide-Source.zip`。脚本使用明确白名单，逐层排除缓存、个人 Xcode 状态及签名材料，拒绝链接，并检查最终 ZIP 目录及必需文件。

## macOS / GitHub Actions 构建

工作流 `.github/workflows/ios.yml` 在推送或手动运行时使用 Intel macOS 15 runner（`macos-15-intel`）和 Xcode 16.4，顺序执行 `swift test --package-path Core`、原生 JPEG 标记样图测试、静态检查、模拟器目标编译、无签名设备编译及打包。任何测试/编译失败都会停止后续设备打包，失败日志仍上传。这里只构建模拟器目标，**不会启动模拟器**。

工作流对完整源码仓库直接使用 checkout；对源码包仓库保持 checkout 不变，记录 ZIP 的 SHA-256，安全检查后解压到 runner 临时目录，所有测试、构建和打包均使用同一工程根目录。仓库权限仅 `contents: read`，checkout 不持久化凭据，不上传 Apple 账号或证书，不写回仓库。用户已选择并授权的构建仓库为 [riky2026/-.](https://github.com/riky2026/-.)；远端实际构建结果以 Actions 运行记录为准。

成功后下载 `FrameGuide-UNSIGNED-device-requires-signing` artifact，内有 **FrameGuide-UNSIGNED.ipa（未签名）**；另有 `FrameGuide-native-build-logs`。artifact 的外层 ZIP 只是下载容器。生成 IPA 不能证明已安装、真机相机可用或兼容 iOS 26。

最新产物：[未签名 IPA 下载](https://github.com/riky2026/-./actions/runs/43885558553/artifacts/14234047218) · [构建日志](https://github.com/riky2026/-./actions/runs/43885558553/artifacts/14234461741)。解出的 IPA 为 305844 字节，SHA-256 为 `e890037fff6b727d97252b53064cc1b67c357b0bc7872f40ae726b1b4cfec2f6`。已核对 arm64、iOS 17.0、权限、资源和未签名状态。

## 签名与安装

未签名 IPA 不能直接安装到正常 iPhone。需要用你自己的有效 Apple 开发签名重新签名，且设备、Bundle ID、描述文件和账号权益必须匹配。

有 Mac 时，解压并用 Xcode 打开 `FrameGuide.xcodeproj`，选择 FrameGuide target → Signing & Capabilities，启用 Automatically manage signing，选择自己的 Team，把 Bundle Identifier 改为自己的唯一标识（例如 `com.yourname.FrameGuide`）。连接并信任 iPhone，在手机开启开发者模式后选择设备运行。账号在你自己的 Xcode 中登录，不向本项目提供密码、证书或私钥。个人免费 Team 的安装有有效期和数量等限制；开发签名与 App Store 分发不同，App Store 还需要相应开发者资格、审核和分发流程。

只有 Windows 时，可以在获得真实构建通过的 IPA 后，按 [Sideloadly 官方站点](https://sideloadly.io/) 的 Windows 说明使用自己的 Apple 账号签名安装；官方说明免费账号通常需要每 7 天重新签名。另一条路线是 [AltStore Classic 官方 Windows 安装指南](https://faq.altstore.io/altstore-classic/how-to-install-altstore-windows)，按其设备信任、开发者模式、安装和续签要求操作，再导入自己的 IPA。以官方当时支持的系统、设备和账号要求为准，不保证未实测设备能成功，也不承诺永久安装。上述入口核对日期为 2026-11-16。

## 验证范围

CI 实测源码 ZIP SHA-256 为 `3669eb6322fc84c8f675e2b60d54eb52e3c54c0a031ae32f76a86f96a54fe34b`。该 CI 对应此前版本。当前源码另已修正「实际 JPEG 尺寸小于请求上限」的取景裁剪，新增原生样图回归；此补丁的 Apple 构建及样图执行待验证。历史构建、剩余警告及验收边界见 `docs/validation.md`；实际操作请按 `docs/device-checklist.md` 逐项记录。
