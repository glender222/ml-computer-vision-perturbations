# CUDA: Convolution-based Unlearnable Datasets

Vinu Sankar Sadasivan University of Maryland

vinu@umd.edu

Mahdi Soltanolkotabi University of Southern California

soltanol@usc.edu

Soheil Feizi University of Maryland

sfeizi@cs.umd.edu

## Abstract

Large-scale training of modern deep learning models heavily relies on publicly available data on the web. This potentially unauthorized usage of online data leads to concerns regarding data privacy. Recent works aim to make unlearnable data for deep learning models by adding small, specially designed noises to tackle this issue. However, these methods are vulnerable to adversarial training (AT) and/or are computationally heavy. In this work, we propose a novel, model-free, Convolution-based Unlearnable DAtaset (CUDA) generation technique. CUDA is generated using controlled class-wise convolutions with filters that are randomly generated via a private key. CUDA encourages the network to learn the relation between filters and labels rather than informative features for classifying the clean data. We develop some theoretical analysis demonstrating that CUDA can successfully poison Gaussian mixture data by reducing the clean data performance of the optimal Bayes classifier. We also empirically demonstrate the effectiveness of CUDA with various datasets (CIFAR-10, CIFAR-100, ImageNet-100, and Tiny-ImageNet), and architectures (ResNet-18, VGG-16, Wide ResNet-34-10, DenseNet-121, DeIT, EfficientNetV2-S, and MobileNetV2). Our experiments show that CUDA is robust to various data augmentations and training approaches such as smoothing, AT with different budgets, transfer learning, and fine-tuning. For instance, training a ResNet-18 on ImageNet-100 CUDA achieves only 8.96%, 40.08%, and 20.58% clean test accuracies with empirical risk minimization (ERM), L∞ AT, and L<sub>2</sub> AT, respectively. Here, ERM on the clean training data achieves a clean test accuracy of 80.66%. CUDA exhibits unlearnability effect with ERM even when only a fraction of the training dataset is perturbed. Furthermore, we also show that CUDA is robust to adaptive defenses designed specifically to break it.

## 1. Introduction

Modern deep learning training frameworks heavily depend on large-scale datasets for achieving high accuracy.

This encourages deep learning practitioners to scrape data from the web for data collection [8, 31, 39, 42]. Since a lot of the data is publicly available online, sometime this scrapping of data is unauthorized. For instance, a recent article [15] discloses that a private company trained a commercial face recognition system using over three billion facial images collected from the internet without any user consent. Although such massive data can significantly boost the performance of deep learning models, it raises serious concerns about data privacy and security.

To prevent the unauthorized usage of personal data, a series of recent papers [10,17,55] propose to poison data with additive noise. The idea is to make datasets unlearnable for deep learning models by ensuring that they learn the correspondence between noises and labels. Thereby, they do not learn much useful information about the clean data, significantly degrading their clean test accuracy. However, in recent works [11, 17, 49], these unlearnability methods are shown to be vulnerable to adversarial training (AT) frameworks [34]. Motivated by this problem, Fu et al. [11] developed Robust Error-Minimization (REM) noises to make unlearnable data that is protected from AT. While the authors show the effectiveness of REM in multiple scenarios, we demonstrate that these methods are still not robust against different data augmentations or training settings (see Section 3.2). Furthermore, current unlearnability frameworks [10, 11, 17, 55] are model-dependent and require expensive optimization steps on deep learning models to obtain the additive noises. They also need to train the deep learning models from scratch to obtain noises for each new data set.

In this paper, we propose a novel Convolution-based Unlearnable DAtaset (CUDA) generation technique. We address limitations of existing unlearnable data generation techniques in Section 3.2 and motivate our CUDA technique in Section 3.3. For generating CUDA, an attacker randomly generates different convolutional filters for each class in the dataset using a private key or seed value. These filters are used to perform controlled class-wise convolutions on the clean training dataset to obtain CUDA. As we describe in Sections 3.3 and 5.2, CUDA generation performs controlled convolutions using a blur parameter $p _ { b }$ to ensure that the semantics of the dataset are preserved (see Figure 1). CUDA generation with a lower blurring parameter $p _ { b }$ adds less perceptible noises to clean samples. A network trained by a defender on CUDA is encouraged to learn the shortcut relation between class-wise convolutional filters and labels rather than useful features for classifying the clean data. Since the seed value for generating the filters are private, its not possible for the defender to obtain clean data from CUDA alone. Additionally, CUDA exhibits unlearnability effect with ERM even when only a fraction of the training dataset is perturbed (see Section 5). While the existing unlearnability works use additive noises, CUDA generation technique enjoys the advantage of introducing multiplicative noises in the Fourier domain due to the convolution theorem (since convolution of signals is the same as element-wise multiplication in the Fourier domain). This lets CUDA generation add higher amounts of noise in the image space, specifically along the edges in images, and makes it resilient to AT with small additive noise budgets. In Figure 2, with the help of t-SNE plots [51], we also find that the noises added by CUDA generation is not linearly separable while they are linearly separable for the existing works on unlearnability [54].

<small><span class="docvortex-page-footnote" data-block-type="page_footnote" style="color:#6b7280">Code available here: [https://github.com/vinusankars/Convolution-based-Unlearnability](https://github.com/vinusankars/Convolution-based-Unlearnability)</span></small>

![](images/image_01.jpg)

Figure 1. CIFAR-10 CUDA images from each of the 10 classes convolved using convolutional filters generated with blur parameter p<sub>b</sub>. As seen in the plot, a higher $p _ { b }$ value results in better unlearnability (lower clean test accuracy), but increased blurring. CIFAR-10 CUDA does not break with ERM for all the three $p _ { b }$ values. In Section 5.2, we propose Deconvolution-based Adversarial Training (DAT) that we specifically design to break CUDA. DAT adversarially learns class-wise filters to deconvolve CUDA images. However, CIFAR-10 CUDA with $p _ { b }   \in   \{ 0 . 3 , 0 . 5 \}$ does not break with DAT. Therefore, we select $p _ { b }   =   0 . 3$ (lesser blurring) for CIFAR-10 CUDA as discussed in Section 5.1. More details on DAT in supplementary material.

In Section 4, we theoretically show that CUDA generation can successfully poison Gaussian mixture data by degrading the clean data accuracy of the optimal Bayes classifier. We state our result informally below while the formal version is presented in Theorem 2.

**Theorem 1 (Informal)** Let D denote a Gaussian mixture data with two modes, $P _ { \mathcal { D } }$ denote the optimal Bayes classifier trained on $\mathcal { D } ,$ and $\tau _ { \mathcal { D } } ( P _ { \mathcal { D } } )$ denote the accuracy of the classifier $P _ { \mathcal { D } }$ on $\mathcal { D } .$ Then, under some assumptions, for every clean data D, there is a CUDA $\tilde { \mathcal { D } }$ such that

$$
\tau_ {\mathcal {D}} (P _ {\tilde {\mathcal {D}}}) <   \tau_ {\mathcal {D}} (P _ {\mathcal {D}}).
$$

Furthermore, our empirical experiments in Section 5 demonstrate the effectiveness of CUDA under various training scenarios such as ERM with various augmentations and regularizations, AT with different budgets, randomized smoothing [6, 22, 27], transfer learning, and fine-tuning. For instance, training a ResNet-18 on CIFAR-10 CUDA achieves only 18.48%, 44.4%, and 51.14% clean test accuracies with ERM, $L _ { \infty }$ AT, and $L _ { 2 }$ AT, respectively (see Figure 2). Here, ERM on the clean training data achieves a clean test accuracy of 94.66%. In addition, we also design adaptive defenses to investigate if CUDA breaks with random or adversarial defense mechanisms. We find that CUDA is robust to the adaptive defenses that we specifically design to break it.

## 2. Related Works

Our CUDA generation technique is intimately related with adversarial and poisoning attacks. We first discuss some of this literature and then explain their relation with CUDA generation.

**Adversarial attacks.** Adversarial examples are specially designed examples that can fool deep learning models at test time [4, 12, 23, 24, 47]. The adversary crafts these examples by adding error-maximizing noises to the clean data. Even slightly perturbed data can serve as adversarial examples. AT is a training framework proposed to make deep learning models robust to adversarial examples [19,25,34,53,59]. AT is a min-max optimization problem where the model is trained to minimize loss on adversarial examples that have the maximum loss.

**Poisoning attacks.** In data poisoning, an attacker aims to hurt the deep learning model’s performance by perturbing the training data [2, 20, 28, 32, 43, 52]. The backdoor attack is a special type of poisoning attack where a trigger pattern is injected into clean data at training time [5, 29, 33, 36]. The model trained on this data would misclassify an image with a trigger pattern at test time. Gu et al. [13] and Li et al. [30] use perceptible amounts of noises similar to CUDA for data poisoning. However, backdoor attacks do not affect the performance of the model on clean data [1, 5, 43].

![](images/image_02.jpg)

Figure 2. In the left column, clean test accuracies of ResNet-18 on clean CIFAR-10 and CIFAR-10 CUDA are shown. The lower the test accuracy, the higher the effectiveness of unlearnability is. We show that CUDA is robust to various training settings such as $\mathrm { E R M } ,   L _ { 2 }   \mathrm { A T } ,$ and $L _ { \infty } \; \mathrm { A T } .$ The error bars represent the standard deviation of test accuracy over 5 independent trials. CUDA is robust to the randomness in its data generation process. In the right column, t-SNEs of the noises generated by REM (top) and CUDA (bottom) for the first 3 classes of CIFAR-10 are plotted.

Recent literature utilize data poisoning to protect data from being used for model training without authorization. Yuan and Wu [55] use neural tangent kernels [18] to generate clean label attacks that can hurt the generalization of deep learning models. Huang et al. [17] show that errorminimizing noise addition can serve as a poisoning technique. Fowl et al. [10] show that error-maximizing noises as well can make strong poison attacks. However, all these poisoning techniques do not offer data protection with AT [49, 54]. Fu et al. [11] proposes a min-min-max optimization technique to generate poisoned data that offers better unlearnability effects with AT.

## 3. Convolution-based Ulearnable DAtaset (CUDA)

In this section, first we give some preliminaries about unlearnability. Then we discuss the limitations of the existing unlearnability methods. Finally, we propose our CUDA generation technique.

## 3.1. Preliminaries

Let $\{ ( { \pmb x } _ { i } , y _ { i } ) \} _ { i = 1 } ^ { n } \; \sim \; \mathcal { D } ^ { n }$ be the clean training dataset where D is the clean data distribution, $\pmb { x } _ { i } \in \mathcal { X } \subset \mathbb { R } ^ { d }$ are the samples, and $y _ { i } \in \mathcal { Y }$ are the corresponding labels. Suppose a network is given as $f _ { \theta } \; : \; { \mathcal { X } } \; \to \; { \mathcal { Y } }$ where $\theta \in \Theta$ is the network parameter and $\ell : \mathcal { Y } \times \mathcal { Y } \rightarrow$ R is the loss function. ERM trains a network using a minimization problem of the form: minθ $\textstyle { \frac { 1 } { n } } \sum _ { i = 1 } ^ { n } \ell ( f _ { \theta } ( { \pmb x } _ { i } ) , y _ { i } )$ . Standard $L _ { p ^ { - } }$ based AT $( \operatorname { f o r } p \in \mathbb { R } ^ { + } )$ solves the following min-max problem: min<sub>θ</sub> $\textstyle \frac { 1 } { n } \sum _ { i = 1 } ^ { n } \operatorname* { m a x } _ { \| \pmb { \delta } _ { i } \| _ { p } \leq \rho _ { a } } \ell ( f _ { \theta } ( \pmb { x } _ { i } { + } \pmb { \delta } _ { i } ) , y _ { i } )$ where $\rho _ { a }$ is the adversarial perturbation radius.

We will use $\tau _ { \mathcal { D } } ( \theta )$ , with θ the clean model parameter, to denote the clean test accuracy of a model trained on the clean training dataset (i.e., clean model accuracy). In unlearnable dataset generation, an attacker uses an algorithm $\mathcal { A } : \mathcal { X } \to \mathcal { X }$ to generate an unlearnable dataset $\{ ( \tilde { \pmb { x } } _ { i } = \mathcal { A } ( \pmb { x } _ { i } ) , y _ { i } ) \} _ { i = 1 } ^ { n } \sim \tilde { \mathcal { D } } ^ { n }$ from the clean training data. Here, the attacker assumes access to the full clean training datatset. Moreover, the attacker cannot modify the unlearnable dataset once it is released publicly. A defender trains using the unlearnable dataset to obtain a network $f _ { \tilde { \theta } } \cdot$ The objective of the attacker is to design an unlearnable dataset such that the defender’s model trained on the unlearnable data achieves a clean test accuracy (i.e., unlearnable model accuracy) worse than the clean model accuracy i.e. $\tau _ { \mathcal { D } } ( \tilde { \theta } ) \ll \tau _ { \mathcal { D } } ( \theta )$

## 3.2. Limitations of existing works

Fu et al. [11] show that the previous unlearnability methods including Error-Minimization (EM) [17], Targeted Adversarial Poisoning (TAP) [10], and Neural Tangent Generalization Attack (NTGA) [55] are vulnerable to AT. Hence, they propose a Robust Error-Minimization (REM) [11] method that exploits a min-min-max optimization procedure to generate unlearnable noises. REM first trains a noise generator $f _ { \theta } ^ { \prime }$ on data points $\{ ( \pmb { x } _ { i } , y _ { i } ) \} _ { i = 1 } ^ { n }$ over a loss func-

Table 1. Time taken for generating unlearnable noises for various datasets using different unlearnability techniques.

| Dataset | EM | TAP | NTGA | REM | CUDA |
| --- | --- | --- | --- | --- | --- |
| CIFAR-10 | 0.4 hr | 0.5 hr | 5.2 hrs | 22.6 hrs | 10.8 s |
| CIFAR-100 | 0.4 hr | 0.5 hr | 5.2 hrs | 22.6 hrs | 15.5 s |
| ImageNet-100 | 3.9 hrs | 5.2 hrs | 14.6 hrs | 51.2 hrs | 0.15 hr |

tion \` as follows

$$
\begin{array}{l} \min _ {\theta} \frac {1}{n} \sum_ {i = 1} ^ {n} \min _ {\| \boldsymbol {\delta} _ {i} ^ {u} \| \leq \rho_ {u}} \mathbb {E} _ {t \sim T} \\ \max _ {\| \boldsymbol {\delta} _ {i} ^ {a} \| \leq \rho_ {a}} \ell \left(f _ {\theta} ^ {\prime} \left(t \left(\boldsymbol {x} _ {i} + \boldsymbol {\delta} _ {i} ^ {u}\right) + \boldsymbol {\delta} _ {i} ^ {a}\right), y\right). \tag {1} \\ \end{array}
$$

Here, T is a distribution over a set of transformations $\{ t :$ $\mathcal { X } \to \mathcal { X } \} ,   \rho _ { u }$ is the defensive perturbation radius, and $\rho _ { a }$ controls the protection level of REM against AT. After training the noise generator, an unlearnable example $( \tilde { x } , y )$ is generated via

$$
\begin{array}{l} \tilde {\boldsymbol {x}} = \boldsymbol {x} + \underset {\| \boldsymbol {\delta} ^ {u} \| \leq \rho_ {u}} {\arg \min} \mathbb {E} _ {t \sim T} \\ \max _ {\| \boldsymbol {\delta} ^ {a} \| \leq \rho_ {a}} \ell \left(f _ {\theta} ^ {\prime} \left(t \left(\boldsymbol {x} + \boldsymbol {\delta} ^ {u}\right) + \boldsymbol {\delta} ^ {a}\right), y\right) \tag {2} \\ \end{array}
$$

First, note that REM is computationally expensive since it needs to generate unlearnability noises through solving optimization equation 2. Moreover, the existing techniques are model-dependent and they require gradient-based training with a neural network to generate unlearnable data. They also require neural network training from scratch for every dataset that is to be made unlearnable. Table 1 shows the amount of time required to generate various unlearnable datasets using $\mathrm { N V I D I A ^ { \circledast } }$ Tesla V100 GPU and 10 CPU cores. CUDA generation is significantly faster than the existing methods since it uses a model-free approach (no training required). Furthermore, REM is sensitive to hyperparameters and norm-budgets of AT since they generate noises with fixed $L _ { \infty }$ norm budgets. For instance, a ResNet-18 trained on clean CIFAR-10 dataset achieves a clean test data accuracy of 94.66%. $L _ { \infty }$ AT with perturbation radius $\rho _ { a } = 4 / 2 5 5$ on REM CIFAR-10 data (generated using $\rho _ { u } = 8 / 2 5 5$ and $\rho _ { a }   =   4 / 2 5 5 )$ achieves only a clean test accuracy of 48.16%. However, $L _ { \infty } ~ \mathrm { A T }$ with perturbation radius $\rho _ { a } = 8 / 2 5 5$ and $L _ { 2 } \; \mathrm { A T }$ with perturbation radius $\rho _ { a } = 0 . 7 5$ can achieve a clean test accuracy of 78.71% and 79.65%, respectively, on the same REM data. We also find that ERM with a ResNet-18 on grayscaled REM CIFAR-10 images can achieve a high test accuracy of 70.76% on the grayscaled CIFAR-10 test data. This shows that REM relies upon the color space for poisoning clean data. Fu et al. [11] also show that REM noise generated using ResNet-18 is not transferable to DenseNet-121.

## 3.3. Our method: CUDA

The major limitations of the previous works are that they are vulnerable to AT, and computationally expensive. We think the major reason for the former limitation is the usage of small additive noises for unlearnability. AT is designed to train in the presence of such additive noises. Increasing the budget of the amount of additive noises for unlearnability might destroy the semantics of the images while perturbing them. The latter limitation arises from the fact that these methods are model-dependent and they require multilevel optimizations. Hence, we are motivated to design a compute-efficient unlearnability method that is robust to AT. CUDA technique can perform convolutions to add larger amounts of noises to clean images without destroying its semantics. This can help CUDA to be robust against AT. CUDA uses randomly generated convolution filters for blurring images from each class. This makes a model trained on CUDA to learn shortcut relations between filters and labels. We empirically support these claims in Section 5.2. Additionally, randomly generating the filters makes our technique model-free. Since the keys for generating the filters are private, it is not possible to reverse the blurring effect in CUDA without having access to the corresponding clean images. Hence, we assume that the data publisher deletes the clean images after perturbing them. Moreover, CUDA technique is a novel class of non-additive noise based poisoning attack that needs to be studied.

CUDA uses convolutional filters $s _ { i } \in \mathbb { R } ^ { k \times k }$ for each class $i \in [ 1 , K ]$ . A random parameter, out of the $k ^ { 2 }$ parameters, in each of the filters is set to have a value of 1. The rest of the filter parameters are randomly initialized from a uniform distribution $\mathcal { U } ( 0 , p _ { b } )$ using a private seed where $p _ { b }$ is the blur parameter. Blur parameter controls the level of blurring that occurs when an image $\pmb { x }   \in   [ 0 , 1 ] ^ { d _ { 1 } \times d _ { 2 } \times d _ { 3 } }$ is convolved with a filter $s _ { i }$ . Here, $d _ { 1 } , d _ { 2 } ,$ , and $d _ { 3 }$ are the height, width, and number of channels of the image, respectively. For example, a CIFAR-10 image has a dimension of $3 2 \times 3 2 \times 3$ . The higher $p _ { b }$ is, the higher the blurring effect is. Let $\hat { \pmb { x } } = \pmb { x } * s _ { i }$ where x belongs to class i. The CUDA data point for x is given by $\tilde { \pmb { x } }   =   \hat { \pmb { x } } / \mathrm { M A X } ( \hat { \pmb { x } } )$ . Rescaling is performed to make sure that the CUDA image pixels lie between 0 and 1. We find that the unlearnability effect gets stronger with larger $p _ { b }$ and k values (see supplementary material for details).

## 4. Theory for CUDA

In this section, we define a binary classification setup similar to [19, 35] to theoretically analyze CUDA. Let D be a clean dataset modelled by an isotropic Gaussian mixture model given by $\mathcal { N } ( y \pmb { \mu } , I )$ , where $y \; \in \; \{ \pm 1 \}$ is the class label, $\pmb { \mu }   \in   \mathbb { R } ^ { d } ,$ , and $I   \in   \mathbb { R } ^ { d \times d }$ is the identity matrix. We defer the proofs for all the lemmas and theorem to the supplementary material. The Bayes optimal decision boundary for classifying this Gaussian mixture model is as follows.

![](images/image_03.jpg)

<details>
<summary>heatmap</summary>

| a1 | a-1 | Value (approx) |
| --- | --- | --- |
| 0.0~0.1 | 0.0~0.1 | 40~100 |
</details>

![](images/image_04.jpg)

<details>
<summary>heatmap</summary>

| a-1 (range) | a1 (range) | Value (range) |
| --- | --- | --- |
| 0.0~0.5 | 0.0~0.5 | 40~100 |
</details>

![](images/image_05.jpg)

<details>
<summary>heatmap</summary>

| a1 | a-1 | Value |
| --- | --- | --- |
| 0.0~0.5 | 0.0~0.5 | 40~100 |
</details>

![](images/image_06.jpg)

<details>
<summary>heatmap</summary>

| a-1 (range) | a1 (range) | Value (range) |
| --- | --- | --- |
| 0.0~0.5 | 0.0~0.5 | 40~100 |
</details>

(a) $\| \pmb { \mu } \| _ { 2 } = 1 . 0$

![](images/image_07.jpg)

<details>
<summary>heatmap</summary>

| a-1 (range) | a1 (range) | Value (range) |
| --- | --- | --- |
| 0.0~0.5 | 0.0~0.5 | -40~100 |
</details>

(b) $\| \pmb { \mu } \| _ { 2 } = 2 . 5$

![](images/image_08.jpg)

<details>
<summary>heatmap</summary>

| a-1 (range) | a1 (range) | Value (range) |
| --- | --- | --- |
| 0.0~0.5 | 0.0~0.5 | 40~100 |
</details>

(c) $\| \pmb { \mu } \| _ { 2 } = 5 . 0$

Figure 3. We plot the contour plots for clean test accuracy of the CUDA Bayes classifier $\tilde { P }$ (given as $\tau _ { \mathcal { D } } ( \tilde { P } ) )$ for varying $a _ { y }$ parameters where $y \in \{ \pm 1 \}$ in Figures 3(a–c). Gaussian mixture model $\mathcal { N } ( y \pmb { \mu } , I )$ denotes the clean data. In the top row, we use Lemmas $1 \; \& \; 4 \; !$ to empirically generate the plots. In the bottom row, we plot the theoretical upper bound of $\tau _ { \mathcal { D } } ( \tilde { P } )$ we obtain using Theorem 2.

**Lemma 1** The Bayes optimal decision boundary for classifying D is given by $P ( { \pmb x } ) \equiv { \pmb \mu } ^ { \top } { \pmb x } = 0 .$ . The accuracy of the decision boundary P on the clean dataset $\mathcal { D } ( i . e . \tau _ { \mathcal { D } } ( P ) )$ is equal to $\phi ( \| \pmb { \mu } \| _ { 2 } )$ . Here, $\phi ( . )$ represents the CDF of the standard normal distribution.

Now, to obtain $\tilde { \mathcal { D } } \; ( \mathrm { i . e . ,   C U D A } )$ , we perform class-wise 1D convolutions. Here, we consider a class of 1D convolutional filters with kernel size 3 of the form $\mathbf { \mathit { f } } _ { a }   =   [ a , 1 , a ]$ where $a \; \in \; [ 0 , 0 . 5 ]$ The convolution of signal $\pmb { x } \in \mathbb { R } ^ { d }$ with filter $f _ { a }$ using stride 1 (denoted by $\pmb { x } * \pmb { f } _ { \pmb { a } } ) ,$ , can be treated as a matrix operation Ax. Here, $A   \in   \mathbb { R } ^ { d \times d }$ is a tri-diagonal Toeplitz matrix denoted by $T ( d ; a , 1 , a )$ . For CUDA generation, we use class-wise convolution matrices $A _ { y } = T ( d ; a _ { y } , 1 , a _ { y } )$ to perturb a clean data point $( { \pmb x } , y )   \sim   { \mathcal D }$ to an unlearnable data point $( A _ { y } { \pmb x } , y )$ , where $a _ { y }   \in   [ 0 , 0 . 5 ]$ for $y   \in   \{ \pm 1 \}$ . Note that in CUDA, the labels remain the same. Next we show that such a perturbed dataset (i.e. CUDA) can be represented as a Gaussian mixture model.

**Lemma 2** A CUDA generated from clean data distribution D and the $A _ { y }   ^ { \prime } s$ defined above can be modelled as a Gaussian mixture model with the distribution given by $\tilde { \mathcal { D } } = \mathcal { N } ( y A _ { y } \pmb { \mu } , A _ { y } ^ { 2 } )$

To characterize the decision boundary for CUDA, we need to use some properties of Toeplitz matrices from Noschese et al. [37] given in the following Lemma 3.

**Lemma 3** Any tri-diagonal Toeplitz matrix $\begin{array} { r l r l } { A } & { { } } & { = } & { { } } \end{array}$ $T ( d ; a , 1 , a ) \in \mathbb { R } ^ { d \times d }$ with $a \in [ 0 , 0 . 5 ]$ can be diagonalized as $A   =   Q D Q$ where $\textstyle Q _ { i , j } \: = \: \big ( \frac { 2 } { d + 1 } \big ) ^ { 1 / 2 } \operatorname { s i n } \big ( \frac { i j \pi } { d + 1 } \big )$

and D is a diagonal matrix with $\begin{array} { r } { D _ { i , i }   =   1 + 2 a \cos \left( \frac { i \pi } { d + 1 } \right) } \end{array}$ $\mathit { f o r } \; 1 \leq i , j \leq d .$

Next we use Lemma 3 to show that the Bayes optimal decision boundary for classifying $\tilde { \mathcal { D } }$ is a quadratic plane.

**Lemma 4** Let $A _ { - 1 } = T ( d ; a _ { - 1 } , 1 , a _ { - 1 } )$ and $\begin{array} { r l } { A _ { 1 } } & { { } = } \end{array}$ $T ( d ; a _ { 1 } , 1 , a _ { 1 } )$ . The Bayes optimal decision boundary $f o r$ classifying $\tilde { \mathcal { D } }$ is given by $\tilde { P } ( \pmb { \dot { x } } ) \equiv \pmb { x } ^ { \top } A \pmb { x } + \pmb { b } ^ { \top } \pmb { x } + c \stackrel { \leftrightarrow } { = } 0 ,$ where $\vec { A } = \vec { A _ { - 1 } ^ { - 2 } } - \vec { A _ { 1 } ^ { - 2 } } , \vec { \pmb { b } } = 2 ( A _ { - 1 } ^ { - 1 } + A _ { 1 } ^ { - 1 } ) \pmb { \mu } ,$ and $\textstyle c = 2 \sum _ { i = 1 } ^ { d } [ \operatorname { l n } ( 1 { + } 2 a _ { - 1 } \operatorname { c o s } ( \frac { i \pi } { d + 1 } ) ) { - } { \operatorname { l n } ( 1 { + } 2 a _ { 1 } \operatorname { c o s } ( \frac { i \pi } { d + 1 } ) ) } ]$

Now we state a lemma regarding the tail of Gaussian quadratic forms which plays a crucial role in our main result.

**Lemma 5** $L e t \parallel \cdot \parallel$ denote the operator norm, $\| \cdot \| _ { 2 }$ denote the vector 2-norm, $z \sim \mathcal { N } ( \mathbf { 0 } , I )$ , and $\boldsymbol { Z } = \boldsymbol { z } ^ { \top } \hat { A } \boldsymbol { z }   +   \boldsymbol { z } ^ { \top } \boldsymbol { b }   +   \boldsymbol { c }$ where $A = Q \Lambda Q ^ { \top }$ . Using Chernoff bound, for any $t \geq 0$ and $\gamma \in \mathbb { R } ,$

$$
\begin{array}{l} \mathbb {P} \{Z - \mathbb {E} Z \geq \gamma \} \leq \\ \frac {\exp \left\{\frac {- t}{4 \| \boldsymbol {b} \| _ {2} ^ {2} \| \Lambda \|} - t (\gamma + \mathrm{Tr} (\Lambda) + \| \boldsymbol {b} \| _ {2}) \right\}}{| I - 2 t \Lambda | ^ {\frac {1}{2}}}. \\ \end{array}
$$

Lemma 5 allows us to provide an upper bound for the accuracy of the unlearnable decision boundary $\tilde { P }$ on the clean dataset D, given as $\tau _ { \mathcal { D } } ( \tilde { P } )$ , in Theorem 2 below.

**Theorem 2 (Main result)** Let $A = Q \Lambda Q = A _ { - 1 } ^ { - 2 } - A _ { 1 } ^ { - 2 } .$ For any non-negative constants $t _ { 1 }$ and $t _ { 2 } ,$ the accuracy of the unlearnable decision boundary $\tilde { P }$ on the clean dataset

## D can be upper-bounded as

$$
\begin{array}{l} \tau_ {\mathcal {D}} (\tilde {P}) \leq \frac {\exp \left[ t _ {1} \left(\boldsymbol {b} ^ {\top} \boldsymbol {\mu} + \boldsymbol {\mu} ^ {\top} A \boldsymbol {\mu} + c - \frac {1}{4 \| 2 A \boldsymbol {\mu} + \boldsymbol {b} \| _ {2} \| \Lambda \|}\right) \right]}{2 | I - 2 t _ {1} \Lambda | ^ {\frac {1}{2}}} \\ + \frac {\exp \left[ t _ {2} \left(\boldsymbol {b} ^ {\top} \boldsymbol {\mu} - \boldsymbol {\mu} ^ {\top} A \boldsymbol {\mu} - c - \frac {1}{4 \| 2 A \boldsymbol {\mu} - \boldsymbol {b} \| _ {2} \| \Lambda \|}\right) \right]}{2 | I - 2 t _ {2} \Lambda | ^ {\frac {1}{2}}} \\ \mathrel {\mathop :} = p _ {1} + p _ {2}. \\ \end{array}
$$

Furthermore, $\mathit { i f } \: \pmb { \mu } ^ { \top } A \pmb { \mu } + \pmb { b } ^ { \top } \pmb { \mu } + c + \mathrm { T r } ( A ) + \lVert 2 A \pmb { \mu } + \pmb { b } \rVert < 0 ,$ we have $\tau _ { \mathcal { D } } ( \tilde { P } ) \leq { \textstyle \frac { 1 } { 2 } } ( p _ { 1 } { + } 1 ) < 1$ . Also, $\it { i f - } \pmb { \mu } ^ { \top } A \pmb { \mu } + \pmb { b } ^ { \top } \pmb { \mu } -$ $c   -   \mathrm { T r } ( A )   +   \| 2 A { \pmb \mu }   -   { \pmb b } \| < 0 ,$ , we have $\tau _ { \mathcal { D } } ( \tilde { P } ) \leq \textstyle \frac { 1 } { 2 } ( 1 + p _ { 2 } ) <$ 1. Moreover, for any $\boldsymbol { \mu } \neq \mathbf { 0 }$ and $a _ { - 1 } \in [ 0 , 0 . 5 ] , \; \exists a _ { 1 }$ such that $\tau _ { \mathcal { D } } ( \tilde { P } ) < \tau _ { \mathcal { D } } ( P )$

Poisoning is effective only if the accuracy of the unlearnable model $\tilde { P }$ is less than that of the clean model $P$ on the clean dataset $\mathcal { D } ,$ that $\operatorname { i s } , \: \tau _ { \mathcal { D } } ( \tilde { P } ) \: < \: \tau _ { \mathcal { D } } ( P )$ To satisfy this condition, we need to carefully select ${ a _ { y } } ^ { \prime } \mathbf { \dot { s } } .$ In Theorem 2, we formally state this condition<sup>1</sup>. Theorem 2 shows that CUDA can effectively poison when there are two distinct modes in the clean Gaussian mixture data model. We validate our theoretical claim through empirical analysis as well (see Figure 3). Details for the analysis is given in supplementary material. We find that our upper bound for the clean test accuracy of CUDA classifiers are consistent with our empirical analysis. In our experiments, we also find that the unlearnability effect is stronger with a larger blur parameter. This effect is evident from Figure 3 where we find that it is likely to get a lower $\tau _ { \mathcal { D } } ( \tilde { P } )$ with higher $a _ { y }$ values. These results are consistent with our experimental results in Section 5 with CIFAR-10, CIFAR-100, and ImageNet-100 datasets.

## 5. Experiments

In this section, we first discuss our experimental setup. More details on the setup is deferred to supplementary material. We then show the robustness of CUDA generation with various datasets and architectures. We also run various experiments to analyze the effectiveness of CUDA under different training techniques (ERM, AT with varying budgets, randomized smoothing, transfer learning, and finetuning) and augmentation techniques (mixup [58], grayscaling, random blurring, cutout [9], cutmix [56], autoaugment [7], and orthogonal regularization [3]). Finally, we also design adaptive defenses to test the robustness of CUDA. One might think that CUDA filters can be obtained by adversarially training them with the data. We show that CUDA is robust to such adaptive defenses that we design.

<sup>1</sup>We note that the conditions µ>Aµ + b>µ + c + Tr(A) + k2Aµ + bk<sub>2</sub> < 0 or −µ>Aµ + b>µ − c − Tr(A) + k2Aµ − bk<sub>2</sub> < 0 can always be satisfied by picking a sufficiently large µ in the direction of an eigenvector corresponding to a negative or positive eigenvalue of A (note that A has negative and positive eigenvalues).

Table 2. Test accuracy (%) of ResNet-18 trained on various unlearnable datasets. We use $L _ { \infty }$ AT with budget $\rho _ { a } = 4 / 2 5 5$

<table><tr><td rowspan="2">Dataset</td><td rowspan="2">Training method</td><td rowspan="2">Clean</td><td colspan="5">Unlearnability method</td></tr><tr><td>EM</td><td>TAP</td><td>NTGA</td><td>REM</td><td>CUDA</td></tr><tr><td rowspan="2">CIFAR-10</td><td>ERM</td><td>94.66</td><td>13.20</td><td>22.51</td><td>16.27</td><td>27.09</td><td>18.48</td></tr><tr><td>AT</td><td>89.51</td><td>88.62</td><td>88.02</td><td>88.96</td><td>48.16</td><td>44.40</td></tr><tr><td rowspan="2">CIFAR-100</td><td>ERM</td><td>76.27</td><td>1.60</td><td>13.75</td><td>3.22</td><td>10.14</td><td>12.69</td></tr><tr><td>AT</td><td>64.50</td><td>63.43</td><td>62.39</td><td>62.44</td><td>27.10</td><td>34.34</td></tr><tr><td rowspan="2">ImageNet-100</td><td>ERM</td><td>80.66</td><td>1.26</td><td>9.10</td><td>8.42</td><td>13.74</td><td>8.96</td></tr><tr><td>AT</td><td>66.62</td><td>63.40</td><td>63.56</td><td>63.06</td><td>41.66</td><td>38.68</td></tr></table>

## 5.1. Experimental setup

**Datasets.** We use three image classification datasets – CIFAR-10, CIFAR-100, [21] and ImageNet-100 (a subset of ImageNet made of the first 100 classes) [40]. We use the data augmentation techniques such as random flipping, cropping, and resizing [44].

**Architectures.** We use ResNet-18 [14], VGG-16 [45], Wide ResNet-34-10 [57], and DenseNet-121 [16]. We train the networks with hyperparameters used in Fu et al. [11]. Previous works mainly employ ResNet-18 for most of their evaluations. Additional experiments on Tiny-ImageNet [26], DeIT [50], EfficientNetV2-S [48], and MobileNetV2 [41] are provided in the supplementary material.

**CUDA generation.** We use filters of size $k = 3$ and blur parameter $p _ { b } \; = \; 0 . 3$ for both CIFAR-10 and CIFAR-100 datsets. ImageNet-100 is a higher dimensional 224×224×3 image dataset when compared to the $3 2   \times   3 2   \times   3$ dimensional CIFAR datasets. Hence, we use larger filters of size $k = 9$ with $p _ { b } = 0 . 0 6$ for ImageNet-100. These hyperparameters are chosen such that the CUDA images are not perceptibly highly perturbed and give good unlearnability effect (see plot in Figure 1). In supplementary matrial, we show the results of training on CUDA with different hyperparameters for data generation.

**Baselines.** We compare CUDA generation technique with four state-of-the-art unlearnability methods – REM [11], EM [17], TAP [10], and NTGA [55]. We adopt the results reported in [11] since we use the same hyperparameters for training. For REM, we select hyperparameters $\rho _ { u }   =   8 / 2 5 5$ and $\rho _ { a }   =   4 / 2 5 5$ , the highest radii values in [11]. For comparing unlearnable methods, we look at the clean test accuracy. The lower the test accuracy, the better the unlearnability method is. As mentioned in the supplementary material, we use publicly released official codebases for reproducing the baselines using their default hyperparameters.

## 5.2. Effectiveness of CUDA

**Why does CUDA work?** In order to measure how much CUDA technique’s blurring affects the dataset’s quality, we compare class-wise blurring (CUDA technique) against universal blurring where a single convolutional filter is used for blurring all the images. We keep the filter generation parameters fixed $( p _ { b } = 0 . 3$ and $k = 3 )$ for both CUDA classwise blurring and universal blurring. ResNet-18 trained with clean CIFAR-10, universally blurred CIFAR-10, and CIFAR-10 CUDA achieve clean test accuracies of 94.66%, 90.47%, and 18.48%, respectively. This suggests that our controlled blurring does not obscure the semantics of the dataset. Hence, the significant drop in the clean test accuracy introduced by CUDA is most likely due to the usage of class-wise filters. This suggests that a model trained on CUDA learns the relation between the class-wise convolution filters and their corresponding labels. Therefore, during test time when this convolution effect is absent, the CUDA trained model fails to make correct classifications. Furthermore, the model trained on CUDA achieves an accuracy of 99.91% on the CIFAR-10 CUDA testset. This strongly supports our claim that the CUDA model learns to classify images based on the convolutional filters used to blur them. In addition, if we permute the class-wise filters for blurring the test set (i.e., blurring class 1 images with class 2 filters, class 2 images with class 3 filters, and so on), we get a very low accuracy of 2.53% on this test set. Further details are provided in the supplementary material. Finally, we note that real-world datasets might also contain blurred images due to various factors such as motion blurring, weather conditions, issues with the camera, etc. Hence, detecting if a blurred image is poisoned might not always be possible. However, one might argue that it is possible to detect if an entire dataset is blurred. Interestingly, later in this section we show that CUDA technique exhibits unlearnability effect even when only a fraction of the training dataset is poisoned.

Table 3. Test accuracy $( \% )$ of network architectures trained on various CIFAR-10 unlearnable datasets with $L _ { \infty }$ AT $( \rho _ { a } = 4 / 2 5 5 )$ .

<table><tr><td rowspan="2">Model</td><td rowspan="2">Clean</td><td colspan="5">Unlearnability method</td></tr><tr><td>EM</td><td>TAP</td><td>NTGA</td><td>REM</td><td>CUDA</td></tr><tr><td>ResNet-18</td><td>89.51</td><td>88.62</td><td>88.02</td><td>88.96</td><td>48.16</td><td>44.40</td></tr><tr><td>VGG-16</td><td>87.51</td><td>86.48</td><td>86.27</td><td>86.65</td><td>65.23</td><td>42.98</td></tr><tr><td>Wide ResNet-34-10</td><td>91.21</td><td>90.05</td><td>90.23</td><td>89.95</td><td>48.39</td><td>53.02</td></tr><tr><td>DenseNet-121</td><td>83.27</td><td>82.44</td><td>81.72</td><td>80.73</td><td>81.48</td><td>45.95</td></tr></table>

Table 4. Test accuracy (%) of ResNet-18 with CUDA under various training settings.

<table><tr><td>Dataset</td><td>Clean</td><td>Training method</td><td>CUDA (ours)</td></tr><tr><td rowspan="7">CIFAR-10</td><td rowspan="7">94.66</td><td>ERM</td><td>18.48</td></tr><tr><td>AT  $L_{\infty}$  ( $\rho_{a} = 4/255$ )</td><td>44.40</td></tr><tr><td>AT  $L_{\infty}$  ( $\rho_{a} = 8/255$ )</td><td>32.85</td></tr><tr><td>AT  $L_{\infty}$  ( $\rho_{a} = 16/255$ )</td><td>19.32</td></tr><tr><td>AT  $L_{2}$  ( $\rho_{a} = 0.25$ )</td><td>39.05</td></tr><tr><td>AT  $L_{2}$  ( $\rho_{a} = 0.50$ )</td><td>51.19</td></tr><tr><td>AT  $L_{2}$  ( $\rho_{a} = 0.75$ )</td><td>51.14</td></tr><tr><td rowspan="4">CIFAR-100</td><td rowspan="4">76.27</td><td>ERM</td><td>12.69</td></tr><tr><td>AT  $L_{\infty}$  ( $\rho_{a} = 4/255$ )</td><td>34.34</td></tr><tr><td>AT  $L_{\infty}$  ( $\rho_{a} = 8/255$ )</td><td>30.00</td></tr><tr><td>AT  $L_{2}$  ( $\rho_{a} = 0.75$ )</td><td>36.90</td></tr><tr><td rowspan="4">ImageNet-100</td><td rowspan="4">80.66</td><td>ERM</td><td>8.96</td></tr><tr><td>AT  $L_{\infty}$  ( $\rho_{a} = 4/255$ )</td><td>38.68</td></tr><tr><td>AT  $L_{\infty}$  ( $\rho_{a} = 8/255$ )</td><td>40.08</td></tr><tr><td>AT  $L_{2}$  ( $\rho_{a} = 0.75$ )</td><td>20.58</td></tr></table>

**Different datasets.** We first compare the effectiveness of CUDA with different datasets using ERM and $L _ { \infty }$ AT with $\rho _ { a }   =   4 / 2 5 5$ . We use ResNet-18 for the experiments. The results are shown in Table 2. These results show that EM, TAP, and NTGA are not robust to AT. However, both CUDA and REM are successful. Here, our method CUDA outperforms REM with CIFAR-10 and ImageNet-100 datasets. Smartly designed additive noise in AT helps in achieving better generalization than ERM on the unlearnable datasets. This experiment thus demonstrates that ERM and AT are not good choices for training with CUDA and REM dataset.

**Different models.** Next we compare the effectiveness of CUDA using various deep learning architectures with $L _ { \infty } \mathrm { ~ A T ~ } ( \rho _ { a }   =   4 / 2 5 5 )$ . We use CIFAR-10 for the experiments. The results are shown in Table 3. As we see in the table, CUDA is effective with all the five network architectures. However, REM is not seen to be transferrable with DenseNet-121.

**Robustness to different training settings.** In Section 3.2, we show that REM is sensitive to the training settings. REM generated using $L _ { \infty }$ radii budgets of $\rho _ { u } \; = \; 8 / 2 5 5$ and $\rho _ { a }   =   4 / 2 5 5$ for CIFAR-10 breaks with $L _ { \infty } \mathrm { ~ A T ~ } ( \rho _ { a } =$ 8/255) and $L _ { 2 } \mathrm { A T } ( \rho _ { a } = 0 . 7 5 )$ to get test accuracy of 78.71% and 79.65%, respectively. Hence, we run experiments to check the robustness of CUDA with various AT norm budgets. The results are shown in Table 4. As we see in the table, CUDA is robust to ERM, $L _ { \infty }$ , and $L _ { 2 }$ AT settings with varying training budgets. Impressively, the highest test accuracy achieved with training on CIFAR-10 CUDA, CIFAR-100 CUDA, and ImageNet-100 CUDA are as low as 51.19%, 36.90%, and 40.08%, respectively. We also find that using a pre-trained ResNet-18 with CIFAR-10 CUDA only achieves clean test accuracy of 42.42% and 48.22% with fine-tuning the full network and a newly trained final layer, respectively (details are deferred to the supplementary material).

**Different protection percentages.** In Section 3.1, we assume that the attacker has access to the full clean training data. However, in real life settings, this might not be always possible. Hence, we train ResNet-18 on a mix of CIFAR-10 CUDA and clean CIFAR-10 training datasets to evaluate the effectiveness of poisoning with varying data protection percentages. Protection percentage denotes the percentage of the training data that is poisoned.

We show the results in Table 5. In the table, the “Mixed” column denotes the clean test accuracy of a model trained using a mix of both clean and poisoned data. The “Clean” column denotes the clean test accuracy of a model trained only using the clean subset of the training data. In the last row of Table 5, we provide the results for CUDA trained using ERM with varying data protection percentages. The remainder of the rows provide the results for the $L _ { \infty }$ AT $( \rho _ { a }   =   4 / 2 5 5 )$ scenario. For example, CUDA trained with ERM using an 80% clean training data partition achieves a test accuracy of 93.75%. Adding the 20% CUDA data partition to the training dataset drops the test accuracy of the model to 93.28%. Results from the last row of Table 5 show that CUDA with varying data protection percentages is effective with the ERM setting. However, with the AT scenario, the unlearnability techniques are not as successful as with ERM with varying data protection percentages. Nevertheless, it is interesting to note that CUDA technique performs better than all the other unlearnability techniques with AT.

Table 5. Test accuracy (%) of ResNet-18 on CIFAR-10 with different data protecion percentages. The last row shows the results for CUDA with ERM setting. The rest of the rows show results for unlearnability methods trained in the $L _ { \infty }$ AT $( \rho _ { a } = 4 / 2 5 5 )$ setting.

<table><tr><td rowspan="3">Unlearnability method</td><td colspan="10">Data Protection Percentage</td></tr><tr><td rowspan="2">0%</td><td colspan="2">20%</td><td colspan="2">40%</td><td colspan="2">60%</td><td colspan="2">80%</td><td rowspan="2">100%</td></tr><tr><td>Mixed</td><td>Clean</td><td>Mixed</td><td>Clean</td><td>Mixed</td><td>Clean</td><td>Mixed</td><td>Clean</td></tr><tr><td>EM</td><td rowspan="5">89.51</td><td>89.60</td><td></td><td>89.40</td><td></td><td>89.49</td><td></td><td>89.10</td><td></td><td>88.62</td></tr><tr><td>TAP</td><td>89.01</td><td rowspan="2">88.17</td><td>88.66</td><td rowspan="2">86.76</td><td>88.40</td><td rowspan="2">85.07</td><td>88.04</td><td rowspan="2">79.41</td><td>88.02</td></tr><tr><td>NTGA</td><td>89.56</td><td>89.35</td><td>89.22</td><td>89.17</td><td>88.96</td></tr><tr><td>REM</td><td>89.60</td><td></td><td>89.34</td><td></td><td>89.61</td><td></td><td>88.09</td><td></td><td>48.16</td></tr><tr><td>CUDA (ours)</td><td>88.54</td><td></td><td>87.24</td><td></td><td>86.03</td><td></td><td>84.34</td><td></td><td>44.40</td></tr><tr><td>CUDA + ERM</td><td>94.66</td><td>93.28</td><td>93.75</td><td>91.34</td><td>92.56</td><td>89.91</td><td>89.77</td><td>85.61</td><td>84.30</td><td>18.48</td></tr></table>

**Robustness to smoothing and data augmentations.** Cohen et al. [6] proposes randomized smoothing which is a provable adversarial defense in $L _ { 2 }$ norm. Since CUDA does not use additive noise, CUDA is robust to randomized smoothing. A smoothed ResNet-18 (with a noise level of 0.5) with CIFAR-10 CUDA achieves only a clean test accuracy of 43.85%. In Section 3.2, we see that REM breaks with grayscaling. While ResNet-18 training using grayscaled REM achieves 70.76% test accuracy, grayscaled CUDA only achieves 20.12% test accuracy on the clean grayscaled CIFAR-10 test data. This shows that CUDA technique exhibits the desirable property of not relying upon the color space for its attack. CIFAR-10 CUDA training achieves only 25.53%, 25.80%, 26.93%, 34.09%, and 50.72% clean test accuracy with mixup, cutout, cutmix, autoaugment, and orthogonal regularization, respectively (see supplementary material for more details).

**Adaptive defenses for CUDA.** Here, we first investigate the effect of training CIFAR-10 CUDA with random blurring augmentations using ResNet-18. Each batch of the CUDA training data is convolved with random 3 × 3 filters of varying blur parameters $p _ { b } ^ { \prime }$ . With $p _ { b } ^ { \prime }$ values of 0.1 and

0.3, CUDA training achieves lower test accuracy 9.2% and 13.37%, respectively. This shows that the noise added by CUDA technique is robust to random convolution augmentations.

One may think that CUDA technique can be broken by learning the private filters from the data. We test this idea by training deconvolution filters to find if we can reverse the blurring effect in CUDA with adversarially trained filters. We use a novel Deconvolution-based Adversarial Training (DAT) technique that is similar to AT (check supplementary material for details). While the adversarial step in AT learns sample-wise error-maximizing additive noises, the adversarial step in DAT learns class-wise error-maximizing deconvolution filters. We train DAT with filters of varying sizes (3, 5, and 7) on CIFAR-10 CUDA using ResNet-18. The filter parameters are constrained within a finite range to make sure that the images do not get distorted with the adversarial step similar to projection in projected gradient descent. We find that CUDA is robust to DAT. DAT using filters of size 3, 5, and 7 with CUDA achieves only test accuracy of 39.05%, 46.21%, and 38.48%, respectively. DAT is not successful against CUDA since we can not invert convolutions without the knowledge of the private filters or clean images corresponding to CUDA.

**Limitations and future directions.** The unlearnability effect of CUDA can be defended if some fraction of the clean data and its corresponding CUDA images are leaked. The defender must also be able to detect if all samples in the dataset are poisoned. However, our work assumes a setup where the filters remain private. For example, a data publisher could simply publish their CUDA images and delete the clean images permanently to prevent this scenario. As discussed in this section, CUDA as well as other prior works do not perform well with different protection percentages with AT. Improving this can be an interesting research direction. We believe that extending CUDA technique to other domains such as tabular and text data is also an interesting future direction. It would also be interesting to see theoretical analysis of CUDA considering more complex setups.

## Acknowledgement

This project was supported in part by Meta grant 23010098, NSF CAREER AWARD 1942230, HR001119S0026 (GARD), ONR YIP award N00014-22-1-2271, Army Grant No. W911NF2120076, a capital one grant, NIST 60NANB20D134 and the NSF award CCF2212458.

## References

[1] Mauro Barni, Kassem Kallas, and Benedetta Tondi. A new backdoor attack in cnns by training set corruption without label poisoning. In 2019 IEEE International Conference on Image Processing (ICIP), pages 101–105. IEEE, 2019. 3
[2] Battista Biggio, Blaine Nelson, and Pavel Laskov. Poisoning attacks against support vector machines. arXiv preprint arXiv:1206.6389, 2012. 2
[3] Andrew Brock, Theodore Lim, James M Ritchie, and Nick Weston. Neural photo editing with introspective adversarial networks. arXiv preprint arXiv:1609.07093, 2016. 6, 19
[4] Nicholas Carlini and David Wagner. Towards evaluating the robustness of neural networks. In 2017 ieee symposium on security and privacy (sp), pages 39–57. IEEE, 2017. 2
[5] Xinyun Chen, Chang Liu, Bo Li, Kimberly Lu, and Dawn Song. Targeted backdoor attacks on deep learning systems using data poisoning. arXiv preprint arXiv:1712.05526, 2017. 3
[6] Jeremy Cohen, Elan Rosenfeld, and Zico Kolter. Certified adversarial robustness via randomized smoothing. In Kamalika Chaudhuri and Ruslan Salakhutdinov, editors, Proceedings of the 36th International Conference on Machine Learning, volume 97 of Proceedings of Machine Learning Research, pages 1310–1320. PMLR, 09–15 Jun 2019. 2, 8, 14
[7] Ekin D Cubuk, Barret Zoph, Dandelion Mane, Vijay Vasudevan, and Quoc V Le. Autoaugment: Learning augmentation policies from data. arXiv preprint arXiv:1805.09501, 2018. 6, 19
[8] Karan Desai and Justin Johnson. Virtex: Learning visual representations from textual annotations. In Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition, pages 11162–11173, 2021. 1
[9] Terrance DeVries and Graham W Taylor. Improved regularization of convolutional neural networks with cutout. arXiv preprint arXiv:1708.04552, 2017. 6, 19
[10] Liam Fowl, Micah Goldblum, Ping-yeh Chiang, Jonas Geiping, Wojtek Czaja, and Tom Goldstein. Adversarial examples make strong poisons. arXiv preprint arXiv:2106.10807, 2021. 1, 3, 6, 14
[11] Shaopeng Fu, Fengxiang He, Yang Liu, Li Shen, and Dacheng Tao. Robust unlearnable examples: Protecting data privacy against adversarial learning. In International Conference on Learning Representations, 2021. 1, 3, 4, 6, 14
[12] Ian J Goodfellow, Jonathon Shlens, and Christian Szegedy. Explaining and harnessing adversarial examples. arXiv preprint arXiv:1412.6572, 2014. 2
[13] Tianyu Gu, Brendan Dolan-Gavitt, and Siddharth Garg. Badnets: Identifying vulnerabilities in the machine learning model supply chain. arXiv preprint arXiv:1708.06733, 2017. 3
[14] Kaiming He, Xiangyu Zhang, Shaoqing Ren, and Jian Sun. Deep residual learning for image recognition. CoRR, abs/1512.03385, 2015. 6
[15] Kashmir Hill. The secretive company that might end privacy as we know it. In Ethics of Data and Analytics, pages 170–177. Auerbach Publications, 2020. 1
[16] Gao Huang, Zhuang Liu, and Kilian Q. Weinberger. Densely connected convolutional networks. CoRR, abs/1608.06993, 2016. 6
[17] Hanxun Huang, Xingjun Ma, Sarah Monazam Erfani, James Bailey, and Yisen Wang. Unlearnable examples: Making personal data unexploitable. arXiv preprint arXiv:2101.04898, 2021. 1, 3, 6, 14
[18] Arthur Jacot, Franck Gabriel, and Clement Hongler. Neu- ´ ral tangent kernel: Convergence and generalization in neural networks. Advances in neural information processing systems, 31, 2018. 3
[19] Adel Javanmard and Mahdi Soltanolkotabi. Precise statistical analysis of classification accuracies for adversarial training. ArXiv, abs/2010.11213, 2020. 2, 4
[20] Pang Wei Koh and Percy Liang. Understanding black-box predictions via influence functions. In International conference on machine learning, pages 1885–1894. PMLR, 2017. 2
[21] Alex Krizhevsky. Learning multiple layers of features from tiny images. Technical report, 2009. 6
[22] Aounon Kumar, Alexander Levine, Soheil Feizi, and Tom Goldstein. Certifying confidence via randomized smoothing. CoRR, abs/2009.08061, 2020. 2
[23] Alexey Kurakin, Ian J Goodfellow, and Samy Bengio. Adversarial examples in the physical world. In Artificial intelligence safety and security, pages 99–112. Chapman and Hall/CRC, 2018. 2
[24] Cassidy Laidlaw and Soheil Feizi. Functional adversarial attacks. In H. Wallach, H. Larochelle, A. Beygelzimer, F. d'Alche-Buc, E. Fox, and R. Garnett, editors, ´ Advances in Neural Information Processing Systems, volume 32. Curran Associates, Inc., 2019. 2
[25] Cassidy Laidlaw, Sahil Singla, and Soheil Feizi. Perceptual adversarial robustness: Defense against unseen threat models. CoRR, abs/2006.12655, 2020. 2
[26] Ya Le and Xuan Yang. Tiny imagenet visual recognition challenge. CS 231N, 7(7):3, 2015. 6, 15
[27] Alexander Levine and Soheil Feizi. Deep partition aggregation: Provable defense against general poisoning attacks. CoRR, abs/2006.14768, 2020. 2
[28] Alexander Levine and Soheil Feizi. Deep partition aggregation: Provable defense against general poisoning attacks. CoRR, abs/2006.14768, 2020. 2
[29] Yige Li, Xixiang Lyu, Nodens Koren, Lingjuan Lyu, Bo Li, and Xingjun Ma. Neural attention distillation: Erasing backdoor triggers from deep neural networks. arXiv preprint arXiv:2101.05930, 2021. 3
[30] Yiming Li, Tongqing Zhai, Baoyuan Wu, Yong Jiang, Zhifeng Li, and Shutao Xia. Rethinking the trigger of backdoor attack. arXiv preprint arXiv:2004.04692, 2020. 3
[31] Tsung-Yi Lin, Michael Maire, Serge Belongie, James Hays, Pietro Perona, Deva Ramanan, Piotr Dollar, and C Lawrence ´ Zitnick. Microsoft coco: Common objects in context. In European conference on computer vision, pages 740–755. Springer, 2014. 1
[32] Sijia Liu, Songtao Lu, Xiangyi Chen, Yao Feng, Kaidi Xu, Abdullah Al-Dujaili, Mingyi Hong, and Una-May O’Reilly. Min-max optimization without gradients: Convergence and applications to black-box evasion and poisoning attacks. In International Conference on Machine Learning, pages 6282–6293. PMLR, 2020. 2
[33] Yunfei Liu, Xingjun Ma, James Bailey, and Feng Lu. Reflection backdoor: A natural backdoor attack on deep neural networks. In European Conference on Computer Vision, pages 182–199. Springer, 2020. 3
[34] Aleksander Madry, Aleksandar Makelov, Ludwig Schmidt, Dimitris Tsipras, and Adrian Vladu. Towards deep learning models resistant to adversarial attacks. arXiv preprint arXiv:1706.06083, 2017. 1, 2, 14
[35] Yifei Min, Lin Chen, and Amin Karbasi. The curious case of adversarially robust models: More data can help, double descend, or hurt generalization. In Uncertainty in Artificial Intelligence, pages 129–139. PMLR, 2021. 4
[36] Tuan Anh Nguyen and Anh Tran. Input-aware dynamic backdoor attack. Advances in Neural Information Processing Systems, 33:3454–3464, 2020. 3
[37] Silvia Noschese, Lionello Pasquini, and Lothar Reichel. Tridiagonal toeplitz matrices: Properties and novel applications. 2006. 5
[38] Adam Paszke, Sam Gross, Francisco Massa, Adam Lerer, James Bradbury, Gregory Chanan, Trevor Killeen, Zeming Lin, Natalia Gimelshein, Luca Antiga, et al. Pytorch: An imperative style, high-performance deep learning library. Advances in neural information processing systems, 32, 2019. 18, 19
[39] Alec Radford, Jong Wook Kim, Chris Hallacy, Aditya Ramesh, Gabriel Goh, Sandhini Agarwal, Girish Sastry, Amanda Askell, Pamela Mishkin, Jack Clark, et al. Learning transferable visual models from natural language supervision. In International Conference on Machine Learning, pages 8748–8763. PMLR, 2021. 1
[40] Olga Russakovsky, Jia Deng, Hao Su, Jonathan Krause, Sanjeev Satheesh, Sean Ma, Zhiheng Huang, Andrej Karpathy, Aditya Khosla, Michael Bernstein, Alexander C. Berg, and Li Fei-Fei. ImageNet Large Scale Visual Recognition Challenge. International Journal of Computer Vision (IJCV), 115(3):211–252, 2015. 6
[41] M. Sandler, A. Howard, M. Zhu, A. Zhmoginov, and L.C. Chen. Mobilenetv2: Inverted residuals and linear bottlenecks. In CVPR, 2018. 6, 15, 16
[42] Mert Bulent Sariyildiz, Julien Perez, and Diane Larlus. Learning visual representations with caption annotations. In European Conference on Computer Vision, pages 153–170. Springer, 2020. 1
[43] Ali Shafahi, W Ronny Huang, Mahyar Najibi, Octavian Suciu, Christoph Studer, Tudor Dumitras, and Tom Goldstein. Poison frogs! targeted clean-label poisoning attacks on neural networks. Advances in neural information processing systems, 31, 2018. 2, 3
[44] Connor Shorten and Taghi Khoshgoftaar. A survey on image data augmentation for deep learning. Journal of Big Data, 6, 07 2019. 6
[45] Karen Simonyan and Andrew Zisserman. Very deep convolutional networks for large-scale image recognition. CoRR, abs/1409.1556, 2015. 6
[46] StubbornAtom. Distributions of quadratic form of a normal random variable. Cross Validated. URL:https://stats.stackexchange.com/q/478682 (version: 2020-07-23). 12
[47] Christian Szegedy, Wojciech Zaremba, Ilya Sutskever, Joan Bruna, Dumitru Erhan, Ian Goodfellow, and Rob Fergus. Intriguing properties of neural networks. arXiv preprint arXiv:1312.6199, 2013. 2
[48] M. Tan and Q. Le. Efficientnetv2: Smaller models and faster training. In ICML, 2021. 6, 15, 16
[49] Lue Tao, Lei Feng, Jinfeng Yi, Sheng-Jun Huang, and Songcan Chen. Better safe than sorry: Preventing delusive adversaries with adversarial training. Advances in Neural Information Processing Systems, 34, 2021. 1, 3
[50] H. Touvron, M. Cord, M. Douze, F. Massa, A. Sablayrolles, and H. Jegou. Training data-efficient image transformers & distillation through attention. In ICML, 2021. 6, 15, 16
[51] Laurens van der Maaten and Geoffrey Hinton. Visualizing data using t-sne. Journal of Machine Learning Research, 9(86):2579–2605, 2008. 2
[52] Wenxiao Wang, Alexander Levine, and Soheil Feizi. Improved certified defenses against data poisoning with (deterministic) finite aggregation. CoRR, abs/2202.02628, 2022. 2
[53] Yisen Wang, Xingjun Ma, James Bailey, Jinfeng Yi, Bowen Zhou, and Quanquan Gu. On the convergence and robustness of adversarial training. arXiv preprint arXiv:2112.08304, 2021. 2
[54] Da Yu, Huishuai Zhang, Wei Chen, Jian Yin, and Tie-Yan Liu. Availability attacks create shortcuts. In Proceedings of the 28th ACM SIGKDD Conference on Knowledge Discovery and Data Mining, KDD ’22, page 2367–2376, New York, NY, USA, 2022. Association for Computing Machinery. 2, 3
[55] Chia-Hung Yuan and Shan-Hung Wu. Neural tangent generalization attacks. In International Conference on Machine Learning, pages 12230–12240. PMLR, 2021. 1, 3, 6, 14
[56] Sangdoo Yun, Dongyoon Han, Seong Joon Oh, Sanghyuk Chun, Junsuk Choe, and Youngjoon Yoo. Cutmix: Regularization strategy to train strong classifiers with localizable features. In Proceedings of the IEEE/CVF international conference on computer vision, pages 6023–6032, 2019. 6, 19
[57] Sergey Zagoruyko and Nikos Komodakis. Wide residual networks. In Edwin R. Hancock Richard C. Wilson and William A. P. Smith, editors, Proceedings of the British Machine Vision Conference (BMVC), pages 87.1–87.12. BMVA Press, September 2016. 6
[58] Hongyi Zhang, Moustapha Cisse, Yann N. Dauphin, and ´ David Lopez-Paz. mixup: Beyond empirical risk minimization. CoRR, abs/1710.09412, 2017. 6, 14, 19
[59] Hongyang Zhang, Yaodong Yu, Jiantao Jiao, Eric Xing, Laurent El Ghaoui, and Michael Jordan. Theoretically principled trade-off between robustness and accuracy. In International conference on machine learning, pages 7472–7482. PMLR, 2019. 2

## A. Appendix

## A.1. Proof for Lemma 1

At the optimal decision boundary the probabilities of any point $\pmb { x } \in \mathbb { R } ^ { d }$ belonging to class $y = - 1$ and $y = 1$ modeled by $\mathcal { D }$ are the same. Here, $\mathbf { \mu } \; = \; \mathbf { \mu _ { 1 } } \; = \; - \mathbf { \mu _ { - 1 } }$ and $I = \Sigma _ { - 1 } = \Sigma _ { 1 }$

$$
\begin{array}{l} \Longrightarrow \frac {\exp \left[ - \frac {1}{2} \left(\boldsymbol {x} - \boldsymbol {\mu} _ {- 1}\right) ^ {\top} \Sigma_ {- 1} ^ {- 1} \left(\boldsymbol {x} - \boldsymbol {\mu} _ {- 1}\right) \right]}{\sqrt {(2 \pi) ^ {d} \left| \Sigma_ {- 1} \right|}} = \\ \frac {\exp \left[ - \frac {1}{2} \left(\boldsymbol {x} - \boldsymbol {\mu_ {1}}\right) ^ {\top} \Sigma_ {1} ^ {- 1} \left(\boldsymbol {x} - \boldsymbol {\mu_ {1}}\right) \right]}{\sqrt {(2 \pi) ^ {d} \left| \Sigma_ {1} \right|}} \\ \Longrightarrow - \frac {1}{2} \log | I | - \frac {1}{2} (\boldsymbol {x} ^ {\top} \boldsymbol {x} - 2 \boldsymbol {x} ^ {\top} \boldsymbol {\mu} _ {- 1} + \boldsymbol {\mu} _ {- 1} ^ {\top} \boldsymbol {\mu} _ {- 1}) = \\ - \frac {1}{2} \log | I | - \frac {1}{2} (\boldsymbol {x} ^ {\top} \boldsymbol {x} - 2 \boldsymbol {x} ^ {\top} \boldsymbol {\mu_ {1}} + \boldsymbol {\mu_ {1} ^ {\top}} \boldsymbol {\mu_ {1}}) \\ \Longrightarrow \boldsymbol {x} ^ {\top} (\boldsymbol {\mu} _ {- 1} - \boldsymbol {\mu} _ {1}) - \frac {1}{2} (\boldsymbol {\mu} _ {- 1} ^ {\top} \boldsymbol {\mu} _ {- 1} - \boldsymbol {\mu} _ {1} ^ {\top} \boldsymbol {\mu} _ {1}) = 0 \\ \Longrightarrow - \boldsymbol {x} ^ {\top} \boldsymbol {\mu} = 0 \\ \Longrightarrow P (\boldsymbol {x}) \equiv \boldsymbol {x} ^ {\top} \boldsymbol {\mu} = 0. \\ \end{array}
$$

Now, the accuracy of the clean model P is to be computed. Note that if $P ( { \pmb x } ) \; < \; 0$ the Bayes optimal classification is class -1, else the classification is class 1. Let $z \sim \mathcal { N } ( \mathbf { 0 } , I )$ , and $Z \sim \mathcal { N } ( 0 , 1 )$ , and $s g n ( . )$ be the signum function.

$$
\begin{array}{l} \tau_ {\mathcal {D}} (P) = \mathbb {E} _ {(\boldsymbol {x}, y) \sim \mathcal {D}} \left[ \mathbb {1} (y = s g n (P (\boldsymbol {x}))) \right] = \mathbb {P} [ y \boldsymbol {x} ^ {\top} \boldsymbol {\mu} > 0 ] \\ = \mathbb {P} [ y (y \boldsymbol {\mu} + \boldsymbol {z}) ^ {\top} \boldsymbol {\mu} > 0 ] \\ = \mathbb {P} [ (\boldsymbol {\mu} + \boldsymbol {z}) ^ {\top} \boldsymbol {\mu} > 0 ] \\ = \mathbb {P} [ \| \boldsymbol {\mu} \| _ {2} ^ {2} + \| \boldsymbol {\mu} \| _ {2} Z > 0 ] = \phi (\| \boldsymbol {\mu} \| _ {2}). \\ \end{array}
$$

![](images/image_09.jpg)

## A.2. Proof for Lemma 2

Let $\mathcal { D } _ { 1 } = \mathcal { N } ( \pmb { \mu } , I )$ . For every data point $( { \pmb x } , y ) \sim { \mathcal D } _ { 1 } ,$ , let the perturbed data $( A _ { 1 } { \pmb x } , y )$ be modelled by a distribution $\tilde { D } _ { 1 }$ . We prove that $\tilde { D } _ { 1 } = \mathcal { N } ( A _ { 1 } \pmb { \mu } , A _ { 1 } ^ { \top } A _ { 1 } )$ .

$$
\mathbb {E} _ {(\boldsymbol {x}, y) \sim \mathcal {D} _ {1}} A _ {1} \boldsymbol {x} = A _ {1} \mathbb {E} _ {(\boldsymbol {x}, y) \sim \mathcal {D} _ {1}} \boldsymbol {x} = A _ {1} \boldsymbol {\mu}.
$$

$$
\begin{array}{l} \mathbb {E} _ {(\boldsymbol {x}, y) \sim \mathcal {D} _ {1}} \left(A _ {1} \boldsymbol {x} - A _ {1} \boldsymbol {\mu}\right) \left(A _ {1} \boldsymbol {x} - A _ {1} \boldsymbol {\mu}\right) ^ {\top} \\ = \mathbb {E} _ {(\boldsymbol {x}, y) \sim \mathcal {D} _ {1}} A _ {1} (\boldsymbol {x} - \boldsymbol {\mu}) \left[ A _ {1} (\boldsymbol {x} - \boldsymbol {\mu}) \right] ^ {\top} \\ = \mathbb {E} _ {(\boldsymbol {x}, y) \sim \mathcal {D} _ {1}} A _ {1} (\boldsymbol {x} - \boldsymbol {\mu}) (\boldsymbol {x} - \boldsymbol {\mu}) ^ {\top} A _ {1} ^ {\top} \\ = A _ {1} \mathbb {E} _ {(\boldsymbol {x}, y) \sim \mathcal {D} _ {1}} (\boldsymbol {x} - \boldsymbol {\mu}) (\boldsymbol {x} - \boldsymbol {\mu}) ^ {\top} A _ {1} ^ {\top} \\ = A _ {1} I A _ {1} ^ {\top} = A _ {1} A _ {1} ^ {\top}. \\ \end{array}
$$

Tri-diagonal Toeplitz matrices $A _ { y } = T ( d ; a _ { y } , 1 , a _ { y } )$ are symmetric. Hence, $\tilde { \mathcal { D } } = \mathcal { N } ( y A _ { y } \pmb { \mu } , A _ { y } ^ { 2 } )$ 

## A.3. Remarks on Lemma 3

A tri-diagonal Toeplitz matrix $T ( d ; a _ { 1 } , a _ { 2 } , a _ { 3 } )$ is represented as

$$
\left[ \begin{array}{c c c c c c} a _ {2} & a _ {3} & 0 & 0 & \dots & 0 \\ a _ {1} & a _ {2} & a _ {3} & 0 & \dots & 0 \\ 0 & a _ {1} & a _ {2} & a _ {3} & \dots & 0 \\ \vdots & \vdots & \vdots & \vdots & \ddots & \vdots \\ 0 & \dots & \dots & 0 & a _ {1} & a _ {2} \end{array} \right] \in \mathbb {R} ^ {d \times d}
$$

The class of matrices $A _ { y } \; = \; T ( d ; a _ { y } , 1 , a _ { y } )$ are symmetric and can be diagonalized as $\begin{array} { r l r l } { Q D Q ^ { \top } . } & { { } ~ Q } & { = } & { { } } \end{array}$ $\textstyle \big ( \big ( \frac { 2 } { d + 1 } \big ) ^ { 1 / 2 } \operatorname { s i n } \big ( \frac { i j \pi } { d + 1 } \big ) \big ) _ { i , j }$ is symmetric and it is the common eigenvector matrix to all $A _ { y }$ matrices. As shown in Lemma 3, Q and $D$ can be represented using trigonometric functions. Also, we have $A ( n ) : = A _ { 1 } ^ { n }   \pm   A _ { - 1 } ^ { n } =$ $Q ( D _ { 1 } ^ { n }   \pm   D _ { - 1 } ^ { n } ) Q$ where $A _ { 1 } = Q D _ { 1 } Q$ and $A _ { - 1 } = Q D _ { - 1 } Q$ Further, $\mathrm { T r } ( A ( n ) )   =   \mathrm { T r } ( Q ( D _ { 1 } ^ { n } \pm D _ { - 1 } ^ { n } ) Q )   =   \mathrm { T r } ( ( D _ { 1 } ^ { n } \pm$ $D _ { - 1 } ^ { n } ) Q ^ { 2 } ) = \mathrm { T r } ( D _ { 1 } ^ { n } \pm D _ { - 1 } ^ { n } )$

## A.4. Proof for Lemma 4

At the optimal decision boundary the probabilities of any point $\pmb { x } \in \mathbb { R } ^ { d }$ belonging to class $y = - 1$ and $y = 1$ modeled by $\tilde { \mathcal { D } }$ are the same. Here, $\mathbf { \mu } = \mathbf { \mu _ { 1 } } = - \mathbf { \mu _ { - 1 } }$ and $A _ { y } { } ^ { \prime } \mathrm { s }$ are symmetric.

$$
\begin{array}{l} \frac {\exp \left[ - \frac {1}{2} \left(\boldsymbol {x} - A _ {- 1} \boldsymbol {\mu} _ {- 1}\right) ^ {\top} \left(A _ {- 1} I A _ {- 1} ^ {\top}\right) ^ {- 1} \left(\boldsymbol {x} - A _ {- 1} \boldsymbol {\mu} _ {- 1}\right) \right]}{\sqrt {(2 \pi) ^ {d} \left| A _ {- 1} I A _ {- 1} ^ {\top} \right|}} \\ = \frac {\exp [ - \frac {1}{2} (\boldsymbol {x} - A _ {1} \boldsymbol {\mu_ {1}}) ^ {\top} (A _ {1} I A _ {1} ^ {\top}) ^ {- 1} (\boldsymbol {x} - A _ {1} \boldsymbol {\mu_ {1}}) ]}{\sqrt {(2 \pi) ^ {d} | A _ {1} I A _ {1} ^ {\top} |}} \\ \Longrightarrow - \frac {1}{2} \ln \frac {\left| A _ {- 1} ^ {2} \right|}{\left| A _ {1} ^ {2} \right|} - \frac {1}{2} \left[ \boldsymbol {x} ^ {\top} \left(A _ {- 1} ^ {- 2} - A _ {1} ^ {- 2}\right) \boldsymbol {x} \right. \\ - 2 (\boldsymbol {\mu} _ {- 1} ^ {\top} A _ {- 1} ^ {- 1} - \boldsymbol {\mu} _ {1} ^ {\top} A _ {1} ^ {- 1}) \boldsymbol {x} \\ \left. + \left(\boldsymbol {\mu} _ {- 1} ^ {\top} \boldsymbol {\mu} _ {- 1} - \boldsymbol {\mu} _ {1} ^ {\top} \boldsymbol {\mu} _ {1}\right) \right] = 0 \\ \Longrightarrow \tilde {P} (\boldsymbol {x}) \equiv \boldsymbol {x} ^ {\top} (A _ {- 1} ^ {- 2} - A _ {1} ^ {- 2}) \boldsymbol {x} \\ - 2 (\boldsymbol {\mu} _ {- 1} ^ {\top} A _ {- 1} ^ {- 1} - \boldsymbol {\mu} _ {1} ^ {\top} A _ {1} ^ {- 1}) \boldsymbol {x} + (\| \boldsymbol {\mu} _ {- 1} \| _ {2} ^ {2} - \| \boldsymbol {\mu} _ {1} \| _ {2} ^ {2}) \\ + \sum_ {i = 1} ^ {d} \ln \left(\frac {1 + 2 a _ {- 1} \cos (\frac {i \pi}{d + 1})}{1 + 2 a _ {1} \cos (\frac {i \pi}{d + 1})}\right) ^ {2} = 0 \\ \Longrightarrow \tilde {P} (\boldsymbol {x}) \equiv \boldsymbol {x} ^ {\top} (A _ {- 1} ^ {- 2} - A _ {1} ^ {- 2}) \boldsymbol {x} \\ + 2 [ (A _ {- 1} ^ {- 1} + A _ {1} ^ {- 1}) \boldsymbol {\mu} ] ^ {\top} \boldsymbol {x} \\ + \sum_ {i = 1} ^ {d} \ln \left(\frac {1 + 2 a _ {- 1} \cos (\frac {i \pi}{d + 1})}{1 + 2 a _ {1} \cos (\frac {i \pi}{d + 1})}\right) ^ {2} = 0 \\ \Longrightarrow \tilde {P} (\boldsymbol {x}) \equiv \boldsymbol {x} ^ {\top} A \boldsymbol {x} + \boldsymbol {b} ^ {\top} \boldsymbol {x} + c = 0. \\ \end{array}
$$

![](images/image_10.jpg)

Note that here if $\tilde { P } ( { \pmb x } )   <   0$ , the Bayes optimal classification is class -1, else the classification is class 1. Here, for shorthand notations we denote $A = ( A _ { - 1 } ^ { - 2 } - A _ { 1 } ^ { - 2 } ) , \; { \pmb b } =$

$$
2 (A _ {- 1} ^ {- 1} + A _ {1} ^ {- 1}) \pmb {\mu}, c = \sum_ {i = 1} ^ {d} \ln \left(\frac {1 + 2 a _ {- 1} \cos (\frac {i \pi}{d + 1})}{1 + 2 a _ {1} \cos (\frac {i \pi}{d + 1})}\right) ^ {2}.
$$

## A.5. Proof for Lemma 5

Let $Z = { \boldsymbol { z } } ^ { \top } A { \boldsymbol { z } }   +   { \boldsymbol { z } } ^ { \top } { \boldsymbol { b } }   +   c$ and $\boldsymbol { z } \sim \mathcal { N } ( \mathbf { 0 } , I ) \subset \mathbb { R } ^ { d }$ where $A = Q \Lambda Q ^ { \top }$ . Also,

$$
\begin{array}{l} Z = \boldsymbol {z} ^ {\top} A \boldsymbol {z} + \boldsymbol {z} ^ {\top} \boldsymbol {b} + c \\ = \left(\boldsymbol {z} + \frac {1}{2} A ^ {- 1} \boldsymbol {b}\right) ^ {\top} A \left(\boldsymbol {z} + \frac {1}{2} A ^ {- 1} \boldsymbol {b}\right) + c - \frac {1}{4} \boldsymbol {b} ^ {\top} A ^ {- 1} \boldsymbol {b}. \\ \end{array}
$$

For any $t \geq 0$ and $\pmb { x } \sim \mathcal { N } ( \mathbf { 0 } , I )$ , we write the moment generating function for a quadratic random variable $Y =$ $x ^ { \top }$ Ax as <sup>2</sup>

$$
\begin{array}{l} \mathbb {E} [ \exp (t Y) ] = \frac {1}{(2 \pi) ^ {d / 2}} \int_ {\mathbb {R} ^ {d}} \exp \{t \boldsymbol {x} ^ {\top} A \boldsymbol {x} \} \\ \exp \left\{- \frac {1}{2} (\pmb {x} - \pmb {\mu}) ^ {\top} (\pmb {x} - \pmb {\mu}) \right\} d \pmb {x} \\ = \frac {\exp \{- \boldsymbol {\mu} ^ {\top} \boldsymbol {\mu} / 2 \}}{(2 \pi) ^ {d / 2}} \int_ {\mathbb {R} ^ {d}} \exp \Big \{- \frac {1}{2} \boldsymbol {x} ^ {\top} (I - 2 t A) \boldsymbol {x} + \boldsymbol {\mu} ^ {\top} \boldsymbol {x} \Big \} d \boldsymbol {x} \\ = \frac {\exp \{- \boldsymbol {\mu} ^ {\top} \boldsymbol {\mu} / 2 \}}{(2 \pi) ^ {d / 2}} \frac {(2 \pi) ^ {d / 2} \exp \left\{\frac {1}{2} \boldsymbol {\mu} ^ {\top} (I - 2 t A) ^ {- 1} \boldsymbol {\mu} \right\}}{| I - 2 t A | ^ {1 / 2}} \\ = \frac {\exp \left\{- \frac {1}{2} \boldsymbol {\mu} ^ {\top} [ I - (I - 2 t A) ^ {- 1} ] \boldsymbol {\mu} \right\}}{| I - 2 t A | ^ {1 / 2}}. \\ \Longrightarrow \mathbb {E} [ \exp (t Z) ] = \\ \frac {\exp \{- \frac {\boldsymbol {b} ^ {\top}}{8} A ^ {- 1} [ I - (I - 2 t A) ^ {- 1} ] A ^ {- 1} \boldsymbol {b} + t [ c - \frac {\boldsymbol {b} ^ {\top}}{4} A ^ {- 1} \boldsymbol {b} ] \}}{| I - 2 t A | ^ {\frac {1}{2}}}. \\ \end{array}
$$

Using the Chernoff bound and $\begin{array} { r l r l } { \mathbb { E } } & { { } z ^ { \top } A z } & { { } = } \end{array}$ $\operatorname { T r } ( A \mathbb { E } [ z z ^ { \top } ] ) = \operatorname { T r } ( A )$ , for some γ,

$$
\begin{array}{l} \mathbb {P} \{Z \geq \mathbb {E} [ Z ] + \gamma \} \leq \frac {\mathbb {E} [ \exp (t Z) ]}{\exp \{t [ \gamma + \mathbb {E} (Z) ] \}} = \\ \frac {\exp \{- \frac {\boldsymbol {b} ^ {\top}}{8} A ^ {- 1} [ I - (I - 2 t A) ^ {- 1} ] A ^ {- 1} \boldsymbol {b} + t [ c - \frac {\boldsymbol {b} ^ {\top}}{4} A ^ {- 1} \boldsymbol {b} ] \}}{\exp \{t ([ \gamma + \operatorname{Tr} (A) + \| b \| _ {2} + c ] \} | I - 2 t A | ^ {\frac {1}{2}}}. \\ \end{array}
$$

Let us take $\begin{array} { r l r } { { \pmb u } } & { { } = } & { Q ^ { \top } { \pmb b } . } \end{array}$ Also, $- \Lambda ^ { - 1 } [ I \; - \; ( I \; - \;$ $\begin{array} { r } { 2 t \Lambda ) ^ { - 1 } ] \Lambda ^ { - 1 } = 2 t \Lambda ^ { - 1 } ( \overset { \cdot } { I } - 2 t \Lambda ) ^ { - 1 } } \end{array}$ since Λ is a diagonal matrix. Using Woodbury matrix identity, we get $( I - 2 t \Lambda ) ^ { - 1 } = I - \tilde { ( I - \textstyle \frac { 1 } { 2 t } \Lambda ^ { - 1 } ) ^ { - 1 } }$ . This gives us

<small><span class="docvortex-page-footnote" data-block-type="page_footnote" style="color:#6b7280"><sup>2</sup>[46]</span></small>

$$
\begin{array}{l} \mathbb {P} \{Z \geq \mathbb {E} [ Z ] + \gamma \} \leq \\ \exp \{- \frac {1}{8} \boldsymbol {b} ^ {\top} A ^ {- 1} [ I - (I - 2 t A) ^ {- 1} ] A ^ {- 1} \boldsymbol {b} \\ \left. + t \left[ c - \frac {1}{4} \boldsymbol {b} ^ {\top} A ^ {- 1} \boldsymbol {b} \right] \right\} \exp \left\{- t \left[ \gamma + \operatorname{Tr} (A) + \| b \| _ {2} + c \right] \right\} \\ | I - 2 t A | ^ {\frac {- 1}{2}} \\ = \exp \left\{- \frac {1}{8} \boldsymbol {u} ^ {\top} \Lambda^ {- 1} \left[ I - (I - 2 t \Lambda) ^ {- 1} \right] \Lambda^ {- 1} \boldsymbol {u} \right. \\ \left. + t \left[ c - \frac {1}{4} \boldsymbol {u} ^ {\top} \Lambda^ {- 1} \boldsymbol {u} \right] \right\} \exp \left\{- t \left[ \gamma + \operatorname{Tr} (\Lambda) + \| b \| _ {2} + c \right] \right\} \\ | I - 2 t \Lambda | ^ {\frac {- 1}{2}} \\ = \exp \left\{\frac {t}{4} \boldsymbol {u} ^ {\top} \Lambda^ {- 1} (I - 2 t \Lambda) ^ {- 1} \boldsymbol {u} \right. \\ \left. + t \left[ c - \frac {1}{4} \boldsymbol {u} ^ {\top} \Lambda^ {- 1} \boldsymbol {u} \right] \right\} \exp \{- t [ \gamma + \operatorname{Tr} (\Lambda) + \| b \| _ {2} + c ] \} \\ | I - 2 t \Lambda | ^ {\frac {- 1}{2}} \\ = \exp \left\{\frac {t}{4} \boldsymbol {u} ^ {\top} \Lambda^ {- 1} \left[ I - \left(I - \frac {1}{2 t} \Lambda^ {- 1}\right) ^ {- 1} \right] \boldsymbol {u} \right. \\ \left. + t \left[ c - \frac {1}{4} \boldsymbol {u} ^ {\top} \Lambda^ {- 1} \boldsymbol {u} \right] \right\} \exp \{- t [ \gamma + \operatorname{Tr} (\Lambda) + \| b \| _ {2} + c ] \} \\ | I - 2 t \Lambda | ^ {\frac {- 1}{2}} \\ = \frac {\exp \left\{\frac {- t}{4} \boldsymbol {u} ^ {\top} \Lambda^ {- 1} \left(I - \frac {1}{2 t} \Lambda^ {- 1}\right) ^ {- 1} \boldsymbol {u} + t c \right\}}{\exp \left\{t \left[ \gamma + \operatorname{Tr} (\Lambda) + \| b \| _ {2} + c \right] \right\} \left| I - 2 t \Lambda \right| ^ {\frac {1}{2}}} \leq \\ \exp \{\frac {- t}{4 \| \boldsymbol {b} \| _ {2} ^ {2}} \lambda_ {\min} (\Lambda^ {- 1} (I - \frac {1}{2 t} \Lambda^ {- 1}) ^ {- 1}) \\ \left. - t \left(\gamma + \operatorname{Tr} (\Lambda) + \| b \| _ {2}\right) \right\} \left| I - 2 t \Lambda \right| ^ {\frac {- 1}{2}} \\ = \exp \{\frac {- t}{4 \| \boldsymbol {b} \| _ {2} ^ {2}} \frac {1}{\| \Lambda \| - 1 / (2 t)} - t (\gamma + \mathrm{Tr} (\Lambda) + \| b \| _ {2}) \} \\ \end{array}
$$

$$
| I - 2 t \Lambda | ^ {\frac {- 1}{2}} \leq \frac {\exp \left\{\frac {- t}{4 \| \boldsymbol {b} \| _ {2} ^ {2} \| \Lambda \|} - t (\gamma + \operatorname{Tr} (\Lambda) + \| b \| _ {2}) \right\}}{| I - 2 t \Lambda | ^ {\frac {1}{2}}}.
$$

![](images/image_11.jpg)

## A.6. Proof for Theorem 2

Note that if $\tilde { P } ( { \pmb x } ) < 0$ , the classifier predicts a label for class -1, else the predicted label would be 1. Here, $x =$ $y \mu + z$ where $z \sim \mathcal { N } ( \mathbf { 0 } , I )$ and $y \in \{ \pm 1 \}$ since $( { \pmb x } , y ) \sim$ $\mathcal { D } .$

$$
\begin{array}{l} \tau_ {\mathcal {D}} (\tilde {P}) = \mathbb {E} \left\{\mathbb {1} \left(y \left(\boldsymbol {x} ^ {\top} A \boldsymbol {x} + \boldsymbol {b} ^ {\top} \boldsymbol {x} + c\right) > 0\right) \right\} \\ = \mathbb {P} \left\{y \left(\boldsymbol {\mu} ^ {\top} A \boldsymbol {\mu} + \boldsymbol {z} ^ {\top} A \boldsymbol {z} + 2 y \boldsymbol {\mu} ^ {\top} A \boldsymbol {z} + \right. \right. \\ \left. y \boldsymbol {b} ^ {\top} \boldsymbol {\mu} + \boldsymbol {b} ^ {\top} \boldsymbol {z} + c\right) > 0 \rbrace \\ = \mathbb {P} (y = 1) \mathbb {P} \{y (\boldsymbol {\mu} ^ {\top} A \boldsymbol {\mu} + \boldsymbol {z} ^ {\top} A \boldsymbol {z} \\ \left. + 2 y \boldsymbol {\mu} ^ {\top} A \boldsymbol {z} + y \boldsymbol {b} ^ {\top} \boldsymbol {\mu} + \boldsymbol {b} ^ {\top} \boldsymbol {z} + c\right) > 0 \mid y = 1 \} + \\ \mathbb {P} (y = - 1) \mathbb {P} \{y (\boldsymbol {\mu} ^ {\top} A \boldsymbol {\mu} + \boldsymbol {z} ^ {\top} A \boldsymbol {z} \\ \left. + 2 y \boldsymbol {\mu} ^ {\top} A \boldsymbol {z} + y \boldsymbol {b} ^ {\top} \boldsymbol {\mu} + \boldsymbol {b} ^ {\top} \boldsymbol {z} + c\right) > 0 \mid y = - 1 \} = \\ \frac {1}{2} \mathbb {P} \left\{\boldsymbol {z} ^ {\top} A \boldsymbol {z} + (\boldsymbol {b} + 2 A \boldsymbol {\mu}) ^ {\top} \boldsymbol {z} + \boldsymbol {\mu} ^ {\top} A \boldsymbol {\mu} + \boldsymbol {b} ^ {\top} \boldsymbol {\mu} + c > 0 \right\} + \\ \frac {1}{2} \mathbb {P} \{- \boldsymbol {z} ^ {\top} A \boldsymbol {z} - (\boldsymbol {b} - 2 A \boldsymbol {\mu}) ^ {\top} \boldsymbol {z} - \boldsymbol {\mu} ^ {\top} A \boldsymbol {\mu} + \boldsymbol {b} ^ {\top} \boldsymbol {\mu} - c > 0 \} \\ \mathrel {\mathop :} = p _ {1} + p _ {2} \\ \end{array}
$$

We can see that

$$
\begin{array}{l} - \gamma_ {1} := \\ \mathbb {E} \left\{\boldsymbol {z} ^ {\top} A \boldsymbol {z} + (\boldsymbol {b} + 2 A \boldsymbol {\mu}) ^ {\top} \boldsymbol {z} + \boldsymbol {\mu} ^ {\top} A \boldsymbol {\mu} + \boldsymbol {b} ^ {\top} \boldsymbol {\mu} + c \right\} \\ = \operatorname{Tr} (\Lambda) + \| \boldsymbol {b} + 2 A \boldsymbol {\mu} \| _ {2} + \boldsymbol {\mu} ^ {\top} A \boldsymbol {\mu} + \boldsymbol {b} ^ {\top} \boldsymbol {\mu} + c, \text {and} \\ - \gamma_ {2} := \\ \mathbb {E} \left\{- \boldsymbol {z} ^ {\top} A \boldsymbol {z} - (\boldsymbol {b} - 2 A \boldsymbol {\mu}) ^ {\top} \boldsymbol {z} - \boldsymbol {\mu} ^ {\top} A \boldsymbol {\mu} + \boldsymbol {b} ^ {\top} \boldsymbol {\mu} - c \right\} \\ = - \operatorname{Tr} (\Lambda) + \| \boldsymbol {b} - 2 A \boldsymbol {\mu} \| _ {2} - \boldsymbol {\mu} ^ {\top} A \boldsymbol {\mu} + \boldsymbol {b} ^ {\top} \boldsymbol {\mu} - c. \\ \end{array}
$$

Using Lemma 5, with $\gamma = \gamma _ { 1 } , t = t _ { 1 }$ for computing $p _ { 1 }$ and $\gamma = \gamma _ { 2 } , t = t _ { 2 }$ for computing $p _ { 2 }$ where $t _ { 1 } , t _ { 2 }$ are some non-negative constants, we get

$$
\begin{array}{l} p _ {1} = \frac {1}{2 \left| I - 2 t _ {1} \Lambda \right| ^ {1 / 2}} \exp \left[ t _ {1} \left(\boldsymbol {\mu} ^ {\top} A \boldsymbol {\mu} + \boldsymbol {b} ^ {\top} \boldsymbol {\mu} + c \right. \right. \\ \left. \left. - \frac {1}{4 \| 2 A \boldsymbol {\mu} + \boldsymbol {b} \| _ {2} \| \Lambda \|}\right) \right], \text {and} \\ p _ {2} = \frac {1}{2 \left| I - 2 t _ {2} \Lambda \right| ^ {1 / 2}} \exp \left[ t _ {2} \left(- \boldsymbol {\mu} ^ {\top} A \boldsymbol {\mu} + \boldsymbol {b} ^ {\top} \boldsymbol {\mu} - c \right. \right. \\ \left. \left. - \frac {1}{4 \| 2 A \boldsymbol {\mu} - \boldsymbol {b} \| _ {2} \| \Lambda \|}\right) \right]. \\ \end{array}
$$

This gives us the upper bound for $\tau _ { \mathcal { D } } ( \tilde { P } )$ . However, to make sure that this upper bound is smaller than 1, we need to assert more conditions. $p _ { 1 }$ and $p _ { 2 }$ become smaller as $\gamma _ { 1 }$ and $\gamma _ { 2 }$ are larger positive numbers. However, $\gamma _ { 1 } +$ $\gamma _ { 2 } = - ( \| 2 A \pmb { \mu } + \pmb { b } \| _ { 2 } + \| 2 A \pmb { \mu } - \pmb { b } \| _ { 2 } + 4 \pmb { \mu } ^ { \top } Q ^ { \top } ( D _ { 1 } ^ { - 1 } +$ $D _ { - 1 } ^ { - 1 } ) Q \pmb { \mu } ) \leq 0$ since $( D _ { 1 } ^ { - 1 } + D _ { - 1 } ^ { - 1 } ) \stackrel { \cdot \cdot } { \succcurlyeq } 0$ . Hence, we look at separately at cases when either $\gamma _ { 1 } > 0 \; \mathbf { o r } \; \gamma _ { 2 } > 0 .$

$\mathrm { I f }   \gamma _ { 1 } > 0 ,$ then $\tau _ { \mathcal { D } } ( \tilde { P } ) = \textstyle \frac { 1 } { 2 } ( p _ { 1 }   +   1 ) < 1$ . Else, if $\gamma _ { 2 } > 0 ,$ then $\tau _ { \mathcal { D } } ( \tilde { P } )   =   \textstyle \frac { 1 } { 2 } ( p _ { 2 }   +   1 )   <   1$ . We know that for $\mu \neq \mathbf { 0 } ,$ $\tau _ { \mathcal { D } } ( P ) = \phi ( \pmb { \mu } ) \stackrel { \_ } { > } \frac { _ 1 } { _ 2 }$ . Moreover, for any $a _ { - 1 } \in [ 0 , 0 . 5 ] , \exists a _ { 1 }$ such that $\tau _ { \mathcal { D } } ( \tilde { P } ) < \tau _ { \mathcal { D } } ( P )$ . This can be satisfied by picking $a _ { 1 }$ such that either $\gamma _ { 1 } \; \mathbf { o r } \; \gamma _ { 2 }$ is very large, i.e., $\begin{array} { r l } { \frac { 1 } { 2 } < \tau _ { \mathcal { D } } ( \tilde { P } ) = } \end{array}$ $\textstyle \frac { 1 } { 2 } [ 1 + \operatorname* { m i n } ( p _ { 1 } , p _ { 2 } ) ] { < } \tau _ { \mathcal { D } } ( P )$ . We note that the conditions $\begin{aligned}\boldsymbol{\gamma}_{1} = \boldsymbol{\mu}^{\top} \boldsymbol{A} \boldsymbol{\mu} + \hat{\boldsymbol{b}}^{\top} \boldsymbol{\mu} + c + \mathrm{Tr}(\boldsymbol{A}) + \| 2 \boldsymbol{A} \boldsymbol{\mu} + \boldsymbol{b} \|_{2} < 0\end{aligned}$ and $- \gamma _ { 2 } = - \pmb { \mu } ^ { \top } A \pmb { \mu } + \pmb { b } ^ { \top } \pmb { \mu } - c - \operatorname { T r } ( A ) + \| 2 A \pmb { \mu } - \pmb { b } \| _ { 2 } < 0$ can always be satisfied by picking a sufficiently large $\mu$ in the direction of an eigenvector corresponding to a negative eigenvalue of A (note that A has negative eigenvalues). 

## A.7. Details on generating Figure 3

We use $\pmb { \mu } \; \in \; \mathbb { R } ^ { d } , \; d \; = \; 1 0 0$ to generate clean dataset with 1000 data points. They are randomly split into training and testing partitions of equal size. All the assumptions are consistent with the details provided in the main body. We use $3 0   \times   3 0$ mesh-grid to plot the contour plots. While plotting the theoretical upper bounds, we choose the best $t _ { 1 } , t _ { 2 }$ with grid search from a search space $[ 2 ^ { 1 } , 2 ^ { 0 } , 2 ^ { - 1 } , 2 ^ { - 2 } , \dot { 2 } ^ { - 3 } , 2 ^ { - 4 } , \dot { 2 } ^ { - 5 } ]$

## A.8. Experimental details

This subsection provides the details for experiments in Section 5.

**Hardware.** We use $\mathrm { N V I D I A ^ { \circledast } }$ RTX A4000 GPU with 16GB memory with 16 $\mathrm { A M D ^ { \circledast } }$ EPYC 7302P CPU cores.

**Data augmentations.** For CIFAR-10 and CIFAR-100, we use random flipping, 4 pixel padding, and random $3 2 \times 3 2$ size cropping. For ImageNet-100, we use random flipping and random cropping with resizing to $2 2 4 \times 2 2 4$ size. All the images are rescaled to have pixel values in the range [0, 1].

**Baselines.** We compare CUDA against error-minimizing noise [17], targeted adversarial poisoning [10], neural tangent generalization attack [55], and robust error-minimizing noise [11]. We use the experimental outputs reported in [11] for our comparisons. For REM we choose $\rho _ { u } = 8 / 2 5 5$ and $\rho _ { a }   =   4 / 2 5 5$ since REM works the best when $\rho _ { u }   =   2 \rho _ { a }$ [11]. We perform experiments on REM not present in their work using their code available publicly on GitHub <sup>3</sup>(MIT License).

**Networks.** For consistency, we use the same architectures used in [11]. We use their GitHub script4for this purpose.

**Training.** We train all the networks for 100 epochs. The initial learning rate is 0.1. Learning rate decays to 0.01 at epoch 40 and to 0.001 at epoch 80. We use a stochastic gradient descent optimizer with a momentum factor of 0.9, weight decay factor of 0.0005, and batch size of 128. For adversarial training, we follow the procedure in [34]. We use 10 steps of projected gradient descent with a step size of $0 . 1 5 \rho _ { a }$

**Analysis of CUDA.** For grayscaling experiments, we use images with their average channel values as the input to the network. Test accuracy is computed on the grayscaled test datasets. For smoothing, we use the GitHub codes<sup>5</sup>from [6] (MIT License). For mixup [58], we use the default value of $\alpha = 1 . 0 .$

**Deconvolution-based adversarial training (DAT).** We experiment with various filter sizes of 3,5, and 7 for the transpose convolution filters. For each batch of data, we use 10 steps of projected gradient descent with a learning rate of 0.1 to learn transpose convolution filters for each class. The weights and biases of the transpose convolution filters are constrained to be within $[ - C , C ]$ . We choose $C = 5 .$ After 10 steps of inner maximizing optimization, the resulting image is rescaled such that the pixel values lie in [0, 1]. See Figure 4 for clean test accuracy vs. epochs plot for DAT with varying transpose filter sizes. As seen in Figure 1, DAT can break CUDA CIFAR-10 with a low blur parameter value of $p _ { b }   =   0 . 1$ to get a clean test accuracy ∼78%. However, with higher $p _ { b }$ values DAT can not achieve more than 50% clean test accuracy. DAT solves the following optimization problem:

$$
\arg \min _ {\theta} \frac {1}{n} \sum_ {k \in [ K ]} \max _ {\| s _ {k} \| _ {\infty} \leq C} \sum_ {i: y _ {i} = k} \ell \left(f _ {\theta} \left(\mathbf {x} _ {i} \star s _ {k}\right), k\right) \tag {3}
$$

where ? denotes the transpose convolution operator, $s _ { y _ { i } }$ denotes the transpose convolution filter for class $y _ { i }$ , and \` is the soft-max cross-entropy loss function.

**CUDA with augmentations.** We use mixup with the default $\alpha = 1 . 0 ~ [ 5 8 ]$ . See Figure 5 for the training curve. For random blurring augmentations, we use $p _ { b } ^ { \prime }   =   0 . 1 , 0 . 3$ and $k = 3$ . With both these parameters, CUDA is seen to be effective. See Figure 5 for the training curve with $p _ { b } ^ { \prime } = 0 . 3$

## A.9. More experimental results

Figure 6 shows the CUDA CIFAR-10 data generated using $k = 3$ and different $p _ { b }$ blur parameters. Figure 7 shows the CUDA CIFAR-100 and CUDA ImageNet-100 data generated using $k = 3 , p _ { b } = 0 . 3$ and $k = 9 , p _ { b } = 0 . 0 6 ,$ , respectively. Figure 8 shows the clean test accuracy of ResNet-18 with CUDA CIFAR-10 generated using different blur parameters. As we see in the plots, higher the blur parameter, better the effectiveness of CUDA is. However, we choose $p _ { b }   =   0 . 3$ for our experiments since the the images generated using this hyperparameter look perceptibly more similar to the clean images (when compared to $p _ { b }   =   0 . 5 )$ while giving a very low clean test accuracy. A lower value of $p _ { b }   =   0 .$ 1 gives better unlearnability. However, CUDA CIFAR-10 generated using $p _ { b } ~ = ~ 0 . 1$ is not robust with our Deconvolution-based Adversarial Training, as shown in Figure 1. Figure 9 shows the clean test accuracy of ResNet-18 with CUDA ImageNet-100 dataset generated using different filter sizes. Figure 10 shows the adversarial training curves for ResNet-18 with different CUDA datasets.

<small><span class="docvortex-page-footnote" data-block-type="page_footnote" style="color:#6b7280"><sup>3</sup>[https://github.com/fshp971/robust-unlearnableexamples](https://github.com/fshp971/robust-unlearnable-examples)</span></small>

<small><span class="docvortex-page-footnote" data-block-type="page_footnote" style="color:#6b7280"><sup>4</sup>[https://github.com/fshp971/robust-unlearnableexamples/tree/main/models](https://github.com/fshp971/robust-unlearnable-examples/tree/main/models)</span></small>

<small><span class="docvortex-page-footnote" data-block-type="page_footnote" style="color:#6b7280"><sup>5</sup>[https : / / github . com / Hadisalman / smoothing adversarial](https://github.com/Hadisalman/smoothing-adversarial)</span></small>

![](images/image_12.jpg)

<details>
<summary>line</summary>

| Epochs | Clean + ERM (%) | k = 3 (%) | k = 5 (%) | k = 7 (%) |
| --- | --- | --- | --- | --- |
| 0 | ~95 | ~12 | ~20 | ~13 |
| 10 | ~95 | ~25 | ~28 | ~22 |
| 20 | ~95 | ~40 | ~20 | ~20 |
| 30 | ~95 | ~40 | ~40 | ~35 |
| 40 | ~95 | ~35 | ~45 | ~45 |
| 50 | ~95 | ~35 | ~50 | ~55 |
| 60 | ~95 | ~40 | ~50 | ~55 |
| 70 | ~95 | ~35 | ~50 | ~50 |
| 80 | ~95 | ~40 | ~45 | ~45 |
| 90 | ~95 | ~40 | ~45 | ~45 |
| 100 | ~95 | ~40 | ~45 | ~45 |
</details>

Figure 4. CUDA CIFAR-10 images $( k   =   3 , p _ { b }   =   0 . 3 )$ trained using ResNet-18 with the Deconvolution-based Adversarial Training framework with varying transpose convolution filter sizes k.

![](images/image_13.jpg)

<details>
<summary>line</summary>

| Epochs | Clean + ERM (%) | Random blurring p'b = 0.3 (%) | Mixup \(\alpha\) = 1.0 (%) |
| --- | --- | --- | --- |
| 0 | ~95 | ~28 | ~32 |
| 10 | ~95 | ~12 | ~20 |
| 20 | ~95 | ~12 | ~22 |
| 30 | ~95 | ~18 | ~20 |
| 40 | ~95 | ~12 | ~26 |
| 50 | ~95 | ~12 | ~24 |
| 60 | ~95 | ~14 | ~26 |
| 70 | ~95 | ~12 | ~22 |
| 80 | ~95 | ~12 | ~26 |
| 90 | ~95 | ~12 | ~26 |
| 100 | ~95 | ~14 | ~26 |
</details>

Figure 5. CUDA CIFAR-10 images $( k = 3 , p _ { b } = 0 . 3 )$ trained using ResNet-18 with mixup and random blurring augmentations.

We show the effectiveness of CUDA with Tiny-ImageNet [26], DeIT [50], EfficientNetV2 [48], and MobileNetV2 [41] below.

![](images/image_14.jpg)

<details>
<summary>natural_image</summary>

Grid of 20 grayscale images showing various animals and objects, no text or symbols present
</details>

(a) p<sub>b</sub> = 0.1

![](images/image_15.jpg)

<details>
<summary>natural_image</summary>

Grid of 20 grayscale images showing various animals and objects, no text or symbols present
</details>

(b) p<sub>b</sub> = 0.3

![](images/image_16.jpg)

<details>
<summary>natural_image</summary>

Grid of 20 grayscale images showing various animals and objects, no text or symbols present
</details>

(c) p<sub>b</sub> = 0.5

Figure 6. CUDA CIFAR-10 images generated using different blur parameters p<sub>b</sub>. The top row shows the clean images, the bottom row shows the corresponding CUDA image, and the middle row shows the normalized difference between the clean and the CUDA image.

| Model | ERM | $L_2$ AT ($\epsilon = 0.5$) |
| --- | --- | --- |
| DeIT [50] | 24.85 % | 38.90 % |
| EfficientNetV2-S [48] | 20.47 % | 42.19 % |
| MobileNetV2 [41] | 21.10 % | 32.00 % |

Table 6. Effectiveness of CUDA on CIFAR-10.

## A.10. Effects of blurring

Here, we study the effects of blurring. We investigate if class-wise blurring is required for achieving unlearnability. For this, we use a universal filter (generated using the same $p _ { b } = 0 . 3$ and k = 3 hyperparameters) to blur all the training images in the dataset. A ResNet-18 trained on this dataset achieves a clean test accuracy of 90.47%. Essentially, the blurring that is performed only degrades the clean test accuracy by ∼4%. This means that class-wise blurring (CUDA) is required for achieving the unlearnability effect

| Training method | Clean | CUDA |
| --- | --- | --- |
| ERM | 48.14 % | 5.98 % |
| $L_{2}$ AT ($\epsilon = 0.5$) | 42.72 % | 14.54 % |

Table 7. Effectiveness of Tiny-ImageNet CUDA with ResNet-18. We use the same hyperparameters as our CIFAR experiments.

![](images/image_17.jpg)

<details>
<summary>natural_image</summary>

Grid of 20 grayscale images showing various objects including animals, objects, and objects with no visible text or symbols
</details>

(a) CIFAR-100

![](images/image_18.jpg)

<details>
<summary>natural_image</summary>

Collage of 20 grayscale images showing animals, vehicles, and abstract shapes (no text or symbols)
</details>

(b) ImageNet-100

Figure 7. CUDA CIFAR-100 and ImageNet-100 images generated using $k = 3 , p _ { b } = 0 . 3$ and $k = 9 , p _ { b } = 0 . 0 6 ,$ respectively. The top row shows the clean images, the bottom row shows the corresponding CUDA image, and the middle row shows the normalized difference between the clean and the CUDA image.

![](images/image_19.jpg)

<details>
<summary>line</summary>

| Epochs | Clean (%) | pb = 0.1 (%) | pb = 0.3 (%) | pb = 0.5 (%) |
| --- | --- | --- | --- | --- |
| 0 | ~95 | ~12 | ~14 | ~26 |
| 20 | ~95 | ~45 | ~20 | ~12 |
| 40 | ~95 | ~40 | ~20 | ~12 |
| 60 | ~95 | ~45 | ~20 | ~12 |
| 80 | ~95 | ~45 | ~20 | ~12 |
| 100 | ~95 | ~48 | ~20 | ~14 |
</details>

Figure 8. ResNet-18 trained using CUDA CIFAR-10 data generated using different blur parameters p<sub>b</sub>.

![](images/image_20.jpg)

<details>
<summary>line</summary>

| Epochs | Clean + ERM (%) | k=3, pb=0.3 (%) | k=5, pb=0.2 (%) | k=7, pb=0.1 (%) | k=9, pb=0.06 (%) |
| --- | --- | --- | --- | --- | --- |
| 0 | 81 | ~10 | ~10 | ~10 | ~10 |
| 20 | 81 | ~33 | ~28 | ~26 | ~23 |
| 40 | 81 | ~46 | ~40 | ~35 | ~32 |
| 60 | 81 | ~52 | ~47 | ~40 | ~36 |
| 80 | 81 | ~57 | ~51 | ~45 | ~40 |
| 100 | 81 | ~59 | ~51 | ~47 | ~41 |
</details>

Figure 9. ResNet-18 trained using CUDA ImageNet-100 dataset generated using different filter sizes k.

![](images/image_21.jpg)

<details>
<summary>line</summary>

| Epochs | Clean test accuracy (%) | Adversarial train accuracy (%) |
| --- | --- | --- |
| 0 | ~38 | ~9 |
| 10 | ~45 | ~75 |
| 20 | ~40 | ~82 |
| 30 | ~48 | ~85 |
| 40 | ~45 | ~88 |
| 50 | ~42 | ~90 |
| 60 | ~42 | ~93 |
| 70 | ~48 | ~96 |
| 80 | ~45 | ~99 |
| 90 | ~45 | ~100 |
| 100 | ~45 | ~100 |
</details>

(a) CIFAR-10

![](images/image_22.jpg)

<details>
<summary>line</summary>

| Epochs | Clean test accuracy (%) | Adversarial train accuracy (%) |
| --- | --- | --- |
| 0 | ~5 | ~5 |
| 10 | ~25 | ~40 |
| 20 | ~25 | ~55 |
| 30 | ~28 | ~58 |
| 40 | ~30 | ~65 |
| 50 | ~33 | ~80 |
| 60 | ~33 | ~88 |
| 70 | ~34 | ~92 |
| 80 | ~34 | ~95 |
| 90 | ~35 | ~98 |
| 100 | ~35 | ~99 |
</details>

(b) CIFAR-100

![](images/image_23.jpg)

<details>
<summary>line</summary>

| Epochs | Clean test accuracy (%) | Adversarial train accuracy (%) |
| --- | --- | --- |
| 0 | ~12 | ~0 |
| 10 | ~25 | ~18 |
| 20 | ~25 | ~20 |
| 30 | ~25 | ~25 |
| 40 | ~25 | ~25 |
| 50 | ~32 | ~35 |
| 60 | ~32 | ~35 |
| 70 | ~32 | ~35 |
| 80 | ~35 | ~40 |
| 90 | ~38 | ~50 |
| 100 | ~38 | ~58 |
</details>

(c) ImageNet-100

Figure 10. Adversarial training curves for ResNet-18 with CIFAR-10, CIFAR-100, and ImageNet-100 CUDA datasets.

(see Figure 11). This experiment also demonstrates that the blurring we perform does not make the dataset useless or destroy its semantics. For this experiment, we use models with fixed initialization and random seeds.

## A.11. Why does CUDA work?

In this section, we perform experiments that show that a model trained on CUDA dataset learns the relation between the class-wise filters and the labels. We train ResNet-18 using the CUDA CIFAR-10 dataset for the experiments. We perform three independent trials for each of the experiments and report the mean performance scores. Trained models achieve a mean clean test accuracy of 21.34%. Now, we use the class-wise filters to perturb the images in the test set based on their corresponding labels. Trained models achieve a very high mean accuracy of 99.91% on this perturbed test set. This shows that the trained models learned the relation between the filters and their corresponding labels. Next, we permute the filters to perturb the test set such that test set images with label i are perturbed with the filters of class (i + 1)%10. Trained models achieve a very low mean accuracy of 2.53% on this perturbed test set. This is evidence that CUDA can also be used for backdoor attacks.

## A.12. Effect of transfer learning

In this section, we experiment the effect of using a pre-trained ResNet-18 with PyTorch [38]. We train it on the CUDA CIFAR-10 dataset in two different ways. First, we fine-tune the whole network on the CUDA dataset with a learning rate of 0.001 for 15 epochs. This achieves a clean test accuracy of 42.42%. Fine-tuning the network with clean training data gives 94.19% clean test accuracy. Next, we freeze all the layers except the final layer to train a linear classifier with the pre-trained weights using the CUDA

![](images/image_24.jpg)

<details>
<summary>line</summary>

| Epochs | FUN | Clean | Universal blurring |
| --- | --- | --- | --- |
| 0 | ~0.36 | ~0.36 | ~0.40 |
| 10 | ~0.25 | ~0.75 | ~0.70 |
| 20 | ~0.22 | ~0.82 | ~0.78 |
| 30 | ~0.25 | ~0.85 | ~0.82 |
| 40 | ~0.25 | ~0.88 | ~0.82 |
| 50 | ~0.26 | ~0.93 | ~0.90 |
| 60 | ~0.26 | ~0.93 | ~0.90 |
| 70 | ~0.25 | ~0.93 | ~0.90 |
| 80 | ~0.26 | ~0.93 | ~0.90 |
| 90 | ~0.25 | ~0.93 | ~0.90 |
| 100 | ~0.26 | ~0.93 | ~0.90 |
</details>

(a) Test accuracy

![](images/image_25.jpg)

<details>
<summary>line</summary>

| Epochs | FUN (Training loss) | Clean (Training loss) | Universal blurring (Training loss) |
| --- | --- | --- | --- |
| 0 | ~1.95 | ~1.95 | ~1.95 |
| 5 | ~0.20 | ~0.60 | ~0.65 |
| 10 | ~0.12 | ~0.40 | ~0.45 |
| 15 | ~0.08 | ~0.30 | ~0.35 |
| 20 | ~0.07 | ~0.25 | ~0.28 |
| 25 | ~0.06 | ~0.20 | ~0.22 |
| 30 | ~0.05 | ~0.18 | ~0.20 |
| 35 | ~0.06 | ~0.17 | ~0.19 |
| 40 | ~0.04 | ~0.15 | ~0.16 |
| 45 | ~0.01 | ~0.05 | ~0.06 |
| 50 | ~0.01 | ~0.02 | ~0.03 |
| 60 | ~0.00 | ~0.01 | ~0.01 |
| 80 | ~0.00 | ~0.00 | ~0.00 |
| 100 | ~0.00 | ~0.00 | ~0.00 |
</details>

(b) Training loss

Figure 11. ResNet-18 trained using clean, CUDA, and universally blurred CIFAR-10 datasets.

CIFAR-10 dataset. We call this “Freeze and learn”. We use a SGD optimizer to train the linear layer for 15 epochs with an initial learning rate of 0.1. The learning rate is decayed by a factor of 10 after every 5 epochs. This achieves a clean test accuracy of 48.22%. The results are shown in Figure 12. This experiment shows that pre-trained network with CUDA data training does not help achieve good generalization on the clean data distribution.

## A.13. Effect of CUDA with regularization techniques

In this section, we study the effect of training a ResNet-18 with CUDA CIFAR-10 dataset using various regularization techniques such as mixup [58], cutout [9], cutmix [56], autoaugment [7], and orthogonal regularization [3]. We perform mixup, cutout, cutmix, autoaugment, and orthogonal regularization to achieve 25.53%, 25.80%, 26.93%, 34.09%, and 50.72%. Even though these regularizations help in improving the vanilla ERM training, these networks still do not achieve good generalization on the clean data distribution. We use cutout using GitHub codes<sup>6</sup> with length=16 and n holes=1, cutmix using GitHub codes<sup>7</sup> with α = 1, autoaugment using PyTorch [38], mixup using GitHub codes<sup>8</sup> with α = 1, and orthogonal regularization using GitHub codes<sup>9</sup> with reg=1e-6 (all MIT licenses).

## A.14. Network parameter distribution

In this section, we compare the network parameter distributions of ResNet-18 trained on clean and CUDA CIFAR-10 datasets (see Figure 13). Both the distributions are similar to normal distributions with a mean of 0. However, the parameter distribution of the clean model has a higher standard deviation than the CUDA-based model’s parameter distribution.

<small><span class="docvortex-page-footnote" data-block-type="page_footnote" style="color:#6b7280"><sup>6</sup>[https://github.com/uoguelph-mlrg/Cutout/blob/ master/util/cutout.py](https://github.com/uoguelph-mlrg/Cutout/blob/master/util/cutout.py)</span></small>

<small><span class="docvortex-page-footnote" data-block-type="page_footnote" style="color:#6b7280"><sup>7</sup>[https://github.com/hysts/pytorch\_cutmix/blob/ master/cutmix.py](https://github.com/hysts/pytorch_cutmix/blob/master/cutmix.py)</span></small>

<small><span class="docvortex-page-footnote" data-block-type="page_footnote" style="color:#6b7280"><sup>8</sup>[https : / / github . com / facebookresearch / mixup - cifar10/blob/main/train.py](https://github.com/facebookresearch/mixup-cifar10/blob/main/train.py)</span></small>

<small><span class="docvortex-page-footnote" data-block-type="page_footnote" style="color:#6b7280"><sup>9</sup>[https://github.com/kevinzakka/pytorch-goodies](https://github.com/kevinzakka/pytorch-goodies)</span></small>

![](images/image_26.jpg)

<details>
<summary>line</summary>

| Epochs | Fine-tuning + FUN | Fine-tuning + Clean | Freeze and learn + FUN |
| --- | --- | --- | --- |
| 1 | ~0.49 | ~0.81 | ~0.44 |
| 2 | ~0.50 | ~0.88 | ~0.42 |
| 3 | ~0.50 | ~0.90 | ~0.46 |
| 4 | ~0.48 | ~0.91 | ~0.50 |
| 5 | ~0.48 | ~0.92 | ~0.46 |
| 6 | ~0.47 | ~0.93 | ~0.49 |
| 7 | ~0.47 | ~0.93 | ~0.49 |
| 8 | ~0.46 | ~0.94 | ~0.48 |
| 9 | ~0.45 | ~0.94 | ~0.50 |
| 10 | ~0.45 | ~0.94 | ~0.48 |
| 11 | ~0.44 | ~0.94 | ~0.48 |
| 12 | ~0.44 | ~0.94 | ~0.48 |
| 13 | ~0.43 | ~0.94 | ~0.48 |
| 14 | ~0.43 | ~0.94 | ~0.49 |
| 15 | ~0.43 | ~0.94 | ~0.48 |
</details>

(a) Test accuracy

![](images/image_27.jpg)

<details>
<summary>line</summary>

| Epochs | Freeze and learn + FUN | Fine-tuning + FUN | Fine-tuning + Clean |
| --- | --- | --- | --- |
| 1 | ~1.5 | ~1.4 | ~1.3 |
| 2 | ~0.77 | ~0.46 | ~0.52 |
| 3 | ~0.67 | ~0.24 | ~0.35 |
| 4 | ~0.74 | ~0.16 | ~0.29 |
| 5 | ~0.69 | ~0.12 | ~0.25 |
| 6 | ~0.70 | ~0.10 | ~0.22 |
| 7 | ~0.55 | ~0.08 | ~0.20 |
| 8 | ~0.54 | ~0.07 | ~0.19 |
| 9 | ~0.53 | ~0.06 | ~0.18 |
| 10 | ~0.53 | ~0.05 | ~0.17 |
| 11 | ~0.52 | ~0.05 | ~0.16 |
| 12 | ~0.52 | ~0.04 | ~0.15 |
| 13 | ~0.51 | ~0.04 | ~0.14 |
| 14 | ~0.51 | ~0.03 | ~0.13 |
| 15 | ~0.52 | ~0.03 | ~0.12 |
</details>

(b) Training loss

Figure 12. Pre-trained ResNet-18 with fine-tuning and training the linear layer using CUDA and clean CIFAR-10 datasets.

![](images/image_28.jpg)

<details>
<summary>histogram</summary>

| Bin (Range) | FUN (Frequency) | Clean (Frequency) |
| --- | --- | --- |
| -0.03~-0.02 | ~0 | ~0 |
| -0.02~-0.01 | ~0 | ~0.05e6 |
| -0.01~0.00 | ~0.05e6 | ~0.4e6 |
| 0.00~0.01 | ~0.05e6 | ~0.4e6 |
| 0.01~0.02 | ~0 | ~0.05e6 |
| 0.02~0.03 | ~0 | ~0 |
</details>

Figure 13. Network parameter distributions of ResNet-18 trained on clean and CUDA CIFAR-10 datasets.