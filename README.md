# story-gen
A story generator gpt model influence by reasoning model Deepseek R1<br/>
The general idea is to teach the model to generate an outline or though before actually comes to generating the story. <br/>
After that,**without the need of dataset with actual story in it.**, the model is then trained based on its think and story (story length, correlation) just based on its generation of the prompts <br/>
<br/>
Pattern for my story represtation:
```
<prompt>
  {prompt}
<think>
  {think/story outline}
<story>
  {story}
<eos>
```
For **reinforcement learning** process, I performed **policy gradient** with **PPO algorithm** <br/> 
The **reward model** is calculate based on these criterias:
* **BERTScore** (weighted with **TF/IDF** with a precalculated **IDF** collected from the dataset)
* **Story length** (flexible and scale with the model performance)
* **2-gram words repetition penalty**
* **Personal format preference** (I prefer proses to mere conversational stories)
<br/>

Main steps: <br/>
Data Preperation
\-> [Pattern Learning](https://colab.research.google.com/drive/1Siri0UirPqgeHjKRThxrrnZVEPbQRHd-)
\-> [Reinforcement Learning](https://www.kaggle.com/code/nabinguyen/gpt-reasoning-rl)

# I. Data Preparation
Utilize the online dataset: [writingprompts dataset](https://huggingface.co/datasets/euclaise/writingprompts) - 272K data of prompts and short stories
## 1. Reasoning generation dataset:
Similar to Deepseek R1 **Cold Start** process, out of almost 300K datapoints, I sample 2K datapoints and generate a dataset contains an additional outline(think) row for each prompt and story.<br/>
Concluded by cleaning the dataset a bit, resulting in 1.97K data: [outline_story dataset](https://huggingface.co/datasets/hungq/outline-story-2000)
## 2. IDF Calculation
IDF is calculated based on both the original dataset and the generated dataset with outline, because the outline dataset is influenced by the generated model writing preferences.<br/>
TF is calculated later during generation while training
## 3. Data format
Finally, the dataset is formatted to match the pattern mentioned before

# II. Pattern learning
Notebook: [gpt-pattern-learning](https://colab.research.google.com/drive/1Siri0UirPqgeHjKRThxrrnZVEPbQRHd-) <br/>
The process of enforcing the model to follow the designated pattern
## 1. One-shot learning
For faster learning time, I provided a static set of simple data to guide the generation onto the right pattern.<br/>
The static dataset contains a few examples and is picked at random and prepended to the input prompt for generation.<br/>
The dataset and process is described in the **notebook**.
## 2. Zero-shot learning
After a mild one-shot training, the model may grasp the pattern and the \<think\> tag intention.<br/>
The model is then trained with no prepended example, preparing for reinforcement learning session.

# III. Reinforcement learning
Notebook: [gpt-reasoning-rl](https://www.kaggle.com/code/nabinguyen/gpt-reasoning-rl)
## Algorithm
Use PPO algorithm as mentioned.<br/>
Omitted the value model because the model is already well-trained, a newly created reward model may hazzard the generation model. Instead of calculating the advantages, he reward function will be tuning by the engineer's observation.<br/>
All the reward logic is encapsulated in the code (_rew_config_ and _reward_function()_)
