<!---
Copyright 2020 The HuggingFace Team. All rights reserved.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
-->

<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://huggingface.co/datasets/huggingface/documentation-images/raw/main/transformers-logo-dark.svg">
    <source media="(prefers-color-scheme: light)" srcset="https://huggingface.co/datasets/huggingface/documentation-images/raw/main/transformers-logo-light.svg">
    <img alt="Hugging Face Transformers Library" src="https://huggingface.co/datasets/huggingface/documentation-images/raw/main/transformers-logo-light.svg" width="352" height="59" style="max-width: 100%;">
  </picture>
  <br/>
  <br/>
</p>

<p align="center">
    <a href="https://circleci.com/gh/huggingface/transformers"><img alt="Build" src="https://img.shields.io/circleci/build/github/huggingface/transformers/main"></a>
    <a href="https://github.com/huggingface/transformers/blob/main/LICENSE"><img alt="GitHub" src="https://img.shields.io/github/license/huggingface/transformers.svg?color=blue"></a>
    <a href="https://huggingface.co/docs/transformers/index"><img alt="Documentation" src="https://img.shields.io/website/http/huggingface.co/docs/transformers/index.svg?down_color=red&down_message=offline&up_message=online"></a>
    <a href="https://github.com/huggingface/transformers/releases"><img alt="GitHub release" src="https://img.shields.io/github/release/huggingface/transformers.svg"></a>
    <a href="https://github.com/huggingface/transformers/blob/main/CODE_OF_CONDUCT.md"><img alt="Contributor Covenant" src="https://img.shields.io/badge/Contributor%20Covenant-v2.0%20adopted-ff69b4.svg"></a>
    <a href="https://zenodo.org/badge/latestdoi/155220641"><img src="https://zenodo.org/badge/155220641.svg" alt="DOI"></a>
</p>

<h4 align="center">
    <p>
        <b>English</b> |
        <a href="https://github.com/huggingface/transformers/blob/main/i18n/README_zh-hans.md">简体中文</a> |
        <a href="https://github.com/huggingface/transformers/blob/main/i18n/README_zh-hant.md">繁體中文</a> |
        <a href="https://github.com/huggingface/transformers/blob/main/i18n/README_ko.md">한국어</a> |
        <a href="https://github.com/huggingface/transformers/blob/main/i18n/README_es.md">Español</a> |
        <a href="https://github.com/huggingface/transformers/blob/main/i18n/README_ja.md">日本語</a> |
        <a href="https://github.com/huggingface/transformers/blob/main/i18n/README_hd.md">हिन्दी</a> |
        <a href="https://github.com/huggingface/transformers/blob/main/i18n/README_ru.md">Русский</a> |
        <a href="https://github.com/huggingface/transformers/blob/main/i18n/README_pt-br.md">Рortuguês</a> |
        <a href="https://github.com/huggingface/transformers/blob/main/i18n/README_te.md">తెలుగు</a> |
        <a href="https://github.com/huggingface/transformers/blob/main/i18n/README_fr.md">Français</a> |
        <a href="https://github.com/huggingface/transformers/blob/main/i18n/README_de.md">Deutsch</a> |
        <a href="https://github.com/huggingface/transformers/blob/main/i18n/README_vi.md">Tiếng Việt</a> |
        <a href="https://github.com/huggingface/transformers/blob/main/i18n/README_ar.md">العربية</a> |
        <a href="https://github.com/huggingface/transformers/blob/main/i18n/README_ur.md">اردو</a> |
    </p>
</h4>

<h3 align="center">
    <p>State-of-the-art Machine Learning for JAX, PyTorch and TensorFlow</p>
</h3>

<h3 align="center">
    <a href="https://hf.co/course"><img src="https://huggingface.co/datasets/huggingface/documentation-images/resolve/main/course_banner.png"></a>
</h3>

🤗 Transformers provides thousands of pretrained models to perform tasks on different modalities such as text, vision, and audio.

These models can be applied on:

* 📝 Text, for tasks like text classification, information extraction, question answering, summarization, translation, and text generation, in over 100 languages.
* 🖼️ Images, for tasks like image classification, object detection, and segmentation.
* 🗣️ Audio, for tasks like speech recognition and audio classification.

Transformer models can also perform tasks on **several modalities combined**, such as table question answering, optical character recognition, information extraction from scanned documents, video classification, and visual question answering.

🤗 Transformers provides APIs to quickly download and use those pretrained models on a given text, fine-tune them on your own datasets and then share them with the community on our [model hub](https://huggingface.co/models). At the same time, each python module defining an architecture is fully standalone and can be modified to enable quick research experiments.

🤗 Transformers is backed by the three most popular deep learning libraries — [Jax](https://jax.readthedocs.io/en/latest/), [PyTorch](https://pytorch.org/) and [TensorFlow](https://www.tensorflow.org/) — with a seamless integration between them. It's straightforward to train your models with one before loading them for inference with the other.

## Adapting 🤗 Transformers for Hidden State Tracking and Intervention
This code base inserts a few lines of code into the modeling_XX.py files of various models to enable hidden state tracking and intervention. I've tried to make the inserts minimal, and not break the rest of the library in any way. Essentially the code saves the mlp and attention outputs (both before mixing it in with the residuals) as well the projection of the mlp output to the vocabulary space as numpy arrays to a dictionary in the model. The config file is used to specific arguments for which layers and hidden states should be tracked. Given a new CausalLM, here are the steps needed to set up this system. Go to the models modeling_XX.py file (example modeling_llama.py):

1. Create the probe_copy_util function:
```python
# **** 
# HIDDEN PROBE CODE
# ****
def probe_util_copy(x):
    return x.detach().cpu().clone()
```
2. In the DecoderLayer Class, add the following code at the end of the __init__ function
```python
        # Rest of the init above
        # **** 
        # HIDDEN PROBE CODE
        # ****
        if "track_layers" in config:
            self.track_layers = [x - 1 for x in config.track_layers] # Makes the offset here because 0 is embedding
        else:
            self.track_layers = []
        if "track_attention" in config:
            self.track_attention = config.track_attention
        else:
            self.track_attention = False
        if "track_mlp" in config:
            self.track_mlp = config.track_mlp 
        else:
            self.track_mlp = False
        if "track_projection" in config:
            self.track_projection = config.track_projection
        else:
            self.track_projection = False
        if "clamp_layers" in config:
            self.clamp_layers = config.clamp_layers
        else:
            self.clamp_layers = {}

        if "shift_activations" in config:
            self.shift_activations = config.shift_activations
        else:
            self.shift_activations = {} # dictionary of layer_idx -> {hidden_key -> (numpy vector to shift along, shift_strength)}
        assert self.clamp_layers == {} or self.shift_activations == {}, "Cannot have both clamp_layers and shift_activations set"
        if layer_idx + 1 not in self.clamp_layers:
            self.do_attention_clamp = False # should be set by the config
            self.do_mlp_clamp = False
        else:
            clamp_instructions = self.clamp_layers[layer_idx + 1]
            if "attention" in clamp_instructions:
                self.attention_clamp_low = clamp_instructions["attention"].get("min", None)
                self.attention_clamp_high = clamp_instructions["attention"].get("max", None)
            else:
                self.attention_clamp_low = None
                self.attention_clamp_high = None
            self.do_attention_clamp = self.attention_clamp_low is not None or self.attention_clamp_high is not None
            if "mlp" in clamp_instructions:
                self.mlp_clamp_low = clamp_instructions["mlp"].get("min", None)
                self.mlp_clamp_high = clamp_instructions["mlp"].get("max", None)
            else:
                self.mlp_clamp_low = None
                self.mlp_clamp_high = None
            self.do_mlp_clamp = self.mlp_clamp_low is not None or self.mlp_clamp_high is not None

        if layer_idx +1 not in self.shift_activations:
            self.shift_attention = False
            self.shift_mlp = False
        else:
            shift_instructions = self.shift_activations[layer_idx + 1]
            if "attention" in shift_instructions:
                self.shift_attention_vector = shift_instructions["attention"]["vector"]
                self.shift_attention_strength = shift_instructions["attention"]["strength"]
                self.shift_attention = True
            else:
                self.shift_attention = False
            if "mlp" in shift_instructions:
                self.shift_mlp_vector = shift_instructions["mlp"]["vector"]
                self.shift_mlp_strength = shift_instructions["mlp"]["strength"]
                self.shift_mlp = True
            else:
                self.shift_mlp = False

        self.probe_hidden_output = {}
        if layer_idx in self.track_layers:
            if self.track_attention:
                self.probe_hidden_output["attention"] = None
            if self.track_mlp:
                self.probe_hidden_output["mlp"] = None
            if self.track_projection:
                self.probe_hidden_output["projection"] = None
```
3. Right after this, create a function in the DecoderLayer class to clear the probe_hidden_outputs
```python
    def probe_reset_hidden_output(self):
        for key in self.probe_hidden_output.keys():
            self.probe_hidden_output[key] = None
```
4. Next, in the __forward__ function of the DecoderLayer class, add the calls to track and intervene on the attention, mlp and projection outputs. For attention and mlp I have chosen to put them before they are mixed into the residual stream, but this is a design decision and you can put the code in either place it will run without issues
```python
        # after hidden_states is output by attention module
        # **** 
        # HIDDEN PROBE CODE
        # ****
        if "attention" in self.probe_hidden_output:
            self.probe_hidden_output["attention"] = probe_util_copy(hidden_states)
        if self.do_attention_clamp:
            hidden_states = torch.clamp(hidden_states, self.attention_clamp_low, self.attention_clamp_high)
        if self.shift_attention:
            attention_torch_vector = torch.tensor(self.shift_attention_vector, device=hidden_states.device)
            hidden_states = hidden_states + self.shift_attention_strength * attention_torch_vector    

        # after hidden_states is output by mlp module
        # **** 
        # HIDDEN PROBE CODE
        # ****
        if "mlp" in self.probe_hidden_output:
            self.probe_hidden_output["mlp"] = probe_util_copy(hidden_states)
        if self.do_mlp_clamp:
            hidden_states = torch.clamp(hidden_states, self.mlp_clamp_low, self.mlp_clamp_high)
        if self.shift_mlp:
            mlp_torch_vector = torch.tensor(self.shift_mlp_vector, device=hidden_states.device)
            hidden_states = hidden_states + self.shift_mlp_strength * mlp_torch_vector
        
        # after hidden_states is added to the residual (block output of the decoder, once again design decision)
        # **** 
        # HIDDEN PROBE CODE
        # ****
        if "projection" in self.probe_hidden_output:
            self.probe_hidden_output["projection"] = hidden_states[:, -1] # don't convert this to numpy yet, we will do torch ops on this later        
```
5. Go to the Model class (not PretrainedModel) and add the tracking setup code to the __init__ function
```python
        # Rest of the init
        # **** 
        # HIDDEN PROBE CODE
        # ****
        if "track_layers" in config:
            self.track_layers = config.track_layers  # if 0 track embedding, if i track i-1th layer from self.layers
        else:
            self.track_layers = []
        if "track_mlp" in config:
            self.track_mlp = config.track_mlp
        self.probe_hidden_output = {}
        for layer_idx in self.track_layers:
            self.probe_hidden_output[layer_idx] = {}
```
6. Right after that add a function to the Model class
```python
    def probe_reset_hidden_output(self):
        for layer_idx in self.probe_hidden_output:
            self.probe_hidden_output[layer_idx] = {}
```
7. Go to the Model class __forward__ function and add
```python
        # After hidden states is output of embedding module
        # **** 
        # HIDDEN PROBE CODE
        # ****
        if 0 in self.track_layers:
            if self.track_mlp:
                self.probe_hidden_output[0]["mlp"] = probe_util_copy(hidden_states)
        # Right before return
        # **** 
        # HIDDEN PROBE CODE
        # ****
        for layer_idx in self.probe_hidden_output:
            if layer_idx == 0:
                continue
            hidden_values_dict = self.layers[layer_idx-1].probe_hidden_output
            for key in hidden_values_dict:
                self.probe_hidden_output[layer_idx][key] = hidden_values_dict[key]
            self.layers[layer_idx-1].probe_reset_hidden_output()
```
8. Go to the ModelForCausalLM class and add to the __init__ function
```python
        # after init
        # **** 
        # HIDDEN PROBE CODE
        # ****
        self.probe_hidden_output = []
        if "track_projection" in config:
            self.track_projection = config.track_projection
```
9. Add another function to the ModelForCausalLM class
```python
    def probe_reset_hidden_output(self):
        self.probe_hidden_output = []
        self.apparent_best_tokens = []
```
10. Add a section in the __forward__ function of ModelForCausalLM class
```python
        # right before returning
        # **** 
        # HIDDEN PROBE CODE
        # ****
        if self.track_projection:
            chosen_token = logits[0, -1].argmax(dim=-1).detach().cpu().item()
            for layer in self.model.probe_hidden_output:
                true_projected = self.lm_head(self.model.probe_hidden_output[layer]["projection"])
                true_projected = torch.nn.functional.log_softmax(true_projected, dim=-1)[:, chosen_token].reshape(-1, 1)
                self.model.probe_hidden_output[layer]["projection"] = probe_util_copy(true_projected)
                del true_projected
        self.probe_hidden_output.append(self.model.probe_hidden_output.copy())
        self.model.probe_reset_hidden_output()
```

And you should be done! The config file controls the options here, if you dont put in any values it defaults to switching the features off and if you dont add any options it should just run the normal transformers code without any changes. The new config file options are:
```
track_layers: list of layers to track
track_mlp: boolean
track_attention: boolean
track_projection: boolean
do_attention_clamp: boolean
do_mlp_clamp: boolean
clamp_layers: dict with structure {layer_idx: {"hidden_type": {"max": numerical or array, "min": numerical or array}}} # once again, if you want to clamp the first decoder layer layer_idx in the dict should be 1 not 0
```

I have a higher level codebase that is not included in this repo for reading from this, one relevant function you may want to use is:
```python
def detect_hidden_states_parameters(single_token_probe_hidden_output):
    assert isinstance(single_token_probe_hidden_output, dict)
    layers = list(single_token_probe_hidden_output.keys())
    assert len(layers) > 0
    input_length = None
    batch_size = None
    keys = []
    layer = layers[0]
    if len(single_token_probe_hidden_output[layer]) == 0:
                warnings.warn("Improper initialization (layer has no keys)")
                return None
    for key in single_token_probe_hidden_output[layer]:
        if batch_size is None or input_length is None:
            value = single_token_probe_hidden_output[layer][key]
            if value is None:
                warnings.warn("Trying to read hidden states before they are set / after they are reset.")
                return None
            else:
                batch_size, input_length, _ = value.shape
        keys.append(key)
    return layers, keys, input_length, batch_size


def read_hidden_states(probe_hidden_output):
    if not isinstance(probe_hidden_output, list):
         probe_hidden_output = [probe_hidden_output]
    n_tokens_generated = len(probe_hidden_output)
    if n_tokens_generated == 0:
        warnings.warn("No tokens generated yet or already reset")
        return None

    details = detect_hidden_states_parameters(probe_hidden_output[0])
    if details is None:
        return None
    layers, keys, input_length, batch_size = details
    ret = {}
    for layer in layers:
        ret[f"layer_{layer}"] = {}
        for key in keys:
            ret[f"layer_{layer}"][key] = [probe_hidden_output[0][layer][key]]
    for hidden_output in probe_hidden_output[1:]:
         for layer in layers:
              for key in keys:
                   ret[f"layer_{layer}"][key].append(hidden_output[layer][key])
    for layer in layers:
         for key in keys:
              ret[f"layer_{layer}"][key] = torch.cat(ret[f"layer_{layer}"][key], dim=1)[0].numpy() # batch size is always 0
    return ret     
```
To see an example of this, see the [Llama model](src/transformers/models/llama/modeling_llama.py) (Search for the pattern 'HIDDEN PROBE CODE')

## Online demos

You can test most of our models directly on their pages from the [model hub](https://huggingface.co/models). We also offer [private model hosting, versioning, & an inference API](https://huggingface.co/pricing) for public and private models.

Here are a few examples:

In Natural Language Processing:
- [Masked word completion with BERT](https://huggingface.co/google-bert/bert-base-uncased?text=Paris+is+the+%5BMASK%5D+of+France)
- [Named Entity Recognition with Electra](https://huggingface.co/dbmdz/electra-large-discriminator-finetuned-conll03-english?text=My+name+is+Sarah+and+I+live+in+London+city)
- [Text generation with Mistral](https://huggingface.co/mistralai/Mistral-7B-Instruct-v0.2)
- [Natural Language Inference with RoBERTa](https://huggingface.co/FacebookAI/roberta-large-mnli?text=The+dog+was+lost.+Nobody+lost+any+animal)
- [Summarization with BART](https://huggingface.co/facebook/bart-large-cnn?text=The+tower+is+324+metres+%281%2C063+ft%29+tall%2C+about+the+same+height+as+an+81-storey+building%2C+and+the+tallest+structure+in+Paris.+Its+base+is+square%2C+measuring+125+metres+%28410+ft%29+on+each+side.+During+its+construction%2C+the+Eiffel+Tower+surpassed+the+Washington+Monument+to+become+the+tallest+man-made+structure+in+the+world%2C+a+title+it+held+for+41+years+until+the+Chrysler+Building+in+New+York+City+was+finished+in+1930.+It+was+the+first+structure+to+reach+a+height+of+300+metres.+Due+to+the+addition+of+a+broadcasting+aerial+at+the+top+of+the+tower+in+1957%2C+it+is+now+taller+than+the+Chrysler+Building+by+5.2+metres+%2817+ft%29.+Excluding+transmitters%2C+the+Eiffel+Tower+is+the+second+tallest+free-standing+structure+in+France+after+the+Millau+Viaduct)
- [Question answering with DistilBERT](https://huggingface.co/distilbert/distilbert-base-uncased-distilled-squad?text=Which+name+is+also+used+to+describe+the+Amazon+rainforest+in+English%3F&context=The+Amazon+rainforest+%28Portuguese%3A+Floresta+Amaz%C3%B4nica+or+Amaz%C3%B4nia%3B+Spanish%3A+Selva+Amaz%C3%B3nica%2C+Amazon%C3%ADa+or+usually+Amazonia%3B+French%3A+For%C3%AAt+amazonienne%3B+Dutch%3A+Amazoneregenwoud%29%2C+also+known+in+English+as+Amazonia+or+the+Amazon+Jungle%2C+is+a+moist+broadleaf+forest+that+covers+most+of+the+Amazon+basin+of+South+America.+This+basin+encompasses+7%2C000%2C000+square+kilometres+%282%2C700%2C000+sq+mi%29%2C+of+which+5%2C500%2C000+square+kilometres+%282%2C100%2C000+sq+mi%29+are+covered+by+the+rainforest.+This+region+includes+territory+belonging+to+nine+nations.+The+majority+of+the+forest+is+contained+within+Brazil%2C+with+60%25+of+the+rainforest%2C+followed+by+Peru+with+13%25%2C+Colombia+with+10%25%2C+and+with+minor+amounts+in+Venezuela%2C+Ecuador%2C+Bolivia%2C+Guyana%2C+Suriname+and+French+Guiana.+States+or+departments+in+four+nations+contain+%22Amazonas%22+in+their+names.+The+Amazon+represents+over+half+of+the+planet%27s+remaining+rainforests%2C+and+comprises+the+largest+and+most+biodiverse+tract+of+tropical+rainforest+in+the+world%2C+with+an+estimated+390+billion+individual+trees+divided+into+16%2C000+species)
- [Translation with T5](https://huggingface.co/google-t5/t5-base?text=My+name+is+Wolfgang+and+I+live+in+Berlin)

In Computer Vision:
- [Image classification with ViT](https://huggingface.co/google/vit-base-patch16-224)
- [Object Detection with DETR](https://huggingface.co/facebook/detr-resnet-50)
- [Semantic Segmentation with SegFormer](https://huggingface.co/nvidia/segformer-b0-finetuned-ade-512-512)
- [Panoptic Segmentation with Mask2Former](https://huggingface.co/facebook/mask2former-swin-large-coco-panoptic)
- [Depth Estimation with Depth Anything](https://huggingface.co/docs/transformers/main/model_doc/depth_anything)
- [Video Classification with VideoMAE](https://huggingface.co/docs/transformers/model_doc/videomae)
- [Universal Segmentation with OneFormer](https://huggingface.co/shi-labs/oneformer_ade20k_dinat_large)

In Audio:
- [Automatic Speech Recognition with Whisper](https://huggingface.co/openai/whisper-large-v3)
- [Keyword Spotting with Wav2Vec2](https://huggingface.co/superb/wav2vec2-base-superb-ks)
- [Audio Classification with Audio Spectrogram Transformer](https://huggingface.co/MIT/ast-finetuned-audioset-10-10-0.4593)

In Multimodal tasks:
- [Table Question Answering with TAPAS](https://huggingface.co/google/tapas-base-finetuned-wtq)
- [Visual Question Answering with ViLT](https://huggingface.co/dandelin/vilt-b32-finetuned-vqa)
- [Image captioning with LLaVa](https://huggingface.co/llava-hf/llava-1.5-7b-hf)
- [Zero-shot Image Classification with SigLIP](https://huggingface.co/google/siglip-so400m-patch14-384)
- [Document Question Answering with LayoutLM](https://huggingface.co/impira/layoutlm-document-qa)
- [Zero-shot Video Classification with X-CLIP](https://huggingface.co/docs/transformers/model_doc/xclip)
- [Zero-shot Object Detection with OWLv2](https://huggingface.co/docs/transformers/en/model_doc/owlv2)
- [Zero-shot Image Segmentation with CLIPSeg](https://huggingface.co/docs/transformers/model_doc/clipseg)
- [Automatic Mask Generation with SAM](https://huggingface.co/docs/transformers/model_doc/sam)


## 100 projects using Transformers

Transformers is more than a toolkit to use pretrained models: it's a community of projects built around it and the
Hugging Face Hub. We want Transformers to enable developers, researchers, students, professors, engineers, and anyone
else to build their dream projects.

In order to celebrate the 100,000 stars of transformers, we have decided to put the spotlight on the
community, and we have created the [awesome-transformers](./awesome-transformers.md) page which lists 100
incredible projects built in the vicinity of transformers.

If you own or use a project that you believe should be part of the list, please open a PR to add it!

## Serious about AI in your organisation? Build faster with the Hugging Face Enterprise Hub.

<a target="_blank" href="https://huggingface.co/enterprise">
    <img alt="Hugging Face Enterprise Hub" src="https://github.com/user-attachments/assets/247fb16d-d251-4583-96c4-d3d76dda4925">
</a><br>

## Quick tour

To immediately use a model on a given input (text, image, audio, ...), we provide the `pipeline` API. Pipelines group together a pretrained model with the preprocessing that was used during that model's training. Here is how to quickly use a pipeline to classify positive versus negative texts:

```python
>>> from transformers import pipeline

# Allocate a pipeline for sentiment-analysis
>>> classifier = pipeline('sentiment-analysis')
>>> classifier('We are very happy to introduce pipeline to the transformers repository.')
[{'label': 'POSITIVE', 'score': 0.9996980428695679}]
```

The second line of code downloads and caches the pretrained model used by the pipeline, while the third evaluates it on the given text. Here, the answer is "positive" with a confidence of 99.97%.

Many tasks have a pre-trained `pipeline` ready to go, in NLP but also in computer vision and speech. For example, we can easily extract detected objects in an image:

``` python
>>> import requests
>>> from PIL import Image
>>> from transformers import pipeline

# Download an image with cute cats
>>> url = "https://huggingface.co/datasets/huggingface/documentation-images/resolve/main/coco_sample.png"
>>> image_data = requests.get(url, stream=True).raw
>>> image = Image.open(image_data)

# Allocate a pipeline for object detection
>>> object_detector = pipeline('object-detection')
>>> object_detector(image)
[{'score': 0.9982201457023621,
  'label': 'remote',
  'box': {'xmin': 40, 'ymin': 70, 'xmax': 175, 'ymax': 117}},
 {'score': 0.9960021376609802,
  'label': 'remote',
  'box': {'xmin': 333, 'ymin': 72, 'xmax': 368, 'ymax': 187}},
 {'score': 0.9954745173454285,
  'label': 'couch',
  'box': {'xmin': 0, 'ymin': 1, 'xmax': 639, 'ymax': 473}},
 {'score': 0.9988006353378296,
  'label': 'cat',
  'box': {'xmin': 13, 'ymin': 52, 'xmax': 314, 'ymax': 470}},
 {'score': 0.9986783862113953,
  'label': 'cat',
  'box': {'xmin': 345, 'ymin': 23, 'xmax': 640, 'ymax': 368}}]
```

Here, we get a list of objects detected in the image, with a box surrounding the object and a confidence score. Here is the original image on the left, with the predictions displayed on the right:

<h3 align="center">
    <a><img src="https://huggingface.co/datasets/huggingface/documentation-images/resolve/main/coco_sample.png" width="400"></a>
    <a><img src="https://huggingface.co/datasets/huggingface/documentation-images/resolve/main/coco_sample_post_processed.png" width="400"></a>
</h3>

You can learn more about the tasks supported by the `pipeline` API in [this tutorial](https://huggingface.co/docs/transformers/task_summary).

In addition to `pipeline`, to download and use any of the pretrained models on your given task, all it takes is three lines of code. Here is the PyTorch version:
```python
>>> from transformers import AutoTokenizer, AutoModel

>>> tokenizer = AutoTokenizer.from_pretrained("google-bert/bert-base-uncased")
>>> model = AutoModel.from_pretrained("google-bert/bert-base-uncased")

>>> inputs = tokenizer("Hello world!", return_tensors="pt")
>>> outputs = model(**inputs)
```

And here is the equivalent code for TensorFlow:
```python
>>> from transformers import AutoTokenizer, TFAutoModel

>>> tokenizer = AutoTokenizer.from_pretrained("google-bert/bert-base-uncased")
>>> model = TFAutoModel.from_pretrained("google-bert/bert-base-uncased")

>>> inputs = tokenizer("Hello world!", return_tensors="tf")
>>> outputs = model(**inputs)
```

The tokenizer is responsible for all the preprocessing the pretrained model expects and can be called directly on a single string (as in the above examples) or a list. It will output a dictionary that you can use in downstream code or simply directly pass to your model using the ** argument unpacking operator.

The model itself is a regular [Pytorch `nn.Module`](https://pytorch.org/docs/stable/nn.html#torch.nn.Module) or a [TensorFlow `tf.keras.Model`](https://www.tensorflow.org/api_docs/python/tf/keras/Model) (depending on your backend) which you can use as usual. [This tutorial](https://huggingface.co/docs/transformers/training) explains how to integrate such a model into a classic PyTorch or TensorFlow training loop, or how to use our `Trainer` API to quickly fine-tune on a new dataset.

## Why should I use transformers?

1. Easy-to-use state-of-the-art models:
    - High performance on natural language understanding & generation, computer vision, and audio tasks.
    - Low barrier to entry for educators and practitioners.
    - Few user-facing abstractions with just three classes to learn.
    - A unified API for using all our pretrained models.

1. Lower compute costs, smaller carbon footprint:
    - Researchers can share trained models instead of always retraining.
    - Practitioners can reduce compute time and production costs.
    - Dozens of architectures with over 400,000 pretrained models across all modalities.

1. Choose the right framework for every part of a model's lifetime:
    - Train state-of-the-art models in 3 lines of code.
    - Move a single model between TF2.0/PyTorch/JAX frameworks at will.
    - Seamlessly pick the right framework for training, evaluation, and production.

1. Easily customize a model or an example to your needs:
    - We provide examples for each architecture to reproduce the results published by its original authors.
    - Model internals are exposed as consistently as possible.
    - Model files can be used independently of the library for quick experiments.

## Why shouldn't I use transformers?

- This library is not a modular toolbox of building blocks for neural nets. The code in the model files is not refactored with additional abstractions on purpose, so that researchers can quickly iterate on each of the models without diving into additional abstractions/files.
- The training API is not intended to work on any model but is optimized to work with the models provided by the library. For generic machine learning loops, you should use another library (possibly, [Accelerate](https://huggingface.co/docs/accelerate)).
- While we strive to present as many use cases as possible, the scripts in our [examples folder](https://github.com/huggingface/transformers/tree/main/examples) are just that: examples. It is expected that they won't work out-of-the-box on your specific problem and that you will be required to change a few lines of code to adapt them to your needs.

## Installation

### With pip

This repository is tested on Python 3.9+, Flax 0.4.1+, PyTorch 1.11+, and TensorFlow 2.6+.

You should install 🤗 Transformers in a [virtual environment](https://docs.python.org/3/library/venv.html). If you're unfamiliar with Python virtual environments, check out the [user guide](https://packaging.python.org/guides/installing-using-pip-and-virtual-environments/).

First, create a virtual environment with the version of Python you're going to use and activate it.

Then, you will need to install at least one of Flax, PyTorch, or TensorFlow.
Please refer to [TensorFlow installation page](https://www.tensorflow.org/install/), [PyTorch installation page](https://pytorch.org/get-started/locally/#start-locally) and/or [Flax](https://github.com/google/flax#quick-install) and [Jax](https://github.com/google/jax#installation) installation pages regarding the specific installation command for your platform.

When one of those backends has been installed, 🤗 Transformers can be installed using pip as follows:

```bash
pip install transformers
```

If you'd like to play with the examples or need the bleeding edge of the code and can't wait for a new release, you must [install the library from source](https://huggingface.co/docs/transformers/installation#installing-from-source).

### With conda

🤗 Transformers can be installed using conda as follows:

```shell script
conda install conda-forge::transformers
```

> **_NOTE:_** Installing `transformers` from the `huggingface` channel is deprecated.

Follow the installation pages of Flax, PyTorch or TensorFlow to see how to install them with conda.

> **_NOTE:_**  On Windows, you may be prompted to activate Developer Mode in order to benefit from caching. If this is not an option for you, please let us know in [this issue](https://github.com/huggingface/huggingface_hub/issues/1062).

## Model architectures

**[All the model checkpoints](https://huggingface.co/models)** provided by 🤗 Transformers are seamlessly integrated from the huggingface.co [model hub](https://huggingface.co/models), where they are uploaded directly by [users](https://huggingface.co/users) and [organizations](https://huggingface.co/organizations).

Current number of checkpoints: ![](https://img.shields.io/endpoint?url=https://huggingface.co/api/shields/models&color=brightgreen)

🤗 Transformers currently provides the following architectures: see [here](https://huggingface.co/docs/transformers/model_summary) for a high-level summary of each them.

To check if each model has an implementation in Flax, PyTorch or TensorFlow, or has an associated tokenizer backed by the 🤗 Tokenizers library, refer to [this table](https://huggingface.co/docs/transformers/index#supported-frameworks).

These implementations have been tested on several datasets (see the example scripts) and should match the performance of the original implementations. You can find more details on performance in the Examples section of the [documentation](https://github.com/huggingface/transformers/tree/main/examples).


## Learn more

| Section | Description |
|-|-|
| [Documentation](https://huggingface.co/docs/transformers/) | Full API documentation and tutorials |
| [Task summary](https://huggingface.co/docs/transformers/task_summary) | Tasks supported by 🤗 Transformers |
| [Preprocessing tutorial](https://huggingface.co/docs/transformers/preprocessing) | Using the `Tokenizer` class to prepare data for the models |
| [Training and fine-tuning](https://huggingface.co/docs/transformers/training) | Using the models provided by 🤗 Transformers in a PyTorch/TensorFlow training loop and the `Trainer` API |
| [Quick tour: Fine-tuning/usage scripts](https://github.com/huggingface/transformers/tree/main/examples) | Example scripts for fine-tuning models on a wide range of tasks |
| [Model sharing and uploading](https://huggingface.co/docs/transformers/model_sharing) | Upload and share your fine-tuned models with the community |

## Citation

We now have a [paper](https://www.aclweb.org/anthology/2020.emnlp-demos.6/) you can cite for the 🤗 Transformers library:
```bibtex
@inproceedings{wolf-etal-2020-transformers,
    title = "Transformers: State-of-the-Art Natural Language Processing",
    author = "Thomas Wolf and Lysandre Debut and Victor Sanh and Julien Chaumond and Clement Delangue and Anthony Moi and Pierric Cistac and Tim Rault and Rémi Louf and Morgan Funtowicz and Joe Davison and Sam Shleifer and Patrick von Platen and Clara Ma and Yacine Jernite and Julien Plu and Canwen Xu and Teven Le Scao and Sylvain Gugger and Mariama Drame and Quentin Lhoest and Alexander M. Rush",
    booktitle = "Proceedings of the 2020 Conference on Empirical Methods in Natural Language Processing: System Demonstrations",
    month = oct,
    year = "2020",
    address = "Online",
    publisher = "Association for Computational Linguistics",
    url = "https://www.aclweb.org/anthology/2020.emnlp-demos.6",
    pages = "38--45"
}
```
