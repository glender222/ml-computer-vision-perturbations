<!-- MinerU 4.0.9, tier advanced, VLM llama.cpp/Vulkan GPU; automatic extraction, tables require comparison with the PDF. -->

[PDF original](../../Personalized%20Privacy%20Protection%20Mask%20Against%20Unauthorized%20Facial%20Recognition.pdf)

# Personalized Privacy Protection Mask Against Unauthorized Facial Recognition

Ka-Ho Chow<sup>1</sup>, Sihao Hu<sup>2</sup>, Tiansheng Huang<sup>2</sup>, and Ling Liu<sup>2</sup>

1 The University of Hong Kong
2 Georgia Institute of Technology

kachow@cs.hku.hk, {sihaohu,thuang374}@gatech.edu, lingliu@cc.gatech.edu

Abstract. Face recognition (FR) can be abused for privacy intrusion. Governments, private companies, or even individual attackers can collect facial images by web scraping to build an FR system identifying human faces without their consent. This paper introduces Chameleon, which learns to generate a user-centric personalized privacy protection mask, coined as P3-Mask, to protect facial images against unauthorized FR with three salient features. First, we use a cross-image optimization to generate one P3-Mask for each user instead of tailoring facial perturbation for each facial image of a user. It enables efficient and instant protection even for users with limited computing resources. Second, we incorporate a perceptibility optimization to preserve the visual quality of the protected facial images. Third, we strengthen the robustness of P3-Mask against unknown FR models by integrating focal diversity-optimized ensemble learning into the mask generation process. Extensive experiments on two benchmark datasets show that Chameleon outperforms three state-of-the-art methods with instant protection and minimal degradation of image quality. Furthermore, Chameleon enables cost-effective FR authorization using the P3-Mask as a personalized de-obfuscation key, and it demonstrates high resilience against adaptive adversaries.

## 1 Introduction

Face recognition (FR) has long been investigated due to its potential for enhancing security and convenience in various domains [22,28,29]. Many pretrained FR models are available online [6, 7]. Once a face database (a.k.a. the gallery) with facial images for each person of interest is provided, these pretrained models can be used to recognize them [34].

While empowering many life-enriching applications, FR can be abused to cause serious privacy issues [2]. Privacy intruders can build a face database of victims of interest from publicly available facial images on the Internet by web scraping (Figure 1 (top)). By utilizing this database, the adversary can perform unauthorized FR of individual users for stalking victims [3], intruding on victims privacy by flooding targeted ads [1], or facilitating criminals to commit identity fraud [5]. This is a real threat. For example, companies like Clearview [4] and PimEyes [8] have collected billions of online images and can recognize millions of citizens without their consent.

![](images/page_1_image_2.jpg)

<details>
<summary>flowchart</summary>

```mermaid
graph LR
  subgraph Protectee["Protectee"]
  A["Direct Sharing\nNo Protection"] --> B["Web Scraping"]
  C["Chameleon\nP3-Mask"] --> D["Surveillance Camera"]
    E["Image Source"] -.-> F["P3-Mask Protected Photo Publishing"]
  end

  subgraph PrivacyIntruder["Privacy Intruder"]
  B --> G["Face Recognition"]
  D --> H["Unknown Face"]
  G --> I["Bradley Cooper\nIdentified\nWithout Authorization"]
  H --> J["Face Recognition"]
  J --> K["Anne Hathaway\nProtected\nAgainst Unauthorized FR"]
  end
```
</details>

Fig. 1: (Top) Without protection, the privacy intruder can build a face database by web scraping and identify the unknown face. (Bottom) Chameleon learns the facial signature of the user (protectee) to generate a P3-Mask, which can be applied to protect any facial images before sharing them online against unauthorized FR.

Such a facial privacy threat has motivated the development of anti-FR technologies [36], which allow users (protectees) to preprocess their facial images before sharing them online. We argue that a competitive anti-FR solution should possess the following four properties. First, from the robustness perspective, the protected facial images should not be matched by an FR model to a query (probe) image with a correct identity (e.g., Bradley Cooper is recognized as Anne Hathaway in Figure 1 (bottom)), including those FR models possibly unknown during the preprocessing but are used by the privacy intruder. Second, from the efficiency perspective, the protection should be done instantly on any device with or without AI accelerators to maintain user experience and support equitable access to privacy protection. Third, from the perception usability perspective, the protected image should (i) preserve the visual quality rather than using excessive noise and (ii) be visually recognizable by humans to be the same person as the one in the original unprotected photo. Finally, the service usability for authorized FR providers is another important property. Existing approaches treat all FR models as unauthorized. We argue that anti-FR solutions should also allow users to grant FR permission to authorized models cost-effectively. This user-initiated de-obfuscation capability is critical to those, e.g., who publish their photos on a social media platform with privacy protection and, at the same time, wish to authorize the platform to perform some FR services (e.g., face tagging [32]) without sending and storing multiple versions of the same photo.

Based on the above objectives, we introduce Chameleon, a user-centric facial privacy protection system with three contributions. First, we develop an optimization algorithm to construct a Personalized Privacy Protection mask, coined as P3-Mask, for each user. The P3-Mask of a user can be used to protect any facial images of the same user with minimal impact on image quality, including those facial images unseen during the P3-Mask generation process. Second, it boosts the robustness of P3-Mask against unknown FR models by a principled approach, leveraging ensemble learning with models of high focal diversity. The per-user P3-Mask provides higher robustness against unknown FR models while preserving perception usability and the service usability for authorized FR providers. Chameleon users can utilize their own P3-Mask to obfuscate their facial images prior to public release, and the same P3-Mask can be used as a personalized de-obfuscation key to grant authorized FR service providers the ability to restore the facial signature of photos for correct recognition. Third, we conduct extensive experiments on two FR benchmark datasets [20,26] to analyze the shortcomings of state-of-the-art anti-FR solutions [31, 39, 40] and show that Chameleon can better protect against unknown FR models with a high success rate. The protection is lightweight and in real-time. It remains effective under adaptive privacy intruders deploying various strategies to counter Chameleon.

## 2 Related Work

Existing anti-FR approaches can be broadly classified into two categories. Synthesisbased approaches [11,16,19,21,33,41] use generative adversarial networks (GANs) [25] to synthesize a face to replace the original one for protection. While the synthetic faces look realistic, they appear to be strangers, not recognizable even by the users themselves, failing to meet the perception usability requirement.

Chameleon falls into the second category, which applies small changes to facial images [12]. Fawkes [31] formulates an untargeted attack to push the facial image away from the original location in the embedding space; TIP-IM [39] improves the image quality with MMD [9]; LowKey [10] improves the robustness under the image processing pipeline in an end-to-end ML system by incorporating it into the optimization process. These techniques can preserve the visual identity of protected faces. However, they both require iterative optimization for each image. Even for a GPU server, it can take over 100 seconds to perturb one facial image (Section 6.3). In contrast, Chameleon provides instant protection on any device, with protection even stronger than those spending minutes to find the best perturbation for each image. OPOM [40] attempts to improve efficiency by finding the embedding subspace enclosing facial images of the same person and generates a privacy protection mask to push any images away from it. However, it fails to maintain protection effectiveness when image processing operations are applied to protected images (Section 6.1), and the mask degrades the image quality much more significantly than Chameleon (Section 6.3).

## 3 Overview

To build an FR system with a pretrained model, the owner first collects a gallery dataset (face database) D, which includes the facial images of individuals of interest. Then, the pretrained FR model F is used to map each gallery image $\tilde { \pmb { x } } \; \in \; \pmb { \mathscr { D } }$ to the embedding F(x˜), and those embeddings should form clusters corresponding to different people. When a facial image x of an unknown identity, called the probe image, is given, the FR system can use the same FR model F to map it to an embedding $F ( x )$ and use the identity of the nearest gallery image to be the identity of the unknown person. The identity, $\mathcal { F R } ( \pmb { x } ; F , \pmb { \mathscr { D } } )$ , of the probe image x given the FR model $F$ and the gallery dataset D is defined as:

![](images/page_3_image_2.jpg)

<details>
<summary>flowchart</summary>

```mermaid
graph LR
  subgraph Offline_Learning_Phase
  A["Offline Learning Phase"] --> B["P3-Mask Generation"]
  B --> C["Cross-image Protection Optimization"]
  B --> D["ML Pipeline-aware Optimization"]
  B --> E["Perceptibility Optimization"]
  C --> F["P3-Mask"]
  D --> F
  E --> F
  end

  subgraph Online_Protection_Phase
    G["Photos to be shared"] -.-> H["Face Detection"]
  H --> I["③"]
  H --> J["④"]
  H --> K["⑤"]
    L["Unauthorized FR\nFAILED"] -.-> M["⑤"]
    N["Authorized FR\nSUCCESS"] -.-> O["⑥"]
  end

  F --> P["Focal Diversity-based Ensemble Optimization"]
  P --> Q["Training Faces (1)"]
  Q --> A
  I --> R["Protectee\n(Example: Bradley Cooper)"]
  J --> S["FR Authorization by Sharing P3-Mask"]
```
</details>

Fig. 2: An overview of Chameleon’s two-phase workflow for offline learning to generate P3-Mask and online protection with P3-Mask for personalized facial signature masking. The P3-Mask can protect any facial images of the same person without further learning.

$$
\mathcal {F R} (\boldsymbol {x}; F, \boldsymbol {\mathcal {D}}) = \mathcal {I} \left(\underset {\tilde {\boldsymbol {x}} \in \boldsymbol {\mathcal {D}}} {\arg \min} \operatorname{Dist} (F (\boldsymbol {x}), F (\tilde {\boldsymbol {x}}))\right), \tag {1}
$$

where $\mathcal { T } ( \tilde { \pmb { x } } )$ denotes the identity of the gallery image x˜, which is known to the FR system, and $\operatorname { D I S T } ( \cdot , \cdot )$ is a distance function such as Euclidean distance.

Threat Model. We consider a threat model where a privacy intruder conducts web scraping to collect images containing citizens’ faces. Web scraping is large-scale and untargeted. Many citizens are included in the face database, and the privacy intruder may not know them. Given a probe image taken by, e.g., a stalker’s camera, the privacy intruder uses an FR model on the face database to search for matched image(s). Then, the privacy intruder can analyze the associated metadata, such as the web page from where it was downloaded or even the exact identity. The goal of Chameleon is to allow the user to preprocess her facial images before posting them online. Even if scraped and included in the face database, they will not be matched to a probe image of her.

## 3.1 Chameleon Design

Chameleon performs preprocessing of the user’s photos before sharing them online by applying the user-specific P3-Mask. This mask is generated offline for a Chameleon user. Figure 2 provides an overview of Chameleon’s workflow.

Offline Learning Phase. Chameleon learns the unique facial signature of a user from a few facial images of her using the P3-Mask generator (Section 4), as shown in Figure 2 ○1 . P3-Mask is designed to apply directly to any facial images of the same user, including those unseen during offline learning, and is robust against lossy image processing operations in an end-to-end ML system without compromising the image quality. We use a focal diversity-optimized ensemble learning method to find a team of FR models such that the P3-Mask generated against them has strong robustness and is effective in countering other FR models that are unknown during offline learning.

Online Protection Phase. After the offline learning, ○2 the P3-Mask can be sent to the user’s local device (e.g., the mobile phone). For a photo of the user to be protected, ○3 a lightweight face detection model, such as MediaPipe [23], can be used by the Chameleon user running on the user’s device to locate the face region and ○4 instantly protect it with the P3-Mask before the user shares it online ○5 . The user can also authorize some trusted entities to conduct FR on their shared photos protected with P3-Mask. The user-specific de-obfuscation key ○6 allows the protected photos to be restored for authorized FR.

## 4 P3-Mask Generation

For a user P, Chameleon generates a P3-Mask by learning the facial signature $\mathcal { M } _ { \mathcal { P } }$ from a set of facial images the user provides. Several key factors need to be accomplished. (1) We need to promote cross-image protection as an optimization objective to generate the most representative facial signature applicable to protect any facial images of the user $\mathcal { P } ,$ including those unknown ones during the generation process (Section 4.1). (2) We need to control the amount of perturbation introduced by the P3-Mask. When applied to an unprotected image by removing the learned facial signature, it ensures minimal perception loss to pre-serve the visual quality (Section 4.2). (3) We need to keep the protected image far away from the original image in the embedding space. Multiple FR models should be used to generate alternative embedding representations to enhance generalizability against unknown models (Section 4.3).

## 4.1 Cross-image Protection Optimization

The cross-image protection capability comes from iterative learning on a set of training images the user provides. The idea is to keep fine-tuning the P3-Mask so that it can offer protection simultaneously to training images while preserving image quality. At the t-th iteration, it samples a mini-batch B from the training dataset Ω containing photos of user P. Then, we find the modification to the P3-Mask that can lead to better protection on each image in the mini-batch with better visual quality with the following operations:

$$
\boldsymbol {\mathcal {M}} _ {\mathcal {P}} ^ {t + 1} = \mathrm{CLIP} _ {[ - \epsilon , \epsilon ]} (\boldsymbol {\mathcal {M}} _ {\mathcal {P}} ^ {t} - \eta \mathrm{SIGN} (\frac {1}{| \boldsymbol {\mathcal {B}} |} \sum_ {\mathcal {X} \in \boldsymbol {\mathcal {B}}} \nabla_ {\boldsymbol {\mathcal {M}} _ {\mathcal {P}} ^ {t}} \mathcal {L} (\mathcal {X}, \boldsymbol {\mathcal {M}} _ {\mathcal {P}} ^ {t}; \boldsymbol {\mathcal {T}}, \omega))). \tag {2}
$$

Specifically, for each image ${ \mathcal { X } } \in { \pmb { \mathcal { B } } } .$ , we use the P3-Mask learned up to the current iteration, $\mathcal { M } _ { \mathcal { P } } ^ { t }$ , to protect it and compute the loss designed with dual goals:

$$
\left. \mathcal {L} \left(\mathcal {X}, \boldsymbol {\mathcal {M}} _ {\mathcal {P}} ^ {t}; \boldsymbol {\mathcal {T}}, \omega\right) = \mathcal {L} _ {\text {Protect}} \left(\mathcal {X}, \boldsymbol {\mathcal {M}} _ {\mathcal {P}} ^ {t}; \boldsymbol {\mathcal {T}}\right) + \mathcal {L} _ {\text {Percept}} \left(\mathcal {X}, \boldsymbol {\mathcal {M}} _ {\mathcal {P}} ^ {t}; \omega\right). \right. \tag {3}
$$

It uses $\mathcal { L } _ { \mathrm { P r o t e c t } }$ to learn how to adjust the P3-Mask $\mathcal { M } _ { \mathcal { P } } ^ { t }$ from the previous iteration to better protect against a team of pre-selected FR models $\mathcal { T }$ and $\mathcal { L } _ { \mathrm { P e r c e p t } }$ to preserve the image quality controlled by ω. We use the Sign of the gradients to update the P3-Mask with a learning rate η. A clipping function $\mathrm { C L I P } _ { [ - \epsilon , \epsilon ] }$ is applied to ensure the changes made by the P3-Mask on the facial image are bounded by ϵ. Such cross-image protection is not limited to those in the training dataset Ω but also unseen images of the same user. We next discuss the design of $\mathcal { L } _ { \mathrm { P r o t e c t } }$ , and $\mathcal { L } _ { \mathrm { P e r c e p t } }$ will be detailed in Section 4.2.

![](images/page_5_image_2.jpg)

<details>
<summary>flowchart</summary>

```mermaid
graph LR
  A["Training Faces"] --> B["Minibatch Sampling B"]
  B --> C["Face Detection \(\mathcal{FD}\)"]
  D["P3-Mask \(\mathcal{M}_p^t\)"] --> E["Quantization & Resizing QR"]
  F["Cropped Face \(\mathcal{FD}(\mathcal{X})\) (378 x 378)"] -.-> E
  E --> G["Resized P3-Mask QR(\(\mathcal{M}_p^t\)) (378 x 378)"]
  H["Padding & Resizing \(\Psi\)"] --> I["Protected Face \(\Theta(\mathcal{X}; \mathcal{M}_p^t)\) (112 x 112)"]
  J["Unprotected Face \(\Psi(\mathcal{FD}(\mathcal{X}))\) (112 x 112)"] --> K["Padding & Resizing \(\Psi\)"]
  L["Protection Optimization: FR Model: EN-MC"] --> M["\(F_{EN-MC}(\Theta(\mathcal{X}; \mathcal{M}_p^t))\)"]
  L --> N["\(F_{EN-MC}(\Psi(\mathcal{FD}(\mathcal{X})))\)"]
  O["FR Model: RN50-GL"] --> P["\(F_{RN50-GL}(\Theta(\mathcal{X}; \mathcal{M}_p^t))\)"]
  O --> Q["\(F_{RN50-GL}(\Psi(\mathcal{FD}(\mathcal{X})))\)"]
  R["Perceptibility Optimization"] --> S["SSIM(\(\Psi(\mathcal{FD}(\mathcal{X}))\), \(\Theta(\mathcal{X}; \mathcal{X}_p^t)\))"]
```
</details>

Fig. 3: The iterative generation of P3-Mask for a user. Chameleon goes through multi-ple facial images of the same user to optimize a P3-Mask with ML pipeline awareness.

For privacy protection, P3-Mask maximizes the distance between the protected and unprotected images in the embedding spaces of a team $\mathcal { T }$ of pre-selected FR models. We incorporate image processing operations into the optimization to avoid those lossy operations degrading P3-Mask’s protection. Consider the raw training image X in the mini-batch in Figure 3. We first conduct face detection to produce the cropped face ${ \mathcal { F } } { \mathcal { D } } ( { \mathcal { X } } )$ . Quantization and resizing are executed on the P3-Mask to match the resolution of the cropped face, denoted by $\mathcal { Q R } ( \mathcal { M } _ { \mathcal { P } } ^ { t } )$ . Then, we can apply the mask to the face for protection:

$$
\left. \Theta (\mathcal {X}; \boldsymbol {\mathcal {M}} _ {\mathcal {P}} ^ {t}) = \Psi \left(\mathrm{CLIP} _ {[ 0, 2 5 5 ]} \left(\mathcal {F D} (\mathcal {X}) - \mathcal {Q R} \left(\boldsymbol {\mathcal {M}} _ {\mathcal {P}} ^ {t}\right)\right)\right), \right. \tag {4}
$$

where $\varPsi$ is a padding and resizing function to meet the input resolution requirement of an FR model $( \mathrm { e . g . , ( 1 1 2 \times 1 1 2 ) }$ in ArcFace [17]). Note that to simplify the notation, we removed certain arguments without causing confusion.

The protection optimization in P3-Mask maximizes the average arccosine distance [17], ArcCos, between the embedding of the protected face $F ( \varTheta ( \mathcal { X } ; \mathcal { M } _ { \mathcal { P } } ^ { t } ) )$ and the embedding of its unprotected counterpart $F ( \varPsi ( \mathcal { F D } ( \mathcal { X } ) ) )$ extracted by every FR model $F \in \mathcal { T }$ in the team:

$$
\mathcal {L} _ {\text {Protect}} \left(\mathcal {X}, \boldsymbol {\mathcal {M}} _ {\mathcal {P}} ^ {t}; \boldsymbol {\mathcal {T}}\right) = \frac {- 1}{| \boldsymbol {\mathcal {T}} |} \sum_ {F \in \boldsymbol {\mathcal {T}}} \operatorname{ArcCos} \left(F \left(\Psi \left(\mathcal {F D} (\mathcal {X})\right)\right), F \left(\Theta \left(\mathcal {X}; \boldsymbol {\mathcal {M}} _ {\mathcal {P}} ^ {t}\right)\right)\right). \tag {5}
$$

## 4.2 Perceptibility Optimization

Preserving the visual quality of the facial image is necessary to maintain the usability of the protected image in practice. Hence, we incorporate a perceptibility optimization term in Equation 3. The idea is to minimize the perceptual difference between unprotected and protected images while learning a user’s facial signature, such that removing the facial signature from a given facial image will have minimal impact on the visual quality of the protected version of the original image. We use the structural similarity (SSIM) [35] to capture perceptual differences, as it has been shown to align with the human perception system:

![](images/page_6_chart_2.jpg)

<details>
<summary>scatter</summary>

| Focal Diversity | Protection Effectiveness |
| --- | --- |
| ~0.085 | ~73 |
| ~0.09 | ~73 |
| ~0.10 | ~69 |
| ~0.105 | ~81 |
| ~0.11 | ~72 |
| ~0.12 | ~84 |
| ~0.125 | ~86 |
| ~0.13 | ~85 |
| ~0.135 | ~82 |
| ~0.14 | ~84 |
| ~0.145 | ~90 |
| ~0.15 | ~84 |
| ~0.155 | ~81 |
| ~0.16 | ~75 |
| ~0.165 | ~89 |
| ~0.17 | ~80 |
| ~0.175 | ~84 |
| ~0.18 | ~92 |
| ~0.25 | ~92 |
| ~0.26 | ~96 |
</details>

(a) Two-model: 28 Options

![](images/page_6_chart_4.jpg)

<details>
<summary>scatter</summary>

| Focal Diversity | Protection Effectiveness |
| --- | --- |
| ~0.095 | ~76 |
| ~0.100 | ~79 |
| ~0.102 | ~74 |
| ~0.118 | ~82 |
| ~0.120 | ~86 |
| ~0.123 | ~93 |
| ~0.125 | ~87 |
| ~0.128 | ~87 |
| ~0.130 | ~87 |
| ~0.132 | ~87 |
| ~0.135 | ~86 |
| ~0.138 | ~94 |
| ~0.140 | ~82 |
| ~0.142 | ~87 |
| ~0.145 | ~84 |
| ~0.148 | ~94 |
| ~0.150 | ~84 |
| ~0.152 | ~85 |
| ~0.155 | ~84 |
| ~0.160 | ~91 |
| ~0.165 | ~92 |
| ~0.170 | ~95 |
| ~0.175 | ~91 |
| ~0.180 | ~96 |
| ~0.185 | ~96 |
| ~0.190 | ~94 |
| ~0.195 | ~96 |
| ~0.205 | ~97 |
</details>

(b) Three-model: 56 Options

Fig. 4: Focal diversity provides a strong indicator of protection effectiveness for selecting an ensemble.

![](images/page_6_chart_7.jpg)

<details>
<summary>bar</summary>

| Category | Protection Effectiveness (%) |
| --- | --- |
| Initial Baseline | ~69 |
| Step 1 | ~71 |
| Step 2 | ~72 |
| Step 3 | ~73 |
| Step 4 | ~73 |
| Step 5 | ~75 |
| Step 6 | ~80 |
| Step 7 | ~81 |
| Step 8 | ~81 |
| Step 9 | ~81 |
| Step 10 | ~82 |
| Step 11 | ~83 |
| Step 12 | ~83 |
| Step 13 | ~83 |
| Step 14 | ~84 |
| Step 15 | ~84 |
| Step 16 | ~84 |
| Step 17 | ~85 |
| Step 18 | ~85 |
| Step 19 | ~86 |
| Step 20 | ~86 |
| Step 21 | ~87 |
| Step 22 | ~89 |
| Step 23 | ~90 |
| Step 24 | ~92 |
| Step 25 | ~92 |
| Final Baseline | ~97 |
</details>

Fig. 5: Out of 28 options of two-model ensembles from a pool of eight models, the ensemble selected by our approach (green) leads to much better protection than a randomly selected one (red).

$$
\mathcal {L} _ {\text {Percept}} \left(\mathcal {X}, \boldsymbol {\mathcal {M}} _ {\mathcal {P}} ^ {t}; \omega\right) = \lambda_ {\text {SSIM}} \max \left[ \frac {1 - \operatorname{SSIM} \left(\Psi \left(\mathcal {F D} (\mathcal {X})\right) , \Theta \left(\mathcal {X} ; \boldsymbol {\mathcal {M}} _ {\mathcal {P}} ^ {t}\right)\right)}{2} - \omega , 0 \right]. \tag {6}
$$

The parameters, ω and $\lambda _ { \mathrm { S S I M } }$ , enable two features. ω controls the SSIM degradation. Any SSIM degradation greater than $2 \omega$ will cause the term to be non-zero, making the optimization adjust the P3-Mask reducing its impact on image quality. λ<sub>SSIM</sub> balances the importance of privacy protection and perceptibility. We use dynamic scheduling [31] such that it will be adjusted automatically.

## 4.3 Focal Diversity Ensemble Optimization

For each original image and its protected version by removing the facial signature learned up to the current iteration, we use two FR models in Figure 3 to generate two embeddings, which serve as alternative channels to learn a highquality facial signature. Choosing FR models that best complement each other can achieve better protection than randomly selected FR models. We use an ensemble optimization method to select the best k-model ensemble among a pool of N FR models. The key idea is to find an ensemble with members making decorrelated mistakes. To quantify this behavior, we leverage the focal diversity framework [13,14,37]. As shown in Figure 4, the higher the focal diversity of the ensemble (a green dot), the stronger the P3-Mask will be in protecting against other FR models not used during the P3-Mask generation process.

Given a collection of N FR models $\{ F _ { 1 } , . . . , F _ { N } \}$ , we first identify the negative samples for each FR model F by locating validation images that $F$ fails to recognize their true identity. To rank ensembles of size $S ,$ we enumerate all $\left( \begin{matrix} { N } \\ { S } \\ \end{matrix} \right)$ combinations. For each combination (ensemble) $\mathcal { T }$ , we consider each member to be the focal model $F _ { \mathrm { f o c a l } }$ and use its negative samples to statistically estimate the level of negative correlation $\lambda _ { \mathrm { f o c a l } } ( \mathcal { T } ; F _ { \mathrm { f o c a l } } )$ between $F _ { \mathrm { f o c a l } }$ and the remaining models in $\mathcal { T }$ , computed by measuring the degree of disagreements using the generalized non-pairwise measure [27]. The same procedure is repeated by considering each member in the ensemble as the focal model, and the focal diversity of the ensemble $\mathcal { T }$ is finalized as $\begin{array} { r } { d _ { \mathrm { f o c a l } } ( \mathcal { T } ) = \frac { 1 } { S } \sum _ { F _ { \mathrm { f o c a l } } \in \mathcal { T } } [ 1 - \lambda _ { \mathrm { f o c a l } } ( \mathcal { T } ; F _ { \mathrm { f o c a l } } ) ] } \end{array}$ Given a team size $S ,$ the ensemble with the highest focal diversity can be deployed. As shown in Figure $5 ,$ the selected two-model team (green bar) is indeed the one leading to the strongest protection among all 28 options, which is over 20% stronger than a randomly selected ensemble (red bar). Note that we only need to analyze the failures of individual FR models, which is significantly more efficient than generating P3-Mask for each option and conducting evaluation.

## 5 Online Image Protection and Refinement

The user $\mathcal { P }$ obtains her P3-Mask $\mathcal { M } _ { \mathcal { P } }$ from Chameleon. Given an image $\mathcal { X } ,$ she can obfuscate her facial identity on her device by applying the P3-Mask:

$$
\operatorname{MASK} \left(\mathcal {X}; \boldsymbol {\mathcal {M}} _ {\mathcal {P}}\right) = \operatorname{CLIP} _ {[ 0, 2 5 5 ]} \left(\mathcal {F D} (\mathcal {X}) - \mathcal {Q R} \left(\boldsymbol {\mathcal {M}} _ {\mathcal {P}}\right)\right), \tag {7}
$$

which is the protected facial region to be put in the original image X to produce the protected version $\mathcal { X } ^ { \prime }$ for sharing.

Chameleon supports authorizing trusted third parties to conduct FR on a user’s facial images in a space-time efficient manner without transmitting and storing multiple versions of the same photo (one protected for public sharing and one unprotected for internal FR). By sharing the P3-Mask $\mathcal { M } _ { \mathcal { P } }$ as the key, the third parties can de-obfuscate the facial signature by unmasking:

$$
\left| \operatorname{UNMASK} \left(\mathcal {X} ^ {\prime}; \boldsymbol {\mathcal {M}} _ {\mathcal {P}}\right) = \operatorname{CLIP} _ {[ 0, 2 5 5 ]} \left(\mathcal {F D} \left(\mathcal {X} ^ {\prime}\right) + \mathcal {Q R} \left(\boldsymbol {\mathcal {M}} _ {\mathcal {P}}\right)\right). \right. \tag {8}
$$

In Section 6.2, we will show two properties of this process: (i) the signaturerestored photos can be used for FR with no accuracy degradation, and (ii) the protection can only be done by the P3-Mask of the same person in the facial image, and the restoration is successful only if the same key is used.

## 6 Experimental Evaluation

We conduct experiments on FaceScrub [26] and LFW [20]. To provide a detailed analysis, ten celebrities are randomly selected as users on FaceScrub (Table 1). For each user, we split their facial images into three parts: (i) 10% are used as probe images (2nd column), (ii) 70% are used as gallery images and are used by Chameleon to train the mask (3rd column), and (iii) 20% are also used as gallery images but unseen during the training process (4th column). The last part is crucial to evaluate the protection of unseen images of the user. In total, we have 123 facial images with “unknown identities.” Following relevant works [31, 40], all facial images of other people are included in the gallery dataset as noise. The same splitting method is also used for LFW. By default, the results reported in this paper focus on FaceScrub due to similar observations on LFW. The source code is available at [https://github.com/git-disl/Chameleon](https://github.com/git-disl/Chameleon).

Table 1: Ten example celebrities (users) on FaceScrub for analysis.

Table 2: The collection of publicly available pretrained FR models.

<table><tr><td rowspan="2">Name</td><td rowspan="2">Probe Images</td><td colspan="3">Gallery Images</td></tr><tr><td>Seen</td><td>Unseen</td><td>Total</td></tr><tr><td>Morena Baccarin</td><td>11</td><td>82</td><td>23</td><td>105</td></tr><tr><td>Bradley Cooper</td><td>12</td><td>89</td><td>25</td><td>114</td></tr><tr><td>America Ferrera</td><td>16</td><td>113</td><td>32</td><td>145</td></tr><tr><td>Gerard Butler</td><td>11</td><td>82</td><td>24</td><td>106</td></tr><tr><td>Eva Longoria</td><td>12</td><td>91</td><td>26</td><td>117</td></tr><tr><td>Melissa Egan</td><td>12</td><td>87</td><td>24</td><td>111</td></tr><tr><td>Kim Cattrall</td><td>13</td><td>96</td><td>27</td><td>123</td></tr><tr><td>Allison Janney</td><td>12</td><td>89</td><td>26</td><td>115</td></tr><tr><td>Roma Downey</td><td>11</td><td>78</td><td>22</td><td>100</td></tr><tr><td>Steve Carell</td><td>13</td><td>96</td><td>28</td><td>124</td></tr><tr><td>Others (Noise)</td><td>/</td><td>/</td><td>/</td><td>48368</td></tr><tr><td colspan="4">Total Number of Gallery Images:</td><td>49528</td></tr></table>

| Model ID | Neural Arch. | Training Dataset | FR Acc. |
| --- | --- | --- | --- |
| EN-MC | EfficientNet | MS-Celeb-1M | 94.94% |
| RN50-GL | ResNet50 | Glint360K | 96.68% |
| RN50-MC | ResNet50 | MS-Celeb-1M | 89.25% |
| RN50-VF | ResNet50 | VGG-Face2 | 94.30% |
| RN50-WF | ResNet50 | WebFace600K | 96.72% |
| RN18-MC | ResNet18 | MS-Celeb-1M | 82.64% |
| RN34-MC | ResNet34 | MS-Celeb-1M | 84.49% |
| RN100-MC | ResNet100 | MS-Celeb-1M | 91.85% |

Public clouds (e.g., Azure) now require manual approval to use their FR services. Hence, we believe that privacy intruders will opt for deploying pretrained FR models, as many high-quality ones are available on the Internet and can be used out of the box. Due to the large number of possible FR algorithms and architectures, we first focus on the eight FR models listed in Table 2, which are based on ArcFace [17], the state-of-the-art FR algorithm, with varying neural architectures and training datasets. Chameleon can be easily extended to incorporate any FR models by simply adding them to the collection. Nonetheless, in Section 6.4, we will show that thanks to the focal diversity-optimized teaming, the P3-Mask generated from a collection of ArcFace-only models can also be effective in protecting against privacy intruders using models of other FR algorithms, such as FaceNet [30] and MagFace [24]. By default, we use the twomodel team (EN-MC, RN50-GL) with the highest focal diversity (Section 4.3). The P3-Mask is trained for 50 epochs on NVIDIA RTX 2080 SUPER GPU with η = 0.001, |B| = 4, ϵ = 0.063, and ω = 0.03. We provide details for reproducibility and additional results in the appendix.

## 6.1 Protection Effectiveness

Table 3 shows the personalized mask for seven users on FaceScrub (other users are available in the appendix). Different users have distinct facial signatures (P3- Mask) learned by Chameleon. We apply these masks to their respective gallery images and test the FR accuracy using probe images. All probe images can be correctly identified when no protection mechanism is used. Under Chameleon, the probe images should be matched to an incorrect identity, resulting in a low FR accuracy. Hence, we define an evaluation metric, Protection Success Rate (PSR), to be (100 − FR Accuacy) reported in percentages.

Table 3: The P3-Mask (rescaled) for seven users on FaceScrub. Chameleon learns the distinct facial signature for each of them.

<table><tr><td>M. Baccarin</td><td>B. Cooper</td><td>A. Ferrera</td><td>G. Butler</td><td>E. Longoria</td><td>M. Egan</td><td>K. Cattrall</td></tr><tr><td><img src="images/page_8_table_image_10_1.jpg"/></td><td><img src="images/page_8_table_image_10_2.jpg"/></td><td><img src="images/page_8_table_image_10_3.jpg"/></td><td><img src="images/page_8_table_image_10_4.jpg"/></td><td><img src="images/page_8_table_image_10_5.jpg"/></td><td><img src="images/page_8_table_image_10_6.jpg"/></td><td><img src="images/page_8_table_image_10_7.jpg"/></td></tr></table>

![](images/page_9_chart_2.jpg)

<details>
<summary>bar</summary>

| FR Model | Chameleon | OPOM | TIP-IM | Fawkes |
| :--- | :--- | :--- | :--- | :--- |
| RN50-VF | ~91 | ~35 | ~64 | ~59 |
| RN50-WF | ~97 | ~35 | ~87 | ~77 |
| RN18-MC | ~98 | ~52 | ~80 | ~84 |
| RN34-MC | ~95 | ~47 | ~71 | ~94 |
| RN100-MC | ~98 | ~51 | ~44 | ~68 |
</details>

(a) Dataset: FaceScrub

![](images/page_9_chart_4.jpg)

<details>
<summary>bar</summary>

| FR Model | Chameleon (%) | OPOM (%) | TIP-IM (%) | Fawkes (%) |
| :--- | :--- | :--- | :--- | :--- |
| RN50-VF | ~91 | ~32 | ~73 | ~74 |
| RN50-WF | ~96 | ~32 | ~85 | ~79 |
| RN18-MC | ~97 | ~49 | ~79 | ~85 |
| RN34-MC | ~94 | ~49 | ~86 | ~80 |
| RN100-MC | ~97 | ~41 | ~75 | ~77 |
</details>

(b) Dataset: LFW

Fig. 6: Protection effectiveness on two benchmarks using Chameleon, OPOM [40], TIP-IM [39], and Fawkes [31] against FR models unknown to them.

Chameleon can protect against unknown FR models. We compare the cross-model protection of Chameleon with OPOM [40], the only anti-FR approach generating one privacy protection mask for each person as Chameleon, and both TIP-IM [39] and Fawkes [31], which conduct per-image optimization. For a fair comparison, all methods are set with the same perturbation budget. Figure 6 summarizes the results on both datasets. All five FR models are <u>unknown</u> to both protection mechanisms. We make two observations. First, Chameleon consistently outperforms OPOM by a large margin. We found that OPOM masks do not transfer well, especially considering the end-to-end ML pipeline. Its PSR can drop by 33% because of those lossy operations. Second, Chameleon can be as competitive as those methods conducting per-image optimization. TIP-IM and Fawkes spend minutes to optimize the protection for each image. Still, Chameleon outperforms them and can be applied instantly.

Table 4 reports the detailed PSR for each user Chameleon achieves when the privacy intruder uses different FR models (2nd to 9th columns). We make two observations. First, when the privacy intruder uses an FR model known to the generation process, a PSR of 100% can be consistently achieved (2nd to 3rd columns). Second, Chameleon can protect against unknown FR models (4th to 9th columns). Even when the FR model used by the intruder is trained on a different dataset (i.e., RN50-VF and RN50-WF) or with a different neural architecture (i.e., RN18-MC, RN34-MC, and RN100-MC) than any FR models known by Chameleon, it still provides a PSR over 90.65%, meaning that facial images of a user are matched to gallery images of a different person. To demonstrate such a mismatch, in Table 5, we show the probe images from two users and their most similar gallery images found by four unknown FR models with two settings: (i) the “Unprotected" scenario where no one employs protection and (ii) the “Chameleon" scenario where the intruder scraped photos protected by our solution. Taking M. Baccarin as an example, the most similar gallery images found in the unprotected scenario belong to her. In contrast, when Chameleon is used, the same probe image is misidentified, e.g., as L. Hartley by RN50-MC.

Table 4: Using the two-model team with high focal diversity (2nd and 3rd columns), Chameleon can offer privacy protection even when the privacy intruder uses FR models unseen during the mask generation process (4th to 9th columns).

<table><tr><td rowspan="2">Name</td><td colspan="10">Protection Success Rate - PSR (%)</td></tr><tr><td>EN-MC</td><td>RN50-GL</td><td>RN50-MC</td><td>RN50-VF</td><td>RN50-WF</td><td>RN18-MC</td><td>RN34-MC</td><td>RN100-MC</td><td>Mean</td><td>Std</td></tr><tr><td>M. Baccarin</td><td>100.00</td><td>100.00</td><td>100.00</td><td>72.73</td><td>90.91</td><td>100.00</td><td>90.91</td><td>100.00</td><td>94.32</td><td>9.64</td></tr><tr><td>B. Cooper</td><td>100.00</td><td>100.00</td><td>83.33</td><td>100.00</td><td>100.00</td><td>100.00</td><td>91.67</td><td>91.67</td><td>95.83</td><td>6.30</td></tr><tr><td>A. Ferrera</td><td>100.00</td><td>100.00</td><td>93.75</td><td>75.00</td><td>100.00</td><td>100.00</td><td>93.75</td><td>100.00</td><td>95.31</td><td>8.68</td></tr><tr><td>G. Butler</td><td>100.00</td><td>100.00</td><td>90.91</td><td>100.00</td><td>100.00</td><td>90.91</td><td>90.91</td><td>100.00</td><td>96.59</td><td>4.70</td></tr><tr><td>E. Longoria</td><td>100.00</td><td>100.00</td><td>100.00</td><td>100.00</td><td>100.00</td><td>100.00</td><td>100.00</td><td>100.00</td><td>100.00</td><td>0.00</td></tr><tr><td>M. Egan</td><td>100.00</td><td>100.00</td><td>100.00</td><td>100.00</td><td>100.00</td><td>100.00</td><td>100.00</td><td>91.67</td><td>98.96</td><td>2.95</td></tr><tr><td>K. Cattrall</td><td>100.00</td><td>100.00</td><td>100.00</td><td>84.62</td><td>100.00</td><td>100.00</td><td>100.00</td><td>100.00</td><td>98.08</td><td>5.44</td></tr><tr><td>A. Janney</td><td>100.00</td><td>100.00</td><td>91.67</td><td>100.00</td><td>91.67</td><td>100.00</td><td>100.00</td><td>100.00</td><td>97.92</td><td>3.86</td></tr><tr><td>R. Downey</td><td>100.00</td><td>100.00</td><td>100.00</td><td>72.73</td><td>90.91</td><td>100.00</td><td>90.91</td><td>100.00</td><td>94.32</td><td>9.64</td></tr><tr><td>S. Carell</td><td>100.00</td><td>100.00</td><td>84.62</td><td>92.31</td><td>92.31</td><td>92.31</td><td>76.92</td><td>100.00</td><td>92.31</td><td>8.22</td></tr><tr><td>Mean</td><td>100.00</td><td>100.00</td><td>93.52</td><td>90.65</td><td>96.58</td><td>97.41</td><td>94.42</td><td>97.42</td><td rowspan="2" colspan="2"></td></tr><tr><td>Std</td><td>0.00</td><td>0.00</td><td>6.40</td><td>11.16</td><td>4.43</td><td>4.18</td><td>7.41</td><td>4.15</td></tr></table>

Table 5: The most similar gallery images on FaceScrub found by different FR models unknown to Chameleon.

<table><tr><td rowspan="3">Probe Image</td><td colspan="8">The Most Similar Gallery Image &amp; Its Identity Found by Unseen FR Models</td></tr><tr><td colspan="2">RN50-MC</td><td colspan="2">RN50-VF</td><td colspan="2">RN34-MC</td><td colspan="2">RN100-MC</td></tr><tr><td>Unprotected</td><td>Chameleon</td><td>Unprotected</td><td>Chameleon</td><td>Unprotected</td><td>Chameleon</td><td>Unprotected</td><td>Chameleon</td></tr><tr><td><img src="images/page_10_table_image_5_1.jpg"/></td><td><img src="images/page_10_table_image_5_2.jpg"/></td><td><img src="images/page_10_table_image_5_3.jpg"/></td><td><img src="images/page_10_table_image_5_4.jpg"/></td><td><img src="images/page_10_table_image_5_5.jpg"/></td><td><img src="images/page_10_table_image_5_6.jpg"/></td><td><img src="images/page_10_table_image_5_7.jpg"/></td><td><img src="images/page_10_table_image_5_8.jpg"/></td><td><img src="images/page_10_table_image_5_9.jpg"/></td></tr><tr><td>M. Baccarin</td><td>M. Baccarin</td><td>L. Hartley</td><td>M. Baccarin</td><td>L. Hartley</td><td>M. Baccarin</td><td>C. Electra</td><td>M. Baccarin</td><td>L. Hartley</td></tr><tr><td><img src="images/page_10_table_image_5_10.jpg"/></td><td><img src="images/page_10_table_image_5_11.jpg"/></td><td><img src="images/page_10_table_image_5_12.jpg"/></td><td><img src="images/page_10_table_image_5_13.jpg"/></td><td><img src="images/page_10_table_image_5_14.jpg"/></td><td><img src="images/page_10_table_image_5_15.jpg"/></td><td><img src="images/page_10_table_image_5_16.jpg"/></td><td><img src="images/page_10_table_image_5_17.jpg"/></td><td><img src="images/page_10_table_image_5_18.jpg"/></td></tr><tr><td>B. Cooper</td><td>B. Cooper</td><td>P. Walker</td><td>B. Cooper</td><td>M. Vartan</td><td>B. Cooper</td><td>M. Vartan</td><td>B. Cooper</td><td>M. Vartan</td></tr></table>

M. Baccarin

M. Baccarin

L. Hartley

M. Baccarin

L. Hartley

M. Baccarin

C. Electra

M. Baccarin

L. Hartley

## 6.2 FR Service Usability

Chameleon allows users to grant FR permission to trusted third parties by sharing their P3-Mask to de-obfuscate the protected image. There is no need to transmit and store an unprotected photo for internal use and a protected one for public visibility, doubling network and storage costs. Table 6 shows the results in FR accuracy. First, using the correct mask (i.e., the one used to protect the images) to reverse the protection can restore FR accuracy. Taking M. Baccarin as an example, the FR accuracy increases from 6.82% before unmasking (b) to 100% after unmasking (c). Second, using an incorrect mask for unmasking cannot restore the FR accuracy and can lead to even worse performance. It drops from 6.82% to 1.14% after unmasking (d).

## 6.3 Protection Cost Analysis

Speed and Resources. Figure 7 compares Chameleon with OPOM, TIP-IM, and Fawkes regarding the protection time per face, compute, and storage costs.

Table 6: The protection by Chameleon can reduce FR accuracy against FR models (b), but the user can provide the P3-Mask used to protect her images as the key for the authorized third party to reverse the protection process (c), which leads to restored FR performance. However, an incorrect key cannot restore FR and may worsen it (d).

<table><tr><td rowspan="2">Scenario</td><td colspan="10">Face Recognition Accuracy (%)</td></tr><tr><td>EN-MC</td><td>RN50-GL</td><td>RN50-MC</td><td>RN50-VF</td><td>RN50-WF</td><td>RN18-MC</td><td>RN34-MC</td><td>RN100-MC</td><td>Mean</td><td>Std</td></tr><tr><td colspan="11">User: M. Baccarin</td></tr><tr><td>(a) No Protection</td><td>100.00</td><td>100.00</td><td>100.00</td><td>100.00</td><td>100.00</td><td>100.00</td><td>100.00</td><td>100.00</td><td>100.00</td><td>0.00</td></tr><tr><td>(b) Protected</td><td>0.00</td><td>0.00</td><td>9.09</td><td>18.18</td><td>9.09</td><td>9.09</td><td>0.00</td><td>9.09</td><td>6.82</td><td>6.43</td></tr><tr><td>(c) Correctly Unmasked</td><td>100.00</td><td>100.00</td><td>100.00</td><td>100.00</td><td>100.00</td><td>100.00</td><td>100.00</td><td>100.00</td><td>100.00</td><td>0.00</td></tr><tr><td>(d) Incorrectly Unmasked</td><td>0.00</td><td>0.00</td><td>0.00</td><td>0.00</td><td>9.09</td><td>0.00</td><td>0.00</td><td>0.00</td><td>1.14</td><td>3.21</td></tr><tr><td colspan="11">User: B. Cooper</td></tr><tr><td>(a) No Protection</td><td>100.00</td><td>100.00</td><td>100.00</td><td>100.00</td><td>100.00</td><td>100.00</td><td>100.00</td><td>100.00</td><td>100.00</td><td>0.00</td></tr><tr><td>(b) Protected</td><td>0.00</td><td>0.00</td><td>16.67</td><td>0.00</td><td>0.00</td><td>0.00</td><td>8.33</td><td>8.33</td><td>4.17</td><td>6.30</td></tr><tr><td>(c) Correctly Unmasked</td><td>100.00</td><td>100.00</td><td>100.00</td><td>100.00</td><td>100.00</td><td>100.00</td><td>100.00</td><td>100.00</td><td>100.00</td><td>0.00</td></tr><tr><td>(d) Incorrectly Unmasked</td><td>0.00</td><td>0.00</td><td>0.00</td><td>0.00</td><td>0.00</td><td>0.00</td><td>0.00</td><td>0.00</td><td>0.00</td><td>0.00</td></tr></table>

![](images/page_11_image_4.jpg)

Fig. 7: Chameleon offers much better privacy protection than OPOM without sacrificing speed and resource usages. It can protect instantly and require negligible compute resources as OPOM, while TIP-IM and Fawkes are slow and costly.

Chameleon and OPOM produce masks that are applicable to any facial image of the same user. The protection can be completed in 0.0076 seconds. Note that Chameleon is significantly more effective than OPOM, as described in Figure 6. Instead, TIP-IM and Fawkes require per-image optimization and take 105.12 seconds to complete the protection with GPUs for acceleration and storage for FR models. Chameleon only needs to conduct simple arithmetic operations. Only the P3-Mask needs to be stored on the user device. Hence, it can be deployed with instant protection even on an edge device with weak computing power.

Image Quality. Chameleon can well preserve image quality. Table 7 provides two examples for four users, contrasting the facial images with no protection (3rd and 5th columns) with the ones with their facial signature removed by Chameleon (4th and 6th columns). They capture different variations of facial images: M. Baccarin’s images have different lighting conditions, B. Cooper’s have different face sizes, E. Longoria’s have different postures, and G. Bulter’s have different expressions. The protected images are visually similar to the unprotected counterparts, but they will not be matched to the clean images of the corresponding person. Indeed, Chameleon can generate higher-quality images than existing methods. As reported in Table 8, Chameleon not only offers better protection (Figure 6) with a similar cost (Figure 7) as OPOM but also achieves much better image quality measured in SSIM (0.9493 vs 0.8839).

Table 7: The P3-Mask can be applied to any facial images of the same person while preserving image quality.

Table 8: Chameleon offers much better protection while maintaining image quality.

<table><tr><td rowspan="2">User</td><td rowspan="2">Mask</td><td colspan="2">Sample Image 1</td><td colspan="2">Sample Image 2</td></tr><tr><td>No Protection</td><td>Protected</td><td>No Protection</td><td>Protected</td></tr><tr><td>M. Baccarin</td><td rowspan="4"><img src="images/page_12_table_image_4_1.jpg"/></td><td rowspan="4"><img src="images/page_12_table_image_4_2.jpg"/></td><td rowspan="4"><img src="images/page_12_table_image_4_3.jpg"/></td><td rowspan="4"><img src="images/page_12_table_image_4_4.jpg"/></td><td rowspan="4"><img src="images/page_12_table_image_4_5.jpg"/></td></tr><tr><td>B. Cooper</td></tr><tr><td>E. Longoria</td></tr><tr><td>G. Image</td></tr></table>

<table><tr><td></td><td>Example</td></tr><tr><td>Original SSIM: 1.0000</td><td></td></tr><tr><td>Chameleon SSIM: 0.9493</td><td></td></tr><tr><td>OPOM SSIM: 0.8839</td><td><img src="images/page_12_table_image_5_1.jpg"/></td></tr><tr><td>TIP-IM SSIM: 0.8850</td><td></td></tr><tr><td>Fawkes SSIM: 0.9612</td><td></td></tr></table>

## 6.4 Focal Diversity-based Teaming

Chameleon automatically selects a high-quality team for deployment with a specified budget. Table 9 compares the detailed PSR using the most diverse and the least diverse teams with (a) two and (b) three models. In both cases, the most diverse team outperforms the least diverse one, which is only effective when the privacy intruder uses the FR model known in the team. The selection of highquality teams is non-trivial because we observe that composing a team of FR models with the highest FR accuracy is not always a good option, and a team of models having different neural architectures is not always the top priority [13].

In Figure 8, we further show that the P3-Mask generated from a carefullychosen team can protect against FR models of unknown algorithms. Specifically, we consider the most and the least diverse three-model teams in Table 9(b). Even though both teams include only FR models based on ArcFace [17], the most diverse one can effectively protect against FR models based on FaceNet [30] or MagFace [24]. The protection effectiveness can be further strengthened by incorporating more FR models into the collection.

Table 9: The most diverse and the least diverse teams with (a) two or (b) three models identified by our focal diversity-based teaming method. The most diverse teams offer significantly better protection in terms of protection success rates.

<table><tr><td rowspan="2">Protection Team</td><td colspan="10">Protection Success Rate - PSR (%)</td></tr><tr><td>EN-MC</td><td>RN50-GL</td><td>RN50-MC</td><td>RN50-VF</td><td>RN50-WF</td><td>RN18-MC</td><td>RN34-MC</td><td>RN100-MC</td><td>Mean</td><td>Std</td></tr><tr><td colspan="11">(a) Two-model Teams</td></tr><tr><td>Most Diverse:(EN-MC, RN50-GL)</td><td>100.00</td><td>100.00</td><td>93.52</td><td>90.65</td><td>96.58</td><td>97.41</td><td>94.42</td><td>97.42</td><td>96.25</td><td>3.23</td></tr><tr><td>Least Diverse:(RN18-MC, RN34-MC)</td><td>11.38</td><td>60.16</td><td>100.00</td><td>55.28</td><td>73.17</td><td>100.00</td><td>100.00</td><td>80.49</td><td>72.56</td><td>28.54</td></tr><tr><td colspan="11">(b) Three-model Teams</td></tr><tr><td>Most Diverse:(EN-MC, RN50-GL, RN50-WF)</td><td>100.00</td><td>100.00</td><td>100.00</td><td>100.00</td><td>92.68</td><td>93.50</td><td>100.00</td><td>100.00</td><td>98.27</td><td>3.00</td></tr><tr><td>Least Diverse:(RN18-MC, RN34-MC, RN50-MC)</td><td>2.44</td><td>78.86</td><td>100.00</td><td>52.03</td><td>84.55</td><td>100.00</td><td>100.00</td><td>93.50</td><td>76.42</td><td>31.82</td></tr></table>

![](images/page_13_chart_2.jpg)

<details>
<summary>bar</summary>

| FR Model | Least Diverse (Mean PSR %) | Most Diverse (Mean PSR %) |
| --- | --- | --- |
| FaceNet | ~12 | ~78 |
| MagFace | ~9 | ~70 |
</details>

Fig. 8: The most diverse teams allow Chameleon to be effective against FR models of unknown algorithms.

![](images/page_13_chart_4.jpg)

<details>
<summary>bar</summary>

| Category | No Attack (%) | JPEG-compression Attack (%) | Gaussian Filter Attack (%) | Median Filter Attack (%) |
| --- | --- | --- | --- | --- |
| Chameleon | ~97 | ~93 | ~82 | ~85 |
| OPOM | ~34 | ~34 | ~33 | ~34 |
| TIP-IM | ~87 | ~58 | ~52 | ~48 |
| Fawkes | ~76 | ~42 | ~43 | ~37 |
</details>

Fig. 9: The PSR of Chameleon against adaptive intruders using different strategies to wash out the patterns in protected images.

## 6.5 Chameleon Against Adaptive Adversaries

An adaptive adversary may run Chameleon to produce the P3-Mask for each person and de-obfuscate their photos. When probe images need to be identified, they will be matched against a “clean" face database. However, it is infeasible, as we show in Section 6.2, that the FR accuracy can only be restored when the masks used for obfuscation and de-obfuscation are identical. It is impossible to reproduce the same P3-Mask used by the user because it requires the same set of clean images and random states.

Alternatively, an adaptive adversary may wash out patterns introduced by Chameleon before performing FR. We consider three popular lightweight and task-agnostic methods studied in the adversarial example domain [18] in Figure 9, including (a) JPEG compression [15], (b) Gaussian Filter, and (d) Median Filter [38]. The results show that their effectiveness is limited on Chameleon. The consideration of the end-to-end ML pipeline increases the robustness of P3-Mask.

## 7 Conclusions

We have presented Chameleon against unauthorized FR. Chameleon generates a P3-Mask for each user with cross-image and perceptibility optimizations to offer (i) instant and lightweight protection on any facial images of the same user and (ii) preservation of image quality. Also, we have shown that P3-Mask enables cost-effective de-obfuscation for authorizing FR services. Such an authorization process can only be conducted by the one with the identical mask used for protection. Chameleon also features a focal diversity-optimized teaming method to select a high-quality FR team to generate P3-Mask with strong robustness against unknown FR models.

## Acknowledgments

This research is partially sponsored by the NSF CISE grants 2302720, 2312758, 2038029, an IBM faculty award, and a grant from CISCO Edge AI program. It is part of the PhD dissertation of the first author, who graduated from Georgia Tech in Spring 2023. The first author acknowledges the support of the IBM PhD Fellowship in 2022-2023 and the support from the HKU-CS Start-up Fund.

## References

1. Brazilian retailer quizzed over facial recognition tech. https://www.zdnet.com/article/brazilian-retailer-quizzed-over-facial-recognition-tech/ (2019)
2. Facial recognition in the united states: Privacy concerns and legal developments. https://www.asisonline.org/security-management-magazine/monthly-issues/security-technology/archive/2021/december/facial-recognition-in-the-us-privacy-concerns-and-legal-developments/ (2021)
3. The secretive company that might end privacy as we know it. https://www.nytimes.com/2020/01/18/technology/clearview-privacy-facial-recognition.html (2021)
4. Clearview AI. https://www.clearview.ai/ (2023)
5. Facial recognition and identity risk. https://www.equifax.co.uk/resources/identity-protection/facial-recognition-and-identity-risk.html (2023)
6. HuggingFace. https://huggingface.co (2023)
7. Insightface: 2d and 3d face analysis project. https://github.com/deepinsight/insightface (2023)
8. PimEyes. https://pimeyes.com/ (2023)
9. Borgwardt, K.M., Gretton, A., Rasch, M.J., Kriegel, H.P., Schölkopf, B., Smola, A.J.: Integrating structured biological data by kernel maximum mean discrepancy. Bioinformatics 22(14), e49–e57 (2006)
10. Cherepanova, V., Goldblum, M., Foley, H., Duan, S., Dickerson, J., Taylor, G., Goldstein, T.: Lowkey: Leveraging adversarial attacks to protect social media users from facial recognition. In: International Conference on Learning Representations (ICLR) (2021)
11. Choi, Y., Choi, M., Kim, M., Ha, J.W., Kim, S., Choo, J.: Stargan: Unified generative adversarial networks for multi-domain image-to-image translation. In: Proceedings of the IEEE conference on computer vision and pattern recognition. pp. 8789–8797 (2018)
12. Chow, K.H., Hu, S., Huang, T., Ilhan, F., Wei, W., Liu, L.: Diversity-driven privacy protection masks against unauthorized face recognition. Proceedings on Privacy Enhancing Technologies 4, 381–392 (2024)
13. Chow, K.H., Liu, L.: Robust object detection fusion against deception. In: Proceedings of the 27th ACM SIGKDD Conference on Knowledge Discovery & Data Mining. pp. 2703–2713 (2021)
14. Chow, K.H., Liu, L.: Boosting object detection ensembles with error diversity. In: 2022 IEEE International Conference on Data Mining (ICDM). pp. 903–908. IEEE (2022)
15. Das, N., Shanbhogue, M., Chen, S.T., Hohman, F., Li, S., Chen, L., Kounavis, M.E., Chau, D.H.: Shield: Fast, practical defense and vaccination for deep learning using jpeg compression. In: Proceedings of the 24th ACM SIGKDD International Conference on Knowledge Discovery & Data Mining. pp. 196–204 (2018)
16. Deb, D., Zhang, J., Jain, A.K.: Advfaces: Adversarial face synthesis. In: 2020 IEEE International Joint Conference on Biometrics (IJCB). pp. 1–10. IEEE (2020)
17. Deng, J., Guo, J., Xue, N., Zafeiriou, S.: Arcface: Additive angular margin loss for deep face recognition. In: Proceedings of the IEEE/CVF conference on computer vision and pattern recognition. pp. 4690–4699 (2019)
18. Guo, C., Rana, M., Cisse, M., Van Der Maaten, L.: Countering adversarial images using input transformations. arXiv preprint arXiv:1711.00117 (2017)
19. Hu, S., Liu, X., Zhang, Y., Li, M., Zhang, L.Y., Jin, H., Wu, L.: Protecting facial privacy: generating adversarial identity masks via style-robust makeup transfer. In: Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition. pp. 15014–15023 (2022)
20. Huang, G.B., Ramesh, M., Berg, T., Learned-Miller, E.: Labeled faces in the wild: A database for studying face recognition in unconstrained environments. Tech. Rep. 07-49, University of Massachusetts, Amherst (October 2007)
21. Li, T., Lin, L.: Anonymousnet: Natural face de-identification with measurable privacy. In: Proceedings of the IEEE/CVF conference on computer vision and pattern recognition workshops. pp. 0–0 (2019)
22. Liu, L., Zhou, B., Zou, Z., Yeh, S.C., Zheng, L.: A smart unstaffed retail shop based on artificial intelligence and iot. In: 2018 IEEE 23rd International workshop on computer aided modeling and design of communication links and networks (CAMAD). pp. 1–4. IEEE (2018)
23. Lugaresi, C., Tang, J., Nash, H., McClanahan, C., Uboweja, E., Hays, M., Zhang, F., Chang, C.L., Yong, M.G., Lee, J., et al.: Mediapipe: A framework for building perception pipelines. arXiv preprint arXiv:1906.08172 (2019)
24. Meng, Q., Zhao, S., Huang, Z., Zhou, F.: Magface: A universal representation for face recognition and quality assessment. In: Proceedings of the IEEE/CVF conference on computer vision and pattern recognition. pp. 14225–14234 (2021)
25. Mirza, M., Osindero, S.: Conditional generative adversarial nets. arXiv preprint arXiv:1411.1784 (2014)
26. Ng, H.W., Winkler, S.: A data-driven approach to cleaning large face datasets. In: 2014 IEEE international conference on image processing (ICIP). pp. 343–347. IEEE (2014)
27. Partridge, D., Krzanowski, W.: Software diversity: practical statistics for its measurement and exploitation. Information and software technology 39(10), 707–717 (1997)
28. Pinto, N., Stone, Z., Zickler, T., Cox, D.: Scaling up biologically-inspired computer vision: A case study in unconstrained face recognition on facebook. In: CVPR 2011 WORKSHOPS. pp. 35–42. IEEE (2011)
29. Sahani, M., Nanda, C., Sahu, A.K., Pattnaik, B.: Web-based online embedded door access control and home security system based on face recognition. In: 2015 International Conference on Circuits, Power and Computing Technologies [ICCPCT-2015]. pp. 1–6. IEEE (2015)
30. Schroff, F., Kalenichenko, D., Philbin, J.: Facenet: A unified embedding for face recognition and clustering. In: Proceedings of the IEEE conference on computer vision and pattern recognition. pp. 815–823 (2015)
31. Shan, S., Wenger, E., Zhang, J., Li, H., Zheng, H., Zhao, B.Y.: Fawkes: Protecting privacy against unauthorized deep learning models. In: Proceedings of the 29th USENIX Security Symposium (2020)
32. Stone, Z., Zickler, T., Darrell, T.: Autotagging facebook: Social network context improves photo annotation. In: 2008 IEEE computer society conference on computer vision and pattern recognition workshops. pp. 1–8. IEEE (2008)
33. Sun, Q., Tewari, A., Xu, W., Fritz, M., Theobalt, C., Schiele, B.: A hybrid model for identity obfuscation by face replacement. In: Proceedings of the European conference on computer vision (ECCV). pp. 553–569 (2018)
34. Wang, M., Deng, W.: Deep face recognition: A survey. Neurocomputing 429, 215–244 (2021)
35. Wang, Z., Bovik, A.C., Sheikh, H.R., Simoncelli, E.P.: Image quality assessment: from error visibility to structural similarity. IEEE transactions on image processing 13(4), 600–612 (2004)
36. Wenger, E., Shan, S., Zheng, H., Zhao, B.Y.: Sok: Anti-facial recognition technology. In: 2023 IEEE Symposium on Security and Privacy (SP). pp. 134–151 (2023)
37. Wu, Y., Liu, L., Xie, Z., Chow, K.H., Wei, W.: Boosting ensemble accuracy by revisiting ensemble diversity metrics. In: Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition. pp. 16469–16477 (2021)
38. Xu, W., Evans, D., Qi, Y.: Feature squeezing: Detecting adversarial examples in deep neural networks (2018)
39. Yang, X., Dong, Y., Pang, T., Su, H., Zhu, J., Chen, Y., Xue, H.: Towards face encryption by generating adversarial identity masks. In: Proceedings of the IEEE/CVF International Conference on Computer Vision. pp. 3897–3907 (2021)
40. Zhong, Y., Deng, W.: Opom: Customized invisible cloak towards face privacy protection. IEEE Transactions on Pattern Analysis and Machine Intelligence (2022)
41. Zhu, Z.A., Lu, Y.Z., Chiang, C.K.: Generating adversarial examples by makeup attacks on face recognition. In: 2019 IEEE International Conference on Image Processing (ICIP). pp. 2516–2520. IEEE (2019)


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

### Página 15

![Página 15](original-pages/page-15.png)

### Página 16

![Página 16](original-pages/page-16.png)

### Página 17

![Página 17](original-pages/page-17.png)

</details>
