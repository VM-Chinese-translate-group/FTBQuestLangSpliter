# FTB Quests 语言文件助手 (LangSpliter)

这是一个 Python 命令行工具，旨在帮助模组包作者和翻译者更高效地处理 FTB Quests 的语言文件。

传统的翻译流程需要直接编辑一个巨大且复杂的 `en_us.snbt` 文件，这很容易出错且难以管理。本工具通过将大型 SNBT 文件**拆分**为按章节和功能分类的、结构清晰的 JSON 文件，极大地简化了翻译工作。翻译完成后，工具还能将这些 JSON 文件安全地**合并**回一个可供游戏使用的 SNBT 语言文件。

--- Author: Maxing ---

## ✨ 主要功能

-   **智能拆分**: 将源语言文件 (`en_us.snbt`) 分解为多个小的、易于管理的 JSON 文件。
    -   按章节自动生成独立的 JSON 文件（例如 `en_us_chapter1.json`）。
    -   自动将与该章节相关的任务（`quest`）、子任务（`task`）和奖励（`reward`）的文本条目归入对应的章节 JSON 文件中。
    -   将其他类型的条目（如 `chapter_group`, `reward_table`）分离到独立的 JSON 文件中。
-   **有序组织**: 在生成的章节 JSON 文件中，所有条目都会按照“章节 -> 任务 -> 子任务 -> 奖励”的逻辑层次进行排序，使上下文更清晰。
-   **安全合并**: 将所有（已翻译的）JSON 文件合并回一个单一的、格式正确的 SNBT 语言文件，为多行文本自动处理转义和列表重建。
-   **高度灵活**:
    -   通过命令行参数可自定义所有文件和目录路径。
    -   提供选项以控制如何处理单行文本列表。

## ⚙️ 环境要求与安装

1.  **Python 3.x**: 请确保您的系统中已安装 Python 3。

2.  **安装依赖库**: 本脚本需要 `snbtlib` 库来解析和生成 SNBT 文件。打开您的终端或命令提示符，运行以下命令进行安装：

    ```bash
    pip install snbtlib
    ```

## 📂 目录结构

为了让脚本正常工作，请按照以下结构组织您的文件。脚本将根据这些默认路径运行，您也可以通过命令行参数指定自定义路径。

```
your_project/
├── chapters/                 # 存放所有章节定义的 .snbt 文件
│   ├── getting_started.snbt
│   └── another_chapter.snbt
│
├── lang/                     # 存放语言文件
│   └── en_us.snbt            # [输入] 原始的英文语言文件
│
├── output_json/              # [输出] 脚本生成的 JSON 文件将存放在这里
│
├── chapter_groups.snbt       # 章节信息文件
│
└── LangSpliter.py            
```

-   `chapters/`: 包含您模组包中所有章节的 `.snbt` 文件。脚本需要读取这些文件来了解任务和章节的对应关系。
-   `lang/en_us.snbt`: 这是您要拆分的源语言文件。
-   `output_json/`: 这是一个输出目录。如果它不存在，脚本会自动创建。

## 🚀 使用方法

本工具通过命令行进行操作。所有命令都以 `python LangSpliter.py` 开头。

### 1. 拆分 SNBT 文件 (`split`)

此任务用于将 `lang/en_us.snbt` 文件拆分为多个 JSON 文件并存入 `output_json/` 目录。

**基本用法 (使用默认路径):**

```bash
python LangSpliter.py split
```

**高级用法:**

-   **展平单行列表**: 如果 SNBT 中的一个列表只有一项（例如 `key: ["line 1"]`），默认会转换为 `key1: "line 1"`。使用此参数可以将其转换为 `key: "line 1"`，不带数字后缀。

    ```bash
    python LangSpliter.py split --flatten-single-lines
    ```

-   **指定自定义路径**:

    ```bash
    python LangSpliter.py split --source-lang "my_lang/en.snbt" --chapters-dir "my_quests" --output-dir "json_output"
    ```

### 2. 合并 JSON 文件 (`merge`)

此任务用于将 `output_json/` 目录中的所有 `.json` 文件合并成一个单一的 SNBT 文件。

#### `merge`

这是推荐的合并方法。它会先对所有键进行简单排序，确保每次生成的 SNBT 文件内容顺序一致。

**基本用法 (生成 `lang/zh_cn.snbt`):**

```bash
python LangSpliter.py merge
```

**指定自定义路径:**

```bash
python LangSpliter.py merge --json-dir "translated_json" --output-snbt "final_lang/fr_fr.snbt"
```

**基本用法:**

```bash
python LangSpliter.py merge-legacy
```

## 📖 推荐工作流程

1.  **准备工作**:
    -   将 `LangSpliter.py` 脚本放入`{整合包文件夹}\config\ftbquests\quests`。

2.  **第一步：拆分**
    -   打开终端，运行拆分命令：
        ```bash
        python LangSpliter.py split
        ```
    -   检查 `output_json/` 目录，您会看到一堆 `en_us_...json` 文件。

3.  **第二步：翻译**
    -   将 `output_json/` 目录下所有文件上传至ParaTrans并进行翻译。

4.  **第三步：合并**
    -   完成所有 `en_us_...json` 文件的翻译后，回到终端。
    -   运行合并命令，并指定输出为您目标语言的文件。
        ```bash
        python LangSpliter.py merge --json-dir "output_json" --output-snbt "lang/zh_cn.snbt"
        ```
    -   **注意**: 脚本会加载 `output_json` 目录下的所有 JSON 文件。

5.  **完成**
    -   现在，`lang/zh_cn.snbt` 就是您最终需要的、可以在模组包中使用的语言文件了！