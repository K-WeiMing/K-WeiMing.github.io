---
layout: post
title: HDB Resale Price Data Analysis / Prediction
subtitle: Excerpt from Soulshaping by Jeff Brown
cover-img: /assets/img/path.jpg
thumbnail-img: /assets/img/thumb.png
share-img: /assets/img/path.jpg
tags: [Resale, HDB, Data Analytics]
---


Current References: https://towardsdatascience.com/bert-explained-state-of-the-art-language-model-for-nlp-f8b21a9b6270


# What is BERT?
BERT stands for Bidirectional Encoder Representations from Transformers (BERT). It was created by Google in 2018 and is currently one of the most popular Natural Language Processing (NLP) models out there. For the English Language, the models were trained using unlabelled data extracted from Books and Wikipedia (consisting of 2500M words).

# How does BERT work?
BERT utilises transformers to generate contextualized representations of words. The transformer consists of two parts, an Encoder which reads the text input and a Decoder that produces predictions for the input words. Since BERT is a pre-trained model, it only consists of the Encoder portion in the transformer.

## Masked Learning Model
BERT is trained by masking. What this means is that before passing in the sentences in for training, 15% of words in each sentence is replaced with a [MASK] token. The model then attempts to predict the masked token based on the surrounding context of the unmasked words

## Next Sentence Prediction
