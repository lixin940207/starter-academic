---
title: Deep learning technologies for long text classification of Legal documents

summary: The deep learning models that cannot be missed in the field of text classification mainly include FastText, TextCNN, HAN, and DPCNN.


# Link this post with a project
projects: []

# Date published
date: "2016-04-20T00:00:00Z"

# Date updated
lastmod: "2020-12-13T00:00:00Z"

# Is this an unpublished draft?
draft: false

# Show this page in the Featured widget?
featured: false

authors:
- admin

tags:
- nlp
- deep learning

categories:
- nlp
- deep learning
---

<p>In the field of NLP, tasks such as text classification and public opinion analysis are easier to obtain a large amount of labeled data than tasks such as text extraction and summarization. Therefore, in the field of text classification, deep learning is easier to obtain better results than traditional methods. It is precisely with the rapid evolution of text classification models that massive legal documents can be processed intelligently to greatly improve efficiency. Let's analyze the current state of art text classification model and their application in the intelligentization of legal documents. The deep learning models that cannot be missed in the field of text classification mainly include FastText, TextCNN, HAN, and DPCNN. This article attempts to summarize the theoretical framework of these classification models after practice, and link these models to each other, so that everyone can have some intuition and inspiration when choosing models and tuning parameters. In the field of deep learning, where practice is king, people often question the uselessness of theory. My personal feeling is that theory is very useful when screening models based on data characteristics. Secondly, it can greatly improve efficiency in the process of tuning parameters. More importantly When the results cannot be recalled, the phrase "this model should not be the result of this" and "this is unscientific" in my mind often provide confidence in holding on to the direction.</p>

### a. FastText
<p>Among them, the FastText structure is very simple, and it is suitable for occasions with particularly high speed requirements. He adds all the word vectors (and N-gram vectors) in an article directly to find the average, and then passes through a single-layer neural network to get the final The classification results. Obviously, this approach loses too much information for complex text classification tasks. A simple enhancement model of FastText is DAN. The change is that after the word vector averaging is completed, several layers of fully connected neural networks are stacked. Correspondingly, FastText can also be regarded as a special case of DAN fully connected neural network with 1 layer.</p>
<p>It is important to note that for the FastText model without the n-gram vector, it is impossible for him to distinguish the position of the negative word. Look at the following two sentences:</p>

 > I don't like this kind of movies, but I like this one. I like this kind of movies, but I don't like this one.
    
<p>Such two sentences are exactly the same when they are sent to the single-layer neural network after the word vector averaging. It is impossible for the classifier to distinguish the difference between the two sentences. The difference can only be made after adding the n-gram feature. Therefore, you need to have a sufficient understanding of your data in actual applications.</p>

### b.TextCNN
<p>
Compared with the fastText model, the structure of TextCNN is more complicated. In 2014, he proposed that he used convolution + max pooling, which are two very successful combinations of good friends in the image field. Let's take a look at his structure first. As shown in the figure below, the input of the first layer of the schematic diagram is a 7*5 word vector matrix, where the word vector dimension is 5 and the sentence length is 7, and then the second layer uses 3 sets of rolls with widths of 2, 3, and 4 respectively. Convolution kernel, two convolution kernels of each width are used in the figure. Among them, each convolution kernel slides over the entire sentence length to obtain n activation values. In the figure, the convolution kernel slides without padding, so the convolution kernel with a width of 4 slides on a sentence of length 7 to obtain 4 Eigenvalues. Then what comes out is the global pooling of good base friends of convolution. The feature value column vector output by each convolution kernel is obtained by taking the maximum value over the entire sentence length to obtain a feature map composed of 6 feature values for the subsequent classifier As a basis for classification.
</p>
{{< figure src="0_0efgxnFIaLTZ2qkY.png" title="TextCNN Structure" >}}

<p>We know that the role of convolution in image processing is to calculate the similarity between each local area and the convolution kernel in the entire image. Generally, the first few layers of convolution kernels can be easily visualized, and the results of the visualization are the first few The convolution kernel of the layer is to find some simple lines in the original input image. The convolution kernel in NLP can't be visualized, so can you not understand what he is doing? In fact, you can infer its role through the structure of the model. Because the convolution in TextCNN is directly global max pooling, then it can only calculate the similarity with certain keywords during the convolution process, and then use the max pooling layer to get the model to focus on whether those keywords are in the entire input text Appears in and how similar is the most similar keyword to the convolution kernel. We assume that the Chinese output is a word vector. Ideally, a convolution kernel represents a keyword, as shown in the following figure:</p>
{{< figure src="2.png" title="TextCNN Structure" >}}
<p>For example, in a two-category public opinion analysis task, if the entire model is regarded as a black box, then to check its output results, you will find that the model is particularly sensitive to whether the input text contains words like "like" and "love", then How did he do it? In the entire model, only the convolution part can be used to traverse the entire sentence to calculate the keyword similarity, because the max pooling of the entire sentence length is directly behind. But because the model faces word vectors, not words, one of his convolution kernels may have learned only half of the keyword word vector, and then another convolution kernel has learned the other half of the keyword word vector. Finally, these feature values ​​are accumulated in the classifier to get the final result. The biggest problem with the TextCNN model is that the global max pooling loses structural information, so it is difficult to find complex patterns such as transition relationships in the text. TextCNN can only know which keywords appear in the text and the similarity intensity distribution. It is impossible to know which keywords appear several times and the order in which they appear. Imagine that if this intermediate result is judged by humans, it is also difficult for humans to obtain classification results for complex texts, so machines obviously cannot do it. To solve this problem, you can try k-max pooling to do some optimizations. K-max pooling not only retains the largest value for each convolution kernel, it retains the first k largest values, and retains the order in which these values ​​appear, that is The k maximum values ​​are arranged in order of position in the text. Compared with 1-max pooling on some more complex texts, it will be improved.</p>

### c. HAN(Hierarchy Attention Network)
<p>Compared with TextCNN, the biggest improvement of HAN is that it completely retains the structural information of the article, and what is particularly commendable is that the attention structure has a strong explanatory nature. His structure is shown below:</p>
{{< figure src="han.png" title="HAN Structure" >}}

<p>After inputting the word vector sequence, after passing the word-level Bi-GRU, each word will have a corresponding Bi-GRU output hidden vector h, and then obtain the attention weight through the dot product of the uw vector and the h vector at each time step. Then the h sequence is made a weighted sum according to the attention weight to obtain the sentence summary vector s2. Each sentence is passed through the same Bi-GRU structure and attention is added to obtain the final output document feature vector v vector, and then the v vector passes the latter dense Add a classifier to the layer to get the final text classification result. The structure of the model is very consistent with the human comprehension process from word->sentence-> to text. The most important thing is that the model has a very good visualization effect while providing better classification accuracy. At the same time, in the process of tuning, we found that the attention part has a great influence on the expression ability of the model. The effect of adjusting L2-Loss in all positions of the entire model on the expression ability of the model is far less than that of the two attention places. Can explain why the visualization effect is better, because attention contributes a lot to the output of the model, and attention can be visualized. Let's take a look at his visualization effect on the crime prediction task in the legal field. The result of the following visualization is not to find a few good results, but in most cases the visualization of the model can explain his output. It should be noted that here, in order to make the relatively important words in the less important sentences not completely invisible, the brightness of the words = sqrt (sentence weight) * word weight. In the very long text, HAN felt that those in the begining were not as useful as the sentence "it was't bad but not sure what hype is about".</p>
{{< figure src="han2.png" title="HAN2" >}}

<p>It can be seen that not all deep learning models are incomprehensible, and this interpretability will also bring a lot of help to practical applications.</p>

### c.DPCNN
<p>So the question is, will the increase in network depth in the text classification field lead to a substantial increase in classification accuracy? We have improved in some more complex tasks and when the amount of data is relatively large (million level), but it is not the decisive improvement of ResNet. The main structure of DPCNN is shown in the figure below:</p>
{{< figure src="dpcnn.png" title="DPCNN Structure" >}}

<p>Start from the word vector (this article focuses on the large structure of the model, so I don’t go to the region embedding part of the article in detail, and directly consider the whole part as a word vector output.) First do it twice with a width of 3 and the number of filters 250 convolutions, and then start to do two adjacent max-pooling, assuming the input sentence length padding to 1024 words, then the sentence length is still 1024 after the first two convolutions are completed. In the pooling position of block 1, the width of max pooling=3, stride=2, that is, the feature map of each dimension in the three adjacent time steps in the sequence takes the largest one of the three positions and leaves it, which is the position Take a maximum value from 0, 1, and 2, then move 2 time steps, and take max once in time steps 2, 3, and 4, then the sequence length of pooling output is 511. In the following analogy, the length of the sequence decreases exponentially, which is the origin of the article's name, Deep Pyramid. Then through the nonlinear transformation of the two convolutions, deeper features are extracted, and then the quick connection path that has not been convolved twice is superimposed on the output place (the key to making the deep network easier to train in ResNet). Because the max pooling in each block is only two adjacent positions for max-pooling, so little structural information is lost each time, and the subsequent convolutional layer can extract more abstract features. Therefore, the final model can perform a deeper nonlinear transformation without losing too much structural information.</p>
    
 > In our actual test, DPCNN will have higher accuracy than HAN in classification tasks with high nonlinearity requirements, and because it is based on CNN, the training speed is much faster than GRU-based HAN.
