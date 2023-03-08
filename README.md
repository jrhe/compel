# Compel
A text prompt weighting and blending library for transformers-type text embedding systems, by [@damian0815](https://github.com/damian0815).

With a flexible and intuitive syntax, you can re-weight different parts of a prompt string and thus re-weight the different parts of the embeddning tensor produced from the string.

Tested and developed against Hugging Face's `StableDiffusionPipeline` but it should work with any diffusers-based system that uses an `Tokenizer` and a `Text Encoder` of some kind.  

Adapted from the [InvokeAI](https://github.com/invoke-ai) prompting code (also by [@damian0815](https://github.com/damian0815)). For now, the syntax is fully documented [here](https://invoke-ai.github.io/InvokeAI/features/PROMPTS/#prompt-syntax-features) - note however that cross-attention control `.swap()` is currently ignored by Compel.

### Installation

`pip install compel`

### Demo

see [compel-demo.ipynb](compel-demo.ipynb)

<a target="_blank" href="https://colab.research.google.com/github/damian0815/compel/blob/main/compel-demo.ipynb">
  <img src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open In Colab"/>
</a>

### Quickstart

with Hugging Face diffusers >=0.12:

```python
from diffusers import StableDiffusionPipeline
from compel import Compel

pipeline = StableDiffusionPipeline.from_pretrained("runwayml/stable-diffusion-v1-5")
compel = Compel(tokenizer=pipeline.tokenizer, text_encoder=pipeline.text_encoder)

# upweight "ball"
prompt = "a cat playing with a ball++ in the forest"
conditioning = compel.build_conditioning_tensor(prompt)

# generate image
images = pipeline(prompt_embeds=conditioning, num_inference_steps=20).images
images[0].save("image.jpg")
```

For batched input, use `torch.cat` to merge multiple conditioning tensors into one:

```python
import torch

from diffusers import StableDiffusionPipeline
from compel import Compel

pipeline = StableDiffusionPipeline.from_pretrained("runwayml/stable-diffusion-v1-5")
compel = Compel(tokenizer=pipeline.tokenizer, text_encoder=pipeline.text_encoder)

prompts = ["a cat playing with a ball++ in the forest", "a dog playing with a ball in the forest"]
prompt_embeds = torch.cat([compel.build_conditioning_tensor(prompt) for prompt in prompts])
images = pipeline(prompt_embeds=prompt_embeds).images

images[0].save("image0.jpg")
images[1].save("image1.jpg")
```

## Changelog
 

### 0.1.9 - add support for prompts longer than the model's max token length. 

To enable, initialize `Compel` with `truncate_long_prompts=False` (default is True). Prompts that are longer than the model's `max_token_length` will be chunked and padded out to an integer multiple of `max_token_length`. 

If you're working with a negative prompt, you will probably need to use `compel.pad_conditioning_tensors_to_same_length()` to avoid having the model complain about mismatched conditioning tensor lengths:

```python
compel = Compel(..., truncate_long_prompts=False)
prompt = "a cat playing with a ball++ in the forest, amazing, exquisite, stunning, masterpiece, skilled, powerful, incredible, amazing, trending on gregstation, greg, greggy, greggs greggson, greggy mcgregface, ..." # very long prompt
negative_prompt = "dog, football, rainforest" # short prompt
conditioning = compel.build_conditioning_tensor(prompt)
negative_conditioning = compel.build_conditioning_tensor(negative_prompt)
[conditioning, negative_conditioning] = compel.pad_conditioning_tensors_to_same_length([conditioning, negative_conditioning])
```

### 0.1.8 - downgrade Python min version to 3.7

### 0.1.7 - InvokeAI compatibility

