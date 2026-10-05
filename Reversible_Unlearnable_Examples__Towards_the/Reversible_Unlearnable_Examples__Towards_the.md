# Reversible Unlearnable Examples: Towards the Copyright Protection in Deep Learning Era

Binze Wang, Jinyu Tian, Member, IEEE, Xingrun Wang,

Xiaochen Yuan, Senior Member, IEEE, Jianqing Li, Senior Member, IEEE

**Abstract—Significant advancements in deep learning have been made possible by the utilization of large datasets, underscoring the critical importance of copyright protection. Adding meticulously designed perturbations to examples, making them unlearnable has become a crucial approach for safeguarding data copyright. Existing methods for creating unlearnable examples overlook the risk of data leakage, which can threaten data ownership. Thus, copyright protection in deep learning faces two main threats: illegal model training and malicious data leakage. We investigate that these two threats cannot be solved by straightforwardly combining existing availability attacks and watermarking techniques as their negative interaction effects. Therefore, in this paper, we propose a novel copyright protection mechanism for the aforementioned security concerns. Considering that the prevention of unauthorized model training requires powerful generalizability of unlearnable perturbations, we generate perturbations to induce the model to learn uncorrelated features of input images. It works by minimizing the mutual information of the input and output of the model. On the other hand, to eliminate the side impact of unlearnable perturbations on the watermark extraction, we design a dual extraction strategy by using two distinct watermark extractors. Extensive experiments on the image datasets ImageNet, CIFAR10, and Pets show that our proposed method could provide comprehensive copyright protection to images. The code is available at https://github.com/Yeah21/ReversibleUnlearnableExamples.**

**Index Terms—Copyright protection, reversible unlearnable examples, watermarking technique.**

## I. INTRODUCTION

OPYRIGHT protection has always been one of the key issues in the development of human civilization. As we enter the Internet era, digital image watermarking plays a vital role in copyright protection by validating image ownership against widespread unauthorized manipulation [1] and leakage on online platforms [2]. This technique operates by embedding identification information into images without compromising

This research was partially supported by the National Natural Science Foundation of China (Grant No. 62202009), the Macau Science and Technology Development Fund (Grant Nos. 0040/2023/ITP1 and 0004/2023/RIB1), and the Basic and Applied Basic Research Foundation of Guangdong Province (Grant No. 2024A1515011755). (Corresponding author: Jinyu Tian.)

B. Wang, J. Tian and J. Li are with the School of Computer Science and Engineering, Faculty of Innovation Engineering, Macau University of Science and Technology, Macau, China (e-mail: 3240007529@student.must.edu.mo; jytian@must.edu.mo; jqli@must.edu.mo).
X. Wang is with the School of Computer Science and Artificial Intelligence of FoShan University, FoShan 528225, China (email:wangxingrun@fosu.edu.cn).
X. Yuan is with the Faculty of Applied Sciences of the Macao Polytechnic University, Macau, China (e-mail: xcyuan@mpu.edu.mo).

Copyright © 2025 IEEE. Personal use of this mater a s perm tte . owever, perm ss on to use t s mater a or any ot er purposes must e o ta ne rom i l i i d H i i hi i l f h b b i d f t e y sen ng an ema to pu s-perm ss ons eee.org. h IEEE b di il b i i @i

![](images/image_01.jpg)

<details>
<summary>bar</summary>

| Category | w/o perturbation (%) | w/ perturbation (%) |
| --- | --- | --- |
| MBRS(8) | 0.51 | 0.98 |
| HiDDeN(8) | 0.21 | 7.04 |
| MBRS(16) | 0.64 | 2.91 |
| HiDDeN(16) | 0.39 | 7.51 |
</details>

Fig. 1. Impact of unlearnable perturbations on watermark extraction performance (Bit Error Rate↓). MBRS [8] and HiDDeN [9] are two watermark embedding methods. The value in the bracket is the budget of unlearnable perturbation.

their usability and enables **ownership verification** by extracting watermarks from the protected images [3]–[7].

In the era of deep learning, digital watermarking techniques have expanded their applicability to emerging challenges, including safeguarding model copyrights [10] and preventing unauthorized model replication [11]. However, a formidable new challenge has emerged with the swift proliferation of deep learning techniques. The risk comes up when valuable images might be taken by unauthorized users to train commercial models, seriously hurting the rights of the image owners [12], [13]. These leaked images, typically hidden from public sight, present an insurmountable barrier to traditional watermarking techniques for copyright verification. As a result, protecting the copyright of images should go beyond just confirming ownership and include preserving their intrinsic semantic information to train deep learning models. For convenience, we regard the second type of copyright protection as **semantic information protection**. A commonly employed approach to protect semantic information involves the introduction of imperceptible adversarial perturbations into the protected images. This strategy effectively thwarts models to learn valuable semantic features from the images, which is often referred to as the unlearnable examples [14]–[21].

The two categories of copyright protection issues include ownership verification and semantic information protection, and there are perfect solutions for each type of problem. However, developing a comprehensive copyright protection mechanism that effectively addresses both challenges is undeniably intricate. The reason is that both digital watermarks and unlearnable perturbations subtly alter images, and when the two techniques are combined, the unlearnable perturbations would complicate the extraction of watermark information. For example, as shown in Fig. 1, when the unlearnable perturbations are involved, the performance of the watermarking extraction is degraded significantly, manifested by the increase in Bit Error Rate (BER) from 0.39% to 7.51% (the rightmost bar). This observation shows a straightforward combination of watermarking and unlearnable example methods cannot effectively deal with both ownership verification and semantic information protection at the same time. Therefore, achieving a harmonious balance between the two objectives requires innovative approaches that can mitigate their interference and achieve a mutually beneficial coexistence.

![](images/image_02.jpg)

<details>
<summary>flowchart</summary>

```mermaid
graph TD
  Cloud["Cloud"] -->|Upload| Data["Data"]
  Data -->|Upload| ShareChannel["Share channel"]
  ShareChannel -->|Share Channel| AuthorizedUser["Authorized user"]
  AuthorizedUser -->|Remove| WatermarkedImage["Watermarked image w/o perturbation"]
  WatermarkedImage -->|Train| Training["Training"]
  Training -->|Train| SuccessfulTraining["Successful training"]
  SuccessfulTraining -->|Training| FinalTraining["Training"]
  FinalTraining -->|Extract| LeakedData["Leaked data"]
  LeakedData -->|Extract| Verification["Verification"]
  AuthorizedUser -->|Remove| FinalTraining
```
</details>

Fig. 2. Application scenario involving data owner, authorized user, unauthorized user, and attacker.

Indeed, current approaches have explored combining watermark information with unlearnable perturbation, although they may not directly address the challenge of interference between the two. The GEAA method [22] tackles the problem by embedding watermark information while removing unlearnable perturbation to avoid interference-related problems. This design restricts watermark extraction to recovered data only, failing to verify ownership for perturbed images. The Adv-watermark [23] utilizes a visible watermark as a small fixed noise element and achieves an evasion attack effect by adjusting the watermark’s position and transparency within the image. However, visible watermarks are very easy to attack [24], resulting in protection failure, and the evasion attack performs unsatisfactorily in the task of availability attack.

In this paper, we present a novel protective mechanism designed to address the emerging challenges in copyright infringement within the context of the deep learning era. Our proposed mechanism facilitates the concurrent utilization of watermarking and unlearnable perturbation. Each part functions normally without compromising the effectiveness of the other. Specifically, we focused on a real-world scenario illustrated in Fig. 2, involving four key roles: the data owner, authorized users, unauthorized users, and a potential attacker. The data owner curates a high-quality dataset and intends to make it publicly accessible while retaining verifiable ownership and preventing unauthorized exploitation. To this end, the data owner takes preventive actions by injecting both watermarks and imperceptible perturbations (the left square region). These perturbations render the semantic information within the data unlearnable by machine learning models, acting as a cryptographic mechanism. Subsequently, these protected images are uploaded to cloud services like Google Drive, enabling public access and download. The embedded unlearnable perturbations play a vital role in effectively deterring unauthorized users misuse for training commercial deep models, serving the goal of semantic information protection (the solid red line). On the flip side, when authorized users access the protected images, they can legally train their deep models by removing the perturbations acquired through a secure channel (the solid green line). This requires the designed unlearnable perturbations to be reversible. The class-wise perturbation design inherently ensures that perturbations are label-dependent and uniformly applied across all images within a class, enabling efficient recovery. However, an attacker may steal decryption keys (e.g., via compromised channels or illicit purchase) to remove perturbations, then redistribute clean data, violating both copyright and accessibility control (the dashed black line). Therefore, the watermark should work normally, regardless of whether there are perturbations in the images. Our work tackles the issue by introducing a dual watermarking extractor, which significantly mitigates the interference caused by the unlearnable perturbation, and ensures that we can extract the same watermark successfully with or without unlearnable

perturbations.

In summary, our work concentrates on the crucial modules in the scenario described above, and derives the following novelties and contributions:

• We propose a practical and comprehensive copyright protection framework for image datasets. This innovative framework employs embedded unlearnable perturbations within protected images, thwarting unauthorized users from training a deep model. Significantly, this framework also pioneers a comprehensive ownership verification approach, ensuring that watermarks can be successfully extracted whether the interference of unlearnable perturbations is present or absent.
We propose a novel methodology to generate unlearnable perturbations from the perspective of mutual information. Our motivation revolves around the viewpoint that an effective unlearnable perturbation could push the output of a model independently with the input of this model. Naturally, the model cannot learn useful information from the unlearnable images.
• To enhance the convenience and feasibility of our copyright protection framework, firstly, we develop a classwise universal unlearnable perturbation (it means that images of the same class share the same unlearnable perturbation), allowing for a low-overhead recovery process of semantic information in images. Moreover, we incorporate the widely employed image preprocessing technique, JPEG compression, into our framework, which significantly enhances the robustness of the watermarks and unlearnable perturbations.
• Empirical evaluations conducted on public datasets show that our approach achieves the lowest average accuracy on unauthorized model training and simultaneously manifests a commendable watermark extraction performance. These experiment results demonstrate that the proposed framework achieves the two predefined goals of copyright protection: namely, ownership verification and semantic information protection.

The rest of this paper is organized as follows: Section II reviews the related works on unlearnable perturbation and digital watermarking. Section III presents robustness of perturbations and watermark verification. Experimental results are given in Section IV and Section V concludes the paper. The main notations used throughout this paper with their definitions can be found in Table I.

## II. RELATED WORKS

## A. Unlearnable Perturbation

Generally, the data availability attack involves adding perturbations to training data, leading to networks malfunctioning on the test data. It is a bi-level optimization problem [25] by minimizing the victim model’s cross-entropy loss on the training data while maximizing its classification error on clean test data. Although direct optimization of the objective is intractable for deep neural networks [26], there are some approximate solutions based on gradient matching [18], [27], [28], and Neural Tangent Kernels [29]. To simplify the problem, simple optimization objectives are proposed, including error-minimizing noise [17], [19], [20] and error-maximizing noise [14]. The former perturbations are optimized to reduce the training loss to zero then the target model will have nothing to learn from the data. The latter shows that the common adversarial examples are sufficient to generate powerful unlearnable perturbations. TUE [30] investigated transferable unlearnable examples, improving the robustness and cross-model applicability of perturbations through optimized design. Besides the classification task, UnSeg [31] is a universal unlearnable example generator designed for image segmentation tasks.

TABLE I SUMMARY OF NOTATIONS

| Notation | Description |
| --- | --- |
| $\boldsymbol{x}_{i}$ | original image |
| $\tilde{\boldsymbol{x}}_{i}$ | watermarked image |
| $\hat{\boldsymbol{x}}_{i}$ | protected image (image with watermark and unlearnable perturbation) |
| $\boldsymbol{m}$ | watermark |
| $E$ | encoder |
| $D$ | decoder |
| $\boldsymbol{\eta}_{c}$ | class-wise unlearnable perturbation |
| $C$ | number of classes |
| $\epsilon$ | budget (the limitation of perturbation) |
| $f$ | the victim model for perturbation generation |
| $\mathcal{J}$ | JPEG compression |
| $\mathcal{D}$ | original dataset |
| $\tilde{\mathcal{D}}$ | watermarked dataset |

In order to generate perturbations more quickly, [15] proposed a method of generating perturbations by a linear filter, which does not require any optimization process. Similarly, [16] unveils a linear-separability property of unlearnable noise and proposes the Synthetic Perturbations method to directly synthesize perturbations as effective unlearnable noise.

## B. Digital Watermark

Digital image watermarking aims to embed secret messages into images and then extract them for verification. In recent years, many DNN-based methods [32], [33] have been applied in watermarking for strong robustness. For example, HiDDeN [9] employs an encoder-decoder-based framework for watermarking. StegaStamp [34] focuses on print-shooting watermarks extraction, incorporating several differential attack methods into the noise layer. A two-stage separable framework [33] is proposed to deal with nondifferentiable operations in the noise layer. JPEG compression is a commonly used method for reducing the file size of images, particularly for storage and transmission purposes. MBRS [8] specifically aims to enhance the robustness of the watermark against JPEG compression artifacts and introduces a random noise layer containing three different JPEG noise types into their network. Sun et al. [35] proposed a novel robust image watermarking method in the DCT domain, specifically tailored for embedding and extracting watermarks in images shared on online social networks. Recent advances further explore robustness in screen-shooting scenarios. WaveRecovery [36] leverages wavelet domain encoding and noise recovery decoding to enhance visual quality and extraction accuracy, while [37] simulates perspective-dependent grayscale deviations (GDF) to improve imperceptibility.

![](images/image_03.jpg)

<details>
<summary>flowchart</summary>

```mermaid
graph TD
  subgraph Stage1["Stage 1: Watermark Embedding and Extraction"]
    direction TB
  A["IEEE Watermark m"] --> B["Transposed Conv"]
  B --> C["SE block"]
  C --> D["Encoder E"]
  E --> D
  D --> E["Skip connection"]
  E --> F["ConvBNReLU"]
  F --> G["Watermarked Image x̃"]
    H["Fully connected layer"] -.-> I["Discriminator A"]
    I -.-> J["LA loss"]
  G --> K["Decoder D"]
  K --> L["IEEE Watermark m"]
  end

  subgraph Stage2["Stage 2: Poisoning Attack by Minimizing MI"]
    direction TB
  M["Watermarked Image x̃"] --> N["Perturbation η"]
  N --> O["Victim model"]
  O --> P["MI loss"]
  P --> Q["End Loop? No"]
  Q --> R["CE loss"]
  R --> N
  Q -->|Yes| S["Perturbation η"]
  S --> T["Perturbation η"]
  end

  subgraph Stage3["Stage 3: Dual Watermark Extraction"]
    direction TB
  U["Protected Image x̃"] --> V["Dual Decoder Dd"]
  V --> W["IEEE Watermark m"]
  W --> X["Dual Decoder Dd"]
  X --> Y["Protection Process"]
  Y --> Z["ConvBNReLU"]
  Z --> AA["SE block"]
  AA --> AB["Forward"]
  AB --> AC["Frozen"]
  AC --> AD["Backward"]
  AD --> AE["Update"]
  AE --> AF["Protection Process"]
  AF --> AG["ConvenBNReLU"]
  AG --> AH["SE block"]
  end
```
</details>

Fig. 3. The pipeline of the proposed copyright protection framework.

## III. PROPOSED METHOD

In this paper, we redefine the goals of copyright protection for data in the deep learning era, including **ownership verification** and **semantic information protection**. Recalling the description of Fig. 2, in our considered application scenario, the data owner’s images face two different aspects of copyright infringement. On one hand, unauthorized users seek to employ these images for training commercial deep models, leading to the infringement of semantic information. On the other hand, the attacker maliciously leaks unprotected data which are without unlearnable perturbations. In this scenario, we need to embed unlearnable perturbations to prevent unauthorized training and watermarks to verify the ownership of images. The watermarks in the protected images should be extractable both before and after the removal of the perturbations. This unique feature sets our proposed copyright protection framework apart from existing studies. We now delve into the comprehensive pipeline of our methodology.

## A. Outline of the Proposed Method

The scheme of our copyright protection methodology follows the diagram in Fig. 3, which consists of the watermark embedding and extraction (upper half), the unlearnable perturbation generation process (lower left half), and dual watermark extraction (lower right half). In the watermark embedding and extraction part, the encoder E and decoder D are trained to process the watermark. Specifically, the encoder embeds a given watermark m into the original images ${ \pmb x } _ { i } { } ^ { \flat } { \bf S }$ and results in their watermarked counterparts $\tilde { \pmb { x } } _ { i }   ^ { \prime } \mathbf { s }$ . Correspondingly, the decoder extracts the watermark information m from the watermark images $\tilde { \pmb { x } } _ { i } { } ^ { \flat } \mathbf { S } .$ . Furthermore, to enhance the watermarked image quality, a discriminator is incorporated during training, enabling adversarial optimization. Unlearnable perturbation generation is an iterative optimization process as follows. Firstly, fix the perturbations and minimize the cross-entropy loss to update the victim model parameters. Secondly, fix the victim model parameters and minimize mutual information objectives to optimize the perturbations. These two steps are repeated until convergence, resulting in class-wise unlearnable perturbations. As we emphasized before, our proposed framework has the unique feature of verifying the ownership of images both with and without the interference of the unlearnable perturbation. We should extract watermark m from both the watermarked image x˜ and the protected image xˆ. Note that the decoder D is jointly trained with the encoder E over original images ${ \pmb x } ^ { \prime } { \bf S } .$ It generally can only extract the watermark from the watermarked image $\tilde { x } _ { i }$ . Therefore, we design a dual decoder $D _ { d }$ to extract a watermark from the protected image $\hat { x } _ { i }$ , which has an unlearnable perturbation $\eta _ { c }$ as the interference factor compared with $\tilde { x } _ { i }$ . We call the designed encoder and decoder structure dual watermark extraction, as two decoders seek to reveal the same information from a common encoder. In the Fig. 3, we highlight the process of obtaining protected images with perturbations and watermarks from clean images with yellow thick arrows. Specifically, for each image $x _ { i }$ of the c-th class, we obtained its protected version ${ \hat { x } } _ { i }$ by first embedding watermark m using the encoder E and then adding the $c -$ th class perturbation $\eta _ { c }$ on it, $i.e., $\hat { \pmb { x } } _ { i } = \pmb { E } ( \pmb { x } _ { i } ) + \pmb { \eta } _ { c }$$ . When ownership verification is required, given the undetectability of imperceptible perturbations $\eta _ { c } ,$ we deploy dual watermark extractors operating in parallel. Decoder D processes the input image under the perturbation-free assumption, yielding extracted watermark $m ^ { \prime }$ . Dual Decoder $D _ { d }$ processes the same input under the perturbation-present assumption, outputting $m ^ { \prime \prime }$ . Ownership is established if either extracted watermark matches the original embedded watermark m.

For generating the class-wise unlearnable perturbations $\eta _ { c } { } ^ { \prime } \mathrm { s } ,$ we alternately train a victim model $f$ and search the perturbations $\eta _ { c } { } ^ { \prime } s$ . The training process is a competition between the model $f$ that aims to learn semantic information from the protected images $\hat { \pmb { x } } _ { i } { } ^ { \prime } \mathbf { s }$ and the unlearnable perturbations $\eta _ { c } { } ^ { \prime } s$ that struggle to impede the learning process of model $f .$ The model generally fulfills the learning process by minimizing the classification error, typically the cross-entropy. To impede the learning process of model $f ,$ we propose a novel **Unlearnable Examples by Minimizing the Mutual Information (MI)** between the model input and output. Since unauthorized models usually differ from the ones used to create unlearnable examples, these examples need to have good transferability. We will explore the proposed MI in Section III-C.

We alternate training watermarks and perturbations. First, a network embeds and extracts watermarks, generating watermarked images. Then, we apply unlearnable perturbations to the watermark image. Finally, the dual watermark extractor is trained on images with both perturbation and watermark. Now, we discuss the watermark generation process and the unlearnable perturbation generation process, respectively.

The training process for generating watermarks and unlearnable perturbations involves several steps. First, we train a network with an encoder and decoder to produce watermarked images. Next, we apply unlearnable perturbations to these watermarked images using our proposed MI method. Finally, we train the dual watermark extractor on images with both perturbations and watermarks. In the following sections, we will delve into the details of how watermarks and unlearnable perturbations are generated.

## B. Watermark Embedding and Extraction

At present, the digital watermark method based on the neural network commonly contains an encoder and a decoder. The encoder E inputs watermark information m and the original image x<sub>i</sub>, outputs watermarked image $\tilde { x } _ { i } ;$ The decoder D extracts the watermark information $m ^ { \prime }$ from ${ \tilde { x } } _ { i }$ . The objective of optimizing network parameters is to ensure high image quality and high accuracy of watermark extraction. To evaluate the image quality, a straightforward selection is the Mean Square Error (MSE) between the watermarked image $\tilde { x } _ { i }$ and the original image $x _ { i }$ . However, the work [38] points out that the similarity perceived by human eyes is not completely consistent with the pixel-level similarity index, and it is more appropriate to use semantic feature extraction by the networks. Therefore, we use a discriminant network to ensure the visual effect further. As for the extraction of watermark information, we treat this task as a binary classification problem where the watermark extraction network should correctly predict each bit of the watermark. This suggests we measure the difference between the embedded and the extracted watermarks using Binary Cross Entropy (BCE). To sum up, the loss of watermark embedding is as follows:

$$
\mathcal {L} _ {1} = \operatorname{MSE} \left(\tilde {\boldsymbol {x}} _ {i} - \boldsymbol {x} _ {i}\right) + \log \left(\boldsymbol {A} \left(\tilde {\boldsymbol {x}} _ {i}\right)\right) + \operatorname{BCE} \left(\boldsymbol {m}, \boldsymbol {m} ^ {\prime}\right), \tag {1}
$$

where

$$
\tilde {\boldsymbol {x}} _ {i} = \boldsymbol {E} \left(\boldsymbol {x} _ {i}, \boldsymbol {m}; \boldsymbol {\theta} _ {\boldsymbol {E}}\right), \boldsymbol {m} ^ {\prime} = \boldsymbol {D} \left(J \left(\tilde {\boldsymbol {x}} _ {i}\right); \boldsymbol {\theta} _ {\boldsymbol {D}}\right), \tag {2}
$$

$\theta _ { E } ,   \theta _ { D }$ are parameters of the encoder and decoder, respectively, and A represents the discriminator. Here, we involve a differentiable JPEG module J [39] to enhance the robustness of watermark extraction against the compression preprocess.

Our watermark embedding and extraction modules are based on the MBRS [8]. The message is reshaped and amplified through a $3 \times 3$ convolution layer, features are expanded and extracted using transposed convolution layers and Squeezeand-Excitation (SE) block [40]. In the encoding process, the cover image undergoes similar amplification and feature extraction, and the resulting features from both the message and cover image are concentrated and mapped through convolutional layers to produce the watermarked image. The decoding process entails amplifying a watermarked image, channel-wise feature conversion, and reshaping to retrieve the decoded message. The discriminator consists of five convolutional blocks followed by a fully connected layer. Each block includes a $3   \times   3$ convolution layer, Batch normalization, and ReLU activation. Fully connected layer with output dimension 1, using sigmoid activation to predict whether the input is a watermarked or original image. Now, we can obtain the watermark encoder E and decoder D by minimizing the integration of $\mathcal { L } _ { 1 }$ . A dual watermark extractor is mentioned in Section III-D to extract watermarks from images with perturbation better.

## C. The Generation of Unlearnable Perturbations

Upon having watermarked images $\tilde { \pmb { x } _ { i } } ^ { \prime } \mathrm { s } ,$ we then generate class-wise unlearnable perturbations $\eta _ { c } { } ^ { \prime } \mathrm { s }$ to obtain the protected images $\hat { \pmb { x } } _ { i }   ^ { \prime } \mathbf { S } .$ In our proposed copyright protection framework, we propose a novel information theory-based unlearnable example method. We observe that existing methods [14]–[18] highly rely on the classification information of the victim model, thereby causing the classification results to become inconsistent with the true labels. Geometrically, perturbations shift normal samples in the direction normal to the decision boundary, enabling minimal alterations to cross the boundary [41], [42]. However, since the decision boundary varies significantly across different models [43], [44], this results in poor generalization of perturbations. Consequently, error-maximization strategies generate perturbations tailored exclusively to the decision boundaries of a specific welltrained victim model.

![](images/image_04.jpg)

<details>
<summary>flowchart</summary>

```mermaid
graph LR
  A["Clean"] --> B["Perturbation"]
  C["Poisoned"] --> D["ResNet18"]
  E["Perturbation"] --> F["VGG19"]
  G["Perturbation"] --> H["MobileNetV2"]
  I["Perturbation"] --> J["GoogleNet"]
  K["Perturbation"] --> L["Dense121"]
```
</details>

Fig. 4. The CAM visualization of the clean model and the poisoned model.

We overcome above limitation by exploiting the commonalities observed across models during training. Specifically, existing research indicates that different models tend to extract similar features from the same image dataset [45]–[49]. As shown in the 1-st row of Fig. 4, our CAM visualizations reveal that diverse models rely on identical semantic features (such as fish texture and contour information). This inspires us that if a deliberately crafted perturbation added to an image can dominate the features extracted by models, it will disrupt their ability to capture semantic information, thus resulting in a transferable poisoning effect.

To achieve effective intervention in the dependency between image semantic information and labels, a suitable metric is required to serve as the optimization objective. Mutual information (MI) is an ideal measure for quantifying the statistical dependence between two random variables. Intuitively, it represents the reduction in uncertainty of one variable when the other is known. The MI is calculated as

$$
I (X; Y) = H (Y) - H (Y | X), \tag {3}
$$

where the entropy $H ( Y )$ measures the inherent uncertainty of variable $Y ,$ and the conditional entropy $H ( Y | X )$ captures the remaining uncertainty in $Y$ given knowledge of X. A smaller MI $I ( X ; Y )$ indicates that even when X is known, $Y$ remains highly uncertain. This aligns perfectly with our objective: we aim to introduce a perturbation η to obscure the semantic information in images $X .$ , preventing the model from establishing a reliable association between X and the label Y . From an information-theoretic perspective, perturbing the image to $X + \eta$ increases the difficulty for the model to determine the label, which is reflected in an increase in conditional entropy: $I ( X + \eta ; Y ) = H ( Y ) - H ( Y | X + \eta ) < H ( Y ) - H ( Y | X ) =$ $I ( X ; Y )$ . Thus, we adopt a mutual information minimization strategy to weaken the dependence between image features and labels, pushing the learned features to be uncorrelated with the image semantic information. The 2-nd row of Fig. 4 demonstrates that when perturbations (ResNet18 as the victim model) are added, all models shift attention exclusively to the perturbed regions, fundamentally disrupting semantic feature extraction. This prevents any model from extracting features correlated with semantic content, making the data fundamentally ”unlearnable”.

Formally, suppose we have obtained the watermarked versions $\tilde { \pmb { x } } _ { i }   ^ { \prime } \mathbf { s }$ of all training images ${ { x } _ { i } } ^ { \prime } \mathbf { { s } } .$ Let $\eta _ { i } { } ^ { \prime } \mathbf { s }$ be the unlearnable perturbation corresponding to each ${ \tilde { x } } _ { i } .$ . Considering the output $\hat { \pmb { y } } _ { i }$ of the victim model $f$ with respect to the input $\hat { \pmb x } _ { i } ,   \mathrm { i . e . , }   \hat { \pmb y } _ { i }   =   f ( \hat { \pmb x } _ { i } )$ , we have the set $\hat { Y }$ of the outputs $\hat { \mathbf { y } _ { i } } ^ { \prime } \mathbf { S } .$ According to the discussion above, we can find unlearnable perturbations $\eta _ { i } { } ^ { \prime } \mathrm { s }$ by minimizing the mutual information between $\hat { X }$ and $\hat { Y }$ . That is,

$$
\min _ {\boldsymbol {\eta} _ {i}} I (\hat {X}; \hat {Y}), \quad s. t. \quad \| \boldsymbol {\eta} _ {i} \| _ {\infty} <   \epsilon , \tag {4}
$$

where $\hat { X }$ is parameterized by the optimization variables $\eta _ { i } { } ^ { \prime } \mathrm { { S } } .$ Following with equation (9) in the reference [50], we can derive the subsequent procedure. According to the definition of mutual information, $I ( \hat { X } ; \hat { Y } ) = H ( \hat { Y } ) \stackrel { \smallsmile } { - } H ( \hat { Y } | \hat { X } )$ , where $H ( \hat { Y } )$ is the entropy of model output $\hat { Y }$ , and $H ( { \hat { Y } } | { \hat { X } } )$ is the conditional entropy of output given input $\hat { X } ,$ . In deterministic models (like neural networks), the output $\hat { Y }$ becomes deterministic when input $\hat { X }$ is given, i.e., $H ( { \hat { Y } } | { \hat { X } } ) = 0 .$ . Therefore, followed with [50], we can derive $I ( \widehat { X } ; \widehat { Y } ) \; = \; H ( \hat { Y } )$ . The optimization objective simplifies to

$$
\min _ {\boldsymbol {\eta} _ {i}} H (\hat {Y}), \quad s. t. \quad \| \boldsymbol {\eta} _ {i} \| _ {\infty} <   \epsilon . \tag {5}
$$

$H ( \hat { Y } ) = - \textstyle \sum _ { j } ( p _ { j } \operatorname { l o g } p _ { j } )$ , where $p _ { j }$ is the model’s predicted probability for the j-th class. Based on the above derivation, we can reformulate Equation (4) as:

$$
\min _ {\| \boldsymbol {\eta} _ {i} \| _ {\infty} <   \epsilon} \mathbb {E} _ {\tilde {\boldsymbol {x}} _ {i} \in \tilde {\mathcal {D}}} \{- f (J (\tilde {\boldsymbol {x}} _ {i} + \boldsymbol {\eta} _ {i})) \log f (J (\tilde {\boldsymbol {x}} _ {i} + \boldsymbol {\eta} _ {i})) \}. \tag {6}
$$

Here we also involve a differentiable JPEG module [39] as the generation of watermarks to enhance the performance of the unlearnable perturbations in practice. To solve the problem (6), we adopt the first-order method as follows:

$$
\boldsymbol {\eta} _ {i} = - l _ {\eta} \nabla_ {\tilde {\boldsymbol {x}} _ {i}} \mathcal {L} _ {p} \left(\tilde {\boldsymbol {x}} _ {i} + \boldsymbol {\eta} _ {i}\right), \tag {7}
$$

where $\mathcal { L } _ { p }$ refers to the objective function in (6). The obtained perturbation then will be clipped by CLIP function so that $\| \eta _ { i } \| _ { \infty } < \epsilon .$

In practice, all unlearnable perturbations $\eta _ { i }$ will be transferred to the authorized user via a secure channel. To mitigate the transmission overhead, we adopt the class-wise perturbation by taking the expectation of those $\eta _ { i }$ of $\tilde { \pmb { x } } _ { i }   ^ { \prime } \mathbf { s }$ from the same class. Thus, we can obtain $C$ perturbations as follows:

$$
\boldsymbol {\eta} _ {c} = \frac {1}{| \tilde {\mathcal {D}} _ {c} |} \sum_ {\tilde {\boldsymbol {x}} _ {i} \in \tilde {\mathcal {D}} _ {c}} \boldsymbol {\eta} _ {i}, \tag {8}
$$

where $\tilde { \mathcal { D } } _ { c }$ contains all watermarked images of the c-th class.

The generation of unlearnable perturbation using our proposed method is summarized in Algorithm 1. We train the victim model $f$ over the protected dataset by minimizing the cross-entropy loss within $T$ epochs (lines 2-5). Upon having a temporary victim model, we then generate unlearnable perturbations for each watermarked image (lines 6-10). These perturbations will be summarized according to labels of watermarked images, resulting in class-wise unlearnable perturbations $\pmb { \eta } _ { c } \quad ( c = 1 , . . . , C )$ (lines 11-13). The above procedures will be repeated until the classification accuracy AC of the victim model achieves a stop criterion e.

Algorithm 1 The Proposed Unlearnable Examples

<div class="docvortex-algorithm" style="white-space: pre-wrap; font-family:monospace;">
Input: watermarked images $\tilde{\mathcal{D}}$, stop criterion $e$, train step $T$, learning rates $l_{\theta}, l_{\eta}$
Output: class-wise unlearnable perturbation $\boldsymbol{\eta}_{c}$ ($c = 1, ..., C$)
Initialize: $\boldsymbol{\eta}_{c} = \boldsymbol{0}(c = 1, ..., C)$
while $AC &lt; e$ do
    ##### training the victim model $f$ #####
    for $t = 1, ..., T$ do
        update the parameters of the model $f$ over the dataset $\tilde{\mathcal{D}}$ with perturbation $\boldsymbol{\eta}$.
    end for
    ### generating the class-wise perturbation ###
    for $\tilde{\boldsymbol{x}}_{i} \in \tilde{\mathcal{D}}$ do
        calculate $\boldsymbol{\eta}_{i}$ according to equation (7).
        $\boldsymbol{\eta}_{i} = \text{CLIP}(\boldsymbol{\eta}_{i}, -\epsilon, \epsilon)$.
    end for
    for $c = 1, ..., C$ do
        calculate $\boldsymbol{\eta}_{c}$ according to equation (8).
    end for
    calculate the classification accuracy $AC$ over $\tilde{\mathcal{D}}$ with $\boldsymbol{\eta}$.
end while
</div>

Our core motivation posits that MI-minimization perturbations fundamentally disrupt the model’s semantic information extraction from images. To validate this, we quantitatively measure the mutual information $I ( { \hat { X } } ; { \hat { Y } } )$ between poisoned inputs $\hat { X }$ and model outputs $\hat { Y }$ across diverse architectures. The perturbations were generated through mutual information minimization on the ImageNet-100 dataset using ResNet18 as the victim model. Using the mutual information of normal models on clean data as the baseline, MI decreased by 97% for ResNet18, 82% for VGG19, 89% for MobileNet, 83% for GoogleNet, and 97% for Dense121, respectively. All models exhibit drastic MI reduction on poisoned data compared with clean data. Consequently, despite perturbations being optimized only for ResNet18, MI values remain uniformly low across all architectures.

## D. Dual Watermark Extraction

Due to the interference of the unlearnable perturbation $\eta ,$ we use a dual decoder $D _ { d } ,$ the same network structure as the watermark decoder D, to extract the same watermarks from the watermarked images with the unlearnable perturbation. The watermark extracted by $D _ { d }$ naturally is measured by

$$
\mathcal {L} _ {2} = \mathrm{BCE} \left(\boldsymbol {m}, \boldsymbol {m} _ {d} ^ {\prime}\right), \boldsymbol {m} _ {d} ^ {\prime} = \boldsymbol {D} _ {d} \left(J \left(\tilde {\boldsymbol {x}} _ {i} + \boldsymbol {\eta} _ {i}\right); \boldsymbol {\theta} _ {\boldsymbol {D} _ {d}}\right), \tag {9}
$$

where $m _ { d } ^ { \prime }$ is the watermark extracted by the dual decoder $D _ { d } ,$ $\eta _ { i }$ is the unlearnable perturbation, and $\theta _ { D _ { d } }$ are parameters of the dual decoder. Now the complete watermark loss function consists of $\mathcal { L } _ { 1 }$ and ${ \mathcal { L } } _ { 2 } ,$ and we designed and compared different optimization strategies in Section IV-D.

## E. Reversibility of Unlearnable Example

Perturbation plays a crucial role in protecting semantic information. After removing the perturbation, the image only contains watermark information, and the model’s performance trained on the watermark image is normal. Dataset learnability is reversible for authorized users, which is achieved through class-wise perturbation design. For each class $c \in \{ 1 , . . . , C \}$ a universal perturbation $\eta _ { c }$ is generated and applied uniformly to all watermarked images $\tilde { x } _ { i }$ belonging to class c. Formally, the protected image is constructed as $\hat { \pmb { x } } _ { i }   =   \tilde { \pmb { x } } _ { i } + \pmb { \eta } _ { c } ,$ where $c = y _ { i }$ . The class-wise perturbations $\{ \eta _ { c } \} _ { c = 1 } ^ { C }$ are transmitted to authorized users via a secure channel. Authorized users possessing both the perturbations and the corresponding class labels $y _ { i }$ can recover the original watermarked images by removing the perturbation: $\tilde { \pmb { x } } _ { i } = \hat { \pmb { x } } _ { i } - \pmb { \eta } _ { c }$

## IV. EXPERIMENTS

Our proposed method is a copyright protection mechanism that could effectively prevent the image dataset from unauthorized training (i.e., semantic information protection) and simultaneously avoid images with or without perturbation leakage by authorized users (i.e., ownership verification). To the best of our knowledge, our work is the first one that considers such a comprehensive copyright protection scenario. We select two recent studies closest to our scenario to assess the effectiveness of our proposed methodology: GEAA [22] and Adv-watermark [23]. The GEAA considers recovered data leakage caused by authorized users and Adv-watermark adopts the visible watermarks which also function as an unlearnable perturbation. The evaluation of our proposed method for tackling the above two threats is carried out on the ImageNet-100 dataset, covering the first 100 categories of ImageNet [51], CIFAR10 [52], and Oxford-IIIT Pets [53].

Ours’ protected data generation includes three parts in order: 1) training the watermarking network including an encoder and a decoder, generating high-quality watermarked images; 2) unlearnable perturbation generation; 3) training the dual decoder to extract the watermark from the images combining the watermarked images and perturbations. The watermark performance and ablation study are detailed in Section IV-B and IV-D, and the unlearnable perturbation performance and robustness analysis are shown in Section IV-A, IV-E and IV-F.

## A. Semantic Information Protection

For the task of semantic information protection, we train the different networks on the protected dataset and test on the clean test dataset. As shown in Table II, we can see that the accuracy of five considered models (the 6-th row of columns 3-8) is degraded closely aligning with the worst accuracy of a 100-class classification task, the 1%. Moreover, the values in parentheses are the accuracy of models trained over the images after removing the unlearnable perturbations. They are quite close to the ideal accuracy of models trained over clean images. In comparison, compared to the original accuracy, GEAA merely exhibited a modest reduction to 42.35% on average and a 6% decrease during authorized training (the 4-th row of columns 3-8). Adv-watermark also performs unsatisfactorily by merely degrading the original accuracy to 58.18% (the 5- th row of columns 3-8). Since the watermarks (or say the unlearnable perturbations) generated by Adv-watermark are irreversible, the authorized training results are not presented. Note that our class-wise unlearnable perturbation is exactly reversible and the slight difference in accuracy between the original model and the authorized training model is caused by the embedded watermarks.

TABLE II THE PERFORMANCE OF PROPOSED COPYRIGHT PROTECTION MECHANISM. THE VALUES IN THE METHOD COLUMN’S BRACKET ARE THE INTENSITY OF UNLEARNABLE PERTURBATIONS. THE RESULTS ARE FORMATTED AS UNLEARNABLE ACCURACY (RECOVERY ACCURACY) IN THE SEMANTIC INFORMATION PROTECTION PART AND BER(PSNR) IN THE TASK OF OWNERSHIP VERIFICATION. THE NOTATION “– –” MEANS THIS MEASURE DOES NOT APPLY TO THIS METHOD.

<table><tr><td rowspan="2">Dataset</td><td rowspan="2">Method</td><td colspan="6">Semantic Information Protection (%)</td><td colspan="2">Ownership Verification</td></tr><tr><td>ResNet18</td><td>VGG19</td><td>MobileNetV2</td><td>GoogLeNet</td><td>Dense121</td><td>Avg.</td><td>BER w/</td><td>BER w/o</td></tr><tr><td rowspan="6">ImageNet-100</td><td>Original</td><td>71.39</td><td>65.09</td><td>76.80</td><td>77.13</td><td>73.50</td><td>72.78</td><td>N/A</td><td>N/A</td></tr><tr><td>GEAA [22]</td><td>40.68(67.42)</td><td>41.88(63.30)</td><td>40.32(62.58)</td><td>47.76(71.64)</td><td>41.12(68.38)</td><td>42.35(66.66)</td><td>-- (9.23)</td><td>26.46%(34.77)</td></tr><tr><td>Adv-watermark [23]</td><td>59.94(--)</td><td>46.88(--)</td><td>60.94(--)</td><td>61.84(--)</td><td>61.30(--)</td><td>58.18(--)</td><td>-- (28.44)</td><td>-- (--)</td></tr><tr><td>Ours(4)</td><td>8.96(69.72)</td><td>2.06(65.69)</td><td>6.67(76.32)</td><td>8.50(76.41)</td><td>7.46(73.04)</td><td>6.76(72.23)</td><td>0.16%(41.25)</td><td>0.22%(48.36)</td></tr><tr><td>Ours(8)</td><td>5.46(69.98)</td><td>2.64(66.44)</td><td>1.28(75.38)</td><td>3.00(75.44)</td><td>6.46(72.36)</td><td>3.77(71.92)</td><td>0.27%(35.54)</td><td>0.26%(48.35)</td></tr><tr><td>Ours(16)</td><td>2.03(69.42)</td><td>1.30(65.92)</td><td>1.16(73.12)</td><td>1.52(75.66)</td><td>1.98(72.60)</td><td>1.60(71.34)</td><td>0.32%(30.74)</td><td>0.35%(48.21)</td></tr><tr><td rowspan="2">Pets</td><td>Original</td><td>76.27</td><td>50.37</td><td>70.79</td><td>81.54</td><td>75.73</td><td>70.94</td><td>N/A</td><td>N/A</td></tr><tr><td>Ours(8)</td><td>2.70(73.36)</td><td>2.70(47.48)</td><td>2.70(70.67)</td><td>2.70(78.30)</td><td>2.70(73.02)</td><td>2.70(68.56)</td><td>0.25%(37.07)</td><td>0.08%(48.80)</td></tr><tr><td rowspan="3">CIFAR10</td><td>Original</td><td>94.49</td><td>90.23</td><td>84.86</td><td>87.42</td><td>94.46</td><td>90.29</td><td>N/A</td><td>N/A</td></tr><tr><td>GEAA [22]</td><td>46.39(81.63)</td><td>45.18(77.71)</td><td>41.98(64.98)</td><td>44.66(54.55)</td><td>49.54(72.84)</td><td>45.55(70.34)</td><td>-- (16.21)</td><td>28.86%(36.28)</td></tr><tr><td>Ours(4)</td><td>10.50(94.43)</td><td>11.68(90.10)</td><td>11.50(84.26)</td><td>14.01(86.96)</td><td>11.26(94.05)</td><td>11.79(89.96)</td><td>2.03%(34.69)</td><td>1.29%(38.27)</td></tr></table>

Besides, it is important to mention that all five models are trained using the same unlearnable dataset obtained from ResNet18 and their accuracy is consistently degraded. This reflects the powerful generalizability of our proposed method, ensuring that we can successfully protect the semantic information of images in real scenarios where the models of authorized users are generally unknown.

The 7-8 rows in Table II show the performance of the semantic information protection using unlearnable perturbation with $L _ { \infty }$ magnitude 8 pixels and 16 pixels, respectively. We can derive the same conclusion as the case of 4 pixels $L _ { \infty }$ magnitude, which reflects that our method does not rely on specific choices of perturbation magnitudes. In a nutshell, the results on ImageNet-100 support the effectiveness of our method of semantic information protection. Besides, we evaluate the proposed method on Pets, the 9-10 rows of Table II, and five models trained on protected Pets data all achieve near-random accuracy 2.7%. For authorized users, we recover 97% of the original accuracy (68.56% vs. 70.94%). A similar conclusion can be derived on the CIFAR10.

In the comparison methods, GEAA [22] is only tested on the CIFAR10 in the original paper, and we use the open-source code setting the same watermark capacity as ours to test on the ImageNet-100. Images were resized to 224 × 224. We trained GEAA for 15 phases, with each phase containing 30 epochs (batch size=8). At each phase start, we reset the learning rate to 0.0001 and decayed it by 0.5 after 10 epochs (lower boundary constraint=10). For Adv-watermark [23], we attacked ResNet-101 using the Berkeley University badge (scale=1/4) with crossover probability=0.9, iterations=3, and population size=50. we attack the 130000 images and run this experiment in 10 days with CPU I9-13900K + GPU 4090 configuration. Because the watermark picture is too large, it cannot be implemented in CIFAR10. More details about training are discussed in Section IV-E .

## B. Ownership Verification

Now we evaluate the effectiveness of our method on the task of ownership verification. We use the Bit Error Rate (BER) metric to assess watermark extraction efficiency. Our work focuses on two types of ownership verification, i.e., the watermark extraction with unlearnable perturbations (denoted by BER w/) and that without perturbations (denoted by BER w/o). Also, we evaluated the quality of the protected images by using the Peak Signal to Noise Ratio (PSNR). It is noteworthy that the watermark is tested with JPEG compression with a widely used quality factor of 75 to simulate the real application scenario. It is worth noting that we do not assume the protector knows whether watermarked images have unlearnable perturbations. If either extractor can extract the information correctly, we achieve copyright verification.

Table II records the performance of ownership verification of images in ImageNet-100, where a random 64-bit message is embedded into each image with a resolution of 256 × 256, respectively. As can be seen from the last two columns, our proposed method attains a BER of 0.16% and 0.22% (the 6-th row of the last two columns) with and without unlearnable perturbation, respectively, while maintaining high image quality. In comparison, GEAA performs unsatisfactorily with BER w/o of 26.46% (the 4-th row of the last column). Notably, the GEAA framework sacrifices visual quality to protect the semantic information of the dataset. Adv-watermark, being a visible watermark attack method, is evaluated solely based on PSNR. With the same setting, we test the watermark robustness on Pets. With perturbations present, BER is 0.25%.

![](images/image_05.jpg)

Fig. 5. Clean images (Top) VS. Images with the watermark and perturbation (Bottom).

TABLE III THE TEST ACCURACY (%) OF MODELS TRAINED BY EASYDL PLATFORMS. THE TRAINING CONFIGURATION ON THE PLATFORM WAS SET TO “FASTEST TRAINING”.

| Data | CIFAR10 | ImageNet10 | Pets |
| --- | --- | --- | --- |
| Clean | 92.20 | 77.00 | 67.03 |
| Emax [14] | 18.88 | 20.60 | 20.58 |
| Emin [20] | 19.47 | 23.80 | 37.24 |
| Ours | 16.33 | 13.80 | 18.14 |

Without perturbations, the BER improves to 0.08%. The effectiveness of watermark extraction is also corroborated by results on CIFAR10 (bottom part of the table), where the 32-bit message was embedded in each 32 × 32 size image, respectively.

From the last two columns, we can observe that the magnitudes of unlearnable perturbations do not impact the performance of watermark extraction, which provides us with a flexible selection of perturbation magnitudes. Some visualization results are presented in Fig. 5. It is evident that the integrated watermark and perturbation have a minimal impact on the visual quality of images, which highlights the practicality of our method. There are more visualization results in Fig. 9, where the images from left to right include clean images, images with perturbation generated by Adv-watermark, and images with and without perturbation by GEAA and Ours. There are PSNR and SSIM values below each image.

## C. Real-World Validation

To test the practicality in real-world scenarios, we conducted experiments on commercial API platforms for deep learning model training. In this setting, the model architectures and training strategies are unknown, simulating a realistic environment where unauthorized users might attempt to exploit the protected dataset. Following established methodology [21], we test robustness on Baidu EasyDL, a widely used commercial deep learning platform. Using ”fastest training” mode configurations, we evaluate three protected datasets: ImageNet10, CIFAR10, and Pets, and results are shown in Table III. Our method degrades accuracy to 13.80-18.14%, significantly below the unprotected performance.

TABLE IV WATERMARK PERFORMANCE.

| Method | $\epsilon$ | BER w/ | BER w/o | PSNR | SSIM |
| --- | --- | --- | --- | --- | --- |
| single | 8 | 0.98% | 0.50% | 48.99 | 0.9950 |
| dual | 8 | 0.38% | 0.19% | 45.82 | 0.9897 |
| $dual_{f}$ | 8 | 0.51% | 0.33% | 48.21 | 0.9938 |
| single | 16 | 2.90% | 0.64% | 48.99 | 0.9950 |
| dual | 16 | 1.33% | 0.34% | 45.82 | 0.9897 |
| $dual_{f}$ | 16 | 0.81% | 0.34% | 48.21 | 0.9938 |

## D. Ablation Study

To empirically demonstrate the disruptive impact of embedding unlearnable perturbations on the watermark extraction process, we consider three methods of perturbation and watermark embedding and test on the validation set of ImageNet-100 containing 5000 images. The first one, denoted as **single**, is a straightforward combination of embedding the watermark followed by the unlearnable perturbation. The watermark extraction process before and after removing the perturbation uses the same extractor. Take the 2-nd row of Table IV as an example, we can see that extraction performances are highly impacted by the perturbation by increasing the BER from 0.50% to 0.98%. The second considered method is our proposed dual extractor, referred to as **dual**, which involves two independent extractors. Compared with the **single** design, our proposed method significantly eliminates the impact of unlearnable perturbations, where the perturbations merely increase the BER from 0.19% to 0.38% (the 3-rd row). Even when the perturbation magnitudes intensify to 16, the impact remains relatively subtle, degrading from 0.34% to 1.33% (the 6-th row). We observe that the dual extractor design moderately deteriorates the image qualities. This is because the encoder of watermarks has to embed more information into the images so that another extractor can successfully retrieve watermarks undergoing the impact of perturbations. To prioritize high image quality, we can freeze the pre-trained watermark encoder module by merely training another extractor to retrieve watermarks from watermarked images with perturbations. We denote this strategy as **dual**f . Compared with the **dual** design, the **dual**f strategy has an improvement in the image quality from PSNR 45.82 to 48.21 (the 5-th column of rows 6-7) with a slight cost of BER degradation from 0.81% to 1.33% (the 3-rd column of rows 6-7).

In our method, the embedding message is randomly generated for each batch in the training process, but the comparison method GEAA only embeds the pre-defined 10 kinds of messages. The training epoch is 100 and the batch size is 32, the best network parameters are good image quality and strong watermark robustness on the validation set. The training process is shown in Fig. 6. The training process of **dual** improves image quality but degrades robustness. In contrast, **dual**f trains the decoder in the case of freezing the encoder parameters, ensuring that the image quality is high when the watermark is strong robustness. The **dual**f method successfully extracts dual watermark information without compromising image quality.

To further prove the necessity and effectiveness of the proposed method, we consider a straightforward alternative by placing the watermark and perturbation in non-intersecting locations within the image. We split the images of CIFAR10 into two equal patches, and then embed watermarks and perturbations into the two patches, respectively. The average accuracy of the five models considered in our method increases from 11.45 to 43.73. The watermark capacity and robustness are also reduced. Those results support the effectiveness and the necessity of our proposed method.

![](images/image_06.jpg)

<details>
<summary>scatter</summary>

| Epoch | BER w/o_A (x10^-1) | BER w/o_B (x10^-1) | BER w/_A (x10^-1) | BER w/_B (x10^-1) | PSNR_A | PSNR_B |
| --- | --- | --- | --- | --- | --- | --- |
| 0 | ~0.05 | ~0.15 | ~0.05 | ~0.15 | ~0.5 | ~44 |
| 10 | ~0.05 | ~0.15 | ~0.05 | ~0.3 | ~0.85 | ~44 |
| 20 | ~0.05 | ~0.15 | ~0.05 | ~0.2 | ~0.85 | ~44 |
| 30 | ~0.05 | ~0.15 | ~0.05 | ~0.2 | ~0.95 | ~44 |
| 40 | ~0.05 | ~0.15 | ~0.05 | ~0.2 | ~0.95 | ~44 |
| 50 | ~0.05 | ~0.15 | ~0.05 | ~0.2 | ~1.05 | ~44 |
| 60 | ~0.05 | ~0.15 | ~0.05 | ~0.2 | ~1.2 | ~44 |
| 70 | ~0.05 | ~0.15 | ~0.05 | ~0.2 | ~1.35 | ~44 |
| 80 | ~0.05 | ~0.15 | ~0.05 | ~0.2 | ~1.35 | ~44 |
| 90 | ~0.05 | ~0.15 | ~0.05 | ~0.2 | ~1.35 | ~44 |
| 100 | ~0.05 | ~0.15 | ~0.05 | ~0.2 | ~1.35 | ~44 |
</details>

Fig. 6. Performance comparison (BER↓ and PSNR↑) of two dual watermark extraction methods. A and B mean dual and dualf method respectively. BER w/o and BER w/ are the watermark robustness of images without and with unlearnable perturbation. Line-style stands for BER value referring to the left y-axis and point-style stands for PSNR referring to the right y-axis.

![](images/image_07.jpg)

Clean

![](images/image_08.jpg)

∈ = 4

![](images/image_09.jpg)

∈ = 8

Fig. 7. Visualization of unlearnable images with different ϵ

## E. Performance of the Proposed Unlearnable Examples

Considering that our semantic information protection task requires a favorable availability attack with powerful generalizability, we propose a novel method to generate unlearnable perturbations by minimizing MI between the input image and the output of the victim model. In this section, let us discuss the performance of our proposed method alone.

In previous adversarial research, the commonly used noise constraint $\| \boldsymbol { \eta } _ { i } \| _ { \infty } < \epsilon = 8$ does not guarantee imperceptibility to humans, as evident in our visualization results in Fig. 7. So we set $\epsilon   =   4$ in our experiments to provide high image quality. To test the transferability of noise, we choose five distinct network architectures: ResNet18 [54], VGG19 [55], MobileNetV2 [56], GoogLeNet [57], and Dense121 [58]. We utilize cross-entropy loss as the criterion, employing SGD with a momentum of 0.9 and a weight decay of 0.0005 as the optimizer. Combined with the CosineAnnealingLR decay scheduler, the initial learning rate is 0.01 and gradually declines to 0. The batch size is 128 for CIFAR10 and 32 for ImageNet. Data augmentation methods include horizontal flipping and random cropping. The training epochs are set to 60 for CIFAR10 and 100 for ImageNet-100.

TABLE V DATA AUGMENTATION ON THE CIFAR10. SIP STANDS FOR SEMANTIC INFORMATION PROTECTION, AND OV FOR OWNERSHIP VERIFICATION. RESULTS MARKED WITH \* REFER TO THE ORIGINAL PAPER.

<table><tr><td>Task</td><td>Method</td><td>Gaussian noise</td><td>Cutmix</td><td>Cutout</td><td>Mixup</td></tr><tr><td rowspan="6">SIP</td><td>Emax [14]</td><td>10.88±0.35</td><td>12.57±0.58</td><td>11.90±0.33</td><td>17.26±0.71</td></tr><tr><td>Emin [20]</td><td>10.62±0.17</td><td>10.54±0.04</td><td>10.91±0.02</td><td>18.14±0.92</td></tr><tr><td>Tattack [14]</td><td>14.78±1.04</td><td>12.18±0.63</td><td>12.06±2.28</td><td>13.42±1.69</td></tr><tr><td>Linear* [17]</td><td>19.83</td><td>20.72</td><td>26.28</td><td>43.88</td></tr><tr><td>Conv* [17]</td><td>21.28</td><td>18.60</td><td>23.04</td><td>36.97</td></tr><tr><td>Ours</td><td>10.37±0.27</td><td>11.04±1.64</td><td>10.52±0.19</td><td>12.88±0.55</td></tr><tr><td>OV</td><td>Ours</td><td>0.06%</td><td>0.05%</td><td>0.03%</td><td>0.04%</td></tr></table>

To assess the effectiveness of our method, we compare it with multiple state-of-the-art availability attacks, including Emax [14], Emin [20], Tattack [14], Linear [17], Conv [17], Autoregressive [15], and Gradient alignment [18]. As presented in Tables VI, the experimental results include the mean and standard deviation of the test accuracy over 6 times repeats. The best scores are highlighted in bold. The results reveal that our method attains the best average score across five models, surpassing the second-best by 6 points on CIFAR10 (the last column). Especially, the accuracy of five different models is quite consistent, which reveals the favorable generalizability of our proposed attack method. Similar trends are mirrored in the results for ImageNet-10 and ImageNet-100. Collectively, these findings underscore the efficacy of our method in safeguarding data copyright.

In Fig. 8, the training process of different methods on CIFAR10 with ϵ = 4 perturbations are shown, with the shadow indicating the standard deviation and signifying training stability. Notably, weak models with lower accuracy in original data training, such as MobileNetV2, and GoogLeNet, exhibited robust to random noise. Perturbations generated by Tattack [14], Emax [14], and Autoregression [15] demonstrated poor transferability on weak models. Compared to Emin [20], our approach yields lower accuracy and showcases greater performance stability.

## F. Robustness Analysis

We address the issue of data compression in cloud-based data repositories and the susceptibility of protected data to attacks by data thieves. As a consequence, it becomes crucial to assess the robustness of protected data.

1) JPEG Compression: For watermark, we report the results of compression of watermarked images in Table II, and our BERs are all under 0.35%. For perturbation, our experiments unveiled the vulnerability of universal adversarial perturbations generated by state-of-the-art methods of availability attack when exposed to JPEG compression. Results are presented in Table VII. Models trained on original images subjected to JPEG75 compression maintained stable accuracy levels (the 2-nd row). However, the accuracy of models trained on unlearnable examples produced by Emin, Emax, Tattack, and our method exhibited an average improvement of 38% upon compression (3-6 rows of the last column). This counteracted the effect of original availability attacks.

![](images/image_10.jpg)

Fig. 8. Training accuracy (↓) of networks on protected CIFAR10 dataset using various availability attack methods. Curves show mean accuracy over 6 experimental repetitions, with shading indicating standard deviation.

TABLE VI GENERALIZABILITY COMPARISON OF UNLEARNABLE PERTURBATION. RESULTS MARKED WITH \* REFER TO THE ORIGINAL PAPER.

<table><tr><td>Dataset</td><td>Method</td><td>ResNet18</td><td>VGG19</td><td>MobilenetV2</td><td>GoogLeNet</td><td>Dense121</td><td>Avg.</td></tr><tr><td rowspan="8">CIFAR10</td><td>Clean</td><td>94.49±0.09</td><td>90.23±0.36</td><td>84.86±0.37</td><td>87.42±0.26</td><td>94.46±0.16</td><td>90.29</td></tr><tr><td>Emax [14]</td><td>9.20±0.63</td><td>20.29±3.70</td><td>19.80±0.65</td><td>28.97±1.66</td><td>9.82±0.65</td><td>17.62</td></tr><tr><td>Emin [20]</td><td>16.51±2.39</td><td>18.56±0.81</td><td>16.94±1.16</td><td>21.73±0.75</td><td>15.37±1.72</td><td>17.82</td></tr><tr><td>Tattack [14]</td><td>11.70±1.83</td><td>22.91±7.52</td><td>17.53±3.80</td><td>66.20±16.46</td><td>11.28±0.72</td><td>25.92</td></tr><tr><td>Tattack(sample-wise)* [14]</td><td>23.79±0.27</td><td>28.49±0.65</td><td>20.53±0.02</td><td>23.05±0.16</td><td>20.90±0.22</td><td>23.35</td></tr><tr><td>Autoregressive [15]</td><td>25.98±1.20</td><td>37.52±0.93</td><td>17.88±1.51</td><td>18.58±1.73</td><td>18.12±1.34</td><td>23.56</td></tr><tr><td>Gradient alignment* [18]</td><td>68.15±0.55</td><td>66.58±0.58</td><td>66.71±0.60</td><td>71.02±0.15</td><td>70.12±0.12</td><td>68.52</td></tr><tr><td>Ours</td><td>10.26±0.15</td><td>11.74±0.23</td><td>11.27±0.45</td><td>13.51±0.35</td><td>10.45±0.40</td><td>11.45</td></tr><tr><td rowspan="5">ImageNet-10</td><td>Clean</td><td>72.77±0.93</td><td>67.60±0.60</td><td>72.37±1.02</td><td>74.47±1.56</td><td>73.67±1.47</td><td>72.18</td></tr><tr><td>Emax [14]</td><td>26.37±0.53</td><td>20.93±0.36</td><td>24.20±0.82</td><td>27.30±0.82</td><td>35.90±1.72</td><td>26.94</td></tr><tr><td>Emin [20]</td><td>17.23±0.37</td><td>10.73±0.51</td><td>14.77±0.99</td><td>22.33±1.91</td><td>42.93±2.93</td><td>21.60</td></tr><tr><td>Tattack [14]</td><td>19.93±2.89</td><td>21.07±0.68</td><td>17.50±2.20</td><td>23.97±1.17</td><td>29.93±2.23</td><td>22.48</td></tr><tr><td>Ours</td><td>13.50±0.46</td><td>20.50±1.61</td><td>16.63±2.05</td><td>17.60±1.65</td><td>21.10±2.51</td><td>17.87</td></tr><tr><td rowspan="5">ImageNet-100</td><td>Clean</td><td>71.39±0.31</td><td>65.09±0.35</td><td>76.80±0.39</td><td>77.13±0.07</td><td>73.50±0.33</td><td>72.78</td></tr><tr><td>Emax [14]</td><td>11.37±0.62</td><td>4.19±2.26</td><td>8.65±0.13</td><td>12.15±0.23</td><td>12.30±0.60</td><td>9.73</td></tr><tr><td>Emin [20]</td><td>9.09±0.35</td><td>2.55±2.17</td><td>2.65±1.37</td><td>1.52±0.17</td><td>24.98±1.48</td><td>8.16</td></tr><tr><td>Tattack [14]</td><td>8.45±0.26</td><td>4.90±0.28</td><td>6.61±0.05</td><td>9.59±0.27</td><td>10.75±0.53</td><td>8.06</td></tr><tr><td>Ours</td><td>8.75±0.05</td><td>1.11±0.05</td><td>6.62±0.89</td><td>0.94±0.33</td><td>14.39±0.51</td><td>6.36</td></tr></table>

In equation (6), we referred to the open-source code from [59] for the implementation of both standard JPEG and differentiable JPEG algorithms. The quality factor (QF) for the differentiable JPEG in the perturbations optimization process is set to 75. We assessed their transferability using various QFs for the standard JPEG. The outcomes in Table VII reveal a substantial enhancement in our method’s robustness against JPEG compression when utilizing differentiable JPEG. The accuracy is reduced from 63% to 39% (the last column of 6-7 rows). Moreover, no significant variation is observed in the effect of different QFs or no compression (the 8-9 rows).
2) Data Augmentation: In real-world scenarios, data augmentation serves as a commonly employed technique to artificially expand data volumes. This approach proves beneficial for enhancing model performance by generating novel and distinct examples for training datasets. However, its utilization might counteract the impact of minor adversarial perturbations. To assess robustness, we examine various data augmentation methods for training a ResNet-18 model, encompassing Gaus-

sian noise, Cutmix, Cutout, and Mixup. In the Gaussian noise attack, each image receives an additional noise layer with a random strength of ϵ = 4. All aforementioned experiments are conducted under standard augmentation settings, including horizontal flip and random crop. The results presented in Table V (the 2-7 rows) demonstrate the minimal influence of common data augmentation techniques on our method. We also pollute watermarks with these augmentation methods, and BERs are closed to 0 (the last row).

3) The Robustness of Adversarial Perturbations: At present, there are some defenses against availability attacks, including adversarial training [19], adversarial augmentation [60], and clean-unlearnable data mixture training. We perform adversarial training on data protected by our method and the comparative methods Emin [20]. Table VIII illustrates that Emin is ineffective for adversarial training, particularly with adversarial perturbation $\rho ~ = ~ 2$ . In comparison, our proposed method can still effectively degrade the performance of models. As for the adversarial augmentation [60], it has a limited impact on our method by simply improving the accuracy of ResNet18 from 10.26% to 15.54% on CIFAR10. For 10% clean data combined with 90% unlearnable data, Emin gets 77.18% and ours is 66.84%. It shows that our method still has an advantage.

![](images/image_11.jpg)

Fig. 9. Visualization of images with and without perturbation generated by different methods.

TABLE VII EFFECT OF JPEG COMPRESSION ON CIFAR10 PROTECTION. QF IS THE JPEG QUALITY FACTOR, AND DISTORTION INDICATES THE AVERAGE CHANGE OF IMAGE PIXELS.

| Method | QF | distortion | ResNet18 | VGG19 | MobilenetV2 | GoogLeNet | Dense121 | Avg. |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Clean | 75 | 5.97 | 92.14±0.15 | 88.61±0.08 | 83.13±0.35 | 85.52±0.16 | 92.28±0.11 | 88.34 |
| Emax [14] | 75 | 6.44 | 46.35±0.25 | 53.07±1.72 | 46.82±1.38 | 75.07±8.43 | 43.26±0.64 | 52.91 |
| Emin [20] | 75 | 6.51 | 56.62±0.69 | 45.12±1.00 | 39.77±0.80 | 51.77±0.93 | 53.41±0.95 | 49.34 |
| Tattack [14] | 75 | 6.46 | 63.12±0.48 | 60.26±22.50 | 71.14±4.90 | 84.26±0.17 | 60.58±0.85 | 67.87 |
| Ours | 75 | 6.62 | 69.24±0.43 | 62.60±0.86 | 53.70±0.80 | 64.87±0.54 | 67.26±1.25 | 63.53 |
| Ours+JPEG | 75 | 6.38 | 35.25±1.06 | 38.80±13.08 | 39.09±0.96 | 50.57±1.13 | 31.88±1.34 | 39.12 |
| Ours+JPEG | 85 | 5.62 | 44.42±0.66 | 39.76±1.17 | 36.09±0.65 | 48.74±0.92 | 38.54±0.80 | 41.51 |
| Ours+JPEG | N/A | 0 | 10.05±0.03 | 12.24±0.77 | 11.06±0.51 | 15.60±1.54 | 10.05±0.02 | 11.80 |

TABLE VIII THE ROBUSTNESS OF ADVERSARIAL TRAINING.

| Method | $\rho$ | ResNet18 | VGG19 | MobileNetV2 | GoogLeNet | Dense121 | Avg |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Emin [20] | 2 | 92.12 | 10.00 | 80.46 | 84.88 | 18.30 | 57.15 |
| Ours | 2 | 16.21 | 10.00 | 16.56 | 14.50 | 18.18 | 15.09 |
| Emin [20] | 4 | 59.36 | 10.00 | 69.59 | 72.81 | 48.97 | 52.15 |
| Ours | 4 | 39.79 | 10.00 | 68.09 | 69.22 | 42.10 | 45.84 |

TABLE IX WATERMARK ROBUSTNESS UNDER EXTENDED DISTORTIONS (BER↓)

| Attack | BER (w/) | BER (w/o) |
| --- | --- | --- |
| No attack | 0 | 0 |
| Speckle (var=0.01) | 0.03% | 0.08% |
| Salt&amp;pepper (density=0.005) | 0.49% | 0.33% |
| Salt&amp;pepper (density=0.01) | 1.50% | 1.06% |
| Median filter [3, 3] | 4.27% | 3.26% |
| Gaussian (σ=0.001) | 0.08% | 0.16% |
| JPEG (QF=75) | 0.28% | 0.34% |
| Resize [256-128-256] | 2.81% | 3.04% |
| Resize [256-512-256] | 0.01% | 0.01% |

4) The Robustness of Watermark: To comprehensively evaluate the robustness of the embedded watermark under diverse distortions, we conducted extensive experiments following [61]. Table IX summarizes the BER values under extended distortions. The watermark exhibits strong resilience to the speckle (var=0.01), salt&pepper (density=0.005), Gaussian (σ=0.001), JPEG (QF=75), resize (256-512-256), with BER consistently below 0.35% even with perturbations.

## V. CONCLUSION

In this paper, we have proposed a data protection method that combines recoverable unlearnable examples and digital watermarking, which can realize the role of copyright verification and semantic information protection of datasets simultaneously. We propose a more effective method based on the theory of information entropy for availability attacks and verify it on multiple datasets. For digital watermarks, we propose a double watermark extraction method that can extract messages from two states of images. This method can solve the actual problems in data copyright protection, namely copyright verification against leaked data and authorization from protected datasets.

## ACKNOWLEDGMENTS

This work was supported in part by the National Natural Science Foundation of China (Grant No. 62202009) and by the Macau Science and Technology Development Fund (Grant No. 0040/2023/ITP1, 0004/2023/RIB1).

## REFERENCES

[1] J. Chen, X. Liao, Z. Qian, and Z. Qin, “Prest-net: Multi-domain probability estimation network for robust image forgery detection,” ACM Trans. Multimedia Comput. Commun. Appl., vol. 21, no. 3, Mar. 2025.
[2] F. Ernawan and D. Ariatmanto, “A recent survey on image watermarking using scaling factor techniques for copyright protection,” Multimedia Tools and Applications, pp. 1–41, 2023.
[3] X. Xiao, Y. Zhang, Z. Hua, Z. Xia, and J. Weng, “Client-side embedding of screen-shooting resilient image watermarking,” IEEE Transactions on Information Forensics and Security, vol. 19, pp. 5357–5372, 2024.
[4] L. Xiong, X. Han, C.-N. Yang, and Y.-Q. Shi, “Robust reversible watermarking in encrypted image with secure multi-party based on lightweight cryptography,” IEEE Transactions on Circuits and Systems for Video Technology, vol. 32, no. 1, pp. 75–91, 2022.
[5] J. You, Y.-G. Wang, G. Zhu, and S. Kwong, “Truncated robust natural watermarking with hungarian optimization,” IEEE Transactions on Circuits and Systems for Video Technology, vol. 32, no. 2, pp. 483–495, 2022.
[6] X. Wang, X. Li, and Q. Pei, “Independent embedding domain based twostage robust reversible watermarking,” IEEE Transactions on Circuits and Systems for Video Technology, vol. 30, no. 8, pp. 2406–2417, 2020.
[7] Y. Tang, S. Wang, C. Wang, S. Xiang, and Y.-M. Cheung, “A highly robust reversible watermarking scheme using embedding optimization and rounded error compensation,” IEEE Transactions on Circuits and Systems for Video Technology, vol. 33, no. 4, pp. 1593–1609, 2023.
[8] Z. Jia, H. Fang, and W. Zhang, “Mbrs: Enhancing robustness of dnnbased watermarking by mini-batch of real and simulated jpeg compression,” in Proceedings of the 29th ACM International Conference on Multimedia, 2021, pp. 41–49.
[9] J. Zhu, R. Kaplan, J. Johnson, and L. Fei-Fei, “Hidden: Hiding data with deep networks,” in Proceedings of the European Conference on Computer Vision, 2018, pp. 657–672.
[10] J. Chen, J. Wang, T. Peng, Y. Sun, P. Cheng, S. Ji, X. Ma, B. Li, and D. Song, “Copy, right? a testing framework for copyright protection of deep learning models,” in 2022 IEEE Symposium on Security and Privacy (SP), 2022, pp. 824–841.
[11] Y. Gao, Y. Sun, X. Ma, Z. Wu, and Y.-G. Jiang, “Modellock: Locking your model with a spell,” in Proceedings of the 32nd ACM International Conference on Multimedia, 2024, p. 11156–11165.
[12] K. Hill, “The secretive company that might end privacy as we know it,” in Ethics of Data and Analytics. Auerbach Publications, 2022, pp. 170–177.
[13] J. Zhang, J. Sang, X. Zhao, X. Huang, Y. Sun, and Y. Hu, “Adversarial privacy-preserving filter,” in Proceedings of the 28th ACM International Conference on Multimedia, 2020, pp. 1423–1431.
[14] L. Fowl, M. Goldblum, P.-y. Chiang, J. Geiping, W. Czaja, and T. Goldstein, “Adversarial examples make strong poisons,” in Proceedings of Advances in Neural Information Processing Systems, 2021, pp. 30 339–30 351.
[15] P. Sandoval-Segura, V. Singla, J. Geiping, M. Goldblum, T. Goldstein, and D. Jacobs, “Autoregressive perturbations for data poisoning,” in Proceedings of Advances in Neural Information Processing Systems, 2022, pp. 27 374–27 386.
[16] D. Yu, H. Zhang, W. Chen, J. Yin, and T.-Y. Liu, “Availability attacks create shortcuts,” in Proceedings of the 28th ACM SIGKDD Conference on Knowledge Discovery and Data Mining, 2022, pp. 2367–2376.
[17] W. Peng and J. Chen, “Learnability lock: Authorized learnability control through adversarial invertible transformations,” in Proceedings of International Conference on Learning Representations, 2022, pp. 1–12.
[18] L. Fowl, P.-y. Chiang, M. Goldblum, J. Geiping, A. Bansal, W. Czaja, and T. Goldstein, “Preventing unauthorized use of proprietary data: Poisoning for secure dataset release,” in Proceedings of International Conference on Learning Representations Workshop, 2021, pp. 1–10.
[19] S. Fu, F. He, Y. Liu, L. Shen, and D. Tao, “Robust unlearnable examples: Protecting data against adversarial learning,” in Proceedings of International Conference on Learning Representations, 2022, pp. 1–13.
[20] H. Huang, X. Ma, S. M. Erfani, J. Bailey, and Y. Wang, “Unlearnable examples: Making personal data unexploitable,” in Proceedings of International Conference on Learning Representations, 2021, pp. 1–12.
[21] J. Zhang, X. Ma, Q. Yi, J. Sang, Y.-G. Jiang, Y. Wang, and C. Xu, “Unlearnable clusters: Towards label-agnostic unlearnable examples,” in Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition, 2023, pp. 3984–3993.
[22] M. Zhao, B. Wang, W. Wang, Y. Kong, T. Zheng, and K. Ren, “Guided erasable adversarial attack (geaa) toward shared data protection,” IEEE Transactions on Information Forensics and Security, pp. 2468–2482, 2022.
[23] X. Jia, X. Wei, X. Cao, and X. Han, “Adv-watermark: A novel watermark perturbation for adversarial examples,” in Proceedings of the 28th ACM International Conference on Multimedia, 2020, pp. 1579–1587.
[24] C. Tian, M. Zheng, T. Jiao, W. Zuo, Y. Zhang, and C.-W. Lin, “A self-supervised cnn for image watermark removal,” IEEE Transactions on Circuits and Systems for Video Technology, vol. 34, no. 8, pp. 7566–7576, 2024.
[25] B. Biggio, B. Nelson, and P. Laskov, “Poisoning attacks against support vector machines,” in Proceedings of the International Conference on Machine Learning, 2012, pp. 1467–1474.
[26] L. Munoz-Gonz ˜ alez, B. Biggio, A. Demontis, A. Paudice, V. Wongras- ´ samee, E. C. Lupu, and F. Roli, “Towards poisoning of deep learning algorithms with back-gradient optimization,” in Proceedings of the 10th ACM workshop on artificial intelligence and security, 2017, pp. 27–38.
[27] W. R. Huang, J. Geiping, L. Fowl, G. Taylor, and T. Goldstein, “Metapoison: Practical general-purpose clean-label data poisoning,” in Proceedings of Advances in Neural Information Processing Systems, 2020, pp. 12 080–12 091.
[28] J. Geiping, L. Fowl, W. R. Huang, W. Czaja, G. Taylor, M. Moeller, and T. Goldstein, “Witches’ brew: Industrial scale data poisoning via gradient matching,” in Proceedings of the International Conference on Learning Representations, 2020, pp. 1–13.
[29] C. Yuan and S. Wu, “Neural tangent generalization attacks,” in Proceedings of International Conference on Machine Learning, 2021, pp. 12 230–12 240.
[30] J. Ren, H. Xu, Y. Wan, X. Ma, L. Sun, and J. Tang, “Transferable unlearnable examples,” in International Conference on Learning Representations, 2023, pp. 1–9.
[31] Y. Sun, H. Zhang, T. Zhang, X. Ma, and Y.-G. Jiang, “Unseg: One universal unlearnable example generator is enough against all image segmentation,” in NeurIPS, 2024.
[32] M. Ahmadi, A. Norouzi, N. Karimi, S. Samavi, and A. Emami, “Redmark: Framework for residual diffusion watermarking based on deep networks,” Expert Systems with Applications, vol. 146, p. 113157, 2020.
[33] Y. Liu, M. Guo, J. Zhang, Y. Zhu, and X. Xie, “A novel two-stage separable deep learning framework for practical blind watermarking,” in Proceedings of the 27th ACM International conference on multimedia, 2019, pp. 1509–1517.
[34] M. Tancik, B. Mildenhall, and R. Ng, “Stegastamp: Invisible hyperlinks in physical photographs,” in Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition, 2020, pp. 2117–2126.
[35] W. Sun, J. Zhou, Y. Li, M. Cheung, and J. She, “Robust highcapacity watermarking over online social network shared images,” IEEE Transactions on Circuits and Systems for Video Technology, vol. 31, no. 3, pp. 1208–1221, 2021.
[36] L. Fu, X. Liao, J. Guo, L. Dong, and Z. Qin, “Waverecovery: Screenshooting watermarking based on wavelet and recovery,” IEEE Transactions on Circuits and Systems for Video Technology, vol. 35, no. 4, pp. 3603–3618, 2025.
[37] Y. Li, X. Liao, and X. Wu, “Screen-shooting resistant watermarking with grayscale deviation simulation,” IEEE Transactions on Multimedia, vol. 26, pp. 10 908–10 923, 2024.
[38] R. Zhang, P. Isola, A. A. Efros, E. Shechtman, and O. Wang, “The unreasonable effectiveness of deep features as a perceptual metric,” in Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition, 2018, pp. 586–595.
[39] G. K. Wallace, “The jpeg still picture compression standard,” IEEE Transactions on Consumer Electronics, vol. 34, no. 4, pp. 30–44, 1991.
[40] J. Hu, L. Shen, S. Albanie, G. Sun, and E. Wu, “Squeeze-and-excitation networks,” in Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition, 2017, pp. 7132–7141.
[41] I. J. Goodfellow, J. Shlens, and C. Szegedy, “Explaining and harnessing adversarial examples,” in Proceedings of International Conference on Learning Representations, 2015.
[42] S.-M. Moosavi-Dezfooli, A. Fawzi, and P. Frossard, “Deepfool: a simple and accurate method to fool deep neural networks,” in Proceedings of
the IEEE/CVF Conference on Computer Vision and Pattern Recognition, 2016, pp. 2574–2582.
[43] A. Fawzi, S.-M. Moosavi-Dezfooli, and P. Frossard, “The robustness of deep networks: A geometrical perspective,” IEEE Signal Processing Magazine, vol. 34, no. 6, pp. 50–62, 2017.
[44] Y. Liu, X. Chen, C. Liu, and D. Song, “Delving into transferable adversarial examples and black-box attacks,” in Proceedings of 5th International Conference on Learning Representations, 2017.
[45] Y. Bansal, P. Nakkiran, and B. Barak, “Revisiting model stitching to compare neural representations,” in Proceedings of Advances in Neural Information Processing Systems, vol. 34, 2021, pp. 225–236.
[46] F. Ding, J.-S. Denain, and J. Steinhardt, “Grounding representation similarity with statistical testing,” in Proceedings of Advances in Neural Information Processing Systems, 2021, pp. 1–13.
[47] S. Kornblith, M. Norouzi, H. Lee, and G. Hinton, “Similarity of neural network representations revisited,” in International Conference on Machine Learning, 2019, pp. 3519–3529.
[48] A. Morcos, M. Raghu, and S. Bengio, “Insights on representational similarity in neural networks with canonical correlation,” in Proceedings of Advances in Neural Information Processing Systems, vol. 31, 2018, pp. 1–10.
[49] T. Nguyen, M. Raghu, and S. Kornblith, “Do wide and deep networks learn the same things? uncovering how neural network representations vary with width and depth,” in International Conference on Learning Representations, 2021, pp. 1–9.
[50] L. Zhao, T. Liu, X. Peng, and D. Metaxas, “Maximum-entropy adversarial data augmentation for improved generalization and robustness,” in Proceedings of Advances in Neural Information Processing Systems, 2020, pp. 14 435–14 447.
[51] J. Deng, W. Dong, R. Socher, L.-J. Li, K. Li, and L. Fei-Fei, “Imagenet: A large-scale hierarchical image database,” in Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition, 2009, pp. 248–255.
[52] R. C. Ç alik and M. F. Demirci, “Cifar-10 image classification with convolutional neural networks for embedded systems,” in 2018 IEEE/ACS 15th International Conference on Computer Systems and Applications, 2018, pp. 1–2.
[53] O. M. Parkhi, A. Vedaldi, A. Zisserman, and C. V. Jawahar, “Cats and dogs,” in IEEE Conference on Computer Vision and Pattern Recognition, 2012.
[54] K. He, X. Zhang, S. Ren, and J. Sun, “Deep residual learning for image recognition,” in Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition, 2016, pp. 770–778.
[55] K. Simonyan and A. Zisserman, “Very deep convolutional networks for large-scale image recognition,” in Proceedings of International Conference on Learning Representations, 2015, pp. 1–10.
[56] M. Sandler, A. Howard, M. Zhu, A. Zhmoginov, and L.-C. Chen, “Mobilenetv2: Inverted residuals and linear bottlenecks,” in Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition, 2018, pp. 4510–4520.
[57] C. Szegedy, W. Liu, Y. Jia, P. Sermanet, S. Reed, D. Anguelov, D. Erhan, V. Vanhoucke, and A. Rabinovich, “Going deeper with convolutions,” in Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition, 2015, pp. 1–9.
[58] G. Huang, Z. Liu, L. Van Der Maaten, and K. Q. Weinberger, “Densely connected convolutional networks,” in Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition, 2017, pp. 4700–4708.
[59] H. Wu, J. Zhou, J. Tian, J. Liu, and Y. Qiao, “Robust image forgery detection against transmission over online social networks,” IEEE Transactions on Information Forensics and Security, pp. 443–456, 2022.
[60] T. Qin, X. Gao, J. Zhao, K. Ye, and C.-Z. Xu, “Learning the unlearnable: Adversarial augmentations suppress unlearnable example attacks,” arXiv preprint arXiv:2303.15127, 2023.
[61] Y. Zhang, D. Ye, C. Xie, L. Tang, X. Liao, Z. Liu, C. Chen, and J. Deng, “Dual defense: Adversarial, traceable, and invisible robust watermarking against face swapping,” IEEE Transactions on Information Forensics and Security, vol. 19, pp. 4628–4641, 2024.

![](images/image_12.jpg)

<details>
<summary>natural_image</summary>

Portrait photo of a young man in a black shirt (no text or symbols visible)
</details>

**Binze Wang** received the B.S. degree in the Chang’an University, Shaanxi, China, 2021 and M.S. from the Chinese Academy of Surveying and Mapping, Beijing, China, 2024. He is currently pursuing the Ph.D. degree in artificial intelligence at the Macau University of Science and Technology, Macau, China. His interests primarily lie in the security aspects of deep learning, especially adversarial attacks, and copyright protection.

![](images/image_13.jpg)

<details>
<summary>natural_image</summary>

Portrait photo of a man wearing glasses and a dark shirt (no text or symbols visible)
</details>

**Jinyu Tian** (Member, IEEE) received the B.S. and M.S. degrees in mathematics from Chongqing University, Chongqing, China, in 2014 and 2017, respectively. He got his Ph.D. degree in computer science from University of Macau, Macau, China, in 2022. He is currently an Assistant Professor at the School of Computer Science and Engineering, Macau University of Science and Technology. His current research interests include adversarial machine learning, security of deep learning, multimedia forensics, and subspace learning. He has published

in several high-impact international journals and conferences, including IEEE TIP, TNNLS, TIFS, TSC, and AAAI, CVPR. He is also an active reviewer for numerous prestigious journals and conferences, such as IEEE TIP, TMM, ACM MM, CVPR, NeurIPS, ICML, etc.

![](images/image_14.jpg)

**Xingrun Wang** received the M.S. degree from the School of Computer Science and Technology, Shandong University in 2019. She recieved the Ph.D. degree with the School of Computer Science and Engineering, Macau University of Science and Technology in 2024. Since 2025, she has been working at Foshan University. Her research interests include information security and protection, computer vision, and artificial intelligence.

![](images/image_15.jpg)

<details>
<summary>natural_image</summary>

Portrait of a woman wearing glasses and a dark blazer with an orange lanyard (no visible text or symbols)
</details>

**Xiaochen Yuan** (Senior Member, IEEE) received the Ph.D. degree in Software Engineering from the University of Macau in 2013. From 2014 to 2015, she was a postdoctoral fellow at the Department of Computer and Information Science of the University of Macau. From 2016 to 2021, she was an Assistant Professor and an Associate Professor at the Faculty of Information Technology of the Macau University of Science and Technology. Since 2021, she joined the Faculty of Applied Sciences of the Macao Polytechnic University, where she is currently an

Associate Professor. Her research interests include Multimedia Forensics and Security, Digital Watermarking, AI Model Security, Quantum Watermarking, Remote Image Processing, and Deep Learning Techniques and Applications. She has published more than 70 SCIE-indexed scientific articles in refereed journals, such as IEEE TIFS, IEEE TII, IEEE TMI, IEEE TIM, IEEE TETC, IEEE JSTARS, etc. and she has been served as reviewers for top-tier journals in related areas, as well as the program committee member, session chair, and regional chair for ICSPS, ICSIP, ICSTE, etc. She is also a member of CCF, a member of CSIG, and a senior member of IEEE.

![](images/image_16.jpg)

<details>
<summary>natural_image</summary>

Portrait of a man wearing glasses and a light blue shirt against a pink textured background (no text or symbols visible)
</details>

**Jianqing Li** (Senior Member, IEEE) received the Ph.D. degree in electronic engineering from the Beijing University of Posts and Telecommunications, Beijing, China, in 1999. From 2000 to 2002, he was a Visiting Professor with Information and Communications University, Daejeon, South Korea. From 2002 to 2004, he was a Research Fellow with Nanyang Technological University, Singapore. He joined the Macau University of Science and Technology in August 2004. He is currently a Professor. His research interests include wireless networks, IoT,

two-dimensional materials for photonics and fiber sensors.