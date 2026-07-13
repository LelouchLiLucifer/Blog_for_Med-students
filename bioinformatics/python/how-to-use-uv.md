---
description: uv
cover: ../../.gitbook/assets/uv.webp
coverY: 0
coverHeight: 219
---

# 高效管理 Python 环境 -- 科学的使用 uv

本教程面向需要运行、复核或维护本课题组 Python 分析代码的医学牲，重点解决三个问题：

1. 如何安装 `uv` 并获得项目要求的 Python；
2. 如何为现有项目创建隔离的虚拟环境并运行代码；
3. 如何让半年后、换一台电脑或交给审稿人时，仍能尽可能重建同一套Python环境。

## 1. uv 是什么

[`uv`](https://docs.astral.sh/uv/) 是 Astral 开发的 Python 包与项目管理工具。在本项目中，它同时承担以下工作：

* 安装和选择合适的 Python 解释器；
* 在项目目录创建隔离的 `.venv` 虚拟环境；
* 根据 `pyproject.toml` 解析直接依赖；
* 使用 `uv.lock` 锁定完整的直接依赖和传递依赖；
* 将锁定环境同步到 `.venv`；
* 在正确的环境中运行 Python 脚本。

对于课题组协作，uv 的核心价值不是“安装得快”，而是把环境声明、依赖解析、环境创建和脚本执行整合为一套可审查的工作流。

## 2. 先理解项目中的环境文件

### 2.1 文件职责

| 文件或目录                   | 本项目中的作用                                  |
| ----------------------- | ---------------------------------------- |
| `.python-version`       | 指定默认 Python 版本请求；例如 `3.11`               |
| `pyproject.toml`        | 声明 Python 范围、直接依赖、支持平台和 PyTorch 软件源      |
| `uv.lock`               | uv 的权威锁文件，记录完整依赖图、版本、来源和哈希               |
| `requirements.lock.txt` | 从 `uv.lock` 导出的 pip 兼容清单，供不支持 uv 的外部工具使用 |
| `.venv/`                | 当前电脑上的项目虚拟环境，可随时从锁文件重建                   |

应共享的是环境的“配方”——`.python-version`、`pyproject.toml` 和 `uv.lock`，而不是体积大、包含本机路径且不可移植的 `.venv`。

### 2.2 案例实际锁定了什么

下面是一个`pyproject.toml` 案例：

```toml
[project]
name = "test"
version = "0.1.0"
description = "test workflow."
requires-python = ">=3.11,<3.12"
dependencies = [
    "numpy==2.3.5",
    "pandas==2.3.3",
    "scipy==1.16.3",
    "scikit-learn==1.6.1",
    "pyreadr==0.5.6",
    "joblib==1.5.3",
    "torch==2.11.0+cu130",
    "matplotlib==3.10.9",
    "seaborn==0.13.2",
    "ipykernel==6.30.1",
    "pip==25.3",
    "setuptools<81",
]

[tool.uv]
package = false
environments = [
    "sys_platform == 'win32' and platform_machine == 'AMD64'",
    "sys_platform == 'linux' and platform_machine == 'x86_64'",
]

[tool.uv.sources]
torch = { index = "pytorch-cu130" }

[[tool.uv.index]]
name = "pytorch-cu130"
url = "https://download.pytorch.org/whl/cu130"
explicit = true
```

这些配置表示：

* 只允许 Python 3.11 系列，不允许 Python 3.10 或 3.12；
* `package = false` 表示该目录作为依赖管理项目使用，不把项目自身安装成 Python 包；
* 锁文件只面向 Windows AMD64 和 Linux x86\_64；
* `torch` 必须从专用的 PyTorch CUDA 13.0 索引获取；
* 其他依赖仍使用正常配置的软件源；
* 当前锁文件包含 `numpy==2.3.5`,`torch==2.11.0+cu130` 等精确版本。

`.python-version` 中的 `3.11` 与 `requires-python = ">=3.11,<3.12"` 相互配合：前者帮助 uv 选择解释器，后者定义项目允许的 Python 范围。

## 3. 安装 uv

安装方法以 [uv 官方安装文档](https://docs.astral.sh/uv/getting-started/installation/) 为准。课题组电脑优先采用官方独立安装器或操作系统包管理器，不建议先创建一个普通 Python 环境再用 `pip install uv`，否则 uv 本身会依赖另一个 Python 环境。

### 3.1 Windows：PowerShell 主线方法

在 PowerShell 中运行官方独立安装器：

```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

安装后关闭并重新打开 PowerShell，然后检查：

```powershell
uv --version
uv python list
```

也可以使用 WinGet （笔者倾向于在windows上使用该方法）：

```powershell
winget install --id=astral-sh.uv -e
```

若使用 WinGet 安装，后续应使用 WinGet 的升级机制，而不是 `uv self update`。

### 3.2 Linux 和 macOS

使用官方独立安装器：

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

重新打开终端后检查：

```bash
uv --version
uv python list
```

macOS 也可以使用 Homebrew：

```bash
brew install uv
```

### 3.3 uv 的升级与版本记录

如果使用官方独立安装器，可执行：

```powershell
uv self update
```

如果使用 WinGet、Homebrew 等包管理器，应通过相同的包管理器升级。

升级 uv 与升级项目依赖是两件不同的事。`uv self update` 不会主动升级 `pyproject.toml` 中的包；同样，`uv sync --locked` 也不负责升级 uv。

每次形成论文最终结果或归档复现材料时，至少记录：

```powershell
uv --version
uv run --locked python -VV
```

如果课题组需要严格固定 uv 自身，可使用官方文档提供的版本化安装地址，例如 `https://astral.sh/uv/{version}/install.ps1` 或 `https://astral.sh/uv/{version}/install.sh`。其中 `{version}` 应替换为课题组经过验证并记录的具体版本。不要把某位成员电脑上碰巧安装的版本直接声明为项目要求。

## 4. 首次复现本项目

{% stepper %}
{% step %}
### 进入正确的项目目录

uv 会从当前目录向上寻找 `pyproject.toml`、`uv.lock` 和 `.python-version`。因此，执行命令前必须进入公开子项目根目录，而不是总项目根目录。

{% tabs %}
{% tab title="Windows PowerShell" %}
```powershell
Set-Location "D:\path\to\file"
```
{% endtab %}

{% tab title="Linux" %}
```bash
cd /path/to/file
```
{% endtab %}
{% endtabs %}

确认当前目录包含三个核心文件：

{% tabs %}
{% tab title="Windows PowerShell" %}
```powershell
Get-ChildItem .python-version, pyproject.toml, uv.lock
```
{% endtab %}

{% tab title="Linux" %}
```bash
ls -l .python-version pyproject.toml uv.lock
```
{% endtab %}
{% endtabs %}
{% endstep %}

{% step %}
### 安装项目要求的 Python

按照项目声明安装最新可用的 Python 3.11 补丁版本：

{% tabs %}
{% tab title="Windows PowerShell" %}
```powershell
uv python install 3.11
```
{% endtab %}

{% tab title="Linux" %}
```bash
uv python install 3.11
```
{% endtab %}
{% endtabs %}

关于 uv 如何查找和安装 Python，可参阅 [Python 版本管理](https://docs.astral.sh/uv/concepts/python-versions/)。
{% endstep %}

{% step %}
### 从锁文件创建虚拟环境

标准复现命令为：

{% tabs %}
{% tab title="Windows PowerShell" %}
```powershell
uv sync --locked
```
{% endtab %}

{% tab title="Linux" %}
```bash
uv sync --locked
```
{% endtab %}
{% endtabs %}

此命令会：

1. 检查 `uv.lock` 是否与 `pyproject.toml` 一致；
2. 如有需要，在项目根目录自动创建 `.venv`；
3. 安装锁文件规定的平台适用依赖；
4. 在锁文件过期时立即失败，而不是悄悄重写锁文件。

若要明确使用已验证的版本，例如 Python 3.11.15，可在首次创建环境时执行：

{% tabs %}
{% tab title="Windows PowerShell" %}
```powershell
uv sync --locked --python 3.11.15
```
{% endtab %}

{% tab title="Linux" %}
```bash
uv sync --locked --python 3.11.15
```
{% endtab %}
{% endtabs %}

复现者不要把 `--locked` 省略。普通 `uv sync` 允许 uv 在发现项目声明变化时更新锁文件；`uv sync --locked` 才能保证复现过程中不改写权威锁文件。

`uv sync` 默认执行精确同步，会移除 `.venv` 内不属于锁定环境的额外包。因此，不要在 `.venv` 中存放数据、脚本、模型或人工编辑的文件。
{% endstep %}

{% step %}
### 检查锁文件状态

```powershell
uv lock --check
```

成功时说明 `uv.lock` 与当前 `pyproject.toml` 一致。该检查不会因为软件源出现了新版本而自动升级包。

完整的锁定与同步行为见 [Locking and syncing](https://docs.astral.sh/uv/concepts/projects/sync/)。
{% endstep %}
{% endstepper %}

## 5. 是否需要激活 `.venv`

### 5.1 推荐：不手工激活，直接使用 uv run

uv 可以自动在项目环境中运行命令：

```powershell
uv run --locked python -VV
```

运行项目脚本时也采用相同形式：

```powershell
uv run --locked python .\scripts.py
```

这种方式的优点是命令明确记录了“使用项目环境”和“不得修改锁文件”，不依赖终端是否已经激活了正确环境。

### 5.2 可选：手工激活

PowerShell：

```powershell
. .\.venv\Scripts\Activate.ps1
python -VV
```

Windows CMD：

```batch
.venv\Scripts\activate.bat
python -VV
```

Linux/macOS：

```bash
source .venv/bin/activate
python -VV
```

退出虚拟环境：

```
deactivate
```

如果 PowerShell 的执行策略阻止激活脚本，不必为了运行项目而修改系统策略，直接使用 `uv run --locked ...` 即可。

## 6. 为什么这套流程可以复现

### 6.1 `pyproject.toml` 描述意图

它记录研究代码直接依赖什么，例如 NumPy、pandas、scikit-learn、PyTorch 和 Jupyter 内核，以及每个包允许的版本。

### 6.2 `uv.lock` 固定完整解析结果

直接依赖还会依赖其他包。只保存一份手写的顶层依赖列表，未来重新解析时可能得到不同的传递依赖。`uv.lock` 记录完整依赖图、精确版本、软件源、平台条件和包哈希，是本项目的权威环境快照。

### 6.3 `--locked` 阻止静默漂移

常规 `uv run` 会在运行前自动检查并同步项目；如果项目声明变化，它可能更新锁文件。复现命令使用 `uv run --locked` 或 `uv sync --locked`，可以在锁文件过期时失败并暴露问题。

不要用 `--frozen` 代替日常的 `--locked`。`--frozen` 会使用现有锁文件但跳过其与项目元数据的一致性检查，适合明确理解其含义的特殊场景，不适合作为课题组默认复现命令。

### 6.4 `requirements.lock.txt` 是兼容导出，不是第二权威来源

某些审稿平台、传统服务器或外部工具只接受 `requirements.txt`。本项目因此保留 `requirements.lock.txt`，但它来自 `uv.lock`，不应被独立手工维护。

正确的关系是：

```
pyproject.toml ──解析──> uv.lock ──导出──> requirements.lock.txt
                           │
                           └──同步──> .venv/
```

## 7. 复现者工作流：只使用锁文件

复现者包括课题组新成员、外部复核者和审稿人。其目标是重建已经批准的环境，不是升级依赖。

### 7.1 标准命令

{% tabs %}
{% tab title="Windows PowerShell" %}
```powershell
Set-Location "D:\path\to\file"
uv --version
uv python install 3.11
uv lock --check
uv sync --locked
uv run --locked python -VV
uv run --locked python .\scripts.py
```
{% endtab %}

{% tab title="Linux" %}
```bash
cd /path/to/file
uv --version
uv python install 3.11
uv lock --check
uv sync --locked
uv run --locked python -VV
uv run --locked python scripts.py
```
{% endtab %}
{% endtabs %}

### 7.2 复现者不应做的事

* 不执行临时的 `pip install`、`uv pip install` 或 Notebook `%pip install` 来“补包”；
* 不手工编辑 `uv.lock` 或 `requirements.lock.txt`；
* 不使用 `uv lock --upgrade`；
* 不把 `.venv` 复制给其他成员或提交到版本控制；
* 不在未记录原因和验证结果时升级 Python、uv 或关键依赖。

遇到缺包时，应把报错、执行命令和 `uv lock --check` 结果交给环境维护者。缺包通常意味着项目声明不完整，应该从源头修复，而不是只修复某一台电脑。

## 8. 环境维护者工作流：受控地改变环境

只有明确负责环境维护的成员，才应修改依赖声明和锁文件。每次变更都应与代码变更、验证结果和变更原因一起审查。

{% stepper %}
{% step %}
### 新增依赖

优先指定经过验证的版本：

```powershell
uv add "example-package==1.2.3"
```

`uv add` 会更新 `pyproject.toml`、`uv.lock` 和当前环境。新增后必须运行项目级验证，不能只测试 `import` 是否成功。
{% endstep %}

{% step %}
### 删除依赖

```powershell
uv remove example-package
```

删除前应确认研究脚本、Notebook、间接调用和导出流程均不再需要该包。
{% endstep %}

{% step %}
### 受控升级单个依赖

若要把单个包升级到经过评估的版本：

```powershell
uv lock --upgrade-package example-package==1.3.0
uv sync --locked
```

若直接依赖的版本约束也需要改变，应使用 `uv add "example-package==1.3.0"`，让 `pyproject.toml` 和锁文件一起更新。

不建议在论文关键阶段直接执行全量升级：

```
uv lock --upgrade
```

全量升级可能同时改变大量传递依赖，应只在计划好的环境更新窗口执行，并重新进行完整分析验证。
{% endstep %}

{% step %}
### 重新导出 pip 兼容清单

确认新锁文件通过验证后，从权威锁文件重新生成：

```powershell
uv export --locked --format requirements.txt --output-file requirements.lock.txt
```

随后审查：

* `pyproject.toml` 中直接依赖是否符合预期；
* `uv.lock` 中是否出现意外的软件源、平台变化或大范围升级；
* `requirements.lock.txt` 是否确实由当前 `uv.lock` 导出；
* 环境自检、单元测试和关键统计结果是否通过；
* 模型性能、校准、解释性结果和发表图表是否出现非预期变化。
{% endstep %}

{% step %}
### 维护完成后的最小检查

```powershell
uv lock --check
uv sync --locked
uv run --locked python -VV
uv run --locked python .\scripts.py
```

注意：依赖升级不仅是工程变更，也可能改变随机性、数值计算、模型拟合、概率输出和图形结果，应按分析方案重新验证。
{% endstep %}
{% endstepper %}

## 9. VS Code 和 Jupyter

### 9.1 VS Code 选择项目解释器

先在项目根目录运行：

```powershell
uv sync --locked
```

然后在 VS Code 中：

1. 打开项目目录；
2. 执行 **Python: Select Interpreter**；
3. 选择项目中的 `.venv\Scripts\python.exe`；
4. 对 Notebook 选择同一个 `.venv` 内核。

Linux 对应解释器路径为 `.venv/bin/python`。

建议提前在锁文件中包含 `ipykernel==6.30.1` 和 `pip==25.3`。(版本号以实际兼容的版本为准)\
如果 VS Code 提示为 `.venv` 安装 `ipykernel` 或 `pip`，不要点击临时安装；先重新运行 `uv sync --locked`。

### 9.2 Notebook 中不要临时安装依赖

以下操作可能让 Notebook 在当前电脑上暂时可运行，但不会形成可审查的项目依赖变更：

```
%pip install some-package
!pip install some-package
!uv pip install some-package
```

如果 Notebook 确实需要新依赖，应退出交互式试错流程，由环境维护者使用 `uv add` 更新项目声明并完成验证。

更多集成方式见 [Using uv with Jupyter](https://docs.astral.sh/uv/guides/integration/jupyter/)。

## 10. 常见问题排查

<details>

<summary>10.1 系统提示找不到 <code>uv</code></summary>

先关闭并重新打开终端，再运行：

```powershell
uv --version
```

仍然找不到时，检查是否使用了与安装方法一致的用户账户和终端，并回到官方安装文档核对 PATH 配置。不要通过另一个未记录的 Python 环境临时安装第二份 uv。

</details>

<details>

<summary>10.2 <code>uv sync --locked</code> 提示锁文件过期</summary>

先运行：

```powershell
uv lock --check
```

如果失败，说明 `pyproject.toml` 与 `uv.lock` 不一致。复现者应停止并联系维护者，不要通过去掉 `--locked` 来绕过检查。维护者需要确认这是未完成的合法依赖变更，还是误改了项目声明。

</details>

<details>

<summary>10.3 Python 版本不符合要求</summary>

检查：

```powershell
uv run --locked python -VV
uv python list
```

然后安装项目允许的版本(例如3.11)并重新同步：

```powershell
uv python install 3.11
uv sync --locked --python 3.11
```

</details>

<details>

<summary>10.4 出现 hardlink 警告</summary>

当 uv 缓存与项目 `.venv` 位于不同文件系统或磁盘时，uv 可能无法创建硬链接。这通常是性能警告，不代表依赖解析失败。

PowerShell 当前会话可设置：

```powershell
$env:UV_LINK_MODE = "copy"
uv sync --locked
```

Windows CMD：

```batch
set UV_LINK_MODE=copy
uv sync --locked
```

Linux/macOS：

```bash
export UV_LINK_MODE=copy
uv sync --locked
```

`copy` 会占用更多磁盘空间，但避免跨文件系统硬链接问题。有关可选值可查阅 [uv CLI reference](https://docs.astral.sh/uv/reference/cli/)。

</details>

## Ref

* [uv 官方文档首页](https://docs.astral.sh/uv/)
* [安装 uv](https://docs.astral.sh/uv/getting-started/installation/)
* [Python 版本管理](https://docs.astral.sh/uv/concepts/python-versions/)
* [项目结构与环境文件](https://docs.astral.sh/uv/concepts/projects/layout/)
* [锁定与同步](https://docs.astral.sh/uv/concepts/projects/sync/)
* [在 Jupyter 中使用 uv](https://docs.astral.sh/uv/guides/integration/jupyter/)
* [uv 命令行参考](https://docs.astral.sh/uv/reference/cli/)
