<p align="center">
  <strong style="font-size: 2.2em;">SheafPrune</strong><br>
  <em>面向问题的视觉 Token 剪枝，让多模态推理更高效</em>
</p>

<p align="center">
  <strong>简体中文</strong> · <a href="README.md">English</a>
</p>

<p align="center">
  <strong>代码：Coming soon</strong>   ·   <strong>权重：Coming soon</strong>
</p>

<p align="center">
  <img src="assets/paper/method.png" width="100%" alt="SheafPrune 方法总览">
</p>

## 项目概览

高分辨率图像会向视觉语言模型输入大量视觉 Token，其中许多与当前问题无关。**SheafPrune 在解码前保留真正有用的证据，移除其余 Token。** 方法将跨模态相关性与视觉结构结合，在受支持约束的图上扩散分数，并在语言模型看到图像 Token 之前完成一次性 Top-K 选择。

主干模型保持冻结，训练集中在轻量级剪枝模块上。保留预算显式可控，因此同一个模型可以在推理时灵活平衡视觉上下文与延迟。

| 论文报告内容              |                                                                 结果 |
| :------------------------ | -------------------------------------------------------------------: |
| 主干模型                  |       4 个：LLaVA-NeXT-8B、Qwen2.5-VL-7B、LLaVA-1.5-7B、InternVL3-8B |
| 评测基准                  |                                                      10 个多模态基准 |
| 最佳 δ 包络领先          |                            **28 个主干–预算设置中领先 22 个** |
| LLaVA-NeXT 保留 10% Token |                                      **90.42%** 相对性能保持率 |
| 同一设置                  | **5.15×** Prefill 净加速 · **84.0%** 端到端 FLOPs 降幅 |

## SheafPrune 如何选择 Token

<p align="center">
  <img src="assets/paper/method.png" width="100%" alt="SheafPrune 四步流程：Token 投影、跨模态相关性、受支持约束的扩散、Top-K 剪枝">
</p>

1. **投影问题与图像。** 将文本与视觉 Token 映射到共享的归一化空间，并注入目标保留预算。
2. **跨模态传播相关性。** 构建稀疏的图文耦合图，把问题相关性传播到视觉 Token。
3. **尊重视觉结构。** 将相关性与视觉先验融合，在受支持约束的视觉图上扩散，保留连贯的证据区域。
4. **在解码前一次完成剪枝。** 最终分数驱动按预算的 Top-K 选择，被移除的视觉 Token 不再进入语言模型 Prefill 序列。

核心思路很直接：问题决定哪些 Token 重要，图结构保证留下的证据仍然具有视觉连贯性。

## 四个主干模型上的结果

<p align="center">
  <img src="assets/paper/results.png" width="100%" alt="四个视觉语言主干模型上的性能保持率与净加速结果">
</p>

图中展示的是论文的**最佳 δ 性能包络**：对于每个剪枝预算，呈现最优 δ 配置。它用于比较可达到的操作点，不应理解为一条固定 δ 的部署曲线。

在 LLaVA-NeXT 的**保留 10% / 剪枝 90%** 设置下，SheafPrune 仍保持 **90.42%** 相对性能，实现 **5.15×** Prefill 净加速，并降低 **84.0%** 的端到端 FLOPs。在**剪枝 70%** 时，性能保持率为 **98.7%**，Prefill 净加速为 **2.70×**。

## 极限预算下的定性证据

以下图片来自论文，并针对 README 做了裁切排版。掩码展示了保留率从 50% 降至 10% 时模型还能看到什么；其中 **Ours** 指 SheafPrune。

<p align="center">
  <img src="assets/paper/qualitative-main.png" width="100%" alt="论文中的定性对比：动物识别与运动场景理解">
</p>

上面的两个任务考察整体场景理解。下面的局部案例则测试另一种更苛刻的情况：答案藏在很小的区域里，但这个区域必须被保留下来。

<table>
  <tr>
    <td align="center" width="50%"><img src="assets/paper/case-hat.png" width="100%" alt="论文案例：读取棒球运动员帽子上的字母"><br><sub>细粒度文字证据：帽子上的字母</sub></td>
    <td align="center" width="50%"><img src="assets/paper/case-brand.png" width="100%" alt="论文案例：识别纸尿裤品牌"><br><sub>局部视觉证据：很小的品牌标识</sub></td>
  </tr>
</table>

在 10% 保留率下，留下的区域仍集中在问题指向的主体附近，而不是均匀散落在整张图上。这正是面向问题的剪枝价值：Token 更少，回答所需的像素仍在序列中。

## 评测范围

论文在 **LLaVA-NeXT-8B、Qwen2.5-VL-7B、LLaVA-1.5-7B、InternVL3-8B** 上进行评测，覆盖 **ChartQA、DocVQA、TextVQA、HR-Bench 4K、HR-Bench 8K、GQA、VQAv2、MMBench-EN、POPE 和 MME**。

## 发布状态

| 资源                   | 状态                              |
| :--------------------- | :-------------------------------- |
| 论文图片与方法概览     | 当前仓库已提供                    |
| 训练与推理代码         | **Coming soon · 即将发布** |
| SheafPrune 预训练权重  | **Coming soon · 即将发布** |
| 复现实验说明与评测脚本 | **Coming soon · 即将发布** |

代码与 Checkpoint 将在本仓库发布。主干模型和评测数据集遵循各自的原始许可协议。

---

<p align="center"><sub>保留关键证据，剪掉其余 Token。</sub></p>
