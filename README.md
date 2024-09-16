# Introduction

This is the artifact for the paper "Divide-and-Conquer: Generating UI Code from Screenshots". This artifact supplies the DCGen toolkit and supplementary materials for the paper. 



In this paper, to explore automatic design-to-code solutions, we first conduct a motivating study on GPT-4o and identify three types of issues in generating UI code: element omission, element distortion, and element misarrangement. We further reveal that a focus on smaller visual segments can help multimodal large language models (MLLMs) mitigate these failures in the generation process. In this paper, we propose DCGen, a divide-and-conquer-based approach to automate the translation of webpage design to UI code. DCGen starts by dividing screenshots into manageable segments, generating descriptions for each segment, and then reassembling them into complete UI code for the entire screenshot. We conduct extensive testing with a dataset comprised of real-world websites and various MLLMs and demonstrate that DCGen achieves up to a 14\% improvement in visual similarity over competing methods. Human evaluations show that DCGen can help developers implement webpages significantly faster and more similar to the UI designs. To the best of our knowledge, DCGen is the first segment-aware MLLM-based approach for generating UI code directly from screenshots.



**This repository contains:**

1. **Code implementation of DCGen**, i.e., the Python script and instructions to run DCGen to preprocess websites, segment images, and generate UI code from screenshot with DCGen algorithm.
2. **Sample dataset**. The sample of our experiment data (original and generated) is available in `/data`. We will release the full dataset as soon as the paper is published.
3. **A user-friendly tool based on DCGen**. 



# Code implementation

## 0. Setup

```sh
pip install -r requirements.txt
```

```python
from utils import *
import single_file
```

## 1. Save & Process Website

```python
single_file("https://zh.singlelogin.re/?ts=1205", "./test.html")
simplify_html("test.html", "test_simplified.html", pbar=True)
driver = get_driver(file="./test.html")
take_screenshot(driver, "test.png")
```

## 2. Image Segmentation

```python
img_seg = ImgSegmentation("0.png", max_depth=1)
seg.display_tree()
```

## 3. DCGen

```python
# Example prompt
prompt_dict = {
    "promt_leaf": 'Here is a screenshot of a webpage with a red rectangular bounding box. Focus on the bounding box area. Respond with the content of the HTML+CSS code.',

    "promt_node": 'Here are 1) a screenshot of a webpage with a red rectangular bounding box , and 2) code of different elements in the bounding box. Utilize the provided code to write a new HTML and CSS file to replicate the website in the bounding box. Here is the code of different parts of the webpage in the bounding box:\n=============\n'
}

bot = GPT4(key_path="./path/to/key.txt", model="gpt-4o-mini")
img_seg = ImgSegmentation("0.png", max_depth=1)
dc_trace = DCGenTrace.from_img_seg(img_seg, bot, prompt_leaf=prompt_dict["promt_leaf"], prompt_node=prompt_dict["promt_node"])
dc_trace.generate_code(recursive=True, cut_out=False)
dc_trace.display_tree()
```

## 4. Calculate Score (linux only)

1. Modify configurations in `./metrics/Design2Code/metrics/multi_processing_eval.py`: 

   ```python
   orig_reference_dir = "path/to/original_data_dir"
   test_dirs = {
           "exp_name": "path/to/exp_data_dir"
       }
   ```

   The `original_data_dir` contains original HTML files `1.html, 2.html, ...`, their corresponding screenshots `1.png, 2.png, ...`, and optionally a placeholder image `placeholder.png`.

   The `exp_data_dir` contains the generated HTML files with the same name as the original ones `1.html, 2.html, ...`.

2. Run the evaluation script

	```shell
	cd metrics/Design2Code
	python metrics/multi_processing_eval.py

