<!-- MinerU 4.0.9, tier advanced, VLM llama.cpp/Vulkan GPU; automatic extraction, tables require comparison with the PDF. -->

[PDF original](../../Semantic%20Deep%20Hiding%20for%20Robust%20Unlearnable%20Examples.pdf)

# Semantic Deep Hiding for Robust Unlearnable Examples

Ruohan Meng, Chenyu Yi\*, Yi Yu, Siyuan Yang, Bingquan Shen, and Alex C. Kot, Life Fellow, IEEE

**Abstract—Ensuring data privacy and protection has become paramount in the era of deep learning. Unlearnable examples are proposed to mislead the deep learning models and prevent data from unauthorized exploration by adding small perturbations to data. However, such perturbations (**e.g.**, noise, texture, color change) predominantly impact low-level features, making them vulnerable to common countermeasures. In contrast, semantic images with intricate shapes have a wealth of high-level features, making them more resilient to countermeasures and potential for producing robust unlearnable examples. In this paper, we propose a Deep Hiding (DH) scheme that adaptively hides semantic images enriched with high-level features. We employ an Invertible Neural Network (INN) to invisibly integrate predefined images, inherently hiding them with deceptive perturbations. To enhance data unlearnability, we introduce a Latent Feature Concentration module, designed to work with the INN, regularizing the intraclass variance of these perturbations. To further boost the robustness of unlearnable examples, we design a Semantic Images Generation module that produces hidden semantic images. By utilizing similar semantic information, this module generates similar semantic images for samples within the same classes, thereby enlarging the inter-class distance and narrowing the intra-class distance.**

**Extensive experiments on CIFAR-10, CIFAR-100, and an ImageNet subset, against 18 countermeasures, reveal that our proposed method exhibits outstanding robustness for unlearnable examples, demonstrating its efficacy in preventing unauthorized data exploitation.**

**Index Terms—Unlearnable examples, deep hiding, semantic images, general robustness.**

## I. INTRODUCTION

HE rapid growth of deep learning is largely attributed to the vast amounts of “free” data available on the internet. However, a significant portion of these datasets might encompass personal information obtained without clear authorization [1], [2]. Such practices have heightened societal concerns regarding the potential misuse of individual data, particularly when leveraged to develop commercial or potentially malicious models absent the owner’s consent [3]. To address

Ruohan Meng, Chenyu Yi, Yi Yu, Siyuan Yang, and Alex C. Kot are with the School of Electrical and Electronic Engineering, Rapid-Rich Object Search (ROSE) Laboratory, Nanyang Technological University, 639798, Singapore. (e-mail: ruohan.meng@ntu.edu.sg; cyyi@ntu.edu.sg; yuyi0010@e.ntu.edu.sg; siyuan.yang@ntu.edu.sg; eackot@ntu.edu.sg)

Bingquan Shen is with the DSO National Laboratories, Singapore. (e-mail: sbingqua@dso.org.sg)

Correspondence author: Chenyu Yi.

This research work was carried out at the Rapid-Rich Object Search (ROSE) Lab, the NTU-PKU Joint Research Institute (sponsored by the Ng Teng Fong Charitable Foundation), Nanyang Technological University, Singapore. The research is supported by the DSO National Laboratories, under project agreement No. DSOCL22332.

![](images/page_0_image_13.jpg)

<details>
<summary>flowchart</summary>

```mermaid
graph LR
  A["Hidden Semantic Image/Perturbations"] -->|"Publish"| B["Unlearnable Examples"]
  B -->|"Train"| C["DNN"]
  C -->|"Test Accuracy"| D["Clean Image"]
```
</details>

Fig. 1. An illustration depicts the concept of unlearnable examples, where semantic images are cleverly embedded within clean images to create these unlearnable examples. When Deep Neural Networks (DNNs) are trained on these examples, they fail to learn the meaningful features of the clean images due to the embedded semantic perturbations. Consequently, the accuracy of the DNN models on clean data becomes significantly and unpredictably poor.

these concerns, the concept of unlearnable examples [4]–[9] was introduced, which aims to prevent a deep learning model’s ability to discern meaningful features from genuine patterns by introducing minor perturbations to clean images, as shown in Fig. 1.

When we deploy unlearnable examples to protect unauthorized data in the real world, their strong robustness against different countermeasures plays a critical role [10]. Existing methods [11]–[13] mainly focus on improving their robustness against adversarial training [14]–[16], since the unlearnable examples like error-minimization [6] or targeted adversarial poison [8] show vulnerability to adversarial training [14]. However, the general robustness of unlearnable examples against various countermeasures (e.g., data augmentations, data preprocessing) has been ignored. For example, Image Shortcut Squeezing [17] reveals simple JPEG compression and grayscale transformation can significantly impact the effectiveness of most existing unlearnable examples methods; OPS [18] demonstrates strong adversarial robustness, while it is extremely fragile to widely used operations including cutout and median filtering. Consequently, we introduce a Deep Hiding scheme, termed DH, designed to generate robust unlearnable examples by adaptively hiding semantic images enriched with high-level features as poisons, thereby providing fortified protection against unauthorized data exploitation. Several studies [19]–[23] indicate that the natural image with semantic information (e.g., intricate shapes) is robust against data augmentations, data preprocessing, and adversarial training. Additionally, the existing image hiding techniques [24]–[31] support adaptively hiding one image within another. Among them, the Invertible Neural Networks (INNs) [26], [32]–[35] are notable for their outstanding capability to render images virtually invisible.

Specifically, our proposed method employs an INN model to invisibly and adaptively hide semantic images, endowed with rich high-level attributes, into clean images, generating deceptive perturbations. To enhance the effectiveness of unlearnable examples, we introduce the Latent Feature Concentration module (LFC) to limit intra-class variance by regularizing the latent feature distance of the perturbations. Additionally, we design a Semantic Images Generation module to produce hidden semantic images, by controlling the semantic features (i.e., shapes, edges) during the generation process. Capitalizing on similar semantic information, this module generates analogous semantic images for samples within identical categories. These modules increase the inter-class separation and minimize the intra-class variance, enhancing the robustness of unlearnable examples.

In our designed scheme, the deep learning model prioritizes the semantic features of hidden images over those of genuine patterns due to the semantic nature of the hidden features. Additionally, semantic images with complex shapes possess rich high-level attributes that exhibit greater resistance to data countermeasures. In the experiments, we implemented two settings of hidden semantic images: class-wise and samplewise, aligning them to a single class to strike a balance between efficiency and exposure risk. Extensive experiments conducted on CIFAR-10, CIFAR-100, and a subset of ImageNet demonstrate that our method outperforms the vast majority of countermeasures. Across common countermeasures, the ResNet-18 [36] models trained on the perturbed CIFAR-10, CIFAR-100 and ImageNet subset have average test accuracy of 33.01%, 18.95% and 11.39% respectively, compared to the best performance of 39.20%, 23.17% and 22.76% by the other unlearnable examples techniques. Our contributions can be summarized as:

• We conceptualize the generation process of unlearnable examples as an image-hiding challenge. To address this, we first introduce a Deep Hiding scheme that invisibly and adaptively hides semantic images, enriched with high-level attributes, into clean images using an INN model.
• We propose the Latent Feature Concentration module, designed to regularize the intra-class variance of perturbations, enhancing the effectiveness of unlearnable examples. Moreover, we design the Semantic Images Generation module to generate hidden semantic images by maintaining semantic feature consistency within a single class, aiming to amplify the robustness of unlearnable examples.
• Extensive experiments conducted on CIFAR-10, CIFAR-100, and a subset of ImageNet demonstrate that our method of deep hiding semantic features as poisons effectively exhibits outstanding robustness against most countermeasures. Both the average and maximum test accuracies consistently show superior performance for our method, highlighting its efficacy in preventing unau-

thorized data exploitation.

## II. RELATED WORK

## A. Unlearnable examples

To safeguard data from unauthorized scraping, there is an emerging research emphasis on techniques to render data “unlearnable” for machine learning models. Considering the surrogate models utilized in training, denoted as surrogatedependent models, Targeted Adversarial Poisoning (TAP) [8] employs adversarial examples as a more effective form of data poisoning, aiming to ensure that models trained on adversarially perturbed data fail to identify even their original counterparts. Building on this, Error-Minimizing (EM) [6] introduces the concept of “unlearnable examples” and employs “errorminimizing noise” through a bi-level optimization process to make data unlearnable. However, this approach is not robust against adversarial training [14]. To address this limitation, Robust Error-Minimizing (REM) [11] introduces a robust errorminimizing noise by incorporating adversarial training and the expectation over transformation [37] technique. Further enhancing the utility of unlearnable examples, ADVersarially Inducing Noise (ADVIN) [12] and Entangled Features (EntF) [13] propose similar methods to enhance the robustness of adversarial training. On another front, Transferable Unlearnbale Examples (TUE) [38] proposes the classwise separability discriminant to improve their transferability across different training settings and datasets. Unlearnable Clusters (UCs) [39] introduces label-agnostic unlearnable examples with clusterwise perturbations without knowing the label information. TUE and UCs can prevent unsupervised exploitation against contrastive learning to a certain extent. However, the generated perturbation derived from gradient learning strongly requires knowledge from the surrogate model. In contrast, Autoregressive (AR) [40] introduces a surrogate-free methodology, proposing AR perturbations that remain independent of both data and models. Besides, Linear separable Synthetic Perturbations (LSP) [41] investigates the underlying mechanisms of availability attacks, identifying that the perturbations serve as “shortcuts” for learning objectives, and introducing synthetic shortcuts by sampling from Gaussian distributions. Another novel approach is One Pixel Shortcut (OPS) [18], a single pixel in each image results in significant degradation of model accuracy.

## B. Robustness

In different stages of deep learning model training, the unlearnable examples will encounter various data manipulations. For example, in the data pre-processing, the unlearnable examples could be corrupted by noise and compression effects; before we feed them into the model for training, we usually apply data augmentations. However, certain data manipulations can diminish the efficacy of the added perturbation [17], [42]–[45]. It is crucial to make the unlearnable examples generally robust against these data manipulations in real-world applications.

To evaluate the robustness of these generated unlearnable examples, Image Shortcut Squeezing (ISS) [17] utilizes fundamental countermeasures based on image compression techniques, such as grayscale transformation, JPEG compression, and bit-depth reduction (BDR), to counteract the effects of perturbations. In addition, techniques such as Gaussian blur, random cropping and flipping, Cutout [46], Cutmix [47], and Mixup [48] are employed to assess the robustness of unlearnable examples. Additionally, as referenced in the unlearnable examples part, adversarial training (AT) stands as a pivotal method to bolster the resilience of crafted unlearnable examples. More contemporarily, UEraser [49] proposes a combination of effective data augmentation policies and lossmaximizing adversarial augmentations to attack the existing unlearnable examples. AVATAR [50] extends the methodology outlined in DiffPure [51], using diffusion models to counteract intentional perturbations while preserving the essential semantics of training images. And Orthogonal Projection (OP) [52] presents a new attack that allows learning from unlearnable datasets perturbed by classwise, linearly separable perturbations. D-VAE [53] adopts rate-constrained variational autoencoders with perturbations disentanglement to purify the unlearnable datasets.

## C. Image hiding

As deep learning continues to evolve, researchers are exploring methods to seamlessly embed whole images within other images using deep neural networks such as encoderdecoder [24], [25], GAN-based [27], and reinforcement learning-driven models [29] to subtly embed one or multiple images within a container image. Leveraging the inverse property of INN for image-to-image tasks [54], [55], HiNet [56] and DeepMIH [57] employ DWT to decompose the input image, and constrain the hiding to implementation in high-frequency sub-bands for invisible hiding. Similarly, iSCMIS [58], ISN [59], and RIIS [43] hide data by using the inverse property, with some models even simulating data transformations to enhance the robust retrieval of hidden data. According to the above methods, INN shows good potential to achieve image hiding. Note that the robustness in image hiding fields means the extraction performance of generated images undergoing some attacks. However, the robustness of unlearnable examples should be focused on unlearnable stability. In the backdoor and adversarial attack fields, image hiding schemes have notably contributed. Specifically, the Backdoor Injection attack [60] utilizes INN to generate robust and subtle adversarial examples, yielding adversarial images that are both less noticeable and more resilient compared to traditional techniques. Poison Ink [61] exploits image structures, employing image hiding schemes to incorporate trigger patterns, thereby ensuring stealthy and robust backdoor attacks. AdvINN [62] utilizes INN to generate robust and subtle adversarial examples, yielding adversarial images that are both less noticeable and more resilient compared to traditional techniques. Such strategies underscore the significant potential of deep image hiding in bolstering the effectiveness of unlearnable examples.

## III. PROPOSED METHOD

## A. Definition

1) Recalling unlearnable examples: Following the existing unlearnable research [6], [8], [11], [40], [41], we focus on the image classification task in this work. Given a clean dataset $\mathcal { D } _ { c }   =   \{ ( \pmb { x } _ { i } , y _ { i } ) \} _ { i = 1 } ^ { n }$ with n training samples, where $\pmb { x } \in \mathcal { X } \subset \mathbb { R } ^ { d }$ represents the images and $y \in \mathcal { Y } = \{ 1 , \cdots , K \}$ denotes its corresponding labels. We assume an unauthorized party will use a classifier given as $f _ { \theta } : { \mathcal { X } } \to { \mathcal { Y } }$ where $\theta   \in   \Theta$ is the classifier parameters. To safeguard the images from unauthorized training, rather than publishing $\mathcal { D } _ { c } ,$ , existing methods [6], [11] introduce perturbation to clean images, generating an unlearnable dataset as:

$$
\mathcal {D} _ {u} = \left\{\left(\boldsymbol {x} _ {i} + \boldsymbol {\delta} _ {i}, y _ {i}\right) \right\} _ {i = 1} ^ {n}, \tag {1}
$$

where $\pmb { \delta } _ { i } \in \Delta _ { \mathcal { D } } \subset \mathbb { R } ^ { d }$ and $\Delta _ { \mathcal { D } }$ is the perturbation set for $\mathcal { D } _ { c } .$ The objective of unlearnability is to ensure that a classifier $f _ { \theta }$ trained on $\mathcal { D } _ { u }$ exhibits poor performance on test datasets.

2) Proposed unlearnable examples: Current approaches typically generate perturbations either through gradient-based training with a surrogate model or by sampling noise from a predefined distribution in model-free manners. These perturbations lack distinguished semantic high-level features and redundancy, making it challenging to generalize robustness against various countermeasures. It is worth noting that while the perturbations generated in REM and EntF exhibit certain semantic patterns, these patterns are tailored to resist AT and are closely aligned with the clean image’s features. It compromises their effectiveness against other countermeasures. Conversely, we propose an adaptive method for embedding a distinguished semantic image $h _ { i }$ characterized by rich highlevel features and completely uncorrelated with the features of the clean image, within a clean image to generate unlearnable examples. Thus, the generated unlearnable dataset is defined as:

$$
\mathcal {D} _ {u} = \left\{\left(\mathcal {F} \left(\boldsymbol {x} _ {i}, \boldsymbol {h} _ {i}\right), y _ {i}\right) \right\} _ {i = 1} ^ {n}, \tag {2}
$$

where $\mathcal { F } ( \cdot , \cdot )$ represents our hiding model. We adaptively hide predefined semantic images into clean datasets $\mathcal { D } _ { c }$ rather than arbitrary perturbation, inherently introducing deceptive perturbations to mislead classifier $f _ { \theta } ,$ thereby enhancing the effectiveness of unlearnable examples.

## B. Deep hiding scheme for robust unlearnable examples

To generate a resilient unlearnable dataset $\mathcal { D } _ { u } ,$ we introduce the Deep Hiding scheme. This framework incorporates the image hiding model, which integrates the specially-designed Latent Feature Concentration module, and the Semantic Images Generation module. Fig. 2 illustrates the overview of the proposed Deep Hiding scheme.

1) Deep hiding model: Inspired by the image hiding methods [24]–[27], [29], we employ the INN-based hiding model, HiNet [56], as our framework, leveraging its superior generation performance. HiNet employs N affine coupling blocks to form two invertible processes, forward hiding and backward revealing, where the hiding process enables inherently embedding predefined semantic images into clean images, as illustrated in Fig. 2. To facilitate invisible deep hiding, Discrete Wavelet Transform (DWT) $\mathcal { T } ( \cdot )$ is applied to decompose the input clean image $x _ { c } ,$ and hidden semantic image $x _ { h }$ into low and high-frequency sub-bands. We denote the sub-bands features of $x _ { c }$ and $x _ { h }$ as $z _ { c } \; = \; \mathcal { T } ( x _ { c } )$ and $z _ { h } \; = \; \mathcal { T } ( x _ { h } )$ respectively. Let $\boldsymbol { z } _ { c } ^ { i }$ and $z _ { h } ^ { i }$ be the input features of the $i ^ { t h }$ affine coupling block, the forward hiding process of this block can be expressed as:

![](images/page_3_image_2.jpg)

<details>
<summary>flowchart</summary>

```mermaid
graph LR
  subgraph Deep_Hiding_System ["Deep Hiding Scheme"]
  A["Clean Image"] --> B["Hidden Semantic Image"]
  C["Revealed Clean Image"] --> D["Revealed Hidden Semantic Image"]
  B --> E["DWT"]
  D --> F["IWT"]
  E --> G["Forward Hiding"]
  F --> H["Backward Revealing"]
  G --> I["DWT"]
  H --> I
  I --> J["Random Noise"]
    K["Lost Info."] -.-> L["Subtraction"]
    M["Unlearnable Example"] -.-> L
  N["Perturbation (Scaled)"] --> O["Latent Feature Concentration"]
  end

  subgraph Latent_Feature_Concentration ["Latent Feature Concentration"]
  O --> P["Feature Extractor"]
  P --> Q["..."]
  P --> R["..."]
  P --> S["..."]
  Q --> T["Latent feature"]
  R --> U["..."]
  S --> V["..."]
  end

  subgraph Semantic_Image_Generation ["Semantic Image Generation"]
    W["Text Prompts Clustering"] -.-> X["ControlNet"]
    Y["Text Prompt A Giraffe Standing Alone Next to Some Trees."] -.-> X
    Z["Canny Edge"] -.-> X
    AA["Generated Semantic Images"] -.-> AA
  end
```
</details>

Fig. 2. Overall pipeline of the proposed method. A generative model is employed to generate the hidden semantic images. These generated images are then hidden within clean images using a Deep Hiding model. The Latent Feature Concentration module is designed to constrain the intra-class variance by regularizing the latent feature distance of perturbations.

$$
\boldsymbol {z} _ {c} ^ {i} = \boldsymbol {z} _ {c} ^ {i - 1} + \phi \left(\boldsymbol {z} _ {h} ^ {i - 1}\right), \tag {3}
$$

$$
\boldsymbol {z} _ {h} ^ {i} = \boldsymbol {z} _ {h} ^ {i - 1} \odot \exp \left(\alpha \cdot \rho \left(\boldsymbol {z} _ {c} ^ {i}\right)\right) + \eta \left(\boldsymbol {z} _ {c} ^ {i}\right), \tag {4}
$$

where $\phi ( \cdot ) , \; \rho ( \cdot )$ , and $\eta ( \cdot )$ are three sub-modules, sharing the same network structure but with different weights, exp(·) is the Exponential function, ⊙ is the Hadamard product operation, and $\alpha$ controls the weight of exponential operation. Given the output features $z _ { c } ^ { N }$ of total $\bar { N } ^ { t h }$ affine coupling block, the unlearnable examples $\pmb { x } _ { u e } = \mathcal { T } ^ { - 1 } ( \pmb { z } _ { c } ^ { N } )$ are generated by Inverse DWT (IDWT).

To ensure the success of the image-hiding procedure, in the backward revealing process, the obtained unlearnable examples are first decomposed by DWT and then together with the randomly sampled latent noises r feed into the HiNet, resulting in the revealed clean image $\pmb { x } _ { c } ^ { \prime } \; = \; \mathcal { T } ^ { - 1 } ( \pmb { z } _ { c } ^ { 1 } )$ and revealed hidden semantic image $\pmb { x } _ { h } ^ { \prime } = \mathcal { T } ^ { - 1 } ( r ^ { 1 } )$ by subsequent IDWT. Such $z _ { c } ^ { 1 }$ and $r ^ { 1 }$ can be obtained by:

$$
\boldsymbol {z} _ {c} ^ {i - 1} = \left(\boldsymbol {z} _ {c} ^ {i} - \eta \left(\boldsymbol {z} _ {c} ^ {i}\right)\right) \odot \exp \left(- \alpha \cdot \rho \left(\boldsymbol {z} _ {c} ^ {i}\right)\right), \tag {5}
$$

$$
r ^ {i - 1} = r ^ {i} - \phi \left(r ^ {i - 1}\right). \tag {6}
$$

Our primary objective is to generate invisible unlearnable examples. To ensure this, motivated by [63], we restrict them to a specific radius ϵ of perturbation, characterized by the hiding loss as:

$$
\mathcal {L} _ {\text {hide}} \left(\boldsymbol {x} _ {u e}, \boldsymbol {x} _ {c}\right) = \max \left(\operatorname{MSE} \left(\boldsymbol {x} _ {u e}, \boldsymbol {x} _ {c}\right), \epsilon^ {2}\right). \tag {7}
$$

For consistency and fairness, we adopt the same radius $\epsilon \quad = \quad 8 / 2 5 5$ as utilized in existing unlearnable examples methodologies [6], [8], [11].

In addition, we adapt the loss functions from HiNet [26] to concurrently ensure optimal image hiding performance. Consequently, the total loss for the Deep Hiding module is represented as follows:

$$
\begin{array}{l} \mathcal {L} _ {\mathrm{DH}} = \mathcal {L} _ {\text {hide}} \left(\boldsymbol {x} _ {u e}, \boldsymbol {x} _ {c}\right) + \omega_ {1} \cdot \mathcal {L} _ {\text {freq}} \left(\mathcal {H} \left(\boldsymbol {x} _ {u e}\right) _ {L L}, \mathcal {H} \left(\boldsymbol {x} _ {c}\right) _ {L L}\right) \tag {8} \\ + \omega_ {2} \cdot \mathcal {L} _ {\text {reveal}} \left(\boldsymbol {x} _ {h} ^ {\prime}, \boldsymbol {x} _ {h}\right). \\ \end{array}
$$

As verified by [26], [32], information hidden in highfrequency components is less detectable than in low-frequency ones. To optimize the anti-detection and invisibility of unlearnable examples, it’s crucial to maintain the low-frequency sub-bands to closely resemble those of clean images. $\mathcal { L } _ { \mathrm { f r e q } }$ measures the $L _ { 2 }$ distance between the low-frequency subbands of clean images and unlearnable examples, further bolstering the stealthiness. $\mathcal { H } ( \cdot ) _ { L L }$ is the function of extracting low-frequency sub-bands after wavelet decomposition. Additionally, $\mathcal { L } _ { \mathrm { r e v e a l } } \left( \pmb { x } _ { h } ^ { \prime } , \pmb { x } _ { h } \right)$ measures the $L _ { 2 }$ distance between revealed hidden images $\boldsymbol { x } _ { h } ^ { \prime }$ and hidden semantic images $x _ { h }$ It’s important to note that the revealing process is crucial for reserving the semantic information in image hiding.

2) Latent feature concentration: Although the deep hiding model effectively embeds high-level features from predefined semantic images into clean images, delivering outstanding unlearnability (see Section IV-C), the adaptive hiding process still results in latent features of perturbations with non-uniform intra-class distribution. A compact distribution of these latent features could significantly mislead the learning trajectory of DNNs, by offering a distinct learning shortcut across similar intra-class images. To address this, we introduce the Latent Feature Concentration module, specifically designed to regularize the intra-class variance of perturbations, further boosting the effectiveness of unlearnable examples. The perturbation represents the variation between the generated unlearnable example and its corresponding clean image, defined as:

$$
\boldsymbol {x} _ {p m} = \boldsymbol {x} _ {u e} - \boldsymbol {x} _ {c}. \tag {9}
$$

We utilize a pre-trained ResNet-18 [36] as the feature extractor, denoted by $\mathcal { G } ( \cdot )$ . The latent features are extracted from the output final convolution layer. Our objective is to minimize the variation between latent features derived from the perturbation maps for images within the same class. Consequently, the concentration loss $\mathcal { L } _ { \mathrm { c o n c } }$ is represented as:

$$
\mathcal {L} _ {\mathrm{conc}} = \sum_ {i, j, y ^ {i} = y ^ {j}} \operatorname{dis} \left(\mathcal {G} (\boldsymbol {x} _ {p m} ^ {i}), \mathcal {G} (\boldsymbol {x} _ {p m} ^ {j})\right), \tag {10}
$$

where dis(·, ·) denotes the cosine distance between the two flattened latent features, and y represents the label. Thus, the total loss of our proposed method is described as:

$$
\mathcal {L} _ {\text {total}} = \mathcal {L} _ {\mathrm{DH}} + \omega_ {3} \cdot \mathcal {L} _ {\text {conc}}. \tag {11}
$$

3) Semantic images generation: Though the deep hiding model can embed human-imperceptible perturbations, it can not ensure efficacy when the hidden images are randomly picked. Consequently, we introduce a generative method specifically designed for controlled hidden semantic image generation, aiming to achieve desired intra-class and interclass distributions; that is, a distinct inter-class distance complemented by a minimal intra-class distance. As shown in Fig. 2, we use pre-trained generative models, i.e., Stable Diffusion [64] and ControlNet [65], to generate hidden semantic images by controlling both text prompts and canny edge maps. These text prompts, sourced from [66], characterize images from the COCO datasets [67]. The canny edge maps are derived by applying the canny filter to the corresponding images.

To maximize the intra-class distance among hidden semantic images, we choose text prompts that exhibit the greatest variation from the rest. We first cluster all text prompts using K-Means [68] based on their semantic features via CLIP text extractor [69]. Subsequently, we identify the top-k distinct clusters, where k represents the number of classes in the targeted dataset $\mathcal { D } _ { c } .$ . In each of these clusters, we choose the text prompt nearest to the cluster center, which represents a unique semantic feature. To minimize the intra-class distance among hidden semantic images, we ensure their consistency in high-level features by controlling the key image attributes, $i . e .$ , shapes. Consequently, we obtain a canny edge map of the text-corresponding image, which acts as the condition for ControlNet [70]. Then, we use the Stable Diffusion [71] model and ControlNet (SD+C) to generate semantic images as the hidden semantic input $x _ { h }$ for our DH scheme. With these specifically generated hidden semantic images, our deep hiding model can guarantee the general robustness of the unlearnable examples.

## C. Properties of deep hiding scheme

DNNs are capable of learning complex features for image understanding. However, they are inclined to overfit to the patterns that are much easier to learn [72], in alignment with the “Principle of Least Effort” [73]. With this phenomenon, many unlearnable examples are proposed to protect the data from being learned by DNNs. Consequently, DNNs predominantly focus on misleading perturbations rather than the intended solutions. Our Deep Hiding scheme exploits the same principle. In our proposed scheme, clean images within a given class are embedded with similar hidden semantic images that share the same global shape but differ in their local textures. Compared to the complex features in the original images, the embedded similar semantic information is much easier to be learned by DNNs. The visual representation in Fig. 4 demonstrates that the perturbations generated by our scheme exhibit clear separability, marked by straightforward decision boundaries. Besides, We utilize Grad-CAM [74] to visualize the attention of DNNs in Fig. 3. It is obvious that the attention is redirected toward the desk (the hidden semantic image) rather than the snake (the clean image) during classification. While DNNs can take non-semantic features as “shortcuts” for more effortless learning, these features are vulnerable to simple data augmentations and data processing. Different from the existing unlearnable examples methods, we incorporate natural images as hidden semantic images to generate perturbations. These perturbations, enriched with deep highlevel semantic attributes, exhibit robustness against diverse countermeasures. As illustrated in Fig. 3, the hidden semantic information can mislead the DNNs to similar key regions after most countermeasures. These findings affirm the efficacy and resilience of using natural images as hidden semantic information when faced with various countermeasures.

## IV. EXPERIMENTS

## A. Experimental setups

1) Datasets and models: We use three image classification datasets: CIFAR-10 [75], CIFAR-100 [75], and 100-class subset of ImageNet [76] in our experiments. We evaluate on the ResNet-18 [36] in our main experiments. To study the transferability of the proposed DH scheme, we consider models with diverse architectures, including ResNet-50 [36], VGG-19 [77], DenseNet-121 [78], and ViT [79].
2) DH model training: Our training exclusively utilizes the ImageNet subset comprising 100 classes for the DH model. As detailed in Section III-B3, for each class, we generate 100 semantic images using paired text prompts and canny edge maps. Our training configuration is as follows: 5k iterations, the Adam optimizer [80] with hyper-parameters set at $\beta _ { 1 } = 0 . 5 ,   \beta _ { 2 } = 0 . 9 9 9$ , and $\epsilon = 1 0 ^ { - 6 } ;$ a consistent learning rate of $1   \times   1 0 ^ { - 4 . 5 }$ ; and a mini-batch size of 24. To ensure stable model training, we assign weights of 1 to $\omega _ { 1 }$ and $\omega _ { 2 }$ , and a weight of 0.0001 to $\omega _ { 3 }$ across all experiments. Subsequent to this, the pre-trained DH model is used to generate unlearnable examples across the three datasets: CIFAR-10, CIFAR-100, and the ImageNet subset. Under the premise of ensuring invisibility, we further adopted a strict boundary to clip the scale of perturbations within 8/255. The unlearnable examples generation follows two settings: class-wise setting and samplewise setting. In the class-wise setting, we hide a consistent semantic image in the clean images of each class; whereas in the sample-wise setting, the hidden semantic images in each

![](images/page_5_image_2.jpg)

<details>
<summary>text_image</summary>

Clean
Hidden
Vanilla
Cutout
Cutmix
Mixup
MeanF
MedianF
BDR
Gray
GaussN
GaussF
JPEG10
JPEG50
AT
</details>

Fig. 3. Grad-CAM visualization of unlearnable examples derived from the ImageNet subset under different countermeasures. Note that red regions typically indicate the areas the model paid the most attention to, while Blue regions colors indicate less attention.

![](images/page_5_image_4.jpg)

Fig. 4. The t-SNE visualization of the models’ feature representations on the clean samples (left) and the perturbation generated by our DH scheme (right) on CIFAR-10.

class are sampled from the generative model with the same text prompt and canny edge map, so they share the same global shape but differ in their local textures.

3) Classifier training: To evaluate the effectiveness of the generated unlearnable examples, we employ a standard classification problem. For both CIFAR-10 and CIFAR-100 datasets, we follow the official train-test division. Regarding the ImageNet subset, we allocate 20% of images from the first 100 classes of the official ImageNet training set for training purposes, using all related images in the official validation set for testing. We perform 40,000 iterations on the CIFAR-10 and CIFAR-100 datasets, and 8,000 iterations on the ImageNet subset.
4) Compared methods: We compare the proposed deep hiding scheme with six state-of-the-art unlearnable examples methods, including four surrogate-dependent methods, EM [6], REM [11], TAP [8], and EntF [13], and three surrogate-free methods, AR [40], LSP [41], and OPS [18]. The perturbations from EM, REM, TAP, and EntF are constrained by an $L _ { \infty }$ norm of 8; AR and LSP use $L _ { 2 } \: = \: 1 ; \: \mathrm { O P S }$ is $L _ { 0 } \; = \; 1 . 0 .$ Additionally, to assess whether other hiding methods used in backdoor attacks can generate unlearnable examples, we tested the performance of methods including Poison Ink [61] and AdvINN [62]. AdvINN is implemented using the proposed Classifier Guided Target Image (CGT) strategy to execute attacks, according to their public codes. Note that we reimplemented all methods based on the publicly available codes.
5) Robustness evaluation: To evaluate the robustness of our generated unlearnable examples, we train ResNet-18 with

them across a variety of data augmentations and preprocessing methods, as suggested in [17]. For augmentation, we employ vanilla (random crop and resize), cutout [46], cutmix [47], and mixup [48]. Additionally, we utilize seven data preprocessing techniques: Mean Filter (MeanF), Median Filter (MedianF), Bit-Depth Reduction (BDR), Grayscale transformation (Gray), Gaussian Noise (GaussN), Gaussian Filter (GaussF), and JPEG compression. Additionally, we also implement adversarial training (AT) [14] with the constraint on both $L _ { 2 }$ and $L _ { \infty }$ norms. In alignment with the settings in REM [11], the specific details are as follows.

• Basic Augmentation (Vanilla). For CIFAR-10 and CIFAR-100 datasets, our data augmentation comprises random flipping, padding by 4 pixels on each side, followed by random cropping to a size of 32 × 32. Each image’s pixels are then rescaled to the range [−0.5, 0.5]. In the case of the ImageNet subset, we augment the data using random cropping, resizing the images to a 224×224 dimension, implementing random flipping, and then rescaling every pixel to the interval [−0.5, 0.5].
Cutout. We adjust the sizes of the squared cutout box to match the image sizes of the datasets: 16 for CIFAR-10 and CIFAR-100, and 112 for the ImageNet subset. The cutout box is randomly placed within each image and maintains a consistent size across all images.
• Cutmix. For a given sample, we first generate a square bounding box centered at a randomly chosen position with a randomly selected size (ranging from 0 to 1). The content within this bounding box is then replaced with content cropped from another randomly chosen image from the same mini-batch.
• Mixup. We randomly select another image and blend it with the current sample using a randomly chosen weight (ranging from 0 to 1). We retain the original label for the current sample during loss function computation.
• Filters. We use a kernel size of 3 for median, mean, and Gaussian smoothing (with a standard deviation of 0.1).
• Bit-Depth Reduction (BDR). We implement 2 bits to perform BDR transformation.
• Grayscale. We first calculate the weighted sum of the three channels and then replicate it across all three

TABLE I TEST ACCURACY (%) OF MODELS TRAINED ON UNLEARNABLE EXAMPLES FROM CIFAR-10, CIFAR-100, AND IMAGENET SUBSET AGAINST DATA AUGMENTATIONS, DATA PREPROCESSING, AND ADVERSARIAL TRAINING. NUMBERS IN BOLD AND UNDERLINE NUMBERS INDICATE THE BEST AND SECOND-BEST RESULTS, RESPECTIVELY.

<table><tr><td></td><td>Norm</td><td>Method</td><td>Vanilla</td><td>Cutout</td><td>Cutmix</td><td>Mixup</td><td>MeanF</td><td>MedianF</td><td>BDR</td><td>Gray</td><td>GaussN</td><td>GaussF</td><td>JPEG10</td><td>JPEG50</td><td> $AT(L_{\infty})$ </td><td> $AT(\ell_2)$ </td><td>Mean</td><td>Max</td></tr><tr><td rowspan="10">CIFAR-10</td><td>-</td><td>Clean</td><td>94.59</td><td>95.00</td><td>94.77</td><td>94.96</td><td>49.70</td><td>86.64</td><td>89.07</td><td>92.80</td><td>88.71</td><td>94.54</td><td>85.22</td><td>90.89</td><td>84.19</td><td>83.54</td><td>87.47</td><td>95.00</td></tr><tr><td rowspan="6"> $L_{\infty}$ </td><td>EM [6]</td><td>10.10</td><td>10.00</td><td>15.39</td><td>16.82</td><td>10.63</td><td>24.27</td><td>35.90</td><td>69.29</td><td>32.96</td><td>10.01</td><td>84.80</td><td>87.82</td><td>84.28</td><td>83.33</td><td>41.11</td><td>87.82</td></tr><tr><td>REM [11]</td><td>29.00</td><td>29.42</td><td>26.13</td><td>28.37</td><td>19.07</td><td>32.80</td><td>39.93</td><td>69.83</td><td>39.97</td><td>28.67</td><td>84.15</td><td>77.65</td><td>85.93</td><td>84.87</td><td>48.27</td><td>85.93</td></tr><tr><td>TAP [8]</td><td>25.90</td><td>32.69</td><td>26.77</td><td>40.46</td><td>31.68</td><td>65.12</td><td>80.25</td><td>26.36</td><td>88.66</td><td>26.09</td><td>84.77</td><td>90.31</td><td>83.57</td><td>82.74</td><td>56.10</td><td>90.31</td></tr><tr><td>EntF [13]</td><td>91.50</td><td>91.30</td><td>90.93</td><td>92.52</td><td>17.85</td><td>70.28</td><td>91.46</td><td>80.33</td><td>90.31</td><td>79.79</td><td>74.36</td><td>83.56</td><td>75.86</td><td>74.05</td><td>78.86</td><td>92.52</td></tr><tr><td>DH(S)</td><td>12.93</td><td>14.03</td><td>13.80</td><td>15.89</td><td>16.06</td><td>25.56</td><td>28.28</td><td>25.65</td><td>65.11</td><td>12.30</td><td>84.93</td><td>89.18</td><td>83.44</td><td>83.27</td><td>37.47</td><td>89.18</td></tr><tr><td>DH(C)</td><td>10.08</td><td>10.00</td><td>10.81</td><td>10.05</td><td>10.20</td><td>15.58</td><td>17.31</td><td>10.32</td><td>30.23</td><td>10.01</td><td>83.98</td><td>78.96</td><td>82.21</td><td>82.34</td><td>33.01</td><td>83.98</td></tr><tr><td> $L_0$ </td><td>OPS [18]</td><td>16.53</td><td>89.73</td><td>83.91</td><td>34.88</td><td>17.31</td><td>86.86</td><td>43.04</td><td>16.65</td><td>36.72</td><td>15.10</td><td>82.79</td><td>57.00</td><td>9.42</td><td>65.60</td><td>46.82</td><td>89.73</td></tr><tr><td rowspan="2"> $L_2$ </td><td>LSP [41]</td><td>19.07</td><td>19.87</td><td>20.89</td><td>26.99</td><td>28.85</td><td>29.85</td><td>66.19</td><td>82.47</td><td>19.25</td><td>16.19</td><td>83.01</td><td>57.87</td><td>84.59</td><td>83.96</td><td>45.65</td><td>84.59</td></tr><tr><td>AR [40]</td><td>13.31</td><td>11.35</td><td>12.21</td><td>13.30</td><td>12.38</td><td>17.04</td><td>37.42</td><td>34.81</td><td>42.29</td><td>12.56</td><td>85.08</td><td>89.63</td><td>83.17</td><td>84.29</td><td>39.20</td><td>89.63</td></tr><tr><td rowspan="10">CIFAR-100</td><td>-</td><td>Clean</td><td>75.82</td><td>74.45</td><td>76.32</td><td>77.07</td><td>14.72</td><td>50.72</td><td>63.51</td><td>70.04</td><td>62.41</td><td>75.86</td><td>57.35</td><td>68.59</td><td>58.25</td><td>57.40</td><td>63.04</td><td>77.07</td></tr><tr><td rowspan="6"> $L_{\infty}$ </td><td>EM [6]</td><td>2.84</td><td>12.05</td><td>7.67</td><td>12.86</td><td>13.52</td><td>43.61</td><td>62.12</td><td>62.37</td><td>62.01</td><td>73.47</td><td>57.29</td><td>67.50</td><td>57.89</td><td>56.63</td><td>42.27</td><td>73.47</td></tr><tr><td>REM [11]</td><td>7.13</td><td>10.32</td><td>11.25</td><td>8.65</td><td>5.90</td><td>12.31</td><td>19.95</td><td>48.48</td><td>26.27</td><td>7.32</td><td>57.15</td><td>65.10</td><td>58.90</td><td>58.75</td><td>28.39</td><td>65.10</td></tr><tr><td>TAP [8]</td><td>14.00</td><td>16.55</td><td>15.99</td><td>22.56</td><td>5.86</td><td>31.95</td><td>55.12</td><td>8.90</td><td>61.40</td><td>13.95</td><td>56.56</td><td>66.67</td><td>56.53</td><td>55.53</td><td>34.40</td><td>66.67</td></tr><tr><td>EntF [13]</td><td>72.55</td><td>69.65</td><td>70.68</td><td>73.81</td><td>8.67</td><td>36.87</td><td>55.22</td><td>67.00</td><td>58.54</td><td>73.10</td><td>51.42</td><td>63.69</td><td>52.44</td><td>50.66</td><td>57.46</td><td>73.81</td></tr><tr><td>DH(S)</td><td>7.81</td><td>4.80</td><td>10.15</td><td>10.27</td><td>10.14</td><td>22.23</td><td>38.13</td><td>15.50</td><td>52.43</td><td>7.97</td><td>56.01</td><td>65.72</td><td>56.48</td><td>56.38</td><td>29.57</td><td>65.72</td></tr><tr><td>DH(C)</td><td>1.22</td><td>1.01</td><td>1.22</td><td>1.09</td><td>1.51</td><td>2.72</td><td>12.08</td><td>0.96</td><td>19.86</td><td>1.01</td><td>55.07</td><td>53.83</td><td>56.91</td><td>56.34</td><td>18.91</td><td>56.91</td></tr><tr><td> $L_0$ </td><td>OPS [18]</td><td>11.69</td><td>71.36</td><td>64.25</td><td>12.59</td><td>3.18</td><td>49.74</td><td>19.31</td><td>18.70</td><td>17.30</td><td>11.79</td><td>56.72</td><td>48.71</td><td>10.22</td><td>48.24</td><td>31.70</td><td>64.25</td></tr><tr><td rowspan="2"> $L_2$ </td><td>LSP [41]</td><td>2.68</td><td>2.55</td><td>2.69</td><td>4.39</td><td>7.15</td><td>6.76</td><td>28.23</td><td>42.77</td><td>22.42</td><td>2.19</td><td>55.23</td><td>33.60</td><td>57.45</td><td>56.32</td><td>23.17</td><td>57.45</td></tr><tr><td>AR [40]</td><td>1.50</td><td>1.47</td><td>1.56</td><td>1.37</td><td>5.35</td><td>3.89</td><td>28.28</td><td>19.68</td><td>59.34</td><td>1.57</td><td>56.99</td><td>65.72</td><td>58.33</td><td>56.60</td><td>25.83</td><td>65.72</td></tr><tr><td rowspan="7">ImageNet subset</td><td>-</td><td>Clean</td><td>63.93</td><td>64.02</td><td>55.10</td><td>64.55</td><td>19.92</td><td>36.08</td><td>56.63</td><td>68.35</td><td>50.62</td><td>65.40</td><td>56.83</td><td>69.36</td><td>48.24</td><td>62.32</td><td>55.81</td><td>69.36</td></tr><tr><td rowspan="5"> $L_{\infty}$ </td><td>EM [6]</td><td>28.99</td><td>18.78</td><td>17.61</td><td>36.55</td><td>7.46</td><td>32.60</td><td>53.43</td><td>17.93</td><td>44.63</td><td>26.04</td><td>53.41</td><td>56.96</td><td>43.56</td><td>42.26</td><td>34.30</td><td>56.96</td></tr><tr><td>REM [11]</td><td>14.78</td><td>14.10</td><td>11.73</td><td>19.88</td><td>15.32</td><td>14.12</td><td>16.48</td><td>44.74</td><td>15.96</td><td>15.34</td><td>50.50</td><td>17.14</td><td>47.52</td><td>21.10</td><td>22.76</td><td>50.50</td></tr><tr><td>TAP [8]</td><td>7.96</td><td>15.02</td><td>15.18</td><td>23.08</td><td>10.44</td><td>15.02</td><td>47.97</td><td>22.93</td><td>46.84</td><td>12.80</td><td>53.40</td><td>37.98</td><td>44.18</td><td>41.56</td><td>28.17</td><td>53.40</td></tr><tr><td>DH(S)</td><td>2.70</td><td>2.98</td><td>1.86</td><td>3.74</td><td>3.44</td><td>18.18</td><td>10.70</td><td>2.22</td><td>19.04</td><td>2.58</td><td>27.84</td><td>18.02</td><td>44.24</td><td>30.76</td><td>13.45</td><td>44.24</td></tr><tr><td>DH(C)</td><td>2.42</td><td>2.30</td><td>3.28</td><td>1.94</td><td>2.38</td><td>3.78</td><td>8.44</td><td>1.38</td><td>27.48</td><td>1.84</td><td>25.12</td><td>10.24</td><td>42.14</td><td>26.66</td><td>11.39</td><td>42.14</td></tr><tr><td> $L_2$ </td><td>LSP [41]</td><td>18.18</td><td>9.52</td><td>34.16</td><td>9.76</td><td>4.14</td><td>5.20</td><td>43.38</td><td>52.66</td><td>34.28</td><td>17.92</td><td>51.80</td><td>49.06</td><td>42.26</td><td>41.08</td><td>29.53</td><td>52.66</td></tr></table>

channels.

• Gaussian Noise. We generate noise for each sample with a distribution of N (0, 0.1).
• JPEG Compression. JPEG compression qualities set at 10% and 50%.
• Adversarial Training (AT) [14]. The $L _ { \infty }$ and $L _ { 2 }$ normbounded perturbations of scale 8/255 and 1, respectively. PGD-10 is employed with a step size of 2/255, training the model on CIFAR-10 for 100 epochs.

In addition to these common countermeasures, we also evaluate the robustness of our method against more contemporary countermeasures using their public codes, including Orthogonal Projection (OP) [52], UEraser [49], and AVATAR [50]. Furthermore, we assess the robustness of our method in an unsupervised contrastive learning scheme, i.e., SimCLR [81].

## B. Invisibility of the proposed method

In this part, invisibility is first assessed using both subjective and objective metrics. For subjective evaluation, we visualize the generated unlearnable examples with their perturbations on the ImageNet subset using our hiding scheme, as shown in Figure 5. Regarding invisibility, our generated unlearnable examples are more imperceptible compared to those from other methods. Additionally, our generated perturbations consistently display high-level features, such as shape, that align with the hidden semantic image. For objective evaluation, we assess the image quality using PSNR between the generated unlearnable examples and their corresponding clean images to analyze the perturbation scale on the ImageNet subset. The results of PSNR are as follows: DH(C) at 36.25, EM at 34.17, REM at 33.96, TAP at 35.86, and LSP at 23.59. We observe that with a perturbation scale of $\epsilon = 8 / 2 5 5$ under the $L _ { \infty }$ norm, the proposed DH method achieves superior image quality. This improvement is due to DH is an adaptive hiding process through INN. In contrast, existing methods that generate unlearnable examples based on the image domain typically restrict the scale of perturbations by the $L _ { p }$ norm, resulting in uniformly distributed perturbations across the entire image. These findings underscore the remarkable invisibility of our proposed DH method.

## C. Effectiveness of the proposed method

Under the strict bound and better invisibility, we evaluate the efficacy of the proposed method by training ResNet18 with unlearnable examples and testing on clean images. In Table I, we present detailed test accuracy results across three datasets: CIFAR-10, CIFAR-100, and the ImageNet subset. Notably, both our class-wise and sample-wise settings consistently achieve the best performance on all three datasets. Specifically, the results of the vanilla training show that our classwise setting degrades the test accuracy to 10.08%, 1.22%, and 2.42% for three datasets respectively, which are nearly random-guessing. It indicates that the models can not learn any useful semantic information for the classification task once we hide the semantic images into clean images. In contrast, the other unlearnable examples techniques fail to maintain consistent performance across datasets. For instance, both EM and LSP result in much higher test accuracies on ImageNet. Even though we use a sample-wise setting to reduce the exposure risk of the hidden image, it still achieves promising performance across datasets, especially on ImageNet. We hypothesize that our unlearnable examples carry abundant information due to their semantic image nature, making them more generally effective in various scenarios.

![](images/page_7_image_2.jpg)

<details>
<summary>text_image</summary>

Clean
Ours (S)
EM
REM
TAP
LSP
Unlearnable
Examples
Hidden
/Perturbation
Unlearnable
Examples
Hidden
/Perturbation
</details>

Fig. 5. The visualization of the unlearnable examples generated by different methods is shown in columns 2-5, where the second and fourth rows correspond to the perturbation maps. Perturbations are absoluted and normalized to [0,1] for a better view.

TABLE II TEST ACCURACY (%) OF MODELS TRAINED ON UNLEARNABLE EXAMPLES FROM CIFAR-10 AGAINST MORE CONTEMPORARY COUNTERMEASURE. NUMBERS IN BOLD AND UNDERLINE NUMBERS INDICATE THE BEST AND SECOND-BEST RESULTS, RESPECTIVELY.

<table><tr><td>Norm</td><td>Method</td><td>OP</td><td>UEraser</td><td>AVATAR</td><td>SimCLR</td><td>Mean</td><td>Max</td></tr><tr><td></td><td>Clean</td><td>90.16</td><td>92.88</td><td>90.35</td><td>74.03</td><td>86.86</td><td>92.88</td></tr><tr><td rowspan="6"> $L_{\infty}$ </td><td>EM [6]</td><td>65.17</td><td>45.52</td><td>90.95</td><td>71.66</td><td>68.33</td><td>90.95</td></tr><tr><td>REM [11]</td><td>25.83</td><td>63.18</td><td>88.49</td><td>68.59</td><td>61.52</td><td>88.49</td></tr><tr><td>TAP [8]</td><td>14.74</td><td>76.21</td><td>90.71</td><td>72.01</td><td>63.42</td><td>90.71</td></tr><tr><td>EntF [13]</td><td>88.43</td><td>90.73</td><td>90.67</td><td>71.39</td><td>85.31</td><td>90.73</td></tr><tr><td>DH(S)</td><td>14.44</td><td>25.54</td><td>88.69</td><td>72.70</td><td>50.34</td><td>88.69</td></tr><tr><td>DH(C)</td><td>10.00</td><td>20.52</td><td>87.38</td><td>71.27</td><td>47.29</td><td>87.38</td></tr><tr><td> $L_{0}$ </td><td>OPS [18]</td><td>87.94</td><td>72.52</td><td>66.16</td><td>71.42</td><td>74.51</td><td>87.94</td></tr><tr><td rowspan="2"> $L_{2}$ </td><td>LSP [41]</td><td>87.99</td><td>85.07</td><td>85.69</td><td>72.27</td><td>82.76</td><td>87.99</td></tr><tr><td>AR [40]</td><td>13.03</td><td>93.16</td><td>91.57</td><td>73.33</td><td>67.77</td><td>93.16</td></tr></table>

## D. Robustness of the proposed method

To evaluate the robustness of our generated unlearnable examples, we adopt a variety of common countermeasures, including four data augmentation, seven data preprocessing techniques, and AT with constraints on both $L _ { 2 }$ and $L _ { \infty }$ norms. As shown in Table I, the experimental results demonstrate that the proposed method consistently outperforms the other techniques, exhibiting robust performance against most countermeasures. On CIFAR-10 dataset, our method reduces the test accuracy to 10%∼17.31% across a broad range of countermeasures. Despite that, DH shows less robustness against JPEG compression and AT, similar to the majority of other methods except OPS and EntF. OPS modifies only one pixel, which might be less affected by JPEG compression and could remain unnoticed in AT scenarios. However, it inherently has trouble with the cropping and filtering operation, leading to diminished results in scenarios like cutout, cutmix, and median filtering. As for EntF, although it demonstrates relative robustness to AT, it is less effective against other countermeasures. This highlights a trade-off where some methods may excel in specific conditions but falter in others. As for mean test accuracy, DH outperforms others with the lowest value noted at 6.19%. This represents a significant improvement over the next best method, AR, and suggests that DH is generally more effective at generating robust unlearnable examples against various countermeasures. Additionally, for maximum test accuracy, which represents the highest performance among all countermeasures, DH still demonstrates superior performance, indicating its potential efficacy in real-world scenarios. The experimental results on CIFAR-100 dataset are similar to those on CIFAR-10. Regarding the mean values, our method still outperforms LSP by about 4% and exceeds EntF by nearly 40% on CIFAR-100. Furthermore, when considering the maximum values, our method consistently achieves the best performance. This highlights the superiority of our proposed method. For ImageNet subset, DH shows more significant improvements compared to other methods and demonstrates general robustness. We speculate that the robustness of ImageNet stems from its high resolutions, which are capable of carrying a richer array of semantic image information. Additionally, since the hiding model is trained on the ImageNet dataset, the effectiveness of hiding is also improved. Overall, by observing the mean and maximum robustness assessment values, our method has achieved the best performance against common countermeasures. Specifically, we obtain mean test accuracy of 33.01%, 18.91%, and 11.39% on CIFAR-10, CIFAR-100, and ImageNet subset, respectively, compared to the best performances of other methods at 39.20%, 23.47%, and 22.76%. Additionally, our method attained maximum test accuracies of 83.98%, 56.91%, and 42.14% on CIFAR-10, CIFAR-100, and ImageNet subset, respectively, compared to the best performances of other methods at 84.59%, 57.45%, and 44.24%. These results represent that our deep hiding scheme obtains a better robust generalization of unlearnable examples with high-level semantic features.

TABLE III TEST ACCURACY (%) OF CIFAR-10 AND CIFAR-100 ON FIVE ARCHITECTURES, INCLUDING RESNET-18 (R18), RESNET-50 (R50), VGG-19 (V19), DENSENET-121 (D121), AND VISION TRANSFORMER (VIT).NUMBERS IN BOLD AND UNDERLINE NUMBERS INDICATE THE BEST AND SECOND-BEST RESULTS, RESPECTIVELY.

<table><tr><td rowspan="2">Norm</td><td>Dataset</td><td colspan="5">CIFAR-10</td><td colspan="5">CIFAR-100</td></tr><tr><td>Model</td><td>R18</td><td>R50</td><td>V19</td><td>D121</td><td>ViT</td><td>R18</td><td>R50</td><td>V19</td><td>D121</td><td>ViT</td></tr><tr><td rowspan="6"> $L_{\infty}$ </td><td>EM [6]</td><td>10.10</td><td>10.00</td><td>10.82</td><td>12.56</td><td>11.88</td><td>2.84</td><td>3.88</td><td>9.23</td><td>64.87</td><td>7.65</td></tr><tr><td>REM [11]</td><td>30.40</td><td>25.10</td><td>24.54</td><td>30.28</td><td>32.36</td><td>7.13</td><td>7.45</td><td>5.26</td><td>12.47</td><td>6.91</td></tr><tr><td>TAP [8]</td><td>25.93</td><td>25.48</td><td>30.36</td><td>78.59</td><td>70.96</td><td>14.00</td><td>14.25</td><td>33.18</td><td>52.64</td><td>14.49</td></tr><tr><td>EntF [13]</td><td>91.50</td><td>91.83</td><td>88.17</td><td>83.30</td><td>69.23</td><td>72.55</td><td>73.19</td><td>65.68</td><td>60.85</td><td>49.43</td></tr><tr><td>DH(S)</td><td>12.93</td><td>10.64</td><td>12.69</td><td>20.52</td><td>65.70</td><td>7.81</td><td>11.51</td><td>14.55</td><td>14.36</td><td>12.48</td></tr><tr><td>DH(C)</td><td>10.18</td><td>9.98</td><td>12.33</td><td>10.00</td><td>9.99</td><td>1.25</td><td>1.16</td><td>0.93</td><td>1.11</td><td>5.54</td></tr><tr><td> $L_0$ </td><td>OPS [18]</td><td>17.46</td><td>16.73</td><td>19.12</td><td>18.40</td><td>28.09</td><td>11.69</td><td>10.90</td><td>5.67</td><td>10.38</td><td>17.23</td></tr><tr><td rowspan="2"> $L_2$ </td><td>LSP [41]</td><td>16.99</td><td>14.55</td><td>11.53</td><td>24.83</td><td>23.78</td><td>2.68</td><td>4.06</td><td>2.84</td><td>27.05</td><td>9.40</td></tr><tr><td>AR [40]</td><td>11.88</td><td>15.83</td><td>13.21</td><td>22.28</td><td>19.84</td><td>1.50</td><td>2.13</td><td>3.48</td><td>19.55</td><td>5.69</td></tr></table>

In addition to evaluating the performance of our proposed method against common countermeasures, we have also tested it against some more contemporary countermeasures. The results, as shown in Table II, indicate that our method effectively thwarts OP [52] and UEraser [49], maintaining exceptionally low test accuracies. Additionally, its performance is comparable to other methods against AVATAR [50]. We further assess the robustness of our method in an unsupervised contrastive learning setting, such as SimCLR [81], and find our results comparable to those of other methods. Additionally, it demonstrates that our method achieves the second-best performance compared to others. Notably, our method demonstrates strong performance against OP with test accuracies of 14.44% and 10.00%. This is attributed to our adaptive hiding manner, which is tailored to the unique features of each clean image. Such variability in perturbations enables our method to effectively resist OP. Overall, the mean and maximum results indicate that our method has better robustness performance compared to other methods.

E. Performance on different architectures and unlearnable percentages

1) Different model architectures: In real-world scenarios, the protector may not know the details of the target model. In such cases, it’s critical for unlearnable examples to be transferable. Hence, we evaluate the effectiveness of the proposed method across various deep learning architectures on CIFAR-10 and CIFAR-100 datasets. As shown in Table III, our approach in the class-wise setting consistently performs well across all five models. To further validate the effectiveness of our proposed methods, we conducted more comprehensive cross-validation. We verify the robustness of our generated unlearnable examples against various countermeasures under different architectures, and the results are shown in Table IV. It supports our claim that the proposed DH maintains its efficacy against different countermeasures across architectures.

2) Different unlearnable percentages: Consider a situation where it’s not feasible to protect all the data. This scenario is realistic since a practitioner who gains access to unlearnable examples might also obtain additional clean data from other avenues. Consequently, it’s common practice to evaluate unlearnable examples’ efficacy by training deep learning models with a random subset of unlearnable examples. To this end, we evaluate the performance of our proposed approach by using varying mixtures of clean images and unlearnable examples, the results are shown in Table V. Besides, we also conducted the transferability studies across architectures with limited unlearnable examples, and the results as shown in Table VI. The test accuracy decreases in a similar trend when we increase the percentage of the unlearnable examples.

## F. Ablation study

1) Effectiveness of Latent Feature Concentration (LFC) module: To understand the pivotal role of LFC in our approach, we conduct an ablation study focused on unlearnable performance. The results are shown in Table VII. Whether in the class-wise or sample-wise settings, the introduction of LFC leads to a discernible reduction in testing accuracy across three datasets. Notably, the CIFAR-10 dataset under the median filter exhibits a sharp decline, from 79.03% to 46.51%. These findings indicate that by focusing the latent feature of perturbations on intra-class characteristics, the unlearnability of the unlearnable examples is enhanced.

TABLE IV TEST ACCURACY (%) OF MODEL TRAIN ON UNLEARNABLE EXAMPLES FROM CIFAR-10 WITH FIVE ARCHITECTURES, INCLUDING RESNET-18 (R18), RESNET-50 (R50), VGG-19 (V19), AND DENSENET-121 (D121), AND VISION TRANSFORMER (VIT), AGAINST DATA AUGMENTATIONS, DATA PREPROCESSING, AND ADVERSARIAL TRAINING.

<table><tr><td></td><td>Model</td><td>Vanilla</td><td>Cutout</td><td>Cutmix</td><td>Mixup</td><td>MeanF</td><td>MedianF</td><td>BDR</td><td>Gray</td><td>GaussN</td><td>GaussF</td><td>JPEG10</td><td>JPEG50</td><td> $AT(L_{\infty})$ </td><td> $AT(\ell_2)$ </td><td>Mean</td></tr><tr><td rowspan="5">Ours(S)</td><td>R18</td><td>12.93</td><td>14.03</td><td>13.80</td><td>15.89</td><td>16.06</td><td>25.56</td><td>28.28</td><td>25.65</td><td>65.11</td><td>12.30</td><td>84.93</td><td>89.18</td><td>83.44</td><td>83.27</td><td>37.47</td></tr><tr><td>R50</td><td>10.64</td><td>13.26</td><td>13.81</td><td>22.40</td><td>17.36</td><td>57.97</td><td>72.39</td><td>22.92</td><td>82.43</td><td>11.06</td><td>84.69</td><td>89.38</td><td>85.26</td><td>84.59</td><td>47.72</td></tr><tr><td>V19</td><td>12.69</td><td>12.31</td><td>12.58</td><td>18.60</td><td>29.63</td><td>54.55</td><td>64.13</td><td>25.34</td><td>77.49</td><td>13.04</td><td>83.38</td><td>88.04</td><td>77.87</td><td>79.13</td><td>46.34</td></tr><tr><td>D121</td><td>20.52</td><td>27.57</td><td>28.29</td><td>25.56</td><td>73.59</td><td>81.89</td><td>75.34</td><td>55.51</td><td>72.16</td><td>21.91</td><td>82.13</td><td>85.03</td><td>75.66</td><td>76.78</td><td>57.29</td></tr><tr><td>ViT</td><td>65.70</td><td>69.02</td><td>59.43</td><td>65.30</td><td>65.87</td><td>69.27</td><td>67.42</td><td>44.95</td><td>66.08</td><td>75.97</td><td>67.16</td><td>73.16</td><td>43.57</td><td>50.49</td><td>63.86</td></tr><tr><td rowspan="5">Ours(C)</td><td>R18</td><td>10.08</td><td>10.00</td><td>10.81</td><td>10.05</td><td>10.20</td><td>15.58</td><td>17.31</td><td>10.32</td><td>30.23</td><td>10.01</td><td>83.98</td><td>78.96</td><td>82.21</td><td>82.34</td><td>33.01</td></tr><tr><td>R50</td><td>9.89</td><td>10.02</td><td>12.93</td><td>10.23</td><td>10.40</td><td>17.06</td><td>17.37</td><td>10.02</td><td>24.29</td><td>9.96</td><td>84.70</td><td>76.42</td><td>83.40</td><td>83.37</td><td>32.86</td></tr><tr><td>V19</td><td>12.33</td><td>9.61</td><td>10.48</td><td>10.39</td><td>10.75</td><td>15.71</td><td>13.68</td><td>11.14</td><td>27.99</td><td>10.72</td><td>83.91</td><td>78.93</td><td>79.67</td><td>79.53</td><td>32.48</td></tr><tr><td>D121</td><td>10.00</td><td>14.06</td><td>14.78</td><td>12.59</td><td>55.75</td><td>59.64</td><td>25.59</td><td>15.93</td><td>30.66</td><td>10.01</td><td>81.83</td><td>74.82</td><td>76.19</td><td>77.31</td><td>39.94</td></tr><tr><td>ViT</td><td>9.99</td><td>10.30</td><td>10.35</td><td>42.85</td><td>64.08</td><td>62.49</td><td>43.88</td><td>15.32</td><td>23.36</td><td>10.07</td><td>66.55</td><td>66.84</td><td>44.44</td><td>49.70</td><td>37.20</td></tr></table>

TABLE V TEST ACCURACY (%) OF CIFAR-10 ON THE MODELS TRAINED BY THE CLEAN DATA MIXED WITH DIFFERENT PERCENTAGES OF UNLEARNABLE EXAMPLES. NUMBERS IN BOLD AND UNDERLINE NUMBERS INDICATE THE BEST AND SECOND-BEST RESULTS, RESPECTIVELY.

<table><tr><td>Norm</td><td>Method</td><td>20%</td><td>40%</td><td>60%</td><td>80%</td></tr><tr><td rowspan="6"> $L_{\infty}$ </td><td>EM [6]</td><td>94.30</td><td>93.09</td><td>91.42</td><td>87.29</td></tr><tr><td>REM [11]</td><td>93.83</td><td>92.69</td><td>91.12</td><td>86.92</td></tr><tr><td>TAP [8]</td><td>93.82</td><td>92.78</td><td>91.96</td><td>88.49</td></tr><tr><td>EntF [13]</td><td> $\underline{93.40}$ </td><td> $\underline{91.71}$ </td><td>91.25</td><td>91.07</td></tr><tr><td>DH(S)</td><td>93.79</td><td>92.64</td><td>91.14</td><td>86.29</td></tr><tr><td>DH(C)</td><td> $\underline{93.33}$ </td><td> $\underline{92.36}$ </td><td> $\underline{90.09}$ </td><td> $\underline{83.99}$ </td></tr><tr><td> $L_{0}$ </td><td>OPS [18]</td><td>93.64</td><td>92.63</td><td> $\underline{90.05}$ </td><td> $\underline{84.42}$ </td></tr><tr><td rowspan="2"> $L_{2}$ </td><td>LSP [41]</td><td>93.50</td><td>92.47</td><td>90.21</td><td>84.81</td></tr><tr><td>AR [40]</td><td>94.07</td><td>92.66</td><td>90.34</td><td>85.18</td></tr></table>

TABLE VI TEST ACCURACY (%) OF CIFAR-10 ON THE DIFFERENT MODELS TRAINED BY THE CLEAN DATA MIXED WITH DIFFERENT PERCENTAGES OF UNLEARNABLE EXAMPLES.

<table><tr><td>Setting</td><td>Model</td><td>20%</td><td>40%</td><td>60%</td><td>80%</td></tr><tr><td rowspan="5">DH(S)</td><td>R18</td><td>93.79</td><td>92.64</td><td>91.14</td><td>86.27</td></tr><tr><td>R50</td><td>94.38</td><td>94.00</td><td>90.90</td><td>86.83</td></tr><tr><td>V19</td><td>92.88</td><td>92.52</td><td>88.24</td><td>83.13</td></tr><tr><td>D121</td><td>88.98</td><td>89.72</td><td>89.27</td><td>83.97</td></tr><tr><td>ViT</td><td>76.41</td><td>76.33</td><td>77.24</td><td>76.08</td></tr><tr><td rowspan="5">DH(C)</td><td>R18</td><td>93.33</td><td>92.36</td><td>90.09</td><td>83.99</td></tr><tr><td>R50</td><td>93.38</td><td>92.56</td><td>90.42</td><td>82.32</td></tr><tr><td>V19</td><td>91.70</td><td>90.83</td><td>88.32</td><td>80.35</td></tr><tr><td>D121</td><td>89.40</td><td>86.83</td><td>83.83</td><td>78.76</td></tr><tr><td>ViT</td><td>76.05</td><td>75.56</td><td>75.86</td><td>69.46</td></tr></table>

2) Effectiveness of Semantic Images Generation (SIG) module: To grasp the critical importance of SIG in our methodology, we undertake an ablation study on SIG including Text Prompts Clustering (TPC), and Stable Diffusion Model and ControlNet (SD+C). The results are shown in Table VII. In the class-wise setting, we find that the improvement of the generation module is marginal. However, in the sample-wise setting, the SIG can degrade the mean accuracy from 87.65% to 37.47%. When we disentangle the TPC and $\scriptstyle \mathrm { S D + C } ,$ we find that SD+C contributes the most and TPC contributes around 9% reduction. The ablation study shows that the SIG model in our proposed method plays an important role in samplewise unlearnable examples to make the robust unlearnable examples.

![](images/page_9_chart_10.jpg)

<details>
<summary>line</summary>

| Iteration | L_hide | L_total |
| --- | --- | --- |
| 0 | 0 | 0 |
| ~100 | 0 | ~0.115 |
| ~200 | 0 | ~0.04 |
| ~300 | 0 | ~0.02 |
| ~400 | 0 | ~0.015 |
| ~500 | 0 | ~0.012 |
| ~600 | 0 | ~0.01 |
| ~700 | 0 | ~0.018 |
| ~800 | 0 | ~0.01 |
| ~900 | 0 | ~0.012 |
| ~1000 | 0 | ~0.008 |
| ~2000 | 0 | ~0.005 |
| ~3700 | 0 | ~0.012 |
| ~4000 | 0 | ~0.003 |
| ~5300 | 0 | ~0.006 |
</details>

Fig. 6. Evaluating the dynamics of training loss: a comparative study using $\mathcal { L } _ { \mathrm { h i d e } }$ and $\mathcal { L } _ { \mathrm { t o t a l } }$ loss functions during the training process to observe the effectiveness of the revealing process in our proposed deep hiding model.

3) Effectiveness of different hyper-parameters’ settings: In our experiments, we have conducted some experiments of different hyper-parameters settings on CIFAR-10 and tabulated the results in class-wise setting, as shown in Table VIII. The optimal results were obtained when $\omega _ { 1 }$ and $\omega _ { 2 }$ were set to 1, and $\omega _ { 3 }$ was set to 0.0001. This configuration is effective because the loss associated with $\omega _ { 3 }$ is typically 1000 times larger than those of $\omega _ { 1 }$ and $\omega _ { 2 }$ . Setting $\omega _ { 3 }$ to 0.0001 helps balance the training process by ensuring that the scales of the losses remain consistent.

Besides, to observe the effectiveness of the revealing process in our proposed deep hiding model, we test the performance when $\omega _ { 1 }$ $\omega _ { 2 }$ , and $\omega _ { 3 }$ are set as 0, which means we just focus exclusively on the hiding loss, denoted as $\mathcal { L } _ { \mathrm { h i d e } } .$ The training loss trends for using only $\mathcal { L } _ { \mathrm { h i d e } }$ compared to our complete loss $\mathcal { L } _ { \mathrm { t o t a l } }$ are depicted in Fig. 6. The $\mathcal { L } _ { \mathrm { t o t a l } }$ exhibits a notable initial rise and subsequent decline, attributed to the optimization of the revealing loss. In contrast, when training with only the hiding loss, the training plot shows the loss remains 0. Since the minimal amount of information is hidden in the clean images during the first step, with the perturbation radius staying below the 8/255 threshold, fulfilling the optimization objectives without further optimization is needed. Furthermore, our evaluation of unlearnable examples generated by the model trained only with $\mathcal { L } _ { h i d e }$ reveals that the test accuracy is close to that of clean images, as shown in Table IX. This indicates that minimal information is hidden in clean images, leading to ineffective unlearnability. Based on the above experimental results and our analysis, we apply the designed total loss $\mathcal { L } _ { \mathrm { t o t a l } }$ in all experiments.

TABLE VII ABLATION STUDIES ON CIFAR-10 FOR DESIGNED LATENT FEATURE CONCENTRATION MODULE (LFC), AND SEMANTIC IMAGES GENERATION MODULE (SIG), INCLUDING TEXT PROMPTS CLUSTERING (TPC) AND STABLE DIFFUSION MODEL AND CONTROLNET (SD+C). NUMBERS IN BOLD INDICATE THE BEST AND SECOND-BEST RESULTS, RESPECTIVELY.

<table><tr><td rowspan="2"></td><td rowspan="2">LFC</td><td colspan="2">SIG</td><td rowspan="2">Vanilla</td><td rowspan="2">Cutout</td><td rowspan="2">Cutmix</td><td rowspan="2">Mixup</td><td rowspan="2">MeanF</td><td rowspan="2">MedianF</td><td rowspan="2">BDR</td><td rowspan="2">Gray</td><td rowspan="2">GaussN</td><td rowspan="2">GaussF</td><td rowspan="2">JPEG10</td><td rowspan="2">JPEG50</td><td rowspan="2"> $\mathrm{AT}(L_{\infty})$ </td><td rowspan="2"> $\mathrm{AT}(\ell_2)$ </td><td rowspan="2">Mean</td></tr><tr><td>TPC</td><td>SD+C</td></tr><tr><td rowspan="5">DH(S)</td><td>×</td><td>×</td><td>×</td><td>94.38</td><td>94.53</td><td>94.39</td><td>94.79</td><td>54.64</td><td>86.65</td><td>89.17</td><td>92.29</td><td>88.80</td><td>94.27</td><td>84.74</td><td>90.79</td><td>84.51</td><td>83.17</td><td>87.65</td></tr><tr><td>×</td><td>✓</td><td>✓</td><td>10.58</td><td>22.75</td><td>16.51</td><td>24.69</td><td>36.95</td><td>79.03</td><td>75.43</td><td>19.48</td><td>87.81</td><td>16.78</td><td>84.34</td><td>90.83</td><td>84.15</td><td>82.57</td><td>52.28</td></tr><tr><td>✓</td><td>×</td><td>✓</td><td>10.05</td><td>10.45</td><td>20.15</td><td>21.31</td><td>18.02</td><td>46.51</td><td>78.31</td><td>11.70</td><td>87.95</td><td>10.34</td><td>84.64</td><td>90.73</td><td>84.03</td><td>83.56</td><td>46.98</td></tr><tr><td>✓</td><td>×</td><td>×</td><td>94.50</td><td>94.48</td><td>94.20</td><td>94.90</td><td>56.45</td><td>84.13</td><td>89.57</td><td>94.24</td><td>89.04</td><td>94.30</td><td>85.06</td><td>90.76</td><td>84.24</td><td>83.26</td><td>87.65</td></tr><tr><td>✓</td><td>✓</td><td>✓</td><td>12.93</td><td>14.03</td><td>13.80</td><td>15.89</td><td>16.06</td><td>25.56</td><td>28.28</td><td>25.65</td><td>65.11</td><td>12.30</td><td>84.93</td><td>89.18</td><td>83.44</td><td>83.27</td><td>37.47</td></tr><tr><td rowspan="5">DH(C)</td><td>×</td><td>×</td><td>×</td><td>21.82</td><td>10.03</td><td>14.38</td><td>15.02</td><td>20.97</td><td>47.28</td><td>38.12</td><td>9.97</td><td>14.95</td><td>10.11</td><td>84.30</td><td>77.16</td><td>84.27</td><td>84.61</td><td>38.08</td></tr><tr><td>×</td><td>✓</td><td>✓</td><td>10.55</td><td>10.84</td><td>11.29</td><td>11.63</td><td>16.29</td><td>45.12</td><td>33.12</td><td>9.99</td><td>25.53</td><td>10.39</td><td>84.58</td><td>88.38</td><td>84.08</td><td>83.79</td><td>37.54</td></tr><tr><td>✓</td><td>×</td><td>✓</td><td>10.13</td><td>13.47</td><td>11.15</td><td>14.79</td><td>24.11</td><td>19.87</td><td>36.42</td><td>10.49</td><td>22.47</td><td>9.89</td><td>84.67</td><td>90.25</td><td>84.02</td><td>83.64</td><td>36.81</td></tr><tr><td>✓</td><td>×</td><td>×</td><td>9.99</td><td>10.01</td><td>10.41</td><td>9.55</td><td>26.59</td><td>19.74</td><td>24.61</td><td>10.02</td><td>13.10</td><td>9.99</td><td>84.83</td><td>86.24</td><td>84.06</td><td>84.34</td><td>34.53</td></tr><tr><td>✓</td><td>✓</td><td>✓</td><td>10.08</td><td>10.00</td><td>10.81</td><td>10.05</td><td>10.20</td><td>15.58</td><td>17.31</td><td>10.32</td><td>30.23</td><td>10.01</td><td>83.98</td><td>78.96</td><td>82.21</td><td>82.34</td><td>33.01</td></tr></table>

TABLE VIII ABLATION STUDIES ON CIFAR-10 FOR DIFFERENT SETTINGS ON PARAMETERS OF ω1, ω2, ω<sub>3</sub>. NUMBERS IN BOLD INDICATE THE BEST AND SECOND-BEST RESULTS, RESPECTIVELY.

<table><tr><td></td><td>Setting</td><td>Vanilla</td><td>Cutout</td><td>Cutmix</td><td>Mixup</td><td>MeanF</td><td>MedianF</td><td>BDR</td><td>Gray</td><td>GaussN</td><td>GaussF</td><td>JPEG10</td><td>JPEG50</td><td> $AT(L_{\infty})$ </td><td> $AT(\ell_2)$ </td><td>Mean</td></tr><tr><td rowspan="4"> $\omega_1$ </td><td>10</td><td>12.14</td><td>10.30</td><td>13.63</td><td>22.21</td><td>21.20</td><td>51.61</td><td>51.76</td><td>11.07</td><td>39.68</td><td>10.01</td><td>84.96</td><td>88.58</td><td>84.49</td><td>83.81</td><td>41.81</td></tr><tr><td>1(DH)</td><td>10.08</td><td>10.00</td><td>10.81</td><td>10.05</td><td>10.20</td><td>15.58</td><td>17.31</td><td>10.32</td><td>30.23</td><td>10.01</td><td>83.98</td><td>78.96</td><td>82.21</td><td>82.34</td><td>33.01</td></tr><tr><td> $10^{-1}$ </td><td>9.99</td><td>9.95</td><td>10.42</td><td>12.51</td><td>27.88</td><td>24.30</td><td>45.82</td><td>11.74</td><td>36.01</td><td>9.97</td><td>84.47</td><td>79.92</td><td>83.08</td><td>83.82</td><td>37.85</td></tr><tr><td> $10^{-2}$ </td><td>10.29</td><td>10.00</td><td>10.00</td><td>10.55</td><td>9.98</td><td>12.97</td><td>24.59</td><td>10.00</td><td>25.05</td><td>10.00</td><td>84.98</td><td>90.37</td><td>84.11</td><td>83.35</td><td>34.02</td></tr><tr><td rowspan="4"> $\omega_2$ </td><td>10</td><td>9.97</td><td>10.02</td><td>10.09</td><td>11.33</td><td>11.17</td><td>11.27</td><td>31.21</td><td>10.55</td><td>24.64</td><td>10.20</td><td>84.68</td><td>85.82</td><td>84.51</td><td>83.61</td><td>34.65</td></tr><tr><td>1(DH)</td><td>10.08</td><td>10.00</td><td>10.81</td><td>10.05</td><td>10.20</td><td>15.58</td><td>17.31</td><td>10.32</td><td>30.23</td><td>10.01</td><td>83.98</td><td>78.96</td><td>82.21</td><td>82.34</td><td>33.01</td></tr><tr><td> $10^{-1}$ </td><td>10.45</td><td>12.71</td><td>10.94</td><td>11.57</td><td>19.92</td><td>17.37</td><td>42.37</td><td>10.44</td><td>31.12</td><td>10.95</td><td>85.07</td><td>86.74</td><td>83.73</td><td>83.65</td><td>36.93</td></tr><tr><td> $10^{-2}$ </td><td>10.00</td><td>10.02</td><td>9.87</td><td>10.02</td><td>10.62</td><td>29.80</td><td>36.62</td><td>10.00</td><td>23.94</td><td>10.00</td><td>84.78</td><td>90.50</td><td>84.42</td><td>84.24</td><td>36.95</td></tr><tr><td rowspan="4"> $\omega_3$ </td><td> $10^{-1}$ </td><td>10.30</td><td>11.38</td><td>11.13</td><td>10.23</td><td>15.27</td><td>14.69</td><td>37.59</td><td>10.04</td><td>29.13</td><td>10.84</td><td>84.91</td><td>87.93</td><td>84.41</td><td>83.41</td><td>35.80</td></tr><tr><td> $10^{-2}$ </td><td>10.09</td><td>9.85</td><td>10.76</td><td>11.03</td><td>10.51</td><td>12.42</td><td>33.74</td><td>11.47</td><td>40.79</td><td>9.33</td><td>85.18</td><td>86.68</td><td>84.29</td><td>82.85</td><td>35.64</td></tr><tr><td> $10^{-3}$ </td><td>10.57</td><td>9.84</td><td>10.46</td><td>14.06</td><td>22.34</td><td>44.82</td><td>43.90</td><td>10.00</td><td>37.76</td><td>6.18</td><td>84.26</td><td>80.49</td><td>82.92</td><td>83.07</td><td>38.62</td></tr><tr><td> $10^{-4}(DH)$ </td><td>10.08</td><td>10.00</td><td>10.81</td><td>10.05</td><td>10.20</td><td>15.58</td><td>17.31</td><td>10.32</td><td>30.23</td><td>10.01</td><td>83.98</td><td>78.96</td><td>82.21</td><td>82.34</td><td>33.01</td></tr></table>

TABLE IX EVALUATION ON THE DIFFERENT HIDING MODEL TRAINED BY SOLELY CONTROLLING THE HIDING LOSS $( \mathcal { L } _ { \mathrm { H I D E } } )$ AND USING OUR DESIGNED LOSS $( \mathcal { L } _ { \mathrm { T O T A L } } )$ . THE TEST ACCURACY (%) IS EVALUATED ON CIFAR-10 IN THE CLASS-WISE SETTING.

| Method | Vanilla | Cutout | Cutmix | Mixup | MeanF | MedianF | BDR | Gray | GaussN | GaussF | JPEG10 | JPEG50 | $AT(L_{\infty})$ | $AT(\ell_2)$ | Mean |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Clean | 94.59 | 95.00 | 94.77 | 94.96 | 49.70 | 86.64 | 89.07 | 92.80 | 88.71 | 94.54 | 85.22 | 90.89 | 84.19 | 83.54 | 87.07 |
| Only $\mathcal{L}_{\text{hide}}$ | 94.69 | 94.95 | 94.48 | 94.86 | 56.45 | 86.10 | 89.06 | 92.79 | 88.49 | 94.44 | 85.24 | 90.75 | 84.07 | 83.46 | 87.89 |
| DH | 10.08 | 10.00 | 10.81 | 10.05 | 10.20 | 15.58 | 17.31 | 10.32 | 30.23 | 10.01 | 83.98 | 78.96 | 82.21 | 82.34 | 33.01 |

TABLE X EVALUATION OF THE EXISTING IMAGE HIDING METHODS IN OTHER ATTACKS. THE TEST ACCURACY (%) IS EVALUATED ON CIFAR-10.

| Method | Vanilla | Cutout | Cutmix | Mixup | MeanF | MedianF | BDR | Gray | GaussN | GaussF | JPEG10 | JPEG50 | $AT(L_{\infty})$ | $AT(\ell_2)$ | Mean |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Clean | 94.59 | 95.00 | 94.77 | 94.96 | 49.70 | 86.64 | 89.07 | 92.80 | 88.71 | 94.54 | 85.22 | 90.89 | 84.19 | 83.54 | 87.47 |
| Poison Ink [61] | 93.58 | 93.55 | 93.16 | 94.21 | 36.42 | 82.92 | 89.12 | 92.19 | 89.12 | 93.30 | 84.35 | 90.07 | 87.10 | 84.52 | 85.97 |
| AdvINN [62] | 85.21 | 79.29 | 81.34 | 80.23 | 49.38 | 82.25 | 77.39 | 87.46 | 84.38 | 88.66 | 82.01 | 87.68 | 84.91 | 83.27 | 80.96 |
| DH | 10.08 | 10.00 | 10.81 | 10.05 | 10.20 | 15.58 | 17.31 | 10.32 | 30.23 | 10.01 | 83.98 | 78.96 | 82.21 | 82.34 | 33.01 |

## G. Evaluation of existing image hiding methods in other attacks

Existing image hiding methods have been applied in other forms of attacks, such as Poison Ink [61] for backdoor attack, and AdvINN [62] for adversarial attack. In this part, we investigate whether these methods could also generate unlearnable examples. The experimental results are shown in Table X. Based on the experimental results, the vanilla test accuracies of Poison Ink and AdvINN are 93.58 and 85.21, respectively. Such results indicate that they do not suit the goal of unlearnable examples generation. The adversarial noise added by Poison Ink is based on the contour features of the clean image, which are strongly correlated with the clean image itself, making it difficult to achieve unlearnability. AdvINN adaptively generates a target image that starts with a constant image (e.g., all pixels set to 0.5), and the resulting pattern remains strongly linked to clean images. The purpose of these attacks differs from that of creating unlearnable examples. Although hiding technology is utilized in this field to create imperceptible samples, it cannot be directly employed to achieve unlearnability.

TABLE XI TEST ACCURACY (%) OF MODELS TRAINED ON UNLEARNABLE EXAMPLES WITH A SOFT RESTRICTION SETTING AGAINST DATA AUGMENTATIONS, DATA PREPROCESSING, AND ADVERSARIAL TRAINING.

<table><tr><td></td><td>Method</td><td>Vanilla</td><td>Cutout</td><td>Cutmix</td><td>Mixup</td><td>MeanF</td><td>MedianF</td><td>BDR</td><td>Gray</td><td>GaussN</td><td>GaussF</td><td>JPEG10</td><td>JPEG50</td><td> $AT(L_{\infty})$ </td><td> $AT(\ell_2)$ </td><td>Mean</td><td>PSNR</td></tr><tr><td rowspan="4">CIFAR-10</td><td>DH(S)</td><td>12.93</td><td>14.03</td><td>13.80</td><td>15.89</td><td>16.06</td><td>25.56</td><td>28.28</td><td>25.65</td><td>65.11</td><td>12.30</td><td>84.93</td><td>89.18</td><td>83.44</td><td>83.27</td><td>37.47</td><td>33.52</td></tr><tr><td>DH&#x27;(S)</td><td>15.36</td><td>10.79</td><td>10.00</td><td>14.72</td><td>17.68</td><td>17.00</td><td>21.12</td><td>17.61</td><td>22.78</td><td>11.16</td><td>80.41</td><td>81.03</td><td>38.31</td><td>68.03</td><td>30.43</td><td>31.63</td></tr><tr><td>DH(C)</td><td>10.08</td><td>10.00</td><td>10.81</td><td>10.05</td><td>10.20</td><td>15.58</td><td>17.31</td><td>10.32</td><td>30.23</td><td>10.01</td><td>83.98</td><td>78.96</td><td>82.21</td><td>82.34</td><td>33.01</td><td>33.55</td></tr><tr><td>DH&#x27;(C)</td><td>10.00</td><td>10.00</td><td>11.25</td><td>10.02</td><td>10.59</td><td>10.04</td><td>13.53</td><td>10.00</td><td>10.00</td><td>10.00</td><td>72.97</td><td>23.62</td><td>10.00</td><td>18.80</td><td>16.49</td><td>31.76</td></tr><tr><td rowspan="4">CIFAR-100</td><td>DH(S)</td><td>7.81</td><td>4.80</td><td>10.15</td><td>10.27</td><td>10.14</td><td>22.23</td><td>38.13</td><td>15.50</td><td>52.43</td><td>7.97</td><td>56.01</td><td>65.72</td><td>56.48</td><td>56.38</td><td>29.57</td><td>33.71</td></tr><tr><td>DH&#x27;(S)</td><td>4.79</td><td>4.13</td><td>5.39</td><td>4.72</td><td>6.22</td><td>10.21</td><td>12.12</td><td>3.72</td><td>19.85</td><td>3.61</td><td>49.50</td><td>34.86</td><td>41.12</td><td>37.56</td><td>16.99</td><td>30.98</td></tr><tr><td>DH(C)</td><td>1.22</td><td>1.01</td><td>1.22</td><td>1.09</td><td>1.51</td><td>2.72</td><td>12.08</td><td>0.96</td><td>19.86</td><td>1.01</td><td>55.07</td><td>53.83</td><td>56.91</td><td>56.34</td><td>18.91</td><td>33.69</td></tr><tr><td>DH&#x27;(C)</td><td>1.47</td><td>1.03</td><td>1.06</td><td>1.47</td><td>1.04</td><td>1.45</td><td>1.72</td><td>1.38</td><td>1.08</td><td>1.00</td><td>44.58</td><td>25.45</td><td>1.39</td><td>17.14</td><td>7.23</td><td>31.26</td></tr></table>

## H. Performance in a soft restriction setting

In this section, we investigate the balance between imperceptibility and protection performance by using a soft restriction strategy. Specifically, we utilize our proposed method to generate unlearnable examples without applying clipping operations to restrict the perturbation within 8/255. It is designed upon the principle of invisibility, which focuses on perturbing regions less discernible to humans. Similar strategies have been employed in [82], [83] to prioritize imperceptibility instead of bound restriction. The experimental results are presented in Table XI. It is important to note that even without the bound restriction for our generated unlearnable examples, the invisibility of our examples can be preserved due to the adaptive hiding manner employed by our hiding model. This manner allows for the concealment of perturbations of varying scales across different regions. In a soft restriction setting, our method significantly enhances the robustness of our unlearnable examples, especially showing a good performance against AT. This phenomenon indicates the general robustness of our method, underscoring its potential practicality in real-world scenarios.

## V. CONCLUSION

In this paper, we present a novel Deep Hiding scheme tailored for the generation of robust unlearnable examples. By embedding clean images with semantically rich high-level attributes, we ensure that the generated unlearnable examples effectively derail the learning processes of unauthorized deep learning models. Additionally, our uniquely conceived Latent

Feature Concentration (LFC) module further enhances the effectiveness of unlearnable examples by regularizing the intraclass variance of perturbations. To guarantee the robustness of unlearnable examples, we introduce the Semantic Images Generation (SIG) module to generate hidden semantic images by maintaining semantic feature consistency within each class. The extensive experimental results demonstrate that our proposed method achieves outstanding unlearnability performance and superior robustness against various countermeasures.

## REFERENCES

[1] D. Mahajan, R. Girshick, V. Ramanathan, K. He, M. Paluri, Y. Li, A. Bharambe, and L. Van Der Maaten, “Exploring the limits of weakly supervised pretraining,” in Proceedings of the European conference on computer vision (ECCV), 2018, pp. 181–196.
[2] V. U. Prabhu and A. Birhane, “Large image datasets: A pyrrhic win for computer vision?” arXiv preprint arXiv:2006.16923, 2020.
[3] H. Drachsler and W. Greller, “Privacy and analytics: it’s a delicate issue a checklist for trusted learning analytics,” in Proceedings of the sixth international conference on learning analytics & knowledge, 2016, pp. 89–98.
[4] J. Shen, X. Zhu, and D. Ma, “Tensorclog: An imperceptible poisoning attack on deep neural network applications,” IEEE Access, vol. 7, pp. 41 498–41 506, 2019.
[5] J. Feng, Q.-Z. Cai, and Z.-H. Zhou, “Learning to confuse: generating training time adversarial data with auto-encoder,” Advances in Neural Information Processing Systems, vol. 32, 2019.
[6] H. Huang, X. Ma, S. M. Erfani, J. Bailey, and Y. Wang, “Unlearnable examples: Making personal data unexploitable,” arXiv preprint arXiv:2101.04898, 2021.
[7] L. Tao, L. Feng, J. Yi, S.-J. Huang, and S. Chen, “Better safe than sorry: Preventing delusive adversaries with adversarial training,” Advances in Neural Information Processing Systems, vol. 34, pp. 16 209–16 225, 2021.
[8] L. Fowl, M. Goldblum, P.-y. Chiang, J. Geiping, W. Czaja, and T. Goldstein, “Adversarial examples make strong poisons,” Advances in Neural Information Processing Systems, vol. 34, pp. 30 339–30 351, 2021.
[9] X. Lin, Y. Yu, S. Xia, J. Jiang, H. Wang, Z. Yu, Y. Liu, Y. Fu, S. Wang, W. Tang et al., “Safeguarding medical image segmentation datasets against unauthorized training via contour-and texture-aware perturbations,” arXiv preprint arXiv:2403.14250, 2024.
[10] E. Radiya-Dixit, S. Hong, N. Carlini, and F. Tramer, “Data poisoning won’t save you from facial recognition,” in International Conference on Learning Representations, 2021.
[11] S. Fu, F. He, Y. Liu, L. Shen, and D. Tao, “Robust unlearnable examples: Protecting data privacy against adversarial learning,” in International Conference on Learning Representations, 2021.
[12] Z. Wang, Y. Wang, and Y. Wang, “Fooling adversarial training with inducing noise,” arXiv preprint arXiv:2111.10130, 2021.
[13] R. Wen, Z. Zhao, Z. Liu, M. Backes, T. Wang, and Y. Zhang, “Is adversarial training really a silver bullet for mitigating data poisoning?” in The Eleventh International Conference on Learning Representations, 2022.
[14] A. Madry, A. Makelov, L. Schmidt, D. Tsipras, and A. Vladu, “Towards deep learning models resistant to adversarial attacks,” arXiv preprint arXiv:1706.06083, 2017.
[15] Y. Yu, W. Yang, Y.-P. Tan, and A. C. Kot, “Towards robust rain removal against adversarial attacks: A comprehensive benchmark analysis and beyond,” in Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition, 2022, pp. 6013–6022.
[16] S. Xia, Y. Yu, X. Jiang, and H. Ding, “Mitigating the curse of dimensionality for certified robustness via dual randomized smoothing,” in The Twelfth International Conference on Learning Representations, 2024. [Online]. Available: [https://openreview.net/forum?id=C1sQBG6Sqp](https://openreview.net/forum?id=C1sQBG6Sqp)
[17] Z. Liu, Z. Zhao, and M. A. Larson, “Image shortcut squeezing: Countering perturbative availability poisons with compression,” in International Conference on Machine Learning, ICML 2023, 23-29 July 2023, Honolulu, Hawaii, USA, ser. Proceedings of Machine Learning Research, vol. 202, 2023, pp. 22 473–22 487.
[18] S. Wu, S. Chen, C. Xie, and X. Huang, “One-pixel shortcut: on the learning preference of deep neural networks,” arXiv preprint arXiv:2205.12141, 2022.
[19] R. Geirhos, P. Rubisch, C. Michaelis, M. Bethge, F. A. Wichmann, and W. Brendel, “Imagenet-trained cnns are biased towards texture; increasing shape bias improves accuracy and robustness,” arXiv preprint arXiv:1811.12231, 2018.
[20] M. D. Zeiler and R. Fergus, “Visualizing and understanding convolutional networks,” in Computer Vision–ECCV 2014: 13th European Conference, Zurich, Switzerland, September 6-12, 2014, Proceedings, Part I 13. Springer, 2014, pp. 818–833.
[21] K. He, X. Zhang, S. Ren, and J. Sun, “Deep residual learning for image recognition,” in Proceedings of the IEEE conference on computer vision and pattern recognition, 2016, pp. 770–778.
[22] Z. Zhou, X. Dong, R. Meng, M. Wang, H. Yan, K. Yu, and K.-K. R. Choo, “Generative steganography via auto-generation of semantic object contours,” IEEE Transactions on Information Forensics and Security, 2023.
[23] X. Li, Y. Grandvalet, F. Davoine, J. Cheng, Y. Cui, H. Zhang, S. Belongie, Y.-H. Tsai, and M.-H. Yang, “Transfer learning in computer vision tasks: Remember where you come from,” Image and Vision Computing, vol. 93, p. 103853, 2020.
[24] S. Baluja, “Hiding images in plain sight: Deep steganography,” Advances in neural information processing systems, vol. 30, 2017.
[25] C. Yu, “Attention based data hiding with generative adversarial networks,” in Proceedings of the AAAI conference on artificial intelligence, vol. 34, no. 01, 2020, pp. 1120–1128.
[26] J. Jing, X. Deng, M. Xu, J. Wang, and Z. Guan, “Hinet: deep image hiding by invertible network,” in Proceedings of the IEEE/CVF international conference on computer vision, 2021, pp. 4733–4742.
[27] R. Zhang, S. Dong, and J. Liu, “Invisible steganography via generative adversarial networks,” Multimedia tools and applications, vol. 78, pp. 8559–8575, 2019.
[28] W. Tang, B. Li, M. Barni, J. Li, and J. Huang, “An automatic cost learning framework for image steganography using deep reinforcement learning,” IEEE Transactions on Information Forensics and Security, vol. 16, pp. 952–967, 2020.
[29] W. Pan, Y. Yin, X. Wang, Y. Jing, and M. Song, “Seek-and-hide: adversarial steganography via deep reinforcement learning,” IEEE Transactions on Pattern Analysis and Machine Intelligence, vol. 44, no. 11, pp. 7871–7884, 2021.
[30] Q. Cui, W. Tang, Z. Zhou, R. Meng, G. Nan, and Y.-Q. Shi, “Meta security metric learning for secure deep image hiding,” IEEE Transactions on Dependable and Secure Computing, 2024.
[31] X. Hu, Z. Fu, X. Zhang, and Y. Chen, “Invisible and steganalysisresistant deep image hiding based on one-way adversarial invertible networks,” IEEE Transactions on Circuits and Systems for Video Technology, 2023.
[32] Z. Guan, J. Jing, X. Deng, M. Xu, L. Jiang, Z. Zhang, and Y. Li, “Deepmih: Deep invertible network for multiple image hiding,” IEEE Transactions on Pattern Analysis and Machine Intelligence, vol. 45, no. 1, pp. 372–390, 2022.
[33] M. Xiao, S. Zheng, C. Liu, Z. Lin, and T.-Y. Liu, “Invertible rescaling network and its extensions,” International Journal of Computer Vision, vol. 131, no. 1, pp. 134–159, 2023.
[34] R. Meng, Z. Zhou, Q. Cui, K.-Y. Lam, and A. Kot, “Traceable and authenticable image tagging for fake news detection,” arXiv preprint arXiv:2211.10923, 2022.
[35] ——, “Traceable and authenticable image tagging for fake news detection,” arXiv preprint arXiv:2211.10923, 2022.
[36] K. He, X. Zhang, S. Ren, and J. Sun, “Deep residual learning for image recognition,” in Proceedings of the IEEE conference on computer vision and pattern recognition, 2016, pp. 770–778.
[37] A. Athalye, L. Engstrom, A. Ilyas, and K. Kwok, “Synthesizing robust adversarial examples,” in International conference on machine learning. PMLR, 2018, pp. 284–293.
[38] J. Ren, H. Xu, Y. Wan, X. Ma, L. Sun, and J. Tang, “Transferable unlearnable examples,” arXiv preprint arXiv:2210.10114, 2022.
[39] J. Zhang, X. Ma, Q. Yi, J. Sang, Y.-G. Jiang, Y. Wang, and C. Xu, “Unlearnable clusters: Towards label-agnostic unlearnable examples,” in Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition, 2023, pp. 3984–3993.
[40] P. Sandoval-Segura, V. Singla, J. Geiping, M. Goldblum, T. Goldstein, and D. Jacobs, “Autoregressive perturbations for data poisoning,” Advances in Neural Information Processing Systems, vol. 35, pp. 27 374–27 386, 2022.
[41] D. Yu, H. Zhang, W. Chen, J. Yin, and T.-Y. Liu, “Availability attacks create shortcuts,” in Proceedings of the 28th ACM SIGKDD Conference on Knowledge Discovery and Data Mining, 2022, pp. 2367–2376.
[42] J. Zhang, S. Peng, Y. Gao, Z. Zhang, and Q. Hong, “Apmsa: adversarial perturbation against model stealing attacks,” IEEE Transactions on Information Forensics and Security, vol. 18, pp. 1667–1679, 2023.
[43] Y. Xu, C. Mou, Y. Hu, J. Xie, and J. Zhang, “Robust invertible image steganography,” in Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition, 2022, pp. 7875–7884.
[44] Y. Zhu, Y. Chen, X. Li, R. Zhang, X. Tian, B. Zheng, and Y. Chen, “Information-containing adversarial perturbation for combating facial manipulation systems,” IEEE Transactions on Information Forensics and Security, vol. 18, pp. 2046–2059, 2023.
[45] H. J. Lee and Y. M. Ro, “Robust proxy: Improving adversarial robustness by robust proxy learning,” IEEE Transactions on Information Forensics and Security, 2023.
[46] T. DeVries and G. W. Taylor, “Improved regularization of convolutional neural networks with cutout,” arXiv preprint arXiv:1708.04552, 2017.
[47] S. Yun, D. Han, S. J. Oh, S. Chun, J. Choe, and Y. Yoo, “Cutmix: Regularization strategy to train strong classifiers with localizable features,” in Proceedings of the IEEE/CVF international conference on computer vision, 2019, pp. 6023–6032.
[48] H. Zhang, M. Cisse, Y. N. Dauphin, and D. Lopez-Paz, “mixup: Beyond empirical risk minimization,” arXiv preprint arXiv:1710.09412, 2017.
[49] T. Qin, X. Gao, J. Zhao, K. Ye, and C.-Z. Xu, “Learning the unlearnable: Adversarial augmentations suppress unlearnable example attacks,” arXiv preprint arXiv:2303.15127, 2023.
[50] H. M. Dolatabadi, S. Erfani, and C. Leckie, “The devil’s advocate: Shattering the illusion of unexploitable data using diffusion models,” arXiv preprint arXiv:2303.08500, 2023.
[51] W. Nie, B. Guo, Y. Huang, C. Xiao, A. Vahdat, and A. Anandkumar, “Diffusion models for adversarial purification,” arXiv preprint arXiv:2205.07460, 2022.
[52] P. Sandoval-Segura, V. Singla, J. Geiping, M. Goldblum, and T. Goldstein, “What can we learn from unlearnable datasets?” Advances in Neural Information Processing Systems, vol. 36, 2024.
[53] Y. Yu, Y. Wang, S. Xia, W. Yang, S. Lu, Y.-P. Tan, and A. C. Kot, “Purify unlearnable examples via rate-constrained variational autoencoders,” in International Conference on Machine Learning, ICML 2024, ser. Proceedings of Machine Learning Research, 2024.
[54] R. Zhao, T. Liu, J. Xiao, D. P. Lun, and K.-M. Lam, “Invertible image decolorization,” IEEE Transactions on Image Processing, vol. 30, pp. 6081–6095, 2021.
[55] J.-J. Huang and P. L. Dragotti, “Winnet: Wavelet-inspired invertible network for image denoising,” IEEE Transactions on Image Processing, vol. 31, pp. 4377–4392, 2022.
[56] J. Jing, X. Deng, M. Xu, J. Wang, and Z. Guan, “Hinet: deep image hiding by invertible network,” in Proceedings of the IEEE/CVF international conference on computer vision, 2021, pp. 4733–4742.
[57] Z. Guan, J. Jing, X. Deng, M. Xu, L. Jiang, Z. Zhang, and Y. Li, “Deepmih: Deep invertible network for multiple image hiding,” IEEE Transactions on Pattern Analysis and Machine Intelligence, vol. 45, no. 1, pp. 372–390, 2022.
[58] F. Li, Y. Sheng, X. Zhang, and C. Qin, “iscmis: Spatial-channel attention based deep invertible network for multi-image steganography,” IEEE Transactions on Multimedia, 2023.
[59] S.-P. Lu, R. Wang, T. Zhong, and P. L. Rosin, “Large-capacity image steganography based on invertible neural networks,” in Proceedings of the IEEE/CVF conference on computer vision and pattern recognition, 2021, pp. 10 816–10 825.
[60] H. Zhong, C. Liao, A. C. Squicciarini, S. Zhu, and D. Miller, “Backdoor embedding in convolutional neural network models via invisible perturbation,” in Proceedings of the Tenth ACM Conference on Data and Application Security and Privacy, 2020, pp. 97–108.
[61] J. Zhang, C. Dongdong, Q. Huang, J. Liao, W. Zhang, H. Feng, G. Hua, and N. Yu, “Poison ink: Robust and invisible backdoor attack,” IEEE Transactions on Image Processing, vol. 31, pp. 5691–5705, 2022.
[62] Z. Chen, Z. Wang, J.-J. Huang, W. Zhao, X. Liu, and D. Guan, “Imperceptible adversarial attack via invertible neural networks,” in Proceedings of the AAAI Conference on Artificial Intelligence, vol. 37, no. 1, 2023, pp. 414–424.
[63] Y. Yu, Y. Wang, W. Yang, S. Lu, Y.-P. Tan, and A. C. Kot, “Backdoor attacks against deep image compression via adaptive frequency trigger,” in Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition, 2023, pp. 12 250–12 259.
[64] R. Rombach, A. Blattmann, D. Lorenz, P. Esser, and B. Ommer, “Highresolution image synthesis with latent diffusion models,” in Proceedings of the IEEE/CVF conference on computer vision and pattern recognition, 2022, pp. 10 684–10 695.
[65] L. Zhang and M. Agrawala, “Adding conditional control to text-to-image diffusion models,” arXiv preprint arXiv:2302.05543, 2023.
[66] R. Gandikota, J. Materzynska, J. Fiotto-Kaufman, and D. Bau, “Erasing ´ concepts from diffusion models,” in Proceedings of the 2023 IEEE International Conference on Computer Vision, 2023.
[67] T.-Y. Lin, M. Maire, S. Belongie, J. Hays, P. Perona, D. Ramanan, P. Dollar, and C. L. Zitnick, “Microsoft coco: Common objects in ´ context,” in Computer Vision–ECCV 2014: 13th European Conference, Zurich, Switzerland, September 6-12, 2014, Proceedings, Part V 13. Springer, 2014, pp. 740–755.
[68] D. Arthur and S. Vassilvitskii, “K-means++ the advantages of careful seeding,” in Proceedings of the eighteenth annual ACM-SIAM symposium on Discrete algorithms, 2007, pp. 1027–1035.
[69] A. Radford, J. W. Kim, C. Hallacy, A. Ramesh, G. Goh, S. Agarwal, G. Sastry, A. Askell, P. Mishkin, J. Clark et al., “Learning transferable visual models from natural language supervision,” in International conference on machine learning. PMLR, 2021, pp. 8748–8763.
[70] L. Zhang, A. Rao, and M. Agrawala, “Adding conditional control to text-to-image diffusion models,” in Proceedings of the IEEE/CVF International Conference on Computer Vision, 2023, pp. 3836–3847.
[71] R. Rombach, A. Blattmann, D. Lorenz, P. Esser, and B. Ommer, “Highresolution image synthesis with latent diffusion models,” in Proceedings of the IEEE/CVF conference on computer vision and pattern recognition, 2022, pp. 10 684–10 695.
[72] R. Geirhos, J.-H. Jacobsen, C. Michaelis, R. Zemel, W. Brendel, M. Bethge, and F. A. Wichmann, “Shortcut learning in deep neural networks,” Nature Machine Intelligence, vol. 2, no. 11, pp. 665–673, 2020.
[73] G. K. Zipf, Human behavior and the principle of least effort: An introduction to human ecology. Ravenio Books, 2016.
[74] R. R. Selvaraju, M. Cogswell, A. Das, R. Vedantam, D. Parikh, and D. Batra, “Grad-cam: Visual explanations from deep networks via gradient-based localization,” in Proceedings of the IEEE international conference on computer vision, 2017, pp. 618–626.
[75] A. Krizhevsky, G. Hinton et al., “Learning multiple layers of features from tiny images,” 2009.
[76] J. Deng, W. Dong, R. Socher, L.-J. Li, K. Li, and L. Fei-Fei, “Imagenet: A large-scale hierarchical image database,” in 2009 IEEE conference on computer vision and pattern recognition. Ieee, 2009, pp. 248–255.
[77] K. Simonyan and A. Zisserman, “Very deep convolutional networks for large-scale image recognition,” arXiv preprint arXiv:1409.1556, 2014.
[78] G. Huang, Z. Liu, L. Van Der Maaten, and K. Q. Weinberger, “Densely connected convolutional networks,” in Proceedings of the IEEE conference on computer vision and pattern recognition, 2017, pp. 4700–4708.
[79] A. Dosovitskiy, L. Beyer, A. Kolesnikov, D. Weissenborn, X. Zhai, T. Unterthiner, M. Dehghani, M. Minderer, G. Heigold, S. Gelly et al., “An image is worth 16x16 words: Transformers for image recognition at scale,” arXiv preprint arXiv:2010.11929, 2020.
[80] D. P. Kingma and J. Ba, “Adam: A method for stochastic optimization,” arXiv preprint arXiv:1412.6980, 2014.
[81] H. He, K. Zha, and D. Katabi, “Indiscriminate poisoning attacks on unsupervised contrastive learning,” arXiv preprint arXiv:2202.11202, 2022.
[82] V. S. Sadasivan, M. Soltanolkotabi, and S. Feizi, “Cuda: Convolutionbased unlearnable datasets,” in Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition (CVPR), June 2023, pp. 3862–3871.
[83] N. Ahn, W. Ahn, K. Yoo, D. Kim, and S.-H. Nam, “Imperceptible protection against style imitation from diffusion models,” arXiv preprint arXiv:2403.19254, 2024.

![](images/page_13_image_26.jpg)

**Ruohan Meng** received her B.E. degree in Software Engineering and Ph.D. degree in Information Security from Nanjing University of Information Science and Technology, China in 2018 and 2023. She is currently a Research Fellow with the School of Electrical and Electronic Engineering, Nanyang Technological University, Singapore. Her research interests include information hiding, covert communication, and privacy protection.

**Chenyu Yi** received the BEng degree from Nanyang Technological University. He is currently pursuing the Ph.D. degree at Nanyang Technological University. His research interests include robust and trustworthy deep learning and visual understanding.

**Yi Yu** received the B.S. degree from the Department of Automation, Tsinghua University, China and the M.S, degree from the Department of Electrical and Computer Engineering, University of California San Diego, United States, in 2019 and 2021, respectively. He is currently working toward the PhD degree in the ROSE Lab, Interdisciplinary Graduate Programme, Nanyang Technology University, Singapore. His research interests include adversarial robustness and low-level vision.

**Siyuan Yang** received the BEng degree from Harbin Institute of Technology and the MSc degree from Nanyang Technological University. He is currently pursuing the Ph.D. degree with the Interdisciplinary Graduate Programme, Nanyang Technological University. His research interests include computer vision, action recognition, and human pose estimation.

**Bingquan Shen** is currently a Principal Member of Technical Staff in DSO National Laboratories, and an Adjunct Assistant Professor in the Engineering Science Program, National University of Singapore. He received his Honours and PhD in Mechanical Engineering (Control and Mechatronics) from the National University of Singapore in 2009 and 2014 respectively. His research interests lie in intelligent systems, multimodal data understanding/generation, and trustworthy AI.

**Alex C. Kot** (Life Fellow, IEEE) has been with the Nanyang Technological University, Singapore since 1991. He was Head of the Division of Information Engineering and Vice Dean Research at the School of Electrical and Electronic Engineering. Subsequently, he served as Associate Dean for College of Engineering for eight years. He is currently Professor and Director of Rapid-Rich Object SEarch (ROSE) Lab and NTU-PKU Joint Research Institute. He has published extensively in the areas of signal processing, biometrics, image forensics and security, d machine learning.

Dr. Kot served as Associate Editor for more than ten journals, mostly for IEEE transactions. He served the IEEE SP Society in various capacities such as the General Co-Chair for the 2004 IEEE International Conference on Image Processing and the Vice-President for the IEEE Signal Processing Society. He received the Best Teacher of the Year Award and is a co-author for several Best Paper Awards including ICPR, IEEE WIFS and IWDW, CVPR Precognition Workshop and VCIP. He was elected as the IEEE Distinguished Lecturer for the Signal Processing Society and the Circuits and Systems Society. He is a Fellow of IEEE, and a Fellow of Academy of Engineering, Singapore.


<details>
<summary>Páginas originales para cotejo</summary>

### Página 1

![Página 1](original-pages/page-01.png)

### Página 2

![Página 2](original-pages/page-02.png)

### Página 3

![Página 3](original-pages/page-03.png)

### Página 4

![Página 4](original-pages/page-04.png)

### Página 5

![Página 5](original-pages/page-05.png)

### Página 6

![Página 6](original-pages/page-06.png)

### Página 7

![Página 7](original-pages/page-07.png)

### Página 8

![Página 8](original-pages/page-08.png)

### Página 9

![Página 9](original-pages/page-09.png)

### Página 10

![Página 10](original-pages/page-10.png)

### Página 11

![Página 11](original-pages/page-11.png)

### Página 12

![Página 12](original-pages/page-12.png)

### Página 13

![Página 13](original-pages/page-13.png)

### Página 14

![Página 14](original-pages/page-14.png)

</details>
