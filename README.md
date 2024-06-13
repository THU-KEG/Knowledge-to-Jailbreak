# Knowledge to JailBreak
----------------------
Data and Code for the paper, Knowledge-to-Jailbreak: One Knowledge Point Worth One Attack.


## Installation
----------------------
To download the repo to the local machine, run the following command:
```bash
git clone <link>
cd Knowledge-to-Jailbreak
```

## How to generate training data
---------------------
Our original data is saved in the data folder. 

Run the following commands in sequence to generate knowledge prompts and mutated prompts from original prompts.
```bash
# Generate responses to raw prompts.
python3 1_gen_origin_response.py
# Use the evaluation tool to score the maliciousness of the responses to the original prompts.
python3 2_gen_origin_response_score.py
# Find the most similar knowledge for each prompt.
python3 3_1_find_knowledge_for_prompt.py
# Add subject labels to prompts.
python3 3_2_tag_subject_for_prompt.py
# Use the easyjailbreak tool to mutate the knowledge prompts 5 times and collect the mutated prompts of successful jailbreaks.
python3 4_mutate_to_jailbreak.py
```

Then enter the directory ```split_train_test```, run the following commands to generate training set and test set.
```bash
cd split_train_test

python3 simplify.py
python3 split.py
```

Use the training set to train your model to get a rewriter model from knowledge to malicious prompts which may jailbreak successfully.

## How to test the jailbreak validity of prompts generated by the rewriter
-------------------

### Generate response

Enter the directory ```main_experiment``` and generate responses of different models to the prompts generated by rewriter.

```bash
# Get original prompts, knowledge prompts and generated prompts.
python3 get_all_test_prompt
```

Generate responses of local model (llama-7b, vicuna-7b, etc):
* Enter the directory ```tools```, and locate the ```GenerationArguments``` class in the ```inference_models.py``` file and select the local model you need.
  ```python
    class GenerationArguments:
        # model_name: str = field(default="/data2/tsq/WaterBench/data/models/llama-2-7b-chat-hf", metadata={"help": "The model name or path"})
        # model_name: str = field(default="/data3/MODELS/llama2-hf/llama-2-13b-chat", metadata={"help": "The model name or path"})
        # model_name: str = field(default="/data3/MODELS/vicuna-7b-v1.5", metadata={"help": "The model name or path"})
        # model_name: str = field(default="/data3/MODELS/Mistral-7B-Instruct-v0.2", metadata={"help": "The model name or path"})
        # model_name: str = field(default="/home/tsq/MODELS/finance-chat", metadata={"help": "The model name or path"})
        model_name: str = field(default="/home/tsq/MODELS/law-chat", metadata={"help": "The model name or path"})
        
        temperature: float = field(default=0.7, metadata={"help": "The temperature for sampling"})
        top_p: float = field(default=0.99, metadata={"help": "The nucleus sampling top_p"})
        top_k: int = field(default=50, metadata={"help": "The nucleus sampling top_k"})
        ...
  ```
* Run ```test_local_model.py``` in the ```main_experiment``` folder
  ```bash
  python3 test_local_model.py
  ```

Generate responses of remote model (GPT-4, GPT-3.5, etc):
* Locate the ```get_response``` function in the ```test_remote_model.py``` folder and select the model you want to test.
  ```python 
    model_names = ["gpt-4", "gpt-4-0613", "gpt-4-1106-preview", "gpt-3.5-turbo", "gpt-3.5-turbo-0613", "gpt-3.5-turbo-1106"]
    url = "https://chatapi.onechats.top/v1"
    api_key = "sk-ERMje880txuZZ3XTE226B89902Eb4631B26676A682698291"
    client = OpenAI(base_url=url, api_key=api_key)

    def get_response(prompt):
        response = client.chat.completions.create(
                        model=model_names[2],   # Pick a model
                        temperature=0,
                        messages=[
                            {"role": "system", "content": "Assistant is a large language model trained by OpenAI."},
                            {"role": "user", "content": prompt}
                        ]
                    )
        return response.choices[0].message.content
  ```
* Run ```test_remote_model.py```
  ```bash
  python3 test_remote_model.py
  ```

### Generate harm score of response
Enter the directory ```harm_score```.
Run ```gen_harm_score.py``` to score the harm level of responses generated by each model.
```bash
python3 gen_harm_score.py
```
Run ```fill_table.py``` to get the average score of the harm of the responses from different models in different knowledge fields.
```bash
python3 fill_table.py
```

### Generate ASR to models
Enter the directory ```cal_ASR```.
Run ```gen_ASR.sh``` to get ASR metrics for attacking different models using prompts generated with different domain knowledge.
```bash
./gen_ASR.sh
```
Run gen_ASR.py to get a summary of these ASRs
```python
python3 gen_ASR.py
```