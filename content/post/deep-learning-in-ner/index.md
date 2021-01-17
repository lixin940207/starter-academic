---
title: Deep learning in NER

summary: A detailed explanation of the application of deep learning in named entity recognition (NER)


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
- ner

categories:
- nlp
- deep learning
---

<p>In recent years, deep learning methods based on neural networks have achieved great success in the fields of computer vision, speech recognition, etc., and also made a lot of progress in the field of natural language processing. In the research of the key basic task of NLP—Named Entity Recognition (NER), deep learning has also achieved good results. Recently, the author has read a series of related papers on NER research based on deep learning, and applied them to the NER basic module of DataGrand. Here I will summarize and share with you.</p>

### 1. NER introduction
<p>NER, also known as proper name recognition, is a basic task in natural language processing and has a wide range of applications. Named entities generally refer to entities with specific meaning or strong referentiality in the text, usually including names of persons, places, names of organizations, dates and times, proper nouns, etc. The NER system extracts the above entities from unstructured input text, and can identify more types of entities, such as product names, models, prices, etc., according to business needs. Therefore, the concept of entity can be very broad, as long as it is a special text fragment required by the business, it can be called an entity.</p>
{{< figure src="1.jpg" title="NER example" >}}
<p>Academically, the named entities involved in NER generally include 3 categories (entity category, time category, number category) and 7 subcategories (person's name, place name, organization name, time, date, currency, percentage).</p>

<p>In practical applications, the NER model usually only needs to identify the person's name, place name, organization name, date and time, and some systems will also provide proper noun results (such as abbreviations, conference names, product names, etc.). Numerical entities such as currency and percentages can be determined through regular rules. In addition, in some application scenarios, entities in a specific field will be given, such as book titles, song titles, and journal titles.

NER is a fundamental and key task in NLP. From the perspective of the natural language processing process, NER can be regarded as a type of unregistered word recognition in lexical analysis. It is the problem of the largest number of unregistered words, the greatest difficulty in recognition, and the greatest impact on word segmentation. At the same time, NER is also the basis for many NLP tasks such as relationship extraction, event extraction, knowledge graphs, machine translation, and question answering systems.

NER is not currently a hot research topic, because some scholars in academia believe that this is a solved problem. Of course, some scholars believe that this problem has not been solved well. The main reasons are: named entity recognition is only obtained in a limited text type (mainly in news corpus) and entity categories (mainly names of persons, places, and organizations). Compared with other information retrieval fields, entity naming evaluation is less expected and prone to over-fitting; named entity recognition focuses more on high recall rate, but in the field of information retrieval, high accuracy rate is more important; general recognition The system performance of many types of named entities is poor.</p>

### 2. Application of deep learning in NER
<p>NER has always been a research hotspot in the field of NLP. From early methods based on dictionaries and rules, to traditional machine learning methods, to methods based on deep learning in recent years, the general trend of NER research progress is roughly as shown in the figure below.</p>
{{< figure src="2.jpg" title="NER trend" >}}
<p>In machine learning-based methods, NER is treated as a sequence labeling problem. Use large-scale corpus to learn the labeling model, so as to label each position of the sentence. Commonly used models in NER tasks include generative model HMM, discriminative model CRF, etc. Conditional Random Field (CRF) is the current mainstream model of NER. Its objective function not only considers the input state characteristic function, but also includes the label transition characteristic function. SGD can be used to learn model parameters during training. When the model is known, finding the predicted output sequence for the input sequence is the optimal sequence that maximizes the objective function. This is a dynamic programming problem, which can be decoded by Viterbi algorithm to obtain the optimal label sequence. The advantage of CRF is that it can utilize rich internal and contextual feature information in the process of marking a location.</p>
{{< figure src="3.jpg" title="CRF" >}}
<p>In recent years, with the development of hardware computing capabilities and the introduction of word embedding, neural networks can effectively handle many NLP tasks. This type of method handles sequence labeling tasks (such as CWS, POS, NER) in a similar way: the token is mapped from the discrete one-hot representation to the low-dimensional space to become dense embedding, and then the embedding sequence of the sentence is input to the RNN In, the neural network is used to automatically extract features, and Softmax is used to predict the label of each token.

This method makes the training of the model an end-to-end process instead of a traditional pipeline. It does not rely on feature engineering and is a data-driven method. However, there are many types of networks, a large dependence on parameter settings, and poor model interpretability. In addition, a disadvantage of this method is that the process of labeling each token is carried out independently, and the previously predicted label cannot be directly used (only the hidden state can be used to transmit the above information), which leads to the predicted label The sequence may be invalid. For example, the label I-PER cannot be followed by B-PER, but Softmax will not use this information.

The academic circle proposed the DL-CRF model for sequence labeling. The output layer of the neural network is connected to the CRF layer (the focus is to use the label transition probability) to do sentence-level label prediction, so that the labeling process is no longer an independent classification of each token.</p>

#### 2.1 BiLSTM-CRF
<p>
The LongShort Term Memory network is generally called LSTM, which is a special type of RNN that can learn long-distance dependent information. LSTM was proposed by Hochreiter & Schmidhuber (1997) and was recently improved and promoted by Alex Graves. On many issues, LSTM has achieved considerable success and has been widely used. LSTM solves the long-distance dependence problem through clever design.

All RNNs have a chain form of repeated neural network units. In a standard RNN, this repeating unit has only a very simple structure, such as a tanh layer.</p>

{{< figure src="4.jpg" title="traditional RNN" >}}
<p>LSTM has the same structure, but the repeating unit has a different structure. Unlike ordinary RNN units, there are four here, interacting in a very special way.</p>
{{< figure src="5.jpg" title="LSTM Structure" >}}
<p>Through three gate structures (input gate, forget gate, output gate), LSTM selectively forgets part of the historical information, adds part of the current input information, and finally integrates into the current state and generates an output state.</p>
{{< figure src="6.jpg" title="CRF" >}}
<p>The biLSTM-CRF model applied to NER is mainly composed of Embedding layer (mainly word vector, word vector and some additional features), bidirectional LSTM layer, and finally CRF layer. Experimental results show that biLSTM-CRF has reached or surpassed the CRF model based on rich features, and has become the most mainstream model of NER based on deep learning. In terms of features, the model inherits the advantages of deep learning methods. It does not require feature engineering. It can achieve good results by using word vectors and character vectors. If there are high-quality dictionary features, it can be further improved.</p>
{{< figure src="7.jpg" title="CRF" >}}

#### 2.2 IDCNN-CRF

For sequence labeling, ordinary CNN has a shortcoming, that is, after convolution, the last layer of neurons may only get a small piece of information in the original input data. For NER, each word in the entire input sentence may have an impact on the current position labeling, which is the so-called long-distance dependency problem. In order to cover all the input information, more convolutional layers need to be added, resulting in deeper and deeper layers and more and more parameters. In order to prevent overfitting, more regularization such as Dropout has to be added, which brings more hyperparameters, and the entire model becomes large and difficult to train. Because of the disadvantages of CNN, people still choose a network structure such as biLSTM for most sequence labeling problems, and use the memory of the network to remember the information of the whole sentence as much as possible to label the current word.

But this brings another problem. BiLSTM is essentially a sequence model, which is not as powerful as CNN in the use of GPU parallel computing. How can we provide a full-fire battlefield for the GPU like CNN, and remember as much input information as possible with a simple structure like LSTM?

Fisher Yu and Vladlen Koltun 2015 proposed the dilated CNN model, which means "expanded" CNN. The idea is not complicated: the filters of normal CNN all act on a continuous area of ​​the input matrix, continuously sliding for convolution. The dilated CNN adds a dilation width to the filter. When it is applied to the input matrix, it will skip all the input data in the middle of the dilation width; and the size of the filter itself remains the same, so that the filter obtains the data on the wider input matrix. It looked like it was "expanded".

In specific use, the dilated width will increase exponentially as the number of layers increases. In this way, as the number of layers increases, the number of parameters increases linearly, while the receptive field increases exponentially, which can quickly cover all input data.
{{< figure src="8.jpg" title="IDCNN" >}}

Figure 8: Schematic diagram of idcnn

It can be seen in Figure 7 that the receptive field expands at an exponential rate. The original receptive field is a 1x1 area at the center point:

(A) In the figure, spread out through the original receptive field with a step length of 1, and 8 1x1 areas are obtained to form a new receptive field with a size of 3x3;

(B) After the diffusion step of 2 in the figure, the 3x3 receptive field of the previous step is expanded to 7x7;

(C) In the figure, after a step of 4 diffusion, the original 7x7 receptive field is expanded to 15x15. The number of parameters in each layer is independent of each other. The receptive field expands exponentially, but the number of parameters increases linearly.

Corresponding to the text, the input is a one-dimensional vector, and each element is a character embedding:

{{< figure src="9.jpg" title="idcnn with expanding step equal to 4" >}}

IDCNN generates a logits for each word of the input sentence, which is exactly the same as the output logits of the biLSTM model. The CRF layer is added and the annotation result is decoded using the Viterbi algorithm.

Connecting the CRF layer to the end of network models such as biLSTM or IDCNN is a very common method of sequence labeling. The biLSTM or IDCNN calculates the tag probability of each word, and the CRF layer introduces the transition probability of the sequence, and finally calculates the loss and feeds it back to the network.

### 3. application
#### 3.1 corpus
<p>Embedding: We choose Chinese Wikipedia corpus to train word vectors and word vectors.

Basic corpus: Select the 1998 corpus marked by People’s Daily as the basic training corpus.

Additional corpus: 98 corpus as the official corpus, its authority and correctness of labeling are guaranteed. However, because it is completely taken from the People's Daily and has a long history, the coverage of entity types is relatively low. Such as new company name, foreign name, foreign place name. In order to improve the ability to recognize new types of entities, we have collected a batch of labeled news corpus. It mainly includes finance, entertainment, and sports, which are relatively lacking in 98 corpus. Due to the quality of annotation, the extra corpus cannot be added too much, about 1/4 of the 98 corpus.</p>

#### 3.2 data augmentation
For deep learning methods, a large amount of annotated corpus is generally required, otherwise it is prone to overfitting and cannot achieve the expected generalization ability. We found in experiments that data enhancement can significantly improve model performance. Specifically, we segment the original corpus, and then randomly concatenate each sentence with bigram and trigram, and finally use the original sentence as the training corpus.

In addition, we use the collected named entity dictionary to replace the entities of the same type in the corpus with random replacement to obtain an enhanced corpus.

The following figure shows the training curve of the BiLSTM-CRF model. It can be seen that the convergence is very slow. In contrast, the convergence of the IDCNN-CRF model is much faster.
{{< figure src="10.jpg" title="BiLSTM training loss" >}}
{{< figure src="11.jpg" title="IDCNN training loss" >}}

#### 3.3 test

The following is an example prediction result using the BiLSTM-CRF model.
{{< figure src="12.jpg" title="BiLSTM-CRF prediction" >}}

### 4. summary
Finally, a summary, CNN/RNN-CRF, which combines neural networks and CRF models, has become the current mainstream model of NER. For CNN and RNN, no one has an absolute advantage, each has its own advantages. Because RNN has a natural sequence structure, RNN-CRF is more widely used. The NER method based on the neural network structure inherits the advantages of the deep learning method without a lot of artificial features. Only word vectors and word vectors can reach the mainstream level, and adding high-quality dictionary features can further enhance the effect. For the problem of a small number of labeled training sets, transfer learning and semi-supervised learning should be the focus of future research.
