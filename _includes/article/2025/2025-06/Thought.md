想记录一下 AI 模型训练和微调的过程中，需要用到的一些脚手架以及项目常用cli使用方式

e.g. llama-factory ms-swift unsloth vllm sglang

主要针对 linux 公网图形服务器

① llama-factory

启动以及使用`gradio` WebUI 面板

GPU服务器 公网访问

```shell
export GRADIO_SHARE=true
llamafactory-cli webui
```

② ms-swift

启动以及使用`gradio` WebUI 面板

```shell
swift web-ui --lang zh --share true
```

一些推理加速和高效微调的库

vllm: 基于 Paged Attention 实现高效解码。支持吞吐量优化和低延迟生成。

sglang: 是一个结构化生成语言，提供前端语言和运行时系统，优化复杂 LLM 程序的执行。支持多模态输入、并行控制和 KV 缓存重用。专注于执行复杂应用，如代理控制、逻辑推理和多轮对话，适合编程需要多个生成调用和控制流的 LLM 应用。

unsloth: 专门为 LLM 微调和强化学习设计的开源框架，优化了速度和内存使用。支持 QLoRA 和 LoRA 等技术，适合在有限硬件资源上进行微调。

---

什么是超参数？常见的超参数有：

· 随机种子（Random Seed）：随机种子是指在模型训练过程中用于初始化随机数生成器的固定值。它可以确保模型在不同的训练过程中使用相同的随机数序列，从而提高模型的可重复性。通常需要根据具体任务进行调整。

· 学习率（Learning Rate）：学习率是指模型在训练过程中更新权重的步长大小。它决定了模型参数更新的速度，通常需要根据具体任务进行调整。较小的学习率可能会导致模型收敛缓慢，而较大的学习率可能会导致模型在训练过程中不稳定。

· 批量大小（Batch Size）：批量大小是指在模型训练过程中每次迭代所处理的样本数量。它直接影响到模型参数更新的方向，通常需要根据可用内存和显存进行调整。较小的批量大小可能会导致模型训练过程中震荡，而较大的批量大小可能会导致模型训练过程中收敛速度变慢。

· 迭代次数（Epoch）：迭代次数是指模型在训练数据集上完整迭代一次所需要的次数。它直接影响到模型训练的时间和效果，通常需要根据具体任务进行调整。较小的迭代次数可能会导致模型欠拟合，而较大的迭代次数可能会导致模型过拟合。

· 隐藏层大小（Hidden Size）：隐藏层大小是指模型中隐藏层神经元的数量。它直接影响到模型的表达能力，通常需要根据具体任务进行调整。较小的隐藏层大小可能会导致模型欠拟合，而较大的隐藏层大小可能会导致模型过拟合。

· 正则化系数（Regularization Coefficient）：正则化系数是指在模型训练过程中用于防止过拟合的系数。它可以通过添加惩罚项来限制模型的复杂度，通常需要根据具体任务进行调整。较小的正则化系数可能会导致模型欠拟合，而较大的正则化系数可能会导致模型过拟合。

· 优化器（Optimizer）：优化器是指用于模型训练过程中更新模型参数的算法。常用的优化器有随机梯度下降（SGD）、Adam、Adagrad 等。不同的优化器可能会对模型训练过程产生不同的影响，需要根据具体任务进行调整。

· 激活函数（Activation Function）：激活函数是指模型中引入非线性因素的函数。常用的激活函数有 Sigmoid、ReLU、Tanh 等。不同的激活函数可能会对模型训练过程产生不同的影响，需要根据具体任务进行调整。

· 损失函数（Loss Function）：损失函数是指模型在训练过程中用于评估模型预测结果与真实标签之间差异的函数。常用的损失函数有均方误差（MSE）、交叉熵损失（Cross-Entropy Loss）等。不同的损失函数可能会对模型训练过程产生不同的影响，需要根据具体任务进行调整。

· 评价指标（Evaluation Metric）：评价指标是指模型在训练过程中用于评估模型性能的指标。常用的评价指标有准确率（Accuracy）、精确率（Precision）、召回率（Recall）等。不同的评价指标可能会对模型训练过程产生不同的影响，需要根据具体任务进行调整。

· 学习率调整策略（Learning Rate Scheduler）：学习率调整策略是指在模型训练过程中动态调整学习率的方法。常用的学习率调整策略有固定学习率、逐步衰减学习率、余弦退火学习率等。不同的学习率调整策略可能会对模型训练过程产生不同的影响，需要根据具体任务进行调整。

---

模型数据集的上传与下载

这里以 modelscope 为例

下载：

```python
from modelscope import snapshot_download

download_dir = 'f:/modelscope'

model_dir = snapshot_download('unsloth/Meta-Llama-3.1-8B-Instruct-bnb-4bit', cache_dir=download_dir)

print(f'Model downloaded to: {model_dir}')
```

```shell
modelscope download
    --model Qwen/Qwen2.5-7B-Instruct
    --cache_dir /root/autodl-tmp/models
```

上传：

```python
from modelscope.hub.api import HubApi

YOUR_ACCESS_TOKEN = 'YOUR-MODELSCOPE-TOKEN'
api = HubApi()
api.login(YOUR_ACCESS_TOKEN)

owner_name = 'owner'
model_name = 'awesome-new-model'
model_id = f"{owner_name}/{model_name}"

api.upload_folder(
    repo_id=f"{owner_name}/{model_name}",
    folder_path='C:\\Users\\31968\\Downloads\\Compressed\\wsj0',
    commit_message='upload dataset folder to repo',
    repo_type='dataset'
)
```

```shell
modelscope upload owner/awesome-new-model /path/to/model_folder
    --token YOUR-MODELSCOPE-TOKEN
    --repo-type model
    --commit-message 'init'
    --commit-description 'my first commit'
```

后台挂载

```shell
nohup python train_wsj0mix.py hparams/WSJ0Mix/dpmamba_M.yaml \
  --data_folder /home/wym/wsj0-mix/2speakers \
  --dynamic_mixing True \
  --base_folder_dm /home/wym/wsj0/si_tr_s \
  --precision bf16 > training_output.log 2>&1 &
```

---

大模型发展史中一些著名框架的整理：

- Transformer 2017 https://github.com/huggingface/transformers
- BERT 2018 https://github.com/google-research/bert
- GPT-2 2019 https://github.com/openai/gpt-2
- ViT 2021 https://github.com/google-research/vision_transformer
- Mamba 2023 https://github.com/state-spaces/mamba
- ResNet 2015 https://github.com/KaimingHe/deep-residual-networks
- CLIP 2021 https://github.com/openai/CLIP