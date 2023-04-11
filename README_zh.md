# ChatGLM Efficient Tuning

![GitHub Repo stars](https://img.shields.io/github/stars/hiyouga/ChatGLM-Efficient-Tuning?style=social)
![GitHub Code License](https://img.shields.io/github/license/hiyouga/ChatGLM-Efficient-Tuning)
![GitHub last commit](https://img.shields.io/github/last-commit/hiyouga/ChatGLM-Efficient-Tuning)

基于 🤗[PEFT](https://github.com/huggingface/peft) 的高效 🤖[ChatGLM-6B](https://github.com/THUDM/ChatGLM-6B) 微调。

\[ [English](README.md) | 中文 \]

## 数据集

目前我们实现了针对以下数据集的支持：

- [Stanford Alpaca](https://github.com/tatsu-lab/stanford_alpaca)
- [Stanford Alpaca (Chinese)](https://github.com/ymcui/Chinese-LLaMA-Alpaca)
- [GPT-4 Generated Data](https://github.com/Instruction-Tuning-with-GPT-4/GPT-4-LLM)
- [BELLE 2M](https://huggingface.co/datasets/BelleGroup/train_2M_CN)
- [BELLE 1M](https://huggingface.co/datasets/BelleGroup/train_1M_CN)
- [BELLE 0.5M](https://huggingface.co/datasets/BelleGroup/train_0.5M_CN)
- [BELLE Dialogue 0.4M](https://huggingface.co/datasets/BelleGroup/generated_chat_0.4M)
- [BELLE School Math 0.25M](https://huggingface.co/datasets/BelleGroup/school_math_0.25M)
- [BELLE Multiturn Chat 0.8M](https://huggingface.co/datasets/BelleGroup/multiturn_chat_0.8M)
- [Guanaco Dataset](https://huggingface.co/datasets/JosephusCheung/GuanacoDataset)
- [Firefly 1.1M](https://huggingface.co/datasets/YeungNLP/firefly-train-1.1M)

使用方法请参考 `config_data.py`。

[23/04/11] 现在我们实现了数据集组合训练！请尝试使用 `dataset1,dataset2` 参数进行组合训练。

## 微调方法

目前我们实现了针对以下高效微调方法的支持：

- [P-Tuning V2](https://github.com/THUDM/P-tuning-v2)
- [LoRA](https://arxiv.org/abs/2106.09685)

## 软件依赖

- Python 3.10, PyTorch 2.0.0
- 🤗Transformers, Datasets, PEFT
- protobuf, cpm_kernels, sentencepiece

以及 **强而有力的 GPU**！

## 如何使用

### 环境搭建（可跳过）

```bash
git clone https://github.com/hiyouga/ChatGLM-Efficient-Tuning.git
conda create -n cet python=3.10
conda activate cet
pip install -r requirements.txt
```

### 微调训练

```bash
CUDA_VISIBLE_DEVICES=0 python finetune_chatglm.py \
    --do_train \
    --dataset alpaca_gpt4_zh \
    --finetuning_type lora \
    --output_dir output \
    --overwrite_cache \
    --overwrite_output_dir \
    --per_device_train_batch_size 4 \
    --gradient_accumulation_steps 4 \
    --lr_scheduler_type cosine \
    --logging_steps 10 \
    --save_steps 1000 \
    --warmup_steps 100 \
    --max_train_samples 10000 \
    --learning_rate 5e-5 \
    --num_train_epochs 1.0 \
    --fp16
```

### 测试效果

```bash
CUDA_VISIBLE_DEVICES=0 python infer_chatglm.py
```

### 硬件需求

| 批处理大小 | LoRA `r` | 模式 | GPU显存 |
| --------- | -------- | ---- | ------ |
|     8     |     8    | FP16 |  24GB  |


## 和现有类似项目的比较

- [THUDM/ChatGLM-6B](https://github.com/THUDM/ChatGLM-6B/tree/main/ptuning)
  - ChatGLM 基于 [P-Tuning v2](https://github.com/THUDM/P-tuning-v2) 微调的官方实现，使用了 [ADGEN](https://aclanthology.org/D19-1321.pdf) 数据集。
  - 本仓库的代码实现绝大部分参考该项目。我们进一步实现了 [LoRA](https://arxiv.org/abs/2106.09685) 微调方法。此外，我们**动态地**将每个批处理数据中的序列进行填充，而非将其填充到模型的最大长度，此改进可以加速模型训练。
- [mymusise/ChatGLM-Tuning](https://github.com/mymusise/ChatGLM-Tuning)
  - ChatGLM 基于 [LoRA](https://arxiv.org/abs/2106.09685) 微调的非官方实现，使用了 [Stanford Alpaca](https://github.com/tatsu-lab/stanford_alpaca) 数据集。
  - 我们借鉴了该项目的一些想法。我们的训练脚本将数据预处理部分**集成**至训练脚本中，以避免事先生成预处理后的数据。
- [ssbuild/chatglm_finetuning](https://github.com/ssbuild/chatglm_finetuning)
  - ChatGLM 基于多种微调方法的非官方实现，使用了 [Stanford Alpaca](https://github.com/tatsu-lab/stanford_alpaca) 数据集。
  - 我们的训练脚本**全部**基于 [Huggingface transformers](https://github.com/huggingface/transformers) 框架实现，不依赖于额外的 [deep_training](https://github.com/ssbuild/deep_training) 框架。
- [lich99/ChatGLM-finetune-LoRA](https://github.com/lich99/ChatGLM-finetune-LoRA)
  - ChatGLM 基于 [LoRA](https://arxiv.org/abs/2106.09685) 微调的非官方实现，使用了 [Stanford Alpaca](https://github.com/tatsu-lab/stanford_alpaca) 数据集。
  - 我们利用 [Huggingface PEFT](https://github.com/huggingface/peft) 框架来引入最先进的微调方法。
- [liucongg/ChatGLM-Finetuning](https://github.com/liucongg/ChatGLM-Finetuning)
  - ChatGLM 基于参数冻结、LoRA 和 P-Tuning 微调的非官方实现，使用了汽车工业数据集。
  - 我们旨在引入更多指令遵循数据集用于微调 ChatGLM 模型。
- [yanqiangmiffy/InstructGLM](https://github.com/yanqiangmiffy/InstructGLM)
  - ChatGLM 微调的非官方实现，旨在探索 ChatGLM 在指令遵循数据集上的潜力。
  - 我们将数据预处理部分集成到训练脚本中。

## TODO

- 加入更多[中文数据集](https://github.com/brightmart/nlp_chinese_corpus)。
  - ~~[BELLE](https://github.com/LianjiaTech/BELLE)~~
  - [pCLUE](https://github.com/CLUEbenchmark/pCLUE)
  - [CLUECorpus](https://github.com/CLUEbenchmark/CLUECorpus2020)
  - ~~[GuanacoDataset](https://huggingface.co/datasets/JosephusCheung/GuanacoDataset)~~
  - ~~[FireflyDataset](https://huggingface.co/datasets/YeungNLP/firefly-train-1.1M)~~
- 加入基于 [ChatGPT](https://openai.com/blog/chatgpt) 和 [GPT-4](https://openai.com/research/gpt-4) 产生的数据集。
  - [Baize](https://github.com/project-baize/baize-chatbot)
  - ~~[GPT-4-LLM](https://github.com/Instruction-Tuning-with-GPT-4/GPT-4-LLM)~~
- 实现参数冻结和 ~~P-Tuning~~ 微调方法。
- 支持多GPU训练。
- 加入模型评估脚本。

## 协议

本仓库的代码依照 [Apache-2.0](LICENSE) 协议开源。

## 引用

如果您觉得此项目有帮助，请考虑以下列格式引用

```bibtex
@Misc{chatglm-efficient-tuning,
  title = {ChatGLM Efficient Tuning},
  author = {hiyouga},
  howpublished = {\url{https://github.com/hiyouga/ChatGLM-Efficient-Tuning}},
  year = {2023}
}
```


## 声明

本项目受益于 [ChatGLM-6B](https://github.com/THUDM/ChatGLM-6B)、 [ChatGLM-Tuning](https://github.com/THUDM/ChatGLM-6B) 和 [yuanzhoulvpi2017/zero_nlp](https://github.com/yuanzhoulvpi2017/zero_nlp)，感谢作者的付出。