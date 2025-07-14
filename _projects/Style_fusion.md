---
layout: page
title: Seamless Dual-Style Image Generation with Diffusion Models
description: Project for Multimodal Generative AI Course
img: assets/img/dual_style_fusion/bear_better.png
importance: 3
category: Graphics & Computer Vision
giscus_comments: false
repository: adriengoldszal/dual-style-image-gen
---

<style>
/* Dark mode compatible table styling with higher specificity */
table td[style*="background-color: yellow"] {
    background-color: rgba(255, 193, 7, 0.2) !important; /* Light yellow for light mode */
}

table tr[style*="background-color: #f0f0f0"] {
    background-color: rgba(108, 117, 125, 0.1) !important; /* Light gray for light mode */
}

table td[style*="border: 1px solid black"], 
table th[style*="border: 1px solid black"] {
    border: 1px solid var(--global-border-color, #dee2e6) !important;
}

/* Dark mode overrides */
html[data-theme="dark"] table td[style*="background-color: yellow"] {
    background-color: rgba(255, 193, 7, 0.3) !important; /* More visible yellow for dark mode */
}

html[data-theme="dark"] table tr[style*="background-color: #f0f0f0"] {
    background-color: rgba(108, 117, 125, 0.2) !important; /* Darker gray for dark mode */
}

html[data-theme="dark"] table td[style*="border: 1px solid black"],
html[data-theme="dark"] table th[style*="border: 1px solid black"] {
    border-color: rgba(255, 255, 255, 0.2) !important;
}

/* Ensure text is visible in both modes */
html[data-theme="dark"] table td,
html[data-theme="dark"] table th {
    color: var(--global-text-color, #fff) !important;
}

table td, table th {
    color: var(--global-text-color, #000) !important;
}
</style>

One little explored frontier of image generation is blending two different styles seamlessly within a single output. In this project, done in collaboration with [Gabriel Mercier](mailto:gabriel.mercier@polytechnique.edu), we present two methods based on latent diffusion models that outperform simple text-based prompting: 
**noise spatial interpolation** and **attention weight interpolation** between two style prompts.

---

<div class="row justify-content-center mb-4">
    <div class="col-auto">
        <a href="https://github.com/adriengoldszal/dual-style-image-gen" class="btn btn-primary btn-sm me-2" target="_blank">
            <i class="fab fa-github"></i> View Repository
        </a>
        <a href="{{ '/assets/pdf/Seamless_Dual_Style_Image_Generation_with_Diffusion_Models.pdf' | relative_url }}" class="btn btn-outline-primary btn-sm" target="_blank">
            <i class="fas fa-file-pdf"></i> Read Paper (PDF)
        </a>
    </div>
</div>

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/dual_style_fusion/image_4.png" title="Original Image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/dual_style_fusion/image_4_eps.png" title="Epsilon Interpolation Result" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/dual_style_fusion/image_4_cross.png" title="Cross-Attention Result" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Original image (left), Epsilon interpolation result (center), and Cross-attention interpolation result (right) showing seamless style fusion between Van Gogh and Minecraft styles.
</div>

## Introduction & Research setting

### State of the art

**Latent diffusion models** have greatly advanced image synthesis quality, especially through iterative denoising processes. These models, such as Stable Diffusion, enable high-quality image generation and the ability to control the output through methods like classifier-free guidance, which allows for fine-tuned and diverse image generation.

Building upon latent diffusion models, numerous methods have been developed to tackle image editing and style transfer tasks. Some approaches, like the deterministic ODE-based settings, leverage the reversible nature of the process to facilitate easier image editing. In contrast, models such as *Prompt-to-Prompt* retain the stochastic properties of diffusion models, utilizing techniques like freezing and modifying the cross-attention layers to perform style transfer and manipulation.

One notable model in this area is **CycleDiffusion**, a state-of-the-art stochastic diffusion model for image editing and style transfer. CycleDiffusion uses the noise trajectory from the reverse diffusion process back to the original image to guide denoising under a new conditioning, preserving the structure of an image while allowing for content modification.

Interpolating between two input images or styles has been explored in previous works, particularly focusing on latent space interpolation. However, smoothly blending styles within a single image is a poorly researched subject and existing methods often face challenges in this task of achieving a seamless fusion of styles while maintaining both visual coherence and distinct stylistic elements within one output.

### Our contribution

**Limitations of Textual Guidance**. When using a single text prompt specifying the spatial arrangement of two distinct styles, results are very poor. The method fails to produce meaningful results and generates artifacts and inconsistent textures.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/dual_style_fusion/2promptmerged.png" title="Text Prompt Failures" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Example outputs specifying a Van Gogh style on the left and a Minecraft style on the right. From left to right: (a) One style dominates (Minecraft), (b) Irregular style separation, (c) Incoherent style fusion.
</div>

**Our aim**. Leveraging CycleDiffusion's style transfer framework, we aim to achieve a **smooth fusion** of styles, ensuring a continuous and visually coherent transition across different regions of the image. We present two methods for this task: one based on noise spatial interpolation and another using cross-attention weight interpolation, both designed to blend the styles seamlessly. We focus on a transition on the horizontal axis between two styles, on the right and the left, but this method can be adapted to different situations and with more than two styles. This approach opens new possibilities for artistic creation, domain adaptation, and mixed-style rendering.

## Proposed methodology

### Noise Merging for Dual-Style Diffusion Guidance

During the standard denoising process of DDIM, the noise term is generated at each step using the trained U-Net neural network architecture. By leveraging the classifier-free guidance formulation, textual guidance can be incorporated into the denoising process as follows:

$$\hat{\epsilon}_\theta(x_t,t,c_i) = \epsilon_\theta(x_t,t,\varnothing) + s\Bigl( \epsilon_\theta(x_t,t,c_i) - \epsilon_\theta(x_t,t,\varnothing) \Bigr)$$

where $$\epsilon_\theta(x_t,t,\varnothing)$$ is the unconditional prediction, $$\epsilon_\theta(x_t,t,c_i)$$ is the text-conditioned prediction, and $$s$$ is the guidance weight.

In our approach, we propose generating two separate noise terms $$\hat{\epsilon}_\theta(x_t,t,c_1)$$ and $$\hat{\epsilon}_\theta(x_t,t,c_2)$$ at each denoising step—one for each style. These are then merged using a spatial mask $$M(x)$$:

$$\hat{\epsilon}_\theta(x_t,t,c_1,c_2) = M(x) \cdot \hat{\epsilon}_\theta(x_t,t,c_1) + (1 - M(x)) \cdot \hat{\epsilon}_\theta(x_t,t,c_2)$$

Then, we integrate this merged noise into the original denoising equation:

$$x_0 = \frac{1}{\sqrt{\bar{\alpha}_t}} \left( x_t - \sqrt{1-\bar{\alpha}_t}\,\hat{\epsilon}_\theta(x_t,t,c_1,c_2) \right)$$

and the update step for $$x_{t-1}$$ becomes:

$$x_{t-1} = \sqrt{\bar{\alpha}_{t-1}}\,x_0 + \sqrt{1-\bar{\alpha}_{t-1}-\sigma_t^2}\,\hat{\epsilon}_\theta(x_t,t,c_1,c_2) + \sigma_t\,z$$

where:

$$\sigma_t = \eta\,\sqrt{\frac{1-\bar{\alpha}_t}{1-\bar{\alpha}_{t-1}}}\,\sqrt{1-\frac{\bar{\alpha}_t}{\bar{\alpha}_{t-1}}}, \quad z \sim \mathcal{N}(0, I)$$

By merging the noise terms before denoising, we ensure that each region of the image receives the correct style influence at every step of the diffusion process. This theoretically allows for a transition between styles while maintaining the structure and coherence of the image.

### Cross-attention weight merging for Dual-Style Diffusion Guidance

Stable-diffusion, on which CycleDiffusion is based, leverages cross-attention between prompt tokens and the image to guide the noise prediction in its U-Net architecture. More specifically, cross-attention helps learn "which parts of the image to modify" by attending more to certain pixels or regions. By changing the cross-attention architecture, we can apply cross-attention on two separate conditionings $$c_1$$ and $$c_2$$ and interpolate using the same spatial mask $$M(x)$$ as follows:

$$\text{Attention}_1(Q, K_1, V_1) = \text{softmax}\left( \frac{Q K_1^T}{\sqrt{d_k}} \right) V_1$$

$$\text{Attention}_2(Q, K_2, V_2) = \text{softmax}\left( \frac{Q K_2^T}{\sqrt{d_k}} \right) V_2$$

Then, these two attention results are merged using the spatial mask $$M(x)$$, which interpolates between them:

$$\text{Merged Output} = M(x) \cdot \text{Attention}_1 + (1 - M(x)) \cdot \text{Attention}_2$$

By doing so, we push certain parts of the image to attend more to prompt tokens of $$c_1$$ and others to prompt tokens of $$c_2$$, helping create a smooth style fusion on the image between the two prompts.

## Experimental Setting

We utilized the default architecture provided by the **CycleDiffusion** codebase, selecting **Stable Diffusion v1.4** from the Hugging Face library as our base model. The experiments were conducted with the following hyperparameter settings:

- $$\eta = 0.1$$
- encoder guidance scale $$s_{\text{encoder}} = 1$$
- decoder guidance scale $$s_{\text{decoder}} = 20$$
- number of denoising steps = 100
- skip steps = 10
- upsampling temperature = 1

For each method, we generated **7 images** using samples from the dataset provided by the CycleDiffusion paper and repository.

### Metrics

To thoroughly test and validate our two style fusion methods, we not only qualitatively analyze the resulting images, but also compare **7 metrics** which measure **pixel, feature and semantic clip-related** elements. Some metrics focus on the image reconstruction quality by comparing the original and generated images, others focus on image quality, and finally, some metrics focus on the styles generated and its distribution over the generated images.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        <table style="width: 100%; border-collapse: collapse; border: 1px solid black;">
            <thead>
                <tr style="background-color: #f0f0f0;">
                    <th colspan="2" style="border: 1px solid black; padding: 8px; text-align: center;"><strong>Similarity-based metrics</strong></th>
                </tr>
                <tr>
                    <th style="border: 1px solid black; padding: 8px; width: 25%;"><strong>Metric</strong></th>
                    <th style="border: 1px solid black; padding: 8px;"><strong>Description and Formula</strong></th>
                </tr>
            </thead>
            <tbody>
                <tr>
                    <td style="border: 1px solid black; padding: 8px;"><strong>L2 Distance</strong></td>
                    <td style="border: 1px solid black; padding: 8px;">Measures the euclidean distance between the generated image and the original image.</td>
                </tr>
                <tr>
                    <td style="border: 1px solid black; padding: 8px;"><strong>LPIPS (Learned Perceptual Image Patch Similarity)</strong></td>
                    <td style="border: 1px solid black; padding: 8px;">Uses deep neural network feature maps (e.g., VGG19) to compute perceptual similarity. Unlike L2, LPIPS captures high-level structural and semantic information.</td>
                </tr>
            </tbody>
        </table>
        <div class="caption">Similarity / Reconstruction Metrics</div>
    </div>
</div>

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        <table style="width: 100%; border-collapse: collapse; border: 1px solid black;">
            <thead>
                <tr style="background-color: #f0f0f0;">
                    <th colspan="2" style="border: 1px solid black; padding: 8px; text-align: center;"><strong>Image quality metrics</strong></th>
                </tr>
                <tr>
                    <th style="border: 1px solid black; padding: 8px; width: 25%;"><strong>Metric</strong></th>
                    <th style="border: 1px solid black; padding: 8px;"><strong>Description and Formula</strong></th>
                </tr>
            </thead>
            <tbody>
                <tr>
                    <td style="border: 1px solid black; padding: 8px;"><strong>SSIM (Structural Similarity Index)</strong></td>
                    <td style="border: 1px solid black; padding: 8px;">Evaluates perceptual similarity by comparing luminance, contrast, and structural details between two images.</td>
                </tr>
                <tr>
                    <td style="border: 1px solid black; padding: 8px;"><strong>PSNR (Peak Signal-to-Noise Ratio)</strong></td>
                    <td style="border: 1px solid black; padding: 8px;">Quantifies how much noise or distortion has been introduced in the generated image compared to the original.</td>
                </tr>
                <tr>
                    <td style="border: 1px solid black; padding: 8px;"><strong>Smoothness (Total Variation)</strong></td>
                    <td style="border: 1px solid black; padding: 8px;">Measures the spatial smoothness of an image by computing the total variation.</td>
                </tr>
            </tbody>
        </table>
        <div class="caption">Image quality Metrics</div>
    </div>
</div>

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        <table style="width: 100%; border-collapse: collapse; border: 1px solid black;">
            <thead>
                <tr style="background-color: #f0f0f0;">
                    <th colspan="2" style="border: 1px solid black; padding: 8px; text-align: center;"><strong>CLIP-Based metrics</strong></th>
                </tr>
                <tr>
                    <th style="border: 1px solid black; padding: 8px; width: 25%;"><strong>Metric</strong></th>
                    <th style="border: 1px solid black; padding: 8px;"><strong>Description and Formula</strong></th>
                </tr>
            </thead>
            <tbody>
                <tr>
                    <td style="border: 1px solid black; padding: 8px;"><strong>CLIP Similarity</strong></td>
                    <td style="border: 1px solid black; padding: 8px;">Measures the semantic alignment between an image and a textual prompt using CLIP embeddings.</td>
                </tr>
                <tr>
                    <td style="border: 1px solid black; padding: 8px;"><strong>Directional CLIP</strong></td>
                    <td style="border: 1px solid black; padding: 8px;">Measures how well the semantic changes between the original and generated images align with an expected direction in the latent space.</td>
                </tr>
            </tbody>
        </table>
        <div class="caption">CLIP-Based metrics</div>
    </div>
</div>

**Remark**: To better measure the style transfer capabilities of our two techniques, a **Gram Matrix based metric** was implemented, comparing the distance between the output of reference images for each style through the convolutional layers of VGG19 and the generated images. For each style, 3 images were used. However, this seemed to not be enough and the results were too similar and did not add to our analysis. It would have been interesting to test with a wider, and more diverse variety of reference images to more accurately judge the style transfer.

### Testing setup

We compare our 2 methods with **two baselines**:

- **Pixel by pixel** merge of two images, which are outputs of the diffusion model for the two prompts separately.
- **One Prompt** only specifying the spatial arrangement as described above.

We thoroughly test these methods in different scenarios:

- **Spatial Mask** We test two different spatial masks representing a Linear and a Logistic (with a sharpness of 20) interpolation.

<div class="row justify-content-center">
    <div class="col-sm-8 col-md-6 mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/dual_style_fusion/image.png" title="Spatial Masks Comparison" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Comparison of linear and logistic spatial masks
</div>

- **Styles studied** Two style pairs are tested: Minecraft / Van Gogh and Mosaic / Andy Warhol Pop Art.

Fixed prompts and parameters are defined for even comparison between methods. The prompts are as follows:

- **MINECRAFT**: A Minecraft-inspired rendering of [prompt], featuring distinct pixelated textures, blocky 3D cube structures, limited color palette with no gradients, sharp right angles and perfect squares, characteristic voxel-based terrain with visible block edges.
- **VAN GOGH**: A Van Gogh-style painting of [prompt], with bold, swirling brushstrokes, rich textures, and vibrant, expressive colors reminiscent of Starry Night
- **MOSAIC**: A highly detailed mosaic of [prompt], made of small, colorful tiles with visible grout lines, creating a textured and handcrafted appearance
- **ANDY WARHOL POP**: A vibrant Andy Warhol-style pop art image of [prompt], featuring bold, contrasting colors, high saturation, thick outlines, and a repeated or silkscreen-like print effect.

## Qualitative Results

In nearly all our examples, qualitative analysis clearly demonstrates that the epsilon interpolation method produces superior results, with both styles distinctly visible. In contrast, the cross-attention interpolation method sometimes causes one of the styles to dominate in the image.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        <table style="width: 100%; border-collapse: collapse; border: 1px solid black;">
            <thead>
                <tr>
                    <th style="border: 1px solid black; padding: 8px; text-align: center;"><strong>Original image</strong></th>
                    <th style="border: 1px solid black; padding: 8px; text-align: center;"><strong>Epsilon</strong></th>
                    <th style="border: 1px solid black; padding: 8px; text-align: center;"><strong>Cross-Attention</strong></th>
                </tr>
            </thead>
            <tbody>
                <tr>
                    <td style="border: 1px solid black; padding: 8px; text-align: center;">
                        {% include figure.liquid loading="eager" path="assets/img/dual_style_fusion/image_4.png" class="img-fluid" style="max-width: 200px;" %}
                    </td>
                    <td style="border: 1px solid black; padding: 8px; text-align: center;">
                        {% include figure.liquid loading="eager" path="assets/img/dual_style_fusion/image_4_eps.png" class="img-fluid" style="max-width: 200px;" %}
                    </td>
                    <td style="border: 1px solid black; padding: 8px; text-align: center;">
                        {% include figure.liquid loading="eager" path="assets/img/dual_style_fusion/image_4_cross.png" class="img-fluid" style="max-width: 200px;" %}
                    </td>
                </tr>
                <tr>
                    <td style="border: 1px solid black; padding: 8px; text-align: center;">
                        {% include figure.liquid loading="eager" path="assets/img/dual_style_fusion/image_7.png" class="img-fluid" style="max-width: 200px;" %}
                    </td>
                    <td style="border: 1px solid black; padding: 8px; text-align: center;">
                        {% include figure.liquid loading="eager" path="assets/img/dual_style_fusion/image_7_eps.png" class="img-fluid" style="max-width: 200px;" %}
                    </td>
                    <td style="border: 1px solid black; padding: 8px; text-align: center;">
                        {% include figure.liquid loading="eager" path="assets/img/dual_style_fusion/image_7_cross.png" class="img-fluid" style="max-width: 200px;" %}
                    </td>
                </tr>
                <tr>
                    <td style="border: 1px solid black; padding: 8px; text-align: center;">
                        {% include figure.liquid loading="eager" path="assets/img/dual_style_fusion/image_8.png" class="img-fluid" style="max-width: 200px;" %}
                    </td>
                    <td style="border: 1px solid black; padding: 8px; text-align: center;">
                        {% include figure.liquid loading="eager" path="assets/img/dual_style_fusion/image_8_eps.png" class="img-fluid" style="max-width: 200px;" %}
                    </td>
                    <td style="border: 1px solid black; padding: 8px; text-align: center;">
                        {% include figure.liquid loading="eager" path="assets/img/dual_style_fusion/image_8_cross.png" class="img-fluid" style="max-width: 200px;" %}
                    </td>
                </tr>
                <tr>
                    <td style="border: 1px solid black; padding: 8px; text-align: center;">
                        {% include figure.liquid loading="eager" path="assets/img/dual_style_fusion/An astronaut riding a horse.png" class="img-fluid" style="max-width: 200px;" %}
                    </td>
                    <td style="border: 1px solid black; padding: 8px; text-align: center;">
                        {% include figure.liquid loading="eager" path="assets/img/dual_style_fusion/horse_epsilon.png" class="img-fluid" style="max-width: 200px;" %}
                    </td>
                    <td style="border: 1px solid black; padding: 8px; text-align: center;">
                        {% include figure.liquid loading="eager" path="assets/img/dual_style_fusion/horse_crossattention.png" class="img-fluid" style="max-width: 200px;" %}
                    </td>
                </tr>
                <tr>
                    <td style="border: 1px solid black; padding: 8px; text-align: center;">
                        {% include figure.liquid loading="eager" path="assets/img/dual_style_fusion/house.png" class="img-fluid" style="max-width: 200px;" %}
                    </td>
                    <td style="border: 1px solid black; padding: 8px; text-align: center;">
                        {% include figure.liquid loading="eager" path="assets/img/dual_style_fusion/house_epsilon.png" class="img-fluid" style="max-width: 200px;" %}
                    </td>
                    <td style="border: 1px solid black; padding: 8px; text-align: center;">
                        {% include figure.liquid loading="eager" path="assets/img/dual_style_fusion/facade_crossattention.png" class="img-fluid" style="max-width: 200px;" %}
                    </td>
                </tr>
                <tr>
                    <td style="border: 1px solid black; padding: 8px; text-align: center;">
                        {% include figure.liquid loading="eager" path="assets/img/dual_style_fusion/lighthouse.png" class="img-fluid" style="max-width: 200px;" %}
                    </td>
                    <td style="border: 1px solid black; padding: 8px; text-align: center;">
                        {% include figure.liquid loading="eager" path="assets/img/dual_style_fusion/lighthouse_epsilon.png" class="img-fluid" style="max-width: 200px;" %}
                    </td>
                    <td style="border: 1px solid black; padding: 8px; text-align: center;">
                        {% include figure.liquid loading="eager" path="assets/img/dual_style_fusion/lighthouse_crossattention.png" class="img-fluid" style="max-width: 200px;" %}
                    </td>
                </tr>
            </tbody>
        </table>
        <div class="caption">Comparison of style fusion results using Epsilon Interpolation and Cross-Attention Interpolation.</div>
    </div>
</div>

## Quantitative Results

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        <table style="width: 100%; border-collapse: collapse; border: 1px solid black; font-size: 12px;">
            <thead>
                <tr>
                    <th style="border: 1px solid black; padding: 4px;"><strong>Metric</strong></th>
                    <th colspan="3" style="border: 1px solid black; padding: 4px; text-align: center;"><strong>Linear</strong></th>
                    <th style="border: 1px solid black; padding: 4px;"><strong>2 prompts</strong></th>
                </tr>
                <tr>
                    <th style="border: 1px solid black; padding: 4px;"></th>
                    <th style="border: 1px solid black; padding: 4px;">Cross Attn</th>
                    <th style="border: 1px solid black; padding: 4px;">Epsilon</th>
                    <th style="border: 1px solid black; padding: 4px;">Pixel</th>
                    <th style="border: 1px solid black; padding: 4px;"></th>
                </tr>
            </thead>
            <tbody>
                <tr style="background-color: #f0f0f0;">
                    <td colspan="5" style="border: 1px solid black; padding: 4px; text-align: center;"><strong>Similarity</strong></td>
                </tr>
                <tr>
                    <td style="border: 1px solid black; padding: 4px;">L2</td>
                    <td style="border: 1px solid black; padding: 4px; background-color: yellow;">200±14</td>
                    <td style="border: 1px solid black; padding: 4px;">214±15</td>
                    <td style="border: 1px solid black; padding: 4px; font-weight: bold;">164±11</td>
                    <td style="border: 1px solid black; padding: 4px;">198±12</td>
                </tr>
                <tr>
                    <td style="border: 1px solid black; padding: 4px;">LPIPS</td>
                    <td style="border: 1px solid black; padding: 4px; background-color: yellow;">0,57±0,02</td>
                    <td style="border: 1px solid black; padding: 4px;">0,59±0,03</td>
                    <td style="border: 1px solid black; padding: 4px; font-weight: bold;">0,54±0,02</td>
                    <td style="border: 1px solid black; padding: 4px;">0,58±0,02</td>
                </tr>
                <tr style="background-color: #f0f0f0;">
                    <td colspan="5" style="border: 1px solid black; padding: 4px; text-align: center;"><strong>Image Quality</strong></td>
                </tr>
                <tr>
                    <td style="border: 1px solid black; padding: 4px;">PSNR</td>
                    <td style="border: 1px solid black; padding: 4px; background-color: yellow;">13,2±0,6</td>
                    <td style="border: 1px solid black; padding: 4px;">12,7±0,8</td>
                    <td style="border: 1px solid black; padding: 4px; font-weight: bold;">14,9±0,6</td>
                    <td style="border: 1px solid black; padding: 4px;">13,3±0,6</td>
                </tr>
                <tr>
                    <td style="border: 1px solid black; padding: 4px;">Smooth.</td>
                    <td style="border: 1px solid black; padding: 4px; background-color: yellow;">0,48±0,09</td>
                    <td style="border: 1px solid black; padding: 4px;">1,01±0,29</td>
                    <td style="border: 1px solid black; padding: 4px;">0,33±0,09</td>
                    <td style="border: 1px solid black; padding: 4px; font-weight: bold;">0,26±0,12</td>
                </tr>
                <tr>
                    <td style="border: 1px solid black; padding: 4px;">SSIM</td>
                    <td style="border: 1px solid black; padding: 4px;">0,48±0,04</td>
                    <td style="border: 1px solid black; padding: 4px; background-color: yellow; font-weight: bold;">0,44±0,04</td>
                    <td style="border: 1px solid black; padding: 4px;">0,53±0,04</td>
                    <td style="border: 1px solid black; padding: 4px;">0,50±0,04</td>
                </tr>
                <tr style="background-color: #f0f0f0;">
                    <td colspan="5" style="border: 1px solid black; padding: 4px; text-align: center;"><strong>CLIP</strong></td>
                </tr>
                <tr>
                    <td style="border: 1px solid black; padding: 4px;">CLIP</td>
                    <td style="border: 1px solid black; padding: 4px;">0,32±0,01</td>
                    <td style="border: 1px solid black; padding: 4px; background-color: yellow;">0,34±0,02</td>
                    <td style="border: 1px solid black; padding: 4px;">0,35±0,01</td>
                    <td style="border: 1px solid black; padding: 4px; font-weight: bold;">0,38±0,01</td>
                </tr>
                <tr>
                    <td style="border: 1px solid black; padding: 4px;">CLIP r.</td>
                    <td style="border: 1px solid black; padding: 4px;">0,31±0,01</td>
                    <td style="border: 1px solid black; padding: 4px;">0,31±0,01</td>
                    <td style="border: 1px solid black; padding: 4px; font-weight: bold;">0,34±0,01</td>
                    <td style="border: 1px solid black; padding: 4px;">0,24±0,01</td>
                </tr>
                <tr>
                    <td style="border: 1px solid black; padding: 4px;">CLIP l.</td>
                    <td style="border: 1px solid black; padding: 4px;">0,34±0,01</td>
                    <td style="border: 1px solid black; padding: 4px;">0,34±0,02</td>
                    <td style="border: 1px solid black; padding: 4px;">0,32±0,01</td>
                    <td style="border: 1px solid black; padding: 4px;">0,25±0,01</td>
                </tr>
                <tr>
                    <td style="border: 1px solid black; padding: 4px;">DCLIP</td>
                    <td style="border: 1px solid black; padding: 4px;">0,10±0,01</td>
                    <td style="border: 1px solid black; padding: 4px; background-color: yellow;">0,13±0,02</td>
                    <td style="border: 1px solid black; padding: 4px;">0,14±0,02</td>
                    <td style="border: 1px solid black; padding: 4px; font-weight: bold;">0,19±0,02</td>
                </tr>
                <tr>
                    <td style="border: 1px solid black; padding: 4px;">DCLIP r.</td>
                    <td style="border: 1px solid black; padding: 4px;">0,09±0,02</td>
                    <td style="border: 1px solid black; padding: 4px;">0,09±0,01</td>
                    <td style="border: 1px solid black; padding: 4px; font-weight: bold;">0,12±0,01</td>
                    <td style="border: 1px solid black; padding: 4px;">0,11±0,01</td>
                </tr>
                <tr>
                    <td style="border: 1px solid black; padding: 4px;">DCLIP l.</td>
                    <td style="border: 1px solid black; padding: 4px;">0,11±0,01</td>
                    <td style="border: 1px solid black; padding: 4px; background-color: yellow; font-weight: bold;">0,13±0,01</td>
                    <td style="border: 1px solid black; padding: 4px;">0,13±0,02</td>
                    <td style="border: 1px solid black; padding: 4px;">0,10±0,01</td>
                </tr>
            </tbody>
        </table>
        <div class="caption">Comparison of different style fusion approaches using various metrics for the linear interpolation. Best results in <strong>bold</strong>, our approach highlighted in yellow.</div>
    </div>
</div>

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        <table style="width: 100%; border-collapse: collapse; border: 1px solid black; font-size: 12px;">
            <thead>
                <tr>
                    <th style="border: 1px solid black; padding: 4px;"><strong>Metric</strong></th>
                    <th colspan="3" style="border: 1px solid black; padding: 4px; text-align: center;"><strong>Logistic</strong></th>
                </tr>
                <tr>
                    <th style="border: 1px solid black; padding: 4px;"></th>
                    <th style="border: 1px solid black; padding: 4px;">Cross Attn</th>
                    <th style="border: 1px solid black; padding: 4px;">Epsilon</th>
                    <th style="border: 1px solid black; padding: 4px;">Pixel</th>
                </tr>
            </thead>
            <tbody>
                <tr style="background-color: #f0f0f0;">
                    <td colspan="4" style="border: 1px solid black; padding: 4px; text-align: center;"><strong>Similarity</strong></td>
                </tr>
                <tr>
                    <td style="border: 1px solid black; padding: 4px;">L2</td>
                    <td style="border: 1px solid black; padding: 4px; background-color: yellow;">196±15</td>
                    <td style="border: 1px solid black; padding: 4px;">208±14</td>
                    <td style="border: 1px solid black; padding: 4px; font-weight: bold;">181±11</td>
                </tr>
                <tr>
                    <td style="border: 1px solid black; padding: 4px;">LPIPS</td>
                    <td style="border: 1px solid black; padding: 4px; background-color: yellow;">0,55±0,03</td>
                    <td style="border: 1px solid black; padding: 4px;">0,59±0,03</td>
                    <td style="border: 1px solid black; padding: 4px; font-weight: bold;">0,54±0,02</td>
                </tr>
                <tr style="background-color: #f0f0f0;">
                    <td colspan="4" style="border: 1px solid black; padding: 4px; text-align: center;"><strong>Image Quality</strong></td>
                </tr>
                <tr>
                    <td style="border: 1px solid black; padding: 4px;">PSNR</td>
                    <td style="border: 1px solid black; padding: 4px; background-color: yellow;">13,6±0,8</td>
                    <td style="border: 1px solid black; padding: 4px;">12,9±0,8</td>
                    <td style="border: 1px solid black; padding: 4px; font-weight: bold;">14,1±0,6</td>
                </tr>
                <tr>
                    <td style="border: 1px solid black; padding: 4px;">Smooth.</td>
                    <td style="border: 1px solid black; padding: 4px; background-color: yellow;">0,41±0,10</td>
                    <td style="border: 1px solid black; padding: 4px;">0,92±0,25</td>
                    <td style="border: 1px solid black; padding: 4px;">0,39±0,10</td>
                </tr>
                <tr>
                    <td style="border: 1px solid black; padding: 4px;">SSIM</td>
                    <td style="border: 1px solid black; padding: 4px;">0,49±0,05</td>
                    <td style="border: 1px solid black; padding: 4px; background-color: yellow;">0,45±0,04</td>
                    <td style="border: 1px solid black; padding: 4px; font-weight: bold;">0,51±0,04</td>
                </tr>
                <tr style="background-color: #f0f0f0;">
                    <td colspan="4" style="border: 1px solid black; padding: 4px; text-align: center;"><strong>CLIP</strong></td>
                </tr>
                <tr>
                    <td style="border: 1px solid black; padding: 4px;">CLIP</td>
                    <td style="border: 1px solid black; padding: 4px;">0,32±0,01</td>
                    <td style="border: 1px solid black; padding: 4px; background-color: yellow;">0,34±0,02</td>
                    <td style="border: 1px solid black; padding: 4px;">0,35±0,01</td>
                </tr>
                <tr>
                    <td style="border: 1px solid black; padding: 4px;">CLIP r.</td>
                    <td style="border: 1px solid black; padding: 4px;">0,31±0,01</td>
                    <td style="border: 1px solid black; padding: 4px;">0,31±0,02</td>
                    <td style="border: 1px solid black; padding: 4px; font-weight: bold;">0,34±0,01</td>
                </tr>
                <tr>
                    <td style="border: 1px solid black; padding: 4px;">CLIP l.</td>
                    <td style="border: 1px solid black; padding: 4px;">0,33±0,01</td>
                    <td style="border: 1px solid black; padding: 4px;">0,34±0,02</td>
                    <td style="border: 1px solid black; padding: 4px;">0,32±0,01</td>
                </tr>
                <tr>
                    <td style="border: 1px solid black; padding: 4px;">DCLIP</td>
                    <td style="border: 1px solid black; padding: 4px;">0,10±0,02</td>
                    <td style="border: 1px solid black; padding: 4px; background-color: yellow;">0,12±0,01</td>
                    <td style="border: 1px solid black; padding: 4px; font-weight: bold;">0,14±0,02</td>
                </tr>
                <tr>
                    <td style="border: 1px solid black; padding: 4px;">DCLIP r.</td>
                    <td style="border: 1px solid black; padding: 4px;">0,09±0,02</td>
                    <td style="border: 1px solid black; padding: 4px;">0,10±0,01</td>
                    <td style="border: 1px solid black; padding: 4px; font-weight: bold;">0,13±0,01</td>
                </tr>
                <tr>
                    <td style="border: 1px solid black; padding: 4px;">DCLIP l.</td>
                    <td style="border: 1px solid black; padding: 4px;">0,11±0,01</td>
                    <td style="border: 1px solid black; padding: 4px; background-color: yellow; font-weight: bold;">0,13±0,01</td>
                    <td style="border: 1px solid black; padding: 4px;">0,12±0,02</td>
                </tr>
            </tbody>
        </table>
        <div class="caption">Comparison of different style fusion approaches using various metrics for logistic interpolation (sharpness 20). Best results in <strong>bold</strong>, our approach highlighted in yellow.</div>
    </div>
</div>

As expected, **pixel interpolation** of images from a state-of-the-art model **outperforms nearly all metrics**.
**Cross-attention** method achieves **superior image reconstruction**, as evidenced by higher similarity and image quality metrics.
**CLIP measurements** highlight the dominance of the **epsilon method** in style fusion showing both styles have better results.

**Remarks on Cross-Attention Interpolation & Results**

This discrepancy could be explained by the fact that cross-attention is only one of the many layers in the U-Net. Other components may **smooth out** the interpolation effect, reducing the visibility of both styles unless additional guidance is applied to reinforce stylistic separation.

In fact, strengthening the guidance with a higher guidance scale, and increasing the sharpness often produce better results with the cross-attention. Further testing is certainly required to analyse this behaviour more in depth.

<div class="row justify-content-center">
    <div class="col-sm-8 col-md-6 mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/dual_style_fusion/bear_better.png" title="Improved Cross-Attention" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Minecraft / Van Gogh Cross-attention interpolation with sharpness 70 & decoder guidance of 30
</div>

In addition, tried doing **multiple passes through the cross-attention** instead of a single one, by passing the resulting attention weights back into the function, with the idea of strengthening the style transfer. However, this seemed to only accentuate the artifacts on the image and smoothen it out more, with a full two passes rendering the final image completely blurry. A weighted sum of multiple pass outputs was done to mitigate this, but the style transfer didn't appear qualitatively better so the single pass method was kept.

## Conclusion & Future Work

In this work, we present two novel methods for dual style fusion on images using diffusion models, allowing for smooth interpolation between two distinct styles on a single image. By leveraging the U-Net architecture, we apply epsilon interpolation, which interpolates noise generated from two prompts, and cross-attention interpolation, which operates on cross-attention weights. Our results demonstrate that these methods outperform traditional text-based prompting, providing more refined and accurate style fusion.

**Further work** should focus on a more in-depth evaluation of these methods, using a larger dataset of images and styles to perform a more comprehensive comparison. The uncertainties in some of our data suggest that additional testing is needed to fully understand the performance across different metrics. It would also be valuable to explore style-transfer quality by revisiting the Gram Matrix technique with a broader variety of images for each style.

Moreover, future research could investigate improved ways to leverage cross-attention layers for style transfer, particularly through multiple passes, to further refine the transfer process and compare the optimal parameters with epsilon interpolation. Finally, exploring direct utilization of better style embeddings, drawing inspiration from recent research, could lead to improved results in style fusion.

## References

1. Ho, J., Jain, A., & Abbeel, P. (2020). Denoising Diffusion Probabilistic Models. *NeurIPS*.

2. Rombach, R., Blattmann, A., Lorenz, D., Esser, P., & Ommer, B. (2022). High-Resolution Image Synthesis with Latent Diffusion Models. *CVPR*.

3. Song, J., Meng, C., & Ermon, S. (2020). Denoising Diffusion Implicit Models. *arXiv preprint arXiv:2010.02502*.

4. Ho, J. & Salimans, T. (2021). Classifier-Free Diffusion Guidance. *arXiv preprint arXiv:2207.12598*.

5. Wu, C. H. & De la Torre, F. (2022). Unifying Diffusion Models' Latent Space, with Applications to CycleDiffusion and Guidance. *arXiv preprint arXiv:2210.05559*.

6. Hertz, A., Mokady, R., Tenenbaum, J., Aberman, K., Pritch, Y., & Cohen-Or, D. (2022). Prompt-to-Prompt Image Editing with Cross Attention Control. *arXiv preprint arXiv:2208.01626*.

7. Wang, C. J. & Golland, P. (2023). Interpolating between Images with Diffusion Models. *arXiv preprint arXiv:2301.06974*.

8. Li, W., Fang, M., Zou, C., Gong, B., Zheng, R., Wang, M., Chen, J., & Yang, M. (2024). StyleTokenizer: Defining Image Style by a Single Instance for Controlling Diffusion Models. *Proceedings of the European Conference on Computer Vision (ECCV)*, 123-140.

{% if page.repository %}
<div class="repository-card">
  <h2>Associated Repository</h2>
  {% include repository/repo.liquid repository=page.repository %}
</div>
{% endif %}
