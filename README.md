# academic-wechat

将一篇学术论文 PDF 转化为**可导入公众号排版器的中文专业推文**的 Claude Agent Skill：自动生成导读与分节正文、按页自动裁切论文原图、输出图片溯源清单。面向具有一般科学素养的读者，以准确传达论文信息为先，不逐句翻译、不代替同行评议。

## 交付物

对一篇论文执行后，在 PDF 所在目录生成：

```text
推文-{作者姓}{年份}/
├── 推文-{作者姓}{年份}.md      # 排版器导入用正文（导读、分节、图题、论文信息、声明）
├── 图片裁剪清单.docx           # 每张图的原图号/页码/裁剪情况/插入位置（溯源用）
└── 论文原图/
    ├── Fig1.png … FigN.png     # 已自动裁切，插入到对应图题上方即可
```

- 正文采用排版器 Markdown 契约（`::: figure 图N`、`::: term`、`::: paper` 等），不嵌 HTML/CSS。
- 原图裁切全自动完成，无需手动裁图；仅当清单中注明「需人工微调」时才需要处理个别图。

## 运行环境（新电脑首次使用前必装）

Skill 本体只是 Markdown 指令，拷贝即被识别；执行依赖 **Python 及其库**，必须**系统级安装**（任何目录可用），**不要**在工作目录内创建临时虚拟环境（venv）。

```bash
# 1) 安装 Python 3.10+ 并加入 PATH
#    Windows: python.org 安装包勾选 "Add python.exe to PATH"，或 winget install Python.Python.3.12
#    macOS:   brew install python

# 2) 全局安装依赖（pip 默认装到用户/系统 site-packages，对所有项目生效）
python -m pip install --upgrade pip
python -m pip install pymupdf pillow python-docx

# 3) 一键验证（三个都打印 ok 即就绪）
python -c "import fitz; print('fitz ok')"
python -c "import PIL; print('PIL ok')"
python -c "import docx; print('docx ok')"
```

| 包 | import 名 | 用途 |
|---|---|---|
| `pymupdf` | `fitz` | 读 PDF 文本、按页自动裁切原图（核心，必需） |
| `pillow` | `PIL` | 裁图自检（缺了跳过目检，仍可出图） |
| `python-docx` | `docx` | 生成 Word 溯源清单（缺了自动降级为 `.txt`） |

可选：`python -m pip install markitdown`（文本型 PDF 的备用抽取通道）。

本 skill **不需要** LaTeX/TeXlive、浏览器插件或联网服务，PDF 解析与裁图全部本地完成。

## 安装

把整个 `academic-wechat/` 文件夹复制（或 `git clone`）到 Claude 的 skills 目录：

```text
Windows:  C:\Users\<用户名>\.claude\skills\academic-wechat\
macOS:    ~/.claude/skills/academic-wechat/
```

重启会话后即被发现。仓库文件结构：

```text
SKILL.md                          # 主流程：工作流、标题/图题格式、排版器契约、发布前检查、环境节
references/editorial-standard.md  # 编辑规范：受众、归因规则、论文类型路由、成稿检查
README.md                         # 本说明
```

## 使用

在装有本 skill 的 Claude 会话中：

```text
请用 academic-wechat 写这篇论文的公众号推文：D:\path\to\paper.pdf
```

按 `SKILL.md` 工作流执行：读取并核实论文 → 建立编辑简报 → 自动裁图 → 同步写稿与图题 → 生成 Word 清单 → 按发布前检查清单校验后交付。

## 后续操作（交付后）

1. 将 `推文-*.md` 导入公众号排版器，选模板后复制到公众号后台；
2. 在各图题（渲染为「图1」「图2」…）正上方插入 `论文原图/Fig1.png`、`Fig2.png`… 对应文件；
3. `图片裁剪清单.docx` 仅作内部核对，不随推文发布。

## 相关规范

- 编辑标准见 [references/editorial-standard.md](references/editorial-standard.md)：准确与归因优先，不评价论文水平，"首次/突破"等词须有原文依据。
- 文末固定声明、`::: paper` 作者全名单（通讯作者标注）、导读字数、章节命名等细则见 `SKILL.md`。
