---
title: Machine Learning and Neuron Nerworks models - Why ONNX?
date: 2020-05-17 00:03
author: selcuk
image: 
  thumbnail: /assets/images/onnx.png
---

**ONNX**, short for *Open Neural Netwrok Exchange*, is an open standard for machine learning (**ML**) models. The question must be asked before we move forward, why yet another ML model standard? One can already name a handful of famous or infamous examples of it. ML model itself is so important in a way that almost all mainstream machine learning frameworks, incluidng TensorFlow, PyTorch, Keras and TVM etc., have their own defintion and implementation of it. Framworks nowadays even ship with some well-defined, pre-trained, state-of-the-art models so that users can just import and use them for tasks like data mining, pattern recognition and search optimization. If the stake is that high, why on earth did Microsoft and Facebook decided to join their hands and re-invent the wheels again? The answer is simple, if not too simple, interoperability.

## Interoperability

Interoperability means the ability of computer systems or software to exchange and make use of information. In the little echo room of ML frameworks, it means how different frameworks will be able to interpret ML models from other platforms. It's always been the Tower of Babel kind of thing (Hope I am using this analogy correctly). Before we climb this tower of babel, I feel it is of some importance to thoroughly develop the idea and structure of some typical ML models on how they become the basic unit of many modern-day machine learning frameworks.

## Machine Learning

Instead of taking explicit instructions or commands, machine learning is a scientific approach for computers to perform specific tasks based on algorithms and statistical models. Nevertheless, machine learning techniques were constrained by their capability to process real-world data in its raw form. Furthermore, traditional machine learning approaches necessitate a massive amount of engineering efforts with tailored algorithms and domain-specific expertise because their performance largely depends on the data representation or features, that are applied to them. Therefore, the choice of such data representation, namely feature engineering, is a dominant factor and bottleneck to the general performance of machine learning algorithms. 

## Representation Learning

Representation learning is proposed to avoid the intensive manual efforts and preliminary knowledge requirements by the automatic discovery of the data representation for detection or classification. Deep learning is a set of representation learning methods that have multiple layers, or levels, of representation and each layer of representation will transform the raw data/representation to a more abstract level and therefore more complicated functions or patterns can be learned. There are a couple of deep learning architectures that are designed to address the above issues, including:

1. Deep Neural Network (**DNN**)
2. Convolutional Neural Network (**CNN**) 
3. Recurrent Neural Network (**RNN**)

## Neural Network

A neural network is composed of many mathematical functions, acting like neurons or processors, to transform the input data or weights. Commonly a layer will collect the output of all neurons and this output will become the input for the next layer of neurons in the topology. A deep neural network differs from a ”shallow” neural network in the number of layers used in it. DNN is more capable of building machine learning models for a large amount of data by utilizing more complex and immense layers of neurons, especially large datasets with more abstract and hidden patterns.

## In a Nutshell

Modern ML frameworks are proposed to train and deploy neural network/ML models.
As mentioned above, the Tower of Babal lays on the fact that most ML frameworks have their *own definition and implementation* of ML models. Let's not deny the fact that there is some overlapping. Some common mathematical functions, well-defined neuron network operations shared here and there. However some seemlingly negligible difference on the implementation could result in substantial divergence on end results, especially for accurancy-driven tasks. Therefore, the lack of portability of neural network models makes each ML framework isolated from others. And in the next post I will talk about how **ONNX** could possibly change this dilemma.









