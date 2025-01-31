---
layout: page
title: cmd2cwl
description: LLM model for coverting tool help document to CWL script.
#img: assets/img/7.jpg
#redirect: https://unsplash.com
url: https://huggingface.co/hubentu/cmd2cwl_Llama-3.2-3B
importance: 3
category: work
---

# [`cmd2cwl`](https://huggingface.co/hubentu/cmd2cwl_Llama-3.2-3B) Model Information

The [`cmd2cwl`](https://huggingface.co/hubentu/cmd2cwl_Llama-3.2-3B) model is an instruction fine-tuned version of the `unsloth/Llama-3.2-3B`. This model has been trained on a custom dataset consisting of help documentation from various command-line tools and corresponding CWL (Common Workflow Language) scripts. Its purpose is to assist users in converting command-line tool documentation into clean and well-structured CWL scripts, enhancing automation and workflow reproducibility.

# Example
## Task
``` python
question = """
Below is an instruction that describes a task, paired with an input that provides further context. Write a response that appropriately completes the request.

### Instruction:
Write a cwl script for md5sum with docker image alpine.

### Input:

    With no FILE, or when FILE is -, read standard input.

    -b, --binary         read in binary mode
    -c, --check          read MD5 sums from the FILEs and check them
    --tag            create a BSD-style checksum
    -t, --text           read in text mode (default)
    -z, --zero           end each output line with NUL, not newline,
    and disable file name escaping

    The following five options are useful only when verifying checksums:
    --ignore-missing  don't fail or report status for missing files
    --quiet          don't print OK for each successfully verified file
    --status         don't output anything, status code shows success
    --strict         exit non-zero for improperly formatted checksum lines
    -w, --warn           warn about improperly formatted checksum lines

    --help     display this help and exit
    --version  output version information and exit

    The sums are computed as described in RFC 1321.  When checking, the input
    should be a former output of this program.  The default mode is to print a
    line with checksum, a space, a character indicating input mode ('*' for binary,
    ' ' for text or where binary is insignificant), and name for each FILE.


### Response:
"""
```

## Using unsloth

``` python
from unsloth import FastLanguageModel
from transformers import TextStreamer

model, tokenizer = FastLanguageModel.from_pretrained(
    model_name = "hubentu/cmd2cwl_Llama-3.2-3B",
    load_in_4bit = False,
)
FastLanguageModel.for_inference(model)

inputs = tokenizer(
    [question],
    return_tensors = "pt").to("cuda")

text_streamer = TextStreamer(tokenizer)
_ = model.generate(**inputs, streamer = text_streamer)

```

## Using AutoModelForCausalLM
``` python
from transformers import AutoTokenizer, AutoModelForCausalLM
from transformers import TextStreamer

model = AutoModelForCausalLM.from_pretrained("hubentu/cmd2cwl_Llama-3.2-3B")
tokenizer = AutoTokenizer.from_pretrained("hubentu/cmd2cwl_Llama-3.2-3B")
model.to('cuda')

text_streamer = TextStreamer(tokenizer)
_ = model.generate(**inputs, streamer = text_streamer, max_length=8192)
```

## Using generator
``` python
from transformers import pipeline
generator = pipeline('text-generation', model="checkpoints/cmd2cwl_Llama-3.2-3B", device='cuda')
resp = generator(question, max_length=8192)
print(resp[0]['generated_text'].split("### Response:\n")[-1])
```

## Output
```
cwlVersion: v1.0
class: CommandLineTool
baseCommand:
- md5sum
requirements:
- class: DockerRequirement
  dockerPull: alpine:latest
label: md5sum
doc: Compute and check MD5 checksums
inputs:
  files:
    label: files
    doc: Input files
    type: File[]
    inputBinding:
      separate: true
outputs:
  md5:
    label: md5
    doc: MD5 checksums
    type: string[]
    outputBinding:
      glob: $(inputs.files.name)
```


# Uploaded  model

- **Developed by:** hubentu
- **License:** apache-2.0
- **Finetuned from model :** unsloth/Llama-3.2-3B
- **Model Home**: <https://huggingface.co/hubentu/cmd2cwl_Llama-3.2-3B>
