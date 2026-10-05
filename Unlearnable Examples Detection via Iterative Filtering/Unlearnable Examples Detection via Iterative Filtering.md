# Unlearnable Examples Detection via Iterative Filtering

Yi Yu<sup>1</sup>⋆, Qichen Zheng<sup>1</sup>⋆, Siyuan Yang<sup>1</sup>⋆⋆, Wenhan Yang<sup>2</sup>, Jun Liu<sup>3</sup>, Shijian Lu<sup>1</sup>, Yap-Peng Tan<sup>1</sup>, Kwok-Yan Lam<sup>1</sup>, and Alex Kot<sup>1</sup>

1 Nanyang Technological University, Singapore

2 PengCheng Laboratory, Shenzhen, China

3 Lancaster University, UK

{yuyi0010, qichen001, siyuan.yang, Shijian.Lu, eyptan, kwokyan.lam, eackot}@ntu.edu.sg, yangwh@pcl.ac.cn, j.liu81@lancaster.ac.uk

Abstract. Deep neural networks are proven to be vulnerable to data poisoning attacks. Recently, a specific type of data poisoning attack known as availability attacks, has led to the failure of data utilization for model learning by adding imperceptible perturbations to images. Consequently, it is quite beneficial and challenging to detect poisoned samples, also known as Unlearnable Examples (UEs), from a mixed dataset. In response, we propose an Iterative Filtering approach for UEs identification. This method leverages the distinction between the inherent semantic mapping rules and shortcuts, without the need for any additional information. We verify that when training a classifier on a mixed dataset containing both UEs and clean data, the model tends to quickly adapt to the UEs compared to the clean data. Due to the accuracy gaps between training with clean/poisoned samples, we employ a model to misclassify clean samples while correctly identifying the poisoned ones. The incorporation of additional classes and iterative refinement enhances the model’s ability to differentiate between clean and poisoned samples. Extensive experiments demonstrate the superiority of our method over state-of-the-art detection approaches across various attacks, datasets, and poison ratios, significantly reducing the Half Total Error Rate (HTER) compared to existing methods.

Keywords: Unlearnable Examples, Detection.

## 1 Introduction

The recent emphasis on data-centric AI [46] underscores the importance and effectiveness of improving the model’s performance from the perspective of data pre-processing optimization instead of solely the model design. It becomes critical to address the challenges tied to those data-centric issues, including data collection, data preprocessing, data quality maintenance, etc., for learning-based methods. In fact, a considerable portion of the machine learning process has been dedicated to the data issue [39]. These efforts have gradually guided researchers towards a consensus: high-quality data is critical for enabling more advanced machine-learning algorithms to reach their full potential, which poses the data on par with the approach itself. However, prevalent challenges persist, given that many real-world datasets are limited in size, dirty [10], biased [34], and in some cases, even contaminated with malicious intents [32].

<small><span class="docvortex-page-footnote" data-block-type="page_footnote" style="color:#6b7280">⋆equal contribution</span></small>

<small><span class="docvortex-page-footnote" data-block-type="page_footnote" style="color:#6b7280">⋆⋆ Corresponding author</span></small>

Together with the impressive performance of the deep neural networks, many concerns have been raised about their related AI security issues [45,37,41,44]. Data poisoning attacks [17,5,22] further intensify these difficulties and reveal the urgent requirement for the attention of the data management community. The significance of this issue is rooted in the deliberate actions of attackers who maliciously manipulate data to undermine the accuracy of AI applications. Compared to the natural degradation of signals or shifts in features, these contaminations are more insidious and can result in greater damage, leading to a more obvious performance drop. In this work, we focus on a specific type of data poisoning attack termed unlearnable examples (UEs) [42,40,20,43,27,38]. These attacks add nearly invisible perturbations to the training data, which makes the trained models fail to obtain useful knowledge for reaching reasonable performance.

The widely employed dataset search engines [2] have heightened the risks of potential threats and misuse. Malicious dataset providers might release metadata to the public, which can be automatically discovered and propagated through search engines. UEs, by introducing invisible perturbations to the original image, present a considerable challenge in distinguishing between clean and poisoned samples. The malicious dataset owner could even create mixed data by poisoning only portions of the data, which further makes the detection more challenging. Researches [16,31] demonstrate that incorporating even a small proportion of clean samples into an unlearnable dataset leads to an increase in test accuracy. As a result, it is difficult for data users to determine whether the dataset is normal because the trained model appears to have reasonable performance. Training on these UEs may not fully leverage the model’s potency, compromising testing performance and wasting computational resources and time.

In this paper, we introduce Iterative Filtering (IF), an effective detection technique tailored for detecting a wide range of visually imperceptible UEs. Our analysis reveals a critical insight: when a classifier is trained on a dataset that combines UEs with clean data, the model tends to adapt more rapidly to the UEs than to the clean data. This suggests that when the model is evaluated on previously unseen UE, it often demonstrates superior accuracy compared to its performance on clean samples. Exploiting the accuracy disparities between training on clean and poisoned samples, we design a model to misclassify clean samples while accurately identifying the poisoned ones, facilitating the detection of tainted data. Furthermore, our findings lead us to the formulation of the Iterative Filtering algorithm. With additional classes and iterative refinement, the proposed approach achieves improved performance in distinguishing these two types of samples. Our contributions can be summarized as:

– In this paper, we address a critical issue in the era of deep learning: how to identify and filter harmful unlearnable data, screening samples that cannot be learned from. Accordingly, we introduce Iterative Filtering (IF), the first detection method that aims at identifying visually imperceptible UEs.
IF capitalizes on the observation that models trained on datasets blending UEs with clean data tend to adjust more swiftly to UEs, resulting in a discernible accuracy differential, to enable the identification of UEs. Specifically, IF integrates additional classes and undergoes iterative refinement, enhancing its discrimination between clean and poisoned samples.
Extensive experiments show the superior performance of our method over SOTA detection approaches when confronted with various types of attacks, datasets, and poison ratios. When employing a detection-purification strategy, the results further emphasize the method’s robustness in strengthening defenses against UEs.

## 2 Related Work

## 2.1 Data poisoning

Data poisoning [1,15,17,18] is an increasingly recognized challenge in the modern machine learning ecosystem. Essentially, it involves the malicious modification of training data (often in a passive manner) to deliberately distort the behavior of a machine learning model. These data poisoning attacks manifest in multiple forms, ranging from targeted attacks aimed at particular categories to UEs seeking to undermine overall test accuracy. For example, Backdoor attacks [5,7,12,28] are characterized by the manipulation of training data instances. This allows attackers to control the target model’s output utilizing a predetermined trigger.

While most of the literature focuses on the malicious use of poisoning attacks, UEs are employed as a protective measure against unauthorized model training. For example, Error-Minimizing (EM) [16] poisons introduce error minimization noise to prevent deep learning models from absorbing knowledge. Targeted Adversarial Poisoning (TAP) [9] uses targeted adversarial examples of pre-trained models for UEs. Robust Error-Minimizing (REM) [11] introduces adversarial training based on the EM method to generate robust UEs. Self-ensemble protection (SEP) [4] uses multiple model checkpoints’ gradients to generate poison in a self-ensemble manner. Linear separable Synthetic Perturbation(LSP) [42] reveals that the perturbations of several existing UEs are (almost) linearly separable and propose to use synthetic shortcuts to perform availability attack. Recently, One Pixel Shortcut (OPS) [40] delves into the model’s susceptibility to sparse poisons and augments its resistance to adversarial training.

## 2.2 Existing Defense against UEs

Defense against UEs can be mainly classified into two distinct categories: preprocessing and training-phase defenses. The method based on preprocessing eliminates the poison added by the attacker by preprocessing the data before training. Recently, [23] proposes to purify the poisoned data using grayscale transformation and JPEG compression [26]. [8] demonstrates the efficacy of diffusion models [14] in removing data protection perturbations. Training-phase defense methods are characterized by alterations in the training procedure to enable model robustness, even when exposed to poisoned data. Existing work tends to adopt adversarial training [25] as the countermeasure. However, this approach is not without its shortcomings – the substantial computational overhead and extended training durations often outweigh its benefits, and there exists a tangible risk of compromising the performance of models trained on clean datasets. Recently, adversarial augmentations [30] introduces an innovative technique of applying a spectrum of augmentations.

## 2.3 Detection of Backdoor attacks

A similar but different task compared to availability attack detection is backdoor attack detection. To date, some defense methods have been proposed to detect and mitigate backdoor attacks. [33] unveils a general defense against poisoning attacks, leveraging outlier or anomaly detection techniques. However, a significant limitation of their approach is the prerequisite of a clean, trusted dataset to effectively train the outlier detector. Addressing this constraint, [3] introduces a pioneering methodology capable of detecting poisonous backdoors without the necessity of a verified and trusted dataset. This approach involves analyzing the neural network activations associated with the training data. It assesses whether the data is poisoned and identifies the specific data points that are affected if poisoning is detected. [35] proposes a Spectral Signature defense that removes the data with the top ϵ eigenvalue. [36] proposes a Neural Cleanse defense that first reverse-engineers a trigger by searching for patches that cause strong misclassification, then prunes neurons with large activations. [29] capitalizes on the observation that adversarial examples exhibit distinct feature distributions in higher layers of a neural network compared to their clean counterparts. They introduce a straightforward yet remarkably effective defense mechanism called Deep k-NN, which is designed for detecting and removing poisoned samples by leveraging these differences in feature distributions. Recently, [21] introduces $\mathrm { T e C o } ,$ a technique anchored in sample corruption consistency for the precise detection of trigger samples during testing.

## 3 Preliminary

Unlearnable Examples. In the era of big data, the internet and search engines bring about massive volumes of data that can be used for training deep models. However, there is a possibility of data contamination caused by unlearnable examples. That is, with small imperceptible perturbations, the data appears normal still in visual appearance but results in the failure of training deep networks with unsatisfactory testing accuracy. Here, these perturbed samples are termed ‘unlearnable examples’ (UEs). Given a clean dataset $\mathcal { D } _ { c } = \{ ( x _ { i } , y _ { i } ) \} _ { i = 1 } ^ { N }$ comprising N training samples, where $x   \in   \mathcal { X }   \subset   R ^ { d }$ represents the image and $y \in \mathcal { Y } = \{ 0 , \cdots , C - 1 \}$ represents its labels. We consider a scenario in which a classifier is trained on the unlearnable data, represented as $f _ { \theta } : { \mathcal { X } } \to { \mathcal { Y } }$ . To corrupt the model training, the existing methods introduce perturbations to the clean images, resulting in an unlearnable dataset defined as:

$$
\mathcal {D} _ {u} = \{(x _ {i} + \delta_ {i}, y _ {i}) \} _ {i = 1} ^ {N}, \tag {1}
$$

where $\delta _ { i }   \in   \varDelta _ { \mathcal { D } }   \subset   R ^ { d }$ , with $\varDelta _ { \mathcal { D } }$ representing the perturbation set for $\mathcal { D } _ { c }$ . The objective of unlearnability is to ensure that a classifier $f _ { \theta }$ trained on $\mathcal { D } _ { u }$ exhibits poor performance on testing datasets when using $\mathcal { D } _ { u }$ in the inference stage.

UEs Detection. Given a mixed dataset $\mathcal { D }   =   \mathcal { D } _ { u } \cup \mathcal { D } _ { c }$ , where $\mathcal { D } _ { u }   =   \{ ( x _ { i } +$ $\left. \left. \left. \delta _ { i } , y _ { i } \right) \right\} _ { i = 1 } ^ { N _ { u } } \right.$ consists of $N _ { w }$ UEs and $\mathcal { D } _ { c } = \{ ( x _ { i } , y _ { i } ) \} _ { i = 1 } ^ { N _ { c } }$ including $N _ { c }$ clean samples, the detection of UEs within the mixed dataset D turns to learn a mapping f appropriate for a binary classification problem, relative to the poison ratio $\begin{array} { r } { \hat { p } = \frac { N _ { u } ^ { - } } { N _ { u } + N _ { c } } } \end{array}$ . Mathematically, the problem can be formulated as:

$$
\min _ {f (x) \in \{0, 1 \}} \sum_ {i = 1} ^ {N _ {u} + N _ {c}} | f (x) - I \{x \in \mathcal {D} _ {u} \} |. \tag {2}
$$

## 4 Methodology

## 4.1 Key Intuition

UEs make faster learners. Recent research indicates that UEs offer easily learnable features that are closely linked to labels, commonly recognized as shortcuts [42,31]. A notable observation is that when training a classifier on a dataset merging UEs and clean data, the model adapts to the UEs more quickly than it does to the clean data [16,9,42]. Basically, when training a classifier $F ( \cdot | \theta )$ on $\mathcal { D } = \mathcal { D } _ { u } \cup \mathcal { D } _ { c } ,$ the optimization process (regarding the loss $\mathcal { L } )$ is given by

$$
\theta = \arg \min _ {\theta} \sum_ {i = 1} ^ {N _ {u}} \mathcal {L} \left(F \left(x _ {i} + \delta_ {i} \mid \theta\right), y _ {i}\right) + \sum_ {i = 1} ^ {N _ {c}} \mathcal {L} \left(F \left(x _ {i} \mid \theta\right), y _ {i}\right). \tag {3}
$$

Since $\delta _ { i }$ serves as a shortcut, the optimization process will prioritize minimizing the loss on UEs over clean data. This indicates that when evaluated on previously unseen UE, which typically follow a similar distribution to the UEs in the training set [23,42,40], the model often shows superior accuracy compared to its performance on clean data.

To validate this phenomenon, we consider a mixed dataset D that includes an equal number of UEs and clean data. Subsequently, we randomly sample 50% of the data from D for classifier training and evaluate on the remaining data. Figure 1 illustrates the testing accuracy for both the unseen UEs and the clean data throughout each epoch. Notably, for EM [16] and OPS [40], the accuracy for UEs increases near 100% within a handful of epochs, while accuracy improvement for the clean data is relatively slow.

![](images/image_01.jpg)

Fig. 1. Test accuracy (%) on the unseen unlearnable data and clean data when training a classifier on a mixed dataset.

![](images/image_02.jpg)

Fig. 2. Test accuracy (%) on the unseen unlearnable data and clean data when training a classifier on a mixed dataset plus additional clean data with label set to $y + C .$

This observation suggests that leveraging the distinction in learnability between UEs and clean data, especially through the introduction of an early stopping during training, can help to filter out potential inaccuracies labeled as clean data. In each iteration, we randomly select training data from the mixed dataset and evaluate on the remaining data. Through this iterative process, we can progressively distinguish the clean data from the rest. Nevertheless, when clean data is part of the optimization process as depicted in Eq. 4, the accuracy for such data might not necessarily remain at a low value. Consequently, if there is no significant gap in accuracy between UEs and clean data, the filtering-based detection approach may fail to obtain the desirable results.

It is simpler to distinguish between UEs and clean data. To tackle the above-mentioned issue, we reveal a key insight: Distinguishing between clean and poisoned samples is demonstrated to be less challenging than guiding the model to misclassify clean samples while accurately classifying poisoned ones. In the filtering-based algorithm described above, the clean data, once filtered, is excluded from model training in subsequent iterations. Moreover, our studies provide evidence suggesting that these excluded clean data can considerably improve the effectiveness of detection.

Considering a mixed training dataset comprising 50% UEs and 50% clean data, each with their true labels y. Additionally, we have the excluded clean data, but in this case, we modify their labels to $y + C \in \{ C , \cdots , 2 C - 1 \}$ , where C represents the number of classes in the dataset. These newly labeled data are then merged with the mixed dataset to form the dataset for the next iteration. The improved version of the optimization process is as follows:

Algorithm 1: Iterative Filtering (IF)

<div class="docvortex-algorithm" style="white-space: pre-wrap; font-family:monospace;">
Input: A mixed dataset $\mathcal{D}$ with $C$ classes, Classifier $F(\cdot|C_F)$ with $C_F$ classes,
Percentage $p$, Threshold $N_{thre}$.
Output: Unlearnable data $\widehat{\mathcal{D}}_u$, Clean data $\widehat{\mathcal{D}}_c$
Initialize: $it = 0$, retrieved = False;
while $N_{update} &gt; N_{thre}$ do
    # filter clean data
    Randomly divide $\mathcal{D}$ into two subsets: $\mathcal{D}_{train}^{(it)}$ and $\mathcal{D}_{val}^{(it)}$, where
    $\frac{|\mathcal{D}_{train}^{(it)}|}{|\mathcal{D}|} = p$;
    Randomly initialize and train a classifier $F(\cdot|2C)$ using $\mathcal{D}_{train}^{(it)}$ with early
    stopping;
    Evaluate $F(\cdot|2C)$ on $\mathcal{D}_{val}^{(it)}$, update $y_i \in \mathcal{D}_{val}^{(it)}$ by
    $y_i = y_i + C \cdot I\{(\hat{y}_i = y_i) \ \&amp; \ (y_i \in [0, C - 1])\}$;
    Record the number of updated $y_i$ as $N_{update}$;
    # retrieve unlearnable data
    if $N_{update} &gt; N_{thre}$ &amp; retrieved = False then
        $\mathcal{D}_{val}^{(it)} = \{(x, y) \mid (x, y) \in \mathcal{D}, y \in [C, 2C - 1]\}$;
        $\mathcal{D}_{train}^{(it)} = \mathcal{D} \backslash \mathcal{D}_{val}^{(it)}$;
        Randomly initialize and train a classifier $F(\cdot|C)$ using $\mathcal{D}_{train}^{(it)}$ with
        early stopping;
        Evaluate $F(\cdot|C)$ on $\mathcal{D}_{val}^{(it)}$, update $y_i \in \mathcal{D}_{val}^{(it)}$ using
        $y_i = y_i - C \cdot I\{\hat{y}_i = y_i - C\}$;
        retrieved = True;
        $N_{update} = 0$;
    end
    $it = it + 1$;
end
Return $\widehat{\mathcal{D}}_u = \{(x, y)|y \in [0, C - 1]\}$, $\widehat{\mathcal{D}}_c = \{(x, y)|y \in [C, 2C - 1]\}$
</div>

$$
\theta = \arg \min _ {\theta} \sum_ {i = 1} ^ {N _ {u}} \mathcal {L} (F (x _ {i} + \delta_ {i} | \theta), y _ {i}) + \sum_ {i = 1} ^ {N _ {c}} \mathcal {L} (F (x _ {i} | \theta), y _ {i}) + \sum_ {i = 1} ^ {N _ {c} ^ {\prime}} \mathcal {L} (F (x _ {i} | \theta), y _ {i} + C).
$$

i=1expanded label space $\{ 0 , \cdots , 2 C - 1 \}$ i=1, and evaluate the remaining data with original labels $y \in \{ 0 , \cdots , C { - } 1 \}$ (4). As illustrated in Figure 2, there is a noticeable Subsequently, we randomly select 50% of the data for training, utilize thedecrease in the accuracy of clean data compared to previous experiments. This expanded label space {0, · · · , 2C   1}, and evaluate the remaining data withresult is intuitive, as introducing clean data with additional class labels can lead original labels y 2 {0, · · · , C 1}. As illustrated into these clean data points being misclassified as $y + C$ 2, there is a noticeable, thereby amplifying the decrease in the accuracy of clean data compared to previous experiments. Thisdifference between UEs and clean data. Moreover, as the number of iterations result is intuitive, as introducing clean data with additional class labels can leadincreases, the percentage of clean data with modified labels is expected to rise, to these clean data points being misclassified likely resulting in more misclassifications as $y + C$ C, thereby amplifying the, thereby further expanding difference between UEs and clean datathe gap between UEs and clean data.

## 4.2 Iterative Filtering (IF) for UEs Detection

The Iterative Filtering, as outlined in Algorithm 4.1, is designed to distinguish between UEs and clean samples, employing one subset for training and the other for testing and classification. Specifically, if a sample is incorrectly identified during testing, its class label is adjusted to $y + C$ , expanding the label space for the entire dataset to $\{ 0 , \cdots , 2 C - 1 \}$ . Consequently, we introduce C additional classes to the original classifier.

![](images/image_03.jpg)

Fig. 3. Performance Vs. Iterations on detecting EM with 80% poison ratio.

In essence, our approach leverages the inherent responses of models when they encounter UEs. Through iterative training and testing, and by categorizing samples based on the model’s prediction capabilities, we can effectively distinguish the UEs and clean samples within the dataset. During the iterative filtering process, poison samples tend to cluster within the initial $C$ categories, whereas clean samples are primarily categorized within the subsequent $C$ classes. However, in the early iterations, given a relatively low proportion of UEs in the training data, there exists a possibility that some UEs might be incorrectly identified as clean data during testing. As illustrated in Figure 3, as iterations increase, there is a corresponding rise in the FRR, indicating a growing number of poisoned samples being misclassified with each subsequent iteration. To address this, we propose the idea of conducting retrieval once after several iterations. Furthermore, we establish a stopping criterion based on the number of correctly classified clean data during each iteration. Conventionally, we set the threshold $N _ { \mathrm { t h r e } }$ to be 2% of the total number of the entire dataset, and $p$ to be 50% for splitting in each iteration.

## 5 Experiments

## 5.1 Experimental Setup

Datasets and models. We use three image classification datasets, CIFAR-10 [19], CIFAR-100 [19], and 100-class subset of ImageNet [6] in our experiments. We implement ResNet-18 [13] as the image classification model.

Methods for Generating UEs. We explore various representative methods to generate UEs: EM [16], REM [11] with a perturbation bound of $\ell _ { \infty }   =   \textstyle { \frac { 8 } { 2 5 5 } }$ AR [31], LSP [42] constrained by $\ell _ { 2 } = 1 . 0$ , and OPS [40] constrained by $\ell _ { 0 } = 1$ We adhere to the original papers to determine these bounds across various norms, ensuring that all introduced perturbations maintain uniform noise levels.

Poison Ratio (R). We select the ratio from {20, 40, 60, 80}% to demonstrate the effectiveness of our methods across different poisoning ratios.

Competing Methods. As our proposed IF is the first UEs detection method without any baselines, we re-implement several state-of-the-art defensive techniques from backdoor attacks for comparison. TeCo [21] is an innovative test-time poisoned sample detection method designed for backdoor attacks. It evaluates the consistency of test-time robustness by measuring the extent of severity deviation, which triggers shifts in predictions across different corruptions. Deep k-NN [29] is a straightforward, yet highly-effective detection against clean-label backdoor attacks, and exploits the property that adversarial examples have different feature distributions than their clean counterparts in deeper layers of the network. Furthermore, these adversarial features are typically aligned closely to the distribution of the target class.

Table 1. Detection performance (%) on CIFAR-10 dataset.

<table><tr><td rowspan="2">R(%)</td><td rowspan="2">UEs</td><td colspan="4">Deep k-NN [29]</td><td colspan="4">TeCo [21]</td><td colspan="4">Our Method</td></tr><tr><td>Acc. ↑</td><td>FAR ↓</td><td>FRR ↓</td><td>HTER ↓</td><td>Acc. ↑</td><td>FAR ↓</td><td>FRR ↓</td><td>HTER ↓</td><td>Acc. ↑</td><td>FAR ↓</td><td>FRR ↓</td><td>HTER ↓</td></tr><tr><td rowspan="5">20</td><td>EM</td><td>47.7</td><td>3.8</td><td>73.4</td><td>38.6</td><td>49.6</td><td>19.5</td><td>79.6</td><td>49.5</td><td>83.3</td><td>5.7</td><td>17.1</td><td>11.4</td></tr><tr><td>OPS</td><td>46.3</td><td>2.3</td><td>73.4</td><td>37.8</td><td>51.5</td><td>19.9</td><td>79.9</td><td>49.9</td><td>91.1</td><td>4.7</td><td>9.5</td><td>7.1</td></tr><tr><td>LSP</td><td>38.0</td><td>16.5</td><td>78.3</td><td>47.4</td><td>49.5</td><td>19.6</td><td>79.7</td><td>49.6</td><td>79.3</td><td>60.7</td><td>19.3</td><td>40.0</td></tr><tr><td>REM</td><td>38.2</td><td>16.3</td><td>78.4</td><td>47.4</td><td>48.5</td><td>20.2</td><td>80.2</td><td>50.2</td><td>79.4</td><td>72.6</td><td>19.9</td><td>46.2</td></tr><tr><td>AR</td><td>38.6</td><td>19.3</td><td>79.6</td><td>49.5</td><td>51.4</td><td>19.7</td><td>79.6</td><td>49.7</td><td>78.7</td><td>78.0</td><td>20.0</td><td>49.0</td></tr><tr><td rowspan="5">40</td><td>EM</td><td>69.9</td><td>3.3</td><td>42.9</td><td>23.1</td><td>49.5</td><td>40.3</td><td>60.2</td><td>50.3</td><td>88.9</td><td>0.1</td><td>15.6</td><td>7.9</td></tr><tr><td>OPS</td><td>61.0</td><td>21.5</td><td>49.4</td><td>35.4</td><td>48.5</td><td>40.5</td><td>60.4</td><td>50.5</td><td>92.9</td><td>0.6</td><td>10.0</td><td>5.4</td></tr><tr><td>LSP</td><td>64.2</td><td>12.1</td><td>46.7</td><td>29.9</td><td>49.7</td><td>39.5</td><td>59.6</td><td>49.6</td><td>80.2</td><td>3.0</td><td>24.4</td><td>13.7</td></tr><tr><td>REM</td><td>57.2</td><td>27.0</td><td>52.3</td><td>39.7</td><td>49.5</td><td>39.9</td><td>59.9</td><td>49.9</td><td>82.2</td><td>4.6</td><td>22.1</td><td>13.3</td></tr><tr><td>AR</td><td>48.1</td><td>36.8</td><td>58.3</td><td>47.6</td><td>48.8</td><td>40.1</td><td>60.1</td><td>50.1</td><td>60.0</td><td>43.0</td><td>40.0</td><td>41.5</td></tr><tr><td rowspan="5">60</td><td>EM</td><td>74.4</td><td>29.5</td><td>23.3</td><td>26.4</td><td>51.1</td><td>59.6</td><td>39.7</td><td>49.7</td><td>95.9</td><td>4.1</td><td>3.9</td><td>4.0</td></tr><tr><td>OPS</td><td>70.8</td><td>32.4</td><td>6.2</td><td>19.3</td><td>50.3</td><td>60.0</td><td>40.0</td><td>50.0</td><td>95.3</td><td>2.4</td><td>8.0</td><td>5.2</td></tr><tr><td>LSP</td><td>60.0</td><td>50.4</td><td>30.6</td><td>40.5</td><td>49.7</td><td>61.4</td><td>41.1</td><td>51.2</td><td>94.0</td><td>4.3</td><td>8.5</td><td>6.4</td></tr><tr><td>REM</td><td>68.8</td><td>35.9</td><td>29.1</td><td>32.5</td><td>49.5</td><td>60.6</td><td>40.6</td><td>50.6</td><td>85.0</td><td>16.0</td><td>13.0</td><td>14.5</td></tr><tr><td>AR</td><td>40.5</td><td>59.5</td><td>39.8</td><td>49.6</td><td>50.3</td><td>60.1</td><td>40.1</td><td>50.1</td><td>40.8</td><td>14.1</td><td>59.7</td><td>36.9</td></tr><tr><td rowspan="5">80</td><td>EM</td><td>43.2</td><td>78.7</td><td>17.6</td><td>48.1</td><td>51.2</td><td>79.8</td><td>19.8</td><td>49.8</td><td>98.1</td><td>2.1</td><td>0.9</td><td>1.5</td></tr><tr><td>OPS</td><td>70.5</td><td>35.6</td><td>25.9</td><td>30.7</td><td>50.1</td><td>80.4</td><td>20.4</td><td>50.4</td><td>96.0</td><td>0.1</td><td>16.5</td><td>8.3</td></tr><tr><td>LSP</td><td>75.6</td><td>57.0</td><td>10.1</td><td>33.5</td><td>52.7</td><td>81.0</td><td>20.8</td><td>50.9</td><td>96.1</td><td>3.2</td><td>7.0</td><td>5.1</td></tr><tr><td>REM</td><td>58.8</td><td>72.5</td><td>13.2</td><td>42.9</td><td>56.6</td><td>79.9</td><td>20.0</td><td>50.0</td><td>90.5</td><td>8.8</td><td>13.4</td><td>11.1</td></tr><tr><td>AR</td><td>49.7</td><td>79.8</td><td>19.9</td><td>49.9</td><td>49.4</td><td>80.7</td><td>20.7</td><td>50.7</td><td>82.4</td><td>17.2</td><td>26.2</td><td>21.7</td></tr><tr><td colspan="2">Average</td><td>56.1</td><td>34.5</td><td>42.4</td><td>38.5</td><td>50.4</td><td>50.1</td><td>50.1</td><td>50.1</td><td>84.5</td><td>17.3</td><td>17.8</td><td>17.5</td></tr></table>

Metrics. To evaluate the performance of UEs detection, we adopt the Half Total Error Rate (HTER) and the accuracy (Acc.) as metrics. HTER, the False Rejection Rate (FRR) and the False Acceptance Rate (FAR) are given by:

$$
\mathrm{HTER} = \frac {\mathrm{FRR} + \mathrm{FAR}}{2}, \mathrm{FAR} = \frac {\mathrm{FP}}{\mathrm{FP} + \mathrm{TN}}, \mathrm{FRR} = \frac {\mathrm{FN}}{\mathrm{FN} + \mathrm{TP}}. \tag {5}
$$

HTER integrates both FRR and FAR to provide a holistic view.

## 5.2 Experimental Results

Results on CIFAR-10 dataset. We evaluate the performance of IF on different UEs methods comprehensively. The results in Table 1 demonstrate that IF can successfully identify the UEs, particularly when the poison ratio exceeds 20%, as confirmed by the fact that most HTER valUE below 10%. However, with only a 20% poison ratio, the poisoned samples represent a minority, and additional experiments reveal that a marginal percentage of UEs does not significantly affect testing performance. Therefore, the effectiveness of detection becomes more critical when dealing with larger poison ratios. Notably, on the CIFAR-10, IF achieves an average HTER of 0.175, F1 score of 0.845, FAR of 0.173, and FRR of 0.178. These results showcase IF’s superiority, surpassing the runner-up by roughly 20% in HTER, 30% in Acc., 17% in FAR, and 25% in FRR. Certainly, there are scenarios in which IF may not achieve success, particularly when dealing with AR. AR utilize autoregressive perturbations, making them more complex and not linearly separable [31]. This complexity can make them more challenging to detect, especially when the poison ratios are low. Figure 3 illustrates the performance at each iteration. It is noticeable that once retrieval is integrated and the filtering process continUE through several iterations, the method demonstrates improved convergence performance compared to the results preceding retrieval. In summary, our work delivers consistent effectiveness across various types of UEs without using extra knowledge.

Table 2. Performance (%) on CIFAR-100 dataset.

<table><tr><td rowspan="2">R(%)</td><td rowspan="2">UEs</td><td colspan="4">Deep k-NN [29]</td><td colspan="4">TeCo [21]</td><td colspan="4">Our Method</td></tr><tr><td>Acc. ↑</td><td>FAR ↓</td><td>FRR ↓</td><td>HTER ↓</td><td>Acc. ↑</td><td>FAR ↓</td><td>FRR ↓</td><td>HTER ↓</td><td>Acc. ↑</td><td>FAR ↓</td><td>FRR ↓</td><td>HTER ↓</td></tr><tr><td rowspan="3">20</td><td>EM</td><td>52.1</td><td>10.8</td><td>73.5</td><td>42.2</td><td>45.8</td><td>19.9</td><td>80.0</td><td>50.0</td><td>81.3</td><td>37.9</td><td>17.6</td><td>27.7</td></tr><tr><td>OPS</td><td>50.8</td><td>10.2</td><td>73.6</td><td>41.9</td><td>48.1</td><td>19.9</td><td>79.9</td><td>49.9</td><td>80.9</td><td>37.8</td><td>18.4</td><td>28.1</td></tr><tr><td>LSP</td><td>45.5</td><td>17.8</td><td>78.4</td><td>48.1</td><td>47.7</td><td>19.9</td><td>79.9</td><td>49.9</td><td>79.2</td><td>79.7</td><td>20.0</td><td>49.8</td></tr><tr><td rowspan="3">40</td><td>EM</td><td>73.1</td><td>2.0</td><td>40.0</td><td>21.0</td><td>48.3</td><td>40.5</td><td>60.4</td><td>50.5</td><td>93.0</td><td>4.2</td><td>8.5</td><td>6.4</td></tr><tr><td>OPS</td><td>69.7</td><td>5.5</td><td>42.7</td><td>24.1</td><td>48.9</td><td>40.9</td><td>60.8</td><td>50.8</td><td>90.4</td><td>2.9</td><td>12.7</td><td>7.8</td></tr><tr><td>LSP</td><td>66.9</td><td>8.3</td><td>44.7</td><td>26.5</td><td>45.9</td><td>42.6</td><td>61.8</td><td>52.2</td><td>78.6</td><td>11.7</td><td>24.5</td><td>18.1</td></tr><tr><td rowspan="3">60</td><td>EM</td><td>90.3</td><td>2.9</td><td>12.9</td><td>7.9</td><td>50.8</td><td>59.9</td><td>39.9</td><td>49.9</td><td>96.9</td><td>4.0</td><td>1.7</td><td>2.9</td></tr><tr><td>OPS</td><td>68.8</td><td>3.7</td><td>43.6</td><td>23.7</td><td>49.7</td><td>61.0</td><td>40.9</td><td>50.9</td><td>92.1</td><td>1.7</td><td>15.2</td><td>8.5</td></tr><tr><td>LSP</td><td>85.3</td><td>7.4</td><td>17.8</td><td>12.6</td><td>50.6</td><td>60.3</td><td>40.2</td><td>50.3</td><td>92.8</td><td>6.3</td><td>8.6</td><td>7.4</td></tr><tr><td rowspan="3">80</td><td>EM</td><td>42.3</td><td>75.7</td><td>9.0</td><td>42.3</td><td>57.0</td><td>81.0</td><td>20.6</td><td>50.8</td><td>97.5</td><td>2.6</td><td>1.8</td><td>2.2</td></tr><tr><td>OPS</td><td>71.2</td><td>60.3</td><td>5.2</td><td>32.8</td><td>49.5</td><td>81.0</td><td>20.9</td><td>50.9</td><td>96.1</td><td>2.0</td><td>11.4</td><td>6.7</td></tr><tr><td>LSP</td><td>71.8</td><td>59.4</td><td>4.8</td><td>32.1</td><td>56.8</td><td>77.4</td><td>18.1</td><td>47.7</td><td>97.1</td><td>1.6</td><td>8.1</td><td>4.9</td></tr><tr><td colspan="2">Average</td><td>59.2</td><td>33.2</td><td>41.5</td><td>37.4</td><td>52.2</td><td>47.5</td><td>50.2</td><td>49.3</td><td>93.7</td><td>11.5</td><td>4.1</td><td>7.9</td></tr></table>

Table 3. Performance (%) on 100-class ImageNet subset.

<table><tr><td rowspan="2">R(%)</td><td rowspan="2">UEs</td><td colspan="4">Deep k-NN [29]</td><td colspan="4">TeCo [21]</td><td colspan="4">Our Method</td></tr><tr><td>Acc. ↑</td><td>FAR ↓</td><td>FRR ↓</td><td>HTER ↓</td><td>Acc. ↑</td><td>FAR ↓</td><td>FRR ↓</td><td>HTER ↓</td><td>Acc. ↑</td><td>FAR ↓</td><td>FRR ↓</td><td>HTER ↓</td></tr><tr><td rowspan="3">20</td><td>EM</td><td>41.4</td><td>19.9</td><td>80.1</td><td>50.0</td><td>49.8</td><td>20.3</td><td>80.2</td><td>50.2</td><td>93.0</td><td>23.6</td><td>1.5</td><td>12.6</td></tr><tr><td>OPS</td><td>42.3</td><td>18.2</td><td>79.1</td><td>48.6</td><td>50.3</td><td>20.8</td><td>80.1</td><td>50.9</td><td>90.1</td><td>26.6</td><td>5.3</td><td>16.0</td></tr><tr><td>LSP</td><td>49.9</td><td>6.6</td><td>73.0</td><td>39.8</td><td>50.2</td><td>17.9</td><td>78.1</td><td>48.0</td><td>88.8</td><td>22.8</td><td>9.0</td><td>15.9</td></tr><tr><td rowspan="3">40</td><td>EM</td><td>47.1</td><td>40.3</td><td>59.9</td><td>50.1</td><td>48.6</td><td>40.9</td><td>60.8</td><td>50.8</td><td>94.0</td><td>11.4</td><td>1.7</td><td>6.5</td></tr><tr><td>OPS</td><td>60.2</td><td>18.0</td><td>49.9</td><td>33.9</td><td>49.0</td><td>41.1</td><td>61.1</td><td>51.1</td><td>91.7</td><td>12.7</td><td>5.0</td><td>8.8</td></tr><tr><td>LSP</td><td>68.6</td><td>4.2</td><td>43.6</td><td>23.9</td><td>55.1</td><td>34.5</td><td>54.8</td><td>44.6</td><td>92.0</td><td>15.4</td><td>1.6</td><td>8.5</td></tr><tr><td rowspan="3">60</td><td>EM</td><td>52.9</td><td>60.3</td><td>39.8</td><td>50.1</td><td>48.9</td><td>62.0</td><td>41.7</td><td>51.9</td><td>95.8</td><td>6.3</td><td>0.1</td><td>3.4</td></tr><tr><td>OPS</td><td>77.9</td><td>23.2</td><td>21.5</td><td>23.4</td><td>48.5</td><td>61.2</td><td>41.3</td><td>51.3</td><td>94.6</td><td>4.8</td><td>6.2</td><td>5.5</td></tr><tr><td>LSP</td><td>85.9</td><td>3.9</td><td>18.1</td><td>11.0</td><td>56.7</td><td>54.0</td><td>35.8</td><td>44.9</td><td>95.7</td><td>4.8</td><td>3.5</td><td>4.1</td></tr><tr><td rowspan="3">80</td><td>EM</td><td>58.8</td><td>79.7</td><td>19.7</td><td>49.7</td><td>49.3</td><td>81.0</td><td>21.0</td><td>51.0</td><td>97.2</td><td>3.1</td><td>1.0</td><td>2.1</td></tr><tr><td>OPS</td><td>80.4</td><td>49.2</td><td>5.6</td><td>27.4</td><td>49.5</td><td>80.7</td><td>20.7</td><td>50.7</td><td>94.7</td><td>4.6</td><td>8.3</td><td>6.5</td></tr><tr><td>LSP</td><td>44.9</td><td>74.9</td><td>7.2</td><td>41.0</td><td>57.7</td><td>76.0</td><td>17.0</td><td>46.5</td><td>96.2</td><td>3.2</td><td>6.1</td><td>4.7</td></tr><tr><td colspan="2">Average</td><td>59.2</td><td>33.2</td><td>41.5</td><td>37.4</td><td>52.2</td><td>47.5</td><td>50.2</td><td>49.3</td><td>93.7</td><td>11.5</td><td>4.1</td><td>7.9</td></tr></table>

Table 4. Ablation study of IF on CIFAR-10. ES is the use of early stopping, and AC is the use of additional classes.

<table><tr><td rowspan="2">UEs</td><td rowspan="2">Methods</td><td colspan="2">40%</td><td colspan="2">60%</td><td colspan="2">80%</td><td colspan="2">Average</td></tr><tr><td>Acc. ↑</td><td>HTER ↓</td><td>Acc. ↑</td><td>HTER ↓</td><td>Acc. ↑</td><td>HTER ↓</td><td>Acc. ↑</td><td>HTER ↓</td></tr><tr><td rowspan="4">EM</td><td>w/o ES</td><td>96.5</td><td>3.9</td><td>88.9</td><td>5.8</td><td>90.5</td><td>4.8</td><td>92.0</td><td>4.8</td></tr><tr><td>w/o retrieve</td><td>94.5</td><td>5.2</td><td>91.4</td><td>8.9</td><td>97.1</td><td>5.1</td><td>94.3</td><td>6.4</td></tr><tr><td>w/o AC</td><td>82.9</td><td>15.7</td><td>78.5</td><td>13.5</td><td>94.6</td><td>4.0</td><td>85.3</td><td>11.1</td></tr><tr><td>Ours</td><td>88.9</td><td>7.9</td><td>96.0</td><td>4.0</td><td>98.1</td><td>1.5</td><td>94.3</td><td>4.5</td></tr><tr><td rowspan="4">OPS</td><td>w/o ES</td><td>94.0</td><td>6.4</td><td>92.3</td><td>6.7</td><td>93.2</td><td>8.9</td><td>93.2</td><td>7.3</td></tr><tr><td>w/o retrieve</td><td>94.3</td><td>4.8</td><td>94.5</td><td>5.9</td><td>94.8</td><td>10.0</td><td>94.5</td><td>6.9</td></tr><tr><td>w/o AC</td><td>83.9</td><td>15.9</td><td>87.8</td><td>10.7</td><td>92.3</td><td>9.6</td><td>88.0</td><td>12.1</td></tr><tr><td>Ours</td><td>93.0</td><td>5.4</td><td>95.3</td><td>5.2</td><td>96.0</td><td>8.3</td><td>94.8</td><td>6.3</td></tr></table>

There are also some interesting findings about baselines. TeCo, though highly effective in detecting poisoned samples for backdoor attacks, appears to completely fail when dealing with UEs. This divergence in performance could be attributed to the differing objectives of backdoor attacks and UEs. Backdoor attacks aim to manipulate the predictions, shifting them from the source class to the target class after the integration of triggers. On the other hand, UEs do not necessarily require a prediction shift after the perturbations are introduced. This difference in objectives illuminates the varying performance of TeCo in these diverse situations. Deep k-NN appears to be effective when the number of UEs and clean data is well balanced. However, it seems to fail when dealing with a critical scenario where UEs significantly outnumber the clean data. This indicates that Deep k-NN’s efficiency is closely tied to data distribution, particularly when there is a significant imbalance between UEs and clean data.

Results on CIFAR-100 dataset and 100-class ImageNet subset. In our experiments on both the 100-class datasets, we select the three most representative unlearnable examples methodologies. As the experimental results are shown in Table 2 and Table 3, our IF strategy consistently outperforms competing methods in terms of detection and defense against UEs.

## 5.3 Ablation Study

We conduct experiments to show the effectiveness of the proposed iterative filtering, comparing it with the method without early stopping, without the retrieval process and without introducing additional classes. The experimental results are shown in Table 4. The results indicate that all introduced modules in IF contribute to the enhancement of detection performance, as measured by the average metrics. Among these strategies, the introduction of additional classes has proven to be the most effective. While early stopping may slightly impact performance at a low poison ratio, it is highly efficient, requiring lower computations, and performs well at high poison ratios. In particular, the retrieval process has proven effective, notably in preserving high accuracy while slightly improving the HTER.

![](images/image_04.jpg)

<details>
<summary>scatter</summary>

| Series | Dimension 1 (range) | Dimension 2 (range) |
| --- | --- | --- |
| Clean data | -70~70 | -60~65 |
| UEs | -70~70 | -60~70 |
</details>

(a) w/o additional clean data

![](images/image_05.jpg)

<details>
<summary>scatter</summary>

| Series | Dimension 1 (range) | Dimension 2 (range) |
| --- | --- | --- |
| Clean data | -60~30 | -50~30 |
| UEs | -60~70 | -80~60 |
</details>

(b) w/ additional clean data

Fig. 4. t-SNE [24] visualizations on CIFAR-10, comparing models trained without and with additional clean data, where the labels for the additional clean data are updated to $y \in [ C , 2 C - 1 ]$ . Note that UEs are generated by EM, and the shared training data consists of 50% UEs and 50% clean data.

Table 5. Test Accuracy (%) of models trained on CIFAR-10 with various defensive methods. Dp denote Diffpure [8].

<table><tr><td>Ratio (%)</td><td>Defensive method</td><td>EM</td><td>OPS</td><td>LSP</td><td>REM</td><td>AR</td><td>Mean</td></tr><tr><td rowspan="3">60</td><td>w/o</td><td>91.86</td><td>91.76</td><td>91.89</td><td>91.71</td><td>91.68</td><td>91.78</td></tr><tr><td>Dp on all samples</td><td>89.45</td><td>69.55</td><td>89.80</td><td>90.02</td><td>89.39</td><td>85.64</td></tr><tr><td>Dp on detected samples</td><td>92.90</td><td>91.83</td><td>92.83</td><td>91.51</td><td>91.79</td><td>92.17</td></tr><tr><td rowspan="3">80</td><td>w/o</td><td>87.94</td><td>87.17</td><td>88.19</td><td>86.58</td><td>87.51</td><td>87.47</td></tr><tr><td>Dp on all samples</td><td>89.86</td><td>70.39</td><td>89.73</td><td>89.87</td><td>89.70</td><td>85.91</td></tr><tr><td>Dp on detected samples</td><td>91.48</td><td>89.87</td><td>90.22</td><td>91.52</td><td>89.97</td><td>90.61</td></tr></table>

Furthermore, through t-SNE [24], we illustrate the feature distribution of the model, comparing training scenarios without and with the additional clean data (having updated labels $y \in [ C , 2 C - 1 ] )$ , as shown in Figure 4. It is evident that the additional clean data serves to separate the clean data from the UEs in the latent space, thereby improving the detection performance.

## 5.4 Detection for Purification

In this section, we demonstrate the effectiveness of our approach by conducting experiments dedicated to defending against UEs while adhering to detectiondefense principles. It is worth noting that we choose Diffpure [8], which utilizes a diffusion model for purification, as the defense method in these experiments. As evident in Table 5, our proposed detection methods prove highly effective in improving defense performance, particularly in scenarios with significant poison ratios. All experiments were conducted on CIFAR-10 dataset.

## 6 Conclusion

In this paper, we present Iterative Filtering (IF), a robust and efficient detection technique designed to identify a wide range of visually imperceptible UEs. Our approach leverages a critical property: UEs tend to be acquired by the classifier more rapidly than clean data when training on a partially poisoned dataset. This difference inspires us to introduce an iterative algorithm for data separation. Moreover, we highlight that distinguishing between clean and poisoned samples is more effective. Through the introduction of the additional classes and the adoption of iterative refinement, our proposed approach achieves improved effectiveness in classifying these two sample types. Extensive experiments conclusively demonstrate that our method outperforms state-of-the-art detection approaches when challenged with various types of attacks, datasets, and poison ratios.

Acknowledgement. This work was done at Rapid-Rich Object Search (ROSE) Lab, School of Electrical & Electronic Engineering, Nanyang Technological University. This research is supported in part by the NTU-PKU Joint Research Institute and the DSO National Laboratories, Singapore, under the project agreement No. DSOCL22332.

## References

1. Biggio, B., Nelson, B., Laskov, P.: Poisoning attacks against support vector machines. In: Proc. Int’l Conf. Machine Learning (2012)
2. Brickley, D., Burgess, M., Noy, N.: Google dataset search: Building a search engine for datasets in an open web ecosystem. In: The world wide web conference (2019)
3. Chen, B., Carvalho, W., Baracaldo, N., Ludwig, H., Edwards, B., Lee, T., Molloy, I., Srivastava, B.: Detecting backdoor attacks on deep neural networks by activation clustering. arXiv preprint arXiv:1811.03728 (2018)
4. Chen, S., Yuan, G., Cheng, X., Gong, Y., Qin, M., Wang, Y., Huang, X.: Self-ensemble protection: Training checkpoints are good data protectors. In: Proc. Int’l Conf. Learning Representations (2023)
5. Chen, X., Liu, C., Li, B., Lu, K., Song, D.: Targeted backdoor attacks on deep learning systems using data poisoning. arXiv preprint arXiv:1712.05526 (2017)
6. Deng, J., Dong, W., Socher, R., Li, L.J., Li, K., Fei-Fei, L.: Imagenet: A largescale hierarchical image database. In: Proc. IEEE Int’l Conf. Computer Vision and Pattern Recognition. pp. 248–255 (2009)
7. Doan, K., Lao, Y., Zhao, W., Li, P.: Lira: Learnable, imperceptible and robust backdoor attacks. In: Proc. IEEE Int’l Conf. Computer Vision. pp. 11966–11976 (2021)
8. Dolatabadi, H.M., Erfani, S., Leckie, C.: The devil’s advocate: Shattering the illusion of unexploitable data using diffusion models. arXiv preprint arXiv:2303.08500 (2023)
9. Fowl, L., Goldblum, M., Chiang, P.y., Geiping, J., Czaja, W., Goldstein, T.: Adversarial examples make strong poisons. In: Proc. Annual Conf. Neural Information Processing Systems. vol. 34, pp. 30339–30351 (2021)
10. Frénay, B., Verleysen, M.: Classification in the presence of label noise: a survey. IEEE Trans. on Neural Networks and Learning Systems 25(5), 845–869 (2013)
11. Fu, S., He, F., Liu, Y., Shen, L., Tao, D.: Robust unlearnable examples: Protecting data privacy against adversarial learning. In: Proc. Int’l Conf. Learning Representations (2022)
12. Gu, T., Liu, K., Dolan-Gavitt, B., Garg, S.: Badnets: Evaluating backdooring attacks on deep neural networks. IEEE Access 7, 47230–47244 (2019)
13. He, K., Zhang, X., Ren, S., Sun, J.: Deep residual learning for image recognition. In: Proc. IEEE Int’l Conf. Computer Vision and Pattern Recognition. pp. 770–778 (2016)
14. Ho, J., Jain, A., Abbeel, P.: Denoising diffusion probabilistic models. In: Proc. Annual Conf. Neural Information Processing Systems. pp. 6840–6851 (2020)
15. Hong, S., Chandrasekaran, V., Kaya, Y., Dumitraş, T., Papernot, N.: On the effectiveness of mitigating data poisoning attacks with gradient shaping. arXiv preprint arXiv:2002.11497 (2020)
16. Huang, H., Ma, X., Erfani, S.M., Bailey, J., Wang, Y.: Unlearnable examples: Making personal data unexploitable. In: Proc. Int’l Conf. Learning Representations (2021)
17. Huang, W.R., Geiping, J., Fowl, L., Taylor, G., Goldstein, T.: Metapoison: Practical general-purpose clean-label data poisoning. In: Proc. Annual Conf. Neural Information Processing Systems. vol. 33, pp. 12080–12091 (2020)
18. Koh, P.W., Steinhardt, J., Liang, P.: Stronger data poisoning attacks break data sanitization defenses. Machine Learning pp. 1–47 (2022)
19. Krizhevsky, A., Hinton, G., et al.: Learning multiple layers of features from tiny images (2009)
20. Lin, X., Yu, Y., Xia, S., Jiang, J., Wang, H., Yu, Z., Liu, Y., Fu, Y., Wang, S., Tang, W., et al.: Safeguarding medical image segmentation datasets against unauthorized training via contour-and texture-aware perturbations. arXiv preprint arXiv:2403.14250 (2024)
21. Liu, X., Li, M., Wang, H., Hu, S., Ye, D., Jin, H., Wu, L., Xiao, C.: Detecting backdoors during the inference stage based on corruption robustness consistency. In: Proc. IEEE Int’l Conf. Computer Vision and Pattern Recognition. pp. 16363–16372 (2023)
22. Liu, Y., Ma, X., Bailey, J., Lu, F.: Reflection backdoor: A natural backdoor attack on deep neural networks. In: Proc. IEEE European Conf. Computer Vision. pp. 182–199 (2020)
23. Liu, Z., Zhao, Z., Larson, M.: Image shortcut squeezing: Countering perturbative availability poisons with compression. In: Proc. Int’l Conf. Machine Learning (2023)
24. Van der Maaten, L., Hinton, G.: Visualizing data using t-sne. Journal of machine learning research 9(11) (2008)
25. Madry, A., Makelov, A., Schmidt, L., Tsipras, D., Vladu, A.: Towards deep learning models resistant to adversarial attacks. In: Proc. Int’l Conf. Learning Representations (2018)
26. Marcellin, M.W., Gormish, M.J., Bilgin, A., Boliek, M.P.: An overview of jpeg-2000. In: Proceedings DCC 2000. Data Compression Conference. pp. 523–541. IEEE (2000)
27. Meng, R., Yi, C., Yu, Y., Yang, S., Shen, B., Kot, A.C.: Semantic deep hiding for robust unlearnable examples. IEEE Trans. on Information Forensics and Security (2024)
28. Nguyen, A., Tran, A.: Wanet–imperceptible warping-based backdoor attack. In: Proc. Int’l Conf. Learning Representations (2021)
29. Peri, N., Gupta, N., Huang, W.R., Fowl, L., Zhu, C., Feizi, S., Goldstein, T., Dickerson, J.P.: Deep k-nn defense against clean-label data poisoning attacks. In: Proc. IEEE European Conf. Computer Vision. pp. 55–70 (2020)
30. Qin, T., Gao, X., Zhao, J., Ye, K., Xu, C.Z.: Learning the unlearnable: Adversarial augmentations suppress unlearnable example attacks. arXiv preprint arXiv:2303.15127 (2023)
31. Sandoval-Segura, P., Singla, V., Geiping, J., Goldblum, M., Goldstein, T., Jacobs, D.: Autoregressive perturbations for data poisoning. In: Proc. Annual Conf. Neural Information Processing Systems. vol. 35, pp. 27374–27386 (2022)
32. Shafahi, A., Huang, W.R., Najibi, M., Suciu, O., Studer, C., Dumitras, T., Goldstein, T.: Poison frogs! targeted clean-label poisoning attacks on neural networks. In: Proc. Annual Conf. Neural Information Processing Systems (2018)
33. Steinhardt, J., Koh, P.W.W., Liang, P.S.: Certified defenses for data poisoning attacks. In: Proc. Annual Conf. Neural Information Processing Systems. vol. 30 (2017)
34. Tommasi, T., Patricia, N., Caputo, B., Tuytelaars, T.: A deeper look at dataset bias. Domain adaptation in computer vision applications pp. 37–55 (2017)
35. Tran, B., Li, J., Madry, A.: Spectral signatures in backdoor attacks. In: Proc. Annual Conf. Neural Information Processing Systems. vol. 31 (2018)
36. Wang, B., Yao, Y., Shan, S., Li, H., Viswanath, B., Zheng, H., Zhao, B.Y.: Neural cleanse: Identifying and mitigating backdoor attacks in neural networks. In: 2019 IEEE Symposium on Security and Privacy (SP). pp. 707–723. IEEE (2019)
37. Wang, C., Yu, Y., Guo, L., Wen, B.: Benchmarking adversarial robustness of image shadow removal with shadow-adaptive attacks. In: Proc. IEEE Int’l Conf. Acoustics, Speech, and Signal Processing. pp. 13126–13130. IEEE (2024)
38. Wang, X., Hu, S., Zhang, Y., Zhou, Z., Zhang, L.Y., Xu, P., Wan, W., Jin, H.: Eclipse: Expunging clean-label indiscriminate poisons via sparse diffusion purification. arXiv preprint arXiv:2406.15093 (2024)
39. Whang, S.E., Roh, Y., Song, H., Lee, J.G.: Data collection and quality challenges in deep learning: A data-centric ai perspective. The VLDB Journal (2023)
40. Wu, S., Chen, S., Xie, C., Huang, X.: One-pixel shortcut: on the learning preference of deep neural networks. In: Proc. Int’l Conf. Learning Representations (2023)
41. Xia, S., Yu, Y., Jiang, X., Ding, H.: Mitigating the curse of dimensionality for certified robustness via dual randomized smoothing. In: The Twelfth International Conference on Learning Representations (2024)
42. Yu, D., Zhang, H., Chen, W., Yin, J., Liu, T.Y.: Availability attacks create shortcuts. In: Proceedings of the 28th ACM SIGKDD Conference on Knowledge Discovery and Data Mining. pp. 2367–2376 (2022)
43. Yu, Y., Wang, Y., Xia, S., Yang, W., Lu, S., Tan, Y.P., Kot, A.C.: Purify unlearnable examples via rate-constrained variational autoencoders. In: International Conference on Machine Learning, ICML 2024 (2024)
44. Yu, Y., Wang, Y., Yang, W., Lu, S., Tan, Y.P., Kot, A.C.: Backdoor attacks against deep image compression via adaptive frequency trigger. In: Proc. IEEE Int’l Conf. Computer Vision and Pattern Recognition. pp. 12250–12259 (June 2023)
45. Yu, Y., Yang, W., Tan, Y.P., Kot, A.C.: Towards robust rain removal against adversarial attacks: A comprehensive benchmark analysis and beyond. In: Proc. IEEE Int’l Conf. Computer Vision and Pattern Recognition. pp. 6013–6022 (June 2022)
46. Zha, D., Bhat, Z.P., Lai, K.H., Yang, F., Jiang, Z., Zhong, S., Hu, X.: Data-centric artificial intelligence: A survey. arXiv preprint arXiv:2303.10158 (2023)