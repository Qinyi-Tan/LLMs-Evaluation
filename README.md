# 介绍
LLMs-Evaluation介绍了如何将何将LLMs、Azure OpenAI服务和[GaoKao-Bench项目](https://github.com/OpenLMLab/GAOKAO-Bench)配合使用，以测评不同大语言模型针对不同学科的做题能力。

# 说明
## 代码修改
**以下将逐步展示本项目修改原代码的部分，完整流程可至[Tutorial.pdf](https://github.com/Qinyi-Tan/LLMs-Evaluation/blob/main/Tutorial.pdf)中查看**

1. 在vscode中打开GaoKao-Bench项目，在openai_gpt4.py中更改引用包的函数为AzureOpenAI
```
from openai import AzureOpenAI
```
更改base_url
```
    def __init__(self, api_key_list:List[str], base_url: str="your_base_url", organization: str=None, model_name:str="your_model_name", temperature:float=0.3, max_tokens: int=4096):
```

2. 加入endpoint以及api key这两个参数
```
                    azure_endpoint="your_azure_endpoint",
                    api_key="your_api_key",
                    api_version="your_api_version"
```

3. 在objective_bench.py中将字符编码改为utf-8，确保能成功编码
```
    with open("Obj_Prompt.json", "r",encoding='utf-8') as f:
```

4. 在bench_function.py中修改原字段的错误（将standard_answer改为answer），使得可以读取answer的json格式
```
        standard_answer = data[i]['answer']
```

5. 在bench_function.py中修改写入文件的方式为每循环一次写一次，使得程序员可以同步做题情况，代替了原代码写完所有的题再输出的形式，避免了因为一个卡顿而无法获得输出的情况（以下所示为修改后的字段）
```
def choice_test_changed(**kwargs):
    """

    Get answers of the Choice Questions

    """

    model_api = kwargs['model_api']
    model_name = kwargs['model_name']
    start_num = kwargs['start_num']
    end_num = kwargs['end_num']
    data = kwargs['data']['example']
    keyword = kwargs['keyword']
    prompt = kwargs['prompt']
    question_type = kwargs['question_type']
    save_directory = kwargs['save_directory']

    file_name = model_name + "_seperate_" + keyword + f"_{start_num}-{end_num - 1}.json"
    file_path = os.path.join(save_directory, file_name)

    for i in tqdm(range(start_num, end_num)):

        index = data[i]['index']
        question = data[i]['question'].strip() + '\n'
        year = data[i]['year']
        category = data[i]['category']
        score = data[i]['score']
        standard_answer = data[i]['answer']
        answer_lenth = len(standard_answer)
        analysis = data[i]['analysis']
        # 调用模型
        model_output = model_api(prompt, question)
        # 提取 model_answer
        model_answer = extract_choice_answer(model_output, question_type, answer_lenth)
        # TODO: which content of temp we expect

        dict = {
            'index': index,
            'year': year,
            'category': category,
            'score': score,
            'question': question,
            'standard_answer': standard_answer,
            'analysis': analysis,
            'model_answer': model_answer,
            'model_output': model_output
        }

        with open(file_path, 'a', encoding='utf-8') as f:
            if f.tell() == 0:
                # If it's the first entry, write the opening brace and the keyword
                f.write('{\n"keyword": "' + keyword + '",\n"example": [\n')
            json.dump(dict, f, ensure_ascii=False, indent=4)
            if i < end_num - 1:
                f.write(',\n')  # Add a comma after each entry except the last one
            else:
                f.write('\n]}\n')

        time.sleep(5)
```

7. 在objective_bench.py中写入需要测试的LLM（可以任意选择Azure AI Studio中的基本模型），以gpt-4o为例
```
    model_name = "gpt-4o"
```

8. 缩小数据集（[这里](https://github.com/Qinyi-Tan/LLMs-Evaluation/tree/main/Data/Objective_Questions)可见）

**修改完成，可对比不同大语言模型的做题能力**

# 测评结果
*（本项目测评的题目均为客观题）*

获得不同大语言模型做不同学科的题目的正确率为
![image](https://github.com/Qinyi-Tan/LLMs-Evaluation/blob/main/Results_Charts/Different_Subjects.png)

将学科分类为文理科后，获得数据
![image](https://github.com/Qinyi-Tan/LLMs-Evaluation/blob/main/Results_Charts/Arts_vs_Sciences.png)

# 致谢

