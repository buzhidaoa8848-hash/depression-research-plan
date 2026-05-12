# LLM 症状特征抽取 Prompt · 抑郁风险筛查项目

> 用途：从 DAIC-WOZ / E-DAIC 访谈转录文本中，按 PHQ-9 九个症状维度抽取结构化特征
> 模型：DeepSeek / GPT-4o-mini（推荐，成本低）
> 输出：每条 utterance 的 9 维症状打分 + 置信度

---

## Prompt 模板（中英双语版）

### 中文版

```
你是一个专业的心理健康症状标注助手。你的任务是从一段访谈对话中，识别受访者（不是 interviewer）表现出的抑郁相关症状。

请按以下 PHQ-9 九个维度，对受访者在这段对话中的表现进行评分：

1. 兴趣/愉悦感缺失（Anhedonia）：对做事缺乏兴趣或愉悦感
2. 情绪低落（Depressed mood）：感到沮丧、抑郁或绝望
3. 睡眠障碍（Sleep disturbance）：入睡困难、易醒或睡得过多
4. 精力不足（Fatigue）：感到疲倦或精力不足
5. 食欲变化（Appetite change）：食欲差或暴食
6. 自我评价低（Self-worth）：觉得自己很糟、失败或让家人失望
7. 注意力困难（Concentration）：难以集中注意力或做决定
8. 精神运动变化（Psychomotor）：说话/动作迟缓或烦躁不安
9. 自伤想法（Suicidal ideation）：有不如死了或伤害自己的想法

评分标准：
0 = 完全没有提及 / 没有迹象
1 = 轻微提及 / 可能有迹象
2 = 明确提及 / 明显迹象
3 = 反复强调 / 严重迹象

注意：
- 只分析**受访者**的语句，不要分析 interviewer 的提问
- 如果受访者没有提到某个维度，请评 0
- 请基于对话内容打分，不要过度推断
- 如果某个维度信息不足，confidence 设为低

访谈对话：
{transcript_text}

请以 JSON 格式输出：
{
  "symptoms": [
    {"dimension": "兴趣缺失", "score": 0-3, "evidence": "引用的原话", "confidence": "高/中/低"},
    {"dimension": "情绪低落", "score": 0-3, "evidence": "...", "confidence": "高/中/低"},
    ...
  ],
  "summary": {
    "total_score": 0-27,
    "key_concern": "最值得关注的症状描述"
  }
}
```

### 英文版（用于 GPT-4o/DeepSeek）

```
You are a professional mental health symptom annotation assistant. Your task is to identify depression-related symptoms expressed by the **interviewee** (not the interviewer) in the following clinical interview transcript.

Rate the interviewee's responses across 9 PHQ-9 dimensions:

1. Anhedonia: Little interest or pleasure in doing things
2. Depressed mood: Feeling down, depressed, or hopeless
3. Sleep disturbance: Trouble sleeping, early waking, or oversleeping
4. Fatigue: Feeling tired or having little energy
5. Appetite change: Poor appetite or overeating
6. Self-worth: Feeling bad about yourself, a failure, or letting family down
7. Concentration: Trouble concentrating on things
8. Psychomotor changes: Moving/speaking slowly or being restless
9. Suicidal ideation: Thoughts of being better off dead or self-harm

Rating:
0 = Not mentioned / no indication
1 = Mildly mentioned / possible indication
2 = Clearly mentioned / evident indication
3 = Repeated emphasis / severe indication

Rules:
- Analyze ONLY the interviewee's statements, NOT the interviewer
- Score 0 if a dimension is not mentioned
- Base scores on actual content, do not over-infer
- Set confidence to "low" if insufficient evidence

Interview transcript:
{transcript_text}

Output JSON format:
{
  "symptoms": [
    {"dimension": "anhedonia", "score": 0-3, "evidence": "quoted text", "confidence": "high/medium/low"},
    ...
  ],
  "summary": {
    "total_score": 0-27,
    "key_concern": "description of most concerning symptom"
  }
}
```

---

## 后处理（从 LLM 输出 → 结构化特征）

```python
# LLM 输出 → 9 维特征向量
symptom_vector = [
    result["symptoms"][0]["score"],  # 兴趣缺失
    result["symptoms"][1]["score"],  # 情绪低落
    result["symptoms"][2]["score"],  # 睡眠障碍
    result["symptoms"][3]["score"],  # 精力不足
    result["symptoms"][4]["score"],  # 食欲变化
    result["symptoms"][5]["score"],  # 自我评价
    result["symptoms"][6]["score"],  # 注意力
    result["symptoms"][7]["score"],  # 精神运动
    result["symptoms"][8]["score"],  # 自伤想法
]
# shape: (9,)  per utterance

# session 级聚合：取所有 utterance 的 mean / max
session_symptom_features = np.mean(all_utterance_vectors, axis=0)  # (9,)
```

---

## 使用方式

```bash
# 单条测试
python3 llm_symptom_extract.py --transcript "path/to/transcript.txt" --model deepseek

# 批量处理
python3 llm_symptom_extract.py --batch --input_dir ./transcripts/ --output_dir ./symptoms/
```

---

## Prompt 设计关键点

| 设计决策 | 原因 |
|:--------|:-----|
| 按 PHQ-9 九个维度打分 | 与标签 PHQ-8 天然对齐，可解释性强 |
| 要求 evidence 引用原话 | 可追溯、可验证，防止 LLM 幻觉 |
| 区分高/中/低 confidence | 后续融合时可做加权 |
| 明确"只分析受访者" | 防止混淆 Ellie 提问句 |
| 0-3 四级评分 | 比二分类更细粒度，比连续分更稳定 |
| JSON 结构化输出 | 直接解析为特征向量，无需二次处理 |
