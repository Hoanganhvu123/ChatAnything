---
title: ChatAnything
emoji: 👀
colorFrom: green
colorTo: green
sdk: gradio
sdk_version: 3.41.0
app_file: app.py
python_version: 3.8.10
pinned: false
---
# ChatAnything: Facetime Chat with LLM-Enhanced Personas

**Yilin Zhao\*, Xinbin Yuan\*, Shanghua Gao\*, Zhijie Lin, Qibin Hou, Jiashi Feng, Daquan Zhou\***  



> What will it be like to Facetime any imaginary concepts? Do you want to see what the LLM-based agent you have been chatting with looks like?
To animate anything, we integrated current open-source models at hand for an animation application for interactive AI-Agent chatting usage. 
>
> Assign any agents a visual appearance!
> 
> To start with, take a look at these incredible faces generated with open-source Civitai models that are to be animated.
> 
<img align="center" src="./resources/readme/short_demo.gif" alt="drawing" width="768"/>

<!-- ![faces](./resources/readme/show.png) -->

Here we provide you with ChatAnything. A simple pipeline Enhanced with currently limitless Large Language Models, yielding imaginary Facetime chats with intended visual appearance!

Remember, the repo and application are totally based on pre-trained deep learning methods and haven't included any training yet. We give all the credit to the open-source community (shout out to you). For details on the pipeline, see our [technical report](https://arxiv.org/abs/2311.06772)
## Updates @Nov 26, 2023
We now add support for open-source LLM-LLaMA to speed up the pipeline. You can try it now!

## Project Milestone Log

- [ ] Fine-tune face rendering module.
- [ ] Better TTS module & voice render module.
- [x] Adding Open-source Language Models.
- [x] Initial release
  - Facetime Animation.
  - Multiple model choices for initial frame generation.
  - Multiple choices for voices.
# Install & Run
Just follow the instructions. Everything would be simple (hopefully). Reach out if you met with any problems!
### Install
First, install the virtual environment.
```
conda env create -f environment.yaml

# then install
conda env update --name chatanything --file environment.yaml
```

The Pipeline integrated Open-Source Models. All Models are to be found online(see [Acknowledgement](#acknowledgement)). We put some important models together on huggingface remotes just to make life easier. Prepare them for the first run with this Python script [prepare_models.py](./python_scripts/prepare_models.py):
```
# prepare the local models
python python_scripts/prepare_models.py

```

### Building Docker
Try build a docker if you find it easier. This part is not fully tested. If you find anything wrong, feel free to contribute~
```
docker build --network=host -t chatanything .
# docker run -dp 127.0.0.1:8901:8901 chatanything
docker run -p 127.0.0.1:8901:8901 -it --gpus all chatanything 
docker run -it --gpus all chatanything bash
``` 

### (Optional) Use Local LLM
You can also use your local LLM instead of ChatGPT to power ChatAnything.
Create a new environment for the [fastchat](https://github.com/lm-sys/FastChat) LLM service ( fastchat is incompatible with python-3.8 which is the required python for ChatAnything). Then follow the Instruct to install the environment and start the Language Model Service. The local LLM chat service would require around 14G of GPU memory for the 7B LLM Model. Run the Language Model service carefully. 
```
# install FastChat
pip install "fschat[model_worker,webui]"

# start a local server
bash script/init_local_llm.sh

# Export local server path
export OPENAI_API_BASE=http://localhost:8000/v1
export OPENAI_API_KEY=EMPTY
```

### Run
Specify a port for the gradio application to run on and set off!
```
PORT=8809 python app.py $PORT
```


# Configuring: From User Input Concept to Appearance & Voice
The first step of the pipeline is to generate an image for SadTalker and at the same time set up the Text to Sound Module for voice chat.

The pipeline would query a powerful LLM (ChatGPT) for the selection in a zero-shot multi-choice selection format.
Three Questions are asked at the initial of every conversation(init frame generation):
1. Provide an image personality for the user input concept.
2. Select a Generative model for the init frame generation.
3. Select a Text-to-Shoud Voice(Model) for the character based on the personality.

We have constructed the model selection to be extendable. Add your ideal model with just a few lines of Configuring! The rest of this section will briefly introduce the steps to add an init-frame generator/language voice.

### Image Generator
Configure the models in the [Model Config](./resources/models.yaml). This Config acts as the memory (or an image-generating tool pool) for the LLM.

The prompt sets up this selection process. Each subfield of the "models" would turn into an option in the multiple-choice question.
the "**desc**" field of each element is what the Language Model would see. The key is not provided to the LM as it would sometimes mislead it.
the others are used for the image generation as listed: 
1. model_dir: the repo-path for diffusers package. As the pre-trained Face-landmark ControlNet is based on stable-diffusion-v1-5, we currently only support the derivatives of it.
2. lora_path: LoRA derivatives are powerful, try a LoRA model also for better stylization. Should directly point to the parameters binary file.
3. prompt_template & negative_prompt: this is used for prompting the text-to-image diffusion model. Find an ideal prompt for your model and stick with it. A "{}" should be in the prompt template for inserting the user input concept.

Here are some **Tips** for configuring your own model.
1. Provide the LLM with a simple description of the generative model. It is worth noting that the description needs to be concise and accurate for a correct selection. 
2. Set the model_dir to a local directory of diffusers stable-diffusion-v1-5 derivatives. Also, you can provide a repo-id on the huggingface hub model space. The model would be downloaded when first chosen, wait for it.
3. To better utilize the resources from the community, we also add in support of the LoRA features. To add the LoRA module, you would need to give the path to the parameter files.

4. Carefully write the prompt template and negative prompt. These affect the initial face generation a lot. Be aware that the prompt template should contain only one pair of "{}" to insert the concept that users wrote on the application webpage. We support the Stable-Diffusion-Webui prompt style as implemented by diffusers, feel free to copy the prompt from Civitai for better prompting the generation and put in the "{}" to the original prompt for ChatAnything!

Again, this model's config acts as an extended tool pool for the LM, the application would drive the LM to choose from this config and use the chosen model to generate. Sometimes the LM fails to choose the correct model or choosing any available model, this would cause the Chatanything app to fail on a generation.

Notice we currently support ONLY stable-diffusion-v1.5 derivatives (Sdxl Pipelines are under consideration, however not yet implemented as we lack a face-landmark ControlNet for it. Reach out if you're interested in training one!)

### Voice TTS
We are using the edge_tts package for text-to-speech support. The voice selection and [voice configuration file](./resources/voices_edge.yaml) is constructed similarly to the Image generation model selection, except now the LM is supposed to choose the voice base on the personality description given by itself earlier. "**gender**" and "**language**" field corresponds to edge_tts.

# On-going tasks.
### Customized Voice.
There is a Voice Changer TextToSpeach-SpeachVoiceConversion Pipeline app, which ensures a better-customized voice. We are trying to leverage its TTS functionality. 

Reach out if you want to add a voice of your own or your hero!

Here are the possible steps for 
You would need to change a little bit in the code first:
1. Alter this [code](./utils.py#14) to import a TTSTalker from chat_anything/tts_talker/tts_voicechanger.py.
2. switch the config to another one, change [code](./utils.py#14) "resources/voices_edge.yaml" -> "resources/voices_voicechanger.yaml"

Try running a [Voice Changer](https://huggingface.co/spaces/kevinwang676/Voice-Changer) on your local machine. Simply set up git-lfs and install the repo and run it for the TTS voice service.
The TTS caller was set to port 7860. 

Make sure the client class is set up with the same port in [here](chat_anything/tts_talker/tts_voicechanger.py#5)
```python
client = Client("http://127.0.0.1:7860/")
```

# Acknowledgement
Again, the project hasn't yet included any training. The pipeline is totally based on these incredible awesome packages and pretrained models. Don't hesitate to take a look and explore the amazing open-source generative communities. We love you, guys.
- [ChatGPT](https://openai.com/chatgpt): GOD
- [SadTalker](https://github.com/OpenTalker/SadTalker): The Core Animation Module
- [Face-Landmark-ControlNet](https://huggingface.co/georgefen/Face-Landmark-ControlNet): An Awesome ControlNet with Face landmark using Stable Diffusion 1.5 as base Model.
- [diffusers](https://github.com/huggingface/diffusers): GOAT of Image Generative Framework🥳.
- [langchain](https://github.com/langchain-ai/langchain): An Awesome Package for Dealing with LLM.
- [edge-tts](https://github.com/rany2/edge-tts): An Awesome Package for Text To Sound Solutions.
- [gradio](https://www.gradio.app/): GOAT😄 Machine Learning based App framework.
- [fastchat](https://github.com/lm-sys/FastChat): Amazing Open-source LLM service!
- [Civitai](https://civitai.com/models) and [Huggingface_hub](https://huggingface.co/models): Find your ideal Image Generative Model on Civitai. These Communities are Crazy🥂. Here are Some Fantastic Derivatives of [stable-diffusion-v1-5](https://huggingface.co/runwayml/stable-diffusion-v1-5):
    - [Game Icon Institute_mode](https://civitai.com/models/47800?modelVersionId=76533)
    - [dreamshaper](https://civitai.com/models/4384/dreamshaper)
    - [3D_Animation_Diffusion](https://civitai.com/models/118086?modelVersionId=128046)
    - [anything-v5](https://huggingface.co/stablediffusionapi/anything-v5)

# Citation
If you like our pipeline and application, don't hesitate to reach out! Let's work on it and see how far it would go!
```bibtex
@misc{zhao2023ChatAnything,
      title={ChatAnything: Facetime Chat with LLM-Enhanced Personas}, 
      author={Yilin, Zhao and Xinbin, Yuan and Shanghua, Gao and Zhijie Lin and Qibin, Hou and Jiashi, Feng and Daquan, Zhou},
      publisher={arXiv:2311.06772},
      year={2023},
}
```
