# Detecting and Corrupting Convolution-based Unlearnable Examples

**Minghui Li**<sup>1\*</sup>**, Xianlong Wang**<sup>2,3\*†</sup>**, Zhifei Yu**<sup>2</sup>**, Shengshan Hu**<sup>2,3</sup>, **Ziqi Zhou**<sup>4</sup>**, Longling Zhang**<sup>2,3</sup>**, Leo Yu Zhang**<sup>5</sup>

1 School of Software Engineering, Huazhong University of Science and Technology
2 School of Cyber Science and Engineering, Huazhong University of Science and Technology
3 Hubei Key Laboratory of Distributed System Security, Hubei Engineering Research Center on Big Data Security
4 School of Computer Science and Technology, Huazhong University of Science and Technology
5 School of Information and Communication Technology, Griffith University {minghuili,wxl99,yzf,hushengshan,zhouziqi,longlingzhang}@hust.edu.cn, leo.zhang@griffith.edu.au

## Abstract

Convolution-based unlearnable examples (UEs) employ class-wise multiplicative convolutional noise to training samples, severely compromising model performance. This firenew type of UEs have successfully countered all defense mechanisms against UEs. The failure of such defenses can be attributed to the absence of norm constraints on convolutional noise, leading to severe blurring of image features. To address this, we first design an Edge Pixel-based Detector (**EPD**) to identify convolution-based UEs. Upon detection of them, we propose the first defense scheme against convolution-based UEs, **CO**rrupting these samples via random matrix multiplication by employing bilinear **IN**terpolation (**COIN**) such that disrupting the distribution of class-wise multiplicative noise. To evaluate the generalization of our proposed COIN, we newly design two convolution-based UEs called VUDA and HUDA to expand the scope of convolution-based UEs. Extensive experiments demonstrate the effectiveness of detection scheme EPD and that our defense COIN outperforms 11 state-of-the-art (SOTA) defenses, achieving a significant improvement on the CIFAR and ImageNet datasets.

## Introduction

The triumph of deep neural networks (DNNs) hinges on co-pious high-quality training data, motivating many commercial enterprises to scrape images from unidentified sources. In this scenario, adversaries may introduce elaborate and imperceptible perturbations to image data to serve as unlearnable examples (UEs) that are subsequently disseminated online. This leads to a diminished generalization capacity of the victim model after being trained on such samples (Huang et al. 2021; Lin et al. 2024; Meng et al. 2024). Previous UEs apply additive perturbations under the $\mathcal { L } _ { p }$ norm constraint to ensure sample’s visual concealment, referred to as bounded UEs (Huang et al. 2021; Fu et al. 2022; Ren et al. 2023; Liu, Wang, and Gao 2024). Correspondingly, many defense schemes (Tao et al. 2021; Liu, Zhao, and Larson 2023; Segura et al. 2023; Wang et al. 2024a) have been proposed, largely compromising bounded UEs. The ease with which bounded UEs are successfully defended can be attributed to the fact that the introduced additive noise is limited, rendering the noise distribution easily disrupted.

| Defenses↓ Metrics→ | Effectiveness against CUDA | Effectiveness against VUDA (Ours) | Effectiveness against HUDA (Ours) | Model-independent |
| --- | --- | --- | --- | --- |
| Cutout (DeVries and Taylor 2017) | ○ | ○ | ○ | ● |
| Mixup (Zhang et al. 2018) | ○ | ○ | ○ | ● |
| Cutmix (Yun et al. 2019) | ○ | ○ | ○ | ● |
| DP-SGD (Hong et al. 2020) | ○ | ○ | ○ | ○ |
| AT (Tao et al. 2021) | ○ | ○ | ○ | ○ |
| AA (Qin et al. 2023) | ○ | ○ | ○ | ● |
| AVATAR (Dolatabadi, Erfani, and Leckie 2023) | ○ | ○ | ○ | ○ |
| OP (Segura et al. 2023) | ○ | ○ | ○ | ○ |
| ISS-G (Liu, Zhao, and Larson 2023) | ○ | ○ | ○ | ● |
| ISS-J (Liu, Zhao, and Larson 2023) | ○ | ○ | ○ | ● |
| ECLIPSE (Wang et al. 2024a) | ○ | ○ | ○ | ○ |
| COIN (Ours) | ● | ● | ● | ● |

Table 1: The effectiveness of existing defenses against three convolution-based UEs and the dependence on models are presented. “•” denotes fully satisfying the condition.

However, the latest proposed UE that employs convolution operations without norm constraints, known as convolution-based UE (Sadasivan, Soltanolkotabi, and Feizi 2023), expands the scope of noise as demonstrated in Fig. 1 (b), compromising the performance of current defenses, as demonstrated in Tab. 1.

To the best of our knowledge, none of the existing defense mechanisms can effectively defend against convolution-based UEs.

To address this, the first step is to detect convolutionbased UEs due to their visual concealment as shown in Fig. 1 (a). We observe that the edge pixel values of the convolutionbased samples are biased towards black. Motivated by this key observation, we propose a detection scheme based on statistics of edge pixel values, successfully identifying convolution-based samples from UEs.

Subsequently, to simplify the challenging problem of defending against convolution-based UEs, we align with (Javanmard and Soltanolkotabi 2022; Min, Chen, and Karbasi 2021) in regarding the image samples as column vectors generated by a Gaussian mixture model (GMM) (Reynolds 2009). In this manner, existing convolution-based UEs can be expressed as the product of a matrix and clean samples. This multiplicative matrix can be directly understood as multiplicative perturbations, in contrast to the additive perturbations from the bounded UEs. Considering that the reason behind the effectiveness of convolution-based UEs is that

<small><span class="docvortex-page-footnote" data-block-type="page_footnote" style="color:#6b7280"><sup>\*</sup>These authors contributed equally.</span></small>

<small><span class="docvortex-page-footnote" data-block-type="page_footnote" style="color:#6b7280"><sup>†</sup>Corresponding author.</span></small>

<small><span class="docvortex-page-footnote" data-block-type="page_footnote" style="color:#6b7280">Copyright © 2025, Association for the Advancement of Artificial Intelligence (www.aaai.org). All rights reserved.</span></small>

![](images/image_01.jpg)

Figure 1: (a) Visual images of clean samples and three convolution-based UEs, CUDA (Sadasivan, Soltanolkotabi, and Feizi 2023), HUDA, and VUDA. (b) The plots of perturbation values from bounded and convolution-based UEs. It can be seen that the bounded perturbation values are limited within a certain range, while convolution-based perturbations lack such constraints.

DNNs incorrectly establish the mapping between class-wise multiplicative noise<sup>1</sup>and the ground truth labels (Sadasivan, Soltanolkotabi, and Feizi 2023). In light of this, our key intuition is to disrupt the distribution of class-wise multiplicative noise by increasing the inconsistency within classes and enhancing the consistency between classes.

Hence, we formally define two quantitative metrics in GMM, the inconsistency within intra-class multiplicative matrix $( \Theta _ { i m i } )$ and the consistency within inter-class multiplicative matrix $( \Theta _ { i m c } )$ . Our defense goal is to design an operation that can simultaneously enhance both $\Theta _ { i m i }$ and $\Theta _ { i m c } .$ Specifically, we leverage a uniform distribution to generate random values and shifts to construct a random matrix $\mathcal { A } _ { r }$ , subsequently applied to multiply the convolutionbased UEs in the GMM scenario. After conducting empirical experiments, we verify that the random matrix we design effectively boosts both $\Theta _ { i m i }$ and $\Theta _ { i m c } ,$ demonstrating defensive capabilities against convolution-based UEs.

Based on these insights, we propose the first defense strategy for countering convolution-based UEs, termed as **COIN**, which employs a randomly multiplicative image transformation as its mechanism. To verify the defense generalization of COIN against convolution-based UEs, we further propose two new types of convolution-based UEs, referred to as HUDA and VUDA as shown in Fig. 1. Extensive experiments reveal that our proposed defense COIN significantly overwhelms all existing defense schemes, ranging from 19.17%-44.63% in accuracy on CIFAR-10 and CIFAR-100 datasets, while ACC exceeds 85% and AUC surpasses 0.85 of proposed detection approach EPD. Our main contributions are summarized as:

• **First Defense Against Convolution-based UEs.** To the best of our knowledge, we propose the first detection and defense approach for convolution-based UEs, which utilizes an edge pixel value detection and a random matrix multiplication transformation.

• **Novel Convolution-based UEs.** We newly propose two convolution-based UEs, termed as VUDA and HUDA, to supply the scope of convolution-based UEs for better convincing defense evaluation.
• **Experimental Evaluations.** Extensive experiments against convolution-based UEs including existing ones and those we newly propose on four benchmark datasets and six types of model architectures validate the effectiveness of our detection and defense strategy.

## Preliminaries

## Threat Model

The attacker crafts convolution-based UEs by employing class-wise convolutional kernel $\mathcal { K } _ { i }$ to each image $x _ { i }$ in the training set $\mathcal { D } _ { c } ,$ thus causing the model $F$ with parameter θ trained on this dataset to generalize poorly to a clean test distribution D (Sadasivan, Soltanolkotabi, and Feizi 2023). Therefore, the attacker formally expects to work out the following bi-level objective:

$$
\max _ {(x, y) \sim \mathcal {D}} \mathbb {E} \left[ \mathcal {L} \left(F \left(x; \theta_ {p}\right), y\right) \right] \tag {1}
$$

$$
s. t. \theta_ {p} = \arg \min _ {\theta} \sum_ {\left(x _ {i}, y _ {i}\right) \in \mathcal {D} _ {c}} \mathcal {L} \left(F \left(x _ {i} \otimes \mathcal {K} _ {i}; \theta\right), y _ {i}\right) \tag {2}
$$

where $( x _ { i } , y _ { i } )$ represents the clean data from $\mathcal { D } _ { c } , \mathcal { L }$ is a loss function, $e . g .$ , cross-entropy loss, and $\otimes$ represents the convolution operation, while ensuring the modifications to $x _ { i }$ are not excessive for preserving the concealment of the sample. As for defenders, in the absence of any knowledge of clean samples $x _ { i } ,$ , they aim to perform certain operations on UEs to achieve the opposite goal of Eq. (1). Bounded UEs (Huang et al. 2021; Yu et al. 2022; Wu et al. 2023; Liu et al. 2024; Wang et al. 2024a) add norm-constrained additive noise to the sample $x _ { i }$ in Eq. (2).

## Broadening the Attack Range

Given the scarcity of research on existing convolution-based UEs, for more convincingly and comprehensively evaluating existing defense approaches against convolution-based UE attacks, we newly propose two types of convolution-based UEs, namely HUDA and VUDA, to expand the scope of convolution-based UEs. In particular, HUDA utilizes classwise horizontal filters to apply convolutional operations, subsequently employed on images based on categories. The definition of HUDA’s convolutional kernel is as follows:

<small><span class="docvortex-page-footnote" data-block-type="page_footnote" style="color:#6b7280"><sup>1</sup>“Class-wise multiplicative noise” means that images from the same class use the same multiplicative convolution, while those from different classes use different ones.</span></small>

![](images/image_02.jpg)

Figure 2: Hypothesis validation. Test accuracy (%) with $\Theta _ { i m c }$ (top row) and $\Theta _ { i m i }$ (bottom row) via changing parameter $a _ { y } .$

$$
\mathcal {K} _ {h} (i, j, k, l) = \left\{ \begin{array}{l l} b _ {y} & \text {if} j = \frac {\mathcal {T}}{2} \quad \text {and} \quad i \in \{0, 1, 2 \} \\ 0 & \text {else} \end{array} \right. \tag {3}
$$

where i represents the channel, j and k denote the rows and columns of the convolutional kernel, and l represents the depth of the convolutional kernel, T denotes the kernel size, and $b _ { y }$ denotes the class-wise blur parameter. Similarly, the definition of the vertical convolutional kernel of VUDA is:

$$
\mathcal {K} _ {v} (i, j, k, l) = \left\{ \begin{array}{l l} b _ {y} & \text {if} k = \frac {\mathcal {T}}{2} \quad \text {and} \quad i \in \{0, 1, 2 \} \\ 0 & \text {else} \end{array} \right. \tag {4}
$$

After obtaining the class-wise convolutional kernels, the entire dataset is processed using convolution operations to add the convolution-based multiplicative perturbations.

## Low-dimensional Representation

**Decomposing in low-dimensional space.** Similar to (Javanmard and Soltanolkotabi 2022; Min, Chen, and Karbasi 2021; Wang et al. 2024b), we define a binary classification problem involving a Bayesian classifier (Friedman, Geiger, and Goldszmidt 1997), and the clean dataset $\mathcal { D } _ { c }$ is sampled from a Gaussian mixture model $\mathcal { N } ( y \pmb { \mu } , I )$ . Here, y represents the labels {±1}, with mean $\pmb { \mu } ^ { \cdot } \in \mathbb { R } ^ { d }$ , and covariance $I   \in   \mathbb { R } ^ { d \times d }$ as the identity matrix (d represents the feature dimension). We denote the clean sample as $x \in \mathbb { R } ^ { d }$ , the convolution-based UE as $x _ { u } ,$ which can be formulated as left-multiplying the class-wise matrices $\mathcal { A } _ { y }$ by x:

$$
x _ {u} = \mathcal {A} _ {y} \cdot x \tag {5}
$$

![](images/image_03.jpg)

Figure 3: Key intuition behind our proposed defense.

Specifically, CUDA employs a tridiagonal matrix $\mathcal { A } _ { t } ,$ characterized by diagonal elements equal to 1, with the lower diagonal elements and upper diagonal elements both set to a pre-defined parameter $a _ { y } .$

**Key intuition.** The reason why convolution-based UEs work is that the model establishes a mapping between class-wise multiplicative noise and labels (Sadasivan, Soltanolkotabi, and Feizi 2023). Therefore, our intuition is that converting class-wise multiplicative noise into random multiplicative noise can disrupt this mapping, as demonstrated in Fig. 3. Specifically, by increasing the inconsistency of intra-class multiplicative matrices or the consistency of inter-class multiplicative matrices, we can make the class-wise multiplicative noise information more disordered. This results in classifiers being unable to learn meaningless information from the multiplicative noise. Based on this, we formally define the objective in the GMM space as follows:

**Definition 1.** We define the **intra-class matrix inconsistency**, denoted as $\Theta _ { i m i } , a s f o l l o w s \colon G i v e n$ the multiplicative matrices $\left\{ \begin{matrix} { \hat { \mathcal { A } _ { i } } } & { | } & { i } & { \stackrel { \circ } { = } } & { 1 , 2 , \cdots , n } \\ \end{matrix} \right\}$ within a certain class $y _ { k }$ (containing n samples), we have an intra-class average matrix defined as $\begin{array} { r l r } { { \mathcal { A } _ { \mu } } _ { k } } & { { } = } & { \frac { 1 } { n } { \sum _ { i = 1 } ^ { n } } { \mathcal { A } _ { i } } , } \end{array}$ an intra-class matrix variance defined as $\begin{array} { r c l } { \hat { \mathbb { D } } _ { k } } & { = } & { \frac { 1 } { n } { \sum _ { i = 1 } ^ { n } } ( \mathcal { A } _ { i } \: - \: \mathcal { A } _ { \mu } { } _ { k } ) ^ { 2 } \quad \in \quad \mathbb { R } ^ { d \times d } ; } \\ \end{array}$ , an intra-class matrix variance mean value defined as $\begin{array} { r c l } { \mathcal { V } _ { m } { } _ { k } } & { = } & { \frac { 1 } { d ^ { 2 } } \sum _ { i = 0 } ^ { d - 1 } \sum _ { j = 0 } ^ { d - 1 } \mathbb { D } _ { k } \left[ i \right] [ j ] , } \\ \end{array}$ , and then we have $\begin{array} { r } { \Theta _ { i m i } ~ = ~ \frac { 1 } { c } \sum _ { k = 0 } ^ { c - 1 } \mathcal { V } _ { m k } } \end{array}$ , where c denotes the number of classes in $\mathcal { D } _ { c }$

![](images/image_04.jpg)

Figure 4: Effectiveness of our proposed defense. Comparison results of test accuracy before and after left-multiplying a random matrix $\mathcal { A } _ { r }$ on all CUDA samples, α is set to 0.5.

**Definition 2.** We define the **inter-class matrix consistency** as $\Theta _ { i m c } ,$ as follows: Given the flattened intra-class average matrices of the j-th and k-th classes ${ f l a t } ( { \mathcal { A } _ { \mu } } _ { j } ) , \: { f l a t } ( { \mathcal { A } _ { \mu } } _ { k } ^ { \tilde { } } ) ,$ we then have $\Theta _ { \mathit { i m c } } { = } { s i m } ( { f l a t } ( \mathcal { A _ { \mu } } _ { j } ) , \check { { f l a t } ( \mathcal { A _ { \mu } } _ { k } ) } )$ , where $f l a t \left( \right)$ denotes flattening the matrix into a row vector and $\overset { \circ } { s i m } ( \overset { \circ } { u } , v ) = u \overset { \circ } { v ^ { T } } / ( \left\| u \right\| \overset { \circ } { \left\| v \right\| } )$ denotes cosine similarity.

**Hypothesis.** Increasing $\Theta _ { i m i } \: o r \: \Theta _ { i m c }$ can both improve the test accuracy of classifiers trained on the convolution-based UEs, and vice versa.

**Empirical validation.** With the two metrics formalized and proposed hypothesis, we can conduct experiments to validate our key intuition. It can be observed from the top row of $Fig. 2 $( \dot { \Theta } _ { i m i }$$ remains constant) that an increasing $\Theta _ { i m c }$ corresponds to an improvement in test accuracy, whereas a decrease in $\Theta _ { i m c }$ leads to a decline in test accuracy. In the bottom row of $Fig. 2 $( \Theta _ { i m c }$$ is held constant), test accuracy increases as $\Theta _ { i m i }$ rises and decreases as $\Theta _ { i m i }$ falls. Hence, these experimental results strongly support our proposed hypothesis and key intuition.

## Design of Low-dimensional Defense

Based on the results above, we are motivated to design a scheme that can increase $\Theta _ { i m i }$ and $\Theta _ { i m c } ,$ thus perturbing the distributions of multiplicative matrix $\mathcal { A } _ { y } .$ Assuming $\mathcal { A } _ { y }$ is the most common diagonal matrix, we left-multiply $\mathcal { A } _ { y }$ by a random matrix $\mathcal { A } _ { r } \in \mathbb { R } ^ { d \times d }$ to disrupt it, where the diagonal of $\mathcal { A } _ { r }$ is set to random values to introduce randomness. However, the form of diagonal matrix remains unchanged by multiplying this diagonal matrix, limiting the randomness. To solve this, we attempt to add another set of random variables above the diagonal, making it random. However, $\mathcal { A } _ { t }$ employed in CUDA still maintains the tridiagonal matrix pattern, still contraining the randomness of the matrix. To introduce more randomness, we come up with the idea of introducing small random offsets to the random variables for each row, thus successfully breaking the tridiagonal form, without introducing additional variables.

Thus, we unify the random values and shifts via a uniform distribution $\mathcal { U } ( \dot { - \alpha } , \alpha )$ , thereby striving to enhance $\Theta _ { i m c }$ as much as possible while already improving $\Theta _ { i m i }$ . We first sample a variable $s   \sim   \mathcal { U } ( - \alpha , \alpha ) , s   \in   \mathbb { R } ^ { d } ,$ , and then obtain $m _ { i } = \left\lfloor s _ { i } \right\rfloor , n _ { i } = s _ { i } - m _ { i } , 0 \leq i \leq d - 1 .   \mathcal { A } _ { r }$ is designed as:

$$
\mathcal {A} _ {r} = \left[ \begin{array}{c c c c c c} \frac {1 - n _ {0}}{0} & \frac {n _ {0}}{1 - n _ {1}} & 0 & 0 & \dots & 0 \\ 0 & 0 & \frac {n _ {1}}{1 - n _ {2}} & \frac {n _ {2}}{n _ {2}} & \dots & 0 \\ \vdots & \vdots & \vdots & \ddots & \vdots & \vdots \\ \vdots & \vdots & \vdots & \vdots & \ddots & \vdots \\ 0 & \dots & \dots & \dots & 0 & \underline {{1 - n _ {d - 1}}} \end{array} \right] \in \mathbb {R} ^ {d \times d} \tag {6}
$$

where the i-th and (i+1)-th elements of the i-th row (0≤ $i \leq d - 1)$ are $1 - n _ { i }$ and $n _ { i } ,$ and $`` 1-n_{i},n_{i}  ''$ means that the positions of these two elements for each row are shifted by $m _ { i }$ units simultaneously. When the new location $i + m _ { i }$ or $i + 1 + m _ { i }$ exceeds the matrix boundaries, we take its modulus with respect to d, thus obtaining their new position. The production process of random matrix $\mathcal { A } _ { r } ( \alpha )$ can be seen in the Alogorithm 1 in the supplementary material.

Whereupon, we obtain test accuracy results by leftmultiplying all CUDA samples with the matrix $\mathcal { A } _ { r } ,$ and compare with training on the CUDA UEs. As shown in Fig. 4, it can be observed that the test accuracy (with employing $\mathcal { A } _ { r } )$ is ahead of the accuracy (without using $\mathcal { A } _ { r } )$ regardless of values of $a _ { 1 }$ , demonstrating the defense effectiveness of our proposed random matrix $\mathcal { A } _ { r }$

## Methodology

## Our Design for Defense Scheme: COIN

Given the effectiveness of our defense scheme $\mathcal { A } _ { r }$ in the low-dimensional space, our key intuition is that extending it to the real-image domain will successfully defend against convolution-based UEs. To achieve this, we note that the two random values in each row in Eq. (6) can be modeled as two weighting coefficients, while the random location offset $m _ { i }$ in each row can be exploited to locate the pixel positions. Therefore, we regard the previous process of multiplying $\mathcal { A } _ { r }$ as a random linear interpolation process, i.e., a random multiplicative transformation. In view of this, the convolutionbased UE $\boldsymbol { x } _ { u } \in \mathbb { R } ^ { C \times H \times W }$ requires variables along with both horizontal and vertical directions and leveraging bilinear interpolation, which are defined as follows:

$$
s _ {x}, s _ {y} \sim \mathcal {U} (- \alpha , \alpha , \text {size} = H \cdot W) \tag {7}
$$

where U denotes a uniform distribution with size of H · W (height × width), α controls the range of the generated random variables. Considering that $s _ { x }$ and $s _ { y }$ are both arrays with size of $H \cdot W$ , we obtain arrays $m _ { x }$ and $m _ { y }$ by rounding down each variable from the arrays to its integer part (i.e., random location offsets), formulated as:

![](images/image_05.jpg)

<details>
<summary>flowchart</summary>

This diagram illustrates a pixel value tensor range calculation process, showing how an unlearnable image is processed by an output image to derive the final pixel value at point φ.
</details>

Figure 5: Our defense scheme COIN

$$
m _ {x i} = \left\lfloor s _ {x i} \right\rfloor , \quad m _ {y _ {i}} = \left\lfloor s _ {y _ {i}} \right\rfloor \tag {8}
$$

where ⌊ ⌋ represents the floor function, and i is the index in the array, ranging from 0 to $H \cdot W - 1$ . Subsequently, the arrays with coefficients $\omega _ { x } , \omega _ { y }$ required for the bilinear interpolation process are computed as follows:

$$
\omega_ {x i} = s _ {x i} - m _ {x i}, \quad \omega_ {y _ {i}} = s _ {x i} - m _ {y _ {i}} \tag {9}
$$

To obtain the coordinates of the pixels used for interpolation, we first initialize a coordinate grid, which is defined as:

$$
c _ {x}, c _ {y} = \mathcal {M} (a r a (W), a r a (H)) \in \mathbb {R} ^ {H \times W} \tag {10}
$$

where $\mathcal { M }$ denotes coordinate grid creation function, ara is employed to produce an array with values evenly distributed within a specified range. Whereupon we obtain the coordinates of the four nearest pixel points around the desired interpolation point in the bilinear interpolation process:

$$
q _ {1 1 i} = ((c _ {x i} + m _ {x i}) \% W, (c _ {y i} + m _ {y i}) \% H) \tag {11}
$$

$$
q _ {2 1 i} = \left(\left(c _ {x i} + m _ {x i} + 1\right) \% W, \left(c _ {y _ {i}} + m _ {y _ {i}}\right) \% H\right) \tag {12}
$$

$$
q _ {1 2 i} = \left(\left(c _ {x i} + m _ {x i}\right) \% W, \left(c _ {y _ {i}} + m _ {y _ {i}} + 1\right) \% H\right) \tag {13}
$$

$$
q _ {2 2 i} = \left(\left(c _ {x i} + m _ {x i} + 1\right) \% W, \left(c _ {y _ {i}} + m _ {y _ {i}} + 1\right) \% H\right) \tag {14}
$$

where $\%$ represents the modulo function, ensuring that the horizontal coordinate ranges from 0 to $W - 1$ and the vertical coordinate ranges from 0 to $H - 1 .$ Hence, we obtain new pixel values by using the pixel values of the four points mentioned above through bilinear interpolation:

$$
\mathcal {F} _ {j} \left(p _ {i}\right) = \left[ \begin{array}{l l} 1 - \omega_ {x i} & \omega_ {x i} \end{array} \right] \left[ \begin{array}{c c} \mathcal {F} _ {j} \left(q _ {1 1 i}\right) & \mathcal {F} _ {j} \left(q _ {1 2 i}\right) \\ \mathcal {F} _ {j} \left(q _ {2 1 i}\right) & \mathcal {F} _ {j} \left(q _ {2 2 i}\right) \end{array} \right] \left[ \begin{array}{c} 1 - \omega_ {y _ {i}} \\ \omega_ {y _ {i}} \end{array} \right] \tag {15}
$$

where $\mathcal { F } _ { j } ( { \bf \Lambda } )$ denotes the pixel value of the j-th channel at a certain coordinate point, $p _ { i }$ denotes the coordinate of the newly generated pixel point. Each generated pixel value should be clipped within the range (0,1). Finally, we gain the transformed image $x _ { t }$ by applying Eq. (15) and clipping to pixel values of each channel of the image $x _ { u }$ . The schematic diagram of the above process is shown in Fig. 5.

## Our Design for Detection Scheme: EPD

**Key Intuition.** It is crucial to detect whether the perturbations are convolution-based or not, because without prior knowledge, defenders have difficulty discerning the type of UEs. The key intuition behind our detection method is that convolution-based UEs typically exhibit lower pixel values around the edges, often appearing as black pixels as shown in Fig. 1 (a). Leveraging this intuition, we propose edge pixel-based detection (EPD) scheme as follows:

**Feature Extraction.** We sum the RGB values along the four edges (i.e., top, bottom, left, and right) of the detection-ready sample $\boldsymbol { x } _ { d } \in \mathbb { \hat { R } } ^ { C \times H \times \hat { W } }$ to serve as the feature for detection. The channel-wise feature vector $v _ { d } ( c )$ is defined as:

$$
v _ {d} (c) = \left[ \sum_ {w = 1} ^ {W} e (c, 1, w), \sum_ {w = 1} ^ {W} e (c, H, w), \sum_ {h = 1} ^ {H} e (c, h, 1), \sum_ {h = 1} ^ {H} e (c, h, W) \right] \tag {16}
$$

where $e ( c , h , w )$ represents the pixel value at channel $c _ { z }$ row $h ,$ and column w. The final feature vector $\mathcal { V } _ { d }$ is calculated by concatenating $v _ { d } ( c )$ across three channels, formulated as:

$$
\mathcal {V} _ {d} = \left[ v _ {d} (R), v _ {d} (G), v _ {d} (B) \right] \tag {17}
$$

**Binary Classification for Detection.** Given the simplicity of the vector $\mathcal { V } _ { d } ,$ , we utilize a support vector machine (SVM) (Huang et al. 2018) for binary classification. The pre-trained SVM classifies the image $x _ { d }$ by comparing the decision score $\mathcal { S }$ with a pre-defined threshold θ (typically set to 0 in SVM), where S is defined as:

$$
\mathcal {S} = \mathbf {w} \cdot \mathcal {V} _ {d} + b \tag {18}
$$

where w is the weight vector of the SVM, and b is the bias term. The predicted label $\hat { y } \in \{ 0 , 1 \}$ is given by:

$$
\hat {y} = \left\{ \begin{array}{l l} 1 & \text {if} S \left(x _ {d}\right) \geq \theta \\ 0 & \text {if} S \left(x _ {d}\right) <   \theta \end{array} \right. \tag {19}
$$

where 1 denotes a convolution-based UE, while 0 does not. It is important to note that since our proposed defense COIN is designed specifically for convolutional UEs, we only apply COIN after our EPD identifies the sample as a convolutional UE. Otherwise, whether the sample is a bounded UE, a clean example, or any other type, we do not consider differentiation or processing.

## Experiments

## Experimental Settings

Networks including ResNet (RN) (He et al. 2016), DenseNet (DN) (Huang et al. 2017), MobileNetV2 (MNV2) (Sandler et al. 2018), and VGG (Simonyan and Zisserman 2014) are selected. Benchmark datasets across from small, medium, and large sizes like CIFAR-10 (32×32), CIFAR-100 (32×32) (Krizhevsky and Hinton 2009), ImageNet20 (64×64), and ImageNet100 (224×224) (Deng et al. 2009) are used. The uniform distribution range α of COIN is empirically set to 2.0. During pre-training the SVM classifier of EPD, the linear kernel function is utilized, and penalty parameter $C _ { p }$ is set to 0.1. Meanwhile, the threshold θ of EPD during detection is set to 0. During training of classification task, we use SGD for training 80 epochs with a momentum of 0.9, a learning rate of 0.1, and batch sizes of 128, 32 for CIFAR and ImageNet, respectively<sup>2</sup>.

<table><tr><td colspan="2">Datasets</td><td colspan="5">CIFAR-10</td><td colspan="5">CIFAR-100</td></tr><tr><td>Defenses↓</td><td>Models→</td><td>ResNet18</td><td>VGG16</td><td>DenseNet121</td><td>MobileNetV2</td><td>AVG</td><td>ResNet18</td><td>VGG16</td><td>DenseNet121</td><td>MobileNetV2</td><td>AVG</td></tr><tr><td colspan="2">w/o</td><td>26.49</td><td>24.65</td><td>27.21</td><td>21.34</td><td>24.92</td><td>14.31</td><td>12.53</td><td>13.90</td><td>12.94</td><td>13.42</td></tr><tr><td colspan="2">MU (Zhang et al. 2018)</td><td>26.72</td><td>28.07</td><td>24.67</td><td>24.63</td><td>26.02</td><td>17.09</td><td>13.35</td><td>19.97</td><td>13.55</td><td>15.99</td></tr><tr><td colspan="2">CM (Yun et al. 2019)</td><td>26.02</td><td>28.53</td><td>24.64</td><td>20.73</td><td>24.98</td><td>12.51</td><td>10.14</td><td>20.77</td><td>10.14</td><td>13.39</td></tr><tr><td colspan="2">CO (DeVries and Taylor 2017)</td><td>20.07</td><td>27.58</td><td>24.86</td><td>20.46</td><td>23.24</td><td>12.80</td><td>10.56</td><td>16.19</td><td>13.56</td><td>13.28</td></tr><tr><td colspan="2">DP-SGD (Hong et al. 2020)</td><td>25.50</td><td>23.02</td><td>25.25</td><td>25.78</td><td>24.89</td><td>12.42</td><td>10.56</td><td>16.36</td><td>12.72</td><td>13.02</td></tr><tr><td colspan="2">AVATAR (Dolatabadi, Erfani, and Leckie 2023)</td><td>30.67</td><td>29.57</td><td>33.15</td><td>28.53</td><td>30.48</td><td>14.49</td><td>10.81</td><td>12.97</td><td>13.85</td><td>13.03</td></tr><tr><td colspan="2">AA (Qin et al. 2023)</td><td>39.85</td><td>38.68</td><td>38.92</td><td>41.06</td><td>39.63</td><td>24.83</td><td>1.00</td><td>27.89</td><td>20.49</td><td>18.55</td></tr><tr><td colspan="2">OP (Segura et al. 2023)</td><td>29.77</td><td>30.33</td><td>33.82</td><td>28.86</td><td>30.70</td><td>20.17</td><td>14.59</td><td>15.55</td><td>23.02</td><td>18.33</td></tr><tr><td colspan="2">ISS-G (Liu, Zhao, and Larson 2023)</td><td>25.77</td><td>21.42</td><td>26.73</td><td>19.85</td><td>23.44</td><td>8.80</td><td>6.40</td><td>11.48</td><td>8.71</td><td>8.85</td></tr><tr><td colspan="2">ISS-J (Liu, Zhao, and Larson 2023)</td><td>45.10</td><td>40.26</td><td>39.79</td><td>41.46</td><td>41.65</td><td>33.62</td><td>26.92</td><td>28.94</td><td>31.23</td><td>30.18</td></tr><tr><td colspan="2">AT (Tao et al. 2021)</td><td>50.59</td><td>45.95</td><td>49.01</td><td>42.59</td><td>47.04</td><td>37.27</td><td>28.18</td><td>34.21</td><td>35.74</td><td>33.85</td></tr><tr><td colspan="2">ECLIPSE (Wang et al. 2024a)</td><td>32.87</td><td>30.82</td><td>32.26</td><td>32.69</td><td>32.16</td><td>18.54</td><td>15.59</td><td>19.24</td><td>20.23</td><td>18.40</td></tr><tr><td colspan="2">COIN (Ours)</td><td>71.90</td><td>73.65</td><td>70.45</td><td>73.63</td><td>72.41</td><td>48.63</td><td>46.74</td><td>45.72</td><td>48.53</td><td>47.41</td></tr></table>

Table 2: Defenses against CUDA. The test accuracy (%) results on CIFAR-10 and CIFAR-100 datasets.

<table><tr><td>Datasets</td><td colspan="5">ImageNet100 224×224</td><td colspan="5">ImageNet20 64×64</td></tr><tr><td>Defenses,Models→</td><td>RN18</td><td>RN50</td><td>DN121</td><td>MNV2</td><td>AVG</td><td>RN18</td><td>RN50</td><td>DN121</td><td>MNV2</td><td>AVG</td></tr><tr><td>w/o</td><td>25.74</td><td>26.66</td><td>21.70</td><td>16.30</td><td>22.60</td><td>39.7</td><td>49.8</td><td>28.8</td><td>31.5</td><td>37.5</td></tr><tr><td>CO</td><td>25.46</td><td>29.20</td><td>23.90</td><td>17.58</td><td>24.04</td><td>42.1</td><td>56.2</td><td>38.2</td><td>36.9</td><td>43.4</td></tr><tr><td>MU</td><td>34.96</td><td>19.38</td><td>27.78</td><td>15.60</td><td>24.43</td><td>23.1</td><td>50.5</td><td>43.0</td><td>30.5</td><td>36.8</td></tr><tr><td>CM</td><td>16.54</td><td>24.04</td><td>23.58</td><td>8.00</td><td>18.04</td><td>32.3</td><td>50.5</td><td>29.8</td><td>25.9</td><td>34.6</td></tr><tr><td>ISS-G</td><td>14.92</td><td>13.50</td><td>9.78</td><td>5.78</td><td>11.00</td><td>12.5</td><td>19.1</td><td>11.3</td><td>11.1</td><td>13.5</td></tr><tr><td>AT</td><td>37.82</td><td>36.80</td><td>30.34</td><td>41.42</td><td>36.60</td><td>53.7</td><td>53.3</td><td>51.9</td><td>51.0</td><td>52.5</td></tr><tr><td>ISS-J</td><td>30.10</td><td>37.04</td><td>25.52</td><td>28.04</td><td>30.18</td><td>43.9</td><td>49.1</td><td>44.2</td><td>43.0</td><td>45.1</td></tr><tr><td>COIN (Ours)</td><td>37.80</td><td>35.38</td><td>35.22</td><td>41.50</td><td>37.48</td><td>62.7</td><td>54.7</td><td>65.1</td><td>61.0</td><td>60.9</td></tr></table>

Table 3: Defenses against CUDA with large datasets. The test accuracy (%) results on ImageNet100 (224×224) and ImageNet20 (64×64).

## Comparison with SOTA Defenses

We compare COIN with 10 SOTA defense schemes, i.e., AT (Tao et al. 2021), ISS (Liu, Zhao, and Larson 2023) including JPEG compression (ISS-J), and grayscale transformation (ISS-G), AVATAR (Dolatabadi, Erfani, and Leckie 2023), OP (Segura et al. 2023), AA (Qin et al. 2023), and ECLIPSE (Wang et al. 2024a). We also apply four defenses, differential privacy SGD (DP-SGD) (Hong et al. 2020; Zhang et al. 2021), cutmix (CM) (Yun et al. 2019), cutout (CO) (DeVries and Taylor 2017), and mixup (MU) (Zhang et al. 2018) that are popularly used to test against UEs.

We do not compare EPD with the UE detection method proposed by (Zhu, Yu, and Gao 2024) because our settings differ. Zhu et al.’s detection scheme is used to distinguish between clean samples and unlearnable samples (unable to distinguish convolution-based UEs), whereas EPD is designed to differentiate convolutional samples from other samples.

## Evaluation Metrics

For evaluation of the COIN’s effectiveness, consistent with previous works (Wang et al. 2024a,c), we employ test accuracy, i.e., the model accuracy on a clean test set after training on the datasets. The higher the test accuracy, the better the defense effectiveness. For evaluation of the detection performance, similar to Liu et al. (2023), we employ binary classification accuracy (ACC) (%) and Area Under the Receiver Operating Characteristic Curve (AUC) to measure the performance, where ACC is calculated as:

$$
A C C = \frac {T P + T N}{T P + T N + F P + F N} \tag {20}
$$

AUC is the area under the ROC curve, indicating the model’s ability to distinguish between classes. Ranging from 0 to 1, a higher AUC signifies better detection performance.

## Evaluation of Defense Scheme COIN

The test accuracy (%) results are presented in Tabs. 2 to 4. “**AVG**” denotes the average value for each row. The values covered by gray denote the best defense effect. It can be observed the average performance of existing defenses against CUDA lag behind COIN, ranging from 19.17% to 44.63% as shown in Tab. 2. Meanwhile, COIN maintains an advantage of 0.88%-26.48% on the large dataset, 8.4%-47.4% on the medium dataset as shown in Tab. 3. The reason AT lags behind COIN can be deduced from that the multiplicative noise underlying convolutional UEs is more easily defeated by the multiplicative random noise used by COIN, rather than the additive noise on which AT is based (Madry et al. 2018). We further evaluate defenses against VUDA and HUDA, as shown in Tab. 4. It can also be seen that COIN is effective in defending against these convolution-based UEs and performs better than other defense solutions by 23.84% to 40%. Regarding practicality, while we acknowledge that COIN cannot be effective for all types of UE, it still shows effectiveness against some bounded UEs including URP (Tao et al. 2021), OPS (Wu et al. 2023), which improves test accuracy from 16.8%, 28.4% to 81.1%, 80.1%.

## Evaluation of Detection Scheme EPD

We formulate the detection scenario with a dataset (size of 50000) combining an even distribution of six UEs among 12 SOTA UEs CUDA (Sadasivan, Soltanolkotabi, and Feizi 2023), HUDA, VUDA, OPS (Wu et al. 2023), AR (Sandoval-Segura et al. 2022), URP (Tao et al. 2021), UHP (Tao et al. 2021), LSP (Yu et al. 2022), TAP (Fowl et al. 2021), EM (Huang et al. 2021), EFP (Wen et al. 2023), and SEP (Chen et al. 2023). We set the following combinations for CIFAR-10:

<small><span class="docvortex-page-footnote" data-block-type="page_footnote" style="color:#6b7280"><sup>2</sup>Our code is available at https://github.com/wxldragon/COIN.</span></small>

<table><tr><td>Datasets</td><td colspan="5">CUDA CIFAR-10</td><td colspan="5">HUDA CIFAR-10</td></tr><tr><td>Defenses↓Models→</td><td>RN18</td><td>VGG16</td><td>DN121</td><td>MNV2</td><td>AVG</td><td>RN18</td><td>VGG16</td><td>DN121</td><td>MNV2</td><td>AVG</td></tr><tr><td>w/o</td><td>40.36</td><td>43.95</td><td>44.03</td><td>36.86</td><td>41.30</td><td>39.65</td><td>36.63</td><td>50.32</td><td>34.73</td><td>40.53</td></tr><tr><td>MU</td><td>41.13</td><td>41.66</td><td>39.98</td><td>39.55</td><td>40.58</td><td>43.45</td><td>44.54</td><td>48.96</td><td>42.01</td><td>43.91</td></tr><tr><td>CM</td><td>42.15</td><td>37.71</td><td>39.62</td><td>37.95</td><td>39.36</td><td>38.56</td><td>39.68</td><td>50.38</td><td>41.37</td><td>41.87</td></tr><tr><td>CO</td><td>39.93</td><td>38.95</td><td>36.64</td><td>38.37</td><td>38.47</td><td>40.53</td><td>40.18</td><td>47.74</td><td>36.58</td><td>40.70</td></tr><tr><td>DP-SGD</td><td>39.39</td><td>41.01</td><td>46.04</td><td>37.26</td><td>40.93</td><td>36.76</td><td>41.08</td><td>46.34</td><td>35.75</td><td>40.17</td></tr><tr><td>AVATAR</td><td>39.25</td><td>38.68</td><td>41.64</td><td>39.37</td><td>39.74</td><td>43.53</td><td>39.84</td><td>52.10</td><td>46.28</td><td>42.27</td></tr><tr><td>AA</td><td>51.87</td><td>10.00</td><td>60.09</td><td>51.42</td><td>43.35</td><td>56.83</td><td>10.00</td><td>60.09</td><td>61.08</td><td>46.27</td></tr><tr><td>OP</td><td>42.67</td><td>39.26</td><td>46.25</td><td>41.04</td><td>42.31</td><td>45.36</td><td>46.77</td><td>59.80</td><td>51.79</td><td>49.21</td></tr><tr><td>ISS-G</td><td>38.50</td><td>32.88</td><td>38.46</td><td>33.66</td><td>35.88</td><td>22.43</td><td>43.08</td><td>29.33</td><td>34.52</td><td>33.05</td></tr><tr><td>ISS-J</td><td>51.49</td><td>45.00</td><td>45.54</td><td>38.57</td><td>45.15</td><td>49.89</td><td>49.48</td><td>42.23</td><td>40.58</td><td>45.47</td></tr><tr><td>AT</td><td>43.40</td><td>38.68</td><td>46.88</td><td>36.33</td><td>41.32</td><td>48.62</td><td>41.19</td><td>50.48</td><td>43.74</td><td>45.07</td></tr><tr><td>ECLIPSE</td><td>31.09</td><td>33.47</td><td>35.56</td><td>29.86</td><td>32.49</td><td>34.57</td><td>39.04</td><td>41.12</td><td>35.35</td><td>37.52</td></tr><tr><td>COIN (Ours)</td><td>74.98</td><td>68.74</td><td>74.96</td><td>71.07</td><td>72.44</td><td>75.18</td><td>71.75</td><td>75.78</td><td>70.10</td><td>73.05</td></tr></table>

Table 4: Defenses against VUDA and HUDA. The test accuracy (%) results on CIFAR-10 dataset.

<table><tr><td rowspan="2">Datasets Settings</td><td colspan="5">CIFAR-10 UEs</td><td colspan="4">ImageNet20 UEs</td></tr><tr><td> $\mathcal{S}_1$ </td><td> $\mathcal{S}_2$ </td><td> $\mathcal{S}_3$ </td><td> $\mathcal{S}_4$ </td><td>AVG</td><td> $\mathcal{S}_5$ </td><td> $\mathcal{S}_6$ </td><td> $\mathcal{S}_7$ </td><td>AVG</td></tr><tr><td>ACC</td><td>90.40</td><td>90.94</td><td>87.89</td><td>87.63</td><td>89.21</td><td>99.92</td><td>99.93</td><td>99.89</td><td>99.91</td></tr><tr><td>AUC</td><td>0.904</td><td>0.909</td><td>0.878</td><td>0.876</td><td>0.891</td><td>0.999</td><td>0.999</td><td>0.998</td><td>0.998</td></tr></table>

Table 5: Evaluation of EPD. The ACC (%) and AUC results with diverse settings on CIFAR-10 and ImageNet datasets.

$\mathcal { S } _ { 1 }$ −→ CUDA, VUDA, HUDA, OPS, AR, URP
$\mathcal { S } _ { 2 }$ −→ CUDA, VUDA, HUDA, OPS, LSP, URP
$\mathcal { S } _ { 3 }$ −→ CUDA, VUDA, HUDA, TAP, EM, EFP
$\mathcal { S } _ { 4 }$ −→ CUDA, VUDA, HUDA, OPS, EFP, SEP,

and the following is for ImageNet-20:

$\mathcal { S } _ { 5 }$ −→ CUDA, VUDA, HUDA, LSP, AR, URP
$\mathcal { S } _ { 6 }$ −→ CUDA, VUDA, HUDA, EM, TAP, UHP
$\mathcal { S } _ { 7 }$ −→ CUDA, VUDA, HUDA, TAP, UHP, AR.

The results of EPD on CIFAR-10 and ImageNet20 are shown in Tab. 5. It can be seen that EPD’s ACC and AUC are both high, demonstrating excellent detection performance.

## Hyperparameter Analysis

We investigate the impact of α on the effectiveness of COIN, as shown in Fig. 6 (a). The test accuracy against CUDA increases initially with the rise in α and then starts to decrease. This is because initially, as α increases, the CUDA perturbations gradually become disrupted. However, as α continues to increase, excessive corruptions damage image features, compromising defense effect. We empirically opt for an α of 2.0 that yields the highest average defense effect. Additionally, given that the penalty parameter $C _ { p }$ is a key hyperparameter in SVM training for regulating overfitting and underfitting, we examine its effect on EPD performance as illustrated in Fig. 6 (b). It can be seen that the average performance is best when $C _ { p }$ is set to 0.1, which is also the default parameter used in our experiments.

## Related Work

## Unlearnable Examples

Current UEs can be classified into bounded UEs and convolution-based UEs. The bounded UE methods utilize shortcut-based approaches to generate additive unlearnable noise (Huang et al. 2021; Fowl et al. 2021; Fu et al. 2022; Wu et al. 2023). Nonetheless, recent studies indicate that commonly used defenses like JPEG compression can address these bounded UEs (Liu, Zhao, and Larson 2023). Recently, Sadasivan, Soltanolkotabi, and Feizi propose a new type of convolution-based UE via class-wise convolutional filters to generate multiplicative noise without norm constraints (CUDA). Unfortunately, all currently available defense approaches prove ineffective against it, and no current research has explored viable defense strategies. To expand the scope of convolution-based UEs, we propose two new types of convolution-based UEs via vertical filters and horizontal filters, termed as VUDA and HUDA, revealing that these convolution-based UEs easily defeat existing defenses.

![](images/image_06.jpg)

Figure 6: Hyper-parameter analysis: (a) The impact of α on the effectiveness of COIN against CUDA. (b) The impact of penalty parameter $C _ { p }$ on the average effectiveness of EPD across from $\mathcal { S } _ { 1 }$ to $\mathcal { S } _ { 4 }$ .

## Defenses Against UEs

Tao et al. (2021) experimentally and theoretically demonstrate AT (Madry et al. 2018) can effectively defend against bounded UEs, and Liu et al. (2023) discover that even simple image transformation such as JPEG compression can effectively defend against UEs. Thereafter, Qin et al. (2023) employ adversarial augmentations (AA), Dolatabadi et al. (2023) (AVATAR) and Wang et al. (2024a) (ECLIPSE) purify UEs through diffusion models (Ho, Jain, and Abbeel 2020), Segura et al. (2023) train a linear regression model to perform orthogonal projection (OP), while Yu et al. (2024) corrupt UEs via rate-constrained variational auto-encoders. Nevertheless, none of these defense approaches are tailormade for convolution-based UEs, with all of them failing against them. As for detection, although Zhu et al. (2024) first propose a detection method for UEs, it is limited to bounded UEs and does not consider convolution types.

## Conclusion

In this work, we reveal that existing defenses are all ineffective against convolution-based UEs. To address this issue, we first propose an edge-pixel detection scheme to identify convolutional samples. Besides, we model this process using multiplicative matrices applied to samples and propose two metrics, demonstrating that increasing these metrics mitigates convolution-based UEs, achieved through the design of a random matrix. Based on this, we propose the first effective defense against convolution-based UEs, utilizing pixel-level random multiplication via interpolation. We further propose two convolution-based UEs, termed as VUDA and HUDA, to expand the scope of convolutional UEs. Extensive experiments verify the effectiveness of our proposed defenses against convolution-based UEs.

## References

- Chen, S.; Yuan, G.; Cheng, X.; Gong, Y.; Qin, M.; Wang, Y.; and Huang, X. 2023. Self-ensemble protection: Training checkpoints are good data protectors. In Proceedings of the 11th International Conference on Learning Representations (ICLR’23).
- Deng, J.; Dong, W.; Socher, R.; Li, L.-J.; Li, K.; and Fei-Fei, L. 2009. Imagenet: A large-scale hierarchical image database. In Proceedings of the 2009 IEEE/CVF Conference on Computer Vision and Pattern Recognition (CVPR’09), 248–255.
- DeVries, T.; and Taylor, G. W. 2017. Improved regularization of convolutional neural networks with cutout. arXiv preprint arXiv:1708.04552.
- Dolatabadi, H. M.; Erfani, S.; and Leckie, C. 2023. The devil’s advocate: Shattering the illusion of unexploitable data using diffusion models. arXiv preprint arXiv:2303.08500.
- Fowl, L.; Goldblum, M.; Chiang, P.-y.; Geiping, J.; Czaja, W.; and Goldstein, T. 2021. Adversarial examples make strong poisons. In Proceedings of the 35th Neural Information Processing Systems (NeurIPS’21), volume 34, 30339–30351.
- Friedman, N.; Geiger, D.; and Goldszmidt, M. 1997. Bayesian network classifiers. Machine Learning, 29: 131–163.
- Fu, S.; He, F.; Liu, Y.; Shen, L.; and Tao, D. 2022. Robust unlearnable examples: Protecting data privacy against adversarial learning. In Proceedings of the 10th International Conference on Learning Representations (ICLR’22).
- He, K.; Zhang, X.; Ren, S.; and Sun, J. 2016. Deep residual learning for image recognition. In Proceedings of the 2016 IEEE/CVF Conference on Computer Vision and Pattern Recognition (CVPR’16), 770–778.
- Ho, J.; Jain, A.; and Abbeel, P. 2020. Denoising diffusion probabilistic models. In Proceedings of the 34th Neural Information Processing Systems (NeurIPS’20), volume 33, 6840–6851.
- Hong, S.; Chandrasekaran, V.; Kaya, Y.; Dumitraş, T.; and Papernot, N. 2020. On the effectiveness of mitigating data poisoning attacks with gradient shaping. arXiv preprint arXiv:2002.11497.
- Huang, G.; Liu, Z.; Van Der Maaten, L.; and Weinberger, K. Q. 2017. Densely connected convolutional networks. In Proceedings of the 2017 IEEE/CVF Conference on Computer Vision and Pattern Recognition (CVPR’17), 4700–4708.
- Huang, H.; Ma, X.; Erfani, S. M.; Bailey, J.; and Wang, Y. 2021. Unlearnable examples: Making personal data unexploitable. In Proceedings of the 9th International Conference on Learning Representations (ICLR’21).
- Huang, S.; Cai, N.; Pacheco, P. P.; Narrandes, S.; Wang, Y.; and Xu, W. 2018. Applications of support vector machine learning in cancer genomics. Cancer genomics & proteomics, 15(1): 41–51.
- Javanmard, A.; and Soltanolkotabi, M. 2022. Precise statistical analysis of classification accuracies for adversarial training. The Annals of Statistics, 50(4): 2127–2156.
- Krizhevsky, A.; and Hinton, G. 2009. Learning multiple layers of features from tiny images.
- Lin, X.; Yu, Y.; Xia, S.; Jiang, J.; Wang, H.; Yu, Z.; Liu, Y.; Fu, Y.; Wang, S.; Tang, W.; et al. 2024. Safeguarding medical image segmentation datasets against unauthorized training via contourand texture-aware perturbations. arXiv preprint arXiv:2403.14250.
- Liu, S.; Wang, Y.; and Gao, X.-S. 2024. Game-theoretic unlearnable example generator. In Proceedings of the AAAI Conference on Artificial Intelligence (AAAI’24), volume 38, 21349–21358.
- Liu, X.; Li, M.; Wang, H.; Hu, S.; Ye, D.; Jin, H.; Wu, L.; and Xiao, C. 2023. Detecting Backdoors During the Inference Stage
- Based on Corruption Robustness Consistency. In Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition (CVPR’23), 16363–16372.
- Liu, Y.; Xu, K.; Chen, X.; and Sun, L. 2024. Stable unlearnable example: Enhancing the robustness of unlearnable examples via stable error-minimizing noise. In Proceedings of the AAAI Conference on Artificial Intelligence (AAAI’24), volume 38, 3783–3791.
- Liu, Z.; Zhao, Z.; and Larson, M. 2023. Image shortcut squeezing: Countering perturbative availability poisons with compression. In Proceedings of the 40th International Conference on Machine Learning (ICML’23).
- Madry, A.; Makelov, A.; Schmidt, L.; Tsipras, D.; and Vladu, A. 2018. Towards deep learning models resistant to adversarial attacks. In Proceedings of the 6th International Conference on Learning Representations (ICLR’18).
- Meng, R.; Yi, C.; Yu, Y.; Yang, S.; Shen, B.; and Kot, A. C. 2024. Semantic deep hiding for robust unlearnable examples. IEEE Transactions on Information Forensics and Security (TIFS’24).
- Min, Y.; Chen, L.; and Karbasi, A. 2021. The curious case of adversarially robust models: More data can help, double descend, or hurt generalization. In Uncertainty in Artificial Intelligence, 129–139. PMLR.
- Qin, T.; Gao, X.; Zhao, J.; Ye, K.; and Xu, C.-Z. 2023. Learning the unlearnable: Adversarial augmentations suppress unlearnable example attacks. arXiv preprint arXiv:2303.15127.
- Ren, J.; Xu, H.; Wan, Y.; Ma, X.; Sun, L.; and Tang, J. 2023. Transferable unlearnable examples. In Proceedings of the 11th International Conference on Learning Representations (ICLR’23).
- Reynolds, D. A. 2009. Gaussian mixture models. Encyclopedia of Biometrics, 741(659-663).
- Sadasivan, V. S.; Soltanolkotabi, M.; and Feizi, S. 2023. Cuda: Convolution-based unlearnable datasets. In Proceedings of the 2023 IEEE/CVF Conference on Computer Vision and Pattern Recognition (CVPR’23), 3862–3871.
- Sandler, M.; Howard, A.; Zhu, M.; Zhmoginov, A.; and Chen, L.- C. 2018. Mobilenetv2: Inverted residuals and linear bottlenecks. In Proceedings of the 2018 IEEE Conference on Computer Vision and Pattern Recognition (CVPR’18), 4510–4520.
- Sandoval-Segura, P.; Singla, V.; Geiping, J.; Goldblum, M.; Goldstein, T.; and Jacobs, D. W. 2022. Autoregressive perturbations for data poisoning. In Proceedings of the 36th Neural Information Processing Systems (NeurIPS’22), volume 35.
- Segura, P. S.; Singla, V.; Geiping, J.; Goldblum, M.; and Goldstein, T. 2023. What can we learn from unlearnable datasets? In Proceedings of the 37th Neural Information Processing Systems (NeurIPS’23).
- Simonyan, K.; and Zisserman, A. 2014. Very deep convolutional networks for large-scale image recognition. arXiv preprint arXiv:1409.1556.
- Tao, L.; Feng, L.; Yi, J.; Huang, S.-J.; and Chen, S. 2021. Better safe than sorry: Preventing delusive adversaries with adversarial training. In Proceedings of the 35th Neural Information Processing Systems (NeurIPS’21), volume 34, 16209–16225.
- Wang, X.; Hu, S.; Zhang, Y.; Zhou, Z.; Zhang, L. Y.; Xu, P.; Wan, W.; and Jin, H. 2024a. ECLIPSE: Expunging clean-label indiscriminate poisons via sparse diffusion purification. In Proceedings of the 29th European Symposium on Research in Computer Security (ESORICS’24).
- Wang, X.; Li, M.; Liu, W.; Zhang, H.; Hu, S.; Zhang, Y.; Zhou, Z.; and Jin, H. 2024b. Unlearnable 3D point clouds: Class-wise transformation is all you need. In Proceedings of the 38th Conference on Neural Information Processing Systems (NeurIPS’24).
- Wang, X.; Li, M.; Xu, P.; Liu, W.; Zhang, L. Y.; Hu, S.; and Zhang, Y. 2024c. PointAPA: Towards availability poisoning attacks in 3D point clouds. In Proceedings of the European Symposium on Research in Computer Security (ESORICS’24), 125–145. Springer.
- Wen, R.; Zhao, Z.; Liu, Z.; Backes, M.; Wang, T.; and Zhang, Y. 2023. Is adversarial training really a silver bullet for mitigating data poisoning? In Proceedings of the 11th International Conference on Learning Representations (ICLR’23).
- Wu, S.; Chen, S.; Xie, C.; and Huang, X. 2023. One-pixel shortcut: On the learning preference of deep neural networks. In Proceedings of the 11th International Conference on Learning Representations (ICLR’23).
- Yu, D.; Zhang, H.; Chen, W.; Yin, J.; and Liu, T.-Y. 2022. Availability attacks create shortcuts. In Proceedings of the 28th ACM SIGKDD Conference on Knowledge Discovery and Data Mining (KDD’22), 2367–2376.
- Yu, Y.; Wang, Y.; Xia, S.; Yang, W.; Lu, S.; Tan, Y.-P.; and Kot, A. C. 2024. Purify unlearnable examples via rate-constrained variational autoencoders. In Proceedings of the 41th International Conference on Machine Learning (ICML’24).
- Yun, S.; Han, D.; Oh, S. J.; Chun, S.; Choe, J.; and Yoo, Y. 2019. CutMix: Regularization strategy to train strong classifiers with localizable features. In Proceedings of the 17th International Conference on Computer Vision (ICCV’19).
- Zhang, H.; Cisse, M.; Dauphin, Y. N.; and Lopez-Paz, D. 2018. Mixup: Beyond empirical risk minimization. In Proceedings of the 6th International Conference on Learning Representations (ICLR’18).
- Zhang, L.; Shen, B.; Barnawi, A.; Xi, S.; Kumar, N.; and Wu, Y. 2021. FedDPGAN: Federated differentially private generative adversarial networks framework for the detection of COVID-19 pneumonia. Information Systems Frontiers, 23(6): 1403–1415.
- Zhu, Y.; Yu, L.; and Gao, X.-S. 2024. Detection and defense of unlearnable examples. In Proceedings of the AAAI Conference on Artificial Intelligence (AAAI’24), volume 38, 17211–17219.