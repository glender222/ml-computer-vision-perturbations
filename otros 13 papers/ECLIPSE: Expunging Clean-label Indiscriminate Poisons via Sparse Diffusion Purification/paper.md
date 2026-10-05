<!-- MinerU 4.0.9, tier advanced, VLM llama.cpp/Vulkan GPU; automatic extraction, tables require comparison with the PDF. -->

[PDF original](../../ECLIPSE%3A%20Expunging%20Clean-label%20Indiscriminate%20Poisons%20via%20Sparse%20Diffusion%20Purification.pdf)

# ECLIPSE: Expunging Clean-label Indiscriminate Poisons via Sparse Diffusion Purification

Xianlong Wang<sup>1,2,4,5†</sup>, Shengshan Hu<sup>1,2,4,5†</sup>(<sup>B</sup>), Yechao Zhang<sup>1,2,4,5†</sup>, Ziqi Zhou<sup>1,2,3</sup>§, Leo Yu Zhang†† , Peng Xu<sup>1,2,4,5†</sup>(<sup>B</sup>), Wei Wan<sup>1,2,4,5†</sup>, and Hai Jin<sup>1,2,3</sup>§

<sup>1</sup>National Engineering Research Center for Big Data Technology and System <sup>2</sup>Services Computing Technology and System Lab <sup>3</sup>Cluster and Grid Computing Lab <sup>4</sup>Hubei Engineering Research Center on Big Data Security <sup>5</sup>Hubei Key Laboratory of Distributed System Security
<sup>†</sup>School of Cyber Science and Engineering, Huazhong University of Science and Technology, Wuhan 430074, China
§School of Computer Science and Technology, Huazhong University of Science and Technology, Wuhan 430074, China
<sup>††</sup>School of Information and Communication Technology, Griffith University, Southport QLD 4215, Australia
{wxl99,hushengshan,ycz,zhouziqi,xupeng,wanwei 0303,hjin}@hust.edu.cn leo.zhang@griffith.edu.au

Abstract. Clean-label indiscriminate poisoning attacks add invisible perturbations to correctly labeled training images, thus dramatically reducing the generalization capability of the victim models. Recently, defense mechanisms such as adversarial training, image transformation techniques, and image purification have been proposed. However, these schemes are either susceptible to adaptive attacks, built on unrealistic assumptions, or only effective against specific poison types, limiting their universal applicability. In this research, we propose a more universally effective, practical, and robust defense scheme called ECLIPSE. We first investigate the impact of Gaussian noise on the poisons and theoretically prove that any kind of poison will be largely assimilated when imposing sufficient random noise. In light of this, we assume the victim has access to an extremely limited number of clean images (a more practical scene) and subsequently enlarge this sparse set for training a denoising probabilistic model (a universal denoising tool). We then introduce Gaussian noise to absorb the poisons and apply the model for denoising, resulting in a roughly purified dataset. Finally, to address the trade-off of the inconsistency in the assimilation sensitivity of different poisons by Gaussian noise, we propose a lightweight corruption compensation module to effectively eliminate residual poisons, providing a more universal defense approach. Extensive experiments demonstrate that our defense approach outperforms 10 state-of-the-art defenses. We also propose an adaptive attack against ECLIPSE and verify the robustness of our defense scheme. Our code is available at [https://github.com/CGCL-codes/ECLIPSE](https://github.com/CGCL-codes/ECLIPSE).

Keywords: Deep neural network · Poisoning attack · Diffusion model.

## 1 Introduction

The success of deep neural networks (DNNs) relies on abundant training data, motivating many commercial firms to supply their training set by automatically scraping images from untrusted sources. However, these untrusted data have the potential to be exploited by adversaries to poison DNNs, challenging their trustworthiness in safety-critical applications [8].

Recently, there has been a rise in the occurrence of clean-label indiscriminate poisoning attacks that add imperceptible perturbations to correctly labeled images, thus dramatically compromising DNNs. These perturbations are usually norm-bounded and together with clean labels, constitute the concealment of such attacks, making them easier to implement in real-world scenarios. Based on this, in this research, we focus on clean-label indiscriminate poisoning attacks with bounded perturbations (CLBPAs) [3,8,9,10,17,32,34,41,42,43], which introduce a great challenge for defenders.

Existing defense strategies have been successively proposed but suffer from the following limitations: 1 Limited effectiveness against certain CLBPA types. Many defense schemes are only effective against specific types of CLB-PAs, e.g., the grayscale transformation in image shortcut squeezing (ISS) [22] is only effective against low-frequency poisons, and OP [31] only works when facing class-wise poisons. It is crucial to design a more universally applicable defense against CLBPAs since the concealment of poisons makes it difficult for defenders to identify the type of bounded perturbations being used as shown in Fig. 1 (b); 2 Making impractical assumptions. Several purification schemes [6,18] are proposed to defend against CLBPAs via diffusion denoising. However, these approaches are impractical as they make unrealistic assumptions about the clean training set, e.g., Dolatabadi et al. [6] assume the defender can obtain the whole clean training set to train a diffusion model, which seriously violates the assumption of CLBPA implemented during the training phase; 3 Fragile to adaptive attacks. Many vulnerable defense schemes are easily compromised by adaptive attacks shortly after their proposal, e.g., Tao et al. [38] suggest that adversarial training (AT) can address CLBPAs, but a series of adaptive attacks [10,41] subsequently compromise AT. ISS is also susceptible to adaptive attacks, as acknowledged by [22].

Additionally, it is intricate to determine whether the training set is clean or poisoned due to the concealment of bounded poisons as shown in Fig. 1 (a) and (b). As a result, any defense against CLBPAs must be applied without significantly compromising accuracy in the absence of poisons. Based on this, we propose clean accuracy, i.e., model accuracy when applied with defense on the clean dataset, to serve as another significant evaluation metric that has been underrepresented in prior works. We then reassess previous state-of-the-art (SOTA) defenses [22,38] with this metric in Tab. 3 and find that they both negatively impact clean training to some extent. Therefore, there is an urgent need to design a defense scheme against CLBPAs that is more universally effective, practical, robust to adaptive attacks, and does not substantially impair clean accuracy.

![](images/page_2_image_2.jpg)

<details>
<summary>text_image</summary>

Poisoned images with Lp bounded perturbations
Clean
(a)
EM↓
TAP↑
REM↓
SEP↑
EFP↓
URP↑
LSP↓
AR↑
(b)
</details>

Fig. 1: We present eight popular clean-label indiscriminate poisoning attacks along with clean samples. The upward and downward arrows represent highfrequency and low-frequency poison perturbations, respectively. It can be observed that it is difficult for the naked eyes to distinguish between clean samples and poisoned samples.

Our intuition starts with a key observation (Fig. 2 (a)) and a theoretical guarantee (Theorem 1), i.e., the morphology of various poisons can be assimilated by gradually introducing random Gaussian noise. Once by denoising the noised input, we can effectively eliminate the noise as well as poison, which motivates us to apply a denoising diffusion probabilistic model [13] to handle the denoising process. In the context of CLBPA where acquiring ample clean training data is not feasible, the challenge of training a diffusion model emerges as a new obstacle for us. We are constrained to assume the defender has privately stored an extremely limited amount of data (less than 5% of the size of the training dataset from the main task) that is distributed identically to the clean training set. To overcome data scarcity in our practical assumption, we propose leveraging a data enlargement module to augment the dataset used for training the diffusion model.

Additionally, owing to discrepancies in the assimilation effects caused by diverse poison patterns, some poisons necessitate more noise for assimilation. However, this will also lead to excessive noise absorption for other poisons, resulting in a damage of image features. To address the trade-off between the purification effects of diverse poisons, after introducing moderate Gaussian noise and applying the denoising process, we further propose a lightweight compensation module that incorporates both probabilistic grayscale transformation and the lightweight Gaussian noise. Only in this way can we achieve a more universally effective and practical defense strategy against diverse CLBPAs, while demonstrating superior clean accuracy compared to existing SOTA defense approaches. Furthermore, we devise an adaptive CLBPA based on information gleaned from our defense, thereby showcasing the effectiveness of our defense in countering this attack. Our main contributions can be summarized as follows:

• We establish the theoretical and experimental evidence supporting the assimilation of poisons by Gaussian noise. Leveraging this insight, we propose a more universally effective defense using forward noise addition and a denoising process facilitated by the diffusion model.

We point out the existing diffusion purification schemes for defending against CLBPAs are impractical as they require a large amount of clean training samples. Instead, we propose training the diffusion model solely with sparse, identically distributed clean data.
• To address the trade-off of the inconsistency in the assimilation sensitivity of different poisons by Gaussian noise, we propose a lightweight compensation module to remove residual poisons and provide an explanation of the roles of corruptions from a frequency perspective.
• Extensive experiments on multiple benchmark datasets including CIFAR-10 and ImageNet demonstrate that our defense scheme can outperform the SOTA defense methods in test accuracy by 38.4% on CIFAR-10 using ResNet18 and in clean accuracy by 4.41% on CIFAR-10. Additionally, our defense’s effectiveness remains robust against our newly proposed adaptive poisoning attack, further affirming its reliability.

## 2 Related Work

## 2.1 Clean-label Indiscriminate Poisoning Attacks

Traditional indiscriminate poisoning attacks inject noisy labels [1,24,47] and can be easily detected by human observers. So existing works focus on cleanlabel indiscriminate poisoning attacks [9,32,41], a.k.a., clean-label generalization attack [7,43], clean-label availability poisoning attack [42], delusive attack [38] or simply unlearnable examples [17], which contaminate correctly labeled training images with $\mathcal { L } _ { p }$ norm bounded perturbations, i.e., CLBPAs, to ensure stealthiness, including solving bi-level optimization problems to produce error-minimizing noises (EM) [17] or serving targeted adversarial samples (TAP) as poison perturbations [9]. Chen et al. [3] enhanced the poisoning effectiveness by employing self-ensemble of model checkpoints (SEP). In addition, some adaptive CLBPAs designed for AT have also been proposed one after another, e.g., REM [10] and EFP [41], reducing the defense universality of AT. While the above schemes often rely on external networks, resulting in substantial time costs, several model-agnostic CLBPAs have emerged. For instance, universal random perturbation (URP) adds the same random Gaussian noise to images from the same category, offering a simpler and more efficient poisoning attack [17,38]. Linearly separable perturbations (LSP) [42] and autoregressive poisons (AR) [32] also fall into this category, prioritizing efficiency and transferability.

## 2.2 Defenses Against Poisoning Attacks

There are many approaches available to defend against data poisoning attacks, including differential privacy [14], and strong data augmentations [2,5,44,45]. However, these schemes are not specifically optimized to handle CLBPAs and proved significantly less effective in addressing them based on our experimental results. In light of this, Tao et al. [38] experimentally and theoretically proved that AT [23,49,50,51] can be applied to defend against CLBPAs. Unfortunately, some stronger adaptive CLBPAs against AT are proposed and can effectively break AT [10,40,41]. Subsequently, Liu et al. [22] found that grayscale transformation is effective against low-frequency poisons and JPEG compression is effective against high-frequency poisons, which can effectively defend against CLBPAs. But its fatal flaw is that each simple transformation only has the best defense effect for specific types of poisons, lacking a universal solution. Soon after, Qin et al. [27] introduced adversarial augmentation, Sandoval et al. [31] proposed orthogonal projection, but they both only work against certain poisons and are not a universally effective defense solution. Another defensive route involves image purification [6,18] through using a diffusion model to denoise the poisoned samples. But they made the unrealistic assumption of owning ample clean training images, seriously violating the scenario definition of data poisoning attacks. It is desirable but challenging to design a versatile, practical, and robust defense approach against CLBPAs.

## 3 Methodology

## 3.1 Threat Model

Following the standard framework of CLBPAs [3,9,17,28,30,38,41,42], we assume the attacker manipulates all the training images with bounded perturbations with $L _ { p }$ norm. The attacker aims to cause the model G with parameter θ trained on the poisoned dataset to generalize poorly to a clean data distribution D. Formally, the attacker expects to work out the following bi-level objective:

$$
\max _ {(x, y) \sim \mathcal {D}} \mathbb {E} \left[ \mathcal {L} \left(G \left(x; \theta_ {p}\right), y\right) \right] \tag {1}
$$

$$
\text {s.t.} \theta_ {p} = \arg \min _ {\theta} \sum_ {\left(x _ {i}, y _ {i}\right) \in \mathcal {D} _ {c}} \mathcal {L} \left(G \left(x _ {i} + \delta_ {i}; \theta\right), y _ {i}\right) \tag {2}
$$

where $( x _ { i } , y _ { i } )$ represents the clean data belonging to the clean training set $\mathcal { D } _ { c } ,   \delta _ { i }$ is the elaborate perturbation to poison the training set with $L _ { p }$ norm constraint, and L is a loss function, $e . g .$ , cross-entropy loss. As for defenders, we only assume that they access to an extremely low proportion, $e . g .$ , 5% of clean samples from the same training distribution. The defenders aim to perform operations on the poisoned images to achieve the opposite goal of Eq. (1), while not involving any knowledge of the victim models.

## 3.2 Motivation for Studying Defenses Against CLBPAs

CLBPAs inject malicious noise into training data, causing a decline in the performance of DNN models, which poses significant harm in real-world scenarios. For instance, poisoning data collected by web crawlers during the training of large models can degrade model performance [9,32,41]. Additionally, poisoning internal data used for peer assessment or academic research by proxy applications and subsequently training models on these contaminated data can result in severely degraded performance [7]. Therefore, researching defense mechanisms against CLBPAs holds strong practical significance for a wide range of DNN-based technologies or applications in real-world settings.

## 3.3 Key Intuition and Theoretical Insight

We visually distinguish eight poison patterns as illustrated in the top row of Fig. 2 (a). These distinct perturbation patterns underscore the complexity of mitigating CLBPAs with a universal scheme, which is completely different from expunging adversarial perturbations composed of only one high-frequency pattern via the existing diffusion purification scheme [26]. Therefore, this poses a thought-provoking question for us:

Can diffusion purification expunge both high and low-frequency poison perturbations?

To answer this, we progressively apply incremental random Gaussian noise to these poisons, then the patterns from different poisoning attacks begin to resemble each other, ultimately converging to Gaussian noise, as shown in $Fig. 2 (a)$ This observation suggests that bounded poison perturbations can be ultimately assimilated by random Gaussian noise. Regardless of how the images themselves change, it will not affect the actual assimilation effect of the Gaussian noise we add. Additionally, we provide the theoretical insight for this conclusion:

Theorem 1: Assuming $p ( x , t )$ and $q ( x , t )$ represent the poisoned data distribution $p ( x , \cdot )$ and clean data distribution $q ( x , \cdot )$ after undergoing the forward Gaussian noise process with time t, respectively (note that the poison perturbations in $p ( x , \cdot )$ are constrained within an $L _ { p }$ norm ball), we have:

$$
\frac {\partial D _ {K L} \left(p (x , t) \| q (x , t)\right)}{\partial t} \leq 0
$$

where $D _ { K L }$ denotes Kullback-Leibler divergence.

## Proof: See Appendix.

This theorem indicates that as the noise level in the forward process increases, the distribution of the poisoned dataset becomes closer to the distribution of the clean dataset after adding noise, which means that the impact of any poison diminishes over time, i.e., the continuously added noise will eventually absorb all types of bounded poison perturbations. Thus we propose serving the diffusion model as a more universally effective denoising tool for eliminating diverse types of poison perturbations.

## 3.4 Challenges and Approaches

To design a more universally effective defense strategy against existing CLBPAs, we suffer from several challenges as follows:

![](images/page_6_image_2.jpg)

Fig. 2: (a) We present eight types of poison perturbations and add Gaussian noise that is subject to normal distribution $\mathcal { N } ( 0 , 0 . 0 1 ^ { 2 } )$ from one to fifty rounds gradually. We observe that the assimilation of Gaussian noise to low-frequency perturbations is slow, while the assimilation to high-frequency perturbations is faster; (b) The high-level overview of our proposed defense scheme ECLIPSE.

Challenge I: In the context of data poisoning attacks, the absence of the clean training set prevents the training of a diffusion model. Existing diffusion defenses are unrealistically assumed by Dolatabadi et al. [6] to obtain 100% clean training images to train a diffusion model and Jiang et al. [18] to obtain 20% clean training images to fine-tune a clean data trained diffusion model. These impractical assumptions motivate us to propose a more realistic one. Firstly, we assume the defender only owns sparse identically distributed clean data instead of directly owning any clean training images. Secondly, our defender does not require any pre-trained diffusion models to fine-tune, thus training from scratch using sparse data.

Challenge II: The universal Gaussian noise scale will lead to asynchrony in the absorption of different poisons. From Fig. 2 (a), it can be observed that low-frequency poisons are assimilated more slowly, indicating that less Gaussian noise is beneficial for absorbing high-frequency poisons while low-frequency poisons cannot be completely absorbed. On the other hand, more Gaussian noise is advantageous for low-frequency poisons but may negatively impact the features of images containing high-frequency poisons.

The promising approach to resolve this dilemma is to set an appropriate value of the intensity of added noise, which is sufficient to absorb high-frequency poison perturbations and then design a compensation module to further expunge the residual poisons while simultaneously minimizing harm to purified poisoned images as much as possible.

## 3.5 Our Design for ECLIPSE

The high-level overview of our defense approach ECLIPSE is shown in Fig. 2 (b) and the specific implementation steps are as follows.

Sparsely training a denoising tool. We assume that the defender privately stored a sparse image set $\mathcal { D } _ { s } = \{ s _ { i } \} _ { i = 1 } ^ { B } ,$ , which only shares the same distribution as the clean training set (the size is N). Ensuring that the sparse dataset and the distribution of clean data are from the same distribution is crucial [6,26]. We set $\begin{array} { r } { B \ll N ~ ( \frac { \dot { B } } { N } } \end{array}$ is less than 5%), making our assumption more practical in poisoning attacks. To address the trade-off between practical assumption and the size of sparse set, we attempt to augment the dataset using various standard data augmentation techniques, including cropping, flipping, rotation, and the strong data augmentation mixup [45]. Unfortunately, the use of these data augmentations has proven ineffective in defending against certain attacks, $e . g .$ SEP [3] (see Fig. 3), limiting the universal effectiveness of our defense solution. We speculate that this is because the essence of training diffusion models lies in the learning the mapping from the noised data distribution to the clean data distribution, and yet the data augmentations alter the original clean data distribution [48], impacting the sampling ability of diffusion models.

![](images/page_7_chart_2.jpg)

<details>
<summary>bar_line</summary>

| Category | SEP (%) | LSP (%) | AR (%) | AVG (%) |
| --- | --- | --- | --- | --- |
| Cropping | ~55 | ~84 | ~88 | ~76 |
| Flipping | ~41 | ~82 | ~88 | ~70 |
| Rotation | ~45 | ~83 | ~88 | ~72 |
| Mixup | ~43 | ~80 | ~88 | ~70 |
| Ours | ~83 | ~84 | ~88 | ~85 |
</details>

Fig. 3: The defense performance of ECLIPSE using diverse data augmentation techniques and our scheme against three CLBPAs, SEP [3], AR [32], and LSP [42] using ResNet18 on the CIFAR-10 dataset

To address this, we propose to directly duplicate the original dataset, thus maintaining the distribution of the augmented dataset entirely consistent with the original, while also increasing the volume of data (the results in Fig. 3 demonstrate the effectiveness of our augmenting approach for training diffusion models with sparse data). We formulize our repetitive data enlargement scheme as:

$$
\mathcal {D} _ {A} = \mathcal {D} _ {s} \cup \mathcal {R} \left(\mathcal {D} _ {s}, M\right) \tag {3}
$$

where function R denotes the dataset obtained after performing the replication operation on the dataset $\mathcal { D } _ { s }$ , M represents the number of replications, and $\mathcal { D } _ { A }$ represents the enlarged dataset used to train the diffusion model.

We then employ an unconditional diffusion process to generate $x _ { 1 } , x _ { 2 } , \ldots ,$ x<sub>T</sub> based on an initial image $x _ { 0 }$ sampled from our enlarged dataset $\mathcal { D } _ { A }$ . The forward random Gaussian noise-adding process is formulated as:

$$
q \left(x _ {0}, x _ {1}, \dots , x _ {T}\right) = q \left(x _ {0}\right) \prod_ {t = 1} ^ {T} q \left(x _ {t} \mid x _ {t - 1}\right) \tag {4}
$$

$$
q \left(x _ {t} \mid x _ {t - 1}\right) = \mathcal {N} \left(x _ {t}; \alpha_ {t} x _ {t - 1}, \beta_ {t} \mathbf {I}\right) \tag {5}
$$

where $q \left( x _ { 0 } , x _ { 1 } , \ldots , x _ { T } \right)$ is the joint distribution of forward process, $\beta _ { t }$ is the variance of random noise at time $t ,   \alpha _ { t }$ and $\beta _ { t }$ satisfy $\alpha _ { t } ^ { 2 } + \beta _ { t } ^ { 2 } \bar { } = 1$ . The training

optimization goal is:

$$
\min D _ {K L} (q \| p) = \int q \log \frac {q}{p} d x _ {0} d x _ {1} \dots d x _ {T} \tag {6}
$$

where $p$ represents the joint distribution of estimated reverse process, $D _ { K L }$ denotes the Kullback-Leibler divergence, which is used to measure the similarity between two distributions. By simplifying Eq. (6), ignoring the constant obtained from the integration and reducing coefficients as suggested by [13], the loss function becomes:

$$
L _ {s} = \mathbb {E} \left[ \left\| \boldsymbol {\epsilon} - \boldsymbol {\epsilon} _ {\boldsymbol {\theta}} \left(\sqrt {1 - \bar {\alpha} _ {t}} \boldsymbol {\epsilon} + \sqrt {\bar {\alpha} _ {t}} x _ {0}, t\right) \right\| ^ {2} \right] \tag {7}
$$

where $\bar { \alpha } _ { t } = \alpha _ { 1 } \cdotp \alpha _ { 2 } \cdots \alpha _ { t } , \thinspace \mathbf { \epsilon } \sim \mathcal { N } ( \mathbf { 0 } , \mathbf { I } ) , \thinspace x _ { 0 } \sim q \left( x _ { 0 } \right) , \thinspace \mathbf { \epsilon } _ { \mathbf { \theta } }$ is used to predict ϵ from $x _ { t }$ working as a function approximator. We utilize a cosine-based variance schedule, set larger diffusion steps, and predict a mixing vector v to learn diagonal variance as an interpolation between $\tilde { \beta } _ { t }   =   \beta _ { t } \cdot \left( 1 - \bar { \alpha } _ { t - 1 } \right) / \left( 1 - \bar { \alpha } _ { t } \right)$ and $\beta _ { t }$ to achieve a better training process of diffusion as suggested by Nichol and Dhariwal [25]. Thus the new optimization objective is:

$$
L = L _ {s} + \gamma \sum_ {t = 0} ^ {T} L _ {t}, L _ {0} := - \log p _ {\theta} \left(x _ {0} \mid x _ {1}\right)
$$

$$
L _ {t - 1} := D _ {K L} \left(q \left(x _ {t - 1} \mid x _ {t}, x _ {0}\right) \| p _ {\theta} \left(x _ {t - 1} \mid x _ {t}\right)\right), L _ {T} := D _ {K L} \left(q \left(x _ {T} \mid x _ {0}\right) \| p \left(x _ {T}\right)\right) \tag {8}
$$

where $p _ { \theta } ( x _ { t - 1 } \mid x _ { t } ) = \mathcal { N } ( x _ { t - 1 } ; \mu _ { \theta } \left( x _ { t } , t \right) , \varSigma _ { \theta } \left( x _ { t } , t \right) ) , \thinspace \gamma$ is a hyper-parameter. The learnable denoising parameters $\mu _ { \theta } ( x _ { t } , t )$ and $\varSigma _ { \theta } \left( x _ { t } , t \right)$ are calculated as:

$$
\mu_ {\theta} \left(x _ {t}, t\right) = \frac {x _ {t} - \frac {1 - \alpha_ {t}}{\sqrt {1 - \bar {\alpha} _ {t}}} \epsilon_ {\boldsymbol {\theta}} \left(x _ {t} , t\right)}{\sqrt {\alpha_ {t}}} \tag {9}
$$

$$
\Sigma_ {\theta} (x _ {t}, t) = \exp ((1 - v) \log \tilde {\beta} _ {t} + v \log \beta_ {t}) \tag {10}
$$

Absorbing and eliminating diverse poisons. Motivated by our key intuition and theoretical insight, we first add random Gaussian noise to poisoned images for absorbing the poison perturbations, which is implemented by the continuoustime diffusion mode [37] as:

$$
x _ {t ^ {*}} = \sqrt {\alpha (t ^ {*})} x _ {p} + \sqrt {1 - \alpha (t ^ {*})} \epsilon \tag {11}
$$

where α $(t) = e^{k_1 t^2 + k_2 t},   k_1$ and $k _ { 2 }$ are constants below 0, $\mathbf { \epsilon } \sim \mathcal { N } ( 0 , I ) ,   x _ { p }$ is the poisoned image, forward step $t ^ { * }$ represents the strength of Gaussian noise added. After absorbing process, we employ the denoising parameters in Eqs. (9) and (10) from the sparse data trained diffusion model to eliminate poison perturbations, which is defined as:

$$
x _ {t - 1} = \mu_ {\theta} (x _ {t}, t) + \Sigma_ {\theta} (x _ {t}, t) \mathbf {z} \tag {12}
$$

where $\mathbf { z } \sim \mathcal { N } ( 0 , I )$ and we set $\mathbf { z } = \mathbf { 0 }$ when $t = 1$

Algorithm 1: Our defense scheme ECLIPSE

<div class="docvortex-algorithm" style="white-space: pre-wrap; font-family:monospace;">
Input: Poisoned dataset $\{(x_{p_i}, y_i) \mid i = 1, 2, ..., N\}$; sparse image set
    $\mathcal{D}_s = \{s_i \mid i = 1, 2, ..., B\} (B \ll N)$; replication times $M$; training
    iteration $I$; diffusion step $T$; forward step $t^*$; grayscale probability $p$;
    standard deviation $\sigma$
Output: Final dataset $\{(x_{f_i}, y_i) \mid i = 1, 2, ..., N\}$
Function: Loss $L$; $\alpha(t)$; corruption function $C(\cdot; p, \sigma)$.
Initialize $\mathcal{D}_A = \mathcal{D}_s$;
for $i = 1$ to $M$ do
    $\mathcal{D}_A = \mathcal{D}_A \cup \mathcal{D}_s$; $\triangleright$ enlarge the sparse set
end
for $i = 1$ to $I$ do
    $\epsilon \sim \mathcal{N}(0, I)$, $t \sim U(1, T)$;
    Randomly sample image $x_0$ from $\mathcal{D}_A$;
    Perform a gradient descent step on $\nabla_\theta L$; $\triangleright$ train the diffusion model
end
Obtain a diffusion model with $\mu_\theta$ and $\Sigma_\theta$;
for $i = 1$ to $N$ do
    $\epsilon \sim \mathcal{N}(0, I)$;
    Noised image $x_{t^*} = \sqrt{\alpha(t^*)} x_{p_i} + \sqrt{1 - \alpha(t^*)} \epsilon$;
    for $t = t^*$ to 1 do
        if $t &gt; 1$ then
            $z \sim \mathcal{N}(0, I)$;
        end
        else
            $z = 0$;
        end
        $x_{t-1} = \mu_\theta(x_t, t) + \Sigma_\theta(x_t, t) \mathbf{z}$; $\triangleright$ image denoising
    end
    Receive a largely purified image $x_{e_i}$;
    $x_{f_i} = C(x_{e_i}; p, \sigma)$;
end
Return: Final dataset $\{(x_{f_i}, y_i) \mid i = 1, 2, ..., N\}$.
</div>

Lightweight corruption compensation module. Owing to variations in the assimilation effects induced by different poison patterns, certain poisons require a greater amount of noise for effective assimilation. Specifically, we observe that low-frequency poisons $( e . g .$ , EM, REM, and LSP) and robust high-frequency poisons $( e . g . ,   \mathrm { S E P } )$ are assimilated more slowly as suggested in Tab. 4 and analyzed in Sec. 4.6. To address this, we propose a lightweight corruption compensation module to expunge these residual poison perturbations while ensuring that im age features are not excessively harmed. Since the low-frequency poison operates in color-sensitive regions of the image, we utilize the probabilistic grayscale transformation to remove residual low-frequency poisons. In addition, we first propose the lightweight Gaussian noise to eliminate robust high-frequency poison, i. $, e . ,$ SEP (see Sec. 4.7). The two-stage lightweight corruption techniques are both capable of effectively expunging residual poisons while ensuring minimal impact on image features, which can be formally defined as:

$$
x _ {f} = C (x _ {e}; p, \sigma) = \left\{ \begin{array}{l l} G (x _ {e}) + \varepsilon & \text {with probability} p \\ x _ {e} + \varepsilon & \text {with probability} 1 - p \end{array} \right. \tag {13}
$$

where $x _ { f }$ represents the final processed image, $x _ { e }$ denotes the purified image, $\varepsilon   \sim   \mathcal { N } ( \dot { 0 } , \sigma ^ { 2 } )$ , and G represents the grayscale transformation function. Please refer to the Algorithm 1 for the detailed process of ECLIPSE.

## 4 Experiments

## 4.1 Experimental Settings

Implementation details. The forward timestep $t ^ { * }$ is 100, M is 4, training iteration I is 250K, grayscale probability p is 0.4, and standard deviation σ is 0.05 unless otherwise stated. We use 4% images with the same distribution as the training set of CIFAR-10 [20], 1.5% images with the same distribution as the training set of ImageNet [4] to serve as sparse sets. Diverse network structures including ResNet [12], VGG [35], and DenseNet [16] are selected. We use SGD for training with a momentum of 0.9, a learning rate of 0.1, and a batch size of 128 for 80 epochs.

<table><tr><td colspan="2">Models  $\rightarrow$ </td><td colspan="10">ResNet18 [12]</td><td colspan="9">VGG19 [35]</td></tr><tr><td colspan="2">Defenses↓ Attacks→</td><td>EM</td><td>TAP</td><td>REM</td><td>SEP</td><td>EFP</td><td>URP</td><td>LSP</td><td>AR</td><td>ADP</td><td>AVG</td><td>EM</td><td>TAP</td><td>REM</td><td>EFP</td><td>URP</td><td>LSP</td><td>AR</td><td>ADP</td><td>AVG</td></tr><tr><td rowspan="5">Invalid defenses</td><td>w/o</td><td>17.58</td><td>26.16</td><td>27.34</td><td>9.01</td><td>86.64</td><td>16.80</td><td>24.48</td><td>10.59</td><td>25.93</td><td>27.17</td><td>19.90</td><td>27.81</td><td>31.22</td><td>81.83</td><td>16.53</td><td>21.80</td><td>13.94</td><td>24.02</td><td>29.63</td></tr><tr><td>Cutout [5]</td><td>17.63</td><td>29.16</td><td>22.42</td><td>9.30</td><td>87.76</td><td>84.41</td><td>22.93</td><td>12.92</td><td>21.39</td><td>34.21</td><td>39.51</td><td>31.69</td><td>22.87</td><td>85.06</td><td>29.20</td><td>25.92</td><td>10.44</td><td>18.38</td><td>32.88</td></tr><tr><td>Mixup [45]</td><td>30.74</td><td>24.36</td><td>29.99</td><td>8.35</td><td>88.76</td><td>17.29</td><td>23.48</td><td>11.46</td><td>31.20</td><td>29.51</td><td>22.75</td><td>28.38</td><td>33.30</td><td>82.49</td><td>16.32</td><td>24.59</td><td>14.47</td><td>37.14</td><td>32.43</td></tr><tr><td>Cutmix [44]</td><td>29.73</td><td>23.70</td><td>29.42</td><td>6.66</td><td>87.74</td><td>84.24</td><td>20.86</td><td>14.56</td><td>23.14</td><td>35.56</td><td>29.85</td><td>25.09</td><td>30.26</td><td>81.71</td><td>10.14</td><td>25.27</td><td>15.71</td><td>24.16</td><td>30.27</td></tr><tr><td>DP-SGD [14]</td><td>18.17</td><td>30.96</td><td>25.92</td><td>8.24</td><td>87.98</td><td>20.49</td><td>22.11</td><td>10.19</td><td>21.51</td><td>27.29</td><td>20.45</td><td>27.09</td><td>24.52</td><td>84.23</td><td>30.77</td><td>25.73</td><td>14.33</td><td>27.66</td><td>31.85</td></tr><tr><td rowspan="4">Limited validity</td><td>ISS-G [22]</td><td>88.42</td><td>21.88</td><td>65.29</td><td>7.57</td><td>86.54</td><td>60.64</td><td>65.89</td><td>38.64</td><td>34.42</td><td>52.14</td><td>86.47</td><td>27.57</td><td>71.16</td><td>83.23</td><td>60.84</td><td>80.91</td><td>39.60</td><td>41.60</td><td>61.42</td></tr><tr><td>AA [27]</td><td>85.30</td><td>67.12</td><td>39.73</td><td>24.94</td><td>87.76</td><td>90.81</td><td>87.38</td><td>51.19</td><td>58.22</td><td>65.83</td><td>78.99</td><td>56.81</td><td>10.00</td><td>78.71</td><td>82.73</td><td>9.99</td><td>25.32</td><td>24.35</td><td>45.86</td></tr><tr><td>OP [31]</td><td>65.42</td><td>45.86</td><td>30.44</td><td>10.01</td><td>82.64</td><td>89.28</td><td>90.14</td><td>33.60</td><td>33.80</td><td>53.47</td><td>79.79</td><td>54.09</td><td>31.88</td><td>78.65</td><td>87.14</td><td>87.43</td><td>13.64</td><td>29.47</td><td>57.76</td></tr><tr><td>AVATAR [6]</td><td>27.45</td><td>86.63</td><td>35.74</td><td>44.97</td><td>75.90</td><td>86.86</td><td>39.93</td><td>83.98</td><td>67.95</td><td>61.05</td><td>34.92</td><td>83.63</td><td>39.41</td><td>74.57</td><td>84.08</td><td>53.96</td><td>83.49</td><td>65.22</td><td>64.91</td></tr><tr><td rowspan="3">Qualified defenses</td><td>AT [38]</td><td>68.31</td><td>82.46</td><td>60.80</td><td>63.23</td><td>71.46</td><td>85.63</td><td>81.94</td><td>84.04</td><td>82.76</td><td>75.63</td><td>64.31</td><td>81.33</td><td>63.28</td><td>66.63</td><td>83.22</td><td>79.15</td><td>80.69</td><td>80.58</td><td>74.90</td></tr><tr><td>ISS-J [22]</td><td>78.35</td><td>80.77</td><td>81.54</td><td>80.93</td><td>70.54</td><td>81.27</td><td>79.55</td><td>81.39</td><td>80.98</td><td>79.48</td><td>78.69</td><td>80.93</td><td>79.16</td><td>68.75</td><td>80.74</td><td>78.49</td><td>81.56</td><td>78.46</td><td>78.35</td></tr><tr><td>ECLIPSE (Ours)</td><td>82.80</td><td>86.13</td><td>82.72</td><td>82.85</td><td>77.20</td><td>86.98</td><td>84.58</td><td>87.32</td><td>82.62</td><td>83.69</td><td>80.73</td><td>84.83</td><td>79.90</td><td>75.87</td><td>85.86</td><td>83.48</td><td>85.84</td><td>80.11</td><td>82.08</td></tr></table>

Table 1: Main results: The test accuracy (%) results on CIFAR-10 with ResNet18 and VGG19. “AVG” denotes the average value of each row, “ADP” denotes our proposed adaptive attack against ECLIPSE. The bold values denote the best defense effect among the qualified defense schemes.

## 4.2 Evaluation of ECLIPSE

Comparison baselines. We compare with five SOTA defenses, ISS [22], OP [31], AA [27], AVATAR [6], and AT [38]. Besides, other common defenses such as DP-SGD [14,46], cutmix [44], mixup [45], and cutout [5] are tested.

Evaluation metrics. Two evaluation metrics are used to evaluate these defense schemes: (i) test accuracy, i.e., the accuracy of the model obtained after ap plying the defense against CLBPAs on clean test set, and (ii) clean accuracy, i.e., the accuracy of the model obtained after applying the defense against the clean training set on clean test set.

<table><tr><td rowspan="2">ArchitecturesDefenses↓ Poisons→</td><td colspan="5">ResNet18</td><td colspan="5">DenseNet121</td></tr><tr><td>TAP</td><td>URP</td><td>AR</td><td>CLEAN</td><td>AVG</td><td>TAP</td><td>URP</td><td>AR</td><td>CLEAN</td><td>AVG</td></tr><tr><td>w/o</td><td>40.8</td><td>51.3</td><td>25.7</td><td>72.1</td><td>47.5</td><td>46.2</td><td>69.5</td><td>22.4</td><td>77.8</td><td>54.0</td></tr><tr><td>ISS-G</td><td>28.5</td><td>27.8</td><td>19.5</td><td>56.9</td><td>33.2</td><td>31.2</td><td>40.5</td><td>23.7</td><td>62.7</td><td>39.5</td></tr><tr><td>AVATAR</td><td>52.3</td><td>65.3</td><td>54.0</td><td>72.3</td><td>61.0</td><td>57.0</td><td>68.1</td><td>48.9</td><td>73.1</td><td>61.8</td></tr><tr><td>AT</td><td>58.2</td><td>61.9</td><td>47.3</td><td>66.8</td><td>58.5</td><td>61.8</td><td>63.1</td><td>41.9</td><td>71.9</td><td>59.7</td></tr><tr><td>ISS-J</td><td>61.2</td><td>60.8</td><td>59.4</td><td>66.2</td><td>61.9</td><td>61.6</td><td>60.8</td><td>62.0</td><td>68.6</td><td>63.3</td></tr><tr><td>ECLIPSE (Ours)</td><td>61.9</td><td>60.4</td><td>58.9</td><td>67.0</td><td>62.1</td><td>60.9</td><td>63.4</td><td>59.4</td><td>72.7</td><td>64.1</td></tr></table>

Table 2: The test accuracy (%) and clean accuracy (%) results on ImageNet dataset using ResNet18 and DenseNet121.

| Defense↓ Model→ | ResNet18 | ResNet50 | VGG16 | VGG19 | DenseNet121 | AVG |
| --- | --- | --- | --- | --- | --- | --- |
| w/o | 94.95 | 94.53 | 93.27 | 93.04 | 93.91 | 93.94 |
| AT | 89.57 | 89.88 | 88.04 | 86.93 | 89.11 | 88.71 |
| ISS-J | 85.23 | 85.85 | 84.42 | 84.26 | 85.08 | 84.97 |
| ECLIPSE (Ours) | 90.43 | 90.17 | 88.38 | 88.58 | 89.35 | 89.38 |

Table 3: The clean accuracy (%) results on CIFAR-10 dataset with SOTA defense schemes across diverse models.

Main results. The values of average test accuracy that are similar between post-defense and undefended scenarios, are covered by gray demonstrated in Tab. 1. This indicates that cutout, mixup, cutmix, and DP-SGD are almost ineffective for CLBPAs. We also highlight the results with accuracy below 50% in light yellow to denote the unqualified defense and accuracy above 80% in light blue to indicate that the defense capability is considered excellent. Therefore, ISS-G, AA, OP, and AVATAR exhibit extreme limitations in countering various types of poisons as shown in Tab. 1, rendering them unsuitable as universal defense solutions. As also demonstrated in Tab. 1, two SOTA defense schemes AT and ISS-J, also lag behind ECLIPSE by more than 8% and 4% in average test accuracy, respectively. In addition, our defense also outperforms these two SOTA defense solutions on ImageNet as shown in Tab. 2.

Given that only AT and ISS-J achieve comparable defense performance in test accuracy, we further only compare the clean accuracy of these two defenses in Tab. 3. It can be seen that ECLIPSE has an absolute and significant advantage in this metric. Meanwhile, ISS-J causes a damage of approximately 9% on clean training, constituting a fatal flaw that compromises this approach (the values in Tabs. 2 and 3 covered by deep orange denote the optimal defense effect, while light orange denotes the suboptimal)

## 4.3 Purification Visual Effect

After undergoing the processes of poison absorption and noise denoising, the resulting image is essentially a purified image, as demonstrated in Fig. 4. It can be seen that the poisoned images clearly have their poison noise removed after passing through our sparse diffusion purification stage.

![](images/page_12_image_2.jpg)

<details>
<summary>text_image</summary>

Clean
Poisoned
Noised
Purified
EM TAP URP LSP AR
EM TAP URP LSP AR
</details>

Fig. 4: Visual presentations of five types of CLBPAs, including clean, poisoned, noised, and purified images.

| Module↓ Poison→ | EM | TAP | REM | SEP | EFP | URP | LSP | AR | AVG |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| A+B+C | 82.80 | 86.13 | 82.72 | 82.85 | 77.20 | 86.98 | 84.58 | 87.32 | 83.82 |
| A+B | 73.22 | 86.31 | 67.48 | 41.30 | 77.58 | 85.59 | 81.58 | 84.22 | 74.66 |
| A+C | 61.30 | 86.94 | 77.33 | 84.07 | 76.72 | 87.52 | 68.70 | 87.59 | 78.77 |
| B+C | 78.82 | 83.32 | 75.51 | 14.66 | 81.62 | 88.87 | 80.08 | 60.96 | 70.48 |
| A | 27.45 | 86.63 | 35.74 | 44.97 | 75.90 | 86.86 | 39.93 | 83.98 | 60.18 |
| B | 78.69 | 30.95 | 67.05 | 7.53 | 86.14 | 58.70 | 68.19 | 37.63 | 54.36 |
| C | 20.43 | 84.90 | 28.83 | 11.93 | 79.06 | 89.54 | 27.11 | 33.14 | 46.87 |

Table 4: The test accuracy (%) results on CIFAR-10 using ResNet18 with diverse combinations. $\text{" A}  ,  \text{" B}  ,  \text{" C}  \text{" }$ denote diffusion purification, grayscale module, and Gaussian noise module. The gray line denotes the best effect in this paper.

## 4.4 Resistance to Potential Adaptive Attacks

We assume the attacker has knowledge of the structure of the diffusion model and compensation module, and then design an adaptive attack against ECLIPSE, termed as ADP, which involves solving the following optimization objective:

$$
\arg \min _ {\theta} \underset {(x, y) \sim \mathcal {D} _ {c}} {\mathbb {E}} \left[ \min _ {\boldsymbol {\delta} _ {\boldsymbol {a}}} \mathcal {L} \left(U (C (x + \boldsymbol {\delta} _ {\boldsymbol {a}}); \theta), y\right) \right]
$$

where $\| \delta _ { a } \| _ { \infty } \leq \epsilon$ is the adaptive poison with $\ell _ { \infty } \mathrm { { - n o r m } }$ bounded, U denotes the U-Net [52] that forms the diffusion model, which has been slightly modified into a classification network (the final layer is replaced by a convolutional layer with global average pooling), and C is the corruption function in Eq. (13). The defense performance against $\mathrm { A D P }$ is reported in Tab. 1. Our ECLIPSE demonstrates excellent performance in defending against ADP, indicating the robustness of our defense scheme ECLIPSE when facing the adaptive attack.

| M↓ Poisson→ | EM | TAP | REM | SEP | EFP | URP | LSP | AR | AVG |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 0 | 78.30 | 86.59 | 81.56 | 77.97 | 78.60 | 87.52 | 83.53 | 87.56 | 82.70 |
| 2 | 79.72 | 86.88 | 82.05 | 81.29 | 78.02 | 86.77 | 84.50 | 86.98 | 83.28 |
| 4 | 82.80 | 86.13 | 82.72 | 82.85 | 77.20 | 86.98 | 84.58 | 87.32 | 83.82 |
| 6 | 80.75 | 86.79 | 82.68 | 82.71 | 78.41 | 86.52 | 83.88 | 87.05 | 83.60 |

Table 5: The test accuracy (%) results on CIFAR-10 using ResNet18 with varying replication times M from ECLIPSE.

![](images/page_13_chart_4.jpg)

<details>
<summary>bar</summary>

| Training iteration (l) | SEP (%) | LSP (%) | AR (%) |
| --- | --- | --- | --- |
| 150k | ~80 | ~85.6 | ~87.8 |
| 200k | ~82.3 | ~85.4 | ~87.7 |
| 250k | ~82.9 | ~84.6 | ~87.1 |
| 300k | ~83.0 | ~85.0 | ~87.2 |
</details>

(a) p=0.4, σ=0.05

![](images/page_13_chart_6.jpg)

<details>
<summary>bar</summary>

| Grayscale probability (p) | SEP (%) | LSP (%) | AR (%) |
| --- | --- | --- | --- |
| 0.3 | ~83.5 | ~82.9 | ~84.6 |
| 0.4 | ~84.1 | ~84.3 | ~85.2 |
| 0.5 | ~82.6 | ~83.0 | ~84.6 |
| 0.6 | ~81.7 | ~82.3 | ~84.6 |
| 0.7 | ~82.1 | ~82.3 | ~84.2 |
</details>

(b) σ=0.1, I=200K

![](images/page_13_chart_8.jpg)

<details>
<summary>bar</summary>

| Standard deviation \((\sigma)\) | SEP (%) | LSP (%) | AR (%) |
| --- | --- | --- | --- |
| 0.025 | ~65 | ~84.5 | ~85.5 |
| 0.05 | ~82 | ~85 | ~87.5 |
| 0.075 | ~84.5 | ~84 | ~86 |
| 0.1 | ~84 | ~84 | ~85 |
| 0.15 | ~81 | ~81 | ~82 |
</details>

(c) p=0.4, I=200K

Fig. 5: The test accuracy (%) results of ECLIPSE on three poisoned CIFAR-10 dataset using ResNet18 with varying hyper-parameters.

## 4.5 Hyper-Parameter Analysis

We investigate the effects of hyperparameters $I ,   p ,   \sigma ,   M ,   B / N$ , and t<sup>∗</sup> on the ECLIPSE defense performance. It can be observed from Fig. 5 (a) that different values of I had little effect on the final defense performance, which can be attributed to the relatively small size of the training set, allowing the diffusion model to converge well within the above range of I. The impact of different probabilities p of Module B on ECLIPSE can be found in Fig. 5 (b). The excessively low probability of grayscale transformation may lead to incomplete removal of low-frequency poisons, whereas an excessively high probability may compromise certain image features. The impact of different standard deviations σ on ECLIPSE is provided in Fig. 5 (c). Both excessively large and excessively small Gaussian noise lead to a decline in the final defensive performance. In addition, we explore the impact from the replication times M in Tab. 5, the ratio of sparse set $( B / N )$ in Tab. 6, and the forward timestep $t ^ { * }$ in Tab. 7. We can see that M mainly influences the defense effect against EM and SEP, which is improved by up to 4.5% and 4.88% compared to not using dataset enlarging module. The potential reason for this improvement is that expanding the total amount of data might help the diffusion model better learn the existing distribution of clean data. The proportion of the sparse set $( B / N )$ does not significantly influence the defense effect. Setting the value of $t ^ { * }$ too large or too small will both lead to a decrease in defense effect.

## 4.6 Ablation Study

Our defense can be divided into three modules, diffusion purification, probabilistic grayscale transformation, and adding Gaussian noise.

| B/N ↓ Poison→ | EM | TAP | REM | SEP | EFP | URP | LSP | AR | AVG |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 2% | 80.25 | 84.28 | 82.45 | 82.39 | 73.72 | 84.70 | 82.04 | 85.08 | 81.86 |
| 4% | 82.80 | 86.13 | 82.72 | 82.85 | 77.20 | 86.98 | 84.58 | 87.32 | 83.82 |
| 6% | 80.07 | 87.05 | 79.61 | 78.51 | 79.82 | 87.54 | 84.77 | 88.25 | 83.20 |
| 8% | 78.13 | 86.75 | 77.65 | 65.51 | 80.39 | 88.26 | 81.82 | 87.48 | 80.75 |

Table 6: The test accuracy (%) results on CIFAR-10 using ResNet18 with varying dataset ratios B/N (%) from ECLIPSE.

| $t^{*} \downarrow$Poison→ | EM | TAP | REM | SEP | EFP | URP | LSP | AR | AVG |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 50 | 77.23 | 87.66 | 78.20 | 55.07 | 81.33 | 88.43 | 81.97 | 88.27 | 79.77 |
| 100 | 82.80 | 86.13 | 82.72 | 82.85 | 77.20 | 86.98 | 84.58 | 87.32 | 83.82 |
| 150 | 80.91 | 84.30 | 82.56 | 83.99 | 74.23 | 83.25 | 79.46 | 84.37 | 81.63 |

Table 7: The test accuracy (%) results on CIFAR-10 using ResNet18 with varying forward timestep t<sup>∗</sup>from ECLIPSE.

Diffusion purification (A). The results obtained using Module B+C in Tab. 4 indicate the absence of Module A leads to a significant decrease in defense performance against SEP and AR, with a 13.34% average performance drop. This strongly demonstrates the importance of incorporating Module A.

Probabilistic grayscale transformation (B). The absence of Module B results in varying degrees of negative effects on low-frequency poisons as shown in Tab. 4 using Module A+C. This effectiveness of Module B against these poisons is attributed to the fact that low-frequency noise typically affects the color channel information of the images, while grayscale transformation can mask color information and thus suppress low-frequency poisons. As for the reason Module A and Module C are less effective against low-frequency poisons is they rely on adding high-frequency Gaussian noise, thus absorbing low-frequency poisons at a slower rate.

Addition of Gaussian noise (C). The absence of Module C results in a significant decline of 41.55% against SEP when using Module A+B as shown in Tab. 4. To analyze this, we visualize the remaining SEP perturbations after passing through Module A in Fig. 6. Surprisingly, the remaining perturbations are highly similar to the clean samples’ features. These high-frequency residual poisons with significant feature similarity can be easily learned by DNNs as shortcuts [11,15,21], leading to a decline in test accuracy. Consequently, by introducing Module C that adds random noise, these relatively fragile high-frequency poisons can be effectively disturbed.

## 4.7 Analysis of ECLIPSE

We categorize existing CLBPAs into three types: low-frequency poisons $( i . e . ,$ EM, REM, EFP, and LSP), robust high-frequency poisons (i.e., SEP), and fragile high-frequency poisons (i.e., TAP, URP, and AR). We conclude that, (i) Module A can remove high-frequency poisons. The effectiveness of Module A against fragile high-frequency poisons can be verified in Tab. 4. In addition, by increasing t<sup>∗</sup>, Module A can effectively remove robust high-frequency poisons as shown in Tab. 7, (ii) Module B is crucial for addressing low-frequency poisons. It is evident from Tab. 4 that as long as Module B is present (i.e., A+B+C, A+B, B+C, and B), the defense against low-frequency poisons is highly effective, and (iii) Module C has a positive impact in expunging low-frequency, robust highfrequency, and fragile high-frequency poisons. The incremental effect brought by Module C can be observed by comparing the results before and after its inclusion as demonstrated in Tab. 4. Thus, lightweight Gaussian noise is indeed beneficial for expunging CLBPA poisons.

![](images/page_15_image_2.jpg)

<details>
<summary>text_image</summary>

Clean
images
Residual
poisons
</details>

Fig. 6: The residual poisons refer to difference between the SEP images after Module A and the corresponding clean images, which shows that the images in two rows exhibit a high feature similarity.

## 5 Conclusion and Limitation

We propose a brand-new defense scheme called ECLIPSE against the recent rise of clean-label indiscriminate poisoning attacks, which is more universally effective, more practical, and robust. Our scheme is capable of expunging diverse invisible poisons of images via a purification process by a sparse data trained diffusion model and a compensation module. Extensive experiments on benchmark datasets verify that our scheme enjoys high defense effectiveness and robustness against adaptive attack. However, ECLIPSE is not that effective when defending against clean-label indiscriminate poisoning attacks based on convolution-based perturbations [29,39] without norm restriction. This is because the purification scheme based on diffusion models can only remove perturbations with norm constraints [26]. We will leave this issue to our future work.

## Acknowledgements

This work is supported by the National Natural Science Foundation of China (Grant No.U20A20177) and Hubei Province Key R&D Technology Special Innovation Project (Grant No.2021BAA032). Shengshan Hu and Peng Xu are co-corresponding authors.

## Appendix

Proof for Theorem 1: Based on the continuous-time forward process defined as the solution to the SDE [36], we have:

$$
\mathrm{d} x = g (t) \mathrm{d} \beta + h (x, t) \mathrm{d} t \tag {14}
$$

where $g ( t )$ is the diffusion coefficient, $h ( x , t )$ is the drift coefficient, and $\beta ( t )$ is a Brownian motion with a diffusion matrix. After this, according to the Fokker-Planck-Kolmogorov equation [33], we have:

$$
\begin{array}{l} \left| \frac {\partial p (x , t)}{\partial t} = - \nabla_ {x} \left(h (x, t) p (x, t) - \frac {g ^ {2} (t)}{2} \nabla_ {x} p (x, t)\right) \right| \\ = - \nabla_ {x} \left(h (x, t) p (x, t) - \frac {g ^ {2} (t)}{2} p (x, t) \nabla_ {x} \log p (x, t)\right) \\ = \nabla_ {x} \left(k _ {p} (x, t) p (x, t)\right) \tag {15} \\ \end{array}
$$

where $k_p(x,t)$ is defined as $-h(x,t)+\frac{g^2(t)}{2}\nabla_x\log p(x,t)$. Then we have:

$$
\begin{array}{l} \frac {\partial D _ {K L} (p (x , t) \| q (x , t))}{\partial t} = \frac {\partial}{\partial t} \int p (x, t) \log \frac {p (x , t)}{q (x , t)} d x \\ = \int \frac {\partial p (x , t)}{\partial t} \log \frac {p (x , t)}{q (x , t)} d x + \int \frac {p (x , t)}{q (x , t)} \frac {\partial q (x , t)}{\partial t} d x + \int \frac {\partial p (x , t)}{\partial t} d x \\ = \int \nabla_ {x} \left(k _ {p} (x, t) p (x, t)\right) \log \frac {p (x , t)}{q (x , t)} d x + \int \nabla_ {x} \left(k _ {q} (x, t) q (x, t)\right) \frac {p (x , t)}{q (x , t)} d x \\ = - \int p (x, t) \left[ k _ {p} (x, t) - k _ {q} (x, t) \right] ^ {T} \left[ \nabla_ {x} \log p (x, t) - \nabla_ {x} \log q (x, t) \right] d x \\ = - \frac {g ^ {2} (t)}{2} \int p (x, t) \| \nabla_ {x} \log p (x, t) - \nabla_ {x} \log q (x, t) \| _ {2} ^ {2} d x \\ = - \frac {g ^ {2} (t)}{2} D _ {F} (p (x, t) | | q (x, t)) \tag {16} \\ \end{array}
$$

where the fourth equality follows from the integration by parts and our assumption of smooth and fast-decaying $p ( x , t )$ and $q ( x , t )$ . Here, $D _ { F }$ denotes the Fisher divergence [19]. Since $g ^ { 2 } ( t ) > 0$ and the Fisher divergence is non-negative, we have:

$$
\frac {\partial D _ {K L} (p (x , t) \| q (x , t))}{\partial t} \leq 0 \tag {17}
$$

where equality holds only if $p ( x , t ) = q ( x , t )$

## References

1. Biggio, B., Nelson, B., Laskov, P.: Support vector machines under adversarial label noise. In: Proceedings of the 3rd Asian Conference on Machine Learning (ACML’11). pp. 97–112 (2011)

2. Borgnia, E., Cherepanova, V., Fowl, L., Ghiasi, A., Geiping, J., Goldblum, M., Goldstein, T., Gupta, A.: Strong data augmentation sanitizes poisoning and backdoor attacks without an accuracy tradeoff. In: Proceedings of the 2021 IEEE International Conference on Acoustics, Speech and Signal Processing (ICASSP’21). pp. 3855–3859 (2021)
3. Chen, S., Yuan, G., Cheng, X., Gong, Y., Qin, M., Wang, Y., Huang, X.: Self-ensemble protection: Training checkpoints are good data protectors. In: Proceedings of the 11th International Conference on Learning Representations (ICLR’23) (2023)
4. Deng, J., Dong, W., Socher, R., Li, L.J., Li, K., Feifei, L.: Imagenet: A large-scale hierarchical image database. In: Proceedings of the 2009 IEEE/CVF Conference on Computer Vision and Pattern Recognition (CVPR’09). pp. 248–255 (2009)
5. DeVries, T., Taylor, G.W.: Improved regularization of convolutional neural networks with cutout. arXiv preprint arXiv:1708.04552 (2017)
6. Dolatabadi, H.M., Erfani, S., Leckie, C.: The devil’s advocate: Shattering the illusion of unexploitable data using diffusion models. arXiv preprint arXiv:2303.08500 (2023)
7. Feng, J., Cai, Q.Z., Zhou, Z.H.: Learning to confuse: Generating training time adversarial data with auto-encoder. In: Proceedings of the 33rd Neural Information Processing Systems (NeruIPS’19). vol. 32, pp. 11971–11981 (2019)
8. Fowl, L., Chiang, P.y., Goldblum, M., Geiping, J., Bansal, A., Czaja, W., Goldstein, T.: Preventing unauthorized use of proprietary data: Poisoning for secure dataset release. arXiv preprint arXiv:2103.02683 (2021)
9. Fowl, L., Goldblum, M., Chiang, P.y., Geiping, J., Czaja, W., Goldstein, T.: Adversarial examples make strong poisons. In: Proceedings of the 35th Neural Information Processing Systems (NeurIPS’21). vol. 34, pp. 30339–30351 (2021)
10. Fu, S., He, F., Liu, Y., Shen, L., Tao, D.: Robust unlearnable examples: Protecting data privacy against adversarial learning. In: Proceedings of the 10th International Conference on Learning Representations (ICLR’22) (2022)
11. Geirhos, R., Jacobsen, J.H., Michaelis, C., Zemel, R., Brendel, W., Bethge, M., Wichmann, F.A.: Shortcut learning in deep neural networks. Nature Machine Intelligence 2, 665–673 (2020)
12. He, K., Zhang, X., Ren, S., Sun, J.: Deep residual learning for image recognition. In: Proceedings of the 2016 IEEE/CVF Conference on Computer Vision and Pattern Recognition (CVPR’16). pp. 770–778 (2016)
13. Ho, J., Jain, A., Abbeel, P.: Denoising diffusion probabilistic models. In: Proceedings of the 34th Neural Information Processing Systems (NeurIPS’20). vol. 33, pp. 6840–6851 (2020)
14. Hong, S., Chandrasekaran, V., Kaya, Y., Dumitraş, T., Papernot, N.: On the effectiveness of mitigating data poisoning attacks with gradient shaping. arXiv preprint arXiv:2002.11497 (2020)
15. Hu, S., Liu, W., Li, M., Zhang, Y., Liu, X., Wang, X., Zhang, L.Y., Hou, J.: Pointcrt: Detecting backdoor in 3d point cloud via corruption robustness. In: Proceedings of the 31st ACM International Conference on Multimedia (MM’23). pp. 666–675 (2023)
16. Huang, G., Liu, Z., Van Der Maaten, L., Weinberger, K.Q.: Densely connected convolutional networks. In: Proceedings of the 2017 IEEE/CVF Conference on Computer Vision and Pattern Recognition (CVPR’17). pp. 4700–4708 (2017)
17. Huang, H., Ma, X., Erfani, S.M., Bailey, J., Wang, Y.: Unlearnable examples: Making personal data unexploitable. In: Proceedings of the 9th International Conference on Learning Representations (ICLR’21) (2021)
18. Jiang, W., Diao, Y., Wang, H., Sun, J., Wang, M., Hong, R.: Unlearnable examples give a false sense of security: Piercing through unexploitable data with learnable examples. In: Proceedings of the 31st ACM International Conference on Multimedia (MM’23). p. 8910–8921 (2023)
19. Kostrikov, I., Fergus, R., Tompson, J., Nachum, O.: Offline reinforcement learning with fisher divergence critic regularization. In: Proceedings of the 38th International Conference on Machine Learning (ICML’21). pp. 5774–5783 (2021)
20. Krizhevsky, A., Hinton, G.: Learning multiple layers of features from tiny images (2009)
21. Liu, X., Li, M., Wang, H., Hu, S., Ye, D., Jin, H., Wu, L., Xiao, C.: Detecting backdoors during the inference stage based on corruption robustness consistency. In: Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition (CVPR’23). pp. 16363–16372 (2023)
22. Liu, Z., Zhao, Z., Larson, M.: Image shortcut squeezing: Countering perturbative availability poisons with compression. In: Proceedings of the 40th International Conference on Machine Learning (ICML’23) (2023)
23. Madry, A., Makelov, A., Schmidt, L., Tsipras, D., Vladu, A.: Towards deep learning models resistant to adversarial attacks. In: Proceedings of the 6th International Conference on Learning Representations (ICLR’18) (2018)
24. Muñoz-González, L., Biggio, B., Demontis, A., Paudice, A., Wongrassamee, V., Lupu, E.C., Roli, F.: Towards poisoning of deep learning algorithms with backgradient optimization. In: Proceedings of the 10th ACM Workshop on Artificial Intelligence and Security (AISec’17). pp. 27–38 (2017)
25. Nichol, A.Q., Dhariwal, P.: Improved denoising diffusion probabilistic models. In: Proceedings of the 38th International Conference on Machine Learning (ICML’21). pp. 8162–8171 (2021)
26. Nie, W., Guo, B., Huang, Y., Xiao, C., Vahdat, A., Anandkumar, A.: Diffusion models for adversarial purification. In: Proceedings of the 39th International Conference on Machine Learning (ICML’22) (2022)
27. Qin, T., Gao, X., Zhao, J., Ye, K., Xu, C.Z.: Learning the unlearnable: Adversarial augmentations suppress unlearnable example attacks. arXiv preprint arXiv:2303.15127 (2023)
28. Ren, J., Xu, H., Wan, Y., Ma, X., Sun, L., Tang, J.: Transferable unlearnable examples. In: Proceedings of the 11th International Conference on Learning Representations (ICLR’23) (2023)
29. Sadasivan, V.S., Soltanolkotabi, M., Feizi, S.: Cuda: Convolution-based unlearnable datasets. In: Proceedings of the 2023 IEEE/CVF Conference on Computer Vision and Pattern Recognition (CVPR’23). pp. 3862–3871 (2023)
30. Sandoval-Segura, P., Singla, V., Fowl, L., Geiping, J., Goldblum, M., Jacobs, D., Goldstein, T.: Poisons that are learned faster are more effective. In: Proceedings of the 2022 IEEE/CVF Conference on Computer Vision and Pattern Recognition Workshops (CVPRW’22). pp. 198–205 (2022)
31. Sandoval-Segura, P., Singla, V., Geiping, J., Goldblum, M., Goldstein, T.: What can we learn from unlearnable datasets? In: Proceedings of the 37th Neural Information Processing Systems (NeurIPS’23) (2023)
32. Sandoval-Segura, P., Singla, V., Geiping, J., Goldblum, M., Goldstein, T., Jacobs, D.W.: Autoregressive perturbations for data poisoning. In: Proceedings of the 36th Neural Information Processing Systems (NeurIPS’22). vol. 35 (2022)
33. Särkkä, S., Solin, A.: Applied stochastic differential equations, vol. 10. Cambridge University Press (2019)
34. Shen, J., Zhu, X., Ma, D.: Tensorclog: An imperceptible poisoning attack on deep neural network applications. IEEE Access 7, 41498–41506 (2019)
35. Simonyan, K., Zisserman, A.: Very deep convolutional networks for large-scale image recognition. arXiv preprint arXiv:1409.1556 (2014)
36. Song, Y., Durkan, C., Murray, I., Ermon, S.: Maximum likelihood training of scorebased diffusion models. In: Proceedings of the 35th Neural Information Processing Systems (NeurIPS’21). vol. 34, pp. 1415–1428 (2021)
37. Song, Y., Sohl-Dickstein, J., P Kingma, D., Kumar, A., Ermon, S., Poole, B.: Scorebased generative modeling through stochastic differential equations. In: Proceedings of the 9th International Conference on Learning Representations (ICLR’21) (2021)
38. Tao, L., Feng, L., Yi, J., Huang, S.J., Chen, S.: Better safe than sorry: Preventing delusive adversaries with adversarial training. In: Proceedings of the 35th Neural Information Processing Systems (NeurIPS’21). vol. 34, pp. 16209–16225 (2021)
39. Wang, X., Hu, S., Li, M., Yu, Z., Zhou, Z., Zhang, L.Y.: Corrupting convolutionbased unlearnable datasets with pixel-based image transformations. arXiv preprint arXiv:2311.18403 (2023)
40. Wang, Z., Wang, Y., Wang, Y.: Fooling adversarial training with inducing noise. arXiv preprint arXiv:2111.10130 (2021)
41. Wen, R., Zhao, Z., Liu, Z., Backes, M., Wang, T., Zhang, Y.: Is adversarial training really a silver bullet for mitigating data poisoning? In: Proceedings of the 11th International Conference on Learning Representations (ICLR’23) (2023)
42. Yu, D., Zhang, H., Chen, W., Yin, J., Liu, T.Y.: Availability attacks create shortcuts. In: Proceedings of the 28th ACM SIGKDD Conference on Knowledge Discovery and Data Mining (KDD’22). pp. 2367–2376 (2022)
43. Yuan, C.H., Wu, S.H.: Neural tangent generalization attacks. In: Proceedings of the 38th International Conference on Machine Learning (ICML’21). pp. 12230–12240 (2021)
44. Yun, S., Han, D., Oh, S.J., Chun, S., Choe, J., Yoo, Y.: Cutmix: Regularization strategy to train strong classifiers with localizable features. In: Proceedings of the 17th International Conference on Computer Vision (ICCV’19) (2019)
45. Zhang, H., Cisse, M., Dauphin, Y.N., Lopez-Paz, D.: Mixup: Beyond empirical risk minimization. In: Proceedings of the 6th International Conference on Learning Representations (ICLR’18) (2018)
46. Zhang, L., Shen, B., Barnawi, A., Xi, S., Kumar, N., Wu, Y.: Feddpgan: Federated differentially private generative adversarial networks framework for the detection of covid-19 pneumonia. Information Systems Frontiers 23(6), 1403–1415 (2021)
47. Zhang, R., Zhu, Q.: A game-theoretic analysis of label flipping attacks on distributed support vector machines. In: Proceedings of the 51st Annual Conference on Information Sciences and Systems (CISS’17). pp. 1–6 (2017)
48. Zhang, Y., Hu, S., Zhang, L.Y., Shi, J., Li, M., Liu, X., Wan, W., Jin, H.: Why does little robustness help? a further step towards understanding adversarial transferability. In: Proceedings of the 45th IEEE Symposium on Security and Privacy (S&P’24). vol. 2 (2024)
49. Zhou, Z., Hu, S., Li, M., Zhang, H., Zhang, Y., Jin, H.: Advclip: Downstreamagnostic adversarial examples in multimodal contrastive learning. In: Proceedings of the 31st ACM International Conference on Multimedia (MM’23). pp. 6311–6320 (2023)
50. Zhou, Z., Hu, S., Zhao, R., Wang, Q., Zhang, L.Y., Hou, J., Jin, H.: Downstreamagnostic adversarial examples. In: Proceedings of the IEEE/CVF International Conference on Computer Vision (ICCV’23). pp. 4345–4355 (2023)
51. Zhou, Z., Li, M., Liu, W., Hu, S., Zhang, Y., Wan, W., Xue, L., Zhang, L.Y., Yao, D., Jin, H.: Securely fine-tuning pre-trained encoders against adversarial examples. In: Proceedings of the 2024 IEEE Symposium on Security and Privacy (SP’24) (2024)
52. Zhou, Z., Rahman Siddiquee, M.M., Tajbakhsh, N., Liang, J.: Unet++: A nested u-net architecture for medical image segmentation. In: Proceedings of the International Workshop on Deep Learning in Medical Image Analysis. pp. 3–11 (2018)


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

### Página 18

![Página 18](original-pages/page-18.png)

### Página 19

![Página 19](original-pages/page-19.png)

### Página 20

![Página 20](original-pages/page-20.png)

### Página 21

![Página 21](original-pages/page-21.png)

</details>
