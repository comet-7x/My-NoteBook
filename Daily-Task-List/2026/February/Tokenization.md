Tokenization is at the heart of much weirdness of LLMs. Do not brush it off.

- Why can't LLM spell words? **Tokenization**.
- Why can't LLM do super simple string processing tasks like reversing a string?**Tokenization**.
- Why is LLM worse at non-English languages (e.g. Japanese)? **Tokenization**.
- Why is LLM bad at simple arithmetic? **Tokenization**.
- Why did GPT-2 have more than necessary trouble coding in Python? **Tokenization.**
- Why did my LLM abruptly halt when it sees the string `"<lendoftextl>"`? **Tokenization**.
- What is this weird warning I get about a "trailing whitespace"? **Tokenization.**
- Why the LLM break if I ask it about "SolidGoldMagikarp"? **Tokenization**.
- Why should I prefer to use YAML over JSON with LLMs? **Tokenization**.
- Why is LLM not actually end-to-end language modeling? **Tokenization**.
- What is the real root of suffering? **Tokenization**.


在线可视化观察不同分词模型：
> [Tiktokenizer](https://tiktokenizer.vercel.app/) 

```
Tokenization is at the heart of much weirdness of LlMs. Donot brush it off.

127 + 677 = 804
1275 +6773 =8041

Egg.
I have an Egg.
egg.
EGG.

"AI技术的发展正在改变世界。"
"The development of AI technology is changing the world."
"AI技術の発展が世界を変えています."
"AI 기술의 발전이 세상을 바꾸고 있습니다." 

for i in range(1, 101):
	if i % 3 == 0 and i % 5 == 0:
		print("FizzBuzz")
	elif i % 3 == 0:
		print("fizz")
	elif i % 5 == 0:
		print("Buzz")
	else:
		print(i)
```

- 分词器的的分词情况是随机的，即使很简单的算术运算也会出现错误的分词情况
- 针对英文来讲，对于同一个概念，相同含义的单词会因为大小写（全部大写，全部小写，混写），位置（句首，句末，句中）的原因出现不同的分词情况
- 分词器会由于数据集（数量，种类等）的原因，导致相同含义的句子因为不同语言的原因导致Tokne数量的不同
- 减少Token的标记数量从而增加模型的上下文长度
- 