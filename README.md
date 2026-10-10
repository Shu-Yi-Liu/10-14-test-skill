# hello skill

将用户从 PDF 复制粘贴的文献文字整理为 **2 张知识卡片**，每张仅包含**标题、摘要**。这是供 AI 助手读取的 Skill 指令与配置，卡片以 Markdown 文字输出；项目没有独立应用或转换程序。

显示名称为 `hello skill`，内部标识为 `hello-skill`，成品目录为 [`skills/hello skill/`](skills/hello%20skill/SKILL.md)。

## 功能与边界

- 接收可读的全文、摘要或节选，按实证、综述、理论或质性文章的内容提炼两个互补主题。通常分别概括核心问题与发现、方法与证据边界，具体分法随原文调整。
- 默认使用中文，可按用户要求使用其他语言。标题是卡片主题的概括，不一定是论文原标题。
- 仅使用实际收到的原文，保留数值、单位、比较对象、不确定性及因果表述的边界；不补写原文缺失的事实。
- 原文有冲突或关键文字无法辨认时，不猜测；材料不足以支撑两张卡片时，请求补充，不输出空卡片。多篇文献混在一起且目标不清时，先确认处理对象。
- 支持分批粘贴：用户说明尚未发送完时等待，确认发送完成后再整合内容。

暂不支持网页抓取、Anki 导出或同步、图形界面。仅提供题名、网址或 DOI，不能据此生成卡片，也不会联网补全文。

本项目没有 PDF 文件解析或 OCR 脚本，不能承诺直接处理 PDF 路径、附件或扫描件。若使用的助手另有本地 PDF 读取能力，可以先由助手提取原文，再按本 Skill 生成卡片；此前聊天中的本地 PDF 阅读采用了这种方式。该工具能力不随本项目提供，文件放入项目目录也不会自动触发转换。

## 使用方法

在能够读取项目文件的 AI 助手中，发送以下请求并粘贴原文：

```text
请读取当前项目的 skills/hello skill/SKILL.md，
按其中的规则将下面的 PDF 文献节选转换为两张知识卡片：

【粘贴可读原文】
```

如果宿主已经加载并识别此 Skill，也可使用内部标识调用：

```text
请使用 $hello-skill，将下面的原文整理为两张知识卡片：
【粘贴原文】
```

项目中的文件本身不代表已完成全局安装；仅输入 `$hello-skill` 能否调用，取决于宿主是否已加载它。分批输入时，可在前几次消息中注明“尚未发送完”，最后一次注明“发送完成”。

## 输入输出示例

以下为自编演示材料，不是真实论文或研究结果。示例展示格式，实际摘要措辞可能不同。

输入：

```text
摘要节选：我们对120名大学生进行横截面调查。每周阅读时间与词汇测验分数呈正相关；研究不能据此判断因果关系，样本仅来自一所大学。
```

输出：

```markdown
### 知识卡片 1
**标题：** 阅读时间与词汇分数的正相关
**摘要：** 基于所提供的摘要节选，120名大学生的横截面调查显示，每周阅读时间与词汇测验分数呈正相关；这一结果不能用于判断因果关系。

### 知识卡片 2
**标题：** 单校横截面调查的样本边界
**摘要：** 研究采用横截面调查，样本仅来自一所大学；所提供的节选未说明抽样方式。
```

## 核心文件

```text
README.md
release-files.txt            发布文件白名单
skills/
  hello skill/
    SKILL.md                 触发说明、处理规则和输出格式
    agents/openai.yaml       显示名称、简介和默认提示词
    LICENSE                  MIT 许可证
tests/
  behavioral-tests.md        行为测试与迭代记录
```

当前项目没有 `.skill` 打包文件。使用入口是 `SKILL.md`；`agents/openai.yaml` 不是可执行程序。

## 测试与验证

[`tests/behavioral-tests.md`](tests/behavioral-tests.md) 记录了9个场景，覆盖四类文章、数值冲突、内容不足、外部标识、分批输入和文献内嵌指令。输入为自编节选，输出由执行任务的助手生成并逐句自检；不是独立评估或自动化测试。记录中的7个材料充足场景各输出两张卡片，其余场景请求补充材料。

开发时使用官方 skill-creator 的 `scripts/init_skill.py` 创建临时骨架，完善后运行了官方 `scripts/quick_validate.py`，结果为 `Skill is valid!`。临时目录已清理，项目没有附带这些官方脚本。

若要重新进行结构验证，需准备 Python、PyYAML 和本机的官方验证器。在项目根目录执行，例如：

```powershell
# 改为验证器相对项目根目录的实际路径；下述 tools 目录不随项目提供。
$validatorPath = '.\tools\skill-creator\scripts\quick_validate.py'
python -X utf8 $validatorPath 'skills\hello skill'
```

这些依赖用于开发验证，不是本 Skill 自带的 PDF 解析依赖。结构验证检查 Skill 的基本格式，不能证明摘要忠实性；行为测试也不能保证所有文章都能正确处理，生成后仍应与原文核对。

## 发布包

[`release-files.txt`](release-files.txt) 是发布文件白名单，仅包含 README、白名单本身、Skill、配置、许可证和行为测试记录。本地论文 PDF 及其下载来源元数据不包含在发布包中；本地原文件仍可保留用于阅读。

在项目根目录用支持 ZIP 和 `-T` 文件清单参数的 `tar` 打包（已在本机 Windows `tar` 验证）：

```powershell
New-Item -ItemType Directory -Path '.\dist' -Force | Out-Null
tar -a -cf '.\dist\hello-skill.zip' -T '.\release-files.txt'
```

输出为 `dist/hello-skill.zip`。新增需公开的文件时，明确加入白名单后重新打包；不要将整个项目目录直接压缩为发布包。

## 许可证

Skill 使用 [MIT 许可证](skills/hello%20skill/LICENSE)。输入文献的版权不因使用本 Skill 而改变。
