# 取景 · FrameGuide

原生 iPhone 相机源码，最低 iOS 17.0，首版拍摄界面固定竖屏。使用 SwiftUI、AVFoundation、Vision、Core Motion、Core Image 和 PhotoKit；本地拍摄与构图无需 API Key 或账号，不设置次数上限；可选 DeepSeek AI 构图使用用户自己的 API Key、配额和余额，无项目服务器。实时本地引导与拍后处理在设备进行，只申请相机及相册「仅添加」权限。

本轮新增可选 DeepSeek AI 构图；其原生 fixture、iOS 编译和真实服务调用尚未验证。下列构建 2 结果仅属于拍摄修复的历史版本，不能作为本轮 AI 源码的构建证据。iPhone 15 / iOS 18.7 所有设备项目均尚未真机验证。

构建 2 针对 iPhone 15 / iOS 18.7 上前后镜头均出现“无法确认当前照片的取景范围”、快门无法工作的反馈。现在相机就绪即可拍摄：若不能可靠确定取景边界，拍后标明“本次保留完整画面”，仍可预览、保存和分享。构建 2 已通过 30 项 Core 测试、原生 JPEG 像素样图验证及模拟器和设备目标编译。iPhone 15 两个镜头的实际拍摄、保存和分享仍待复测。

## 获取源码

仓库支持两种布局：

- 完整源码仓库：正常克隆/下载后，在含 `FrameGuide.xcodeproj` 的目录操作。
- 源码包发布仓库：下载根目录的 **FrameGuide-Source.zip** 并完整解压。ZIP 内直接是工程根目录，没有多余外层文件夹。后文所有 `Core/`、`FrameGuide/`、`scripts/`、`docs/` 和 `.github/` 路径都在解压后的目录中；仅含 ZIP 的网页仓库不直接提供这些文档页面。阅读 ZIP 内的 `docs/validation.md` 和 `docs/device-checklist.md` 了解验证证据及验收步骤。

源码包包含完整原生工程、Core 与测试、资源、批准的设计文档、脚本和工作流，不包含原始 IPA、分析材料、Git 历史、个人签名或构建缓存。

## 功能入口

首次启动先显示本地拍摄用途，点「继续」才请求相机权限；拒绝后可前往设置或仅重新检查权限。允许相机后进入竖向 3:4 取景器。底部切换自动/人像/风景，轻点主体对焦及选择主体；顶部按钮控制辅助线、闪光灯和设置，底部右侧切换前后镜头。设置内可选智能/居中/左侧三分/右侧三分、构图引导及就绪触感。

引导在容差内稳定至少 0.6 秒才就绪；主体丢失超过 0.8 秒移除旧引导。快门不依赖构图就绪。前置为镜像预览及镜像照片。快门记录取景范围；预览、最近缩略图及导出均在原生照片像素内裁剪，无法确认对应范围时保留原生完整画面。风景无显著区域时可点选主体或按水平提示调整；需以真机核对取景范围和像素尺寸，不承诺所有机型相同分辨率。

拍后选择原片/暖调/清透/黑白，保存到照片或通过系统分享面板分享当前效果。左下角只回看本次会话最近拍摄照片；不会读取整个相册，退出应用不保留会话照片或设置。请先保存需要留存的照片。

## 可选 DeepSeek AI 构图与数据处理

在设置中自行用安全输入框填写自有 API Key，保存到本机 Keychain；不要在聊天中发送 Key。未配置 Key 仍可拍照、保存、分享和使用本地构图。点「AI 构图」只截取一张当前取景快照，先查看方向、前置左右和 3:4 画面，再点「发送并分析」才发起请求。快照最长边不超过 1280 像素，以重新编码的 JPEG 移除位置和原始照片元数据；图像内容本身仍可能透露个人信息。

确认后发送单张 JPEG 与构图提示给固定的 `https://api.deepseek.com/chat/completions`，使用 `deepseek-flash` 和本机 API Key 认证。普通预览、快门和本地引导不会自动上传画面。供应商处理依其当时政策和服务条款；本项目不承诺零保留、零训练或供应商即时删除。删除 Key 只删除本机凭据，不能撤回已发送请求。

返回最多三套文字构图方案，选择方案会切换本地场景和构图位置规则，再由本地识别引导。镜头切换、离开相机、后台、取消或重新分析会使旧请求/结果失效；明显画面变化及超时也会清除旧结果。这是保守变化检测，不是三维跟踪。网络、认证或余额错误不应阻断快门；这些行为尚未真机验证。

隐私清单保守申报照片/视频用于 App 功能、与 API 账户关联、无跟踪；维持空的 required-reason API 列表。这不是完整的 App Store Connect 隐私申报。Apple 的收集定义区分实时请求服务所需的暂存和更长保留；不能因为发送按钮可选就断言无需披露，发布前须核对包括第三方在内的实际数据处理及所有相关类型。[Apple 隐私定义](https://developer.apple.com/app-store/app-privacy-details/) · [Apple manifest 键](https://developer.apple.com/documentation/bundleresources/describing-data-use-in-privacy-manifests) · [DeepSeek API](https://api-docs.deepseek.com/api/create-chat-completion/) · [DeepSeek 隐私政策](https://cdn.deepseek.com/policies/en-US/deepseek-privacy-policy.html)。核对日期：2026-11-17；供应商通用政策不替代下游应用自身披露。

## Windows 能做什么

Windows 可以阅读和修改源码、执行 Python 静态检查、生成源码 ZIP，以及从获授权的 GitHub Actions macOS runner 下载构建日志和未签名产物。Windows 本机无法运行 Xcode、Apple iOS SDK、actool 或 iOS 模拟器；Windows 静态检查不等同于 iOS 编译成功。

在完整源码或解压后的根目录运行：

```powershell
python scripts/validate_project.py
powershell -NoProfile -ExecutionPolicy Bypass -File scripts/package_source.ps1
```

输出 `build/FrameGuide-Source.zip`。脚本使用明确白名单，逐层排除缓存、个人 Xcode 状态及签名材料，拒绝链接，并检查最终 ZIP 目录及必需文件。

## macOS / GitHub Actions 构建

工作流 `.github/workflows/ios.yml` 在推送或手动运行时使用 Intel macOS 15 runner（`macos-15-intel`）和 Xcode 16.4，顺序执行 `swift test --package-path Core`、原生 JPEG 标记样图测试、离线 AI 客户端/快照/取消 fixture、静态检查、模拟器目标编译、无签名设备编译及打包。任何测试/编译失败都会停止后续设备打包，失败日志仍上传。这里只构建模拟器目标，**不会启动模拟器**。

工作流对完整源码仓库直接使用 checkout；对源码包仓库保持 checkout 不变，记录 ZIP 的 SHA-256，安全检查后解压到 runner 临时目录，所有测试、构建和打包均使用同一工程根目录。仓库权限仅 `contents: read`，checkout 不持久化凭据，不上传 Apple 账号或证书，不写回仓库。用户已选择并授权的构建仓库为 [riky2026/-.](https://github.com/riky2026/-.)；远端实际构建结果以 Actions 运行记录为准。

成功后下载 `FrameGuide-UNSIGNED-device-requires-signing` artifact，内有 **FrameGuide-UNSIGNED.ipa（未签名）**；另有 `FrameGuide-native-build-logs`。artifact 的外层 ZIP 只是下载容器。生成 IPA 不能证明已安装、真机相机可用或兼容 iOS 26。

以下为历史拍摄修复产物，不包含本轮 AI 功能。本轮未生成经原生 CI 验证的新 IPA。

构建 2 的未签名 IPA 为 314064 字节，SHA-256 为 `397d887ebfe32d4c3ca1d040a35f0f71602be9855f5fa13b0a4dcc2e2362e3fd`。已核对构建号 2、arm64 和最低 iOS 17.0。[下载构建 2](https://github.com/riky2026/-./actions/runs/44065811434/artifacts/14320083542) · [构建日志](https://github.com/riky2026/-./actions/runs/44065811434/artifacts/14320233227)

## 签名与安装

未签名 IPA 不能直接安装到正常 iPhone。需要用你自己的有效 Apple 开发签名重新签名，且设备、Bundle ID、描述文件和账号权益必须匹配。

有 Mac 时，解压并用 Xcode 打开 `FrameGuide.xcodeproj`，选择 FrameGuide target → Signing & Capabilities，启用 Automatically manage signing，选择自己的 Team，把 Bundle Identifier 改为自己的唯一标识（例如 `com.yourname.FrameGuide`）。连接并信任 iPhone，在手机开启开发者模式后选择设备运行。账号在你自己的 Xcode 中登录，不向本项目提供密码、证书或私钥。个人免费 Team 的安装有有效期和数量等限制；开发签名与 App Store 分发不同，App Store 还需要相应开发者资格、审核和分发流程。

只有 Windows 时，可以在获得真实构建通过的 IPA 后，按 [Sideloadly 官方站点](https://sideloadly.io/) 的 Windows 说明使用自己的 Apple 账号签名安装；官方说明免费账号通常需要每 7 天重新签名。另一条路线是 [AltStore Classic 官方 Windows 安装指南](https://faq.altstore.io/altstore-classic/how-to-install-altstore-windows)，按其设备信任、开发者模式、安装和续签要求操作，再导入自己的 IPA。以官方当时支持的系统、设备和账号要求为准，不保证未实测设备能成功，也不承诺永久安装。上述入口核对日期为 2026-11-16。

## 验证范围

当前 AI 源码已加入新原生 CI 门禁，但尚未执行；真实 API 调用、模拟器交互、个人签名安装及 iPhone 15 / iOS 18.7 设备清单均待完成。源码包仓库上传时须同步外层 workflow 与 ZIP 内副本；CI 会逐字节核对并记录 workflow SHA-256。

构建 2 的 CI 实测源码 ZIP SHA-256 为 `3c0dff937ca2dd204aab070bf601ae2dd924d1462fee586eb178466bb239a0ad`。本次交付在被测版本上更新了 README 和验证记录，因此交付 ZIP 的哈希不同。构建 1 的历史产物和验证记录见 `docs/validation.md`。真机复测步骤见 `docs/device-checklist.md`。
