# 介绍
LLMs-Evaluation介绍了如何将何将LLMs、Azure OpenAI服务和[GaoKao-Bench项目](https://github.com/OpenLMLab/GAOKAO-Bench)配合使用，以测评不同大语言模型针对不同学科的做题能力。

# 说明
## 调用Azure OpenAI服务
**完整流程可至[Tutorial.pdf](https://github.com/Qinyi-Tan/LLMs-Evaluation/blob/main/Tutorial.pdf)中查看**
1. 我们需要使用以下命令安装 OpenAI Python 客户端库
```
pip install openai
```
2. 注册Azure AI Studio账号，转到Azure AI Studio中的**资源和密钥**，检索**api key**以及**endpoint**（两个必要参数），使得后续能成功调用Azure OpenAI

## 代码修改
**以下将逐步展示本项目修改原代码的部分，完整流程可至[Tutorial.pdf](https://github.com/Qinyi-Tan/LLMs-Evaluation/blob/main/Tutorial.pdf)中查看**

1. 在vscode中打开GaoKao-Bench项目（确保以及克隆其仓库），在openai_gpt4.py中更改引用包的函数为AzureOpenAI
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

# 总结
**1. 学科比较**

   ① 数学、地理、生物、英语：这四个学科中，gpt-4o-mini的正确率都高于gpt-35-turbo。特别是在数学和地理学科上，它的正确率超过85%，而gpt-35-turbo的正确率只有约60%
   
   ② 物理、化学：这两个学科中，两个模型都表现不佳，而gpt-35-turbo的表现最弱（两个学科都只有20%）
   
   ③ 政治、语文：这两个学科两个模型表现都不太理想，然而gpt-4o-mini的表现仍然优于gpt-35-turbo 
   
**2. 模型比较**

   ① gpt-4o-mini
   
   ·理科：正确率76%，表现相对较好，具有较高正确性
   
   ·文科：正确率略低于理科，为62.5%，但也相对稳定
   
   ② gpt-35-turbo
   
   ·理科与文科的表现都较差，正确率不高于50%，但相比较而言文科正确率略高
   
   ③ gpt-4o
   
   ·理科：正确率高达94%，是所有模型中在理科题目上表现最好的，非常突出
   
   ·文科：虽然没有得到文科数据，但从理科的表现来看，可以推测它在文科的表现上不会太差
   
**3. 总体趋势**

   ①gpt-4o-mini在所有学科的表现上均优于gpt-35-turbo，但与自己相比，在个别学科（如政治和化学）中表现也有短板
   
   ②gpt-4o-mini在理科题目上的表现优于文科，且均优于gpt-35-turbo
   
   ③总体来看，gpt-4o在理科方面表现最佳，gpt-4o-mini次之，最后是gpt-35-turbo

# 致谢

