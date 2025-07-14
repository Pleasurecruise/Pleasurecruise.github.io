这篇思考就主要记录暑研人工智能在语音分离中遇到的一个使用范围较广的一体化语音工具包和语音研究平台

Repo Url: https://github.com/speechbrain/speechbrain

DeepWiki: https://deepwiki.com/speechbrain/speechbrain

Mamba-Transnet ClearerVoice的模型训练中用到了该工具包

> 一些语音指标：SZDAx
> 信噪比（SNR）
> 语音质量感知评估（PESQ）
> 短时客观可懂度（STOI）
> 深度降噪平均意见得分（DNSMOS）
> 尺度不变信噪比（SI-SDR）
> 以及更多质量基准

① 使用背景（Accroding to Repo Readme）

语音处理和自然语言处理（NLP）等领域如今已变得非常接近，是时候推出一个整体工具包了，它模仿人脑的功能，统一支持构建复杂的对话式AI系统所需的多种技术。涵盖了语音识别、说话人识别、语音增强、语音分离、语言建模、对话以及更广泛的领域。


包含的一些工具