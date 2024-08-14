# LLMs-Evaluation
LLMs-Evaluation介绍了如何将何将各种LLMs、Azure OpenAI服务，和[GaoKao-Bench项目](https://github.com/OpenLMLab/GAOKAO-Bench)配合使用，以测评不同大语言模型的做题能力。

# 说明
以下将一一说明本项目如何修改原代码

①在vscode中打开GaoKao-Bench项目，在openai_gpt4.py中更改引用包的函数为AzureOpenAI、更改base_url
![image](https://github.com/Qinyi-Tan/LLMs-Evaluation/blob/main/Graphs/graph1.png)
