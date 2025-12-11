# WSL+Docker+VScode搭建Unstructured环境

## 环境准备：

- WSL2 - Ubuntu-24.04
- Docker Desktop for Windows
    - 安装时勾选 Enable WSL 2 Windows Feathures
- VScode插件
    - Dev Containers
    - WSL

## 配置docker镜像：

[轩辕镜像](https://docker.xuanyuan.me/#mirror-tutorial)

```json
  "registry-mirrors": [
    "<https://docker.xuanyuan.me>"
  ]

```

直接在docker的Windows客户端配置一下即可

## 拉取Unstructured镜像

[Unstructured官方安装文档](https://docs.unstructured.io/open-source/installation/docker-installation#docker-cli)

进入wsl子系统：Ubuntu-24.04

```bash
docker pull downloads.unstructured.io/unstructured-io/unstructured:latest

```

## 创建项目目录

```bash
mkdir ~/my_unstructured_project

```

## 配置Dev Container

在`~/my_unstructured_project`目录下创建Dev Container配置文件`.devcontainer/devcontainer.json`

```bash
mkdir ~/my_unstructured_project/.devcontainer && vi ~/my_unstructured_project/.devcontainer/devcontainer.json

```

`~/my_unstructured_project/.devcontainer/devcontainer.json`配置文件如下：

```json
{
    "name": "unstructured",

    "image": "downloads.unstructured.io/unstructured-io/unstructured:latest",

    //"remoteUser": "developer",

    // 添加 Docker run 参数，为容器指定固定名称
    "runArgs": [
        "--name=my_unstructured_dev" // 你可以自定义这个名字
    ],

    // 指定容器启动后的工作区文件夹，这里挂载到 /worksapce
    "workspaceFolder": "/worksapce",

    // 配置挂载：将当前项目文件夹从WSL挂载到容器的 /home/developer/worksapce
    "mounts": [
        "source=${localWorkspaceFolder},target=/worksapce,type=bind,consistency=cached"
    ],

    // （可选）容器创建后执行的命令，例如安装额外的开发工具
    // "postCreateCommand": "apt-get update && apt-get install -y git vim",

    // （可选）在容器内安装VS Code扩展，提供更好的开发体验
    "customizations": {
        "vscode": {
            "extensions": [
                "ms-python.python" // Python语言支持
            ]
        }
    }

}

```

## vscode 连接到容器

vscode连接到wsl并打开

```
~/my_unstructured_project/
```

文件夹，右下角自动弹出对话框，

**选择在容器中重新打开**

到此VSCode成功连接到容器

开发环境已经就绪：

终端：VS Code内置的终端已经直接连接到容器内部，可以在其中运行pip list查看已预装的`unstructured` 相关包。

运行代码：在 /workspace 目录下编写的代码将在容器环境中运行。(该目录映射在wsl的`~/my_unstructured_project/`下)

通过 devcontainer.json 的 postCreateCommand 或后续的配置文件实现持久化。

## 使用uv

[uv官方文档](https://docs.astral.sh/uv/)

使用VScode内置的终端，操作容器内部

### 安装uv

```bash
wget -qO- <https://astral.sh/uv/install.sh> | sh

```

### 配置uv PyPI 镜像

[官方配置文档](https://docs.astral.sh/uv/concepts/configuration-files/)[清华源PyPI仓库](https://mirrors.tuna.tsinghua.edu.cn/help/pypi/)

在 `~/.config/uv/uv.toml`(用户级) 或者 `/etc/uv/uv.toml`(系统级) 填写下面的内容:

```
[[index]]
url = "<https://mirrors.tuna.tsinghua.edu.cn/pypi/web/simple/>"
default = true

```

### 创建venv虚拟环境

### 创建项目目录

```bash
mkdir unstructured_project && cd unstructured_project/

```

### 创建虚拟环境

[uv官方操作文档](https://docs.astral.sh/uv/getting-started/features/#the-pip-interface)

```bash
uv venv

```

### 激活虚拟环境

```bash
source .venv/bin/activate

```

## 虚拟环境下安装unstructured[pdf]

因为我要处理pdf文档，所有我安装`"unstructured[pdf]"`

```bash
uv pip install "unstructured[pdf]"

```

右下角解释器要切换成虚拟环境的解释器

到此环境配置完毕

## 提取pdf文本

```python
from unstructured.partition.pdf import partition_pdf

# 使用partition_pdf
elements = partition_pdf(
    filename="baichuan2.pdf",
    strategy="hi_res",  # 或 "ocr_only" 获取图片文本
    extract_images_in_pdf=True,  # 提取图片用于OCR
    infer_table_structure=True,  # 提取表格结构
    include_page_breaks=True,    # 保留分页信息
)

for element in elements:
    if element.metadata.page_number == 3:
        print(element.text)

```

## 最后

此方法也可以用于其他开发环境