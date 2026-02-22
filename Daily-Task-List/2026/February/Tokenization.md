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
I have an Egg.egg.
EGG.

"AI技术的发展正在改变世界 (Innovation drives progress). "
"今日、私たちは未来の扉を開いています。 "
"인공지능은 언어의 장벽을 허물 수 있을까요? "
"繁體中文的測試也是必不可少的。" 

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