# iDetect-AI-powered-Retinal-Disease-Detection-from-Fundus-Images

## Project Metadata
### Authors
- **Team:** Shahbaaz Ahmed Sadiq, Fahad Alothman
- **Supervisor Name:** Dr. Muzammil Behzad
- **Affiliations:** SABIC, ARAMCO and KFUPM 

## Introduction
Retinal diseases such as diabetic retinopathy, glaucoma, and age-related macular degeneration are among the leading causes of vision loss worldwide. Detecting these conditions early through color fundus imaging is critical, since timely treatment can often prevent permanent damage. However, manual diagnosis is not only time-consuming but also heavily dependent on the availability and expertise of ophthalmologists, which creates challenges in many parts of the world. Advances in artificial intelligence, particularly in deep learning, have opened new possibilities for automating this process and making retinal disease screening more accurate and accessible. Building on recent research in vision–language models, this project, titled iDetect: AI powered Retinal Disease Detection from Fundus Images, focuses on developing an intelligent system that can classify retinal diseases using the Peacein color fundus eye dataset. By combining state-of-the-art pretrained models with fine-tuning for specific conditions, the project aims to create a tool that could support doctors in making faster and more reliable diagnoses, ultimately helping patients receive timely care.

## Problem Statement
Retinal diseases such as diabetic retinopathy and glaucoma are major causes of vision loss. Diagnosing them from fundus images requires expert review, which is slow and subjective. The goal is to build an AI system that uses fundus images and textual information to detect multiple retinal diseases automatically, improving accuracy and consistency even with limited labeled data.

## Application Area and Project Domain
Our project lies at the intersection of medical image analysis (ophthalmic imaging) and AI/vision + clinical decision support systems. The application area is ophthalmology / retinal disease screening.
The domain is diagnostic support: We aim to assist or augment clinicians in screening retinal fundus images, flagging high-risk cases or referring those needing further examination.

## What is the paper trying to do, and what are you planning to do?
### What the RET-CLIP Paper Does

The RET-CLIP paper presents a foundation-style vision-language model tailored for retinal imaging. It uses a large-scale dataset of color fundus photographs paired with clinical diagnostic reports (text) to pretrain a CLIP-style embedding: mapping images and corresponding textual diagnostic descriptions into a shared embedding space. They adopt a tripartite optimization scheme at the left-eye, right-eye, and patient level to reflect clinical relationships. After pretraining, RET-CLIP is fine-tuned or adapted for downstream tasks across multiple retinal disease classification benchmarks (diabetic retinopathy, glaucoma, multi-label disease diagnosis) and achieves state-of-the-art performance across eight datasets. The strength lies in learning strong, generalizable retinal image features via vision-text co-training, so that fewer labels are needed for downstream tasks.

### What You Intend to Do

We implement the RET-CLIP approach on the ODIR-5K binocular fundus dataset (left + right eye per patient). Instead of the paper’s original Chinese text tower, we use English biomedical encoders (PubMedBERT/BioBERT) and generate plain-English, side-aware prompts from the ODIR keywords (e.g., “left eye: signs of diabetic changes,” “right eye: normal”). We build a clean, fast pipeline (resize to 224×224, normalize, pack to LMDB with split artifacts) and train/evaluate in two modes: (1) zero-shot, where the model predicts by matching images to prompts, and (2) a linear probe, where a small logistic-regression head is trained on frozen image features to measure feature separability. We report Accuracy, Macro-F1, and Weighted-F1, analyze confusion matrices, and (optionally) compare against a vision-only baseline (e.g., ViT-B/16) to quantify the benefit of adding text prompts and English biomedical encoders.

### Project Documents
- **Presentation:** [Project Presentation](/iDETECT.pptx)
- **Report:** [Project Report](/iDetect_Report.pdf)

### Reference Paper
- [RET-CLIP: A Retinal Image Foundation Model Pre-trained with Clinical Diagnostic Reports](https://arxiv.org/abs/2405.14137)

### Reference Dataset
- [Ocular Disease Intelligent Recognition (ODIR-5K)](https://www.kaggle.com/datasets/andrewmvd/ocular-disease-recognition-odir5k)
- [Peacein/color fundus eye (Hugging Face)](https://huggingface.co/datasets/Peacein/color-fundus-eye)


## Project Technicalities

### Terminologies
- **Fundus Image:** A color photo of the back of the eye (retina), showing the optic disc, macula, and vessels.
- **Binocular Input (Left/Right):** Using both eyes’ images together; differences between eyes can be diagnostic.
- **Laterality:** Whether an image is from the left or right eye; we keep this in prompts and metadata.
- **Text Encoder (PubMedBERT/BioBERT):** The language model that turns a prompt into a numeric embedding.
- **Vision Encoder (ViT-B/16):** The image model that turns a fundus image into a numeric feature vector.
- **Tokenization:** Splitting a prompt into tokens (sub-words) so the text encoder can process it.
- **Embedding (Image/Text):** The numeric vector produced by encoders; used to compare images with prompts.
- **Contrastive Learning (CLIP-style / InfoNCE):** Training that pulls matching image–text pairs together in the embedding space and pushes non-matching pairs apart.
- **Cosine Similarity:** A score measuring how close an image embedding is to a prompt embedding (used for zero-shot predictions).
- **Zero-Shot Classification:** Predicting a condition by matching an image directly to text prompts—no task-specific training head needed.
- **Linear Probe:** A small classifier (logistic regression) trained on frozen image features to test how linearly separable the features are.
- **LMDB (Data Store):** A fast key-value database we use to store images for quick, reproducible loading.
- **Calibration:** Adjusting scores so confidence better reflects correctness (useful for clinical settings).
  
### Problem Statements
- **Problem 1:** Language mismatch and prompt sensitivity. Using English prompts with a non-English (Chinese) text encoder—and limited coverage of synonyms/abbreviations—makes zero-shot predictions unstable and lowers accuracy.
- **Problem 2:** Limited compute and runtime instability. Running on a CPU-only PC with tight RAM caused very slow training/feature extraction, out-of-memory errors, and occasional runtime disconnections, slowing experimentation and reproducibility.
- **Problem 3:** Dataset and evaluation challenges. ODIR-5K has class imbalance, variable image quality, and occasional left/right (laterality) issues; together with domain shift, these factors depress Macro-F1, increase class confusions, and require better calibration and binocular handling.

### Loopholes or Research Areas
- **Evaluation Metrics:** Lack of robust metrics to effectively assess the quality of generated images.
- **Output Consistency:** Inconsistencies in output quality when scaling the model to higher resolutions.
- **Computational Resources:** Training requires significant GPU compute resources, which may not be readily accessible.
- **Prompt and language robustness:** Zero-shot performance depends on wording and encoder language. We should study English biomedical encoders vs. non-English text towers, build prompt ensembles/templates, and quantify how paraphrases, abbreviations, and negations affect results.

### Problem vs. Ideation: Proposed 3 Ideas to Solve the Problems
1. **Problem 1 — Language mismatch & prompt sensitivity → Idea 1: Language-robust zero-shot**
   - Swap the text tower to English biomedical encoders (PubMedBERT/BioBERT).
   - Use prompt templates/ensembles (full name + synonyms + abbreviations; side-aware wording).
   - Add a fallback: auto-translate prompts to the encoder’s native language if needed and compare.
   - Run a prompt-sensitivity suite (paraphrases, abbreviations, negations) and keep the best-performing prompt set.
   
2. **Problem 2 — Limited compute & runtime instability → Idea 2: Resource-aware training loop**
   - Cache features to LMDB and reuse them; start with linear-probe-first to iterate fast.
   - Use smaller batches, fewer workers, and gradient accumulation (when on GPU) to avoid OOM.
   - Prefer lightweight adapter/LoRA tuning on the text tower instead of full finetuning.
   - Add resume checkpoints and shorter “smoke tests” to avoid losing progress on disconnections.
     
3. **Problem 3 — Dataset/evaluation challenges (imbalance, binocular noise, domain shift) → Idea 3: Binocular consistency + calibration**
   - Enforce inter-eye consistency with simple augmentations: eye-swap, single-eye dropout, and laterality checks; add basic quality filters (blurry/overexposed reject).
   - Use class-aware sampling or threshold tuning to mitigate imbalance; always report Macro-F1 alongside Weighted-F1.
   - Add calibration (temperature scaling) and an abstain option for low-confidence zero-shot predictions.
   - Test domain shift with prompt ensembles and small adapter tweaks; log per-class precision/recall and confusion matrices to pinpoint failures.

### Proposed Solution: Code-Based Implementation
This repository implements a RET-CLIP style retinal pipeline in PyTorch that aligns pairs of fundus photos (left/right) with plain-English clinical prompts and evaluates both zero-shot and linear-probe performance.

- Vision & text towers: ViT-B/16 for images and English biomedical text encoders (PubMedBERT/BioBERT) for prompts.
- Training objective: CLIP-style contrastive (InfoNCE) with binocular pairing (left↔text, right↔text), optimized with AdamW and a cosine LR schedule with warm-up.
- Data pipeline: deterministic 224×224 resize + normalization, side-aware prompt generation from ODIR keywords, and LMDB + JSONL/TSV/CSV artifacts for fast, reproducible I/O.
- Evaluations:
  - Zero-shot: classify by matching image embeddings to prompt embeddings via cosine similarity.
  - Linear probe: train a small logistic regression on frozen image features to assess feature separability.
- Utilities: split builders, feature extractors, confusion-matrix plotting, and prompt-ensemble hooks.

### Key Components
- **`[generate_retclip_prompts.ipynb]`**: Contains the code to generate prompts.
- **`[ODIR_RETCLIP_UNIFIED_PIPELINE.ipynb]`**: Contains the main code.

## Model Workflow
The workflow of the Enhanced Stable Diffusion model is designed to translate textual descriptions into high-quality artistic images through a multi-step diffusion process:

1. **Input:**
   - Pairs of fundus photos (left and right eye) from ODIR-5K, plus short, plain-English descriptions (prompts) derived from the dataset keywords (e.g., “left eye: signs of diabetic changes”, “right eye: normal”).
   - Images are resized to 224×224 and normalized; prompts are lowercased, cleaned, and tokenized.

2. **Feature encoding**
   - Image side: a ViT-B/16 vision encoder turns each eye photo into an image embedding.
   - Text side: an English biomedical text encoder (PubMedBERT/BioBERT) turns each prompt into a text embedding.

3. **Training loop (feature learning)**
   - CLIP-style contrastive learning aligns matching image–text pairs and separates non-matches.
   - Binocular pairing is used: left↔prompt and right↔prompt (optionally encouraging inter-eye consistency).
   - Optimization follows the CLIP recipe (AdamW + cosine learning-rate with warm-up).
     
4. **Zero-shot inference**
   - For each image, compute cosine similarity to all class prompts.
   - Pick the nearest prompt to predict the condition (single prompt or an ensemble of prompt phrasings).

5. **Linear-probe evaluation**
   - Freeze the image encoder and extract image features for train/test splits.
   - Train a multinomial logistic-regression classifier on the training features.
   - Evaluate on the test set to measure how linearly separable the learned features are.

6. **Outputs**
   - Metrics: Top-1 Accuracy, Macro-F1, Weighted-F1.
   - Diagnostic plots: confusion matrix to visualize class confusions.

## Pipeline Overview

1. Setup & Configuration        → Install packages, authenticate APIs
2. Load ODIR-5K Dataset         → Auto-download with kagglehub + metadata
3. Generate Clinical Prompts    → DSPy + OpenRouter (3 prompts/patient)
4. Preprocess for RET-CLIP      → TSV + JSONL with eye_side annotations
5. Build LMDB Database          → Efficient PyTorch DataLoader format
6. Train RET-CLIP               → 10 epochs contrastive learning
7. Zero-Shot Evaluation         → Vision-language alignment test
8. Linear Probing Evaluation    → Feature quality assessment
9. Final Report                 → Metrics, comparison, artifacts

## Prerequisites

1. **Google Colab** with A100 GPU (or T4 for testing)
2. **API Keys** (add to Colab Secrets - 🔑 icon in left sidebar):
   - `HF_TOKEN`: HuggingFace token from https://huggingface.co/settings/tokens
   - `OPENROUTER_API_KEY`: OpenRouter key from https://openrouter.ai/keys
   - `KAGGLE_USERNAME`: Your Kaggle username
   - `KAGGLE_KEY`: Kaggle API key from https://www.kaggle.com/settings/account
3. **ODIR-5K Dataset**: Will be downloaded automatically via kagglehub

 ## ⏱️ Estimated Runtime

| Mode | Patients | Prompts Time | Training Time | Total |
|------|----------|--------------|---------------|-------|
| TEST | 100 | ~30 min | ~30 min (2 epochs) | **~2-3 hours** |
| FULL | 5,000 | ~4-5 hours | ~12-15 hours (10 epochs) | **~18-24 hours** |
   
## Acknowledgments
- **Open-Source Communities:** We’re grateful to the communities behind PyTorch, Hugging Face Transformers, scikit-learn, NumPy, Pandas, Pillow, Matplotlib, and LMDB—the RET-CLIP + ODIR pipeline in this repo depends on their work.
- **Datasets & references:** Thanks to the ODIR-5K contributors and hosting on Kaggle for providing accessible binocular fundus images and keywords that made our experiments possible, and to the RET-CLIP authors for inspiring the vision-language approach we adapted.
- **Mentorship & teamwork:** Sincere thanks to our supervisor, Dr. Muzammil Behzad, for guidance and feedback throughout, and to our team—Shahbaaz Ahmed Sadiq and Fahad Alothman—for the sustained effort on data prep, modeling, and evaluation.
- **Resource Providers:** Gratitude to Google Colab for providing the computational resources necessary for this project.
