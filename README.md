# LLMs-Evaluation
LLMs-Evaluation介绍了如何将何将各种LLMs、Azure OpenAI服务，和[GaoKao-Bench项目](https://github.com/OpenLMLab/GAOKAO-Bench)配合使用，以测评不同大语言模型的做题能力。
*该项目需要用户先自行克隆GaoKao-Bench仓库*

# 说明
**以下将逐步展示本项目如何修改原代码的部分，完整流程可至[Tutorial.pdf](https://github.com/Qinyi-Tan/LLMs-Evaluation/blob/main/Tutorial.pdf)中查看**

1. 在vscode中打开GaoKao-Bench项目，在openai_gpt4.py中更改引用包的函数为AzureOpenAI、更改base_url
![image](https://github.com/Qinyi-Tan/LLMs-Evaluation/blob/main/Graphs/graph1.png)

2. 加入endpoint以及api key这两个参数
![image](https://github.com/Qinyi-Tan/LLMs-Evaluation/blob/main/Graphs/graph2.png)

3. 在objective_bench.py中将字符编码改为utf-8，确保能成功编码
![image](https://github.com/Qinyi-Tan/LLMs-Evaluation/blob/main/Graphs/graph3.png)

4. 在bench_function.py中修改原字段的错误（将standard_answer改为answer），使得可以读取answer的json格式
![image](https://github.com/Qinyi-Tan/LLMs-Evaluation/blob/main/Graphs/graph4.png)

5. 在bench_function.py中修改写入文件的方式为每循环一次写一次，使得程序员可以同步做题情况，代替了原代码写完所有的题再输出的形式，避免了因为一个卡顿而无法获得输出的情况（下图所示为修改后的字段）
![image](https://github.com/Qinyi-Tan/LLMs-Evaluation/blob/main/Graphs/graph5.1.png)
![image](https://github.com/Qinyi-Tan/LLMs-Evaluation/blob/main/Graphs/graph5.2.png)

6. 在objective_bench.py中写入需要测试的LLM（可以任意选择Azure AI Studio中的基本模型），以gpt-4o为例
![image](https://github.com/Qinyi-Tan/LLMs-Evaluation/blob/main/Graphs/graph6.png)

**修改完成**
