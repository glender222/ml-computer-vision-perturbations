# T2UE: Generating Unlearnable Examples from Text Descriptions

## Xingjun Ma

Fudan University

Shanghai, China

xingjunma@fudan.edu.cn

## Hanxun Huang

The University of Melbourne

Melbourne, Australia

hanxun@unimelb.edu.au

## Tianwei Song

Fudan University

Shanghai, China

22210240266@m.fudan.edu.cn

## Ye Sun

Fudan University

Shanghai, China

yesun23@m.fudan.edu.cn

## Yifeng Gao

Fudan University

Shanghai, China

yifenggao23@m.fudan.edu.cn

## Yu-Gang Jiang<sup>∗</sup>

Fudan University

Shanghai, China

ygj@fudan.edu.cn

## Abstract

Large-scale pre-training frameworks like CLIP have revolutionized multimodal learning, but their reliance on web-scraped datasets, frequently containing private user data—raises serious concerns about misuse. Unlearnable Examples (UEs) have emerged as a promising countermeasure against unauthorized model training, employing carefully crafted unlearnable noise to disrupt the learning of meaningful representations from protected data. Current approaches typically generate UEs by jointly optimizing unlearnable noise for both images and their associated text descriptions (or labels). However, this optimization process is often computationally prohibitive for on-device execution, forcing reliance on external thirdparty services. This creates a fundamental privacy paradox: users must initially expose their data to these very services to achieve protection, thereby compromising privacy in the process. Such a contradiction has severely hindered the development of practical, scalable data protection solutions. To resolve this paradox, we introduce **Text-to-Unlearnable Example (T2UE)**, a novel framework that enables users to generate UEs using only text descriptions. T2UE circumvents the need for original image data by employing a text-to-image (T2I) model to map text descriptions into the image (noise) space, combined with an errorminimization framework to produce effective unlearnable noise. Extensive experiments show that T2UE-protected data substantially degrades performance in downstream tasks (e.g., cross-modal retrieval) for state-of-the-art models. Notably, the protective effect generalizes across diverse architectures and even to supervised learning settings. Our work demonstrates the feasibility of “zero-contact data protection", where personal data can be safeguarded based solely on their textual descriptions, eliminating the need for direct data exposure.

![](images/image_01.jpg)

<details>
<summary>flowchart</summary>

```mermaid
graph LR
  subgraph Image_Dependent_Image_Dependent_Text_Image_Guidance_UEs["Image-Dependent[\"Image-Dependent (Text+Image Guidance) UEs\"]"]
  User["User"] -->|"A man in a leather jacket"| ThirdParty["Third-party Services"]
  UnlearnableImage["Unlearnable Image"] --> ThirdParty
    User <-->|"Private Image & Text"| ThirdParty
  ThirdParty -->|"A man in a leather jacket"| User
  end

  subgraph Image_Independent_Image_Independent_Text_Guidance_Only_UEs["Image-Independent[\"Image-Independent (Text Guidance Only) UEs\"]"]
    User <-->|"Text Only"| ThirdParty
  ThirdParty -->|"Unlearnable Noise"| User
    User <-->|"Local Assembly"| User
  end
```
</details>

Figure 1: Illustration of our T2UE data protection approach. Existing image-dependent methods require access to both sensitive images and their corresponding text for third-party generation of UE, introducing a risk of data leakage. In contrast, image-independent T2UE shares only textual descriptions with the third party, ensuring zero contact with the sensitive image.

## 1 Introduction

The advent of large-scale multi-modal models, such as CLIP [30] and ALIGN [18], has significantly advanced multi-modal pre-training by demonstrating powerful capabilities in bridging vision and language. These models derive their strength from pre-training on massive, non-curated webscraped datasets, often comprising billions of image-text pairs [35, 36, 9, 45]. A growing concern is that several large-scale datasets have been collected without user consent and subsequently used to train commercial models [1, 14, 27]. The potential for unauthorized exploitation of such data for model training underscores an urgent need for effective data protection methods.

Unlearnable Examples (UEs) have emerged as a promising proactive strategy for data misuse preservation [16, 8, 31, 53, 32, 43, 26, 41, 48, 10, 28]. By introducing meticulously crafted, often imperceptible perturbations into data, UEs aim to disrupt the training process, rendering the data unusable for effective model learning. The underlying mechanism often involves creating misleading features that the model easily learns instead of the true data features [34]. While initially developed for unimodal tasks like image classification [16, 7] and subsequently extended to areas like segmentation [38], 3D point clouds [43], time series [19], and neural code learning [17]. Adapting UEs to the multi-modal domain and protecting against large-scale pre-training presents unique challenges, as it requires disrupting the learned alignment between continuous image features and diverse textual semantics, rather than merely associating perturbations with discrete class labels [16].

The existing approach for multi-modal unlearnable examples (MUEs) exemplified by multi-step error minimization (MEM) [25], proposes jointly optimizing image perturbations alongside specific textual "triggers" appended to the original captions. However, this strategy exhibits significant limitations hindering practical adoption. Firstly, its reliance on discrete textual triggers makes the unlearnable effect fragile and can be easily nullified if the triggers are altered, paraphrased, or removed by common text processing, undermining robustness. Optimizing these triggers is also computationally inefficient. Second, and more critically, MEM as well as most existing UE methods, requires access to the original user image to compute perturbations. This process is often computationally intensive and may depend on hardware or resources not readily available to end users, necessitating the use of third-party services. As illustrated in Figure 1, this creates a fundamental paradox: To safeguard data, users must first expose their potentially sensitive data (e.g. by uploading them to an external service), creating inherent risks of stealing, intercepting, misuse, or leakage during the protection process itself.

In this work, we introduce **Text-to-Unlearnable Example (T2UE)**, a novel paradigm that fully decouples the generation of unlearnable examples (UEs) from source image data. As illustrated in Figure 1, our key innovation lies in learning a mapping from the textual semantic space to the unlearnable perturbation space, enabling the creation of effective protective perturbations guided solely by textual descriptions. This approach eliminates the need for original image access during the protection phase, reducing the risk of data exposure. The perturbations produced by T2UE are semantically tied to the text and are specifically designed to disrupt the contrastive alignment objective central to multi-modal pre-training, regardless of image content. Furthermore, T2UE improves task transferability, a known limitation of prior UE methods. By leveraging a pre-trained CLIP model as a surrogate to simulate the outer optimization in the error-minimization framework [16], we show that T2UE-generated UE generalize beyond contrastive learning to impact supervised tasks. Collectively, these contributions mark a significant step toward practical data misuse protection in large-scale multi-modal pre-training.

Table 1: A summary of existing UE generation methods. Properties fully satisfied (✓) and not satisfied (✗) are indicated for each method.

| Method | Image-Independent | Text-Independent | Transferability | Robustness | Imperceptibility | Efficiency |
| --- | --- | --- | --- | --- | --- | --- |
| UEs [16] | ✗ | ✓ | ✗ | ✗ | ✓ | ✗ |
| RUEs [8] | ✗ | ✓ | ✗ | ✓ | ✗ | ✗ |
| TUEs [31] | ✗ | ✓ | ✓ | ✗ | ✓ | ✗ |
| SynPer [50] | ✓ | ✓ | ✗ | ✗ | ✗ | ✓ |
| MEM [25] | ✗ | ✗ | ✗ | ✗ | ✓ | ✗ |
| T2UE (Ours) | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |

In summary, our main contributions are:

• We propose T2UE, a novel UE generation framework that operates solely on textual input, overcoming the critical image-dependency limitation of existing approaches.
• We introduce a simple yet powerful approach to achieve T2UE. We use a text-guided unlearnable generation model to learn a mapping from the text semantic space to the unlearnable perturbation space, by leveraging CLIP encoders as a surrogate model.
• We conduct comprehensive evaluations demonstrating that T2UE effectively prevents data misuse in contrastive pre-training, exhibits strong protection transferability across diverse multi-modal architectures and tasks, as well robust to different scenarios.

## 2 Related Work

**Unlearnable Examples.** Unlearnable Examples (UEs) represent a proactive privacy-preserving technique that prevents effective learning by machine learning models through the injection of carefully crafted, imperceptible noise into training data [16]. Subsequent research elucidated their mechanism as the creation of easily learnable input-output "shortcuts" that mislead model training [34, 33]. Enhancing practicality has driven efforts to improve UE robustness against defenses like adversarial training [46, 26] and boost transferability across learning paradigms and tasks [16]. Notably, UE applications have expanded beyond classification to finer-grained tasks like image segmentation (e.g., UnSeg [38]) and are emerging for other modalities like 3D point clouds [43]. However, extending UEs to multimodal models like CLIP is challenging; existing methods often jointly optimize image and text perturbations [25], facing issues with text optimization difficulty and inherent privacy risks from requiring original image access. Indeed, as summarized in Table 1, this critical requirement for image access is common among most prior UE techniques and the existing MUE method MEM, starkly contrasting with the image-independent approach of our proposed T2UE. Distinct from prior work, our method generates protective noise solely using textual information. This text-only approach circumvents the limitations of previous MUE strategies, offering a more secure, robust, and practical pathway for multimodal data privacy.

**Conditional GAN.** Conditional GAN (CGAN) was initially proposed by Mirza and Osindero [29] with the core idea of enabling controlled image generation in GANs, rather than random synthesis. Specifically, CGAN incorporates conditional information into both the generator and discriminator inputs. The generated images must not only appear realistic but also align with the given conditions to be accepted by the discriminator. This work has inspired a substantial body of subsequent research. For instance, Radford et al. proposed DCGAN [44], which integrated convolutional structures with CGAN to enhance the visual quality of generated images, while StackGAN [51] achieved high-resolution text-to-image generation through a two-stage CGAN framework. In cross-modal tasks, models like pix2pix further leveraged paired data to learn image-to-image mapping, where conditional information is provided in the form of input images. Subsequently, the application of conditional generation expanded to diverse scenarios such as video prediction (e.g., VGAN [40]) and medical image synthesis (e.g., MedGAN [11]). The recent study SSGAN [22] proposed a semanticspatial aware module to achieve fine-grained text-to-image alignment, significantly enhancing the quality of generated images.

![](images/image_02.jpg)

Figure 2: An overview of our proposed T2UE framework, which leverages text description embedding as guidance (stage 1) to guide the generation of general unlearnable noise (stage 2). The noise generator is trained with a surrogate CLIP model to minimize image-text similarity (stage 3).

## 3 Text-to-Unlearnable Examples

We begin by introducing our threat model, followed by an overview of the overall T2UE framework. The detailed components of T2UE are described in subsections 3.1 and 3.2.

**Threat Model.** We consider a scenario involving two primary roles: the **data protector** (e.g., an individual user or platform) and the **data hacker** (e.g., an entity scraping data for unauthorized model training). The protector aims to proactively safeguard their multi-modal data (image-text pairs) before potential exposure, such as uploading to public platforms or cloud storage. To this end, the protector utilizes a perturbation generation mechanism (potentially leveraging a pre-trained generator model) to transform their original data into Unlearnable Examples (UEs). Conversely, the hacker gathers these publicly available, yet ostensibly protected, UEs to train their own models. We assume the hacker may employ prevalent training paradigms, including CLIP [30] or standard supervised learning for specific downstream tasks. The hacker’s objective is to achieve high model performance despite training on the protected dataset. A critical vulnerability emerges with prior UE generation methods [25] on multi-modal data, which require the protector to access and process the original, sensitive image data to compute the protective perturbation. As highlighted in our Figure 1, this workflow necessitates handling or transmitting the very data intended for protection, creating significant risks such as data theft or leakage during this process. Therefore, our threat model centers on a protector who seeks a more secure approach: the ability to generate effective UE without requiring access to the original image data, ideally leveraging only readily available, less sensitive associated information like textual descriptions. The ultimate goal remains unchanged: the generated UEs must effectively disrupt the hacker’s model training across both anticipated learning paradigms (CLIP and supervised learning).

**Framework Overview.** The T2UE framework, illustrated in Figure 2, is designed to train a generator that maps text semantics to effective unlearnable perturbations. The process involves three key stages. First, **Text Feature Extraction** utilizes a frozen, pre-trained text encoder (e.g., from CLIP text encoder) to obtain a semantic embedding from the input text. Second, **Text-Guided Noise Generation** employs the learnable generator network, which takes the text embedding and a random latent vector to synthesize the unlearnable perturbation.

Third, **CLIP-Based Surrogate Model** guides the optimization of the generator. In this stage, the unlearnable perturbation is added to the original image to produce a protected image (UE), while the corresponding text descriptions remain unchanged. We adopt the standard InfoNCE contrastive loss and use pre-trained encoders as surrogate models due to their strong generalization capabilities.

![](images/image_03.jpg)

<details>
<summary>flowchart</summary>

```mermaid
graph LR
  InputText["Input Text"] --> Encoder["Text Encoder"]
  Encoder --> Feature["Text features"]
  Feature --> SSACN["SSACN"]
  SSACN --> Conv["Conv"]
  Conv --> Tanh["Tanh"]
  FC["FC"] --> Reshape["Reshape 4 × 4 × 512"]
  RandomLatent["Random Latent"] --> FC
  RandomLatent --> Reshape
  RandomLatent --> SSACN
  SSACN --> Conv
  Conv --> Tanh
  UnlearnableNoise["Unlearnable Noise"]
```
</details>

Figure 3: Architecture of T2UE generation model.

## 3.1 T2UE Generator Architecture

To achieve the mapping from text semantics to unlearnable noise, we instantiate the generator $G$ by adapting the generator architecture from the Semantic-Space-Aware GAN (SSA-GAN) [23], which was originally designed for high-fidelity text-to-image synthesis. We selected it over alternatives like large diffusion models primarily for its balance of effectiveness in conditioning generation on text semantics and its comparatively favourable computational efficiency (reduced parameters, memory footprint, and training time), which is crucial for training the generator within our optimization framework. The generator ${ \dot { G } } ,$ parameterized by $\theta _ { G } ,$ , accepts two inputs: the text embedding embt $\in$ $\mathbb { R } ^ { d _ { t } }$ derived from the frozen text encoder and a random latent vector $\boldsymbol { z } \in \mathbb { R } ^ { d _ { z } }$ sampled from a standard normal distribution, $z \sim \mathcal { N } ( 0 , I )$ . The unlearnable noise is generated by G as follows:

$$
\delta^ {u} = G \left(\operatorname{emb} _ {t}, z; \theta_ {G}\right) \quad \text {s.t.} \quad \| \delta^ {u} \| _ {\infty} \leq \epsilon . \tag {1}
$$

The core structure of G comprises a sequence of Semantic-Space-Aware Convolutional Network (SSACN) blocks [23]. These blocks progressively refine and upsample spatial feature representations, beginning with an initial feature map derived from the latent vector $z .$ The key component enabling fine-grained textual control within each SSACN block is Semantic-Space Conditional Batch Normalization (SSCBN). Instead of using standard batch normalization, SSCBN predicts text-specific affine parameters (scale $\gamma$ and shift $\bar { \beta } )$ based on the input text embedding emb<sub>t</sub>. These parameters dynamically modulate the normalization of the intermediate feature maps h:

$$
\mathrm{SSCBN} (h, \mathrm{emb} _ {t}) = \gamma (\mathrm{emb} _ {t}) \odot \left(\frac {h - \mu (h)}{\sqrt {\sigma^ {2} (h) + \epsilon_ {b n}}}\right) + \beta (\mathrm{emb} _ {t}) \tag {2}
$$

where $\mu ( h )$ and $\sigma ^ { 2 } ( h )$ are the mean and variance of h across the batch and spatial dimensions, $\epsilon _ { b n }$ is a small constant for numerical stability, $\gamma ( \cdot ) , \beta ( \cdot )$ are small neural networks $( \mathbf { e . g . }$ , linear layers or MLPs) mapping em $\mathbf { b } _ { t }$ to the channel-wise scale and shift parameters, and ⊙ represents element-wise multiplication. By conditioning the feature statistics on the text embedding at multiple hierarchical levels within the generator, SSCBN allows G to synthesize unlearnable noise patterns $\delta ^ { u }$ whose characteristics are semantically correlated with the input text $T .$ This semantic correlation is vital for establishing the protective feature intended to mislead the CLIP. Through the sequential application of these SSACN blocks, incorporating residual connections and upsampling operations, the generator synthesizes the final noise perturbation $\delta \in \mathbb { R } ^ { H \times W \times C }$ , typically matching the original image resolution.

Table 2: The main results of T2UE’s adversarial impact on the CLIP model (with ResNet50 as the vision backbone) in both text-to-image (T2I) and image-to-text (I2T) retrieval tasks. The evaluation covers three benchmark datasets: Flickr8k validation set, Flickr30k, and Tiny-MSCOCO validation set. T2UE demonstrates a significant reduction in the CLIP model’s test performance across all datasets. The best protection results are highlighted in boldface.

<table><tr><td rowspan="3">Method</td><td rowspan="3">Zero Contact</td><td colspan="4">Flick8k</td><td colspan="4">Flick30k</td><td colspan="4">Tiny-MSCOCO</td></tr><tr><td colspan="2">Image → Text</td><td colspan="2">Text → Image</td><td colspan="2">Image → Text</td><td colspan="2">Text → Image</td><td colspan="2">Image → Text</td><td colspan="2">Text → Image</td></tr><tr><td>Hit@10</td><td>Medr</td><td>Hit@10</td><td>Medr</td><td>Hit@10</td><td>Medr</td><td>Hit@10</td><td>Medr</td><td>Hit@10</td><td>Medr</td><td>Hit@10</td><td>Medr</td></tr><tr><td>Random Guess</td><td>-</td><td>1.2</td><td>727</td><td>0.9</td><td>490</td><td>1.0</td><td>711</td><td>1.0</td><td>498</td><td>0.2</td><td>3419</td><td>0.2</td><td>2491</td></tr><tr><td>No Protection</td><td>-</td><td>17.7</td><td>91</td><td>18.6</td><td>78</td><td>49.3</td><td>11</td><td>44.1</td><td>15</td><td>33.3</td><td>26</td><td>15.5</td><td>247</td></tr><tr><td>EM</td><td>✗</td><td>7.9</td><td>223</td><td>8.9</td><td>154</td><td>5.5</td><td>315</td><td>6.1</td><td>190</td><td>3.7</td><td>402</td><td>3.9</td><td>344</td></tr><tr><td>TAP</td><td>✗</td><td>14.3</td><td>120</td><td>12.8</td><td>92</td><td>39.7</td><td>19</td><td>37.2</td><td>23</td><td>7.4</td><td>194</td><td>6.0</td><td>289</td></tr><tr><td>MEM-3</td><td>✗</td><td>15.9</td><td>92</td><td>17.5</td><td>78</td><td>47.6</td><td>12</td><td>43.5</td><td>16</td><td>28.5</td><td>31</td><td>15.5</td><td>236</td></tr><tr><td>MEM-5</td><td>✗</td><td>17.6</td><td>92</td><td>17.7</td><td>77</td><td>47.1</td><td>13</td><td>43.3</td><td>16</td><td>29.2</td><td>30</td><td>15.4</td><td>246</td></tr><tr><td>T2UE (Ours)</td><td>✓</td><td>11.6(6.1↓)</td><td>155(68↓)</td><td>8.02(10.56↓)</td><td>194(116↓)</td><td>18.2(31.1↓)</td><td>78(67↓)</td><td>15.62(28.48↓)</td><td>106(91↓)</td><td>7.4(25.9↓)</td><td>220(194↓)</td><td>5.14(10.38↓)</td><td>331(84↓)</td></tr></table>

Table 3: Performance comparison of state-of-the-art methods in supervised learning. Experiments are conducted on CIFAR-10 and CIFAR-100 using ResNet-18 as the target model. T2UE (ViT/B-16) denotes that the generator aligns with CLIP’s ViT-B/16 model, while T2UE (ViT/B-32) indicates alignment with CLIP’s ViT-B/32. T2UE achieves competitive performance with the best baseline method while maintain zero-contact to the original images.

<table><tr><td rowspan="2">Method</td><td rowspan="2">Zero Contact</td><td colspan="2">CIFAR-10</td><td colspan="2">CIFAR-100</td></tr><tr><td>Class-wise</td><td>Sample-wise</td><td>Class-wise</td><td>Sample-wise</td></tr><tr><td>No Protection</td><td>✓</td><td>90.34</td><td>90.34</td><td>65.98</td><td>65.98</td></tr><tr><td>EM</td><td>✗</td><td>11.57</td><td>15.37</td><td>1.07</td><td>3.21</td></tr><tr><td>Emax</td><td>✗</td><td>-</td><td>45.39</td><td>-</td><td>47.69</td></tr><tr><td>TUE</td><td>✗</td><td>-</td><td>10.06</td><td>-</td><td>1.05</td></tr><tr><td>UC</td><td>✗</td><td>79.41</td><td>-</td><td>41.38</td><td>-</td></tr><tr><td>AR</td><td>✓</td><td>10.41</td><td>-</td><td>5.11</td><td>-</td></tr><tr><td>SynPer</td><td>✓</td><td>-</td><td>17.00</td><td>-</td><td>1.89</td></tr><tr><td>TAP</td><td>✗</td><td>-</td><td>10.78</td><td>-</td><td>60.60</td></tr><tr><td>CP</td><td>✗</td><td>10.12</td><td>83.15</td><td>1.31</td><td>49.91</td></tr><tr><td>T2UE(ViT/B-16)</td><td>✓</td><td>10.02(80.32↓)</td><td>13.76(76.58↓)</td><td>1.08(64.81↓)</td><td>1.82(64.16↓)</td></tr><tr><td>T2UE(ViT/B-32)</td><td>✓</td><td>12.17(78.17↓)</td><td>13.89(76.45↓)</td><td>1.42(64.56↓)</td><td>1.85(64.13↓)</td></tr></table>

## 3.2 T2UE Generator Training

The objective of T2UE is to train the text-guided generator G, that it produces image perturbations $\delta ^ { u }$ that render data attacker’s training ineffective. We achieve this by optimizing $\bar { G }$ using a frozen pre-trained CLIP model $( f _ { I } , f _ { T } )$ , as a surrogate to simulate potential data exploitation, as illustrated in Figure 2. Formally, given a training data distribution D of image-text pairs $( I , T )$ , the objective of standard multi-modal contrastive learning is to train image $( f _ { I } )$ and text $( f _ { T } ^ { \dot { } } )$ encoders to align representations of corresponding pairs while distinguishing non-corresponding pairs within a shared embedding space. This alignment is commonly achieved by minimizing a contrastive loss over batches of N pairs, such as the symmetric InfoNCE loss:

$$
\min _ {\theta_ {I}, \theta_ {T}} \mathbb {E} _ {\left\{\left(I _ {i}, T _ {i}\right) \right\} _ {i = 1} ^ {N} \sim \mathcal {D}} \left[ \mathcal {L} _ {\text {InfoNCE}} \left(\left\{f _ {I} \left(I _ {i}; \theta_ {I}\right) \right\} _ {i = 1} ^ {N}, \left\{f _ {T} \left(T _ {i}; \theta_ {T}\right) \right\} _ {i = 1} ^ {N}\right) \right]. \tag {3}
$$

Our T2UEs aims to establish a strong, protective features between the input text $T$ and the generated $\delta ^ { u }$ , thereby disrupting the model’s ability to learn the true relationship between original images I and texts ${ \dot { T } } .$ Thus, the optimization objective is to find the optimal generator parameters $\theta _ { G } ^ { * }$ by minimizing the expected alignment loss computed by the surrogate model on protected data:

$$
\theta_ {G} ^ {*} = \arg \min _ {\theta_ {G}} \mathbb {E} _ {(I, T) \sim \mathcal {D}} \left[ \mathcal {L} _ {\text {InfoNCE}} \left(\left\{f _ {I} \left(I + \delta^ {u}\right) \right\}, \left\{f _ {T} (T) \right\}\right) \right]. \tag {4}
$$

Minimizing this objective (Eq. 4) compels the generator G to synthesize perturbations $\delta ^ { u }$ that, when added to any image $I ,$ cause the resulting image feature $f _ { I } ( I + \delta ^ { \prime } )$ to align strongly with the feature $f _ { T } ( T )$ corresponding to the conditioning text T. Consequently, models subsequently trained on datasets $\{ ( I + \delta ^ { \prime } , T ) \}$ learn this easily predictable feature instead of the meaningful image-text relationships, thus achieving the unlearnable effect.

Table 4: Transferability evaluation of UE across different model Architectures. The noise for all baseline methods is generated using the ResNet-50 model and evaluated on various network architectures (ResNet-18/50[13], VGG-11/13[37], GoogLeNet[39], DenseNet-121[15]) on the CIFAR-10 and CIFAR-100 datasets. T2UE demonstrates superior transferability across different model backbones.

<table><tr><td rowspan="2">Method</td><td colspan="2">CIFAR-10</td><td colspan="2">CIFAR-100</td><td colspan="2">CIFAR-10</td><td colspan="2">CIFAR-100</td></tr><tr><td>Class-wise</td><td>Sample-wise</td><td>Class-wise</td><td>Sample-wise</td><td>Class-wise</td><td>Sample-wise</td><td>Class-wise</td><td>Sample-wise</td></tr><tr><td>Target Model</td><td colspan="4">Resnet18</td><td colspan="4">Resnet50</td></tr><tr><td>No Protection</td><td>90.34</td><td>90.34</td><td>65.98</td><td>65.98</td><td>85.56</td><td>85.56</td><td>47.44</td><td>47.44</td></tr><tr><td>EM</td><td>11.57</td><td>15.37</td><td>1.07</td><td>3.21</td><td>12.81</td><td>11.40</td><td>0.88</td><td>3.10</td></tr><tr><td>Emax</td><td>-</td><td>45.39</td><td>-</td><td>47.69</td><td>-</td><td>38.67</td><td>-</td><td>41.76</td></tr><tr><td>TUE</td><td>-</td><td>10.06</td><td>-</td><td>1.05</td><td>-</td><td>10.18</td><td>-</td><td>0.85</td></tr><tr><td>UC</td><td>79.41</td><td>-</td><td>41.38</td><td>-</td><td>79.18</td><td>-</td><td>37.28</td><td></td></tr><tr><td>AR</td><td>10.41</td><td>-</td><td>5.11</td><td>-</td><td>10.92</td><td>-</td><td>2.18</td><td></td></tr><tr><td>SynPer</td><td>-</td><td>17.00</td><td>-</td><td>1.89</td><td>-</td><td>13.06</td><td>-</td><td>1.79</td></tr><tr><td>TAP</td><td>-</td><td>10.78</td><td>-</td><td>60.6</td><td>-</td><td>22.80</td><td>-</td><td>43.55</td></tr><tr><td>CP</td><td>10.12</td><td>83.15</td><td>1.31</td><td>49.91</td><td>10.27</td><td>84.42</td><td>1.05</td><td>42.61</td></tr><tr><td>T2UE(ViT/B-16)</td><td>10.02(80.32↓)</td><td>13.76(76.58↓)</td><td>1.08(64.81↓)</td><td>1.82(64.16↓)</td><td>10.04(75.52↓)</td><td>10.97(75.59↓)</td><td>1.07(46.37↓)</td><td>1.36(46.08↓)</td></tr><tr><td>T2UE(ViT/B-32)</td><td>12.17(78.17↓)</td><td>13.89(76.45↓)</td><td>1.42(64.56↓)</td><td>1.85(64.13↓)</td><td>10.41(75.15↓)</td><td>10.41(75.15↓)</td><td>1.18(46.26↓)</td><td>1.21(46.23↓)</td></tr><tr><td>Target Model</td><td colspan="4">VGG11</td><td colspan="4">VGG13</td></tr><tr><td>No Protection</td><td>84.65</td><td>84.65</td><td>57.78</td><td>57.78</td><td>86.30</td><td>86.30</td><td>61.50</td><td>61.50</td></tr><tr><td>EM</td><td>15.31</td><td>12.90</td><td>1.07</td><td>3.63</td><td>13.40</td><td>12.03</td><td>1.00</td><td>3.60</td></tr><tr><td>Emax</td><td>-</td><td>80.36</td><td>-</td><td>57.39</td><td>-</td><td>80.58</td><td>-</td><td>58.17</td></tr><tr><td>TUE</td><td>-</td><td>13.53</td><td>-</td><td>1.76</td><td>-</td><td>12.06</td><td>-</td><td>1.56</td></tr><tr><td>UC</td><td>82.31</td><td>-</td><td>56.94</td><td>-</td><td>85.43</td><td>-</td><td>56.83</td><td>-</td></tr><tr><td>AR</td><td>29.55</td><td>-</td><td>50.18</td><td>-</td><td>10.32</td><td>-</td><td>28.46</td><td>-</td></tr><tr><td>SynPer</td><td>-</td><td>18.94</td><td>-</td><td>4.19</td><td>-</td><td>15.68</td><td>-</td><td>3.43</td></tr><tr><td>TAP</td><td>-</td><td>50.46</td><td>-</td><td>57.39</td><td>-</td><td>40.55</td><td>-</td><td>43.15</td></tr><tr><td>CP</td><td>11.13</td><td>83.31</td><td>2.45</td><td>57.50</td><td>11.11</td><td>86.29</td><td>1.84</td><td>58.69</td></tr><tr><td>T2UE(ViT/B-16)</td><td>10.96(73.69↓)</td><td>28.43(56.22↓)</td><td>3.58(54.20↓)</td><td>9.55(48.23↓)</td><td>11.93(74.37↓)</td><td>19.73(66.57↓)</td><td>1.21(46.37↓)</td><td>6.32(55.18↓)</td></tr><tr><td>T2UE(ViT/B-32)</td><td>15.65(69.06↓)</td><td>24.58(59.98↓)</td><td>2.89(54.89↓)</td><td>15.08(42.70↓)</td><td>10.44(75.86↓)</td><td>18.42(67.88↓)</td><td>1.24(60.26↓)</td><td>16.13(45.37↓)</td></tr><tr><td>Target Model</td><td colspan="4">GoogLeNet</td><td colspan="4">DenseNet121</td></tr><tr><td>No Protection</td><td>89.89</td><td>89.89</td><td>69.74</td><td>69.74</td><td>87.81</td><td>87.81</td><td>70.29</td><td>70.29</td></tr><tr><td>EM</td><td>11.93</td><td>17.75</td><td>1.00</td><td>3.40</td><td>12.86</td><td>13.86</td><td>1.14</td><td>3.42</td></tr><tr><td>Emax</td><td>-</td><td>25.61</td><td>-</td><td>67.88</td><td>-</td><td>41.85</td><td>-</td><td>64.33</td></tr><tr><td>TUE</td><td>-</td><td>11.94</td><td>-</td><td>1.00</td><td>-</td><td>12.07</td><td>-</td><td>1.05</td></tr><tr><td>UC</td><td>80.29</td><td>-</td><td>40.38</td><td>-</td><td>85.99</td><td>-</td><td>66.52</td><td>-</td></tr><tr><td>AR</td><td>11.76</td><td>-</td><td>4.91</td><td>-</td><td>12.27</td><td>-</td><td>4.68</td><td>-</td></tr><tr><td>SynPer</td><td>-</td><td>17.81</td><td>-</td><td>10.00</td><td>-</td><td>13.42</td><td>-</td><td>2.12</td></tr><tr><td>TAP</td><td>-</td><td>23.01</td><td>-</td><td>65.86</td><td>-</td><td>19.97</td><td>-</td><td>66.66</td></tr><tr><td>CP</td><td>11.67</td><td>55.85</td><td>1.24</td><td>67.99</td><td>10.51</td><td>86.69</td><td>1.69</td><td>67.87</td></tr><tr><td>T2UE(ViT/B-16)</td><td>9.44(80.45↓)</td><td>11.44(78.45↓)</td><td>1.06(68.68↓)</td><td>1.94(67.8↓)</td><td>10.02(77.79↓)</td><td>13.21(74.6↓)</td><td>1.11(69.18↓)</td><td>3.06(67.23↓)</td></tr><tr><td>T2UE(ViT/B-32)</td><td>10.35(79.54↓)</td><td>11.29(78.6↓)</td><td>1.37(68.37↓)</td><td>2.82(66.92↓)</td><td>12.12(75.69↓)</td><td>17.08(70.73↓)</td><td>1.57(68.72↓)</td><td>3.02(67.27↓)</td></tr></table>

Table 5: Mixed Poisoning Robustness Evaluation on CIFAR-10/100 with ResNet-50 (0% Clean to 100% Poisoned).

<table><tr><td>Dataset</td><td colspan="2">CIFAR-10</td><td colspan="2">CIFAR-100</td><td colspan="2">CIFAR-10</td><td colspan="2">CIFAR-100</td></tr><tr><td>T2UE Version</td><td colspan="2">T2UE(ViT/B-16)</td><td colspan="2">T2UE(ViT/B-32)</td><td colspan="2">T2UE(ViT/B-16)</td><td colspan="2">T2UE(ViT/B-32)</td></tr><tr><td>Method</td><td>Class-wise</td><td>Sample-wise</td><td>Class-wise</td><td>Sample-wise</td><td>Class-wise</td><td>Sample-wise</td><td>Class-wise</td><td>Sample-wise</td></tr><tr><td>No Protection</td><td>90.34</td><td>90.34</td><td>65.98</td><td>65.98</td><td>90.34</td><td>90.34</td><td>65.98</td><td>65.98</td></tr><tr><td>20%</td><td>79.46</td><td>82.65</td><td>52.68</td><td>49.90</td><td>81.22</td><td>82.37</td><td>47.3</td><td>46.87</td></tr><tr><td>40%</td><td>78.01</td><td>81.00</td><td>45.33</td><td>46.15</td><td>79.11</td><td>78.64</td><td>43.11</td><td>41.27</td></tr><tr><td>60%</td><td>74.46</td><td>77.88</td><td>34.76</td><td>41.16</td><td>74.24</td><td>74.25</td><td>35.56</td><td>33.63</td></tr><tr><td>80%</td><td>68.44</td><td>70.37</td><td>26.27</td><td>28.82</td><td>64.57</td><td>64.65</td><td>24.22</td><td>26.72</td></tr><tr><td>100%</td><td>10.02</td><td>13.76</td><td>1.08</td><td>1.82</td><td>12.17</td><td>13.89</td><td>1.42</td><td>1.41</td></tr></table>

**Zero-Contact Data Protection.** Once trained, the generator produces unlearnable noise $\delta ^ { u }$ based solely on a text prompt $T$ and a random vector $z ,$ without requiring access to the target image. This noise can then be applied to any image for downstream tasks. For classification, specific noise patterns can be generated as needed—for example, class-wise patterns using fixed templates and z, or sample-wise patterns using varied templates and z—demonstrating the flexibility of the textguided approach.

## 4 Experiment

**Datasets and Target Models.** Our T2UE generator was trained on the MSCOCO dataset [24]. We evaluated its effectiveness in disrupting CLIP-like models (using ResNet50 [13] or ViT-B/32 [6] image backbones) from scratch on protected versions of Flickr8k [49], Flickr30k [49], and Tiny

Table 6: The test accuracy (%) of ResNet50 trained using different defense methods on unlearnable CIFAR-10 and CIFAR-100 crafted by T2UE.

<table><tr><td>Dataset</td><td colspan="4">CIFAR-10</td><td colspan="4">CIFAR-100</td></tr><tr><td>Defense</td><td>None</td><td>CutOut</td><td>MixUp</td><td>AutoAug</td><td>None</td><td>CutOut</td><td>MixUp</td><td>AutoAug</td></tr><tr><td>No Protection</td><td>83.91</td><td>83.14</td><td>86.17</td><td>83.53</td><td>65.98</td><td>56.59</td><td>49.88</td><td>53.78</td></tr><tr><td>T2UE(ViT/B-16)-C</td><td>10.02</td><td>9.99</td><td>11.44</td><td>10.02</td><td>1.08</td><td>1.62</td><td>1.51</td><td>1.03</td></tr><tr><td>T2UE(ViT/B-16)-S</td><td>13.76</td><td>10.97</td><td>10.33</td><td>14.35</td><td>1.82</td><td>1.64</td><td>2.06</td><td>1.81</td></tr><tr><td>T2UE(ViT/B-32)-C</td><td>12.17</td><td>11.21</td><td>9.98</td><td>11.97</td><td>1.42</td><td>1.15</td><td>0.98</td><td>1.57</td></tr><tr><td>T2UE(ViT/B-32)-S</td><td>13.89</td><td>12.30</td><td>10.56</td><td>11.03</td><td>1.41</td><td>1.77</td><td>1.46</td><td>1.64</td></tr></table>

![](images/image_04.jpg)

<details>
<summary>line</summary>

| Epoch | clean | EM | GP | UC | AR | T2UE (V16) | T2UE (V32) |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 0 | ~2.1 | ~2.1 | ~2.1 | ~2.1 | ~2.1 | ~2.1 | ~2.1 |
| 10 | ~0.5 | ~0.1 | ~0.1 | ~0.1 | ~0.1 | ~0.1 | ~0.1 |
| 20 | ~0.2 | ~0.05 | ~0.05 | ~0.05 | ~0.05 | ~0.05 | ~0.05 |
| 40 | ~0.1 | ~0.05 | ~0.05 | ~0.05 | ~0.05 | ~0.05 | ~0.05 |
| 60 | ~0.1 | ~0.05 | ~0.05 | ~0.05 | ~0.05 | ~0.05 | ~0.05 |
| 80 | ~0.1 | ~0.05 | ~0.05 | ~0.05 | ~0.05 | ~0.05 | ~0.05 |
| 100 | ~0.1 | ~0.05 | ~0.05 | ~0.05 | ~0.05 | ~0.05 | ~0.05 |
</details>

(a) Class-wise $\Delta _ { c }$ Loss

![](images/image_05.jpg)

<details>
<summary>line</summary>

| Epoch | clean | EM | CP | UC | AR | T2UE (V16) | T2UE (V32) |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 0 | ~12 | ~12 | ~12 | ~12 | ~12 | ~12 | ~12 |
| 10 | ~70 | ~13 | ~12 | ~12 | ~12 | ~12 | ~12 |
| 20 | ~75 | ~13 | ~12 | ~12 | ~12 | ~12 | ~12 |
| 30 | ~80 | ~13 | ~12 | ~12 | ~12 | ~12 | ~12 |
| 40 | ~80 | ~13 | ~12 | ~12 | ~12 | ~12 | ~12 |
| 50 | ~80 | ~13 | ~12 | ~12 | ~12 | ~12 | ~12 |
| 60 | ~80 | ~13 | ~12 | ~12 | ~12 | ~12 | ~12 |
| 70 | ~80 | ~13 | ~12 | ~12 | ~12 | ~12 | ~12 |
| 80 | ~80 | ~13 | ~12 | ~12 | ~12 | ~12 | ~12 |
| 90 | ~80 | ~13 | ~12 | ~12 | ~12 | ~12 | ~12 |
| 100 | ~80 | ~13 | ~12 | ~12 | ~12 | ~12 | ~12 |
</details>

(b) Class-wise $\triangle _ { c }$ Accuracy

![](images/image_06.jpg)

<details>
<summary>line</summary>

| Epoch | clean | EM | SynPer | TUE | CP | Emax | T2UE (V16) | T2UE (V32) |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 0 | ~2.1 | ~2.1 | ~2.1 | ~2.1 | ~2.1 | ~2.1 | ~2.1 | ~2.1 |
| 10 | ~0.3 | ~0.1 | ~0.1 | ~0.1 | ~0.1 | ~0.1 | ~0.1 | ~0.1 |
| 20 | ~0.1 | ~0.05 | ~0.05 | ~0.05 | ~0.05 | ~0.05 | ~0.05 | ~0.05 |
| 40 | ~0.05 | ~0.02 | ~0.02 | ~0.02 | ~0.02 | ~0.02 | ~0.02 | ~0.02 |
| 60 | ~0.03 | ~0.01 | ~0.01 | ~0.01 | ~0.01 | ~0.01 | ~0.01 | ~0.01 |
| 80 | ~0.02 | ~0.01 | ~0.01 | ~0.01 | ~0.01 | ~0.01 | ~0.01 | ~0.01 |
| 100 | ~0.01 | ~0.01 | ~0.01 | ~0.01 | ~0.01 | ~0.01 | ~0.01 | ~0.01 |
</details>

(c) Sample-wise $\Delta _ { s }$ Loss

![](images/image_07.jpg)

<details>
<summary>line</summary>

| Epoch | clean | EM | SynPer | TUE | DP | tEmix | T2UE (V16) | T2UE (V32) |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 0 | ~25 | ~25 | ~25 | ~25 | ~25 | ~25 | ~25 | ~25 |
| 100 | ~85 | ~15 | ~15 | ~10 | ~15 | ~15 | ~15 | ~15 |
</details>

(d) Sample-wise $\Delta _ { s }$ Accuracy

Figure 4: (a-b): For class-wise noise, learning curves of ResNet50 on CIFAR-10 dataset with different types of training data: 1) Loss curve, 2) Test Accuracy Curve. (c-d): For sample-wise noise, learning curves of ResNet50 on CIFAR-10 dataset.

MSCOCO [2]. Transferability to supervised learning was assessed by training standard classification backbones (ResNet18/50/101 [13], ViT-B/16/32 [6]) on protected CIFAR-10/-100 [21] and STL-10 [3].

**Implementation Details.** The T2UE generator leverages a CLIP ViT-B/32 [30] alignment model and was trained for 500 epochs (ViT-B/16 variant: 300 epochs). For the generator training stage, following the training setup of the generator model in [22], we choose a batch size of 128 and we employ an Adam optimizer with initial learning rate to 0.0001 and decoupled weight decay regularization, while the learning rate decays using a cosine scheduler. We train these models from scratch on 1 A100 GPU for 32 epochs. Generated unlearnable noise $\delta ^ { u }$ were constrained with $| | \delta | | _ { \infty } \leq 8 / 2 5 5$ . Target supervised models were trained for 100 epochs. Following the parameters in [16], we use SGD with a learning rate of 0.1, momentum (0.9), and weight decay $( \bar { 5 } . 0   \times   \bar { 1 } 0 ^ { - 4 } )$ , along with cosine annealing scheduling of the learning rate For fair comparison, target model architectures and training setups were kept consistent across compared methods within each experiment.

**Baselines.** In the CLIP setting, we compare T2UE with three applicable baselines: **EM**[16], **TAP**[7], and **MEM** [25]. As our setup simulates user-side application, we adapt MEM by using only its image noise (MEM-3/MEM-5 denote variants omitting 3/5-token triggers). In the supervised setting, we compare against established UE methods including **EM** [53], **Emax** [20], **AR** [33], **TUE** [31], **UC** [53], **SynPer** [50], **TAP** [7], **CP** [12], using ResNet18 as the surrogate model where required.

**Evaluation Metrics.** For MCL, we measure zero-shot retrieval performance degradation using Hit@10 (Recall@10) and Median Rank (MedR) for I2T and T2I tasks [46, 47]; lower Hit@10 and higher MedR indicate better protection. For supervised learning, we report Top-1 accuracy on clean test sets; lower accuracy indicates better protection.

Appendix A.1 presents an ablation study on the generator, while Appendix A.2 offers a computational cost analysis, showing that T2UE achieves significant efficiency advantages.

## 4.1 Main Results

**Effectiveness on CLIP.** We evaluate T2UE’s effectiveness against CLIP using zero-shot retrieval metrics (Hit@10/Medr) on Flickr8k, Flickr30k, and Tiny-MSCOCO after pre-training a ResNet50- based CLIP model (Table 2). Critically, the MEM method [25], when applied using only its image noise (MEM-3/MEM-5, simulating a realistic user scenario without text trigger modification), shows negligible protection effect compared to the ‘no protection’ baseline, highlighting its dependence on fragile text triggers. TAP [7] provides some protection but is inconsistent, notably degrading on the larger Flickr30k dataset. While the image-dependent EM [16] method severely degrades performance, it requires image access, violating our core privacy constraint. In contrast, T2UE consistently delivers the strongest protection among the image-agnostic methods across all datasets. For instance, on Flickr30k, T2UE drastically reduces I2T Hit@10 to 18.2, far surpassing TAP (39.7) and MEM-5 (47.1). This demonstrates that T2UE achieves substantial disruption of CLIP training entirely without image access, effectively fulfilling the **"zero-contact"** requirement while providing significantly better protection than comparable baselines.

**Effectiveness of Transferring to Supervised Learning.** We evaluate the transferability of T2UE’s protective noise to standard supervised classification tasks. Specifically, we directly apply the T2UE Generator to supervised learning scenarios on CIFAR-10 and CIFAR-100. Table 3 presents the Top-1 accuracy achieved by a ResNet-18 model trained for 100 epochs on data perturbed by T2UE and various baselines. T2UE(ViT/B-16) and T2UE(ViT/B-32) refer to variants generated using CLIP ViT-B/16 and ViT-B/32 models, respectively. The results clearly demonstrate the effectiveness of T2UE in the supervised setting. On CIFAR-10, T2UE(ViT/B-16) reduces the model’s accuracy to 10.02%, matching the performance of top class-wise methods like CP (10.12%) and leading samplewise methods such as TUE (10.06%). On CIFAR-100, T2UE(ViT/B-16) achieves an accuracy of 1.08%, closely aligning with the strongest class-wise baseline EM (1.07%) and the sample-wise baseline TUE (1.05%). Notably, T2UE attains this highly competitive performance without any access to CIFAR images during noise generation, relying solely on text guidance. More importantly, this result underscores the strong transferability of T2UE’s unlearnable noise, which remains effective even outside its targeted training framework. The transferability to different tasks was a critical bottleneck in prior work [16].

**Effectiveness of Transferring to Different Architectures.** Table 4 presents the results of transfer attack experiments using different models. As shown, the T2UE consistently maintains the unlearnability effect across various models. Notably, sample-level noise exhibits higher model dependency than class-level noise. For sample-level noise, the classification accuracy on unlearnable data remains relatively stable regardless of the training model. In contrast, class-level noise tends to show improved model accuracy when transferred to different architectures. These results demonstrate that T2UE can simultaneously achieve cross-task and cross-model transferability of unlearnable noise, all while maintaining complete “zero-contact” with the original data.

## 4.2 Robustness Evaluations

In this subsection, we perform a standard evaluation of UE by examining: (1) the impact of varying proportions of unlearnable data in the training set, and (2) its robustness to data augmentation.

**Different proportion of unlearnable data in the training set.** Table 5 presents the poisoning effects of T2UE noise on ResNet-18. The experiments were conducted on CIFAR-10/100 with poisoning ratios ranging from 20% to 100%. The results demonstrate that higher poisoning ratios lead to more pronounced effects. For instance, the accuracy on CIFAR-10 drops from 79.46% at 20% poisoning to 10.02% at 100% poisoning, indicating that even partial poisoning can effectively degrade model performance.

**Robustness against data augmentation.** Following prior works [16, 53], we conduct a robustness evaluation using commonly adopted data augmentation techniques, including CutOut [5], MixUp [52], and AutoAugment (AutoAug)[4]. Table 6 presents the performance of T2UE when these augmentations are applied during the training process. T2UE consistently preserves the unlearnability effect across various data augmentation scenarios, demonstrating its robustness in bypassing a range of augmentation strategies and preventing the model from learning meaningful data representations.

**Training curves under supervised learning.** Figure 6 illustrates the training loss and test accuracy over epochs for the ResNet18 model. The “clean” curve represents training on unaltered data, while the others correspond to various poisoned datasets. Except for UC, all methods converge to a test accuracy close to random guessing (10%) by the 100th epoch. Although UC achieves higher accuracy, its training loss closely mirrors that of clean training, indicating a failure to induce true unlearnability.

In contrast, our proposed T2UE method, along with other unlearnable baselines, demonstrates rapid convergence in training loss while maintaining consistently low test accuracy—evidence of effective poisoning. Notably, T2UE sustains low inference accuracy even in the early stages of training, outperforming methods like AR, which are susceptible to early stopping strategies. Furthermore, T2UE preserves strong unlearnability at the sample level.

**Varying text descriptions.** In practice, different users may focus on varying aspects of the same image, the textual descriptions they construct can differ significantly. To better align with real-world scenarios, we explore whether our T2UE can still be effective under such settings. Specifically, this study utilizes the GIT model [42] released by Microsoft to generate diverse textual descriptions for each image in the dataset, thereby expanding and enriching the original training data. To further enhance the diversity of generated descriptions, a series of image augmentation techniques—including random cropping, horizontal flipping, random rotation, and other common methods—are applied to each image, improving the model’s adaptability to various scenarios. The experimental results are presented in Table 7. They show that T2UE is robust to variations in text descriptions, consistently providing effective protection under these conditions.

**Additional Ablation Studies.** In Appendix A, we provide further ablation studies on the effect of generator training duration, along with an efficiency analysis. The results show that (1) the generator progressively improves and achieves strong performance given sufficient initial training, and (2) our T2UE is notably more efficient than existing approaches.

Table 7: Evaluation results on varying different text descriptions. Lower Hit / higher MedR indicates better protection.

<table><tr><td></td><td></td><td colspan="4">Image → Text</td><td colspan="4">Text → Image</td></tr><tr><td>Dataset</td><td>Method</td><td>Hit@1</td><td>Hit@5</td><td>Hit@10</td><td>Medr</td><td>Hit@1</td><td>Hit@5</td><td>Hit@10</td><td>Medr</td></tr><tr><td rowspan="2">Flickr8K</td><td>Clean</td><td>1.0</td><td>3.6</td><td>5.4</td><td>226</td><td>0.54</td><td>2.7</td><td>5.2</td><td>258</td></tr><tr><td>T2UE</td><td>0.3</td><td>2.9</td><td>5.6</td><td>249</td><td>0.48</td><td>2.32</td><td>4.92</td><td>267</td></tr><tr><td rowspan="2">Flickr30K</td><td>Clean</td><td>1.2</td><td>5.4</td><td>8.1</td><td>224</td><td>0.82</td><td>3.24</td><td>5.7</td><td>265</td></tr><tr><td>T2UE</td><td>0.6</td><td>2.3</td><td>3.0</td><td>465</td><td>0.76</td><td>3.08</td><td>4.94</td><td>291</td></tr><tr><td rowspan="2">Tiny-MSCOCO</td><td>Clean</td><td>0.7</td><td>3.1</td><td>4.9</td><td>301</td><td>0.46</td><td>2.00</td><td>3.46</td><td>373</td></tr><tr><td>T2UE</td><td>0.6</td><td>1.9</td><td>2.8</td><td>427</td><td>0.36</td><td>1.66</td><td>2.92</td><td>392</td></tr></table>

## 4.3 Visualization

In Figure 5, we present visualizations of different unlearnable example (UE) methods, including EM [16], TUE [31], and our proposed T2UE. Notably, T2UE produces distinctive noise patterns that are visually different from those generated by other methods.

## 5 Limitation and Conclusion

In this paper, we introduced Text-to-Unlearnable-Example (T2UE), a novel image-independent framework that generates strong protective perturbations without requiring access to the original image, thereby achieving zero-contact data protection. By learning a mapping from text semantics to the perturbation space, T2UE effectively decouples noise generation from sensitive visual content. We demonstrated that T2UE can effectively disrupt CLIP, outperforming existing multimodal unlearnable example baselines. Its protective effect also transfers remarkably well to supervised classification tasks, achieving performance comparable to strong image-dependent methods—all without accessing the images themselves. In addition, T2UE provides substantial computational efficiency gains. By addressing the core limitation of image dependency, T2UE presents a practical, secure, and effective solution for user-centric protection against unauthorized multimodal learning, paving the way for more trustworthy multimodal data protection.

While T2UE offers a promising image-independent approach to generating unlearnable examples, several limitations remain. Its effectiveness can be sensitive to the quality and consistency of input text descriptions—poorly constructed or ambiguous prompts may weaken the protective effect.

![](images/image_08.jpg)

<details>
<summary>heatmap</summary>

| Category | X-axis Range | Y-axis Range | Description |
| --- | --- | --- | --- |
| Clean Image | 0~300 | 0~50 | Image with color-coded overlays |
| EM Noise | 0~300 | 0~50 | Heatmap visualization showing noise intensity across categories |
| EM Image | 0~300 | 0~100 | Image with color-coded overlays |
| TUE Noise | 0~300 | 0~100 | Heatmap visualization showing noise intensity across categories |
| TUE Image | 0~300 | 0~150 | Image with color-coded overlays |
| T2UE Noise | 0~300 | 0~200 | Heatmap visualization showing noise intensity across categories |
| T2UE Image | 0~300 | 0~200 | Image with color-coded overlays |
</details>

Figure 5: Visualization of different UEs and their corresponding protective noise generated on CIFAR-10. The noise patterns have been normalized for visualization purposes.

Additionally, while T2UE achieves performance comparable to that of image-dependent unlearnable example methods, it does not yet surpass them. We believe this gap can be narrowed or even reversed by scaling up the training data used for the generator and further improving the model’s capacity and generalization.

## Acknowledgments

This work is in part supported by the National Key R&D Program of China (Grant No. 2021ZD0112804) and the National Natural Science Foundation of China (Grant No. 62276067). The computations in this research were performed using the CFFF platform of Fudan University.

## References

[1] Abeba Birhane and Vinay Uday Prabhu. Large image datasets: A pyrrhic win for computer vision? In WACV, 2021.
[2] Xinlei Chen, Hao Fang, Tsung-Yi Lin, Ramakrishna Vedantam, Saurabh Gupta, Piotr Dollár, and C Lawrence Zitnick. Microsoft coco captions: Data collection and evaluation server. arXiv preprint arXiv:1504.00325, 2015.
[3] Adam Coates, Andrew Ng, and Honglak Lee. An analysis of single-layer networks in unsupervised feature learning. In Proceedings of the fourteenth international conference on artificial intelligence and statistics, 2011.
[4] Ekin D. Cubuk, Barret Zoph, Dandelion Mané, Vijay Vasudevan, and Quoc V. Le. Autoaugment: Learning augmentation strategies from data. In CVPR, 2019.
[5] Terrance DeVries and Graham W Taylor. Improved regularization of convolutional neural networks with cutout. arXiv preprint arXiv:1708.04552, 2017.
[6] Alexey Dosovitskiy, Lucas Beyer, Alexander Kolesnikov, Dirk Weissenborn, Xiaohua Zhai, Thomas Unterthiner, Mostafa Dehghani, Matthias Minderer, Georg Heigold, Sylvain Gelly, Jakob Uszkoreit, and Neil Houlsby. An image is worth 16x16 words: Transformers for image recognition at scale. ICLR, 2021.
[7] Liam Fowl, Micah Goldblum, Ping-yeh Chiang, Jonas Geiping, Wojciech Czaja, and Tom Goldstein. Adversarial examples make strong poisons. NeurIPS, 2021.
[8] Shaopeng Fu, Fengxiang He, Yang Liu, Li Shen, and Dacheng Tao. Robust unlearnable examples: Protecting data against adversarial learning. In ICLR, 2022.
[9] Samir Yitzhak Gadre, Gabriel Ilharco, Alex Fang, Jonathan Hayase, Georgios Smyrnis, Thao Nguyen, Ryan Marten, Mitchell Wortsman, Dhruba Ghosh, Jieyu Zhang, et al. Datacomp: In search of the next generation of multimodal datasets. In NeurIPS, 2023.
[10] Xueluan Gong, Yuji Wang, Yanjiao Chen, Haocheng Dong, Yiming Li, Mengyuan Sun, Shuaike Li, Qian Wang, and Chen Chen. Armor: Shielding unlearnable examples against data augmentation. arXiv preprint arXiv:2501.08862, 2025.
[11] Kehua Guo, Jie Chen, Tian Qiu, Shaojun Guo, Tao Luo, Tianyu Chen, and Sheng Ren. Medgan: An adaptive gan approach for medical image generation. Computers in Biology and Medicine, 163:107119, 2023.
[12] Hao He, Kaiwen Zha, and Dina Katabi. Indiscriminate poisoning attacks on unsupervised contrastive learning. In ICLR, 2023.
[13] Kaiming He, Xiangyu Zhang, Shaoqing Ren, and Jian Sun. Deep residual learning for image recognition. In CVPR, 2016.
[14] Kashmir Hill. The secretive company that might end privacy as we know it. In Ethics of Data and Analytics, pages 170–177. Auerbach Publications, 2022.
[15] Gao Huang, Zhuang Liu, Laurens Van Der Maaten, and Kilian Q Weinberger. Densely connected convolutional networks. In Proceedings of the IEEE conference on computer vision and pattern recognition, pages 4700–4708, 2017.
[16] Hanxun Huang, Xingjun Ma, Sarah Monazam Erfani, James Bailey, and Yisen Wang. Unlearnable examples: Making personal data unexploitable. In ICLR, 2021.
[17] Zhenlan Ji, Pingchuan Ma, and Shuai Wang. Unlearnable examples: Protecting open-source software from unauthorized neural code learning. In SEKE, 2022.
[18] Chao Jia, Yinfei Yang, Ye Xia, Yi-Ting Chen, Zarana Parekh, Hieu Pham, Quoc Le, Yun-Hsuan Sung, Zhen Li, and Tom Duerig. Scaling up visual and vision-language representation learning with noisy text supervision. In ICML, 2021.
[19] Yujing Jiang, Xingjun Ma, Sarah Monazam Erfani, and James Bailey. Unlearnable examples for time series. In PAKDD, 2024.
[20] Pang Wei Koh and Percy Liang. Understanding black-box predictions via influence functions. In ICML. PMLR, 2017.
[21] Alex Krizhevsky. Learning multiple layers of features from tiny images. 2009.
[22] Wentong Liao, Kai Hu, Michael Ying Yang, and Bodo Rosenhahn. Text to image generation with semantic-spatial aware gan. arXiv preprint arXiv:2104.00567, 2021.
[23] Wentong Liao, Kai Hu, Michael Ying Yang, and Bodo Rosenhahn. Text to image generation with semantic-spatial aware gan. In CVPR, 2022.
[24] Tsung-Yi Lin, Michael Maire, Serge Belongie, James Hays, Pietro Perona, Deva Ramanan, Piotr Dollár, and C Lawrence Zitnick. Microsoft coco: Common objects in context. In ECCV, 2014.
[25] Xinwei Liu, Xiaojun Jia, Yuan Xun, Siyuan Liang, and Xiaochun Cao. Multimodal unlearnable examples: Protecting data against multimodal contrastive learning. In MM, 2024.
[26] Yixin Liu, Kaidi Xu, Xun Chen, and Lichao Sun. Stable unlearnable example: Enhancing the robustness of unlearnable examples via stable error-minimizing noise. In AAAI, volume 38, pages 3783–3791, 2024.
[27] Shayne Longpre, Robert Mahari, Anthony Chen, Naana Obeng-Marnu, Damien Sileo, William Brannon, Niklas Muennighoff, Nathan Khazam, Jad Kabbara, Kartik Perisetla, et al. The data provenance initiative: A large scale audit of dataset licensing & attribution in ai. Nature Machine Intelligence, 2023.
[28] Xingjun Ma, Yifeng Gao, Yixu Wang, Ruofan Wang, Xin Wang, Ye Sun, Yifan Ding, Hengyuan Xu, Yunhao Chen, Yunhan Zhao, et al. Safety at scale: A comprehensive survey of large model safety. arXiv preprint arXiv:2502.05206, 2025.
[29] Mehdi Mirza and Simon Osindero. Conditional generative adversarial nets. Computer Science, pages 2672–2680, 2014.
[30] Alec Radford, Jong Wook Kim, Chris Hallacy, Aditya Ramesh, Gabriel Goh, Sandhini Agarwal, Girish Sastry, Amanda Askell, Pamela Mishkin, Jack Clark, et al. Learning transferable visual models from natural language supervision. In ICML, 2021.
[31] Jie Ren, Han Xu, Yuxuan Wan, Xingjun Ma, Lichao Sun, and Jiliang Tang. Transferable unlearnable examples. arXiv preprint arXiv:2210.10114, 2022.
[32] Vinu Sankar Sadasivan, Mahdi Soltanolkotabi, and Soheil Feizi. Cuda: Convolution-based unlearnable datasets. In CVPR, 2023.
[33] Pedro Sandoval-Segura, Vasu Singla, Jonas Geiping, Micah Goldblum, Tom Goldstein, and David Jacobs. Autoregressive perturbations for data poisoning. NeurIPS, 2022.
[34] Pedro Sandoval-Segura, Vasu Singla, Jonas Geiping, Micah Goldblum, and Tom Goldstein. What can we learn from unlearnable datasets? NeurIPS, 2023.
[35] Christoph Schuhmann, Richard Vencu, Romain Beaumont, Robert Kaczmarczyk, Clayton Mullis, Aarush Katta, Theo Coombes, Jenia Jitsev, and Aran Komatsuzaki. Laion-400m: Open dataset of clip-filtered 400 million image-text pairs. arXiv preprint arXiv:2111.02114, 2021.
[36] Christoph Schuhmann, Romain Beaumont, Richard Vencu, Cade Gordon, Ross Wightman, Mehdi Cherti, Theo Coombes, Aarush Katta, Clayton Mullis, Mitchell Wortsman, et al. Laion-5b: An open large-scale dataset for training next generation image-text models. NeurIPS, 2022.
[37] Karen Simonyan and Andrew Zisserman. Very deep convolutional networks for large-scale image recognition. arXiv preprint arXiv:1409.1556, 2014.
[38] Ye Sun, Hao Zhang, Tiehua Zhang, Xingjun Ma, and Yu-Gang Jiang. Unseg: One universal unlearnable example generator is enough against all image segmentation. In NeurIPS, 2024.
[39] Christian Szegedy, Wei Liu, Yangqing Jia, Pierre Sermanet, Scott Reed, Dragomir Anguelov, Dumitru Erhan, Vincent Vanhoucke, and Andrew Rabinovich. Going deeper with convolutions. In CVPR, 2015.
[40] Carl Vondrick, Hamed Pirsiavash, and Antonio Torralba. Generating videos with scene dynamics. NeurIPS, 29, 2016.
[41] Derui Wang, Minhui Xue, Bo Li, Seyit Camtepe, and Liming Zhu. Provably unlearnable data examples. arXiv preprint arXiv:2405.03316, 2024.
[42] Jianfeng Wang, Zhengyuan Yang, Xiaowei Hu, Linjie Li, Kevin Lin, Zhe Gan, Zicheng Liu, Ce Liu, and Lijuan Wang. Git: A generative image-to-text transformer for vision and language. arXiv preprint arXiv:2205.14100, 2022.
[43] Xianlong Wang, Minghui Li, Wei Liu, Hangtao Zhang, Shengshan Hu, Yechao Zhang, Ziqi Zhou, and Hai Jin. Unlearnable 3d point clouds: Class-wise transformation is all you need. In NeurIPS, 2024.
[44] Qiufeng Wu, Yiping Chen, and Jun Meng. Dcgan-based data augmentation for tomato leaf disease identification. IEEE access, 8:98716–98728, 2020.
[45] Hu Xu, Saining Xie, Xiaoqing Tan, Po-Yao Huang, Russell Howes, Vasu Sharma, Shang-Wen Li, Gargi Ghosh, Luke Zettlemoyer, and Christoph Feichtenhofer. Demystifying clip data. In ICLR, 2024.
[46] Wenhan Yang, Jingdong Gao, and Baharan Mirzasoleiman. Robust contrastive language-image pretraining against data poisoning and backdoor attacks. NeurIPS, 2024.
[47] Ziqing Yang, Xinlei He, Zheng Li, Michael Backes, Mathias Humbert, Pascal Berrang, and Yang Zhang. Data poisoning attacks against multimodal encoders. In ICML. PMLR, 2023.
[48] Kai Ye, Liangcai Su, and Chenxiong Qian. How far are we from true unlearnability? In ICLR, 2025.
[49] Peter Young, Alice Lai, Micah Hodosh, and Julia Hockenmaier. From image descriptions to visual denotations: New similarity metrics for semantic inference over event descriptions. Transactions of the Association for Computational Linguistics, 2:67–78, 2014.
[50] Da Yu, Huishuai Zhang, Wei Chen, Jian Yin, and Tie-Yan Liu. Availability attacks create shortcuts. In SIGKDD, 2022.
[51] Han Zhang, Tao Xu, Hongsheng Li, Shaoting Zhang, Xiaogang Wang, Xiaolei Huang, and Dimitris N Metaxas. Stackgan: Text to photo-realistic image synthesis with stacked generative adversarial networks. In ICCV, 2017.
[52] Hongyi Zhang. mixup: Beyond empirical risk minimization. arXiv preprint arXiv:1710.09412, 2017.
[53] Jiaming Zhang, Xingjun Ma, Qi Yi, Jitao Sang, Yu-Gang Jiang, Yaowei Wang, and Changsheng Xu. Unlearnable clusters: Towards label-agnostic unlearnable examples. In CVPR, 2023.

## A More Ablation Results

## A.1 Impact of Generator Training Duration

![](images/image_09.jpg)

<details>
<summary>line</summary>

| Epoch | Cifar10 | Cifar100 |
| --- | --- | --- |
| 0 | ~0.87 | ~0.58 |
| 50 | ~0.43 | ~0.38 |
| 100 | ~0.19 | ~0.10 |
| 150 | ~0.13 | ~0.05 |
| 200 | ~0.18 | ~0.05 |
| 250 | ~0.11 | ~0.03 |
| 300 | ~0.21 | ~0.02 |
</details>

(a) ViT-B/16

![](images/image_10.jpg)

<details>
<summary>line</summary>

| Epoch | Cifar10 (Accuracy) | Cifar100 (Accuracy) |
| --- | --- | --- |
| 0 | ~0.92 | ~0.65 |
| 25 | ~0.92 | ~0.62 |
| 50 | ~0.90 | ~0.63 |
| 75 | ~0.51 | ~0.43 |
| 100 | ~0.27 | ~0.35 |
| 125 | ~0.20 | ~0.14 |
| 150 | ~0.10 | ~0.02 |
| 175 | ~0.12 | ~0.03 |
| 200 | ~0.10 | ~0.03 |
| 225 | ~0.11 | ~0.03 |
| 250 | ~0.18 | ~0.03 |
| 275 | ~0.18 | ~0.03 |
| 300 | ~0.13 | ~0.02 |
| 325 | ~0.18 | ~0.03 |
| 350 | ~0.12 | ~0.03 |
| 375 | ~0.13 | ~0.03 |
| 400 | ~0.10 | ~0.04 |
| 425 | ~0.10 | ~0.05 |
| 450 | ~0.14 | ~0.04 |
| 475 | ~0.15 | ~0.03 |
| 500 | ~0.11 | ~0.03 |
| 525 | ~0.12 | ~0.04 |
| 550 | ~0.16 | ~0.03 |
| 575 | ~0.13 | ~0.02 |
| 600 | ~0.18 | ~0.04 |
| 625 | ~0.13 | ~0.02 |
</details>

(b) ViT-B/32

Figure 6: Acc across test set with epoch of generator training in CLIP (a) ViT-B/16 and (b) ViT-B/32

We investigate how the T2UE generator’s training duration influences the potency of the generated noise. Figures 6a and 6b plot the downstream ResNet18 test accuracy on CIFAR-10/100 using noise generated at different checkpoint epochs of the ViT-B/16 (trained 300 epochs) and ViT-B/32 (trained 500 epochs) based generators, respectively. A clear inverse relationship is observed: as the generator trains for more epochs, the resulting noise becomes significantly more effective at disrupting downstream classification, leading to substantial accuracy degradation on both CIFAR datasets. The most significant performance drop occurs relatively early in the generator’s training. For instance, with the ViT-B/32 generator (Figure 6b), downstream accuracy plummets within the first 120 epochs before largely stabilizing at near-zero levels. This indicates that the generator progressively learns the mapping from text semantics to potent unlearnable perturbations, achieving strong efficacy after sufficient initial training.

## A.2 Efficiency Analysis

We evaluate the computational efficiency by measuring the time required to generate the unlearnable perturbations for the datasets used in our experiments in Section 4.1. The results are reported in Table 8, which highlights a significant advantage for T2UE. For instance, generating noise for the entire Flickr8k dataset takes merely 0.3 hours with T2UE. This contrasts sharply with the time overhead of baseline methods: 1.1 hours for EM [16], 4.4 hours for MEM-3 [25], 10.2 hours for MEM-5 [25], and the combined cost for TAP [7] which includes noise generation plus a prerequisite model training phase (4.1 hours in total). Similar efficiency gains for T2UE are observed on Flickr30k and MSCOCO. This disparity stems from the computationally intensive nature of the baselines: EM involves nested optimization loops, MEM requires complex joint optimization across modalities, and UAP necessitates costly prerequisite training. T2UE bypasses these intensive procedures by leveraging its pre-trained text-to-perturbation generator for direct noise synthesis. Consequently, T2UE offers a much more practical approach, achieving strong protection effectiveness (as shown previously) with significantly reduced computational cost.

Table 8: Time cost for generating UEs (in hours). Note that the TAP method requires additional training of a separate model, leading to a significantly higher time overhead. In contrast, our proposed T2UE method efficiently constructs UE with the lowest time cost among all compared methods.

<table><tr><td rowspan="2">Method</td><td colspan="3">RN50</td><td colspan="3">ViTB/32</td></tr><tr><td>Flickr8K</td><td>Flickr30K</td><td>MSCOCO</td><td>Flickr8K</td><td>Flickr30K</td><td>MSCOCO</td></tr><tr><td>EM [16]</td><td>1.1</td><td>4.4</td><td>2.7</td><td>1.1</td><td>5.5</td><td>2.3</td></tr><tr><td>MEM3 [25]</td><td>4.4</td><td>36.2</td><td>20.4</td><td>8.0</td><td>33.3</td><td>18.1</td></tr><tr><td>MEM5 [25]</td><td>10.2</td><td>42.4</td><td>21.2</td><td>36.2</td><td>37.2</td><td>21.2</td></tr><tr><td>TAP [7]</td><td>4.1</td><td>8.1</td><td>5.5</td><td>4.0</td><td>8.4</td><td>5.0</td></tr><tr><td>T2UE</td><td>0.3</td><td>1.8</td><td>0.7</td><td>0.3</td><td>1.8</td><td>0.7</td></tr></table>