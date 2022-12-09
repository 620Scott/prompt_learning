# prompt_text_classification
基于prompt的中文文本分类。

# 前言

我们可以通过构建模板，来预测[MASK]，该[MASK]与我们的类别相关，比如：

```python
from pprint import pprint

from transformers import pipeline, BertModel, BertTokenizer, BertForMaskedLM

model_path = "./model_hub/chinese-bert-wwm-ext"
model = BertForMaskedLM.from_pretrained(model_path)
tokenizer = BertTokenizer.from_pretrained(model_path)
masker = pipeline("fill-mask", model=model, tokenizer=tokenizer)


verblize_dict = {"正面": ["好", "棒", "酷", "美"], "负面": ["差", "坏", "烂", "糟"]}
hash_dict = dict()
for k, v in verblize_dict.items():
    for v_ in v:
        hash_dict[v_] = k

text = "我不喜欢这部电影。"
prompt_template = "因为它很[MASK]。"
p_text = text + prompt_template
pred = masker(p_text)
pprint(pred)
pprint([{"label": hash_dict.get(i["token_str"], ""), "score":i["score"]} for i in pred])

[{'score': 0.13008198142051697,
  'sequence': '我 不 喜 欢 这 部 电 影 。 因 为 它 很 烂 。',
  'token': 4162,
  'token_str': '烂'},
 {'score': 0.10611282289028168,
  'sequence': '我 不 喜 欢 这 部 电 影 。 因 为 它 很 长 。',
  'token': 7270,
  'token_str': '长'},
 {'score': 0.0895572155714035,
  'sequence': '我 不 喜 欢 这 部 电 影 。 因 为 它 很 短 。',
  'token': 4764,
  'token_str': '短'},
 {'score': 0.08486492931842804,
  'sequence': '我 不 喜 欢 这 部 电 影 。 因 为 它 很 差 。',
  'token': 2345,
  'token_str': '差'},
 {'score': 0.06530498713254929,
  'sequence': '我 不 喜 欢 这 部 电 影 。 因 为 它 很 糟 。',
  'token': 5136,
  'token_str': '糟'}]
[{'label': '负面', 'score': 0.13008198142051697},
 {'label': '', 'score': 0.10611282289028168},
 {'label': '', 'score': 0.0895572155714035},
 {'label': '负面', 'score': 0.08486492931842804},
 {'label': '负面', 'score': 0.06530498713254929}]
```

当然我们也可以进行命名实体识别任务：

```python
text = "我喜欢的书是《一千零一夜》。"
prompt_template = "一千零一夜的类型是[MASK]。"
p_text = text + prompt_template
pred = masker(p_text)
pprint(pred)

[{'score': 0.4005790650844574,
  'sequence': '我 喜 欢 的 书 是 《 一 千 零 一 夜 》 。 一 千 零 一 夜 的 类 型 是 书 。',
  'token': 741,
  'token_str': '书'},
 {'score': 0.06575167924165726,
  'sequence': '我 喜 欢 的 书 是 《 一 千 零 一 夜 》 。 一 千 零 一 夜 的 类 型 是 爱 。',
  'token': 4263,
  'token_str': '爱'},
 {'score': 0.05279282107949257,
  'sequence': '我 喜 欢 的 书 是 《 一 千 零 一 夜 》 。 一 千 零 一 夜 的 类 型 是 诗 。',
  'token': 6408,
  'token_str': '诗'},
 {'score': 0.0526682585477829,
  'sequence': '我 喜 欢 的 书 是 《 一 千 零 一 夜 》 。 一 千 零 一 夜 的 类 型 是 的 。',
  'token': 4638,
  'token_str': '的'},
 {'score': 0.025359274819493294,
  'sequence': '我 喜 欢 的 书 是 《 一 千 零 一 夜 》 。 一 千 零 一 夜 的 类 型 是 " 。',
  'token': 107,
  'token_str': '"'}]
```

我们可以构建模板，让模型预测[MASK]，该[MASK]对应着类别。

# BERT文本分类

```python
python bert_classification.py

 precision    recall  f1-score   support

          其他       0.62      0.77      0.69      2875
          喜好       0.63      0.62      0.63      1330
          悲伤       0.63      0.48      0.54      1079
          厌恶       0.53      0.34      0.41      1147
          愤怒       0.45      0.53      0.49      649
          高兴       0.66      0.61      0.63      947

    accuracy                           0.61      8027
    macro avg          0.59      0.56      0.57      8027
    weighted avg        0.60      0.61      0.60      8027
```

# Prompt文本分类

```python
python bert_prompt.py

              precision    recall  f1-score   support

          其他       0.52      1.00      0.68      2801
          喜好       1.00      0.57      0.73      1333
          悲伤       1.00      0.55      0.71      1090
          厌恶       1.00      0.29      0.45      1132
          愤怒       1.00      0.44      0.61       649
          高兴       1.00      0.64      0.78      1009

    accuracy                           0.68      8014
    macro avg       0.92      0.58      0.66      8014
    weighted avg       0.83      0.68      0.67      8014

进行预测：
texts = [
      ["银色的罗马高跟鞋，圆球吊饰耳饰单带，个性十足，有非常抢眼！", 1],
      ["稳吾到嘛？", 3],
      ["以后打死也不吃了", 3],
      ["来广州两天都没能织围脖,一直都在忙,再加上又感冒了,好痛苦[泪]不过广州给我的感觉灰常好!", 5],
      ["对骂我从来没怕过，你们也就只能考虑暗杀了，这样就充分保护动物了，臭傻逼们[打哈气]", 4],
      ["你这么说的我都不好意思呢", 0],
      ["我到了，文，好惨啊…", 2],
      ["真是不容易啊。。", 0]
    ]

银色的罗马高跟鞋，圆球吊饰耳饰单带，个性十足，有非常抢眼！
预测： 1 1 喜好
真实： 1 喜好
====================================================================================================
稳吾到嘛？

预测： 0 0 其他
真实： 3 厌恶
====================================================================================================
以后打死也不吃了
预测： 0 0 其他
真实： 3 厌恶
====================================================================================================
来广州两天都没能织围脖,一直都在忙,再加上又感冒了,好痛苦[泪]不过广州给我的感觉灰常好!
预测： 5 5 高兴
真实： 5 高兴
====================================================================================================
对骂我从来没怕过，你们也就只能考虑暗杀了，这样就充分保护动物了，臭傻逼们[打哈气]
预测： 4 4 愤怒
真实： 4 愤怒
====================================================================================================
你这么说的我都不好意思呢
预测： 0 0 其他
真实： 0 其他
====================================================================================================
我到了，文，好惨啊…
预测： 2 2 悲伤
真实： 2 悲伤
====================================================================================================
真是不容易啊。。
预测： 2 2 悲伤
真实： 0 其他
====================================================================================================
```

# 说明

我们构建模板：text + prompt，比如：

```python
text = "对骂我从来没怕过，你们也就只能考虑暗杀了，这样就充分保护动物了，臭傻逼们[打哈气]"
prompt = "情感是[MASK][MASK]。"
然后得到："对骂我从来没怕过，你们也就只能考虑暗杀了，这样就充分保护动物了，臭傻逼们[打哈气]情感是[MASK][MASK]。"
```

我们的标签是6大类：

```python
label2id = {
    "其他": 0,
    "喜好": 1,
    "悲伤": 2,
    "厌恶": 3,
    "愤怒": 4,
    "高兴": 5,
}
```

查询出标签在vocab.txt里面每一个字对应的索引：

```python
label2ind = {
    "其他": [1071, 800],
    "喜好": [1599, 1962],
    "悲伤": [2650, 839],
    "厌恶": [1328, 2626],
    "愤怒": [2699, 2584],
    "高兴": [7770, 1069],
}
```

在计算损失的时候，先找出[MASK]的位置，再取对应的字的输出，然后和真实的标签计算损失。这里我们分别计算第一个字和第二个字的损失，最终相加。：

```python
pred1 = logits[i, mask1][[1071, 1599, 2650, 1328, 2699, 7770]]
pred2 = logits[i, mask2][[800, 1962, 839, 2626, 2584, 1069]]

loss1 = self.criterion(pred1.unsqueeze(0), label[i].unsqueeze(0))
loss2 = self.criterion(pred2.unsqueeze(0), label[i].unsqueeze(0))
```

预测的时候，我们要判断预测的两个字是否是在标签中的，比如**其好**就不是，我们就认为该标签为0。

# 补充

- 如果我们的标签的长度不一致那要怎么做呢？

	将标签作为特殊符号加入到vocab.txt中，这样就可以通过一个[MASK]就预测出所属的类别。

# 参考

>  https://mp.weixin.qq.com/s/nBE2e8VY_cI9AqtTjvnQIQ