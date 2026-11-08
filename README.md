# QuotOS_Scheien Lite · 0.6B

中国法律**偏科**小模型：只答劳动争议、合同纠纷、侵权与人身损害赔偿、民间借贷四类问题，
0.596 B 参数，纯 CPU 就能跑，单个 GGUF 文件 378 MB。

**下载页（点这里，不要从源码目录里下）：**
https://lynnescheutie.github.io/QuotOS_Scheien-Lite/

> 这个仓库只放发布物和下载页，训练数据、法条库、训练脚本都在另一个私有仓库里。

## 发布包

| 文件 | 体积 | 用途 |
|---|---|---|
| `QuotOS_Scheien-Lite-Q4_K_M.gguf` | 378 MB | **推荐**。Ollama / LM Studio / llama.cpp 直接加载，不用装 Python |
| `QuotOS_Scheien-Lite-fp16.zip` | 1.01 GB | transformers 直接跑完整权重，含分词器与 chat template |
| `QuotOS_Scheien-Lite-adapter-step60.zip` | 35.6 MB | 只有 LoRA 增量，需自备 Qwen3-0.6B |
| `USAGE-zh.txt` | 3 KB | 三种加载方式的命令 + 必须知道的缺陷 |

三个包都出自验证集最优的 `step_00060`（valid_loss = 1.0694），不是步数最多的那个版本。
直接下载：`https://github.com/LynneScheutie/QuotOS_Scheien-Lite/releases/latest/download/<文件名>`

## 三十秒上手

```
# Modelfile
FROM ./QuotOS_Scheien-Lite-Q4_K_M.gguf
PARAMETER temperature 0.7
SYSTEM """你是中国法律咨询助手，只处理劳动争议、合同纠纷、侵权与人身损害赔偿、民间借贷。
回答结构固定：①结论 ②法律依据（法律名称+条号+条文要点，最多3条）③要件如何套用到本案
④风险与时效提示 ⑤最后一句「以上不构成正式法律意见」。信息不足时先追问，不要猜。
超范围问题直接说明并建议咨询执业律师。"""

ollama create scheien-lite -f Modelfile
ollama run scheien-lite
```

本机实测（i5-1145G7，CPU 6 线程）：加载 0.29 秒，提示词 498 tok/s，生成 63.8 tok/s，
一条完整五段式回答约 3 秒。

## 边界（别跳过这段）

- **条号能对上，条文会讲错。** 训练门禁只校验「这部法有没有这一条」，不校验条文有没有被曲解。
  GGUF 实测里它把降职降薪的依据写成了《劳动法》第三十条，正确依据应是《劳动合同法》第三十五条。
  任何引用都要回法条库核对。
- 训练只有 234 条样本、180 步，覆盖面窄，别当通用法律助手用。
- 不构成法律意见。

完整评测与缺陷清单见 [MODEL_CARD.md](MODEL_CARD.md)。
