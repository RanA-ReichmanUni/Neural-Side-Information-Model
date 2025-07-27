# Image Reconstruction of Visual Diffusion Artifacts with Side Information

# A Friendly High Level Recap Of My Work

## Motivation

In fields like **satellite imaging**, and **medical imaging**, images are often degraded due to hardware limits, bandwidth constraints, or aggressive compression. These distortions can obscure critical visual information, a blurred satellite photo may hide key infrastructure, and a noisy X-ray may conceal critical diagnostic features.

However, many real-world systems in these domains benefit from **multi-origin data capture**, for example, satellites often include both high-resolution and low-resolution sensors, or multiple modalities (e.g., optical + infrared), while medical systems may combine scans from different machines (like CT and X-ray) or store both compressed previews and full-resolution data. Even when these individual inputs are degraded, they offer **complementary structural or semantic cues** that can guide restoration.

Traditional restoration methods, including recent **diffusion models**, perform well on natural image synthesis but often **hallucinate or miss structure** during reconstruction. 

Our goal is to **restore realistic and semantically accurate images**, even when key parts are lost or noisy.

---

## Dual Restoration Modes

We introduce a reconstruction framework that works in **two practical modes**:

- **Mode 1 – Reconstruction from Noisy Sampling:**  
  We simulate natural image degradation (like noisy satellite sensors or single-pixel cameras) and enhance the result with side information.

- **Mode 2 – Restoration from Compressed Sources:**  
  We simulate lossy compression using additive Gaussian noise (based on [Kipnis & Reeves, 2021]) and use side information to refine heavily compressed outputs.

In both modes, a pretrained **Diffusion Model** performs an initial denoising. Our **neural network then refines the output**, using additional structural or semantic guidance from side information.

---

## What is Side Information?

Side information refers to **compact visual cues** extracted from the original image (or its noisy/compressed form) that help guide the reconstruction process.

We tested several types:

- **Low-Resolution Images** – small thumbnails (e.g. 24×24 or 64×64) that carry strong semantic content  
- **Edge Maps** – contour information showing object structure  
- **Segmentation Maps** – high-level semantic regions (e.g. "sky", "car", "tree")  
- **Sparse Grayscale Images** – grayscale images sampled with a binary mask, leaving structured holes. Only the valid pixels are stored, which makes this format highly compact while preserving structural cues  
- **Single-Pixel Imaging Reconstructions (Compressed Sensing)** – images reconstructed from global intensity measurements using a small number of Hadamard patterns. Instead of pixels, these projections encode image content in a compressed format, simulating hardware like single-pixel cameras


These forms of guidance are **lightweight** and practical to transmit or store, yet offer significant benefits to restoration quality.

---

## Model Flow

We combine the side information with the diffusion model’s output through a refinement network:

<!-- Replace this with your actual image -->
![Model Flow Diagram](images/flow.png)

1. A noisy or compressed image is denoised using a Diffusion Model.
2. The output is then refined by our model using side information.
3. The result is a perceptually enhanced reconstruction.

---

## Network Architecture

Our refinement network builds on a **U-Net** backbone and incorporates:

- Dual input branches (baseline + side info)  
- Fusion mechanisms tailored to the side info type  
- GAN-based training (adversarial + perceptual + pixel loss)

This setup enables the model to align structural cues from side information with plausible content from diffusion.

---

## Key Results

- **All side information types outperformed diffusion based denoising baseline**, especially at high degradation levels.
- Even **very low-resolution images** significantly boosted PSNR and SSIM.
- **Edge maps** improved structural coherence (object boundaries, spatial layout).
- **Compressed sensing reconstructions** excelled under extreme degradation, restoring fine details missed by diffusion.

We also introduced a **bitrate savings metric**, quantifying how much less data is needed to achieve the same image quality using side information.

---

## Conclusion

Our approach:
- Bridges **generative denoising** with **guided reconstruction**
- Leverages cheap, available side data for significant quality improvements
- Supports both **compression-aware** and **sensor-noise-aware** use cases

This flexible framework can improve visual quality in real-world systems where data is **incomplete, compressed, or noisy**, offering practical impact for bandwidth-constrained or low-quality sensing environments.

---

## Visual Demo: Alan Turing Teddy Bear Reconstruction

This section walks through the reconstruction process on a sample image, showing the effect of each step and how side information boosts results. 

**Higher PSNR/SSIM indicates a higher similarity to the original image, in a logarithmic scale.**



### 1. Original Image
<!-- Replace with: images/original.png -->
![Original Image](images/original.png)  
*This is the reference image before any compression or degradation.*

---

### 2. Compressed / Noisy Image
<!-- Replace with: images/noisy.png -->
![Noisy Image](images/noisy.png)  
*The image after simulated compression or noise addition. Structural and semantic content is visibly degraded.*

---

### 3. After Baseline Diffusion Denoising
<!-- Replace with: images/denoised.png -->
![Denoised Image](images/denoised.png)  
*Denoised by a pretrained diffusion model. The image appears plausible but lacks sharp structure and fine details. May also hallucinate elements.*

---

### 4. Side Information Used – Low Resolution (64×64)
<!-- Replace with: images/sideinfo_lowres.png -->
![Side Info – Low Res](images/sideinfo_lowres.png)  
*A small thumbnail (64×64) of the original image saved separately as auxiliary input.*

---

### 5. Simulation of Extremly Noisy Intial Image (0.12 bpp), Reconstructed Image Using Denoised + Low Res Side Information
<!-- Replace with: images/reconstructed_lowres.png -->
![Reconstruction with Low Res](images/reconstructed_lowres.png)  
*Final reconstruction using both the denoised image and the low-res side info.*  
**PSNR:** 26.8 dB  
> Compared to:  
> - **Denoised image alone:** 19.5 dB  
> - **Low-res input alone:** also distorted  
> The model fuses both to produce a sharper, semantically accurate image.

---


### 6. Simulation Using Extremly Noisy Intial Image (0.12 bpp), Reconstructed Image Using Denoised + Sparse Grayscale Side Information
<!-- Replace with: images/reconstructed_sparse.png -->
![Reconstruction with Sparse Info](images/reconstructed_sparse.png)  
*Reconstruction using sparse grayscale side info. Preserves fine structure that diffusion missed on its own.*
**PSNR:** 25.02 dB  
> Compared to:  
> - **Denoised image alone:** 19.18 dB  
> - **Sparse Grayscale input alone:** Lacking many pixels, grayscale   
> The model fuses both to produce a much more semantically accurate image. 

---
