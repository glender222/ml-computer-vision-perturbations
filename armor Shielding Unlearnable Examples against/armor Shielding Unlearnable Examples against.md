# ARMOR: Shielding Unlearnable Examples against Data Augmentation

Xueluan Gong, Member, IEEE, Yuji Wang, Yanjiao Chen, Senior Member, IEEE, Haocheng Dong, Yiming Li, Mengyuan Sun, Shuaike Li, Qian Wang, Fellow, IEEE, and Chen Chen

**Abstract—Private data, when published online, may be collected by unauthorized parties to train deep neural networks (DNNs). To protect privacy, defensive noises can be added to original samples to degrade their learnability by DNNs. Recently, unlearnable examples [19] are proposed to minimize the training loss such that the model learns almost nothing. However, raw data are often pre-processed before being used for training, which may restore the private information of protected data. In this paper, we reveal the data privacy violation induced by data augmentation, a commonly used data pre-processing technique to improve model generalization capability, which is the first of its kind as far as we are concerned. We demonstrate that data augmentation can significantly raise the accuracy of the model trained on unlearnable examples from 21.3% to 66.1%. To address this issue, we propose a defense framework, dubbed** ARMOR**, to protect data privacy from potential breaches of data augmentation. To overcome the difficulty of having no access to the model training process, we design a non-local moduleassisted surrogate model that better captures the effect of data augmentation. In addition, we design a surrogate augmentation selection strategy that maximizes distribution alignment between augmented and non-augmented samples, to choose the optimal augmentation strategy for each class. We also use a dynamic step size adjustment algorithm to enhance the defensive noise generation process. Extensive experiments are conducted on 4 datasets and 5 data augmentation methods to verify the performance of** ARMOR**. Comparisons with 6 state-of-the-art defense methods have demonstrated that** ARMOR **can preserve the unlearnability of protected private data under data augmentation.** ARMOR **reduces the test accuracy of the model trained on augmented protected samples by as much as 60% more than baselines. We also show that** ARMOR **is robust to adversarial training. We will open-source our codes upon publication.**

**Index Terms—Unlearnable examples, data augmentation, and data privacy preservation.**

## I. INTRODUCTION

HE success of deep learning models is, to a great extent, attributed to the availability of expansive data samples [31], [48], [41]. To train a well-performed DNN model, the model trainer may collect data samples from various sources. Although many public datasets can be easily downloaded

- X. Gong and C. Chen are with Nanyang Technological University, Singapore (E-mail: {xueluan.gong, chen.chen}@ntu.edu.sg).
- Y. Chen is with the College of Electrical Engineering, Zhejiang University, China (E-mail: chenyanjiao@zju.edu.cn).
- Y. Wang, H. Dong, M. Sun, S. Li, and Q. Wang are with the School of Cyber Science and Engineering, Wuhan University, China (E-mail:{yujiwang, hc dong, mengyuansun, shuaikeli, qianwang}@whu.edu.cn).
- Y. Li is with the ZJU-Hangzhou Global Scientific and Technological Innovation Center (HIC) and also with the State Key Laboratory of Blockchain and Data Security, Zhejiang University, China (e-mail: li-ym@zju.edu.cn).

online, private data samples are enticing thanks to their large quantity and diversity. For instance, an enormous number of facial images are published on social media and accessible to unauthorized parties to train well-performed face recognition DNN models. To protect personal data from being abused, worldwide regulations have been enacted, such as the European Union’s General Data Protection Regulation (GDPR) [45] and California Consumer Privacy Act (CCPA) [13].

To protect private data samples from being used as training data for DNN models, researchers have proposed to add defensive noises to original samples. The resulting protected samples can be published as they have little learnability but maintain the perceptual quality of original samples. To achieve this objective, early works borrow techniques from adversarial example attacks, creating imperceptible adversarial noises to mislead the training process of DNN models [29], [10], [50]. Recently, the concept of unlearnable examples has been proposed [19], [26]. The main rationale is to minimize the training loss such that little can be learned from protected samples. Uncovering that adversarial training undermines unlearnability, Fu et al. [12] proposed to generate robust unlearnable examples that minimize adversarial training loss.

In this paper, we present a new threat to data privacy posed by data augmentation, a commonly used data pre-processing technique [7], [36]. Data augmentation is usually performed on raw data samples to resolve class imbalance issues and improve the generalization capability of the trained model. As far as we are concerned, we are the first to unveil the potential data privacy threat induced by data augmentation. We demonstrate that state-of-the-art unlearnable examples (e.g., EMIN [19]) can effectively suppress the accuracy of trained models to no more than 15%. Unfortunately, if data augmentation is applied to unlearnable examples, the accuracy of trained models will be elevated to more than 60%, severely damaging the unlearnability of protected data.

To address this challenge, we develop ARMOR, a defense framework that can protect data privacy against advanced data augmentation. The main challenge is that the defender has no knowledge or control over the training process conducted by the attacker. More specifically, the defender does not know the model structure and the data augmentation strategy chosen by the attacker. To overcome this difficulty, we propose to use a carefully designed surrogate model and a surrogate augmentation selection strategy for generating defensive noises that minimize the training loss. For surrogate model construction, we design a non-local module that widens the receptive field of the surrogate model to capture more benefits from data augmentation. For surrogate augmentation strategy selection, we search for the best augmentation strategy to ensure that the augmented samples follow the distribution of the original samples. Under such strong surrogate models and surrogate strategies, the defensive noises are optimized via gradient descent. To further enhance the optimization process, we adopt a learning rate scheduling mechanism to dynamically adjust the learning rate based on the gradient norm.

We have conducted extensive experiments to evaluate the data protection performance of ARMOR. We compare ARMOR with 6 state-of-the-art defense methods, including EMAX [29], TAP [10], NTGA [50], EMIN [19], REM [12], and random noise, on four datasets, i.e., CIFAR-10, CIFAR-100, Mini-ImageNet, and VGG-Face. The experiment results confirm that ARMOR can reduce the test accuracy of the model trained on the augmented protected dataset by as much as 60% more than baselines. Furthermore, ARMOR showcases resilience against sophisticated data augmentation strategies, including Mixup [53], Feature distillation [27], PuzzleMix [21], FastAA [25], and DeepAA [54]. It is also verified that ARMOR is robust to adversarial training.

To conclude, we make the following contributions.

• We reveal the potential data privacy violation caused by data augmentation. To the best of our knowledge, we are the first to discover such a potential data privacy threat.
• We develop a defense framework that can effectively resist advanced data augmentation strategies. To render the defense effective in the black-box setting, we carefully build the surrogate model and select the surrogate augmentation to generate robust defensive noises.
• Extensive experiments show that ARMOR surpasses stateof-the-art data protection methods by significantly reducing model test accuracy under advanced data augmentation strategies. Moreover, ARMOR exhibits robustness against adversarial training and proves to be effective in generating class-wise defensive noises.

## II. PRELIMINARIES

## A. Data Privacy in Deep Learning

A deep neural network<sup>1</sup>(DNN) is a function $f _ { \Theta }$ parameterized by Θ, mapping an input x to the output y. The parameters Θ are optimized by minimizing the aggregated prediction error $( \mathbf { i . e . }$ , defined as the loss function $\mathcal { L } )$ on the training dataset $\mathcal { D } _ { t r a i n } = \{ \mathbf { x } _ { i } , \mathbf { y } _ { i } \} _ { i = 1 } ^ { N }$ as

$$
\min _ {\Theta} \frac {1}{N} \sum_ {i = 1} ^ {N} \mathcal {L} \left(f _ {\Theta} \left(\mathbf {x} _ {i}\right), \mathbf {y} _ {i}\right). \tag {1}
$$

Training datasets are considered as valuable assets since their quality has a considerable influence on the model performance. Training data samples are usually collected from various sources, including individual users whose private data may be sensitive [22], [1]. Potential data privacy breaches may occur in the pre-deployment and the post-deployment phases of deep learning [15], [38]. In the pre-deployment phase,

<sup>1</sup>Without loss of generality, we consider classification tasks in this paper and will explore the issue in generative DNNs in our future works.

attackers may collect a large pool of unauthorized data $( \mathbf { e . g . }$ from online social networks) to construct their training datasets [38], [9], [35], [18], [5], [19], [12]. In the post-deployment phase, attackers may infer the membership [39], [3], [42] or attributes [14], [11], [16] of training data samples.

In this paper, we focus on protecting data privacy in the pre-deployment phase. More specifically, the main objective is to protect private user data from being exploited by unauthorized parties to train a well-performed DNN model. To attain this goal, existing research works have proposed to add defensive noises to original data samples to reduce their learnability, i.e., the DNN model trained on the perturbed data samples will not reach satisfactory accuracy. Given an original sample x, its privacy-preserving version $\mathbf { x } ^ { \prime }$ is created as

$$
\mathbf {x} ^ {\prime} = \mathbf {x} + \delta . \tag {2}
$$

where δ is the defensive noise.

There are various choices for $\delta ,$ a naive one being the Gaussian noise. Adversarial noises are often adopted as they may mislead the training process. EMAX [29], UTAP [10], CTAP [10] and NTGA [50] all followed this line. Recently, the idea of unlearnable samples emerged [19], generating the defensive noise $\delta$ via an optimization problem

$$
\min _ {\theta} \mathbb {E} _ {(\mathbf {x}, \mathbf {y}) \in \mathcal {D} _ {c}} \left[ \min _ {\delta} \mathcal {L} \left(f _ {\theta} (\mathbf {x} + \delta), \mathbf {y}\right) \right], \tag {3}
$$

where $f _ { \theta }$ represents a hypothetical DNN model under training, $\mathcal { D } _ { c }$ is a clean training dataset used to train $f _ { \theta } ,$ and $\mathcal { L }$ is the loss function for training $f _ { \theta } .$ The bi-level optimization problem consists of an outer and an inner minimization problems. The outer minimization problem searches for the parameters θ that minimize the training error of $f _ { \theta }$ on clean training dataset $\mathcal { D } _ { c }$ . The inner minimization problem searches for an $L _ { p }$ -norm bounded noise δ that minimizes the loss of $f _ { \theta }$ on sample $\mathbf { x }   +   \delta$ such that model $f _ { \theta }$ learns almost nothing from sample $\mathbf { x } + { \boldsymbol { \delta } } .$

Despite the effort to minimize the learnability of original samples, unlearnable examples are found to be vulnerable to adversarial training [12]. More specifically, adversarial examples created based on these unlearnable examples will yield non-minimal loss, allowing the model to learn useful information, thus undermining privacy protection. To tackle this issue, Fu et al. [12] proposed a robust unlearnable example generation method using min-min-max optimization as

$$
\min _ {\theta} \mathbb {E} _ {(\mathbf {x}, \mathbf {y}) \in \mathcal {D} _ {c}} \min _ {\delta} \max _ {\epsilon} \mathcal {L} \left(f _ {\theta} (\mathbf {x} + \epsilon + \delta), \mathbf {y}\right), \tag {4}
$$

where ϵ is the adversarial noise added to maximize the loss for adversarial training. Both ϵ and δ are bounded to satisfy imperceptibility.

In this paper, we will reveal the vulnerability of existing defense methods, especially unlearnable examples, to data augmentation. We further propose corresponding countermeasures to strengthen data privacy protection.

## B. Data Augmentation

Data augmentation [44], [30], [43] is a technique that enriches a training dataset by generating new examples through transformations of original data samples, e.g., flipping, rotating, scaling, cropping, and changing color. The purpose of data augmentation is to help improve the performance and generalization capability of machine learning models, especially if the available training data samples are limited. By generating more diverse examples, data augmentation helps the model learn to be more robust to data variations. It is shown that data augmentation can also help defend against adversarial example attacks [34], [51] and backdoor attacks [2], [33]. In this paper, we review five state-of-the-art representative data augmentation methods, i.e., Mixup [53], feature distillation [27], PuzzleMix [21], Fast AutoAugment [25], and DeepAA [54].

TABLE I TEST ACCURACY OF MODELS TRAINED ON PROTECTED SAMPLES WITH OR WITHOUT DATA AUGMENTATION. THE AUGMENTATION METHOD IS DEEPAA [54]. PROTECTED SAMPLES ARE GENERATED BY GAUSSIAN NOISE, EMIN [19], AND REM [12].

<table><tr><td rowspan="9">CIFAR-10</td><td rowspan="3">No protection</td><td>No augmentation</td><td>VGG-16</td><td>ResNet-18</td><td>ResNet-50</td><td>DenseNet-121</td><td>WRN_34_10</td></tr><tr><td rowspan="2">Augmentation</td><td>92.66%</td><td>94.09%</td><td>94.38%</td><td>94.89%</td><td>95.52%</td></tr><tr><td>93.86%</td><td>94.49%</td><td>94.52%</td><td>95.28%</td><td>96.54%</td></tr><tr><td rowspan="2">Gaussian</td><td>No augmentation</td><td>92.49%</td><td>94.11%</td><td>93.56%</td><td>94.76%</td><td>95.61%</td></tr><tr><td>Augmentation</td><td>93.68%</td><td>92.79%</td><td>90.80%</td><td>93.63%</td><td>96.17%</td></tr><tr><td rowspan="2">EMIN [19]</td><td>No augmentation</td><td>25.59%</td><td>25.08%</td><td>19.19%</td><td>21.70%</td><td>21.38%</td></tr><tr><td>Augmentation</td><td>62.69%</td><td>56.05%</td><td>51.01%</td><td>57.32%</td><td>66.14%</td></tr><tr><td rowspan="2">REM [12]</td><td>No augmentation</td><td>29.61%</td><td>24.15%</td><td>20.50%</td><td>23.55%</td><td>24.50%</td></tr><tr><td>Augmentation</td><td>47.58%</td><td>43.25%</td><td>41.24%</td><td>44.39%</td><td>50.38%</td></tr><tr><td rowspan="9">CIFAR-100</td><td rowspan="3">No protection</td><td>No augmentation</td><td>VGG-16</td><td>ResNet-18</td><td>ResNet-50</td><td>DenseNet-121</td><td>WRN_34_10</td></tr><tr><td rowspan="2">Augmentation</td><td>69.43%</td><td>71.84%</td><td>71.69%</td><td>73.04%</td><td>75.77%</td></tr><tr><td>65.06%</td><td>75.01%</td><td>73.42%</td><td>76.06%</td><td>79.25%</td></tr><tr><td rowspan="2">Gaussian</td><td>No augmentation</td><td>69.30%</td><td>71.22%</td><td>69.61%</td><td>72.67%</td><td>75.05%</td></tr><tr><td>Augmentation</td><td>64.04%</td><td>72.15%</td><td>72.62%</td><td>73.76%</td><td>76.92%</td></tr><tr><td rowspan="2">EMIN [19]</td><td>No augmentation</td><td>10.81%</td><td>15.19%</td><td>11.24%</td><td>13.64%</td><td>10.95%</td></tr><tr><td>Augmentation</td><td>40.36%</td><td>39.92%</td><td>42.45%</td><td>40.32%</td><td>46.03%</td></tr><tr><td rowspan="2">REM [12]</td><td>No augmentation</td><td>15.10%</td><td>13.18%</td><td>11.91%</td><td>13.37%</td><td>11.90%</td></tr><tr><td>Augmentation</td><td>30.70%</td><td>27.07%</td><td>25.74%</td><td>32.74%</td><td>30.25%</td></tr><tr><td rowspan="9">Mini-ImageNet</td><td rowspan="3">No protection</td><td>No augmentation</td><td>VGG-16</td><td>ResNet-18</td><td>ResNet-50</td><td>DenseNet-121</td><td>WRN_34_10</td></tr><tr><td rowspan="2">Augmentation</td><td>75.69%</td><td>74.56%</td><td>68.34%</td><td>76.07%</td><td>71.11%</td></tr><tr><td>77.66%</td><td>77.87%</td><td>70.50%</td><td>72.53%</td><td>74.30%</td></tr><tr><td rowspan="2">Gaussian</td><td>No augmentation</td><td>59.37%</td><td>65.55%</td><td>59.60%</td><td>68.45%</td><td>60.69%</td></tr><tr><td>Augmentation</td><td>67.90%</td><td>70.84%</td><td>69.35%</td><td>68.64%</td><td>68.56%</td></tr><tr><td rowspan="2">EMIN [19]</td><td>No augmentation</td><td>14.98%</td><td>11.67%</td><td>16.81%</td><td>15.45%</td><td>22.96%</td></tr><tr><td>Augmentation</td><td>58.06%</td><td>63.51%</td><td>58.84%</td><td>53.08%</td><td>50.38%</td></tr><tr><td rowspan="2">REM [12]</td><td>No augmentation</td><td>26.81%</td><td>23.43%</td><td>24.02%</td><td>33.97%</td><td>33.40%</td></tr><tr><td>Augmentation</td><td>44.46%</td><td>43.60%</td><td>42.52%</td><td>53.95%</td><td>47.92%</td></tr><tr><td rowspan="9">VGG-Face</td><td rowspan="3">No protection</td><td>No augmentation</td><td>VGG-16</td><td>ResNet-18</td><td>ResNet-50</td><td>DenseNet-121</td><td>WRN_34_10</td></tr><tr><td rowspan="2">Augmentation</td><td>93.59%</td><td>93.09%</td><td>93.62%</td><td>95.56%</td><td>94.54%</td></tr><tr><td>97.18%</td><td>97.21%</td><td>93.02%</td><td>97.29%</td><td>97.73%</td></tr><tr><td rowspan="2">Gaussian</td><td>No augmentation</td><td>94.24%</td><td>94.28%</td><td>95.80%</td><td>96.20%</td><td>93.65%</td></tr><tr><td>Augmentation</td><td>97.21%</td><td>97.61%</td><td>95.78%</td><td>90.91%</td><td>95.12%</td></tr><tr><td rowspan="2">EMIN [19]</td><td>No augmentation</td><td>1.46%</td><td>1.49%</td><td>1.57%</td><td>1.53%</td><td>1.65%</td></tr><tr><td>Augmentation</td><td>42.63%</td><td>48.75%</td><td>41.14%</td><td>40.84%</td><td>35.75%</td></tr><tr><td rowspan="2">REM [12]</td><td>No augmentation</td><td>3.21%</td><td>4.32%</td><td>5.53%</td><td>3.46%</td><td>4.21%</td></tr><tr><td>Augmentation</td><td>36.86%</td><td>46.43%</td><td>40.78%</td><td>42.76%</td><td>37.43%</td></tr></table>

<u>Mixup.</u> Mixup [53] constructs a weighted combination of randomly selected pairs of samples from the training dataset. Given two samples $( \mathbf { x } _ { i } , \mathbf { y } _ { i } )$ and $( \mathbf { x } _ { j } , \mathbf { y } _ { j } )$ , an augmented sample is generated as $( \lambda \mathbf { x } _ { i } + ( 1 - \lambda ) \mathbf { x } _ { j } , \lambda \mathbf { y } _ { i } + ( 1 - \lambda ) \mathbf { y } _ { j } )$ , where $\lambda \in [ 0 , 1 ]$ is a mixing coefficient that determines the weight of each sample in the linear combination.

<u>Feature distillation.</u> Feature distillation [27] was initially designed to mitigate adversarial example attacks. It redesigns the quantization procedure of a traditional JPEG compression algorithm [46]. Depending on the location of the derived quantization in JPEG, feature distillation has the one-pass and the two-pass modes. The one-pass feature distillation inserts a quantization/de-quantization in the decompression process of the original JPEG algorithm. The two-pass feature distillation embeds a crafted quantization at the sensor side to compress the raw data samples.

<u>PuzzleMix.</u> PuzzleMix [21] is an extension of Mixup, resolving the problem of generating unnatural augmented samples. PuzzleMix finds the optimal mixing mask based on the saliency information. PuzzleMix is shown to have outperformed state-of-the-art Mixup methods in terms of generalization and robustness against data corruption and adversarial perturbations.

<u>FastAA.</u> AutoAugment [6] automates the search for an optimal data augmentation strategy given the original dataset using reinforcement learning. To reduce the computational complexity of AutoAugment, Fast AutoAugment (FastAA) [25] uses a more efficient search strategy based on density matching. The main idea is to learn a probability distribution over a space of candidate augmentation strategy that maximizes the performance of a given model on a validation dataset. The distribution is learned by matching the density of a learned feature space of augmented samples to that of original samples. In this way, FastAA significantly speeds up the search process compared to the original AutoAugment algorithm.

<u>DeepAA.</u> Deep AutoAugment (DeepAA) [54] constructs a multi-layer data augmentation framework. In each layer, the augmentation strategy is optimized to maximize the cosine similarity between the gradient of the original data and that of the augmented data along the low-variance direction. To avoid an exponential increase in the dimensionality of the search space, DeepAA incrementally stacks layers according to the data distribution transformed by all prior augmentation layers.

Given a set of augmentation methods, the augmentation strategy of each layer is represented as a probability of using each augmentation method.

$$
\arg \max _ {\mathbf {p}} \frac {\mathbf {g} ^ {T} \cdot \nabla \mathbf {g} ^ {A} (\mathbf {x} , \mathbf {p})}{\left| \left| \mathbf {g} ^ {T} \right| \right| \cdot \left| \left| \nabla \mathbf {g} ^ {A} (\mathbf {x} , \mathbf {p}) \right| \right|}, \tag {5}
$$

where p is the augmentation strategy, $\nabla \mathbf { g } ^ { A } ( \mathbf { x } , \mathbf { p } )$ is the average gradient of the augmented samples, $\mathbf { g } ^ { T }$ is the gradient of the original data samples, and || · || is the $L_{2} - \mathrm{norm}$

## III. UNVEILING THE EFFECTS OF DATA AUGMENTATION ON DATA PRIVACY

We reveal that data augmentation will impair the privacy protection provided by existing defensive noise generation methods. As far as we are concerned, this is the first time such a vulnerability has been exposed.

We first present the performance of different models trained on protected samples with or without data augmentation. As shown in Table I, we assess the protection performance of Gaussian noise and two state-of-the-art defense methods based on unlearnable examples, i.e., EMIN [19] and REM [12]. We use DeepAA [54] as the data augmentation method.

## Effect of Data Augmentation on Data Privacy

• **Without data augmentation**, data privacy protection provided by unlearnable examples are effective.
• **With data augmentation**, data privacy protection provided by unlearnable examples are undermined.

As shown in Table I, without data augmentation, EMIN and REM effectively prevent DNN models from achieving high test accuracy by learning from protected data samples. For instance, DNNs trained on unprotected clean CIFAR-10 dataset yield test accuracies of 92.66% (VGG-16) and 94.09% (ResNet-18). These accuracies drop to 25.59% (VGG-16) and 25.08% (ResNet-18) under EMIN protection. Unfortunately, when data augmentation is adopted, EMIN and REM struggle to keep the test accuracy of unauthorized models below 30%, especially for EMIN. For example, the test accuracy of the WRN 34 10 model is only 21.38% under EMIN for CIFAR-10 dataset, but jumps to 66.14% once DeepAA is utilized.

To further verify our findings, we assess other state-of-the-art data augmentation strategies, including Mixup [53], Feature distillation [27], PuzzleMix [21], and FastAA [25]. The experimental results are presented in Table II. The results demonstrate that various data augmentation strategies can undermine data privacy protection. Besides DeepAA, both FastAA and PuzzMix can substantially improve model test accuracy (e.g., >60% in several cases) even if EMIN and REM are used for privacy protection. More evaluation results of the effects of data augmentation on defense methods EMAX [29], UTAP [10], CTAP [10] and NTGA [50] can be found in Table IV and Table V.

Through comprehensive experiments, we have discovered that almost all existing defensive noises are susceptible to data augmentation. This finding highlights the importance of mitigating potential data privacy violations introduced by data augmentation.

## IV. PROTECTING DATA PRIVACY UNDER DATA AUGMENTATION

In this paper, we propose ARMOR to enhance data privacy protection under the impact of data augmentation.

## A. Threat Model

We consider two parties, i.e., the defender and the attacker.

**Defender**. The role of the defender is usually assumed by a data owner who possesses a private dataset $\mathcal { D } _ { c } = { ( \mathbf { x } _ { i } , y _ { i } ) } _ { i = 1 } ^ { n } .$ The goal of the defender is to generate a corresponding protected dataset $\mathcal { D } _ { u }$ to be published on social media, where $\mathcal { D } _ { u }   =   \{ ( \mathbf { x } _ { i } + \delta _ { i } , y _ { i } ) \} _ { i = 1 } ^ { n }$ . It should be ensured that a model trained on the published protected dataset $\mathcal { D } _ { u }$ performs poorly on the test set $\mathcal { D } _ { t }$ , even if the model trainer utilizes advanced data augmentation strategies. The defensive noise δ is bounded by $| | \delta | | _ { p } \leq \epsilon \mathrm {  ~ t o ~ }$ guarantee imperceptibility. Following existing defense works [19], we assume the defender has full access to the private dataset $\mathcal { D } _ { c } .$ . However, the defender does not have any knowledge or control over the training process of the attacker model. Additionally, the defender cannot modify the protected examples once they are published.

**Attacker**. The role of the attacker is usually assumed by a model trainer who aims to train a well-performed model $f$ with the public dataset $\mathcal { D } _ { u }$ . The attacker can employ any data augmentation strategy to improve the model performance. We also explore the effectiveness of ARMOR when the attacker adopts adversarial training to bolster model performance.

## B. Privacy Protection Problem Formulation

To generate defensive noise δ that can resist data augmentation, we formulate the optimization problem as

$$
\min _ {\theta} \mathbb {E} _ {(\mathbf {x}, \mathbf {y}) \sim \mathcal {D} _ {c}} \left[ \min _ {\delta} \mathcal {L} \left(f \left(\mathcal {A} (\mathbf {x} + \delta , \mathbf {p}), \mathbf {y}\right) \right], \right. \tag {6}
$$

where $\mathcal { A } ( \mathbf { x }   +   \delta , \mathbf { p } )$ is the function that enforces data augmentation strategy p on the protected sample $\mathbf { x }   +   \delta .$ The key idea of Equation (6) is to minimize the loss of the protected sample even if data augmentation is applied. In Equation (6), $f ( \cdot )$ and p are the surrogate model and the surrogate augmentation strategy that the defender uses to mimic the real model and real augmentation strategy adopted by the attacker. Therefore, the choice of $f ( \cdot )$ and p are essential to determine the privacy protection performance. In addition, to solve the min-min problem to attain the optimal defensive noise δ is non-trivia. To address these challenges, we equip ARMOR with three main building blocks, as depicted in Figure 1.

• Surrogate model construction. With no information on the structure of the attacker model, the defender may have to initialize a surrogate model. Our empirical study reveals that a surrogate model, established with the aid of data augmentation, can capture more informative features, leading to improved protection performance. Since data augmentation encourages the expansion of the model’s receptive field to a broader sample region, we introduce a non-local module as a plug-in into the surrogate model to capture a global receptive field of the sample.

TABLE II TEST ACCURACY OF MODELS TRAINED ON PROTECTED SAMPLES WITH OR WITHOUT DATA AUGMENTATION. DIFFERENT DATA AUGMENTATION METHODS ARE ADOPTED. PROTECTED SAMPLES ARE GENERATED BY EMIN AND REM.

<table><tr><td rowspan="6">Mixup [53]</td><td rowspan="2">Unprotected</td><td rowspan="2">Augmentation</td><td>CIFAR-10</td><td>CIFAR-100</td><td>Mini-ImageNet</td><td>VGG-Face</td></tr><tr><td>95.03%</td><td>77.19%</td><td>76.89%</td><td>94.49%</td></tr><tr><td rowspan="2">EMIN [19]</td><td>No Augmentation</td><td>25.08%</td><td>15.19%</td><td>14.98%</td><td>1.46%</td></tr><tr><td>Augmentation</td><td>33.65%</td><td>18.02%</td><td>17.79%</td><td>47.96%</td></tr><tr><td rowspan="2">REM [12]</td><td>No Augmentation</td><td>24.15%</td><td>13.18%</td><td>26.81%</td><td>3.21%</td></tr><tr><td>Augmentation</td><td>31.58%</td><td>17.61%</td><td>36.98%</td><td>36.76%</td></tr><tr><td rowspan="6">Feature distillation [27]</td><td rowspan="2">Unprotected</td><td rowspan="2">Augmentation</td><td>CIFAR-10</td><td>CIFAR-100</td><td>Mini-ImageNet</td><td>VGG-Face</td></tr><tr><td>92.91%</td><td>68.48%</td><td>77.11%</td><td>95.40%</td></tr><tr><td rowspan="2">EMIN [19]</td><td>No Augmentation</td><td>25.08%</td><td>15.19%</td><td>14.98%</td><td>1.46%</td></tr><tr><td>Augmentation</td><td>31.17%</td><td>22.08%</td><td>21.40%</td><td>38.65%</td></tr><tr><td rowspan="2">REM [12]</td><td>No Augmentation</td><td>24.15%</td><td>13.18%</td><td>26.81%</td><td>3.21%</td></tr><tr><td>Augmentation</td><td>32.67%</td><td>25.05%</td><td>36.13%</td><td>27.34%</td></tr><tr><td rowspan="6">PuzzleMix [21]</td><td rowspan="2">Unprotected</td><td rowspan="2">Augmentation</td><td>CIFAR-10</td><td>CIFAR-100</td><td>Mini-ImageNet</td><td>VGG-Face</td></tr><tr><td>95.37%</td><td>68.65%</td><td>76.46%</td><td>94.36%</td></tr><tr><td rowspan="2">EMIN [19]</td><td>No Augmentation</td><td>25.08%</td><td>15.19%</td><td>14.98%</td><td>1.46%</td></tr><tr><td>Augmentation</td><td>40.52%</td><td>28.57%</td><td>18.14%</td><td>48.78%</td></tr><tr><td rowspan="2">REM [12]</td><td>No Augmentation</td><td>24.15%</td><td>13.18%</td><td>26.81%</td><td>3.21%</td></tr><tr><td>Augmentation</td><td>40.43%</td><td>21.12%</td><td>64.95%</td><td>35.87%</td></tr><tr><td rowspan="6">FastAA [25]</td><td rowspan="2">Unprotected</td><td rowspan="2">Augmentation</td><td>CIFAR-10</td><td>CIFAR-100</td><td>Mini-ImageNet</td><td>VGG-Face</td></tr><tr><td>94.81%</td><td>73.64%</td><td>76.60%</td><td>94.97%</td></tr><tr><td rowspan="2">EMIN [19]</td><td>No Augmentation</td><td>25.08%</td><td>15.19%</td><td>14.98%</td><td>1.46%</td></tr><tr><td>Augmentation</td><td>44.00%</td><td>32.27%</td><td>18.94%</td><td>40.08%</td></tr><tr><td rowspan="2">REM [12]</td><td>No Augmentation</td><td>24.15%</td><td>13.18%</td><td>26.81%</td><td>3.21%</td></tr><tr><td>Augmentation</td><td>38.40%</td><td>24.72%</td><td>38.73%</td><td>35.74%</td></tr></table>

• Surrogate augmentation strategy selection. Our extensive experiment results in Section III have demonstrated that DeepAA [54] raises the accuracy of the attack model the most. Given an augmentation operation set, for each sample, DeepAA computes the optimal augmentation strategy to achieve gradient alignment, according to Equation (III). Inspired by DeepAA, we propose an augmentation strategy selection algorithm to dynamically select classspecific augmentation strategies.
• Defensive noise generation. We adopt the Projected Gradient Descent (PGD) [29] to solve the min-min optimization problem. PGD performs iterative updates to search for the optimal solution, transforming the noise generation into an iterative process of augmentation policy update and noise update. Besides, we propose to use an adaptive learning rate scheduling scheme in PGD, which enhances the efficiency of the iteration process and mitigates the risk of overfitting.

## C. Surrogate Model Construction

Ideally, the surrogate model should have the same structure as the attacker model. However, we assume that the defender has no access to the attacker model. Previous works [19], [12] have revealed that unlearnable examples generated by one specific model type can still show good protection performance under other trained model types. In this case, we may opt to choose a commonly used model structure, e.g., ResNet-18, for constructing the surrogate model.

Our extensive empirical study reveals that ARMOR provides better protection when the surrogate model benefits more from data augmentation. Based on this, we improve the fundamental surrogate model structure to amplify the impact of data augmentation.

Data augmentation aims to enhance model generalization capability by reducing the model’s reliance on local features, which helps mitigate overfitting. By increasing data diversity, data augmentation encourages the model to enlarge the receptive field to a broader sample region. Inspired by this principle, after initializing the surrogate model with a commonly-used DNN model, we further incorporate a non-local module [47] that captures a global receptive field of the sample.

The non-local module transforms an input a into a samedimensional output b as

$$
\mathbf {b} _ {i} = \frac {1}{\mathcal {C} (\mathbf {a})} \sum_ {\forall j} F \left(\mathbf {a} _ {i}, \mathbf {a} _ {j}\right) G \left(\mathbf {a} _ {j}\right), \tag {7}
$$

where $\mathbf { b } _ { i }$ is the i-th feature of output b, $\mathbf { a } _ { i }$ is the i-th feature of input a, $\mathcal { F } ( \cdot , \cdot )$ is a pairwise function whose output depends on the relationship between $\mathbf { a } _ { i }$ and $\mathbf { a } _ { j } ,$ and $\mathcal { G } ( \cdot )$ computes the j-th feature of the representation of input a. In addition, C(a) refers to a factor used for normalization.

The key to a non-local module is the design of G and ${ \mathcal { F } } .$ For G, we choose a linear transformation as $\mathcal { G } ( \mathbf { a } _ { j } ) = \mathbf { W } _ { G } \mathbf { a } _ { j } ,$ where $\mathbf { W } _ { G }$ is a learned weight matrix obtained in the training process. For $\mathcal { F }$ , we develop a Gaussian-embedded structure as

$$
\mathcal {F} \left(\mathbf {a} _ {i}, \mathbf {a} _ {j}\right) = e ^ {\varphi \left(\mathbf {a} _ {i}\right) ^ {T} \cdot \phi \left(\mathbf {a} _ {j}\right)}, \tag {8}
$$

where $\varphi ( \cdot )$ and $\phi ( \cdot )$ are linear transformation functions, and $\mathbf { x } ^ { T }$ is the transportation of matrix x.

![](images/image_01.jpg)

Fig. 1. Overview of ARMOR. ARMOR mainly contains three key components, i.e., surrogate model construction, surrogate augmentation strategy selection, and defensive noise generation.

TABLE III A LIST OF STANDARD AUGMENTATION SEARCH SPACE. “IDENTITY” DENOTES APPLYING NO DATA AUGMENTATION TO THE DATA SAMPLE.

| Operation | Magnitude |
| --- | --- |
| Identity | - |
| ShearX | [-0.3, 0.3] |
| ShearY | [-0.3, 0.3] |
| TranslateX | [-0.45, 0.45] |
| TranslateY | [-0.45, 0.45] |
| Rotate | [-30, 30] |
| AutoContrast | - |
| Invert | - |
| Equalize | - |
| Solarize | [0, 256] |
| Posterize | [4, 8] |
| Contrast | [0.1, 1.9] |
| Color | [0.1, 1.9] |
| Brightness | [0.1, 1.9] |
| Sharpness | [0.1, 1.9] |
| Flips | - |
| Cutout | [8, 16] |
| Crop | - |

The non-local module can be a plug-in to be inserted between any layers of a DNN model.

## D. Surrogate Augmentation Strategy Selection

Data augmentation enriches a dataset by synthesizing additional data points that follow the same underlying distribution [17]. Thus, an effective data augmentation should maintain the underlying data distribution, ensuring that the distribution of augmented samples aligns with that of the original samples [4]. To achieve this goal, DeepAA adopts gradient alignment, i.e., aligning the gradients computed on the augmented sample batch with those computed based on the original sample batch [54]. As the attacker only gets the protected version of private

## Algorithm 1 Surrogate Augmentation Strategy Selection

**Require:** Clean dataset $D _ { c }$ , existing noise δ, existing augmentation strategy p, pretrained auxiliary model ${ \bar { f ^ { \prime } } } ,$ num of classes c and training epoch $r .$

**Ensure:** Augmentation strategy p.

1: **for** k in $1 , 2 , . . . r$ **do**
2: Sample a minibatch $( x _ { k } , y _ { k } ) \sim D _ { c } .$
3: $x _ { k } ^ { \prime } = x _ { k } + \delta .$
4: Update the auxiliary model f′ based on minibatch $( x _ { k } ^ { \prime } , y _ { k } )$
5: **end for**
6: **for** i in $0 , 1 , 2 , . . . c - 1$ **do**
7: Randomly select sample batches $x _ { 1 }$ and $x _ { 2 }$ of the same label i.
8: //Gradient matching
9: $\mathbf { p } _ { i } = \operatorname { a r g } \operatorname { m a x } _ { o \in \mathbb { O } } \mathsf { C S } ( \nabla x _ { 1 } , \nabla \mathcal { A } ( x _ { 2 } , \mathbf { o } ) ) ,$ ,
10: **end for**
11: Return p.

samples, we select the augmentation strategy that maximizes the cosine similarity between the average gradients of the protected data and those of the augmented protected data.

Mathematically, we search for the optimal augmentation strategy p as

$$
\begin{array}{l} \mathbf {p} = \arg \max _ {\mathbf {p}} \mathrm{CS} (\nabla x _ {1}, \nabla \mathcal {A} (x _ {2}, \mathbf {p})) \\ = \arg \max _ {\mathbf {p}} \frac {\nabla x _ {1} \times \nabla \mathcal {A} \left(x _ {2} , \mathbf {p}\right)}{\left| \left| \nabla x _ {1} \right| \right| \times \left| \left| \nabla \mathcal {A} \left(x _ {2} , \mathbf {p}\right) \right| \right|}, \tag {9} \\ \end{array}
$$

where $\operatorname { C S } ( \cdot , \cdot )$ calculates the cosine similarity, || · || represents the $L _ { 2 }$ norm, and $x _ { 1 }$ and $x _ { 2 }$ are two batches of images from the current perturbed data. To improve efficiency, we adopt the class-wise augmentation strategy, i.e., we select a specific strategy for each class in every epoch, since samples from the same class usually share similar feature representations.

Algorithm 2 Defensive Noise Generation

<div class="docvortex-algorithm" style="white-space: pre-wrap; font-family:monospace;">
Require: Clean dataset  $D_{c}$ , existing noise  $\delta$ , existing augmentation strategy p, training epoch of surrogate model  $r_{f}$ , and noise training epoch  $r_{d}$ .
</div>

Ensure: Updated noise δ.

<div class="docvortex-algorithm" style="white-space: pre-wrap; font-family:monospace;">
// The process of noise update.
for $k$ in $1, 2, \ldots, r_f$ do
    Sample a minibatch $(x_k, y_k) \sim D_c$.
    Update surrogate model $f$ based on minibatch $(\mathcal{A}(x_k + \delta, \mathbf{p}), y_k)$.
end for
$\delta_0 = \delta$;
//PGD method iteration with the adaptive step size; the first subscript indicates the image index, the second subscript indicates the number of update rounds; $\beta, c, \gamma$ are default constants.
</div>

8: for t in $1 , 2 , . . . r _ { d }$ do

<div class="docvortex-algorithm" style="white-space: pre-wrap; font-family:monospace;">
9: $n_{i,t} = \beta n_{i,t-1} + (1 - \beta)||\nabla_{x_{i,t-1}}\mathrm{L}(x_{i,t-1},y_i)||_2^2.$
10: $\alpha_t = \frac{\gamma}{c+\sqrt{\sum_i n_{i,t}}}.$
11: $\delta_t = \delta_{t-1} - \alpha_t * \mathrm{sign}(\nabla_\delta \mathcal{L}(f(\mathcal{A}(\mathbf{x} + \delta_{t-1},\mathbf{p}),\mathbf{y}))),$
</div>

12: end for

13: Return $\delta .$

For each class, we determine the optimal augmentation strategy using an auxiliary model $f ^ { \prime } .$ This model is initially trained with clean samples and subsequently updated using perturbed data during each epoch of augmentation strategy selection. Let $\mathbf { p } _ { i }$ represent the selected augmentation operation for the i-th class. The augmentation strategy selection is updated as

$$
\mathbf {p} _ {i} = \arg \max _ {o \in \mathbb {O}} \mathbf {C S} \left(\nabla x _ {1}, \nabla \mathcal {A} \left(x _ {2}, \mathbf {o}\right)\right), \tag {10}
$$

where O denotes the augmentation operation set as defined in Table III, and $x _ { 1 }$ and $x _ { 2 }$ represent two batches of images labeled i. The algorithm for selecting the surrogate augmentation strategy is outlined in Algorithm 1.

## E. Defensive Noise Generation

The defensive noise is generated to make the attacker model have a small loss on the protected sample. Thus, the attacker model can hardly learn any constructive information from the image. We use cross-entropy as the loss function and the firstorder optimization method PGD [29] to update the noise as

$$
\delta_ {t + 1} = \delta_ {t} - \alpha * \operatorname{sign} (\nabla \mathcal {L} (f (\mathcal {A} (\mathbf {x} + \delta_ {t}, \mathbf {p}), \mathbf {y}))), \tag {11}
$$

where t is the current iteration, and α is the learning rate.

The learning rate α determines the length of each step taken in the negative direction of the gradient during the gradient descent iteration process. If the learning rate is too small, the training process may be too slow to converge. If the learning rate is too large, the training loss may fluctuate and fail to decrease monotonically. Therefore, it is essential to dynamically adjust the learning rate according to the status of the current iteration. Traditionally, PGD uses a constant learning rate.

Instead, we adjust the learning rate to be inversely proportional to the input gradient norm. Specifically, samples with larger gradient norms are given lower learning rates [20], and vice versa.

$$
\begin{array}{l} n _ {i, t} = \beta n _ {i, t - 1} + (1 - \beta) | | \nabla_ {\mathbf {x} _ {i, t - 1}} \mathcal {L} (x _ {i, t - 1}, y _ {i}) | | _ {2} ^ {2}, \\ \alpha_ {t} = \frac {\gamma}{c + \sqrt {\sum_ {i} n _ {i , t}}}, \tag {12} \\ \end{array}
$$

where $x _ { i , t }$ is i-th sample at the t-th iteration, $\beta$ is the momentum, $n _ { i , t }$ is the moving average of the gradient norm, and $\gamma , c$ are hyperparameters to prevent $\alpha _ { t }$ from being too large.

The defensive noise generation process is iterated until the training process of the surrogate model converges. Note that our noise generation algorithm can generate both sample-wise noise and class-wise noise. Sample-wise noise can be directly optimized via Equation (11). For class-wise noise, we compute and average the generated defensive noise of each sample in a given class.

The defensive noise generation process is summarized in Algorithm 2.

## V. EVALUATION SETUP

## A. Model and Datasets

We conduct experiments on four widely-used datasets, i.e., CIFAR-10 [23], CIFAR-100 [23], Mini-ImageNet [8], and VGG-Face [32]. We utilize ResNet-18 to train the surrogate model on CIFAR-10, CIFAR-100, Mini-ImageNet, and VGG-Face by default.

**CIFAR-10**. CIFAR-10 [23] consists of 60,000 color images, each of size $3 2   \times   3 2$ pixels, divided into 10 different classes. Each class contains 6,000 images. The images in CIFAR-10 are quite diverse and can be challenging to classify accurately due to variations in lighting, scale, orientation, and the presence of overlapping objects. We randomly select 50,000 samples from the dataset to form the training set, while the remaining 10,000 samples are reserved for the test set. We train the model on the training set for 100 epochs. The learning rate is 0.1, the batch size is 128, the weight decay is 0.0005, and the momentum stochastic gradient descent is 0.9. A CosineAnnealingLR scheduler is used in the training process, with a T max value of 100 and an eta min value of 0.

**CIFAR-100**. CIFAR-100 [23] is an extension of the CIFAR-10 dataset. It consists of 60,000 color images, each of size 32 × 32 pixels, but unlike CIFAR-10, it contains 100 different classes. Each class in CIFAR-100 represents a fine-grained category, and there are 600 images per class. We divide the dataset into a training set and a test set, with 50,000 images for training and 10,000 images for testing. We train the model for 150 epochs. We set the learning rate as 0.1, the batch size as 128, and the momentum stochastic gradient descent as 0.9. A CosineAnnealingLR scheduler is used in the training process, with a T max value of 150 and an eta min value of 0.

**Mini-ImageNet.** Mini-ImageNet, a widely used subset of ImageNet [8], holds significant popularity within the research community [49], [28]. Mini-ImageNet is carefully selected to include diverse object categories and a range of image variations, making it suitable for evaluating and comparing different image classification models. Mini-ImageNet comprises a total of 60000 images, and we divide 40,000 images as the training set and 10,000 images as the test set. Each image has a high resolution with a dimension of 224 × 224. We train the model for 150 epochs. We set the learning rate as 0.1, the batch size as 64, the momentum of stochastic gradient descent as 0.9, and weight decay as 0.0005. A CosineAnnealingLR scheduler is used in the training process, with a T max value of 150 and an eta min value of 0.

![](images/image_02.jpg)

Fig. 2. Visualization results of different types of defensive noise and corresponding unlearnable samples of EMAX [29], TAP [10], NTGA [50], EMIN [19], REM [12], and ARMOR. For each dataset, we randomly select two samples as the example.

**VGG-Face.** VGGFace [32] is a large-scale face recognition dataset that was created by researchers at the Visual Geometry Group (VGG). VGGFace contains a vast collection of face images, encompassing a wide range of identities from various sources. It includes approximately 2.6 million images of over 2,600 individuals, making it one of the largest face recognition datasets available. Each image has a high resolution with a dimension of 224 × 224. In this paper, we randomly select 200 categories of images to form a dataset. There are 53,811 images for training and 10,762 images for testing. We resize the images to 3 × 224 × 224. We set the learning rate as 0.01. The mini-batch size is set as 64. A CosineAnnealingLR scheduler is used in the training process, with a T max value of 150 and an eta min value of 0.

## B. Evaluation Metrics

In line with prior research [12], [19], we employ test accuracy as a metric to evaluate the privacy-preserving capability of the noise. A lower test accuracy indicates that the model has acquired minimal knowledge from the training data, thereby suggesting a robust privacy protection ability of the noise.

In the experiments, we use sample-wise noise by default unless otherwise specified. We assume the commercial model trainer primarily uses DeepAA [54], a widely-used advanced automatic data augmentation strategy. The search space of the optimal augmentation operations is shown in Table III. We also explore the effectiveness of ARMOR when the model trainer adopts other state-of-the-art data augmentation strategies, such as Mixup [53], Feature distillation [27], PuzzleMix [21], and FastAA [25] in the experiments.

## C. Experiential Settings of Various Defensive Noise Generation Methods

We compare ARMOR with several state-of-the-art data protection methods, including Gaussian noise, EMAX [29], TAP [10], NTGA [50], EMIN [19], and REM [12].

**Gaussian noise.** We randomly generate noise independently for each training example or each class, sampling from the interval [−ϵ, ϵ].

**EMAX.** EMAX [29] is generated based on the gradients of a pre-trained model. We generate the noise for EMAX using a PGD-20 attack on a pre-trained ResNet-18 model applied to the training dataset, and the step size is set to 2.

**EMIN.** EMIN [19] is generated according to Equation (3). In every epoch, we train the surrogate model for a few epochs to solve the outer minimization problem, and PGD is employed to solve the inner minimization problem. The number of training epochs is set to 10, and the PGD attack is parameterized with 10 rounds, each with a step size of 0.8. The stop condition error rate is $\lambda = 0 . 1$ for sample-wise noise and λ = 0.01 for class-wise noise.

TABLE IV TEST ACCURACY OF MODELS TRAINED ON DATA PROTECTED BY ARMOR AND BASELINE DEFENSIVE NOISES IN THE NON-DATA-AUGMENTATION SCENARIO. ARMOR AND BASELINES USE SAMPLE-WISE NOISE.

<table><tr><td rowspan="2">Model</td><td colspan="9">CIFAR-10</td></tr><tr><td>Unprotected</td><td>GaussNoise</td><td>EMAX [29]</td><td>UTAP [10]</td><td>CTAP [10]</td><td>NTGA [50]</td><td>EMIN [19]</td><td>REM [12]</td><td>ARMOR</td></tr><tr><td>VGG-16</td><td>92.66%</td><td>92.49%</td><td>90.91%</td><td>85.90%</td><td>34.70%</td><td>42.13%</td><td>25.59%</td><td>29.61%</td><td>14.66%</td></tr><tr><td>ResNet-18</td><td>94.09%</td><td>94.11%</td><td>91.10%</td><td>84.72%</td><td>20.74%</td><td>33.88%</td><td>25.08%</td><td>24.15%</td><td>12.85%</td></tr><tr><td>ResNet-50</td><td>94.38%</td><td>93.56%</td><td>91.78%</td><td>85.35%</td><td>18.82%</td><td>20.55%</td><td>19.19%</td><td>20.50%</td><td>11.74%</td></tr><tr><td>DenseNet-121</td><td>94.89%</td><td>94.76%</td><td>87.89%</td><td>79.25%</td><td>18.43%</td><td>31.09%</td><td>21.70%</td><td>23.55%</td><td>11.09%</td></tr><tr><td>WRN_34_10</td><td>95.52%</td><td>95.61%</td><td>89.84%</td><td>82.76%</td><td>18.77%</td><td>25.52%</td><td>21.38%</td><td>24.50%</td><td>11.94%</td></tr><tr><td rowspan="2">Model</td><td colspan="9">CIFAR-100</td></tr><tr><td>Unprotected</td><td>GaussNoise</td><td>EMAX [29]</td><td>UTAP [10]</td><td>CTAP [10]</td><td>NTGA [50]</td><td>EMIN [19]</td><td>REM [12]</td><td>ARMOR</td></tr><tr><td>VGG-16</td><td>69.43%</td><td>69.30%</td><td>66.16%</td><td>63.26%</td><td>62.45%</td><td>35.86%</td><td>10.81%</td><td>15.10%</td><td>9.35%</td></tr><tr><td>ResNet-18</td><td>71.84%</td><td>71.22%</td><td>68.10%</td><td>66.15%</td><td>64.00%</td><td>22.41%</td><td>15.19%</td><td>13.18%</td><td>12.62%</td></tr><tr><td>ResNet-50</td><td>71.69%</td><td>69.61%</td><td>69.79%</td><td>66.10%</td><td>65.31%</td><td>19.74%</td><td>13.24%</td><td>11.91%</td><td>12.63%</td></tr><tr><td>DenseNet-121</td><td>73.04%</td><td>72.67%</td><td>71.50%</td><td>67.32%</td><td>67.66%</td><td>28.58%</td><td>13.64%</td><td>13.37%</td><td>13.23%</td></tr><tr><td>WRN_34_10</td><td>75.77%</td><td>75.05%</td><td>73.45%</td><td>70.67%</td><td>68.50%</td><td>18.57%</td><td>10.95%</td><td>11.90%</td><td>7.46%</td></tr><tr><td rowspan="2">Model</td><td colspan="9">Mini-ImageNet</td></tr><tr><td>Unprotected</td><td>GaussNoise</td><td>EMAX [29]</td><td>UTAP [10]</td><td>CTAP [10]</td><td>NTGA [50]</td><td>EMIN [19]</td><td>REM [12]</td><td>ARMOR</td></tr><tr><td>ResNet-18</td><td>75.69%</td><td>59.37%</td><td>57.45%</td><td>61.39%</td><td>21.00%</td><td>50.76%</td><td>14.98%</td><td>26.81%</td><td>3.72%</td></tr><tr><td>ResNet-50</td><td>74.56%</td><td>65.55%</td><td>57.25%</td><td>60.37%</td><td>18.70%</td><td>52.20%</td><td>11.67%</td><td>23.43%</td><td>4.48%</td></tr><tr><td>DenseNet-121</td><td>68.34%</td><td>59.60%</td><td>59.43%</td><td>54.74%</td><td>20.12%</td><td>57.94%</td><td>16.81%</td><td>24.02%</td><td>6.00%</td></tr><tr><td>EfficientNet</td><td>76.07%</td><td>68.45%</td><td>58.54%</td><td>52.28%</td><td>30.33%</td><td>53.22%</td><td>15.45%</td><td>33.97%</td><td>2.25%</td></tr><tr><td>ResNext-50</td><td>71.11%</td><td>60.69%</td><td>57.75%</td><td>45.14%</td><td>17.62%</td><td>54.36%</td><td>22.96%</td><td>33.40%</td><td>2.90%</td></tr><tr><td rowspan="2">Model</td><td colspan="9">VGG-Face</td></tr><tr><td>Unprotected</td><td>GaussNoise</td><td>EMAX [29]</td><td>UTAP [10]</td><td>CTAP [10]</td><td>NTGA [50]</td><td>EMIN [19]</td><td>REM [12]</td><td>ARMOR</td></tr><tr><td>ResNet-18</td><td>93.59%</td><td>94.24%</td><td>69.88%</td><td>84.73%</td><td>65.76%</td><td>87.54%</td><td>1.46%</td><td>3.21%</td><td>0.28%</td></tr><tr><td>ResNet-50</td><td>93.09%</td><td>94.28%</td><td>60.56%</td><td>85.61%</td><td>70.73%</td><td>90.48%</td><td>1.49%</td><td>4.32%</td><td>0.19%</td></tr><tr><td>DenseNet-121</td><td>93.62%</td><td>95.80%</td><td>61.36%</td><td>85.92%</td><td>72.54%</td><td>91.36%</td><td>1.57%</td><td>5.53%</td><td>0.25%</td></tr><tr><td>EfficientNet</td><td>95.56%</td><td>96.20%</td><td>61.53%</td><td>87.63%</td><td>73.25%</td><td>88.94%</td><td>1.53%</td><td>3.46%</td><td>0.27%</td></tr><tr><td>ResNext-50</td><td>94.54%</td><td>93.65%</td><td>62.42%</td><td>84.32%</td><td>67.56%</td><td>86.75%</td><td>1.65%</td><td>4.21%</td><td>0.17%</td></tr></table>

**TAP.** TAP [10] is generated by performing a targeted adversarial attack on the model trained with clean data. In this attack, we follow the hyperparameter settings as specified in the original work.

**NTGA.** NTGA [50] is an efficient method enabling cleanlabel, black-box generalization attacks against Deep Neural Networks. This work is based on the development of Neural Tangent Kernels (NTKs). On each dataset, we randomly split 10% of examples from the training dataset as a validation set to assist in the generation of noise.

**REM**. REM [12] follows Equation (4) to generate the noise. We set the adversarial training perturbation radius as 2, as it is commonly used in the experiments of the original work. All the other hyperparameter settings follow the specifications of the original work.

**ARMOR**. In surrogate model construction, the non-local module is respectively added to the first two residual blocks of ResNet-18. For surrogate augmentation strategy selection, we choose ResNet-18 as our auxiliary model structure and pre-train the auxiliary model from scratch on the entire training dataset for 10 epochs. During each iteration, we train the auxiliary model with 10 batches of perturbed data, and then use it to update our surrogate augmentation strategy. In defensive noise generation, we follow the same process as EMIN. We train the surrogate model on 10 batches, and the PGD attack is parameterized with 10 rounds, each with a step size of 0.8.

The stop condition is also the same as EMIN.

Note that for baselines and ARMOR, the radius of defensive perturbation ϵ is set to 8/255. Based on prior studies in adversarial research, the defensive noise under this constraint is imperceptible to human observers.

All experiments are implemented in Python and run on a 14-core Intel(R) Xeon(R) Gold 5117 CPU @2.00GHz and NVIDIA GeForce RTX 3080 Ti GPU machine running Ubuntu 18.04 system.

## VI. EVALUATION RESULTS

## A. Comparison with Baselines

We compare ARMOR with several state-of-the-art data protection methods, including EMAX [29], TAP [10], NTGA [50], EMIN [19], and REM [12]. We implement these baselines according to their open-source codes. We also use Gaussian noise as a baseline.

We compare ARMOR with the baselines in both non-dataaugmentation and data-augmentation scenarios. In the two scenarios, there are two kinds of noises, i.e., sample-wise noise and class-wise noise. Most of these methods generate samplewise noise. Only EMAX [29] and EMIN [19] also include class-wise noise. Thus, we only compare ARMOR with EMAX [29] and EMIN [19] for the performance of class-wise noise. In the experiments, we use different model structures to test the effectiveness and transferability of the noise generated by ARMOR and baselines.

We first compare ARMOR with baselines in terms of samplewise noise. The comparison results are shown in Table IV and Table V. We can see that models trained on unlearnable data generated by ARMOR consistently yield the lowest test accuracy across various datasets and model architectures. Besides, random noise has almost no effect on protecting private data from being used to train a well-performed model. In the non-data-augmentation scenario, take CIFAR-10 as an example, ARMOR brings the test accuracy from 92.66% down to 14.66% (VGG-16), from 94.09% down to 12.85% (ResNet-18), from 94.38% down to 11.74% (ResNet-50), from 94.89% down to 11.09% (DesNet-121), from 95.52% down to 11.94% (WRN 34 10), respectively. In contrast, the lowest model test accuracy of baseline methods is still as high as 25.59% (VGG-16), 20.74% (ResNet-18), 18.82% (ResNet-50), 18.43% (DenseNet-121), and 18.77% (WRN 34 10). For the high-resolution datasets, ARMOR can significantly reduce the test accuracy of different models to less than 2.25% (Mini-ImageNet) and less than 0.17% (VGG-Face), which are more effective than the baselines. As shown in Table V, we can see that the clean model accuracy improves after applying the data augmentation strategy, and most of the noises of baselines are ineffective in this case. For example, EMIN can reduce the test accuracy from 92.66% down to 25.59% without data augmentation for the CIFAR-10 dataset using VGG-16, however, it can only decrease the test accuracy from 93.86% down to 62.69% with data augmentation. In comparison, ARMOR can also successfully reduce the model test accuracy to less than 30% in most cases under the data augmentation. These results show that ARMOR is robust to data augmentation.

TABLE V TEST ACCURACY OF MODELS TRAINED ON DATA PROTECTED BY ARMOR AND BASELINE DEFENSIVE NOISES IN THE DATA-AUGMENTATION SCENARIO. ARMOR AND BASELINES USE SAMPLE-WISE NOISE.

<table><tr><td rowspan="2">Model</td><td rowspan="2">Unprotected</td><td rowspan="2">Random</td><td rowspan="2">EMAX [29]</td><td colspan="6">CIFAR-10</td></tr><tr><td>UTAP [10]</td><td>CTAP [10]</td><td>NTGA [50]</td><td>EMIN [19]</td><td>REM [12]</td><td>ARMOR</td></tr><tr><td>VGG-16</td><td>93.86%</td><td>93.68%</td><td>90.33%</td><td>85.28%</td><td>50.02%</td><td>89.95%</td><td>62.69%</td><td>47.58%</td><td>29.59%</td></tr><tr><td>ResNet-18</td><td>94.49%</td><td>92.79%</td><td>89.42%</td><td>85.79%</td><td>40.20%</td><td>90.71%</td><td>56.05%</td><td>43.25%</td><td>28.84%</td></tr><tr><td>ResNet-50</td><td>94.52%</td><td>90.80%</td><td>83.96%</td><td>80.61%</td><td>38.95%</td><td>90.25%</td><td>51.01%</td><td>41.24%</td><td>28.07%</td></tr><tr><td>DenseNet-121</td><td>95.28%</td><td>93.63%</td><td>87.31%</td><td>78.43%</td><td>39.04%</td><td>80.91%</td><td>57.32%</td><td>44.39%</td><td>31.32%</td></tr><tr><td>WRN_34_10</td><td>96.54%</td><td>96.17%</td><td>90.43%</td><td>84.00%</td><td>42.30%</td><td>82.55%</td><td>66.14%</td><td>50.38%</td><td>30.73%</td></tr><tr><td rowspan="2">Model</td><td rowspan="2">Unprotected</td><td rowspan="2">Random</td><td rowspan="2">EMAX [29]</td><td colspan="6">CIFAR-100</td></tr><tr><td>UTAP [10]</td><td>CTAP [10]</td><td>NTGA [50]</td><td>EMIN [19]</td><td>REM [12]</td><td>ARMOR</td></tr><tr><td>VGG-16</td><td>65.06%</td><td>64.04%</td><td>60.31%</td><td>58.26%</td><td>54.68%</td><td>63.15%</td><td>40.36%</td><td>30.70%</td><td>22.60%</td></tr><tr><td>ResNet-18</td><td>75.01%</td><td>72.15%</td><td>69.59%</td><td>67.44%</td><td>64.91%</td><td>52.30%</td><td>39.92%</td><td>27.07%</td><td>25.10%</td></tr><tr><td>ResNet-50</td><td>73.42%</td><td>72.62%</td><td>69.17%</td><td>68.51%</td><td>63.38%</td><td>62.64%</td><td>42.45%</td><td>25.74%</td><td>24.04%</td></tr><tr><td>DenseNet-121</td><td>76.06%</td><td>73.76%</td><td>70.35%</td><td>67.00%</td><td>64.26%</td><td>62.63%</td><td>40.32%</td><td>32.74%</td><td>26.48%</td></tr><tr><td>WRN_34_10</td><td>79.25%</td><td>76.92%</td><td>73.88%</td><td>71.11%</td><td>71.12%</td><td>54.41%</td><td>46.03%</td><td>30.25%</td><td>26.40%</td></tr><tr><td rowspan="2">Model</td><td rowspan="2">Unprotected</td><td rowspan="2">Random</td><td rowspan="2">EMAX [29]</td><td colspan="6">Mini-ImageNet</td></tr><tr><td>UTAP [10]</td><td>CTAP [10]</td><td>NTGA [50]</td><td>EMIN [19]</td><td>REM [12]</td><td>ARMOR</td></tr><tr><td>ResNet-18</td><td>77.66%</td><td>67.90%</td><td>59.99%</td><td>63.26%</td><td>37.69%</td><td>64.28%</td><td>58.06%</td><td>44.46%</td><td>11.69%</td></tr><tr><td>ResNet-50</td><td>77.87%</td><td>70.84%</td><td>58.29%</td><td>60.92%</td><td>36.42%</td><td>60.56%</td><td>63.51%</td><td>43.60%</td><td>14.27%</td></tr><tr><td>DenseNet-121</td><td>70.50%</td><td>69.35%</td><td>54.59%</td><td>54.02%</td><td>35.12%</td><td>63.06%</td><td>58.84%</td><td>42.52%</td><td>11.96%</td></tr><tr><td>EfficientNet</td><td>72.53%</td><td>68.64%</td><td>62.19%</td><td>51.79%</td><td>40.74%</td><td>66.42%</td><td>53.08%</td><td>53.95%</td><td>13.67%</td></tr><tr><td>ResNext-50</td><td>74.30%</td><td>68.56%</td><td>57.50%</td><td>45.74%</td><td>34.44%</td><td>58.23%</td><td>60.78%</td><td>47.92%</td><td>12.80%</td></tr><tr><td rowspan="2">Model</td><td rowspan="2">Unprotected</td><td rowspan="2">Random</td><td rowspan="2">EMAX [29]</td><td colspan="6">VGG-Face</td></tr><tr><td>UTAP [10]</td><td>CTAP [10]</td><td>NTGA [50]</td><td>EMIN [19]</td><td>REM [12]</td><td>ARMOR</td></tr><tr><td>ResNet-18</td><td>97.18%</td><td>97.21%</td><td>89.65%</td><td>97.02%</td><td>80.66%</td><td>94.35%</td><td>42.63%</td><td>36.86%</td><td>23.10%</td></tr><tr><td>ResNet-50</td><td>97.21%</td><td>97.61%</td><td>89.66%</td><td>97.00%</td><td>82.57%</td><td>96.57%</td><td>48.75%</td><td>46.43%</td><td>28.36%</td></tr><tr><td>DenseNet-121</td><td>93.02%</td><td>95.78%</td><td>90.73%</td><td>96.49%</td><td>85.10%</td><td>93.78%</td><td>41.14%</td><td>40.78%</td><td>26.11%</td></tr><tr><td>EfficientNet</td><td>97.29%</td><td>90.91%</td><td>86.34%</td><td>96.42%</td><td>83.11%</td><td>95.32%</td><td>40.84%</td><td>42.76%</td><td>20.79%</td></tr><tr><td>ResNext-50</td><td>97.73%</td><td>95.12%</td><td>87.83%</td><td>97.67%</td><td>82.10%</td><td>92.90%</td><td>35.75%</td><td>37.43%</td><td>18.97%</td></tr></table>

TABLE VI TEST ACCURACY OF MODELS TRAINED ON DATA PROTECTED BY ARMOR AND BASELINE DEFENSIVE NOISES IN THE NON-DATA-AUGMENTATION SCENARIO. ARMOR AND BASELINES USE CLASS-WISE NOISE.

<table><tr><td>Dataset</td><td>Model</td><td>Unprotected</td><td>EMIN</td><td>EMAX</td><td>ARMOR</td></tr><tr><td rowspan="5">CIFAR-10</td><td>VGG-16</td><td>92.66%</td><td>19.75%</td><td>27.89%</td><td>11.53%</td></tr><tr><td>ResNet-18</td><td>94.09%</td><td>12.21%</td><td>10.85%</td><td>10.69%</td></tr><tr><td>ResNet-50</td><td>94.38%</td><td>12.49%</td><td>12.74%</td><td>11.78%</td></tr><tr><td>DenseNet-121</td><td>94.89%</td><td>11.76%</td><td>13.27%</td><td>9.40%</td></tr><tr><td>WRN_34_10</td><td>95.52%</td><td>15.64%</td><td>15.44%</td><td>10.46%</td></tr><tr><td rowspan="5">CIFAR-100</td><td>VGG-16</td><td>69.43%</td><td>2.25%</td><td>37.21%</td><td>1.58%</td></tr><tr><td>ResNet-18</td><td>71.84%</td><td>5.17%</td><td>29.74%</td><td>1.69%</td></tr><tr><td>ResNet-50</td><td>71.69%</td><td>1.97%</td><td>24.44%</td><td>1.55%</td></tr><tr><td>DenseNet-121</td><td>73.04%</td><td>1.61%</td><td>23.93%</td><td>1.12%</td></tr><tr><td>WRN_34_10</td><td>75.77%</td><td>1.89%</td><td>23.31%</td><td>1.71%</td></tr><tr><td rowspan="5">Mini-ImageNet</td><td>ResNet-18</td><td>75.69%</td><td>1.40%</td><td>7.81%</td><td>1.34%</td></tr><tr><td>ResNet-50</td><td>74.56%</td><td>1.95%</td><td>7.92%</td><td>1.52%</td></tr><tr><td>DenseNet-121</td><td>68.34%</td><td>2.05%</td><td>6.38%</td><td>1.59%</td></tr><tr><td>MobileNet</td><td>76.34%</td><td>1.29%</td><td>4.05%</td><td>1.46%</td></tr><tr><td>ResNext-50</td><td>71.11%</td><td>2.11%</td><td>6.14%</td><td>2.04%</td></tr><tr><td rowspan="5">VGG-Face</td><td>ResNet-18</td><td>93.59%</td><td>1.28%</td><td>1.28%</td><td>0.51%</td></tr><tr><td>ResNet-50</td><td>93.09%</td><td>0.39%</td><td>1.28%</td><td>0.32%</td></tr><tr><td>DenseNet-121</td><td>93.62%</td><td>0.67%</td><td>0.48%</td><td>0.26%</td></tr><tr><td>ResNext-50</td><td>88.72%</td><td>0.70%</td><td>1.28%</td><td>0.28%</td></tr><tr><td>EfficientNet</td><td>95.56%</td><td>0.43%</td><td>0.51%</td><td>0.48%</td></tr></table>

TABLE VII TEST ACCURACY OF MODELS TRAINED ON DATA PROTECTED BY ARMOR AND BASELINE DEFENSIVE NOISES IN THE DATA-AUGMENTATION SCENARIO. ARMOR AND BASELINES USE CLASS-WISE NOISE.

<table><tr><td>Dataset</td><td>Model</td><td>Unprotected</td><td>EMIN</td><td>EMAX</td><td>ARMOR</td></tr><tr><td rowspan="5">CIFAR-10</td><td>VGG-16</td><td>93.86%</td><td>54.41%</td><td>65.80%</td><td>27.23%</td></tr><tr><td>ResNet-18</td><td>94.49%</td><td>41.45%</td><td>51.71%</td><td>25.48%</td></tr><tr><td>ResNet-50</td><td>94.52%</td><td>51.74%</td><td>47.30%</td><td>23.68%</td></tr><tr><td>DenseNet-121</td><td>95.38%</td><td>45.79%</td><td>51.41%</td><td>26.58%</td></tr><tr><td>WRN_34_10</td><td>96.54%</td><td>37.14%</td><td>55.06%</td><td>25.45%</td></tr><tr><td rowspan="5">CIFAR-100</td><td>VGG-16</td><td>65.06%</td><td>14.98%</td><td>62.26%</td><td>13.50%</td></tr><tr><td>ResNet-18</td><td>75.01%</td><td>24.53%</td><td>59.64%</td><td>22.48%</td></tr><tr><td>ResNet-50</td><td>73.42%</td><td>19.17%</td><td>49.71%</td><td>13.17%</td></tr><tr><td>DenseNet-121</td><td>76.06%</td><td>14.06%</td><td>51.02%</td><td>12.09%</td></tr><tr><td>WRN_34_10</td><td>79.25%</td><td>16.36%</td><td>48.88%</td><td>13.59%</td></tr><tr><td rowspan="5">Mini-ImageNet</td><td>ResNet-18</td><td>77.66%</td><td>22.68%</td><td>44.26%</td><td>7.67%</td></tr><tr><td>ResNet-50</td><td>77.87%</td><td>22.48%</td><td>46.92%</td><td>8.19%</td></tr><tr><td>DenseNet-121</td><td>70.50%</td><td>24.33%</td><td>38.35%</td><td>6.42%</td></tr><tr><td>EfficientNet</td><td>72.53%</td><td>18.56%</td><td>42.22%</td><td>4.06%</td></tr><tr><td>ResNext-50</td><td>74.30%</td><td>29.97%</td><td>44.59%</td><td>8.43%</td></tr><tr><td rowspan="5">VGG-Face</td><td>ResNet-18</td><td>97.18%</td><td>19.87%</td><td>21.25%</td><td>4.13%</td></tr><tr><td>ResNet-50</td><td>97.21%</td><td>26.22%</td><td>33.22%</td><td>5.56%</td></tr><tr><td>DenseNet-121</td><td>96.02%</td><td>17.15%</td><td>22.98%</td><td>9.36%</td></tr><tr><td>ResNext-50</td><td>93.77%</td><td>28.54%</td><td>33.15%</td><td>1.28%</td></tr><tr><td>EfficientNet</td><td>97.29%</td><td>10.86%</td><td>18.28%</td><td>2.72%</td></tr></table>

TABLE VIII ABLATION STUDY IN THE NON-DATA-AUGMENTATION SCENARIO.

<table><tr><td>Dataset</td><td>Model</td><td>Unprotected</td><td>Base Noise</td><td>Base+ Non-local</td><td>Base+Non-local+ Adaptive_Stepsize</td></tr><tr><td rowspan="5">CIFAR-10</td><td>VGG-16</td><td>92.66%</td><td>24.54%</td><td>16.75%</td><td>14.66%</td></tr><tr><td>ResNet-18</td><td>94.09%</td><td>18.98%</td><td>15.43%</td><td>12.85%</td></tr><tr><td>ResNet-50</td><td>94.38%</td><td>14.54%</td><td>14.34%</td><td>11.74%</td></tr><tr><td>DenseNet-121</td><td>94.89%</td><td>18.14%</td><td>16.96%</td><td>11.09%</td></tr><tr><td>WRN_34_10</td><td>95.52%</td><td>17.55%</td><td>16.90%</td><td>11.94%</td></tr><tr><td rowspan="5">CIFAR-100</td><td>VGG-16</td><td>69.43%</td><td>13.30%</td><td>12.03%</td><td>9.35%</td></tr><tr><td>ResNet-18</td><td>71.84%</td><td>20.60%</td><td>15.25%</td><td>12.62%</td></tr><tr><td>ResNet-50</td><td>71.69%</td><td>16.94%</td><td>11.96%</td><td>11.63%</td></tr><tr><td>DenseNet-121</td><td>73.04%</td><td>23.43%</td><td>17.79%</td><td>13.23%</td></tr><tr><td>WRN_34_10</td><td>75.77%</td><td>20.11%</td><td>16.20%</td><td>7.46%</td></tr><tr><td rowspan="5">Mini-ImageNet</td><td>ResNet-18</td><td>75.69%</td><td>9.55%</td><td>6.03%</td><td>3.72%</td></tr><tr><td>ResNet-50</td><td>74.56%</td><td>8.78%</td><td>4.99%</td><td>4.48%</td></tr><tr><td>DenseNet-121</td><td>68.34%</td><td>10.86%</td><td>6.00%</td><td>4.90%</td></tr><tr><td>EfficientNet</td><td>76.07%</td><td>12.97%</td><td>6.46%</td><td>2.25%</td></tr><tr><td>ResNext-50</td><td>71.11%</td><td>8.91%</td><td>4.49%</td><td>2.90%</td></tr><tr><td rowspan="5">VGG-Face</td><td>ResNet-18</td><td>93.59%</td><td>0.32%</td><td>0.29%</td><td>0.28%</td></tr><tr><td>ResNet-50</td><td>93.09%</td><td>0.54%</td><td>0.36%</td><td>0.19%</td></tr><tr><td>DenseNet-121</td><td>93.62%</td><td>0.60%</td><td>0.39%</td><td>0.25%</td></tr><tr><td>ResNext-50</td><td>88.72%</td><td>0.38%</td><td>0.33%</td><td>0.27%</td></tr><tr><td>EfficientNet</td><td>95.56%</td><td>0.45%</td><td>0.19%</td><td>0.17%</td></tr></table>

TABLE IX ABLATION STUDY IN THE DATA-AUGMENTATION SCENARIO.

<table><tr><td>Dataset</td><td>Model</td><td>Unprotected</td><td>Base Noise</td><td>Base+ Non-local</td><td>Base+Non-local+ Adaptive_Stepsize</td></tr><tr><td rowspan="5">CIFAR-10</td><td>VGG-16</td><td>93.86%</td><td>46.76%</td><td>40.89%</td><td>29.59%</td></tr><tr><td>ResNet-18</td><td>94.49%</td><td>42.24%</td><td>41.92%</td><td>28.84%</td></tr><tr><td>ResNet-50</td><td>94.52%</td><td>44.66%</td><td>40.62%</td><td>28.07%</td></tr><tr><td>DenseNet-121</td><td>95.28%</td><td>45.27%</td><td>43.56%</td><td>31.32%</td></tr><tr><td>WRN_34_10</td><td>96.54%</td><td>44.07%</td><td>43.66%</td><td>30.73%</td></tr><tr><td rowspan="5">CIFAR-100</td><td>VGG-16</td><td>65.06%</td><td>27.72%</td><td>27.31%</td><td>22.60%</td></tr><tr><td>ResNet-18</td><td>75.01%</td><td>32.25%</td><td>29.11%</td><td>25.10%</td></tr><tr><td>ResNet-50</td><td>73.42%</td><td>33.21%</td><td>24.43%</td><td>24.04%</td></tr><tr><td>DenseNet-121</td><td>76.06%</td><td>30.29%</td><td>29.67%</td><td>26.48%</td></tr><tr><td>WRN_34_10</td><td>79.25%</td><td>34.15%</td><td>28.98%</td><td>26.40%</td></tr><tr><td rowspan="5">Mini-ImageNet</td><td>ResNet-18</td><td>77.66%</td><td>25.34%</td><td>18.64%</td><td>11.69%</td></tr><tr><td>ResNet-50</td><td>77.87%</td><td>26.90%</td><td>25.17%</td><td>14.27%</td></tr><tr><td>DenseNet-121</td><td>70.50%</td><td>24.90%</td><td>19.39%</td><td>11.96%</td></tr><tr><td>EfficientNet</td><td>72.53%</td><td>24.67%</td><td>24.10%</td><td>13.67%</td></tr><tr><td>ResNext-50</td><td>74.30%</td><td>21.88%</td><td>14.39%</td><td>12.80%</td></tr><tr><td rowspan="5">VGG-Face</td><td>ResNet-18</td><td>97.18%</td><td>27.68%</td><td>26.43%</td><td>23.10%</td></tr><tr><td>ResNet-50</td><td>97.21%</td><td>33.69%</td><td>33.54%</td><td>31.36%</td></tr><tr><td>DenseNet-121</td><td>96.02%</td><td>29.13%</td><td>26.89%</td><td>26.11%</td></tr><tr><td>ResNext-50</td><td>93.77%</td><td>30.45%</td><td>26.65%</td><td>20.79%</td></tr><tr><td>EfficientNet</td><td>97.29%</td><td>21.76%</td><td>20.99%</td><td>8.97%</td></tr></table>

We also compare ARMOR with baselines in terms of classwise noise. The comparison results are shown in Table VI and Table VII. It is shown that ARMOR can also achieve the best protection ability in most cases for all datasets compared with state-of-the-art class-wise noise generation methods, especially for the data-augmentation scenario. We also discovered that class-wise noise is more effective than sample-wise noise in decreasing the model test accuracy. This discrepancy can be attributed to the explicit correlation exhibited by class-wise noise with the corresponding label. By learning this correlation, the model can effectively reduce training errors. Consequently, the model becomes unintentionally focused on learning the noise itself instead of capturing the true underlying content. However, class-wise noise is shown to be more likely to be exposed [19]. In the case of sample-wise noise, each individual sample is subjected to a distinct noise pattern, and there is no explicit correlation between the noise and the corresponding label. In this scenario, the model only tends to disregard lowerror samples while attaching more importance to normal and high-error examples.

TABLE X EFFECTIVENSS OF ARMOR AND BASELINES UNDER DIFFERENT DATA AUGMENTATION STRATEGIES.

<table><tr><td>Dataset</td><td>Method</td><td>Unprotected</td><td>EMIN</td><td>CTAP</td><td>NTGA</td><td>REM</td><td>ARMOR</td></tr><tr><td rowspan="5">CIFAR-10</td><td>FastAA</td><td>94.81%</td><td>44.00%</td><td>42.71%</td><td>86.96%</td><td>38.40%</td><td>22.91%</td></tr><tr><td>Mixup</td><td>95.03%</td><td>33.65%</td><td>32.78%</td><td>44.66%</td><td>31.58%</td><td>21.15%</td></tr><tr><td>PuzzleMix</td><td>95.37%</td><td>40.52%</td><td>45.53%</td><td>34.85%</td><td>40.43%</td><td>17.93%</td></tr><tr><td> $FD^1$ </td><td>92.91%</td><td>31.17%</td><td>68.11%</td><td>81.58%</td><td>32.67%</td><td>16.13%</td></tr><tr><td>DeepAA</td><td>94.49%</td><td>56.05%</td><td>40.20%</td><td>90.71%</td><td>43.25%</td><td>28.84%</td></tr><tr><td>Dataset</td><td>Method</td><td>Unprotected</td><td>EMIN</td><td>CTAP</td><td>NTGA</td><td>REM</td><td>ARMOR</td></tr><tr><td rowspan="5">CIFAR-100</td><td>FastAA</td><td>73.64%</td><td>32.27%</td><td>64.92%</td><td>51.47%</td><td>24.72%</td><td>17.12%</td></tr><tr><td>Mixup</td><td>77.19%</td><td>18.02%</td><td>64.69%</td><td>28.74%</td><td>17.61%</td><td>14.73%</td></tr><tr><td>PuzzleMix</td><td>68.65%</td><td>28.57%</td><td>62.73%</td><td>24.37%</td><td>21.12%</td><td>12.55%</td></tr><tr><td> $FD^1$ </td><td>68.48%</td><td>22.08%</td><td>60.39%</td><td>54.69%</td><td>25.05%</td><td>21.57%</td></tr><tr><td>DeepAA</td><td>75.01%</td><td>39.92%</td><td>64.91%</td><td>52.30%</td><td>27.07%</td><td>22.60%</td></tr><tr><td>Dataset</td><td>Method</td><td>Unprotected</td><td>EMIN</td><td>CTAP</td><td>NTGA</td><td>REM</td><td>ARMOR</td></tr><tr><td rowspan="5">Mini-ImageNet</td><td>FastAA</td><td>76.60%</td><td>18.94%</td><td>50.32%</td><td>58.49%</td><td>38.73%</td><td>13.77%</td></tr><tr><td>Mixup</td><td>76.89%</td><td>17.79%</td><td>44.97%</td><td>50.77%</td><td>36.98%</td><td>5.42%</td></tr><tr><td>PuzzleMix</td><td>76.46%</td><td>18.14%</td><td>60.64%</td><td>55.78%</td><td>64.95%</td><td>5.02%</td></tr><tr><td> $FD^1$ </td><td>77.11%</td><td>21.40%</td><td>46.12%</td><td>58.80%</td><td>36.13%</td><td>11.10%</td></tr><tr><td>DeepAA</td><td>77.66%</td><td>58.06%</td><td>63.26%</td><td>64.28%</td><td>44.46%</td><td>11.69%</td></tr><tr><td>Dataset</td><td>Method</td><td>Unprotected</td><td>EMIN</td><td>CTAP</td><td>NTGA</td><td>REM</td><td>ARMOR</td></tr><tr><td rowspan="5">VGG-Face</td><td>FastAA</td><td>94.97%</td><td>20.08%</td><td>72.78%</td><td>93.59%</td><td>15.74%</td><td>3.30%</td></tr><tr><td>Mixup</td><td>94.49%</td><td>7.96%</td><td>65.91%</td><td>90.12%</td><td>6.76%</td><td>0.57%</td></tr><tr><td>PuzzleMix</td><td>94.36%</td><td>8.78%</td><td>66.36%</td><td>91.62%</td><td>5.87%</td><td>0.65%</td></tr><tr><td> $FD^1$ </td><td>95.40%</td><td>8.65%</td><td>67.84%</td><td>92.92%</td><td>7.34%</td><td>0.65%</td></tr><tr><td>DeepAA</td><td>97.18%</td><td>42.63%</td><td>80.66%</td><td>94.35%</td><td>36.86%</td><td>23.10%</td></tr></table>

1 Short for Feature distillation.

TABLE XI IMPACT OF SURROGATE MODEL STRUCTURE ON PROTECTION ABILITY IN THE NON-DATA-AUGMENTATION SCENARIO.

| Dataset | Unprotected | VGG-16 | ResNet-18 | ResNet-50 | DenseNet-121 |
| --- | --- | --- | --- | --- | --- |
| CIFAR-10 | 94.09% | 22.79% | 12.85% | 22.61% | 26.82% |
| CIFAR-100 | 71.84% | 11.11% | 12.62% | 13.65% | 16.39% |
| Dataset | Unprotected | ResNet-18 | ResNet-50 | DenseNet-121 | ResNext-50 |
| Mini-ImageNet | 75.69% | 3.72% | 9.06% | 10.95% | 8.95% |
| Dataset | Unprotected | ResNet-18 | ResNet-50 | DenseNet-121 | EffientNet |
| VGG-Face | 93.59% | 0.28% | 0.54% | 0.46% | 0.43% |

To explore the invisibility of the generated noise, we visualize the results of different types of defensive noise and the corresponding examples in Figure 2. We can see the unlearnable samples generated by ARMOR retain a natural appearance, preserving the usability of the original image.

TABLE XII IMPACT OF SURROGATE MODEL STRUCTURE ON PROTECTION ABILITY IN THE DATA-AUGMENTATION SCENARIO.

| Dataset | Unprotected | VGG-16 | ResNet-18 | ResNet-50 | DenseNet-121 |
| --- | --- | --- | --- | --- | --- |
| CIFAR-10 | 94.49% | 29.82% | 30.84% | 29.49% | 30.41% |
| CIFAR-100 | 75.01% | 24.26% | 25.10% | 27.50% | 29.68% |
| Dataset | Unprotected | ResNet-18 | ResNet-50 | DenseNet-121 | ResNext-50 |
| Mini-ImageNet | 77.66% | 11.69% | 26.12% | 25.78% | 18.76% |
| Dataset | Unprotected | ResNet-18 | ResNet-50 | DenseNet-121 | EffientNet |
| VGG-Face | 97.18% | 23.10% | 28.65% | 29.03% | 23.33% |

TABLE XIII IMPACT OF ϵ ON THE EFFECTIVENESS OF ARMOR IN THE NON-DATA-AUGMENTATION SCENARIO.

| Dataset | Unprotected | $\epsilon = 4$ | $\epsilon = 8$ | $\epsilon = 12$ | $\epsilon = 16$ |
| --- | --- | --- | --- | --- | --- |
| CIFAR-10 | 94.09% | 14.03% | 12.85% | 10.41% | 9.87% |
| CIFAR-100 | 71.84% | 42.17% | 12.62% | 8.95% | 8.05% |
| Mini-ImageNet | 75.69% | 9.27% | 3.72% | 3.50% | 3.02% |
| VGG-Face | 93.59% | 2.45% | 0.65% | 0.46% | 0.28% |

## B. Ablation Study

In this section, we conduct an ablation study to examine the necessity of the base augmentation-resistant noise generation framework, the non-local module, and the dynamic step size adjustment algorithm. The results are shown in Table VIII and Table IX. The column of “Unprotected” represents the model test accuracy on unprotected data. The column of “Base Noise” represents the test accuracy on protected data with only base noise of ARMOR without non-local module and dynamic step size adjustment. The “Base+Non-local” method uses the non-local module. The “Base+Non-local+Adaptive Stepsize” method is the complete data protection method of ARMOR. In this section, we use the sample-wise noise.

In both non-data-augmentation and data-augmentation scenarios, comparing “Unprotected” and “Base Noise”, we can observe that our base noise generation framework can significantly decrease the model test accuracy. For example, when using the VGG-16 model, the clean test accuracy is 93.86% for CIFAR-10 with data augmentation but reaches as low as 46.76% using the “Base Noise” protection method. The reduction is more than 47.10%.

Compared with “Base Noise”, the “Base+Non-local” method further decreases the model test accuracy, which shows the effectiveness of the non-local module. The success of the non-local module is that it encourages the surrogate model to also focus on information from other areas and avoid overemphasizing localized features.

It is shown that the dynamic step size adjustment algorithm can also further decrease the model test accuracy in almost all cases for all datasets. For example, when using the VGG-16 model, the “Base+Non-local+Adaptive Stepsize” protection method can further decrease test accuracy from 40.89% to 29.59% under the data augmentation scenario. The reduction is as high as 11.3%.

TABLE XIV IMPACT OF ϵ ON THE EFFECTIVENESS OF ARMOR IN THE DATA-AUGMENTATION SCENARIO.

| Dataset | Unprotected | $\epsilon = 4$ | $\epsilon = 8$ | $\epsilon = 12$ | $\epsilon = 16$ |
| --- | --- | --- | --- | --- | --- |
| CIFAR-10 | 94.49% | 33.56% | 30.84% | 30.50% | 29.05% |
| CIFAR-100 | 75.01% | 60.07% | 25.10% | 23.68% | 19.60% |
| Mini-ImageNet | 77.66% | 28.50% | 11.69% | 10.05% | 9.04% |
| VGG-Face | 97.18% | 95.60% | 70.68% | 55.56% | 23.10% |

## C. Performance under Different Data Augmentation Methods

We explore the effectiveness of ARMOR and baselines when the model trainer adopts other state-of-the-art data augmentation strategies, such as Mixup [53], Feature distillation [27], PuzzleMix [21], and FastAA [25]. Given that CTAP surpasses UTAP in data protection, we exclusively showcase the CTAP results here. The results are shown in Table X.

We can see that various data augmentation strategies can make the protected data learnable again, especially for TAP and NTGA. While EMIN and REM occasionally demonstrate resilience to data augmentation (i.e., reduce the test accuracy to less than 30%), strategies like FastAA and PuzzMix can successfully improve the model test accuracy to larger than 40% even 60% for these methods. In comparison, ARMOR can successfully decrease the model test accuracy to less than 30% (in most cases less than 20%) for all the list advanced data augmentation methods for all datasets.

## D. Impact of Surrogate Model Structure

By default, the surrogate model for the four datasets employs the ResNet-18 architecture. We explore whether ARMOR is also effective when using other surrogate model structures, such as VGG-16, ResNet-50, DenseNet-121, EfficientNet, and ResNext-50. Note that we set the commercial unauthorized model structure as ResNet-18.

The results are shown in Table XI and Table XII. It is shown that ARMOR is robust to the structures of the surrogate model. ARMOR can effectively reduce the test accuracy of the unauthorized commercial model regardless of the surrogate model structure.

## E. Impact of the ϵ Value

In ARMOR, we use the first-order optimization method PGD [29] to update the noise δ. In PGD, ϵ is the maximum perturbation allowed for the noise. We vary ϵ values and investigate its impact on ARMOR. The results are shown in Table XIII and Table XIV.

A higher ϵ offers enhanced protection but can make noise more noticeable. The choice of ϵ involves a trade-off between protection effectiveness and concealment. Extensive experimentation has revealed that an ϵ of 8 is ideal for CIFAR-10, CIFAR-100, and Mini-ImageNet datasets, making the noise nearly invisible. For the VGG-Face dataset, we found that a ϵ of 16 generates sufficiently natural images and can achieve the defense goal. The need for increased noise in the VGG-Face dataset arises from its characteristics. Images in VGG-Face are close together within the same category (inter-class distances) but far from different ones (inter-class distances). As a result, the VGG-Face dataset demonstrates a higher level of linear separability, leading to high classification accuracy [37]. To effectively confuse the features between different categories, a higher ϵ is essential.

TABLE XV IMPACT OF THE NUMBER OF PROTECTION LABELS ON THE PROTECTION ABILITY IN NON-DATA-AUGMENTATION SCENARIOS.

<table><tr><td rowspan="2">Dataset</td><td rowspan="2">Protected class</td><td rowspan="2">Test accuracy of #</td><td colspan="3">Sample-wise</td><td colspan="3">Class-wise</td></tr><tr><td>EMAX</td><td>EMIN</td><td>ARMOR</td><td>EMAX</td><td>EMIN</td><td>ARMOR</td></tr><tr><td rowspan="6">CIFAR-10</td><td rowspan="2">Class 0 ~ 1</td><td>Unprotected label</td><td>94.50%</td><td>94.41%</td><td>94.24%</td><td>94.49%</td><td>94.36%</td><td>94.71%</td></tr><tr><td>Protected label</td><td>10.85%</td><td>7.65%</td><td>5.35%</td><td>3.40%</td><td>11.30%</td><td>0.60%</td></tr><tr><td rowspan="2">Class 0 ~ 4</td><td>Unprotected label</td><td>97.72%</td><td>97.22%</td><td>97.48%</td><td>97.42%</td><td>97.28%</td><td>97.80%</td></tr><tr><td>Protected label</td><td>2.75%</td><td>2.96%</td><td>1.24%</td><td>0.36%</td><td>1.32%</td><td>0.08%</td></tr><tr><td rowspan="2">Class 0 ~ 7</td><td>Unprotected label</td><td>98.60%</td><td>98.20%</td><td>98.25%</td><td>98.50%</td><td>98.60%</td><td>98.70%</td></tr><tr><td>Protected label</td><td>1.84%</td><td>1.48%</td><td>0.44%</td><td>0.89%</td><td>0.68%</td><td>0.05%</td></tr><tr><td rowspan="8">CIFAR-100</td><td rowspan="2">Class 0 ~ 9</td><td>Unprotected label</td><td>72.12%</td><td>71.53%</td><td>72.93%</td><td>72.12%</td><td>72.30%</td><td>71.71%</td></tr><tr><td>Protected label</td><td>18.36%</td><td>22.90%</td><td>20.50%</td><td>29.40%</td><td>10.80%</td><td>9.40%</td></tr><tr><td rowspan="2">Class 0 ~ 19</td><td>Unprotected label</td><td>74.78%</td><td>71.60%</td><td>72.79%</td><td>74.48%</td><td>74.06%</td><td>73.98%</td></tr><tr><td>Protected label</td><td>14.10%</td><td>15.65%</td><td>12.10%</td><td>11.80%</td><td>4.60%</td><td>3.85%</td></tr><tr><td rowspan="2">Class 0 ~ 49</td><td>Unprotected label</td><td>78.02%</td><td>74.24%</td><td>77.54%</td><td>78.06%</td><td>78.40%</td><td>77.54%</td></tr><tr><td>Protected label</td><td>11.90%</td><td>9.48%</td><td>7.42%</td><td>12.26%</td><td>5.24%</td><td>3.02%</td></tr><tr><td rowspan="2">Class 0 ~ 79</td><td>Unprotected label</td><td>86.46%</td><td>83.35%</td><td>86.60%</td><td>89.05%</td><td>89.45%</td><td>87.95%</td></tr><tr><td>Protected label</td><td>5.88%</td><td>6.54%</td><td>5.52%</td><td>8.32%</td><td>4.37%</td><td>2.32%</td></tr><tr><td rowspan="6">Mini-ImageNet</td><td rowspan="2">Class 0 ~ 19</td><td>Unprotected label</td><td>73.56%</td><td>72.54%</td><td>73.47%</td><td>73.40%</td><td>73.79%</td><td>73.94%</td></tr><tr><td>Protected label</td><td>23.25%</td><td>28.05%</td><td>6.95%</td><td>27.60%</td><td>19.15%</td><td>19.60%</td></tr><tr><td rowspan="2">Class 0 ~ 49</td><td>Unprotected label</td><td>76.22%</td><td>76.44%</td><td>74.66%</td><td>76.76%</td><td>75.04%</td><td>73.80%</td></tr><tr><td>Protected label</td><td>21.40%</td><td>11.54%</td><td>3.64%</td><td>19.22%</td><td>0.48%</td><td>0.20%</td></tr><tr><td rowspan="2">Class 0 ~ 79</td><td>Unprotected label</td><td>86.15%</td><td>86.45%</td><td>83.40%</td><td>86.00%</td><td>85.10%</td><td>84.00%</td></tr><tr><td>Protected label</td><td>2.59%</td><td>4.61%</td><td>0.59%</td><td>7.84%</td><td>0.10%</td><td>0.08%</td></tr><tr><td rowspan="6">VGG-Face</td><td rowspan="2">Class 0 ~ 39</td><td>Unprotected label</td><td>94.61%</td><td>94.61%</td><td>94.44%</td><td>94.51%</td><td>94.39%</td><td>94.54%</td></tr><tr><td>Protected label</td><td>74.55%</td><td>9.77%</td><td>8.42%</td><td>0.29%</td><td>0.00%</td><td>0.00%</td></tr><tr><td rowspan="2">Class 0 ~ 99</td><td>Unprotected label</td><td>95.07%</td><td>94.82%</td><td>94.97%</td><td>94.92%</td><td>94.75%</td><td>94.75%</td></tr><tr><td>Protected label</td><td>71.99%</td><td>1.11%</td><td>0.98%</td><td>0.10%</td><td>0.08%</td><td>0.00%</td></tr><tr><td rowspan="2">Class 0 ~ 159</td><td>Unprotected label</td><td>95.98%</td><td>95.29%</td><td>94.43%</td><td>94.15%</td><td>94.22%</td><td>94.60%</td></tr><tr><td>Protected label</td><td>63.68%</td><td>0.01%</td><td>0.00%</td><td>0.06%</td><td>0.04%</td><td>0.00%</td></tr></table>

## F. Impact of Protection Label Number

We then explore the number of protection labels on the protection ability. In this case, defensive noise shields only a subset of labels, leaving samples from the remaining labels unprotected upon release. The test accuracy results of models trained on unprotected labels and the protected labels are shown in Table XV and Table XVI. As for both ARMOR and baselines, we can see that as the number of protection labels rises, our defensive capability improves, leading to a decrease in the test accuracy of the protected label.

## G. Time Cost of Baselines and ARMOR

We calculate the time costs for generating different types of defensive noise on different datasets. The results are shown in Table XVII. The format of $t _ { 1 } + t _ { 2 }$ represents that noise generation methods include two stages: model pre-training stage $( t _ { 1 } )$ and noise generation stage $( t _ { 2 } )$ . We can see that AR-MOR exhibits outstanding performance in terms of execution efficiency, surpassing several baselines such as TAP, NTGA, and REM by a significant margin. While ARMOR requires additional time compared to EMIN and EMAX, the noise generated by ARMOR exhibits significantly higher protection capabilities than that of EMIN and EMAX.

## H. Robustness to Adversarial Training

Adversarial training is a technique proposed to enhance the robustness of deep learning models against adversarial examples. The idea behind adversarial training is to augment the training process by including adversarial examples during the training phase. The standard adversarial training [29] addresses a min-max problem, where the objective is to minimize the loss function while the adversarial examples are present. In our study, we investigate the resilience of ARMOR against adversarial training by considering commonly-used adversarial training strategies, namely Super-convergence [40] and Propagate [52]. Super-convergence [40] is based on the adversarial examples generated by FGSM [24], and Propagate [52] is based on the adversarial examples generated by PGD [29]. We compared ARMOR with REM, whose defensive noise is specifically designed to resist adversarial training. The results are shown in Table XVIII.

Similarly to REM, we can see that ARMOR is also robust to adversarial training. ARMOR can effectively reduce the model test accuracy even if the model trainer adopts state-of-the-art adversarial training strategies. Take CIFAR as an example, after applying [29], the test accuracy decreases significantly from 93.74% to 12.90%, highlighting the robustness of AR-MOR against adversarial training.

## VII. CONCLUSION

This paper reveals the vulnerability of existing unlearnable examples to data augmentation, a widely-used pre-processing technique. To tackle this, we propose a novel unlearnable example generation framework, dubbed ARMOR, to safeguard data privacy against potential breaches arising from data augmentation. In ARMOR, we introduce a non-local moduleassisted surrogate model to overcome the difficulty of having no access to the model training process. Additionally, we design a surrogate augmentation strategy selection algorithm that maximizes distribution alignment between augmented and non-augmented samples. Moreover, we propose using an adaptive learning rate adjustment algorithm to improve the defensive noise generation process. Extensive experiments on various datasets, including CIFAR-10, CIFAR-100, Mini-ImageNet, and VGG-Face, verify the effectiveness and superiority of ARMOR in both sample-wise and class-wise noises. ARMOR has also demonstrated robustness to adversarial training.

TABLE XVI IMPACT OF THE NUMBER OF PROTECTION LABELS ON THE PROTECTION ABILITY IN THE DATA-AUGMENTATION SCENARIO.

<table><tr><td rowspan="2">Dataset</td><td rowspan="2">Protected class</td><td rowspan="2">Test accuracy of #</td><td colspan="3">Sample-wise</td><td colspan="3">Class-wise</td></tr><tr><td>EMAX</td><td>EMIN</td><td>ARMOR</td><td>EMAX</td><td>EMIN</td><td>ARMOR</td></tr><tr><td rowspan="6">CIFAR-10</td><td rowspan="2">Class 0 ~ 1</td><td>Unprotected label</td><td>92.46%</td><td>93.08%</td><td>93.05%</td><td>93.36%</td><td>92.89%</td><td>93.15%</td></tr><tr><td>Protected label</td><td>36.12%</td><td>50.80%</td><td>28.65%</td><td>16.54%</td><td>28.50%</td><td>11.50%</td></tr><tr><td rowspan="2">Class 0 ~ 4</td><td>Unprotected label</td><td>97.30%</td><td>96.64%</td><td>96.32%</td><td>97.14%</td><td>96.80%</td><td>96.52%</td></tr><tr><td>Protected label</td><td>31.25%</td><td>26.19%</td><td>7.40%</td><td>9.55%</td><td>7.40%</td><td>3.12%</td></tr><tr><td rowspan="2">Class 0 ~ 7</td><td>Unprotected label</td><td>98.50%</td><td>97.90%</td><td>97.45%</td><td>98.10%</td><td>97.80%</td><td>97.65%</td></tr><tr><td>Protected label</td><td>4.64%</td><td>13.20%</td><td>5.75%</td><td>6.08%</td><td>5.04%</td><td>1.34%</td></tr><tr><td rowspan="8">CIFAR-100</td><td rowspan="2">Class 0 ~ 9</td><td>Unprotected label</td><td>75.22%</td><td>74.62%</td><td>74.68%</td><td>74.89%</td><td>72.61%</td><td>75.74%</td></tr><tr><td>Protected label</td><td>21.60%</td><td>32.00%</td><td>22.40%</td><td>38.80%</td><td>14.50%</td><td>14.00%</td></tr><tr><td rowspan="2">Class 0 ~ 19</td><td>Unprotected label</td><td>76.26%</td><td>75.59%</td><td>76.11%</td><td>76.02%</td><td>76.29%</td><td>76.40%</td></tr><tr><td>Protected label</td><td>14.80%</td><td>25.95%</td><td>18.45%</td><td>26.95%</td><td>7.75%</td><td>7.25%</td></tr><tr><td rowspan="2">Class 0 ~ 49</td><td>Unprotected label</td><td>80.14%</td><td>77.50%</td><td>78.06%</td><td>80.10%</td><td>79.60%</td><td>79.16%</td></tr><tr><td>Protected label</td><td>12.23%</td><td>21.54%</td><td>9.32%</td><td>16.06%</td><td>8.22%</td><td>6.36%</td></tr><tr><td rowspan="2">Class 0 ~ 79</td><td>Unprotected label</td><td>87.56%</td><td>85.80%</td><td>85.70%</td><td>90.45%</td><td>87.94%</td><td>84.85%</td></tr><tr><td>Protected label</td><td>9.95%</td><td>18.59%</td><td>8.04%</td><td>13.38%</td><td>5.86%</td><td>3.71%</td></tr><tr><td rowspan="6">Mini-ImageNet</td><td rowspan="2">Class 0 ~ 19</td><td>Unprotected label</td><td>77.81%</td><td>77.61%</td><td>77.34%</td><td>77.89%</td><td>77.34%</td><td>77.09%</td></tr><tr><td>Protected label</td><td>30.14%</td><td>33.15%</td><td>12.00%</td><td>30.90%</td><td>33.60%</td><td>11.20%</td></tr><tr><td rowspan="2">Class 0 ~ 49</td><td>Unprotected label</td><td>80.20%</td><td>79.44%</td><td>79.82%</td><td>80.14%</td><td>80.14%</td><td>78.80%</td></tr><tr><td>Protected label</td><td>27.10%</td><td>25.38%</td><td>5.70%</td><td>25.28%</td><td>16.20%</td><td>5.76%</td></tr><tr><td rowspan="2">Class 0 ~ 79</td><td>Unprotected label</td><td>89.70%</td><td>88.35%</td><td>87.45%</td><td>88.65%</td><td>86.75%</td><td>87.20%</td></tr><tr><td>Protected label</td><td>20.00%</td><td>25.64%</td><td>2.50%</td><td>20.99%</td><td>4.68%</td><td>1.98%</td></tr><tr><td rowspan="6">VGG-Face</td><td rowspan="2">Class 0 ~ 39</td><td>Unprotected label</td><td>97.25%</td><td>97.46%</td><td>97.20%</td><td>97.41%</td><td>90.43%</td><td>97.34%</td></tr><tr><td>Protected label</td><td>47.17%</td><td>77.89%</td><td>36.38%</td><td>44.30%</td><td>62.75%</td><td>24.19%</td></tr><tr><td rowspan="2">Class 0 ~ 99</td><td>Unprotected label</td><td>97.63%</td><td>97.76%</td><td>97.81%</td><td>97.83%</td><td>97.77%</td><td>97.57%</td></tr><tr><td>Protected label</td><td>29.13%</td><td>72.55%</td><td>25.97%</td><td>22.84%</td><td>51.20%</td><td>22.06%</td></tr><tr><td rowspan="2">Class 0 ~ 159</td><td>Unprotected label</td><td>97.61%</td><td>98.27%</td><td>98.20%</td><td>97.47%</td><td>98.37%</td><td>97.47%</td></tr><tr><td>Protected label</td><td>20.83%</td><td>48.94%</td><td>15.02%</td><td>10.77%</td><td>39.31%</td><td>4.05%</td></tr></table>

TABLE XVII COMPUTING COST OF ARMOR AND BASELINES.

| Dataset | EMIN | EMAX | TAP | REM | NTGA | ARMOR |
| --- | --- | --- | --- | --- | --- | --- |
| CIFAR-10 | 0.56h | 0.30h+0.42h | 9.50h | 38.00h+2.50h | 17.67h | 0.50h+1.90h |
| CIFAR-100 | 1.50h | 0.50h+0.40h | 10.70h | 33.50h+3.50h | 18.56h | 0.65h+3.13h |
| Mini-ImageNet | 3.50h | 2.24h+2.08h | 16.00h | 27.00h+22.00h | 32.50h | 1.10h+6.12h |
| VGG-Face | 5.60h | 2.64h+2.88h | 13.00h | 18.75h+34.00h | 38.35h | 1.35h+10.25h |

TABLE XVIII EFFECTIVENESS OF ARMOR AND REM UNDER ADVERSARIAL TRAINING.

<table><tr><td>Dataset</td><td>Method</td><td> $\rho^{*}$ </td><td>Unprotected</td><td>REM</td><td>ARMOR</td></tr><tr><td rowspan="3">CIFAR-10</td><td>Super-convergence</td><td>-</td><td>88.20%</td><td>43.24%</td><td>14.16%</td></tr><tr><td rowspan="2">Propagate</td><td>1</td><td>93.74%</td><td>10.10%</td><td>12.90%</td></tr><tr><td>2</td><td>91.16%</td><td>31.47%</td><td>34.38%</td></tr><tr><td rowspan="3">CIFAR-100</td><td>Super-convergence</td><td>-</td><td>66.56%</td><td>14.83%</td><td>26.54%</td></tr><tr><td rowspan="2">Propagate</td><td>1</td><td>72.10%</td><td>12.06%</td><td>11.75%</td></tr><tr><td>2</td><td>68.89%</td><td>14.69%</td><td>28.18%</td></tr><tr><td rowspan="3">Mini-ImageNet</td><td>Super-convergence</td><td>-</td><td>69.18%</td><td>6.78%</td><td>9.32%</td></tr><tr><td>Propagate</td><td>1</td><td>73.42%</td><td>3.84%</td><td>6.90%</td></tr><tr><td>Propagate</td><td>2</td><td>70.26%</td><td>4.44%</td><td>8.17%</td></tr><tr><td rowspan="3">VGG-Face</td><td>Super-convergence</td><td>-</td><td>94.66%</td><td>70.77%</td><td>32.35%</td></tr><tr><td rowspan="2">Propagate</td><td>1</td><td>91.68%</td><td>17.72%</td><td>35.26%</td></tr><tr><td>2</td><td>92.93%</td><td>27.19%</td><td>35.72%</td></tr></table>

∗ The hyperparameter $\rho$ refers to the perturbation radius of PGD attack.

## REFERENCES

[1] Mohammad Al-Rubaie and J Morris Chang. Privacy-preserving machine learning: Threats and solutions. IEEE Security & Privacy, 17(2):49–58, 2019.
[2] Eitan Borgnia, Valeriia Cherepanova, Liam Fowl, Amin Ghiasi, Jonas Geiping, Micah Goldblum, Tom Goldstein, and Arjun Gupta. Strong data augmentation sanitizes poisoning and backdoor attacks without an accuracy tradeoff. In ICASSP 2021-2021 IEEE International Conference on Acoustics, Speech and Signal Processing, pages 3855–3859. IEEE, 2021.
[3] Nicholas Carlini, Steve Chien, Milad Nasr, Shuang Song, Andreas Terzis, and Florian Tramer. Membership inference attacks from first principles. In IEEE Symposium on Security and Privacy, pages 1897–1914, 2022.
[4] Shuxiao Chen, Edgar Dobriban, and Jane H Lee. A group-theoretic framework for data augmentation. The Journal of Machine Learning Research, 21(1):9885–9955, 2020.
[5] Valeriia Cherepanova, Micah Goldblum, Harrison Foley, Shiyuan Duan, John Dickerson, Gavin Taylor, and Tom Goldstein. Lowkey: Leveraging adversarial attacks to protect social media users from facial recognition. arXiv preprint arXiv:2101.07922, 2021.
[6] Ekin D Cubuk, Barret Zoph, Dandelion Mane, Vijay Vasudevan, and Quoc V Le. Autoaugment: Learning augmentation strategies from data. In IEEE/CVF Conference on Computer Vision and Pattern Recognition, pages 113–123, 2019.
[7] Ekin D Cubuk, Barret Zoph, Jonathon Shlens, and Quoc V Le. Randaugment: Practical automated data augmentation with a reduced search space. In IEEE/CVF Conference on Computer Vision and Pattern Recognition Workshops, pages 702–703, 2020.
[8] fast.ai. Imagenette: A smaller subset of 10 easily classified classes from imagenet. Available: https://github.com/fastai/imagenette.
[9] Liam Fowl, Ping-yeh Chiang, Micah Goldblum, Jonas Geiping, Arpit Bansal, Wojtek Czaja, and Tom Goldstein. Preventing unauthorized use
of proprietary data: Poisoning for secure dataset release. arXiv preprint arXiv:2103.02683, 2021.
[10] Liam Fowl, Micah Goldblum, Ping-yeh Chiang, Jonas Geiping, Wojciech Czaja, and Tom Goldstein. Adversarial examples make strong poisons. Advances in Neural Information Processing Systems, 34:30339–30351, 2021.
[11] Matt Fredrikson, Somesh Jha, and Thomas Ristenpart. Model inversion attacks that exploit confidence information and basic countermeasures. In ACM SIGSAC Conference on Computer and Communications Security, pages 1322–1333, 2015.
[12] Shaopeng Fu, Fengxiang He, Yang Liu, Li Shen, and Dacheng Tao. Robust unlearnable examples: Protecting data against adversarial learning. In International Conference on Learning Representations, 2022.
[13] Eric Goldman. An introduction to the california consumer privacy act (ccpa). Santa Clara Univ. Legal Studies Research Paper, 2020.
[14] Neil Zhenqiang Gong and Bin Liu. Attribute inference attacks in online social networks. ACM Transactions on Privacy and Security, 21(1):1–30, 2018.
[15] Xueluan Gong, Yanjiao Chen, Qian Wang, Meng Wang, and Shuyang Li. Private data inference attacks against cloud: Model, technologies, and research directions. IEEE Communications Magazine, 60(9):46–52, 2022.
[16] Xueluan Gong, Ziyao Wang, Yanjiao Chen, Qian Wang, Cong Wang, and Chao Shen. Netguard: Protecting commercial web apis from model inversion attacks using gan-generated fake samples. In ACM Web Conference, pages 2045–2053, 2023.
[17] Ryuichiro Hataya, Jan Zdenek, Kazuki Yoshizoe, and Hideki Nakayama. Faster autoaugment: Learning augmentation strategies using backpropagation. In European Conference on Computer Vision, pages 1–16. Springer, 2020.
[18] Shengshan Hu, Xiaogeng Liu, Yechao Zhang, Minghui Li, Leo Yu Zhang, Hai Jin, and Libing Wu. Protecting facial privacy: generating adversarial identity masks via style-robust makeup transfer. In IEEE/CVF Conference on Computer Vision and Pattern Recognition, pages 15014–15023, 2022.
[19] Hanxun Huang, Xingjun Ma, Sarah Monazam Erfani, James Bailey, and Yisen Wang. Unlearnable examples: Making personal data unexploitable. In International Conference on Learning Representations, 2021.
[20] Zhichao Huang, Yanbo Fan, Chen Liu, Weizhong Zhang, Yong Zhang, Mathieu Salzmann, Sabine Susstrunk, and Jue Wang. Fast adversarial ¨ training with adaptive step size, 2022.
[21] Jang-Hyun Kim, Wonho Choo, and Hyun Oh Song. Puzzle mix: Exploiting saliency and local statistics for optimal mixup. In International Conference on Machine Learning, pages 5275–5285. PMLR, 2020.
[22] Nishat Koti, Mahak Pancholi, Arpita Patra, and Ajith Suresh. Swift: Super-fast and robust privacy-preserving machine learning. In USENIX Security Symposium, pages 2651–2668, 2021.
[23] Alex Krizhevsky and Geoffrey Hinton. Learning multiple layers of features from tiny images. 2009.
[24] Alexey Kurakin, Ian Goodfellow, and Samy Bengio. Adversarial machine learning at scale. In International Conference on Learning Representations, 2017.
[25] Sungbin Lim, Ildoo Kim, Taesup Kim, Chiheon Kim, and Sungwoong Kim. Fast autoaugment. In Advances in Neural Information Processing Systems, 2019.
[26] Zhuoran Liu, Zhengyu Zhao, Alex Kolmus, Tijn Berns, Twan van Laarhoven, Tom Heskes, and Martha Larson. Going grayscale: The road to understanding and improving unlearnable examples. arXiv preprint arXiv:2111.13244, 2021.
[27] Zihao Liu, Qi Liu, Tao Liu, Nuo Xu, Xue Lin, Yanzhi Wang, and Wujie Wen. Feature distillation: DNN-oriented JPEG compression against adversarial examples. In Conference on Computer Vision and Pattern Recognition, pages 860–868. IEEE, 2019.
[28] Shaohao Lu, Yuqiao Xian, Ke Yan, Yi Hu, Xing Sun, Xiaowei Guo, Feiyue Huang, and Wei-Shi Zheng. Discriminator-free generative adversarial attack. In ACM International Conference on Multimedia, pages 1544–1552, 2021.
[29] Aleksander Madry, Aleksandar Makelov, Ludwig Schmidt, Dimitris Tsipras, and Adrian Vladu. Towards deep learning models resistant to adversarial attacks. In International Conference on Learning Representations, 2018.
[30] Kiran Maharana, Surajit Mondal, and Bhushankumar Nemade. A review: Data pre-processing and data augmentation techniques. Global Transitions Proceedings, 3(1):91–99, 2022.
[31] Maryam M Najafabadi, Flavio Villanustre, Taghi M Khoshgoftaar, Naeem Seliya, Randall Wald, and Edin Muharemagic. Deep learning
applications and challenges in big data analytics. Journal of Big Data, 2(1):1–21, 2015.
[32] Omkar M Parkhi, Andrea Vedaldi, and Andrew Zisserman. Deep face recognition. In British Machine Vision Conference, pages 41.1–41.12. BMVA Press, 2015.
[33] Han Qiu, Yi Zeng, Shangwei Guo, Tianwei Zhang, Meikang Qiu, and Bhavani Thuraisingham. Deepsweep: An evaluation framework for mitigating dnn backdoor attacks using data augmentation. In ACM Asia Conference on Computer and Communications Security, pages 363–377, 2021.
[34] Han Qiu, Yi Zeng, Tianwei Zhang, Yong Jiang, and Meikang Qiu. Fencebox: A platform for defeating adversarial examples with data augmentation techniques. arXiv preprint arXiv:2012.01701, 2020.
[35] Evani Radiya-Dixit, Sanghyun Hong, Nicholas Carlini, and Florian Tramer. Data poisoning won’t save you from facial recognition. \` arXiv preprint arXiv:2106.14851, 2021.
[36] Sylvestre-Alvise Rebuffi, Sven Gowal, Dan Andrei Calian, Florian Stimberg, Olivia Wiles, and Timothy A Mann. Data augmentation can improve robustness. Advances in Neural Information Processing Systems, 34:29935–29948, 2021.
[37] Jie Ren, Han Xu, Yuxuan Wan, Xingjun Ma, Lichao Sun, and Jiliang Tang. Transferable unlearnable examples, 2022.
[38] Shawn Shan, Emily Wenger, Jiayun Zhang, Huiying Li, Haitao Zheng, and Ben Y Zhao. Fawkes: Protecting privacy against unauthorized deep learning models. In USENIX Security Symposium, 2020.
[39] Reza Shokri, Marco Stronati, Congzheng Song, and Vitaly Shmatikov. Membership inference attacks against machine learning models. In IEEE Symposium on Security and Privacy, pages 3–18, 2017.
[40] Leslie N. Smith and Nicholay Topin. Super-convergence: Very fast training of neural networks using large learning rates, 2018.
[41] Chen Sun, Abhinav Shrivastava, Saurabh Singh, and Abhinav Gupta. Revisiting unreasonable effectiveness of data in deep learning era. In IEEE International Conference on Computer Vision, pages 843–852, 2017.
[42] Xinyu Tang, Saeed Mahloujifar, Liwei Song, Virat Shejwalkar, Milad Nasr, Amir Houmansadr, and Prateek Mittal. Mitigating membership inference attacks by self-distillation through a novel ensemble architecture. In USENIX Security Symposium, pages 1433–1450, 2022.
[43] Martin A Tanner and Wing Hung Wong. The calculation of posterior distributions by data augmentation. Journal of the American statistical Association, 82(398):528–540, 1987.
[44] David A Van Dyk and Xiao-Li Meng. The art of data augmentation. Journal of Computational and Graphical Statistics, 10(1):1–50, 2001.
[45] Paul Voigt and Axel Von dem Bussche. The eu general data protection regulation (gdpr). A Practical Guide, 1st Ed., Cham: Springer International Publishing, 10(3152676):10–5555, 2017.
[46] Gregory K Wallace. The JPEG still picture compression standard. IEEE Transactions on Consumer Electronics, 38(1):xviii–xxxiv, 1992.
[47] Xiaolong Wang, Ross Girshick, Abhinav Gupta, and Kaiming He. Non-local neural networks. In IEEE Conference on Computer Vision and Pattern Recognition, pages 7794–7803, 2018.
[48] Xizhao Wang, Yanxia Zhao, and Farhad Pourpanah. Recent advances in deep learning. International Journal of Machine Learning and Cybernetics, 11:747–750, 2020.
[49] Chong Xiang, Arjun Nitin Bhagoji, Vikash Sehwag, and Prateek Mittal. Patchguard: A provably robust defense against adversarial patches via small receptive fields and masking. In 30th USENIX Security Symposium, 2021.
[50] Chia-Hung Yuan and Shan-Hung Wu. Neural tangent generalization attacks. In International Conference on Machine Learning, pages 12230–12240. PMLR, 2021.
[51] Yi Zeng, Han Qiu, Gerard Memmi, and Meikang Qiu. A data augmentation-based defense method against adversarial attacks in neural networks. In International Conference on Algorithms and Architectures for Parallel Processing, pages 274–289. Springer, 2020.
[52] Dinghuai Zhang, Tianyuan Zhang, Yiping Lu, Zhanxing Zhu, and Bin Dong. You only propagate once: Accelerating adversarial training via maximal principle, 2019.
[53] Hongyi Zhang, Moustapha Cisse, Yann N Dauphin, and David Lopez-Paz. mixup: Beyond empirical risk minimization. arXiv preprint arXiv:1710.09412, 2017.
[54] Yu Zheng, Zhi Zhang, Shen Yan, and Mi Zhang. Deep autoaugment. In The Tenth International Conference on Learning Representations, 2022.

![](images/image_03.jpg)

<details>
<summary>natural_image</summary>

Portrait of a woman in formal business attire against a solid blue background (no text or symbols visible)
</details>

etc.

**Xueluan Gong** received her B.S. degree in Computer Science and Electronic Engineering from Hunan University in 2018. She received her Ph.D. degree in the School of Computer Science, Wuhan University, in 2023. Her research interests include network security and AI security. Her research has been published in multiple top-tier conferences and journals, such as IEEE S&P, USENIX Security, NDSS, WWW, IJCAI, IEEE JSAC, IEEE TDSC, and IEEE TIFS. She also served as the reviewer of ICML, NeurIPS, ICLR, IEEE TIFS, IEEE TDSC,

![](images/image_04.jpg)

<details>
<summary>natural_image</summary>

Portrait of a young woman wearing a white cardigan and bow tie against a solid blue background (no text or symbols visible)
</details>

**Mengyuan Sun** is currently pursuing the B.E. at the School of Cyber Science and Engineering from Wuhan University, China. Her research interest is AI security.

![](images/image_05.jpg)

<details>
<summary>natural_image</summary>

Portrait photo of a young man in formal suit and tie against a solid blue background (no text or symbols visible)
</details>

**Yuji Wang** is currently pursuing the B.E. at the School of Cyber Science and Engineering from Wuhan University, China. His research interests include AI security and information security.

![](images/image_06.jpg)

<details>
<summary>natural_image</summary>

Portrait photo of a young man against a solid red background (no text or symbols visible)
</details>

**Shuaike Li** is currently pursuing the B.E. at the School of Cyber Science and Engineering from Wuhan University, China. His research interest is AI security.

![](images/image_07.jpg)

<details>
<summary>natural_image</summary>

Portrait photo of a woman with long dark hair wearing a dark blazer (no text or symbols visible)
</details>

**Yanjiao Chen** received her B.E. degree in Electronic Engineering from Tsinghua University in 2010 and Ph.D. degree in Computer Science and Engineering from Hong Kong University of Science and Technology in 2015. She is currently a Bairen Researcher in Zhejiang University, China. Her research interests include spectrum management for Femtocell networks, network economics, network security, and Quality of Experience (QoE) of multimedia delivery/distribution.

![](images/image_08.jpg)

<details>
<summary>natural_image</summary>

Portrait of a man wearing glasses and a patterned shirt (no text or symbols visible)
</details>

**Qian Wang** is a Professor in the School of Cy ber Science and Engineering at Wuhan University, China. He was selected into the National High-level Young Talents Program of China, and listed among the World’s Top 2% Scientists by Stanford University. He also received the National Science Fund for Excellent Young Scholars of China in 2018. He has long been engaged in the research of cyberspace security, with focus on AI security, data outsourcing security and privacy, wireless systems security, and applied cryptography. He was a recipient of the 2018

![](images/image_09.jpg)

<details>
<summary>natural_image</summary>

Portrait of a young man wearing a dark blue polo shirt with a light blue striped collar (no text or symbols visible)
</details>

**Haocheng Dong** is currently pursuing the B.E. at the School of Cyber Science and Engineering from Wuhan University, China. His research interests include AI security and information security.

IEEE TCSC Award for Excellence in Scalable Computing (early career researcher) and the 2016 IEEE ComSoc Asia-Pacific Outstanding Young Researcher Award. He has published 200+ papers, with 120+ publications in top-tier international conferences, including USENIX NSDI, ACM CCS, USENIX Security, NDSS, ACM MobiCom, ICML, etc., with 20000+ Google Scholar citations. He is also a co-recipient of 8 Best Paper and Best Student Paper Awards from prestigious conferences, including ICDCS, IEEE ICNP, etc. In 2021, his PhD student was selected under Huawei’s “Top Minds” Recruitment Program. He serves as Associate Editors for IEEE Transactions on Dependable and Secure Computing (TDSC) and IEEE Transactions on Information Forensics and Security (TIFS). He is a fellow of the IEEE, and a member of the ACM.

![](images/image_10.jpg)

<details>
<summary>natural_image</summary>

Portrait of a man wearing glasses and a suit (no text or symbols visible)
</details>

**Yiming Li** is currently a Research Professor in the School of Cyber Science and Technology at Zhejiang University. Before that, he received his Ph.D. degree with honors in Computer Science and Technology from Tsinghua University (2023) and his B.S. degree with honors in Mathematics from Ningbo University (2018). His research interests are in the domain of Trustworthy ML and Responsible AI, especially backdoor learning and AI copyright protection. His research has been published in multi-ple top-tier conferences and journals, such as ICLR,

NeurIPS, and IEEE TIFS. He served as the Area Chair of ACM MM and Senior Program Committee Member of AAAI, and the reviewer of IEEE TPAMI, IEEE TIFS, IEEE TDSC, etc. His research has been featured by major media outlets, such as IEEE Spectrum. He was the recipient of the Best Paper Award at PAKDD and the Rising Star Award at WAIC.

![](images/image_11.jpg)

<details>
<summary>natural_image</summary>

Portrait of a man in formal attire with glasses against a blue background (no text or symbols visible)
</details>

**Chen Chen** received his Ph.D. degree in computer science from Nanyang Technological University, Singapore, in 2024, his Master of Computer Science from the University of New South Wales, Australia, in 2018, and his Bachelor degree from the University of Science and technology Beijing, China, in 2012. His research interests lie in the area of AI safety, Knowledge Graphs and Large Language Models.