# Robust Learning with Adversarial Perturbations and Label Noise: A Two-Pronged Defense Approach

Peng-Fei Zhang<sup>1</sup>, Zi Huang<sup>1</sup>, Xin Luo<sup>2</sup>, Pengfei Zhao<sup>2</sup>

<sup>1</sup>University of Queensland, <sup>2</sup>Shandong University

mima.zpf@gmail.com,huang@itee.uq.edu.au,{luoxin.lxin,lazpf1}@gmail.com

## ABSTRACT

Despite great success achieved, deep learning methods are vulnerable to noise in the training dataset, including adversarial perturbations and annotation noise. These harmful factors significantly influence the learning process of deep models, leading to less confident models. However, existing methods have not yet studied this practical and challenging issue.

In this paper, we propose a novel robust learning method, i.e., Two-Pronged Defense (TPD), which is capable of eliminating negative effects of both data perturbations and label noise during the learning process. On the one hand, to defend against delusive adversarial examples, the proposed method designs an asymmetric adversarial contrastive learning strategy to craft worse-case noisy example for original training data, and train the model to align the semantic between the perturbed data and the original data. In light of this, the TPD would be able to improve the generalization ability of the model on the potential adversarial examples. On the other hand, to combat noisy labels, the TPD applies semi-supervised learning by identifying and discarding noisy labels via a novel designed identification method. Extensive experiments on benchmarks demonstrate the incapability of existing methods and the effectiveness of the proposed method when facing both data and label noise. This work is the very first attempt in learning with data and label noise, and we hope it can pave the way for future studies in related fields.

## CCS CONCEPTS

• **Computing methodologies** → **Supervised learning**.

## KEYWORDS

Robust Learning; Adversarial Perturbations; Noisy Labels; Supervised Deep Learning

## ACM Reference Format:

Peng-Fei Zhang<sup>1</sup>, Zi Huang<sup>1</sup>, Xin Luo<sup>2</sup>, Pengfei Zhao<sup>2</sup>. 2022. Robust Learning with Adversarial Perturbations and Label Noise: A Two-Pronged Defense Approach. In ACM Multimedia Asia (MMAsia ’22), December 13–16, 2022, Tokyo, Japan. ACM, New York, NY, USA, 7 pages. [https://doi.org/10.1145/3551626.3564934](https://doi.org/10.1145/3551626.3564934)

Permission to make digital or hard copies of all or part of this work for personal or classroom use is granted without fee provided that copies are not made or distributed for profit or commercial advantage and that copies bear this notice and the full citation on the first page. Copyrights for components of this work owned by others than ACM must be honored. Abstracting with credit is permitted. To copy otherwise, or republish, to post on servers or to redistribute to lists, requires prior specific permission and/or a fee. Request permissions from permissions@acm.org.

MMAsia ’22, December 13–16, 2022, Tokyo, Japan

© 2022 Association for Computing Machinery.

ACM ISBN 978-1-4503-9478-9/22/12. . . \$15.00

[https://doi.org/10.1145/3551626.3564934](https://doi.org/10.1145/3551626.3564934)

## 1 INTRODUCTION

Deep learning techniques have recently sparked much attention in research communities due to its prominent non-linear feature learning ability, such as computer vision [5–7], information retrieval [22, 38, 39, 42, 44], natural language processing [4, 18] etc. The great success highly relies on massive training data of high quality. Nevertheless, in real-world applications, data are not always reliable, which might contain potential adversarial perturbations and annotation noise. On the one hand, malicious attackers would inject a set of perturbations into the original data, which would then adversely influence the testing-time behaviours of the models trained on these data (a.k.a. training-time attack) [10]. On the other hand, labelling massive data often resorts to web annotation and crowdsourcing considering workload and efficiency, which would inevitably introduce annotation errors [11, 32, 34]. These adverse factors would significantly influence the performance of resulted deep models, making it urgent to design robust learning strategies against data and label noise.

In the literature, there are attempts to deal with one of the above problems. For example, to defend against training-time attack, Tao et al. [33] performs adversarial training with worse-case data in a specific ∞-Wasserstein ball to improve the robustness of the tobe-learnt model. To combat noisy labels, many studies propose to construct the noise transition matrix to measure the probabilities of true labels being flipped into noisy ones, and use it to correct loss [12, 24, 27]. Another line of work is to learn to identify and accordingly discard noisy labels in the dataset, and conduct semi-supervised learning on the resulted dataset [17, 21, 26]. Despite progress achieved, these methods only concentrate on one type of noise in the dataset, i.e., adversarial noise or label noise, which are not applicable in complex scenarios, where both noise exist.

In this paper, we propose a robust learning method, i.e., Two-Pronged Defense (TPD), to deal with adversarial and label noise simultaneously. The two kinds of noise correspond to the input and target of model training, respectively. The proposed method attempts to eliminate the influence of them by considering the input and output. Specifically, to enhance the robustness of the model to the input noise, the proposed TPD endeavors to improve the generalization ability of the to-be-learnt model, so that the training trajectory would not be influenced by adversarial perturbations in data. To fulfil this, the proposed TPD leverages the well-designed asymmetric adversarial contrastive learning to craft worse-case adversarial training data within a specific ∞-Wasserstein ball, and trains the model to keep consistency between the generated adversarial data with the original data. In addition, the TPD learns worse-case data without the guidance of the label, which prevents the method from the negative influence of label noise. In terms of the label noise, the TPD proposes to filter out noisy labels by a novel designed sample selection strategy, and applies semi-supervised learning on the resulted dataset. Through the two-pronged strategy, the proposed method is able to mitigate the adverse influence of both the adversarial perturbations and label noise in the training data. As a result, a confident model would be obtained. Extensive experimental results on benchmark datasets demonstrate the effectiveness of the TPD in defending against two kinds of noise.

The main contributions of this work are summarized as follows:

• To the best of our knowledge, this is the first work that studies the practical and challenging case that both adversarial perturbations and label noise exist in training data. Furthermore, we empirically demonstrate the adverse influence of both noise in the training dataset.
• A novel two-pronged method is proposed, which improves the robustness of the to-be-learnt model by improving its generalization ability to the potential noises and dynamically filtering out unreliable supervision.
• A contrastive learning based asymmetric adversarial training is introduced to effectively generate worse-case data, which is not influenced by label noise. A novel sample selection method is designed to confidently sieve out noisy labels. In light of this, the proposed TPD can effectively combat various noise in the training data.
• Extensive experiments are performed on two widely used benchmarks, and the impressive results demonstrate the promising performance of the TPD.

## 2 RELATED WORK

## 2.1 Data poisoning attack and defense

Data poisoning attack aims to manipulate inference-time behaviours of models by injecting adversarial perturbations into training data [1, 36, 41, 43]. Earlier work mainly focus on non-deep method, such as SVM [25], Naive Bayes [2], Linear Regression [15]. With the prevalence of deep neural networks, poisoning attacks against deep models have received much attention. For example, Koh et al. [19] leverage influence functions to measure the impact of data on model predictions and accordingly crafts adversarial examples to poison deep neural models. Shafahi et al. [30] propose the targeted attack to fool the model on one specific test instance by designing an optimization-based method. Chen et al. [3] generate backdoor instances into the training dataset, so that the model which learns with them would misclassify the test backdoor data as a target label. These poisoning attacks only focus on modifying a few training samples.

Feng et al. [10] propose to inject imperceivable perturbations into the whole dataset for implementing the attack, i.e., training-time attack. The method does not change the label of data. A model that learns with these adversarial data would make wrong predictions on the normal data (a.k.a. clean data) while giving reliable predictions on the data with adversarial perturbations. This kind of attack is more dangerous, as it would not significantly change the model behaviours on the adversarial data, but only on the normal data. As a result, it is hard to be perceived in the training procedure. Unfortunately, there are few works proposed to deal with this challenging problem. [33] is the first defending method against data poisoning training-time attack, which applies adversarial training to learn the model with the generated worse-case data within a specific ∞-Wasserstein ball.

## 2.2 Learning with noisy labels

Designing robust models against label noise has been a long-standing research topic. In the literature, prior methods are mainly proposed to deal with the label-level (instance-independent) noise, where the label corruption only depends on the true labels. They commonly take measures to estimate the label transition matrix, which indicates the probability of labels being flipped to noisy ones. For instance, a line of work proposes to add a noise adaptation layer at the top of the original model to mimic the label transition matrix [12, 32]. Some consider to design robust loss functions to minimize the risk on unseen clean labels [11, 23, 34, 45]. There are also methods focusing on re-weighting the importance of data instances, attempting to reduce the adverse influence of noisy labels [16, 29]. Another strategy is to endeavor to identify clean labels and noisy labels during the training process and make the model to be trained with clean labels [8, 13, 17, 21, 26, 31, 37, 40].

In summary, adversarial perturbations and label noise significantly influence the training of deep learning models. Existing defending methods only concentrate on one type of noise, which deters them from real-world applications. In this work, we focus on studying this practical and challenging problem and propose an effective robust learning method against various noise.

## 3 PROPOSED METHOD

## 3.1 Preliminaries

Denote the dataset with 𝑛 instances as $\mathcal { D } = \{ ( x _ { i } , y _ { i } ) \} _ { i = 1 } ^ { n } \in \mathcal { X } \times \mathcal { Y } ,$ where $x _ { i } \in \mathcal { X } \subseteq \mathbb { R } ^ { d }$ and $y _ { i } \in \mathcal { Y } \subseteq \{ 0 , 1 \} ^ { K }$ are the 𝑖-th image and its associated label, respectively. For the supervised learning for classification that our work concentrate on, typical methods learn the classification model $f _ { \theta } ( \cdot ) \: ( \theta$ is model parameters) by directly minimizing the empirical risk as follows:

$$
\mathcal {L} (f _ {\theta}, \mathcal {D}) = \frac {1}{| \mathcal {D} |} \sum_ {(x, y) \sim \mathcal {D}} \ell (f _ {\theta} (x), y), \tag {1}
$$

where ℓ is a surrogate loss for the classification, such as the Cross-Entropy (CE) loss.

**Training-time attack with adversarial perturbations.** The goal of the training-time attack is to craft adversarial examples by adding imperceptible perturbations into the original data, so that a model that learns with these perturbed data would make wrong predictions on the clean test data. Formally, the process is defined as follows:

$$
\begin{array}{l} \max _ {\rho} \mathcal {L} \left(f _ {\theta^ {*}}, \mathcal {D}\right), \\ \text {L} \text {t} \theta^ {*} \text {ormin} f \left(f _ {\theta^ {*}} (\mathcal {D}, \mathcal {D})\right) \end{array} \tag {2}
$$

$$
s. t. \theta^ {*} = \arg \min _ {\theta} \mathcal {L} (f _ {\theta}, \rho (\mathcal {D}, \epsilon)),
$$

Where $\rho : X \to X$ is the perturbation function, which takes a sample $x \sim \mathcal { D }$ as the input and outputs its adversarial counterpart by learning an unperceivable perturbation pattern injected into 𝑥. 𝜖 is the constraint to limit the magnitude of perturbations, where

$$
\forall x \sim \mathcal {D}, \| \rho (x, \epsilon) - x \| _ {\infty} \leq \epsilon . \tag {3}
$$

![](images/image_01.jpg)

<details>
<summary>flowchart</summary>

This flowchart illustrates a machine learning pipeline for data flow generation and prediction generation. It shows how original data flows through augmentation and noise labeling, which are then processed through EMA Encoder and Classifier to produce final model outputs.
</details>

Figure 1: An illustration of the TPD framework. At each epoch, the TPD would first divide the dataset into a clean subset and a noisy subset according to the novel devised sample selection strategy. After that, semi-supervised learning is performed with the resulted datasets. At the same time, the proposed asymmetric adversarial contrastive learning is conducted, where the perturbation function and the classification model take each other as the adversary. Particularly, the perturbation function is tasked to produce worse-case adversarial data via contrastive learning, aiming to enlarge the semantic consistency between the original data and the adversarial data. While the classification model learns to produce correct and consistent predictions on both the original data and adversarial data.

## 3.2 Overview

To combat the adversarial perturbations and label noise, we propose a novel Two-Pronged Defense (TPD) framework, which aims to improve the generalization ability of the to-be-learnt model against adversarial perturbations, and train the model with reliable supervision by filtering out noisy labels. To fulfill this, the TPD utilizes a well-designed asymmetric adversarial contrastive learning to train the model with dynamically-crafted adversarial examples, which is expected to enhance the robustness of the model to perturbations on the data. Furthermore, the TPD leverages a novel sample selec tion strategy to identify clean and noisy labels, and train the model via a semi-supervised learning by discarding noisy labels.

## 3.3 Asymmetric adversarial contrastive learning for combating adversarial perturbations

An intuition for combating the training-time attack is to train the model to align the consistency between the original data with the perturbed data, so that the model would be endowed with the strong generation ability. During the test, the model would give reliable predictions on clean data. The obstacle here is that we have no idea about how training data are perturbed, and it thus deters us from eliminating the noise influence. To tackle this, we propose to dynamically generate a bunch of adversarial data with a large semantic gap between the training data, and train the model to correctly perceive them and preserve semantic consistency between them and the original training data. Through this, the model is expected to establish strong robustness to the potential adversarial perturbations. Inspired by the [33], we propose to generate worstcase adversarial examples within a specific ∞-Wasserstein ball through asymmetric adversarial contrastive learning without label guidance. Specifically, denote the to-be-learnt model as $f _ { \theta } ,$ which consists of a feature encoder 𝑔 and a classifier ℎ as shown in Figure 1. We additionally construct the exponential moving average (EMA) counterpart of the model $f _ { \theta ^ { \prime } }$ as the key network, which is updated as follows:

$$
\theta_ {t} ^ {\prime} = \alpha \theta_ {t - 1} ^ {\prime} + (1 - \alpha) \theta_ {t}, \tag {4}
$$

where $\alpha \in [ 0 , 1 )$ is a smoothing parameter. We let $g ^ { \prime }$ and $h ^ { \prime }$ represent the corresponding feature encoder and classifier, respectively.

For $\mathcal { D } = \{ ( x _ { i } , y _ { i } ) \} _ { i = 1 } ^ { n } ,$ we transfer them into $\mathcal { D } ^ { q } = \{ ( x _ { i } ^ { \bar { q } } , y _ { i } ) \} _ { i = 1 } ^ { n }$ and $\mathcal { D } ^ { k } = \{ ( x _ { i } ^ { k } , y _ { i } ) \} _ { i = 1 } ^ { n }$ by data augmentation. Given the encoder network, we learn the perturbation function $\rho$ by maximizing the following loss:

$$
\begin{array}{l} \mathcal {L} _ {c} (\rho , \mathcal {D} ^ {q}, \mathcal {D} ^ {k}) = \\ - \frac {1}{| \mathcal {D} ^ {q} |} \sum_ {x ^ {q} \sim \mathcal {D} ^ {q}} \log \frac {\exp \left(g \left(\rho \left(\mathbf {x} ^ {q} , \epsilon\right)\right) ^ {T} \cdot g ^ {\prime} \left(\mathbf {x} ^ {k}\right) / \tau\right)}{\sum_ {\mathbf {x} ^ {k} \sim \mathcal {D} ^ {k}} \exp \left(g \left(\rho \left(\mathbf {x} ^ {q} , \epsilon\right)\right) ^ {T} \cdot g ^ {\prime} \left(\mathbf {x} ^ {k}\right) / \tau\right)}, \tag {5} \\ \end{array}
$$

where $\tau > 0$ is a temperature parameter. Given the generated adversarial data, we train the classification model to keep semantic consistency between the adversarial data with the original data via semi-supervised learning introduced in the following section. As the two parties, i.e., the perturbation function and the classification model take different loss functions, we name this learning strategy as asymmetric adversarial contrastive learning. With such a strategy, it is able to learn a perturbation function that can generate adversarial examples that are significantly dissimilar to the original ones in the feature space. Using these worse-case data with original data to train the model is expected to improve its generalization ability, thereby making it robust to the potential training-time attack. Another benefit is that the training of the perturbation function is independent of the original labels, thus would not be influenced by potential label noise.

## 3.4 Sample selection based semi-supervised learning for combating noisy labels

As indicated by previous work [17, 21, 26], deep models tend to learn easy examples $( \mathbf { i . e . } ,$ clean labels in the field that our work focuses on) before fitting into hard ones $( \mathbf { i . e . } ,$ noisy labels). In light of this, it is feasible to make use of the knowledge of the model learnt at the early training stage to identify clean and noisy labels, and provide predictions to guide the learning with noisy labels. To this end, we calculate the agreement between model predictions and labels, and the agreement between EMA model predictions and labels. We then set a threshold for sample selection by measuring the average agreement between ensemble model predictions and labels $( \mathbf { i . e . }$ , loss distribution). In particular, the average value of agreement between EMA model predictions and labels on the whole dataset is calculated as $\begin{array} { r } { \mu = \frac { 1 } { | \mathcal { D } | } \sum _ { ( x , y ) \sim \mathcal { D } } \ell ( f _ { \theta ^ { \prime } } , y ) } \end{array}$ . For a sample $x _ { i } ,$ the agreement between the model prediction and its label is $\ell ( f _ { \theta } ( x _ { i } ) , y _ { i } )$ . We compare it with the threshold to judge whether the sample is clean or not. If the $\ell ( f _ { \theta } ( x _ { i } ) , y _ { i } )$ exceeds the threshold, it would be identified as noisy, or vice visa.

With identified results, the dataset would be further divided into a clean subset $\mathcal { D } _ { c l n }$ and a noisy subset $\mathcal { D } _ { n o y }$ . We apply semi-supervised learning with the resulted datasets to train the model. Specifically, for the clean subset, we utilize the Cross-Entropy loss. While for the noisy subset, we discard the original labels and use the predictions from both models $( \mathrm { i . e . } , f _ { \theta }$ and $f _ { \theta } ^ { \prime } )$ to guide the learning of $f _ { \theta }$ via the mean squared error. The losses are defined as follows:

$$
\begin{array}{l} \mathcal {L} _ {s} (f _ {\theta}, \mathcal {D} _ {c l n}, \mathcal {D} _ {n o y}) = \\ \mathcal {L} _ {c l n} (f _ {\theta}, \mathcal {D} _ {c l n}) + \beta \cdot \mathcal {L} _ {n s y} (f _ {\theta}, \mathcal {D} _ {n o y}) + \mathcal {L} _ {r e g} (f _ {\theta}, \mathcal {D} _ {c l n} + \mathcal {D} _ {n o y}), \\ s. t. \mathcal {L} _ {c l n} (f _ {\theta}, \mathcal {D} _ {c l n}) = \frac {1}{3 | \mathcal {D} _ {c l n} |} \sum_ {(x, y) \in \mathcal {D} _ {c l n}} (\ell (f _ {\theta} (x), y) \\ + \ell \left(f _ {\theta} (a u g (x)), y\right) + \lambda \cdot \ell \left(f _ {\theta} (\rho (x, \epsilon)), y)\right), \\ \mathcal {L} _ {n s y} (f _ {\theta}, \mathcal {D} _ {n o y}) = \frac {1}{3 | \mathcal {D} _ {n o y} |} \sum_ {x \in \mathcal {D} _ {n o y}} \left(\| f _ {\theta} (x) - M \| _ {2} ^ {2} \right. \\ + \left\| f _ {\theta} (a u g (x)) - M \right\| _ {2} ^ {2} + \lambda \cdot \left\| f _ {\theta} (\rho (x, \epsilon)) - M \right\| _ {2} ^ {2}), \\ \mathcal {L} _ {r e g} (f _ {\theta}, \mathcal {D} _ {c l n} + \mathcal {D} _ {n o y}) = \sum_ {k} \pi_ {k} \log ( \\ \frac {\pi_ {k}}{\frac {1}{\left| \mathcal {D} _ {c l n} + \mathcal {D} _ {n o y} \right|} \sum_ {x \in \mathcal {D} _ {c l n} + \mathcal {D} _ {n o y}} \left(f _ {\theta} (x) + f _ {\theta} (a u g (x)) + f _ {\theta} (\rho (x , \epsilon))\right)}), \tag {6} \\ \end{array}
$$

$$
\begin{array}{l} \mathcal {L} _ {n s y} (f _ {\theta}, \mathcal {D} _ {n o y}) = \frac {1}{3 | \mathcal {D} _ {n o y} |} \sum_ {x \in \mathcal {D} _ {n o y}} \left(\| f _ {\theta} (x) - M \| _ {2} ^ {2} \right. \\ + \left\| f _ {\theta} (a u g (x)) - M \right\| _ {2} ^ {2} + \lambda \cdot \left\| f _ {\theta} (\rho (x, \epsilon)) - M \right\| _ {2} ^ {2}), \\ \end{array}
$$

where 𝑎𝑢𝑔(·) is the hard data augmentation operation, and $\beta , \lambda > 0$ are balancing parameters. $\begin{array} { r } { M = \frac { 1 } { 2 } \big ( f _ { \theta } ( x ) + f _ { \theta } ^ { \prime } ( x ) \big ) } \end{array}$ is the average of predictions of the model and the ensemble model. The third term is for regularization, where $\begin{array} { r } { \pi _ { k } = \frac { 1 } { K } } \end{array}$ . The semi-supervised learning aligns the semantics between predictions on the original data and adversarial data by imposing the same supervision.

Algorithm 1: The Two-Pronged Defense method

<div class="docvortex-algorithm" style="white-space: pre-wrap; font-family:monospace;">
Require: Training data $\mathcal{D} = \{(x_i, y_i)\}_{i=1}^n$, mini-batch size $m$, iteration times $T_m, T_r$, parameters $\alpha, \beta, \tau, \lambda, \epsilon$;
Require: Randomly initialize $\theta = \theta'$;
// WarmUp;
$\{(x_i^q, x_i^k, y_i)\}_{i=1}^n \leftarrow \{(x_i, y_i)\}_{i=1}^n$ // Data augmentation;
for epoch = 1 → $T_1$ do
    for $t = 1 \rightarrow n/m$ do
        $\mathcal{D}' = \{(x_i^q, x_i^k, y_i)\}_{i=1}^m \sim \mathcal{D}$ // Sample mini-batch;
        Maximize Eq (5) to obtain $\rho$;
        Minimize Eq (1) to obtain $\theta$;
        Update ensemble model following Eq (4);
    end
end
// Formal Training;
$\mathcal{D}_{cln}, \mathcal{D}_{nsy} \leftarrow \{(x_i, y_i)\}_{i=1}^n$ // Identify clean and noisy data;
for epoch = 1 → $T_2$ do
    for $t = 1 \rightarrow iter\_num$ do
        $\mathcal{D}'_{cln} = \{(x_i, y_i)\}_{i=1}^m \sim \mathcal{D}_{cln}$ // Sample mini-batch from the clean subset;
        $\mathcal{D}'_{nsy} = \{x_i\}_{i=1}^m \sim \mathcal{D}_{nsy}$ // Sample mini-batch from the noisy subset;
        $\mathcal{D}'_{cln} = \{(x_i, aug(x_i), y_i)\}_{i=1}^m \leftarrow \{(x_i, y_i)\}_{i=1}^m$ // Data augmentation;
        $\mathcal{D}'_{nsy} = \{(x_i, aug(x_i))\}_{i=1}^m \leftarrow \{x_i\}_{i=1}^m$ // Data augmentation;
        Maximize Eq (5) to obtain $\rho$;
        Minimize Eq (6) to obtain $\theta$;
        Update ensemble model following Eq (4);
    end
end
return $\theta$;
</div>

In summary, for the perturbation function, we adopt contrastive learning, while semi-supervised learning for the classification model. We name it as the asymmetric adversarial contrastive learning. The benefit of it is that it prevents the potential adverse influence of noisy labels on training perturbation function, facilitating final results. The optimization scheme is summarized in **Algorithm 1**.

## 4 EXPERIMENTS

## 4.1 Datasets

Two widely-used datasets for classification tasks, i.e., CIFAR-10 and CIFAR-100 [20], are taken to evaluate the performance of the proposed method. They contain 60, 000 images, belonging to 10 and 100 concepts, respectively. 50,000 and 10,000 samples randomly selected from the original dataset are used for training and testing. **Data corruption.** In this work, we use the method in the [10] to introduce adversarial perturbations into the whole training dataset. **Label corruption.** There are three types of label noise: (1) Symmetric noise: the original correct labels are randomly flipped into other possible labels with a probability 𝜅; (2) Asymmetric noise: correct labels are randomly replaced to another similar class with a probability 𝜅, e.g., deer → horse, dog → cat; (3) Instance-dependent label noise: original clean labels are flipped based on both the original labels and features. In this work, we utilize the method proposed in [35] to generate instance-dependent label noise.

Table 1: Test accuracies (%) on CIFAR-10 and CIFAR-100 under adversarial perturbations and instance-based label noise. $\mathbf { \ " A T \" }$ is developed for the task of learning with adversarial perturbations, so that we do not report results on clean data $\mathbf { ( i . e . , } \mathbf { \nabla } ^ { a } C$ and “LN”). “DAT”, “CORES<sup>2</sup>” and “DivideMix” are designed for the task of learning with noise labels, so we do not report results on clean labels $\mathbf { ( i . e . , } \mathbf { \nabla } ^ { a } C \mathbf { \nabla }$ and “AP”). The TPD is devised for the case that both noise exists, so that we only report results on “AP & $L N ^ { \prime \prime } , \alpha _ { \dagger } \omega$ denotes the results of models on their original tasks. “▼” indicates performance degradation. The best results for each category are shown in boldface.

<table><tr><td rowspan="2">Dataset</td><td rowspan="2">Method</td><td rowspan="2">C</td><td rowspan="2">AP</td><td colspan="3">LN (Inst.)</td><td colspan="3">AP &amp; LN (Inst.)</td></tr><tr><td>κ = 0.2</td><td>κ = 0.4</td><td>κ = 0.6</td><td>κ = 0.2</td><td>κ = 0.4</td><td>κ = 0.6</td></tr><tr><td rowspan="6">CIFAR-10</td><td>CE</td><td>95.20 †</td><td>62.91▼</td><td>87.50▼</td><td>74.85▼</td><td>45.23▼</td><td>40.76▼</td><td>14.90▼</td><td>12.75▼</td></tr><tr><td>AT</td><td>-</td><td>87.50 †</td><td>-</td><td>-</td><td>-</td><td>37.98▼</td><td>13.29▼</td><td>10.36▼</td></tr><tr><td>DAT</td><td>-</td><td>-</td><td>84.22 †</td><td>74.71 †</td><td>31.31 †</td><td>27.78▼</td><td>17.25▼</td><td>10.48▼</td></tr><tr><td>CORES2</td><td>-</td><td>-</td><td>86.32 †</td><td>82.65 †</td><td>78.43 †</td><td>14.18▼</td><td>14.62▼</td><td>14.43▼</td></tr><tr><td>DivideMix</td><td>-</td><td>-</td><td>93.45 †</td><td>94.78 †</td><td>64.14 †</td><td>75.33▼</td><td>16.17▼</td><td>24.74▼</td></tr><tr><td>TPD</td><td>-</td><td>-</td><td>-</td><td>-</td><td>-</td><td>79.06</td><td>42.47</td><td>25.22</td></tr><tr><td rowspan="6">CIFAR-100</td><td>CE</td><td>66.48 †</td><td>25.01▼</td><td>58.88▼</td><td>40.76▼</td><td>25.78▼</td><td>22.51▼</td><td>18.35▼</td><td>7.10▼</td></tr><tr><td>AT</td><td>-</td><td>29.28 †</td><td>-</td><td>-</td><td>-</td><td>16.21▼</td><td>11.05▼</td><td>11.0▼</td></tr><tr><td>DAT</td><td>-</td><td>-</td><td>56.62 †</td><td>39.13 †</td><td>20.66 †</td><td>16.85▼</td><td>11.16▼</td><td>6.18▼</td></tr><tr><td>CORES2</td><td>-</td><td>-</td><td>61.56 †</td><td>48.08 †</td><td>36.34 †</td><td>19.47▼</td><td>14.97▼</td><td>10.66▼</td></tr><tr><td>DivideMix</td><td>-</td><td>-</td><td>77.50 †</td><td>74.57 †</td><td>48.57 †</td><td>48.49▼</td><td>42.65▼</td><td>21.17▼</td></tr><tr><td>TPD</td><td>-</td><td>-</td><td>-</td><td>-</td><td>-</td><td>51.41</td><td>46.06</td><td>24.67</td></tr></table>

## 4.2 Setup

**Baselines.** The proposed method is the very first work to defend against both adversarial perturbations and label noise, so that there are no existing methods for comparison. To demonstrate the effectiveness of the proposed TPD, we take methods either combat adversarial perturbations or combat label noise as the competitors, which may have the potential to defend against both noise. Through extensive experiments, we show that existing methods proposed for one kind of noise are incapable of handling the challenging scenarios that both noise exists.

Specifically, for adversarial examples, we take AT [33] as the baseline, which is the only method proposed for defending against the training-time attack by injecting adversarial perturbations into the whole dataset. While for learning with noisy labels, we choose the-state-of-the-art methods, i.e., (1) CE, which learns the model with the conventional CE loss; (2) CORES<sup>2</sup>[8], which designs a confidence regularized sample sieve strategy to filter out noisy labels; (3) DAT [28], which learns to model by aligning the distribution gap between clean data and noisy data; (4) DivideMix [21], which utilizes a mixture model to identify clean and noisy data and trains the model through semi-supervised learning.

**Implementation Details.** The source codes of most baselines are kindly provided by the authors. We carefully tuned parameters of these models. The parameters of the TPD are set as follows. $\alpha = 0 . 9 9 9 , \tau = 0 . 0 7 , \lambda = 0 . 1$ . The setting of 𝛽 is the same as [21]. On CIFAR-10, we set 𝜖 = 0.1 while $\epsilon = 0 . 2$ on CIFAR-100. ResNet34 [14] is adopted as the backbone of all methods. For the TPD, we apply SGD to optimize the network, with an initial learning rate of $\dot { [ 2 e ^ { - 2 } }$ (decayed by 10 at 60 epochs.), a momentum of 0.9 and a weight decay of $1 5 e ^ { - 4 }$ . The epochs for WarmUp and Formal Training are set as 10 and 90 for CIFAR-10, while 30 and 70 for CIFAR-100. Two types of augmentation are utilized, i.e., weak and strong data augmentation. The weak augmentation is just the random transformation of cropping and flipping, while for the strong one, we use the method introduced in [9]. In particular, at the WarmUp stage, we only adopt the soft augmentation on the training data. While for Formal Training, we utilize the hard augmentation for the 𝑎𝑢𝑔(·) operation, and the soft one for others. Training data would be randomly cropped and flipped before being poisoned.

## 4.3 Results and Discussions

To test the effectiveness of the proposed TPD, we conduct extensive experiments on two datasets under different noise conditions. The results are reported in Table 1, Table 2 and Table 3. In the table, the “C”, “AP”, and “LN” represent the training dataset with clean labels and data, adversarial perturbations, and label noise, respectively. “Inst.”, “Symm.” and “Asymm.” denote instance-dependent, symmetric and asymmetric label noise.

From the results, we have the following observations. First, it can be seen that the model learnt with conventional CE loss loses effectiveness when facing no matter adversarial perturbations or label noise. This indicates the adverse impacts of noise and highlights the importance of proposing robust learning methods. Second, the methods proposed to combat either adversarial perturbations or label noise can only perform well when facing their assumed noise. When facing challenging scenarios, where both noise exists, they would also lose the learning power. In terms of quantitative analysis, on CIFAR-10, CE experiences the performance degradation of 33.90%, 16.90% and 75.0% on adversarial perturbations, label noise, and in the case that both exist, respectively. While on CIFAR-100, the drops are 62.38%, 39.60% and 56.92%. The robust learning method against adversarial perturbations, i.e., AT, can perform well when there are adversarial perturbations. However, the label noise can significantly influence its effectiveness, causing average performance drops of 69.06% and 26.21% on two datasets. Similarly, the methods proposed for fighting against label noise are also incapable of dealing with the cases of both noise in the training set. The existence of adversarial examples and label noise leads to average performance drops of 69.74%, 72.15%, and 43.68% for DAT, CORES<sup>2</sup>, DivideMiX, respectively. On the contrary, the proposed TPD achieves promising results and performs better than other baselines (with an average improvement of 20.95%), demonstrating its superiority.

Table 2: Test accuracies (%) on CIFAR-10 and CIFAR-100 under adversarial perturbations and symmetric label noise.

<table><tr><td rowspan="2">Dataset</td><td rowspan="2">Method</td><td rowspan="2">C</td><td rowspan="2">AP</td><td colspan="3">LN (Symm.)</td><td colspan="3">AP &amp; LN (Symm.)</td></tr><tr><td>κ = 0.2</td><td>κ = 0.4</td><td>κ = 0.6</td><td>κ = 0.2</td><td>κ = 0.4</td><td>κ = 0.6</td></tr><tr><td rowspan="6">CIFAR-10</td><td>CE</td><td>95.20 †</td><td>62.91▼</td><td>86.38▼</td><td>81.74▼</td><td>75.03▼</td><td>26.89▼</td><td>18.59▼</td><td>17.79▼</td></tr><tr><td>AT</td><td>-</td><td>87.50 †</td><td>-</td><td>-</td><td>-</td><td>25.75▼</td><td>22.94▼</td><td>10.14▼</td></tr><tr><td>DAT</td><td>-</td><td>-</td><td>83.76 †</td><td>83.18 †</td><td>81.37 †</td><td>29.74▼</td><td>32.13▼</td><td>29.22▼</td></tr><tr><td>CORES2</td><td>-</td><td>-</td><td>90.38 †</td><td>85.15†</td><td>80.27 †</td><td>14.08▼</td><td>14.30▼</td><td>13.15▼</td></tr><tr><td>DivideMix</td><td>-</td><td>-</td><td>93.58 †</td><td>94.99 †</td><td>94.79 †</td><td>72.60▼</td><td>69.86▼</td><td>14.58▼</td></tr><tr><td>TPD</td><td>-</td><td>-</td><td>-</td><td>-</td><td>-</td><td>80.33</td><td>79.0</td><td>71.98</td></tr><tr><td rowspan="6">CIFAR-100</td><td>CE</td><td>66.48 †</td><td>25.01▼</td><td>62.28▼</td><td>49.64▼</td><td>38.31▼</td><td>26.67▼</td><td>20.09▼</td><td>11.73▼</td></tr><tr><td>AT</td><td>-</td><td>29.28 †</td><td>-</td><td>-</td><td>-</td><td>26.84▼</td><td>15.16▼</td><td>11.22▼</td></tr><tr><td>DAT</td><td>-</td><td>-</td><td>56.29 †</td><td>43.84 †</td><td>34.55 †</td><td>13.42▼</td><td>13.62▼</td><td>4.71▼</td></tr><tr><td>CORES2</td><td>-</td><td>-</td><td>60.71 †</td><td>45.52 †</td><td>34.49 †</td><td>23.31▼</td><td>20.19▼</td><td>13.31▼</td></tr><tr><td>DivideMix</td><td>-</td><td>-</td><td>79.58 †</td><td>75.53 †</td><td>48.50 †</td><td>49.52▼</td><td>45.80▼</td><td>33.87▼</td></tr><tr><td>TPD</td><td>-</td><td>-</td><td>-</td><td>-</td><td>-</td><td>54.68</td><td>46.83</td><td>36.18</td></tr></table>

Table 3: Test accuracies (%) on CIFAR-10 and CIFAR-100 under adversarial perturbations and asymmetric label noise.

<table><tr><td rowspan="2">Dataset</td><td rowspan="2">Method</td><td rowspan="2">C</td><td rowspan="2">AP</td><td colspan="3">LN (Asymm.)</td><td colspan="3">AP &amp; LN (Asymm.)</td></tr><tr><td>κ = 0.2</td><td>κ = 0.3</td><td>κ = 0.4</td><td>κ = 0.2</td><td>κ = 0.3</td><td>κ = 0.4</td></tr><tr><td rowspan="6">CIFAR-10</td><td>CE</td><td>95.20 †</td><td>62.91▼</td><td>88.47▼</td><td>87.12▼</td><td>85.1▼</td><td>35.52▼</td><td>23.65▼</td><td>21.64▼</td></tr><tr><td>AT</td><td>-</td><td>87.50 †</td><td>-</td><td>-</td><td>-</td><td>48.05▼</td><td>36.65▼</td><td>30.55▼</td></tr><tr><td>DAT</td><td>-</td><td>-</td><td>86.10 †</td><td>83.86 †</td><td>81.47 †</td><td>27.40▼</td><td>28.09▼</td><td>28.44▼</td></tr><tr><td>CORES2</td><td>-</td><td>-</td><td>90.62 †</td><td>90.53 †</td><td>87.59 †</td><td>14.59▼</td><td>14.39▼</td><td>13.70▼</td></tr><tr><td>DivideMix</td><td>-</td><td>-</td><td>91.98 †</td><td>93.61 †</td><td>87.21 †</td><td>72.50▼</td><td>56.49▼</td><td>27.25▼</td></tr><tr><td>TPD</td><td>-</td><td>-</td><td>-</td><td>-</td><td>-</td><td>76.32</td><td>59.82</td><td>60.32</td></tr><tr><td rowspan="6">CIFAR-100</td><td>CE</td><td>66.48 †</td><td>25.01▼</td><td>59.20▼</td><td>51.40▼</td><td>42.35▼</td><td>31.46▼</td><td>24.73▼</td><td>21.99▼</td></tr><tr><td>AT</td><td>-</td><td>29.28 †</td><td>-</td><td>-</td><td>-</td><td>17.68▼</td><td>14.16▼</td><td>12.01▼</td></tr><tr><td>DAT</td><td>-</td><td>-</td><td>56.86 †</td><td>50.26 †</td><td>42.46 †</td><td>19.21▼</td><td>14.98▼</td><td>11.76▼</td></tr><tr><td>CORES2</td><td>-</td><td>-</td><td>62.72 †</td><td>56.04 †</td><td>45.88 †</td><td>19.36▼</td><td>21.32▼</td><td>18.68▼</td></tr><tr><td>DivideMix</td><td>-</td><td>-</td><td>77.05 †</td><td>76.96 †</td><td>55.10 †</td><td>49.77▼</td><td>45.06▼</td><td>33.85▼</td></tr><tr><td>TPD</td><td>-</td><td>-</td><td>-</td><td>-</td><td>-</td><td>53.12</td><td>47.18</td><td>38.86</td></tr></table>

## 5 CONCLUSION

In this paper, we present a novel robust learning method, i.e., Two-Pronged Defense (TPD), to eliminate the adverse influence of adversarial perturbations and label noise on the learning process. The

TPD leverages asymmetric adversarial contrastive learning to improve the generalization ability of the model by training it with the dynamically generated adversarial examples. Furthermore, the TPD adopts semi-supervised learning by filtering out noisy labels from the dataset to make sure that the model is trained with clean labels. In light of these, the proposed TPD is expected to enhance the robustness of the to-be-learnt model to various noise in the training set. Extensive experiments conducted on two benchmarks demonstrate the effectiveness of the proposed TPD. This is the very first work towards solving the challenging scenarios, where both data perturbations and label noise exist in the training set. In the future work, we may exploit to improve the robustness of the model under the data-free setting, where the model has been trained with the noisy dataset and the original data are no longer available.

## 6 ACKNOWLEDGMENTS

This work was partially supported by Australian Research Council Discovery Project (ARC DP190102353, CE200100025), and China Scholarship Council.

## REFERENCES

[1] Scott Alfeld, Xiaojin Zhu, and Paul Barford. 2016. Data poisoning attacks against autoregressive models. In Proceedings of the AAAI Conference on Artificial Intelligence. 480–501.
[2] Battista Biggio, Blaine Nelson, and Pavel Laskov. 2012. Poisoning attacks against support vector machines. In Proceedings of the International Coference on International Conference on Machine Learning. 1467–1474.
[3] Xinyun Chen, Chang Liu, Bo Li, Kimberly Lu, and Dawn Song. 2017. Targeted backdoor attacks on deep learning systems using data poisoning. arXiv preprint arXiv:1712.05526 (2017).
[4] Zhi Chen, Jingjing Li, Yadan Luo, Zi Huang, and Yang Yang. 2020. Canzsl: Cycleconsistent adversarial networks for zero-shot learning from natural language. In Proceedings of the IEEE/CVF Winter Conference on Applications of Computer Vision. 874–883.
[5] Zhi Chen, Yadan Luo, Ruihong Qiu, Sen Wang, Zi Huang, Jingjing Li, and Zheng Zhang. 2021. Semantics disentangling for generalized zero-shot learning. In Proceedings of the IEEE/CVF International Conference on Computer Vision.
[6] Zhi Chen, Yadan Luo, Sen Wang, Ruihong Qiu, Jingjing Li, and Zi Huang. 2021. Mitigating generation shifts for generalized zero-shot learning. In Proceedings of the ACM International Conference on Multimedia.
[7] Zhi Chen, Sen Wang, Jingjing Li, and Zi Huang. 2020. Rethinking generative zeroshot learning: An ensemble learning perspective for recognising visual patches. In Proceedings of the ACM International Conference on Multimedia. 3413–3421.
[8] Hao Cheng, Zhaowei Zhu, Xingyu Li, Yifei Gong, Xing Sun, and Yang Liu. 2020. Learning with instance-dependent label noise: A sample sieve approach. In Proceedings of the International Conference on Learning Representations.
[9] Ekin D Cubuk, Barret Zoph, Dandelion Mane, Vijay Vasudevan, and Quoc V Le. 2018. Autoaugment: Learning augmentation policies from data. arXiv preprint arXiv:1805.09501 (2018).
[10] Ji Feng, Qi-Zhi Cai, and Zhi-Hua Zhou. 2019. Learning to confuse: Generating training time adversarial data with auto-encoder. In Proceedings of the International Conference on Neural Information Processing Systems. 11994–12004.
[11] Aritra Ghosh, Himanshu Kumar, and PS Sastry. 2017. Robust loss functions under label noise for deep neural networks. In Proceedings of the AAAI Conference on Artificial Intelligence. 1919–1925.
[12] Jacob Goldberger and Ehud Ben-Reuven. 2017. Training deep neural-networks using a noise adaptation layer. In Proceedings of the International Conference on Learning Representations.
[13] Bo Han, Quanming Yao, Xingrui Yu, Gang Niu, Miao Xu, Weihua Hu, Ivor W Tsang, and Masashi Sugiyama. 2018. Co-teaching: Robust training of deep neural networks with extremely noisy labels. In Proceedings of the International Conference on Neural Information Processing Systems. 8536–8546.
[14] Kaiming He, Xiangyu Zhang, Shaoqing Ren, and Jian Sun. 2016. Deep residual learning for image recognition. In Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition. 770–778.
[15] Matthew Jagielski, Alina Oprea, Battista Biggio, Chang Liu, Cristina Nita-Rotaru, and Bo Li. 2018. Manipulating machine learning: Poisoning attacks and countermeasures for regression learning. In Proceedings of the IEEE Symposium on Security and Privacy. 19–35.
[16] Lu Jiang, Zhengyuan Zhou, Thomas Leung, Li-Jia Li, and Li Fei-Fei. 2018. Mentornet: Learning data-driven curriculum for very deep neural networks on corrupted labels. In Proceedings of the International Conference on Machine Learning. 2304–2313.
[17] Nazmul Karim, Mamshad Nayeem Rizve, Nazanin Rahnavard, Ajmal Mian, and Mubarak Shah. 2022. UNICON: Combating label noise through uniform selection and contrastive learning. In Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition. 9676–9686.
[18] Jacob Devlin Ming-Wei Chang Kenton and Lee Kristina Toutanova. 2019. BERT: Pre-training of deep bidirectional transformers for language understanding. In Proceedings of the Conference of the North American Chapter of the Association for Computational Linguistics: Human Language Technologies. 4171–4186.
[19] Pang Wei Koh and Percy Liang. 2017. Understanding black-box predictions via influence functions. In Proceedings of the International Conference on Machine Learning. 1885–1894.
[20] Alex Krizhevsky, Geoffrey Hinton, et al. 2009. Learning multiple layers of features from tiny images. (2009).
[21] Junnan Li, Richard Socher, and Steven CH Hoi. 2020. Dividemix: Learning with noisy labels as semi-supervised learning. In Proceedings of the International Conference on Learning Representations.
[22] Yang Li, Tong Chen, Peng-Fei Zhang, and Hongzhi Yin. 2021. Lightweight self-attentive sequential recommendation. In Proceedings of the ACM International Conference on Information & Knowledge Management. 967–977.
[23] Yang Liu and Hongyi Guo. 2020. Peer loss functions: Learning from noisy labels without knowing noise rates. In Proceedings of the International Conference on
Machine Learning. 6226–6236.
[24] Xingjun Ma, Yisen Wang, Michael E Houle, Shuo Zhou, Sarah Erfani, Shutao Xia, Sudanthi Wijewickrema, and James Bailey. 2018. Dimensionality-driven learning with noisy labels. In Proceedings of the International Conference on Machine Learning. 3355–3364.
[25] Blaine Nelson, Marco Barreno, Fuching Jack Chi, Anthony D Joseph, Benjamin IP Rubinstein, Udam Saini, Charles Sutton, J Doug Tygar, and Kai Xia. 2008. Exploiting machine learning to subvert your spam filter. In Proceedings of the Usenix Workshop on Large-Scale Exploits and Emergent Threats.
[26] Duc Tam Nguyen, Chaithanya Kumar Mummadi, Thi Phuong Nhung Ngo, Thi Hoai Phuong Nguyen, Laura Beggel, and Thomas Brox. 2019. Self: Learning to filter noisy labels with self-ensembling. In Proceedings of the International Conference on Learning Representations.
[27] Giorgio Patrini, Alessandro Rozza, Aditya Krishna Menon, Richard Nock, and Lizhen Qu. 2017. Making deep neural networks robust to label noise: A loss correction approach. In Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition. 1944–1952.
[28] Yuntao Qu, Shasha Mo, and Jianwei Niu. 2021. DAT: Training deep networks robust to label-noise by matching the feature distributions. In Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition. 6821–6829.
[29] Mengye Ren, Wenyuan Zeng, Bin Yang, and Raquel Urtasun. 2018. Learning to reweight examples for robust deep learning. In Proceedings of the International Conference on Machine Learning. 4334–4343.
[30] Ali Shafahi, W Ronny Huang, Mahyar Najibi, Octavian Suciu, Christoph Studer, Tudor Dumitras, and Tom Goldstein. 2019. Poison frogs! targeted clean-label poisoning attacks on neural networks. Advances in Neural Information Processing Systems, 6106–6116.
[31] Yanyao Shen and Sujay Sanghavi. 2019. Learning with bad training data via iterative trimmed loss minimization. In Proceedings of the International Conference on Machine Learning. 5739–5748.
[32] Sainbayar Sukhbaatar, Joan Bruna, Manohar Paluri, Lubomir Bourdev, and Rob Fergus. 2015. Training convolutional networks with noisy labels. In Proceedings of the International Conference on Learning Representations.
[33] Lue Tao, Lei Feng, Jinfeng Yi, Sheng-Jun Huang, and Songcan Chen. 2021. Better safe than sorry: Preventing delusive adversaries with adversarial training. Advances in Neural Information Processing Systems 34 (2021), 16209–16225.
[34] Yisen Wang, Xingjun Ma, Zaiyi Chen, Yuan Luo, Jinfeng Yi, and James Bailey. 2019. Symmetric cross entropy for robust learning with noisy labels. In Proceedings of the IEEE International Conference on Computer Vision. 322–330.
[35] Xiaobo Xia, Tongliang Liu, Bo Han, Nannan Wang, Mingming Gong, Haifeng Liu, Gang Niu, Dacheng Tao, and Masashi Sugiyama. 2020. Part-dependent label noise: Towards instance-dependent label noise. In Proceedings of the International Conference in Neural Information Processing Systems. 7597–7610.
[36] Huang Xiao, Battista Biggio, Gavin Brown, Giorgio Fumera, Claudia Eckert, and Fabio Roli. 2015. Is feature selection secure against training data poisoning?. In Proceedings of the International Conference on Machine Learning. 1689–1698.
[37] Xingrui Yu, Bo Han, Jiangchao Yao, Gang Niu, Ivor Tsang, and Masashi Sugiyama. 2019. How does disagreement help generalization against label corruption?. In Proceedings of the International Conference on Machine Learning. 7164–7173.
[38] Yu-Wei Zhan, Xin Luo, Yongxin Wang, and Xin-Shun Xu. 2020. Supervised hierarchical deep hashing for cross-modal retrieval. In Proceedings of the ACM International Conference on Multimedia. 3386–3394.
[39] Peng-Fei Zhang, Jiasheng Duan, Zi Huang, and Hongzhi Yin. 2021. Joint-teaching: Learning to refine knowledge for resource-constrained unsupervised cross-modal retrieval. In Proceedings of the ACM International Conference on Multimedia. 1517–1525.
[40] Peng-Fei Zhang, Zi Huang, Guangdong Bai, and Xin-Shun Xu. 2022. IDEAL: Highorder-ensemble adaptation network for learning with noisy labels. In Proceedings of the ACM International Conference on Multimedia. 325–333.
[41] Peng-Fei Zhang, Zi Huang, and Xin-Shun Xu. 2021. Proactive privacy-preserving learning for retrieval. In Proceedings of the AAAI Conference on Artificial Intelligence. 3369–3376.
[42] Peng-Fei Zhang, Yang Li, Zi Huang, and Xin-Shun Xu. 2021. Aggregation-based graph convolutional hashing for unsupervised cross-modal retrieval. IEEE Transactions on Multimedia 24 (2021), 466–479.
[43] Peng-Fei Zhang, Yang Li, Zi Huang, and Hongzhi Yin. 2021. Privacy protection in deep multi-modal retrieval. In Proceedings of the International ACM SIGIR Conference on Research and Development in Information Retrieval. 634–643.
[44] Peng-Fei Zhang, Yadan Luo, Zi Huang, Xin-Shun Xu, and Jingkuan Song. 2021. High-order nonlocal hashing for unsupervised cross-modal retrieval. World Wide Web 24 (2021), 563–583.
[45] Zhilu Zhang and Mert R Sabuncu. 2018. Generalized cross entropy loss for training deep neural networks with noisy labels. In Proceedings of the International Conference on Neural Information Processing Systems. 8792–8802.