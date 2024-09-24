This is the artifact for the paper "Divide-and-Conquer: Generating UI Code from Screenshots". This artifact supplies the DCGen toolkit and supplementary materials for the paper. 


This repository contains:

1. **Code implementation of DCGen**, i.e., the Python script and instructions to run DCGen to preprocess websites, segment images, and generate UI code from screenshots with DCGen algorithm. 
2. **Sample dataset**. The sample of our experiment data (both original and generated) is available in `/data`. We will release the full dataset as soon as the paper is published.
3. **Link to supplementary materials.** We provide all the screen recordings in the usefulness study and our prompt details via this [link](https://drive.google.com/drive/folders/1FnR6MTKCSWFsUP__qO-J5YRhSB7RRDI-?usp=sharing).
4. **A user-friendly tool based on DCGen**.


Quick links: [Demo video](#Demo-video) | [DCGen Examples](#Examples) | [Code usage](#Code-usage) | [Tool usage](#DCGen-tool) 


# Abstract

To explore automatic design-to-code solutions, we begin with a motivating study on GPT-4o, identifying three key issues in UI code generation: element omission, distortion, and misarrangement. We find that focusing on smaller visual segments helps multimodal large language models (MLLMs) mitigate these failures. In this paper, we introduce DCGen, a divide-and-conquer approach that automates the translation of webpage designs into UI code. DCGen divides screenshots into manageable segments, generates descriptions for each, and reassembles them into a complete UI code for the full design. Extensive testing on real-world websites and various MLLMs demonstrates that DCGen improves visual similarity by up to 14% compared to competing methods. Human evaluations confirm that DCGen enables developers to implement webpages faster and with greater fidelity to the original designs. To our knowledge, DCGen is the first segment-aware, MLLM-based solution for generating UI code directly from screenshots.



# Demo video

This video demonstrates how developers can use DCGen to create a webpage from a UI design through *simple copy and paste*. DCGen enables users to review and regenerate code for specific image segments, easily replacing any erroneous code with the correct version for the final webpage.

https://github.com/user-attachments/assets/93e70ae9-c119-4838-94c8-6628be5af7d5

# Examples

Here are two examples from the usefulness study. DCGen demonstrates its effectiveness by significantly reducing element omissions and distortions, leading to faster development and improved webpage quality.

<img src="./assets/case_usefulness.png" alt="case_usefulness" style="zoom: 30%;" />


# Code usage

## 0. Setup

```she
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



# DCGen tool

1. Start the server

```shell
cd Tool
python app.py
```

2. Visit http://127.0.0.1:5000 via local browser

3. Usage:

   **Generate image for entire screenshot**
   <img src="./assets/dcgenui1.png" alt="dcgenui1" style="zoom:20%;" />

   **View the code of any image segment**
   <img src="./assets/dcgenui2.png" alt="dcgenui2" style="zoom:20%;" />

   **Generate code for a image segment**
   <img src="./assets/dcgenui3.png" alt="dcgenui3" style="zoom:20%;" />

   
