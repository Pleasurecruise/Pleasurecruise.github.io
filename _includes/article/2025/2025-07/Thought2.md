想记录一下暑研人工智能医学图像分割中遇到的一个认可度高的图卷积神经网络框架 nnUNet

Repo Url: https://github.com/MIC-DKFZ/nnUNet

DeepWiki: https://deepwiki.com/MIC-DKFZ/nnUNet

向 U-Mamba nnWNet 等 UNet 变体 都用到了该框架

- 编码器（Encoder）

通常使用下采样操作（如卷积 + 池化），逐层提取高维语义特征。
在3D U-Net中，使用的是 3D 卷积，处理体积图像（如CT、MRI数据）。
- 解码器（Decoder）

使用上采样操作（如转置卷积）将特征分辨率逐步恢复。
同时利用 skip connections（跳跃连接）融合编码器中的浅层信息，保留空间细节。
- Bottleneck（中间连接部分）

编码与解码之间的最深部分，通常进行一些处理（卷积+激活函数等），决定高层语义信息。
- 跳跃连接（Skip Connections）

将编码器中的浅层特征与解码器中对应级别的上采样特征进行拼接。
帮助保持空间信息并缓解梯度消失的问题。

U-Mamba 结合了卷积层和状态空间模型（SSM）的优点，能够同时捕获局部特征并聚合长距离依赖性，优于现有的基于CNN和Transformer的分割网络

① 使用背景（Accroding to Repo Readme）

图像数据集具有极大的多样性

- 图像维度（2D、3D）
- 模态/输入通道（RGB图像、CT、MRI、显微镜图像等）
- 图像尺寸、体素尺寸
- 类别比例、目标结构特性等方面在不同数据集中差别显著。

传统上，面对一个新的问题，需要手动设计和优化定制的解决方案——这一过程容易出错、难以扩展，且其成功在很大程度上取决于实验者的技能。
不仅需要考虑众多的设计选择和数据属性，而且它们之间还紧密关联，使得可靠的手动流程优化几乎不可能实现。

而 nnU-Net 是一种语义分割方法，能够自动适应给定的数据集。它将分析提供的训练案例，并自动配置一个匹配的基于 U-Net 的分割流程。

只需将你的数据集转换为nnU-Net格式，非常适合用于大模型的图像分割、训练领域大模型

> 鲁棒性的意思是：面对干扰和不确定性时仍然稳定可靠

② 如何工作

为每个数据集创建几种U-Net配置：

- 2d：一个2D U-Net（适用于2D和3D数据集）
- 3d_fullres：一个在高图像分辨率上运行的3D U-Net（仅用于3D数据集）
- 3d_lowres → 3d_cascade_fullres：一个3D U-Net级联，其中第一个3D U-Net在低分辨率图像上运行，然后第二个高分辨率3D U-Net优化前者的预测结果（仅用于具有大图像尺寸的3D数据集）

③ 数据结构

```text
TaskXXX_TaskName/
├── dataset.json  # 包含类别标签、模态等元数据
├── imagesTr/     # 训练图像（命名需含患者ID，如patient001_0000.nii.gz）
├── labelsTr/     # 训练标签
└── imagesTs/     # 测试图像（可选）
```

```text
 imagesTs
 ├── la_001_0000.nii.gz
 ├── la_002_0000.nii.gz
 ├── la_006_0000.nii.gz
 ├── ...
```

dataset.json 格式
```json
{ 
 "channel_names": {  # formerly modalities
   "0": "T2", 
   "1": "ADC"
 }, 
 "labels": {  # THIS IS DIFFERENT NOW!
   "background": 0,
   "PZ": 1,
   "TZ": 2
 }, 
 "numTraining": 32, 
 "file_ending": ".nii.gz"
 "overwrite_image_reader_writer": "SimpleITKIO"  # optional! If not provided nnU-Net will automatically determine the ReaderWriter
 }
```

推理所使用的数据格式必须与原始数据所使用的格式一致。文件名必须以唯一的标识符开头，然后是一个4位数的模态标识符。

nnUNet_preprocessed 预处理完的数据集

nnUNet_raw 原始数据集

nnUNet_results 模型训练结果

④ 使用命令

> MSD 即 Medical Segmentation Decathlon 医学影像分割领域的公共数据集

数据处理相关

```shell
nnUNetv2_convert_MSD_dataset -i RAW_FOLDER
nnUNetv2_convert_old_nnUNet_dataset OLD_FOLDER
nnUNetv2_plan_and_preprocess -d DATASET_ID --verify_dataset_integrity
```

训练模型

```shell
nnUNetv2_train DATASET_NAME_OR_ID UNET_CONFIGURATION FOLD --val --npz
nnUNetv2_find_best_configuration DATASET_NAME_OR_ID -c CONFIGURATIONS 
```

> 五折交叉验证是一种常用的交叉验证方法，属于机器学习中评估模型泛化能力的技术。在五折交叉验证中，数据集被划分为五个互不重叠的子集（也称为“折”或“ folds”）。模型会进行五轮训练和测试：每一轮选择不同的子集作为测试集，其余四个子集合并用作训练集。

| 参数名             | 含义说明 |
|------------------|--------|
| `dataset_name_or_id` | 指定训练所用的数据集名称或 ID（例如：`Task001_BrainTumour` 或 `1`）|
| `configuration`      | 指定训练配置，例如 `3d_fullres`、`2d`、`3d_lowres`、`3d_cascade_fullres`等|
| `fold`               | 指定交叉验证的折（fold），值为 0 到 4 之间的整数（支持 5 折交叉验证）|

推理与后处理

```shell
nnUNetv2_predict -i INPUT_FOLDER -o OUTPUT_FOLDER -d DATASET_NAME_OR_ID -c CONFIGURATION --save_probabilities
```

| 类别             | 目的                   | 关键命令                                                         |
|------------------|------------------------|------------------------------------------------------------------|
| 数据集准备       | 处理原始数据集         | `nnUNetv2_extract_fingerprint`, `nnUNetv2_convert_MSD_dataset`  |
| 实验规划         | 配置训练参数           | `nnUNetv2_plan_experiment`                                       |
| 数据预处理       | 为训练准备数据         | `nnUNetv2_preprocess`                                             |
| 综合准备         | 执行所有准备步骤       | `nnUNetv2_plan_and_preprocess`                                    |
| 模型训练         | 训练分割模型           | `nnUNetv2_train`                                                  |
| 推理预测         | 运行预测任务           | `nnUNetv2_predict`, `nnUNetv2_predict_from_modelfolder`         |
| 模型集成         | 组合多个模型的预测结果 | `nnUNetv2_ensemble`                                               |
| 模型评估         | 评估模型性能           | `nnUNetv2_find_best_configuration`, `nnUNetv2_evaluate_folder`  |
| 模型管理         | 共享和复用模型         | `nnUNetv2_export_model_to_zip`, `nnUNetv2_download_pretrained_model_by_url` |
| 工具与实用程序   | 可视化和辅助工具       | `nnUNetv2_plot_overlay_pngs`                                      |