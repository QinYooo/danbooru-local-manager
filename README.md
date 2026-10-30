# Danbooru Local Manager

本地插画图库管理工具。使用 PixAI 标签模型和画面类型模型筛选图片，以 Danbooru 标签和角色组织图库，通过浏览器完成导入、浏览与整理。

运行和分类不需要 QQ Agent、聊天模型或云端图片服务。默认只监听 `127.0.0.1:8766`。

## 1.1 更新

- 放大预览新增左右按钮、键盘切图和底部可横向滚动的缩略图队列，支持沿当前筛选与排序跨页切换。
- 收录时间、文件大小、分辨率均支持正反两个方向排序。
- 支持拖入图片与文件夹，递归读取子文件夹并保留相对路径。
- 新增明确的多选入口、全选本页与批量移入回收站，跨页选择按批提交，失败项保留选择。
- 支持输入页码跳转，批量删除后自动回到有效页码。

安装器不会自动清理已有图库。图片、数据库、本机配置和清理记录不包含在发布包中。

升级：下载新版 ZIP 后重新运行安装器，`-LibraryPath` 指向原图库即可复用数据；模型缓存可继续使用。旧网页服务正在运行时，先停止旧服务再启动新版。

## 安装

Windows 10/11 x64，在 PowerShell 中运行：

```powershell
irm https://raw.githubusercontent.com/QinYooo/danbooru-local-manager/v1.1.0/bootstrap.ps1 | iex
```

也可以从 [Releases](https://github.com/QinYooo/danbooru-local-manager/releases) 下载 ZIP，解压后双击 `install.cmd`。

安装器准备 Python 3.11 独立环境、依赖和模型，并创建桌面启动入口。默认图库是用户的 `Pictures\Danbooru`，运行环境和模型缓存在 `%LOCALAPPDATA%\DanbooruToolkit`。两个 Danbooru 项目可以共用这一缓存。

自动分类需要可用的 NVIDIA 显卡和驱动；安装的是 CUDA 12.6 版 PyTorch，无需手工安装 CUDA Toolkit。权重约 1.95 GB，首次安装还要下载 Python 和 PyTorch，请预留约 8 GB 运行空间。浏览、编辑和恢复已有图库不需要显卡。安装器未找到 NVIDIA 驱动工具时会采用浏览模式。

常用安装选项：

```powershell
# 指定图库位置
.\install.ps1 -LibraryPath 'D:\MyLibrary'

# 只安装浏览/整理环境，跳过推理依赖和大模型
.\install.ps1 -BrowseOnly

# 只配置，不打开网页或创建快捷方式
.\install.ps1 -NoLaunch -NoShortcut
```

以后双击 `start.cmd` 或桌面入口打开网页，`stop.cmd` 停止服务。重新运行安装器可恢复下载或补装分类环境。`config.json` 是本机配置，不提交到 Git。

## 功能

- 新图、待整理、未识别角色、收藏与完整图库。
- 按中文名或原始标签搜索；多个标签组合筛选，按时间、大小或分辨率排序。
- 单张预览、角色和主要角色调整、画面标签编辑、SFW/NSFW 修改、备注与整理标记。
- 批量选择、收藏和移入回收站；从回收站恢复文件和数据库记录。
- 图片多选、拖放、文件夹选择和本地路径批量导入。
- 导入进度、筛除原因、重复项及异常明细。取消任务会在当前图片处理完成后停止。

导入使用同一条本地筛选流水线：尺寸和清晰度检查、完整插画判定、截图/梗图过滤、人物和角色识别、分级，以及全库精确和近似去重。角色未识别但人物明确的合格插画可进入 `_unknown_character`。这些规则用于收集训练候选图，不保证作品一定适合任何特定训练任务；可在 `settings.default.json` 中调整。

## 图库格式

```text
library/
  SFW/<character-tag>/<sha256>.<ext>
  NSFW/<character-tag>/<sha256>.<ext>
  _state/archive.sqlite3
  _state/manager/           # 缩略图、导入暂存、备份与日志
  _trash/                   # 可恢复的图片
```

原图保留原始字节。标签、识别分数、角色与管理状态存入 SQLite，不生成逐图 JSON/TXT。修改主要角色或分级会同步移动保存文件，并更新图片和去重索引。删除先从活动图库移除，再放入可恢复回收站。每日首次启动前会保留数据库备份；数据库备份不替代原图备份。

可以直接打开 QQ 插件版生成的图库。两个程序共享同一图库时使用相同数据库格式，修改能同步反映；不要在外部直接覆盖数据库文件。

## 开发

网页使用原生 HTML/CSS/JavaScript，无前端构建步骤。后端使用 Python 标准库 HTTP 服务、SQLite、Pillow 和 NumPy，分类进程按需加载模型。

```bash
python -m venv .venv
# 激活虚拟环境后
python -m pip install -r requirements-web.txt
python -B -m unittest discover -s tests -p 'test_*.py' -v
node --check web/static/app.js
```

完整推理依赖还需安装 CUDA 12.6 的 `torch==2.8.0`、`torchvision==0.23.0` 和 `requirements.txt`。运行 `tools/setup_runtime.py --help` 查看配置入口。桌面安装器针对 Windows；Linux 的核心与数据接口在 CI 中测试，Linux 桌面启动流程不在当前支持范围。

## 许可证与来源

项目代码使用 MIT。模型、翻译表和名称数据的来源及许可见 [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md)。大模型不会存入 Git 或 Release ZIP；安装时按固定版本下载并检查 SHA-256。上游角色名称语料安装时下载，不随本仓库分发，名称候选仍可能有歧义。

交互设计参考了 [PhotoPrism 的标签与整理流程](https://docs.photoprism.app/user-guide/organize/labels/)和 [Immich 的标签浏览](https://docs.immich.app/features/tags/)。本项目没有使用它们的服务端或前端代码。
