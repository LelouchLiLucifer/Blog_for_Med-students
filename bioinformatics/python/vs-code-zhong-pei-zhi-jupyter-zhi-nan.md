---
description: 一起“大战代码”
---

# VS Code中配置 Jupyter 指南

俺在使用 Python 进行数据分析时通常使用 Visual Studio Code (简称：VS Code ~~又名"微软大战代码"~~) ，并在其中配置和使用 Jupyter Notebook 以获得交互式的数据分析体验。

以下是详细的分析环境配置步骤：

{% stepper %}
{% step %}
### 环境准备 (安装必备软件)

在开始配置之前，请确保电脑上已经安装了以下核心软件：

1. **Visual Studio Code**：如果尚未安装，请前往 [VS Code 官网](https://code.visualstudio.com/) 下载并安装适合你操作系统的版本。
2. **Conda (Anaconda 或 Miniconda)**：为了避免不同项目间的 Python 依赖冲突，俺更加推荐使用 Conda 进行环境管理。你可以选择功能全面的 [Anaconda](https://www.anaconda.com/)，或者跟俺一样使用更轻量级的 [Miniconda](https://docs.conda.io/en/latest/miniconda.html)。这里附上清华源的[Miniconda 下载链接](https://mirrors.tuna.tsinghua.edu.cn/anaconda/miniconda/)。安装时，请确保配置好系统环境变量。
{% endstep %}

{% step %}
### 在 VS Code 中安装核心扩展

为了让 VS Code 原生支持 Jupyter 界面和交互操作，需要安装官方提供的扩展包。

1. 打开 VS Code。
2. 点击左侧活动栏的**扩展**图标 (或使用快捷键 `Ctrl+Shift+X` / macOS 为 `Cmd+Shift+X`)。
3. 在搜索框中输入 `Jupyter`，找到由 **Microsoft** 发布的官方扩展并点击**安装**。
4. _注意：通常安装 Jupyter 扩展时，VS Code 会自动为你安装配套的 **Python** 扩展（同样由 Microsoft 提供）。如果未自动安装，请手动搜索“Python”并完成安装。该扩展将为你提供代码高亮、自动补全和强大的调试功能。_
{% endstep %}

{% step %}
### 创建 Conda 环境并安装 Jupyter 内核

这一步是打通 VS Code 与分析项目的关键。俺们需要创建一个独立的 Conda 环境，并安装让 Jupyter 能够识别该环境的“桥梁”程序。

1. 打开系统终端（Windows 用户推荐打开 **Anaconda Prompt**，macOS/Linux 用户直接使用 Terminal）。
2.  **创建专用的 Conda 环境**（这里以环境名为 `data_science_env`，Python 版本为 3.14 为例）：

    ```bash
    conda create -n data_science_env python=3.14
    ```
3.  **激活刚刚创建的环境**：

    ```bash
    conda activate data_science_env
    ```
4.  **安装核心依赖包**：在激活的环境下，安装 `jupyter` 和 `ipykernel`。`ipykernel` 的作用是将当前 Conda 环境注册为一个可供 Jupyter 调用的内核。

    ```bash
    conda install jupyter ipykernel -y
    ```

{% hint style="info" %}
**备选提示**：如果使用的是纯 Python 程序而非 Conda，则可以通过命令 `pip install jupyter ipykernel` 来完成此步骤。
{% endhint %}
{% endstep %}

{% step %}
### 在 VS Code 中创建或打开 Notebook

俺们的底层环境已经搭建完毕，可以开始编写代码了。

1. **创建新的 Notebook**：
   * 在 VS Code 中，按下 `Ctrl+Shift+P` (macOS 为 `Cmd+Shift+P`) 唤出命令面板。
   * 输入并选择命令：**"Create: New Jupyter Notebook"**。
   * 一个全新的、后缀名为 `.ipynb` 的空白笔记本文件就会出现在编辑器中。
2. **打开已有的 Notebook**：
   * 直接点击菜单栏的“文件” > “打开文件”，然后选中本地的 `.ipynb` 文件即可。
{% endstep %}

{% step %}
### 选择内核 (Kernel) —— 绑定你的 Conda 环境

这是最重要的一步！俺们需要告诉 VS Code 使用哪一个 Python 环境来运行代码。

1. 在 Notebook 界面右上角，找到并点击 **"选择内核" (Select Kernel)** 按钮（或者在你尝试运行第一个代码单元格时，VS Code 也会自动弹窗提示）。
2. 在弹出的下拉菜单中，选择 **"Python Environments"**（Python 环境）。
3. VS Code 会扫描系统中所有的 Python 解释器。仔细在列表中寻找带有 **`(conda)`** 标签，且名称为你刚刚创建的 **`data_science_env`** 的选项。
4. **选中该环境**。完成绑定后，VS Code 会记住此设置，下次打开该文件时会自动连接。

{% hint style="info" %}
如果在列表中没有看到新建的 Conda 环境，可以尝试刷新 VS Code 窗口。按下 `Ctrl+Shift+P`，输入并执行 `Developer: Reload Window`，再次点击选择内核即可。
{% endhint %}
{% endstep %}

{% step %}
### 开始高效编码与数据分析

* **运行代码**：点击代码单元格左侧的“▶ 播放”按钮，或者使用快捷键 `Shift+Enter`（运行当前单元格并跳转至下一个）或 `Ctrl+Enter`（运行并停留在当前单元格）。
* **全部运行**：点击编辑器顶部工具栏的“全部运行” (Run All) 按钮，可以一键执行所有单元格。
* **智能代码提示**：得益于 Python 扩展，你可以享受到类似 IDE 级别的智能提示 (IntelliSense)、参数自动补全和语法检查。
* **变量查看器 (Variables View)**：在顶部工具栏点击“变量”按钮。这会在底部面板打开一个实时视图，展示当前 Conda 内核中内存里所有的 DataFrame、列表和变量值，这对于数据清洗和分析来说是非常方便的。
{% endstep %}
{% endstepper %}

***

Learn more:

1. [VS Code 中的Jupyter Notebook 用户入门](https://learn.microsoft.com/zh-tw/shows/visual-studio-code/getting-started-with-jupyter-notebooks-in-vs-code)
