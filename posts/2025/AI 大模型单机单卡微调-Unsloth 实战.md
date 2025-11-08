---
title: AI 大模型单机单卡微调-Unsloth 实战
description: 聚焦单机单卡（如 RTX 3090/4090）AI 大模型微调场景，以 Unsloth 工具为核心，涵盖微调技术与工具横向对比、双系统（Ubuntu/Windows）环境搭建、数据集处理、模型配置与预加载、训练执行与监控、模型部署测试及常见问题排查全流程，提供可直接参照的命令与代码示例，助力技术人员实现轻量化高效微调，同时给出适配不同显存的选型建议，平衡显存占用、训练速度与微调效果
date: 2025-11-05
lastmod: 2025-11-06
author: Sherry
avatar: https://s3.aiedu.qzz.io/website/2025/10/c9636914b0782ae3f9269b50dcb88659.png
categories:
  - AI
  - 编程
tags:
  - 技术
  - 经验
series: 开发指南
image: https://s3.aiedu.qzz.io/website/2025/11/95e07365779a7604d919d4383b83ac0b.png
---

本文聚焦单机单卡硬件场景，围绕 Unsloth 工具展开 AI 大模型微调技术实践，覆盖从环境搭建、数据处理、模型配置、训练执行到部署测试的全流程。手册内容以实操为核心，包含详细命令、代码示例及问题排查方案，适用于需要在消费级 GPU（如 RTX 3090/4090）上实现轻量化微调的技术人员，同时提供主流微调技术与工具的横向对比，为技术选型提供参考。

## 一、微调核心技术与工具横向对比

### 1.1 核心微调技术对比

| 技术类型           | 显存占用 | 训练速度 | 微调效果 | 硬件门槛 | 适用场景              |
| ------------------ | -------- | -------- | -------- | -------- | --------------------- |
| 全参数微调         | 极高     | 慢       | 最佳     | 80GB+    | 企业级高精度需求      |
| LoRA（低秩适配）   | 中       | 中       | 优良     | 24GB+    | 通用场景任务定制      |
| QLoRA（量化 LoRA） | 低       | 中       | 良好     | 16GB+    | 消费级 GPU 轻量化微调 |
| Unsloth 全参数微调 | 中低     | 极快     | 最佳     | 24GB+    | 单机单卡高精度需求    |
| Adapter 微调       | 低       | 快       | 一般     | 12GB+    | 简单任务快速适配      |

#### 关键技术解析

- **Unsloth 优化机制**：通过动态量化、稀疏优化、梯度检查点改进、激活值卸载四大技术，将全参数微调显存占用降低 75%，训练速度提升 30 倍，无需插入 Adapter 层即可直接训练全部参数。

- **QLoRA 局限性**：4-bit 量化可能导致精度损失，复杂任务（如医疗问答、企业知识库）表现弱于 Unsloth 全参数微调。

- **Unsloth 全参数微调优势**：通过优化器状态分页（Paged Optimizer）将优化器状态分页至 CPU，24GB 显存可支持 7B 模型全参数微调。

### 1.2 主流微调工具性能评测

| 工具名称             | 核心优势                | 单机单卡适配性 | 支持技术类型           | 易用性 | 训练速度（相对值） | 显存优化率 |
| -------------------- | ----------------------- | -------------- | ---------------------- | ------ | ------------------ | ---------- |
| Unsloth              | 极致显存优化 + 超高速度 | 极佳           | LoRA/QLoRA/ 全参数微调 | 高     | 30 倍              | 75%        |
| PEFT（Hugging Face） | 生态完善 + 兼容性强     | 一般           | LoRA/Adapter/QLoRA     | 中     | 1 倍               | 30%        |
| Axolotl              | 配置化操作 + 多平台集成 | 良好           | 全参数 / LoRA/QLoRA    | 极高   | 1.2 倍             | 40%        |
| LoRAX                | 推理优化 + 低延迟部署   | 一般           | LoRA 系列              | 中     | 1.5 倍             | 35%        |

> 测试环境：RTX 4090（24GB），微调 Llama 3.2-1B 模型，序列长度 2048，batch size=1

数据来源：Unsloth 社区实测与官方基准测试

#### 工具选型建议

- **优先选 Unsloth**：单机单卡场景下，兼顾速度、显存占用和微调效果的最优解。

- **次选 Axolotl**：需要多模型支持或配置化操作（无需编写大量代码）时选择。

- **谨慎选 PEFT**：仅当需要与 Hugging Face 生态（如 Transformers Pipeline）深度集成时考虑。

## 二、系统环境搭建（Ubuntu 22.04/Windows 11 双系统适配）

### 2.1 Ubuntu 22.04 系统配置

#### 2.1.1 显卡驱动安装（RTX 3090/4090 示例）

```shell
# 1. 卸载旧驱动（避免版本冲突）
sudo apt purge nvidia-*
sudo apt autoremove

# 2. 添加NVIDIA官方驱动源
sudo add-apt-repository ppa:graphics-drivers/ppa
sudo apt update

# 3. 查看推荐驱动版本（需≥535.xx，确保兼容CUDA 11.8）
ubuntu-drivers devices
# 输出示例：recommended: nvidia-driver-550-server（选择推荐版本）

# 4. 安装推荐驱动
sudo apt install nvidia-driver-550-server

# 5. 重启系统生效
sudo reboot

# 6. 验证驱动安装（显示GPU型号、驱动版本、CUDA版本）
nvidia-smi
# 预期输出：Driver Version: 550.xx，CUDA Version: 12.4（仅显示，非实际安装）
```

#### 2.1.2 CUDA Toolkit 11.8 安装（Unsloth 最佳兼容版本）

```shell
# 1. 下载CUDA 11.8安装脚本
wget https://developer.download.nvidia.com/compute/cuda/11.8.0/local_installers/cuda_11.8.0_520.61.05_linux.run

# 2. 赋予脚本执行权限
chmod +x cuda_11.8.0_520.61.05_linux.run

# 3. 安装CUDA Toolkit（仅安装工具包，不重复安装驱动）
sudo ./cuda_11.8.0_520.61.05_linux.run --toolkit --override

# 4. 配置CUDA环境变量（写入.bashrc实现持久化）
echo 'export PATH=/usr/local/cuda-11.8/bin:$PATH' >> ~/.bashrc
echo 'export LD_LIBRARY_PATH=/usr/local/cuda-11.8/lib64:$LD_LIBRARY_PATH' >> ~/.bashrc
source ~/.bashrc

# 5. 验证CUDA安装（显示版本11.8）
nvcc -V
# 预期输出：release 11.8, V11.8.89
```

### 2.2 Windows 11 系统配置

#### 2.2.1 显卡驱动与 CUDA 安装

1. **驱动下载**：访问 NVIDIA 官网（[https://www.nvidia](https://www.nvidia.com/)[.com/](https://www.nvidia.com/)），选择 GPU 型号（如 RTX 4090），下载 Windows 驱动（版本≥535.xx）。

2. **驱动安装**：双击安装程序，选择 “精简安装”，完成后重启电脑。

3. **CUDA 11.8 安装**：

- - 下载地址：[https://developer.nvidi](https://developer.nvidia.com/cuda-11-8-0-download-archive?target_os=Windows&target_arch=x86_64&target_version=11&target_type=exe_local)[a.com](https://developer.nvidia.com/cuda-11-8-0-download-archive?target_os=Windows&target_arch=x86_64&target_version=11&target_type=exe_local)[/cuda](https://developer.nvidia.com/cuda-11-8-0-download-archive?target_os=Windows&target_arch=x86_64&target_version=11&target_type=exe_local)[-11-8](https://developer.nvidia.com/cuda-11-8-0-download-archive?target_os=Windows&target_arch=x86_64&target_version=11&target_type=exe_local)[-0-do](https://developer.nvidia.com/cuda-11-8-0-download-archive?target_os=Windows&target_arch=x86_64&target_version=11&target_type=exe_local)[wnloa](https://developer.nvidia.com/cuda-11-8-0-download-archive?target_os=Windows&target_arch=x86_64&target_version=11&target_type=exe_local)[d-arc](https://developer.nvidia.com/cuda-11-8-0-download-archive?target_os=Windows&target_arch=x86_64&target_version=11&target_type=exe_local)[hive](https://developer.nvidia.com/cuda-11-8-0-download-archive?target_os=Windows&target_arch=x86_64&target_version=11&target_type=exe_local)[?targe](https://developer.nvidia.com/cuda-11-8-0-download-archive?target_os=Windows&target_arch=x86_64&target_version=11&target_type=exe_local)[t_os=](https://developer.nvidia.com/cuda-11-8-0-download-archive?target_os=Windows&target_arch=x86_64&target_version=11&target_type=exe_local)[Windo](https://developer.nvidia.com/cuda-11-8-0-download-archive?target_os=Windows&target_arch=x86_64&target_version=11&target_type=exe_local)[ws&ta](https://developer.nvidia.com/cuda-11-8-0-download-archive?target_os=Windows&target_arch=x86_64&target_version=11&target_type=exe_local)[rget_](https://developer.nvidia.com/cuda-11-8-0-download-archive?target_os=Windows&target_arch=x86_64&target_version=11&target_type=exe_local)[arch=](https://developer.nvidia.com/cuda-11-8-0-download-archive?target_os=Windows&target_arch=x86_64&target_version=11&target_type=exe_local)[x86_6](https://developer.nvidia.com/cuda-11-8-0-download-archive?target_os=Windows&target_arch=x86_64&target_version=11&target_type=exe_local)[4&tar](https://developer.nvidia.com/cuda-11-8-0-download-archive?target_os=Windows&target_arch=x86_64&target_version=11&target_type=exe_local)[get_v](https://developer.nvidia.com/cuda-11-8-0-download-archive?target_os=Windows&target_arch=x86_64&target_version=11&target_type=exe_local)[ersio](https://developer.nvidia.com/cuda-11-8-0-download-archive?target_os=Windows&target_arch=x86_64&target_version=11&target_type=exe_local)[n=11&](https://developer.nvidia.com/cuda-11-8-0-download-archive?target_os=Windows&target_arch=x86_64&target_version=11&target_type=exe_local)[targe](https://developer.nvidia.com/cuda-11-8-0-download-archive?target_os=Windows&target_arch=x86_64&target_version=11&target_type=exe_local)[t_typ](https://developer.nvidia.com/cuda-11-8-0-download-archive?target_os=Windows&target_arch=x86_64&target_version=11&target_type=exe_local)[e=exe](https://developer.nvidia.com/cuda-11-8-0-download-archive?target_os=Windows&target_arch=x86_64&target_version=11&target_type=exe_local)[_loca](https://developer.nvidia.com/cuda-11-8-0-download-archive?target_os=Windows&target_arch=x86_64&target_version=11&target_type=exe_local)[l](https://developer.nvidia.com/cuda-11-8-0-download-archive?target_os=Windows&target_arch=x86_64&target_version=11&target_type=exe_local)

- - 运行安装程序，选择 “自定义安装”，仅勾选 “CUDA Toolkit 11.8”，取消勾选 “NVIDIA GeForce Experience”，安装路径默认（C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v11.8）。

1. **验证**：

- - 打开命令提示符（CMD），输入nvidia-smi，查看驱动版本与 CUDA 版本。

- - 输入nvcc -V，确认输出 CUDA 11.8 版本信息。

#### 2.2.2 WSL2 环境配置（推荐 Windows 用户，兼容 Linux 工具链）

```shell
# 1. 管理员身份打开PowerShell，启用WSL2
wsl --install
# 安装完成后重启电脑

# 2. 安装Ubuntu 22.04子系统
wsl --install -d Ubuntu-22.04
# 首次启动设置Ubuntu用户名和密码

# 3. 进入WSL2环境（后续操作同Ubuntu系统）
wsl
```

### 2.3 Python 环境与依赖配置

#### 2.3.1 虚拟环境创建（避免依赖冲突）

```shell
# 1. 安装virtualenv（Ubuntu/WSL/Windows通用）
pip install virtualenv

# 2. 创建虚拟环境（命名为unsloth-env，指定Python 3.10）
virtualenv unsloth-env -p python3.10
# 注：Unsloth推荐Python 3.10，Ubuntu需提前安装：sudo apt install python3.10 python3.10-venv

# 3. 激活虚拟环境
# Ubuntu/WSL
source unsloth-env/bin/activate
# Windows CMD
unsloth-env\Scripts\activate.bat
# Windows PowerShell
.\unsloth-env\Scripts\Activate.ps1
# 激活后终端前缀显示：(unsloth-env)
```

#### 2.3.2 依赖精确安装（版本锁定，确保兼容性）

```js
# 1. 创建依赖清单文件（requirements.txt）
cat > requirements.txt << EOF
unsloth==2024.5
transformers==4.41.2
peft==0.11.1
accelerate==0.30.1
bitsandbytes==0.43.0
datasets==2.19.1
tokenizers==0.19.1
torch==2.2.2+cu118
torchvision==0.17.2+cu118
torchaudio==2.2.2+cu118
wandb==0.16.6
tensorboard==2.16.2
jsonlines==4.0.0
sentencepiece==0.2.0
EOF

# 2. 安装依赖（指定CUDA 11.8版本的PyTorch）
pip install -r requirements.txt --extra-index-url https://download.pytorch.org/whl/cu118

# 3. 验证依赖安装
# 检查Unsloth版本
python -c "import unsloth; print(f'Unsloth version: {unsloth.__version__}')"
# 检查PyTorch与CUDA关联（需显示True和11.8）
python -c "import torch; print(f'Torch version: {torch.__version__}'); print(f'Torch CUDA available: {torch.cuda.is_available()}'); print(f'Torch CUDA version: {torch.version.cuda}')"
```

## 三、数据集本地化处理

### 3.1 数据集结构与格式

#### 3.1.1 目录结构

```shell
dataset/
├── train.json  # 训练集（建议≥100条样本，保证微调效果）
└── test.json   # 测试集（约为训练集规模的20%，用于验证）
```

#### 3.1.2 样本格式（ChatML 标准，以医疗问答为例）

```js
[
  {
    "conversations": [
      {"role": "system", "content": "你是专业医疗问答助手，回答需基于医学常识，避免误导用户"},
      {"role": "user", "content": "糖尿病患者可以吃水果吗？"},
      {"role": "assistant", "content": "糖尿病患者可以适量吃水果，但需注意以下几点：1. 选择低GI（血糖生成指数）水果，如苹果、梨、柚子、草莓等，避免高GI水果如西瓜、荔枝、榴莲；2. 控制食用量，每天建议200-300克，分次食用，避免一次性大量摄入；3. 食用时间选择两餐之间（如上午10点、下午3点），避免餐后立即食用，防止血糖叠加升高；4. 食用水果后需监测血糖，根据自身血糖变化调整种类和量；5. 若血糖控制不佳（空腹血糖>8mmol/L或餐后2小时血糖>11.1mmol/L），建议暂时避免食用水果，优先控制血糖。"}
    ]
  },
  {
    "conversations": [
      {"role": "system", "content": "你是专业医疗问答助手，回答需基于医学常识，避免误导用户"},
      {"role": "user", "content": "感冒时应该多喝水吗？"},
      {"role": "assistant", "content": "感冒时建议适量多喝水，原因如下：1. 补充水分：感冒常伴随发热、出汗，多喝水可补充身体流失的水分，防止脱水；2. 稀释痰液：有助于稀释呼吸道内的痰液，促进痰液排出，缓解咳嗽、鼻塞等症状；3. 调节体温：通过出汗散热，帮助降低体温；4. 促进代谢：加速身体新陈代谢，帮助排出体内毒素和病毒。但需注意：1. 饮水量适中，一般成年人每天1500-2000毫升即可，避免过量饮水增加肾脏负担；2. 特殊人群如心肾功能不全者，需根据医生建议控制饮水量；3. 可选择温开水、淡盐水或蜂蜜水（1岁以下婴儿禁用），避免饮用含糖量高的饮料。"}
    ]
  }
]
```

### 3.2 数据集加载与清洗代码

```python
from datasets import Dataset
import jsonlines
import json

def load_local_json_dataset(train_path, test_path):
    """加载本地JSON数据集并清洗格式错误样本"""
    # 读取训练集
    train_data = []
    with jsonlines.open(train_path, mode='r') as reader:
        for obj in reader:
            train_data.append(obj)
    # 读取测试集
    test_data = []
    with jsonlines.open(test_path, mode='r') as reader:
        for obj in reader:
            test_data.append(obj)
    
    # 数据清洗：过滤缺少关键角色（system/user/assistant）或空内容的样本
    def clean_data(raw_data):
        cleaned = []
        for item in raw_data:
            conv = item.get("conversations", [])
            roles = [turn.get("role") for turn in conv]
            # 检查是否包含所有关键角色
            if {"system", "user", "assistant"}.issubset(set(roles)):
                # 检查所有content非空
                if all(turn.get("content", "").strip() != "" for turn in conv):
                    cleaned.append({"conversations": conv})
        return cleaned
    
    cleaned_train = clean_data(train_data)
    cleaned_test = clean_data(test_data)
    
    # 转换为datasets.Dataset格式（便于后续处理）
    train_dataset = Dataset.from_list(cleaned_train)
    test_dataset = Dataset.from_list(cleaned_test)
    
    return {"train": train_dataset, "test": test_dataset}

# 加载数据集（替换为实际路径）
dataset = load_local_json_dataset(
    train_path="./dataset/train.json",
    test_path="./dataset/test.json"
)

# 验证加载结果
print(f"训练集样本数量：{len(dataset['train'])}")
print(f"测试集样本数量：{len(dataset['test'])}")
print(f"训练集第一条样本：\n{json.dumps(dataset['train'][0], indent=2, ensure_ascii=False)}")
```

### 3.3 企业知识库场景数据增强（可选）

```python
from unsloth.dataprep.synthetic import prepare_qa_generation
from unsloth.chat_templates import apply_chat_template

# 从企业文档生成QA样本（支持TXT/PDF格式）
dataset = prepare_qa_generation(
    output_folder="company_kb_dataset",  # 输出目录
    max_generation_tokens=512,           # 单条回答最大长度
    temperature=0.7,                     # 生成随机性（0-1）
    overlap=64,                          # 文档片段重叠度（避免信息丢失）
    default_num_pairs=25,                # 每个文档生成25组QA
    cleanup_threshold=0.85               # 过滤低质量样本（0-1）
)

# 应用ChatML模板统一格式
dataset = apply_chat_template(
    dataset,
    tokenizer=tokenizer,  # 后续加载的模型Tokenizer
    chat_template="chatml",
    default_system_message="你是公司内部知识库助手，仅回答与公司业务相关的问题"
)
```

## 四、模型配置与预加载

### 4.1 模型缓存路径设置（避免重复下载）

```shell
# Ubuntu/WSL：设置缓存路径（替换为实际路径）
export TRANSFORMERS_CACHE="/home/your_username/.cache/huggingface/hub"
# 持久化配置（写入.bashrc）
echo "export TRANSFORMERS_CACHE=$TRANSFORMERS_CACHE" >> ~/.bashrc
source ~/.bashrc

# Windows CMD：设置缓存路径（替换为实际路径）
set TRANSFORMERS_CACHE=D:\huggingface_cache
# 持久化配置（管理员身份）
setx TRANSFORMERS_CACHE "%TRANSFORMERS_CACHE%" /M

# Windows PowerShell：设置缓存路径
$env:TRANSFORMERS_CACHE="D:\huggingface_cache"
# 持久化配置
[Environment]::SetEnvironmentVariable("TRANSFORMERS_CACHE", "D:\huggingface_cache", "Machine")
```

### 4.2 模型预加载（提前下载权重，节省训练时间）

```python
from unsloth import FastLanguageModel
from transformers import AutoTokenizer
import os

# 选择模型（以Llama 3.2-1B为例，24GB显存可支持）
model_name = "meta-llama/Llama-3.2-1B-Instruct"

# 1. 下载Tokenizer
tokenizer = AutoTokenizer.from_pretrained(model_name)
# 保存Tokenizer到本地（可选）
tokenizer.save_pretrained("./llama-3.2-1b-tokenizer")
print(f"Tokenizer保存路径：./llama-3.2-1b-tokenizer")

# 2. 下载模型权重（加载到CPU，不占用GPU显存）
model = FastLanguageModel.from_pretrained(
    model_name=model_name,
    max_seq_length=2048,       # 序列最大长度
    load_in_4bit=False,        # 不启用量化（仅下载）
    device_map="cpu",          # 加载到CPU
    cache_dir=os.getenv("TRANSFORMERS_CACHE")  # 使用自定义缓存路径
)

# 3. 保存模型到本地（可选，后续直接加载）
model.save_pretrained("./llama-3.2-1b-local")
print(f"模型保存路径：./llama-3.2-1b-local")

# 后续加载本地模型示例
# model, tokenizer = FastLanguageModel.from_pretrained(
#     model_name="./llama-3.2-1b-local",
#     max_seq_length=2048,
#     load_in_4bit=True,
#     device_map="auto"
# )
```

### 4.3 模型加载配置（分场景适配显存）

#### 4.3.1 16GB 显存配置（如 RTX 4080，QLoRA 微调）

```python
from unsloth import FastLanguageModel

model, tokenizer = FastLanguageModel.from_pretrained(
    model_name="meta-llama/Llama-3.2-1B-Instruct",  # 或本地路径
    max_seq_length=2048,
    load_in_4bit=True,                  # 启用4-bit量化（降低显存占用）
    use_gradient_checkpointing="unsloth",# Unsloth优化版梯度检查点
    max_lora_rank=16,                   # 降低LoRA秩，减少显存使用
    device_map="auto"
)

# 配置LoRA微调
model = FastLanguageModel.get_peft_model(
    model,
    r=16,                               # LoRA秩（与max_lora_rank一致）
    target_modules=("q_proj", "k_proj", "v_proj", "o_proj", "gate_proj", "up_proj", "down_proj"),
    lora_alpha=32,                      # LoRA缩放因子
    lora_dropout=0.05,                  #  dropout比例（防止过拟合）
    bias="none",
    random_state=3407                   # 固定随机种子（保证可复现）
)
```

#### 4.3.2 24GB 显存配置（如 RTX 4090，全参数微调）

```python
from unsloth import FastLanguageModel

model, tokenizer = FastLanguageModel.from_pretrained(
    model_name="mistralai/Mistral-7B-Instruct-v0.3",  # 7B模型，全参数微调
    max_seq_length=2048,
    load_in_8bit=True,                  # 8-bit量化（平衡精度与显存）
    use_gradient_checkpointing="unsloth",
    optimizer="paged_adamw_8bit",       # 分页优化器（降低显存占用）
    device_map="auto"
)

# 全参数微调无需PEFT配置，直接切换训练模式
model.train()
```

## 五、微调实战流程

### 5.1 训练参数配置

```python
from transformers import TrainingArguments, Trainer

training_args = TrainingArguments(
    output_dir="./unsloth_finetune",    # 训练结果输出目录
    per_device_train_batch_size=1,      # 单卡batch size（16GB显存建议1，24GB可尝试2）
    gradient_accumulation_steps=8,      # 梯度累积（等效batch size=1*8=8）
    learning_rate=2e-5,                 # 学习率（全参数微调建议1e-5~3e-5）
    num_train_epochs=3,                 # 训练轮次（避免过拟合）
    logging_steps=10,                   # 日志输出间隔（每10步打印一次损失）
    save_steps=100,                     # 模型保存间隔（每100步保存一次）
    fp16=True,                          # 启用混合精度训练（加速且省显存）
    optim="paged_adamw_8bit",           # 8-bit优化器（降低显存占用）
    report_to="none",                   # 暂不启用监控工具（后续单独配置）
    evaluation_strategy="steps",        # 按步验证
    eval_steps=100,                     # 验证间隔（每100步验证一次）
    load_best_model_at_end=True         # 训练结束加载最优模型
)
```

### 5.2 启动训练

```python
# 初始化Trainer
trainer = Trainer(
    model=model,
    args=training_args,
    train_dataset=dataset["train"],
    eval_dataset=dataset["test"],
)

# 开始训练
print("训练启动，日志将按配置间隔输出...")
trainer.train()

# 保存最终模型
trainer.save_model("./fine_tuned_model")
tokenizer.save_pretrained("./fine_tuned_model")
print(f"微调完成，模型保存路径：./fine_tuned_model")
```

## 六、训练监控工具配置

### 6.1 TensorBoard 配置

```python
from torch.utils.tensorboard import SummaryWriter

# 创建日志目录
tb_writer = SummaryWriter(log_dir="./unsloth_finetune/tensorboard")

# 定义监控回调（记录损失、学习率）
def tb_callback(trainer, args, state, control, **kwargs):
    if state.global_step % args.logging_steps == 0:
        # 记录训练损失
        train_loss = state.log_history[-1].get("loss", 0.0)
        tb_writer.add_scalar("Train/Loss", train_loss, state.global_step)
        # 记录学习率
        lr = trainer.optimizer.param_groups[0]["lr"]
        tb_writer.add_scalar("Train/Learning_Rate", lr, state.global_step)
        # 记录验证损失（若有）
        if "eval_loss" in state.log_history[-1]:
            eval_loss = state.log_history[-1]["eval_loss"]
            tb_writer.add_scalar("Eval/Loss", eval_loss, state.global_step)

# 重新初始化Trainer（添加回调）
trainer = Trainer(
    model=model,
    args=training_args,
    train_dataset=dataset["train"],
    eval_dataset=dataset["test"],
    callbacks=[tb_callback]
)

# 启动训练后，新终端执行TensorBoard（替换为实际路径）
# tensorboard --logdir ./unsloth_finetune/tensorboard
# 浏览器访问：http://localhost:6006
```

### 6.2 WandB 配置

```python
# 1. 安装并登录WandB（终端执行）
pip install wandb
wandb login
# 按提示输入API密钥（从WandB官网：https://wandb.ai/settings获取）
import wandb

# 初始化WandB项目
wandb.init(
    project="unsloth-single-gpu-finetune",  # 项目名称（自定义）
    name="llama-3.2-1b-medical-qa",         # 实验名称（自定义）
    config={                                # 记录实验参数（便于复现）
        "model_name": "meta-llama/Llama-3.2-1B-Instruct",
        "batch_size": 1,
        "gradient_accumulation_steps": 8,
        "learning_rate": 2e-5,
        "epochs": 3,
        "max_seq_length": 2048
    }
)

# 配置TrainingArguments（启用WandB报告）
training_args.report_to = "wandb"

# 重新初始化Trainer
trainer = Trainer(
    model=model,
    args=training_args,
    train_dataset=dataset["train"],
    eval_dataset=dataset["test"],
)

# 启动训练
trainer.train()

# 训练结束后关闭WandB
wandb.finish()
# 浏览器访问WandB项目页面查看监控：https://wandb.ai/你的用户名/unsloth-single-gpu-finetune
```

## 七、模型部署与测试

### 7.1 本地推理测试

```python
from unsloth import FastLanguageModel
import torch

# 加载微调后的模型
model, tokenizer = FastLanguageModel.from_pretrained(
    model_name="./fine_tuned_model",  # 微调后模型路径
    max_seq_length=2048,
    load_in_4bit=True,
    device_map="auto"
)

def generate_response(prompt, system_prompt="你是专业医疗问答助手"):
    """生成模型回复"""
    # 构建对话格式
    messages = [
        {"role": "system", "content": system_prompt},
        {"role": "user", "content": prompt}
    ]
    
    # 编码输入
    inputs = tokenizer.apply_chat_template(
        messages,
        add_generation_prompt=True,
        return_tensors="pt"
    ).to(model.device)
    
    # 生成回复
    outputs = model.generate(
        inputs,
        max_new_tokens=512,  # 单条回复最大长度
        temperature=0.7,     # 随机性（0-1，越低越稳定）
        top_p=0.9,           # 采样阈值（过滤低概率词）
        do_sample=True,      # 启用采样生成（避免重复）
        pad_token_id=tokenizer.eos_token_id
    )
    
    # 解码输出（去除特殊字符）
    response = tokenizer.decode(outputs[0], skip_special_tokens=True)
    # 提取assistant回复
    assistant_response = response.split("assistant")[-1].strip()
    return assistant_response

# 测试推理
test_prompts = [
    "高血压患者可以吃鸡蛋吗？",
    "感冒发烧到38.5℃需要吃退烧药吗？"
]

for prompt in test_prompts:
    print(f"用户：{prompt}")
    response = generate_response(prompt)
    print(f"助手：{response}\n")
```

### 7.2 Flask API 服务部署（支持 HTTP 调用）

```python
# 安装Flask依赖
pip install flask flask-cors
from flask import Flask, request, jsonify
from flask_cors import CORS
from unsloth import FastLanguageModel
import torch

# 初始化Flask应用
app = Flask(__name__)
CORS(app)  # 允许跨域请求（前端调用）

# 加载微调后的模型（服务启动时加载）
model, tokenizer = FastLanguageModel.from_pretrained(
    model_name="./fine_tuned_model",
    max_seq_length=2048,
    load_in_4bit=True,
    device_map="auto"
)

# 定义推理接口（POST方法）
@app.route("/generate", methods=["POST"])
def generate():
    try:
        # 获取请求参数
        data = request.json
        prompt = data.get("prompt", "")
        system_prompt = data.get("system_prompt", "你是专业医疗问答助手")
        max_new_tokens = data.get("max_new_tokens", 512)
        temperature = data.get("temperature", 0.7)
        
        # 校验参数
        if not prompt.strip():
            return jsonify({"code": 400, "message": "prompt不能为空"}), 400
        
        # 生成回复
        messages = [
            {"role": "system", "content": system_prompt},
            {"role": "user", "content": prompt}
        ]
        
        inputs = tokenizer.apply_chat_template(
            messages,
            add_generation_prompt=True,
            return_tensors="pt"
        ).to(model.device)
        
        outputs = model.generate(
            inputs,
            max_new_tokens=max_new_tokens,
            temperature=temperature,
            top_p=0.9,
            do_sample=True,
            pad_token_id=tokenizer.eos_token_id
        )
        
        response = tokenizer.decode(outputs[0], skip_special_tokens=True)
        assistant_response = response.split("assistant")[-1].strip()
        
        # 返回结果
        return jsonify({
            "code": 200,
            "message": "success",
            "data": {
                "prompt": prompt,
                "response": assistant_response
            }
        })
    
    except Exception as e:
        # 异常处理
        return jsonify({
            "code": 500,
            "message": f"服务器错误：{str(e)}"
        }), 500

# 启动服务（生产环境建议用Gunicorn，此处为测试用）
if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000, debug=False)

# API测试命令（终端执行，替换为实际参数）
# curl -X POST http://localhost:5000/generate \
# -H "Content-Type: application/json" \
# -d '{"prompt":"高血压患者可以吃鸡蛋吗？", "max_new_tokens":300, "temperature":0.6}'
```

### 7.3 导出为 GGUF 格式（适配 Ollama 部署）

```shell
# 导出GGUF格式（支持Ollama、 llama.cpp等工具）
model.save_pretrained_gguf(
    "./exported_model_gguf",  # 输出目录
    tokenizer,
    quantization_method="q4_k_m"  # 量化方式（平衡精度与速度）
)

# Ollama部署命令（终端执行）
# 1. 创建Modelfile
# FROM ./exported_model_gguf/your_model.gguf
# 2. 构建Ollama模型
# ollama create medical-qa -f Modelfile
# 3. 启动对话
# ollama run medical-qa
```

## 八、常见问题排查与优化

### 8.1 依赖安装问题

| 问题现象                          | 解决方案                                        | 操作命令                                                     |
| --------------------------------- | ----------------------------------------------- | ------------------------------------------------------------ |
| bitsandbytes 安装报错（Windows）  | 安装 Windows 适配版本                           | pip install bitsandbytes-windows==0.43.0                     |
| PyTorch 与 CUDA 版本不匹配        | 卸载后重新安装对应版本                          | pip uninstall torch torchvision torchaudio; pip install torch==2.2.2+cu118 torchvision==0.17.2+cu118 torchaudio==2.2.2+cu118 --extra-index-url https://download.pytorch.org/whl/cu118 |
| Unsloth 安装报错（CUDA 版本过低） | 升级 CUDA 到 11.8+，或安装 CPU 版本（仅测试用） | pip install unsloth[cpu]==2024.5                             |

### 8.2 GPU 与显存问题

| 问题现象                                       | 解决方案                                  | 操作命令 / 配置                                              |
| ---------------------------------------------- | ----------------------------------------- | ------------------------------------------------------------ |
| nvidia-smi 命令找不到                          | 重新安装显卡驱动                          | Ubuntu：sudo apt reinstall nvidia-driver-550-server; Windows：重新运行 NVIDIA 驱动安装程序 |
| CUDA 版本不匹配（nvcc 与 nvidia-smi 显示不同） | 配置环境变量指定 CUDA 11.8                | Ubuntu：echo 'export PATH=/usr/local/cuda-11.8/bin:$PATH' >> ~/.bashrc; source ~/.bashrc |
| 训练中途 OOM（显存不足）                       | 启用 4-bit 量化 + 梯度检查点 + 优化器分页 | 模型加载时设置 load_in_4bit=True、use_gradient_checkpointing="unsloth"、optim="paged_adamw_8bit" |
| batch size 无法增大                            | 增加梯度累积步数                          | training_args.gradient_accumulation_steps=16（等效 batch size=1*16=16） |

### 8.3 模型与数据问题

| 问题现象                             | 解决方案                    | 操作命令 / 配置                                              |
| ------------------------------------ | --------------------------- | ------------------------------------------------------------ |
| 模型下载速度慢 / 超时                | 使用国内镜像源              | pip config set global.index-url https://mirrors.aliyun.com/pypi/simple/; export HF_ENDPOINT=https://hf-mirror.com（Ubuntu/WSL） |
| Llama 系列模型权限不足               | 申请权限并登录 Hugging Face | 1. 访问https://ai.meta.com/resources/models-and-libraries/llama-downloads/申请；2. huggingface-cli login |
| 训练过拟合（训练损失低，验证损失高） | 增加数据量 + 调整正则化参数 | 1. 使用 prepare_qa_generation 生成更多样本；2. 模型配置中设置 lora_dropout=0.1 |
| 数据格式错误（训练报错）             | 重新检查 JSON 格式并清洗    | 使用 load_local_json_dataset 函数中的 clean_data 逻辑过滤错误样本 |

## 九、总结与选型建议

### 9.1 核心结论

1. **Unsloth 是单机单卡最优选择**：在显存优化（降低 75%）、训练速度（提升 30 倍）和微调效果上，显著优于 PEFT、Axolotl 等工具，24GB 显存可支持 7B 模型全参数微调。

2. **技术选型原则**：

- - 16GB 显存：优先 QLoRA 微调（Llama 3.2-1B/Phi-4-Mini）。

- - 24GB 显存：推荐 Unsloth 全参数微调（Mistral-7B/Llama 3.2-1B）。

- - 80GB + 显存：全参数微调（Llama 3.2-70B，需调整 batch size 与梯度累积）。

### 9.2 扩展建议

1. **多模态微调**：使用 Unsloth 分离训练策略，冻结视觉编码器，仅微调语言解码器（如 LLaVA 系列模型）。

2. **推理优化**：导出 GGUF 格式适配 Ollama，推理速度提升 30%；或使用 TensorRT 加速（需额外配置）。

3. **持续迭代**：关注 Unsloth 社区更新（如动态批处理、混合量化），定期同步工具版本以获取性能优化。