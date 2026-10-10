## <font style="color:rgb(31, 35, 40);">BLIP-2: Bootstrapping Language-Image Pre-training with Frozen Image Encoders and Large Language Models</font>
<font style="color:rgb(31, 35, 40);">This is the official implementation of BLIP-2 </font>[BLIP-2: Bootstrapping Language-Image Pre-training with Frozen Image Encoders and Large Language Models](https://arxiv.org/abs/2301.12597)<font style="color:rgb(31, 35, 40);">, a generic and efficient pre-training strategy that easily harvests development of pretrained vision models and large language models (LLMs) for vision-language pretraining. BLIP-2 beats Flamingo on zero-shot VQAv2 (</font>**<font style="color:rgb(31, 35, 40);">65.0</font>**<font style="color:rgb(31, 35, 40);"> vs </font>**<font style="color:rgb(31, 35, 40);">56.3</font>**<font style="color:rgb(31, 35, 40);">), establishing new state-of-the-art on zero-shot captioning (on NoCaps </font>**<font style="color:rgb(31, 35, 40);">121.6</font>**<font style="color:rgb(31, 35, 40);"> CIDEr score vs previous best </font>**<font style="color:rgb(31, 35, 40);">113.2</font>**<font style="color:rgb(31, 35, 40);">). Equipped with powerful LLMs (e.g. OPT, FlanT5), BLIP-2 also unlocks the new </font>**<font style="color:rgb(31, 35, 40);">zero-shot instructed vision-to-language generation</font>**<font style="color:rgb(31, 35, 40);"> capabilities for various interesting applications!</font>

![](https://github.com/salesforce/LAVIS/raw/main/projects/blip2/blip2_illustration.png)

### <font style="color:rgb(31, 35, 40);">Install:</font>
```python
pip install salesforce-lavis
```

<font style="color:rgb(31, 35, 40);">or install from source following LAVIS instruction.</font>

### <font style="color:rgb(31, 35, 40);">Demo:</font>
<font style="color:rgb(31, 35, 40);">Try out our </font>[LAVIS/examples/blip2_instructed_generation.ipynb at main · salesforce/LAVIS](https://github.com/salesforce/LAVIS/blob/main/examples/blip2_instructed_generation.ipynb)<font style="color:rgb(31, 35, 40);"> on instructed vision-to-language generation: </font>![](https://camo.githubusercontent.com/96889048f8a9014fdeba2a891f97150c6aac6e723f5190236b10215a97ed41f3/68747470733a2f2f636f6c61622e72657365617263682e676f6f676c652e636f6d2f6173736574732f636f6c61622d62616467652e737667)

### <font style="color:rgb(31, 35, 40);">BLIP-2 Model Zoo</font>
```python
# ==================================================
# Architectures                  Types
# ==================================================
# blip2_opt                      pretrain_opt2.7b, caption_coco_opt2.7b, pretrain_opt6.7b, caption_coco_opt6.7b
# blip2_t5                       pretrain_flant5xl, caption_coco_flant5xl, pretrain_flant5xxl
# blip2                          pretrain, coco
```

+ <font style="color:rgb(31, 35, 40);">Use</font><font style="color:rgb(31, 35, 40);"> </font>`<font style="color:rgb(31, 35, 40);">pretrained_{LLM}</font>`<font style="color:rgb(31, 35, 40);"> </font><font style="color:rgb(31, 35, 40);">model types for zero-shot image-to-text generation with prompts.</font>
+ <font style="color:rgb(31, 35, 40);">Use</font><font style="color:rgb(31, 35, 40);"> </font>`<font style="color:rgb(31, 35, 40);">caption_coco_{LLM}</font>`<font style="color:rgb(31, 35, 40);"> </font><font style="color:rgb(31, 35, 40);">model types to generate coco-style captions.</font>
+ <font style="color:rgb(31, 35, 40);">Use</font><font style="color:rgb(31, 35, 40);"> </font>`<font style="color:rgb(31, 35, 40);">blip2</font>`<font style="color:rgb(31, 35, 40);"> </font><font style="color:rgb(31, 35, 40);">model architecture for image-text feature extraction and retrieval.</font>

### <font style="color:rgb(31, 35, 40);">Image-to-text Generation Example</font>
<font style="color:rgb(31, 35, 40);">Let’s see how to use BLIP-2 models to perform zero-shot instructed image-to-text generation. We first load a sample image from local.</font>

```python
import torch
from PIL import Image
# setup device to use
device = torch.device("cuda") if torch.cuda.is_available() else "cpu"
# load sample image
raw_image = Image.open("../../docs/_static/merlion.png").convert("RGB")
display(raw_image.resize((596, 437)))
```

<font style="color:rgb(31, 35, 40);">Then we load a pre-trained BLIP-2 model with its preprocessors (transforms).</font>

```python
import torch
from lavis.models import load_model_and_preprocess
# loads BLIP-2 pre-trained model
model, vis_processors, _ = load_model_and_preprocess(name="blip2_t5", model_type="pretrain_flant5xxl", is_eval=True, device=device)
# prepare the image
image = vis_processors["eval"](raw_image).unsqueeze(0).to(device)
```

<font style="color:rgb(31, 35, 40);">Given the image and a text prompt, ask the model to generate the response.</font>

```python
model.generate({"image": image, "prompt": "Question: which city is this? Answer:"})
# 'singapore'
```

<font style="color:rgb(31, 35, 40);">Ask the model to explain its answer.</font>

```python
model.generate({
    "image": image,
    "prompt": "Question: which city is this? Answer: singapore. Question: why?"})
# 'it has a statue of a merlion'
```

<font style="color:rgb(31, 35, 40);">Ask a follow-up question.</font>

```python
# prepare context prompt
context = [
    ("which city is this?", "singapore"),
    ("why?", "it has a statue of a merlion"),
]
question = "where is the name merlion coming from?"
template = "Question: {} Answer: {}."
prompt = " ".join([template.format(context[i][0], context[i][1]) for i in range(len(context))]) + " Question: " + question + " Answer:"
print(prompt)
# generate model's response
model.generate({"image": image,"prompt": prompt})
# 'merlion is a portmanteau of mermaid and lion'
```

### <font style="color:rgb(31, 35, 40);">Feature Extraction Example</font>
<font style="color:rgb(31, 35, 40);">BLIP-2 supports the Unified Feature Extraction Interface of LAVIS. Checkout this </font>[LAVIS/examples/blip2_feature_extraction.ipynb at 3446bac20c5646d35ae383ebe6d13cec4f8b00cb · salesforce/LAVIS](https://github.com/salesforce/LAVIS/blob/3446bac20c5646d35ae383ebe6d13cec4f8b00cb/examples/blip2_feature_extraction.ipynb)<font style="color:rgb(31, 35, 40);"> for an example.</font>

### <font style="color:rgb(31, 35, 40);">Image-Text Matching Example</font>
<font style="color:rgb(31, 35, 40);">BLIP-2 can compute the image-text matching score using the same interface as BLIP. Checkout this </font>[LAVIS/examples/blip2_image_text_matching.ipynb at 3446bac20c5646d35ae383ebe6d13cec4f8b00cb · salesforce/LAVIS](https://github.com/salesforce/LAVIS/blob/3446bac20c5646d35ae383ebe6d13cec4f8b00cb/examples/blip2_image_text_matching.ipynb)<font style="color:rgb(31, 35, 40);"> for an example.</font>

### <font style="color:rgb(31, 35, 40);">Benchmark Evaluation</font>
<font style="color:rgb(31, 35, 40);">Follow </font>[Dataset Zoo — LAVIS documentation](https://opensource.salesforce.com/LAVIS//latest/getting_started.html#auto-downloading-and-loading-datasets)<font style="color:rgb(31, 35, 40);"> to prepare common vision-language datasets.</font>

<font style="color:rgb(31, 35, 40);">Run </font>[LAVIS/run_scripts/blip2/eval at main · salesforce/LAVIS](https://github.com/salesforce/LAVIS/tree/main/run_scripts/blip2/eval)<font style="color:rgb(31, 35, 40);"> for evaluating pretrained and finetuned models.</font>

### <font style="color:rgb(31, 35, 40);">Training</font>
<font style="color:rgb(31, 35, 40);">Stage-1 Pre-training (from scratch):</font><font style="color:rgb(31, 35, 40);"> </font>`<font style="color:rgb(31, 35, 40);">bash run_scripts/blip2/train/pretrain_stage1.sh</font>`

<font style="color:rgb(31, 35, 40);">Stage-2 Pre-training:</font><font style="color:rgb(31, 35, 40);"> </font>`<font style="color:rgb(31, 35, 40);">bash run_scripts/blip2/train/pretrain_stage2.sh</font>`

<font style="color:rgb(31, 35, 40);">Finetune for image captioning:</font><font style="color:rgb(31, 35, 40);"> </font>`<font style="color:rgb(31, 35, 40);">bash run_scripts/blip2/train/train_caption_coco.sh</font>`

<font style="color:rgb(31, 35, 40);">The</font><font style="color:rgb(31, 35, 40);"> </font>[<font style="color:rgb(31, 35, 40);">config files</font>](https://github.com/salesforce/LAVIS/tree/main/lavis/projects/blip2/train)<font style="color:rgb(31, 35, 40);"> </font><font style="color:rgb(31, 35, 40);">can be modified for customized training.</font>

### <font style="color:rgb(31, 35, 40);">Citing BLIP-2</font>
```python
@inproceedings{li2023blip2,
               title={{BLIP-2:} Bootstrapping Language-Image Pre-training with Frozen Image Encoders and Large Language Models}, 
author={Junnan Li and Dongxu Li and Silvio Savarese and Steven Hoi},
year={2023},
booktitle={ICML},
}
```

### <font style="color:rgb(31, 35, 40);">🤗</font><font style="color:rgb(31, 35, 40);"> Hugging Face integration</font>
<font style="color:rgb(31, 35, 40);">BLIP-2 is integrated into the Hugging Face </font><font style="color:rgb(31, 35, 40);">🤗</font><font style="color:rgb(31, 35, 40);"> </font>[<font style="color:rgb(31, 35, 40);">Transformers</font>](https://github.com/huggingface/transformers)<font style="color:rgb(31, 35, 40);"> </font><font style="color:rgb(31, 35, 40);">library, and allows to leverage int8 quanitization thanks to</font><font style="color:rgb(31, 35, 40);"> </font>[<font style="color:rgb(31, 35, 40);">bitsandbytes</font>](https://github.com/TimDettmers/bitsandbytes)<font style="color:rgb(31, 35, 40);">. This roughly halves the amount of memory required to load the model, without performance degradation.</font>

<font style="color:rgb(31, 35, 40);">Documentation can be found </font>[here](https://huggingface.co/docs/transformers/main/model_doc/blip-2)<font style="color:rgb(31, 35, 40);">.</font>

<font style="color:rgb(31, 35, 40);">Usage in half precision (float16) is as follows:</font>

```python
from PIL import Image
import requests
from transformers import Blip2Processor, Blip2ForConditionalGeneration
import torch

device = "cuda" if torch.cuda.is_available() else "cpu"

processor = Blip2Processor.from_pretrained("Salesforce/blip2-opt-2.7b")
model = Blip2ForConditionalGeneration.from_pretrained(
    "Salesforce/blip2-opt-2.7b", torch_dtype=torch.float16
)
model.to(device)
url = "http://images.cocodataset.org/val2017/000000039769.jpg"
image = Image.open(requests.get(url, stream=True).raw)

inputs = processor(images=image, return_tensors="pt").to(device, torch.float16)

generated_ids = model.generate(**inputs)
generated_text = processor.batch_decode(generated_ids, skip_special_tokens=True)[0].strip()
print(generated_text)
```

<font style="color:rgb(31, 35, 40);">To leverage the int8 algorithm, you can run the model as follows:</font>

```python
import torch
import requests
from PIL import Image
from transformers import Blip2Processor, Blip2ForConditionalGeneration

processor = Blip2Processor.from_pretrained("Salesforce/blip2-opt-2.7b")
model = Blip2ForConditionalGeneration.from_pretrained("Salesforce/blip2-opt-2.7b", load_in_8bit=True, device_map="auto")

img_url = 'https://storage.googleapis.com/sfr-vision-language-research/BLIP/demo.jpg' 
raw_image = Image.open(requests.get(img_url, stream=True).raw).convert('RGB')

question = "how many dogs are in the picture?"
inputs = processor(raw_image, question, return_tensors="pt").to("cuda", torch.float16)

out = model.generate(**inputs)
print(processor.decode(out[0], skip_special_tokens=True))
```

<font style="color:rgb(31, 35, 40);">All models can be found on the：  </font>

[https://huggingface.co/models?other=blip-2](https://huggingface.co/models?other=blip-2)

<font style="color:rgb(31, 35, 40);">.</font>

