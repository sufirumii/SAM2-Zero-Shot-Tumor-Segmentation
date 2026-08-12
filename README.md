SAM2-Zero-Shot-Tumor-Segmentation
Zero-shot brain and breast tumor segmentation in MRI scans using Meta's SAM 2, paired with an automatic prompt generation engine driven by MRI signal physics.

Overview
This project applies Meta's Segment Anything Model 2 to medical imaging without any fine-tuning, labeled data, or manual annotation. An automatic prompt generation engine derives bounding box and point prompts directly from MRI signal characteristics, enabling SAM 2 to segment tumor regions across brain and breast MRI scans out of the box.

The system was evaluated on brain MRI and breast imaging and handles multiple tumor regions per scan simultaneously, working across different tumor types and anatomical locations without modification.

This is a research and educational project. All outputs require radiologist review before any clinical use.

Key Features
Fully automatic, no manual clicks or annotation required
Handles multiple tumor regions per scan in a single pass
Works across brain and breast MRI without modification
Runs on Kaggle free-tier GPU (T4 16GB, well within VRAM limits)
Clean clinical dashboard UI built with Gradio
Region-level report with pixel area and percentage coverage per detected region
Demo
Brain Tumor Segmentation
Input MRI (T1-contrast)	Segmentation Overlay
Brain MRI Input	Brain Segmentation Output
Raw axial brain MRI slice fed into the pipeline	SAM 2 zero-shot segmentation mask overlaid on the input
Breast Tumor Segmentation
Input MRI	Segmentation Overlay
Breast MRI Input	Breast Segmentation Output
Raw breast MRI slice fed into the pipeline	SAM 2 zero-shot segmentation mask overlaid on the input
Architecture
The system is composed of two decoupled stages: a Prompt Generation Engine that converts MRI physics into geometric prompts, and the SAM 2 Inference Core that consumes those prompts to produce segmentation masks.

flowchart LR    subgraph PE["Prompt Generation Engine"]        direction TB        A["MRI Input"] --> B["CLAHE Enhancement"]        B --> C["Percentile Thresholding"]        C --> D["Morphological Cleanup"]        D --> E["Artifact Stripping"]        E --> F["BBox + Point Prompts"]    end    subgraph S2["SAM 2 Core"]        direction TB        G["Hiera Image Encoder"]        H["Streaming Memory Attention"]        I["Mask Decoder"]    end    A --> G    F --> I    G --> H    H --> I    I --> J["Multi-Mask Candidates"]    J --> K["Highest IoU Selection"]    K --> L["Segmentation Overlay"]    K --> M["Region-Level Report"]
High-Level System Flow
flowchart TD    IN["MRI Slice<br/>(JPG / PNG / TIFF)"] --> ENG["Prompt Generation Engine"]    ENG -->|BBox + Center Point| SAM["SAM 2 Large<br/>224M params"]    SAM --> POST["Mask Selection<br/>(IoU ranking)"]    POST --> OUT1["Visualization Overlay"]    POST --> OUT2["Region Report<br/>(area, % coverage)"]
How It Works
Standard SAM 2 deployments require a human to click on the object of interest to generate prompts. This system replaces that manual step with an automated pipeline built around MRI signal physics.

Tumors appear hyper-intense on T1-contrast and FLAIR sequences. The prompt engine exploits this property by processing the raw image through a series of deterministic steps to identify and localize those regions automatically, then feeding the resulting coordinates to SAM 2 as native bounding box and point prompts. No human input is required at any stage.

Stage 1 — Prompt Generation Engine
Step	Operation	Purpose
1	CLAHE Contrast Enhancement	Normalize local contrast so hyper-intense regions become separable
2	Top Intensity Percentile Thresholding	Isolate the brightest signal regions, which correspond to suspicious tissue on contrast-enhanced sequences
3	Morphological Noise Removal	Remove spurious pixel clusters using opening/closing operations
4	Skull and Border Artifact Stripping	Discard high-intensity borders and skull rim artifacts that could be misclassified
5	Bounding Box + Center Point Extraction	Convert remaining connected components into SAM 2 native prompt format
Stage 2 — SAM 2 Inference
Step	Operation	Purpose
6	Hiera Image Encoder	Produces dense image embeddings from the MRI slice
7	Prompt Encoding	Bounding boxes and points encoded into the prompt token space
8	Mask Decoder	Cross-attends image embeddings with prompt tokens to generate candidate masks
9	Multi-Mask Candidate Generation	SAM 2 outputs multiple plausible masks per prompt
10	Highest IoU Mask Selection	Pick the mask with the best predicted IoU score for each region
Stage 3 — Output
Step	Operation	Purpose
11	Segmentation Overlay	Composite mask on the original MRI for visualization
12	Region-Level Report	Compute pixel area and percentage coverage per detected region
Pipeline (Compact View)
MRI Input   |CLAHE Contrast Enhancement   |Top Intensity Percentile Thresholding   |Morphological Noise Removal   |Skull and Border Artifact Stripping   |Bounding Box + Center Point Extraction   |SAM 2 Large (224M parameters)   |Multi-Mask Candidate Generation   |Highest IoU Mask Selection   |Segmentation Overlay + Region Report
Model
Property	Detail
Model	SAM 2 Large
Source	facebook/sam2-hiera-large
Parameters	224M
Encoder	Hiera Hierarchical Vision Transformer
Decoder	Streaming Memory Attention + Mask Decoder
Fine-tuning	None
Labeled data used	None
Installation
pip install git+https://github.com/facebookresearch/sam2.git \            transformers gradio torch torchvision \            pillow numpy opencv-python matplotlib
Usage
python brain_tumor_sam2.py
The Gradio interface will launch and generate a public shareable link via share=True. Upload any brain or breast MRI scan in JPG, PNG, or TIFF format and click Run Segmentation.

Supported Inputs
T1-contrast enhanced MRI (recommended)
T2-weighted MRI
FLAIR sequences
Axial slice screenshots from DICOM viewers
JPG, PNG, TIFF formats
Hardware Requirements
Resource	Minimum	Recommended
GPU VRAM	4 GB	8 GB+
RAM	8 GB	16 GB
Tested on	Kaggle T4 16GB	Kaggle 2x T4
Limitations
Performance depends on tumor contrast relative to surrounding tissue. Low-contrast lesions on non-enhanced sequences may not segment reliably.
The intensity thresholding logic assumes tumor regions are among the brightest structures in the image. This assumption holds well for T1-contrast and FLAIR but may vary on other sequences.
This is not a validated medical device and has not been evaluated on a clinical benchmark dataset.
Clinical Disclaimer
This tool is intended for research and educational purposes only. It is not a certified medical device. All segmentation outputs must be reviewed and verified by a qualified radiologist before informing any clinical decision.

Citation
If you use this project in your work, please cite the original SAM 2 paper:

@article{ravi2024sam2,  title={SAM 2: Segment Anything in Images and Videos},  author={Ravi, Nikhila and Gabeur, Valentin and Hu, Yuan-Ting and others},  journal={arXiv preprint arXiv:2408.00714},  year={2024}}
Author
Rumi Iqbal Sufi
AI Engineer, Healthcare
GitHub: https://github.com/sufirumii
LinkedIn: https://www.linkedin.com/in/rumi-sufi-6323a5265/

License
Apache License 2.0
