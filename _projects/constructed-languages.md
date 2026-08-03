---
layout: post
title: "Generative Constructed Languages"
description: "Generative pipeline for creating a fully translatable constructed language from natural source languages."
date: 2026-07-20
categories: software
title-image: "/assets/images/ConLangMap.png"
featured: true
---
{% include mathjax.html %}

This project and description are currently in progress! More to come.

# Why am I creating this generator?

I enjoy writing stories. I can sink hours and days into trying to create an immersive world. A lot of times, I'll create the world for others to take an active part in, like when I write for games like Dungeons and Dragons. Immersion depends on a lot of things, but I feel that having a developed history for whatever setting that your dropping your readers/players into gives the feeling that there is more to the world than what you are discovering directly on the page. Language is an incredibly effective route to build a history of a place. Embedded within language are cultural cornerstones and values. Nuances in translation of one language to the next can reveal nuances in the differences between the socieities that speak them. 

But languages are typically a very difficult thing to design; many authors of constructed languages like Tolkien for Sindarin or Quenya or Mark Okrand for Klingon had extensive backgrounds in linguistics and took a very incremental approach to creating their languages focusing on taking linguistic building blocks and putting them together. While their methods have resulted in rich languages with large communities of adopted speakers, their approach is largely out of reach for your typical dungeon master who is trying to make a homebrew campaign.

This project is focused on bridging that gap. I hope to create a language generation pipeline that allows for directed input from a user to create entirely new and unique languages that can be translatable to and from English. 

# The Idea

When constructed languages are created, a lot of effort goes into designing a language that has roots in natural language, whether that be through sound, meaning, syntax, or any other linguistic property. I created this language creation pipeline to draw upon existing analysis of natural languages when training models to produce constructed ones. Doing so allows for quick iterations of different types of languages with unique phonology and semantics that draw upon that of natural languages. Do you want a language that *sounds* like a mixture of Arabic and Spanish, but with the *semantics* of Flemish? The goal of this project is to allow a user to select those choices and then end up with a language that both sounds and reads naturally. 

The generation pipeline that I have architected goes through the following broad steps:

1. generation of learned semantic word embeddings without attached words,
2. generation of words based off learned phoneme strings, and
3. assignment of words to embeddings based off shared phoneme/semantic patterns. 

When creating a constructed language, the primary goal is to ensure that the language is translatable with natural language. While this could be accomplished by creating a "code" that replaces english words with made-up words so that every translation is one-to-one, this is neither satisfying nor a valid representation of natural language translation. Therefore translation relies on being able to identify similarities in meaning between the natural and constructed language as well as transformations of grammar. My approach to translation is therefore accomplished in the following way: 

1. calculation of matching target words to a source word via the orthogonal residual method,
2. selection of target words based on the Earth Mover's Score algorithm, and
3. ordering of words by matching a structural probe for syntactic tree depth for the source to the target.

# Learning and Generating Word Embeddings

Word embeddings are a representation of words as a vector. While there are many approaches to defining and producing these embeddings, the commonality is that word embeddings are a representation of how words appear in relation to one another. In a system like fasttext, each time a word i appears next to another word j in a large corpus of literature, the cells (i, j) and (j, i) increase by one. After training, the matrix is reduced via SVD to 300 dimensions so that each word i is represented as a vector with 300 components. In models like BERT or MBERT, the model is trained to identify a masked word in a sentence using bidirectional attention on the rest of the phrase. Doing so creates highly contextual embeddings for each word, allowing the model to distinguish between various senses of polysemic words. 

The power of word embeddings lies in the universality of the high-dimensional space that they exist in. For example, two distinct languages like English and Spanish will share an extremely similar topology of the point-clouds of their entire vocabulary. This means that if these point clouds are aligned (which they automatically are when using an MBERT embedding) words with similar meaning will be very close to one another. This is an incredible boon when developing a constructed language, because if you want to develop a word that has a certain meaning or melds meanings together, you only need to know what direction in the high-dimensional space you need that word embedding vector to point in. 

Instead of doing this process by developing words and assigning them meaning one-by-one, I instead wrote and trained a Variational Auto Encoder (VAE) that learned a compression of the word embeddings from languages I selected into a lower-dimensional latent space. This way random noise can be fed into the VAE and it will return embeddings that fall along areas in the word embedding space that are well established in the languages I selected. Additionally a conditional value is attached to the data that tells the model the ranking of frequency of the word. For most literature, a vocabulary size of about 30k (not including multiple meanings associated with polysemic words) is decent enough to allow full translation. 

![VAE Verification](/assets/images/VAEVerification.png)

You can see in the plot of the word embeddings along two UMAP directions (reduced dimensions from the original 768 optimized to separate data points in space) how the word embeddings form into strands and clusters that the VAE can learn. For this small subset, you can see the English words dealing with travel and roads. The constructed language embeddings in red then lie in the space between these meanings. For example, the word "pɾofaɡada" might mean something that has to deal with a change in a road into something else. 

For training data, I took a selection of Wikipedia articles and excerpts from books in each language that I trained upon. As a test case, I used English, Italian, and Finnish language corpora. 

# Learning Phonemic Patterns

An IPA chart describes phonemes that are internationally standardized for a one-to-one mapping of a symbol to a producable sound in human speech. In the IPA chart[^ladefoge] you can see how different phonemes are mapped to different ways to generate sounds. For pulmonic consonants: the columns represent the placement of the consonant (from the front of the vocal tract to the back) and the rows represent the method of generation. Where pairs appear, the left consonant is the un-voiced version and the right side is the voiced version. The same is generally true for vowels: the columns describe placement of the vowel from front to back and the rows describe the method of generation from closed to open. 

![IPA Chart](/assets/images/ipaChart.png)

In order to generate words for a constructed language that approximate the sounds of source languages, I buit a deep-learning model using LSTM units and attention that was tasked with predicting masked phonemes in a string of phonemes that corresponded to a real word. For example, the word "phoneme" in English has a (broad) phonetic transcription as "foʊnim". When trained on a word like this, the model is given the task to take a word like ['f', 'MASK', 'ʊ', 'n', 'i', 'm'] and correctly predict that the MASK token corresponds to the phoneme 'o'. 

For this task, I used a large dataset of phoneme transcriptions of words for each source language using a crawl of Wikipedia data. A one-hot vector representation of each phoneme was generated where phonemes were ordered in terms of their vocal placement and manner (in accordance with the IPA chart) in order to relieve training stress on the model. This is a result of the 'embedding' that the model creates for each individual phoneme, much like the mBERT model does for words (in fact these models are based on the same principle, except this one has a far lesser number of parameters). If one-hot vectors are already ordered in terms of phonetic placement, the model is pre-seeded with the information that similar phonemes are slightly interchangable in similar contexts. For three source languages top-5 validation accuracy landed at about the 90% mark. While this is not perfect, it produces words that resemble the source languages remarkably well. Additionally, look at the resulting 2-d representation of the phoneme embeddings relative to one another using UMAP as a flattening device. Phonemes tend to cluster as one would expect given the IPA Chart shown above. Voiced and un-voiced versions of consonants are generally very near to one another (shown with black circles) and vowels (red) and consonants (green) with similar placement are placed closely together. 

![Phoneme Embeddings](/assets/images/PhonemeEmbeddingsUMAP.png)

After training, words were generated via inference of the model. The generation process involved taking a full sequence of MASK tokens and iteratively probing the model to fill in the MASK tokens until a word appeared. New words were constantly created while throwing out words that had duplicates. Word length was monitored and resulted in a nearly normal distribution centered around 8 or 9 characters and biased towards shorter words. For example, for a model trained on a wide array of languages including German, French, Serbo-Croation, and Bengali among others, some words that were created were: 'salandaɾeada', 'boɾona', 'paːspanan', and 'saraːta'.

# Assigning Meaning to Generated Words

With generated words in hand, I wanted to assign words to meaning in a controlled manner so as to mirror the tendency of natural language to convey changes of meaning with changes in phonology. For example, an expression of plurality in English is typically associated with the addition of an -s or -z sound to the end of a word. A commonly observed phenomenon in word embedding spaces is that changes in meaning are often strongly represented by directional changes in the high-dimensional space that describes those word embeddings. For example, the difference between the vectors representing "king" and "queen" and the difference between the vectors for "man" and "woman" will align very strongly with one another. This process could be perfectly mirrored for the phoneme strings since the model architecture was similar between the phoneme model and mBERT. 

The generated direction vectors were reduced via UMAP from 768 dimensions to 50 to reduce computational time. For each space, an unsupervised variational-density clustering scheme was run to define different common directions in the embedding space. This method of clustering allows for clusters of varying density, but also separates clusters from what it deems as noise. In a word embedding space, direction is associated with changes in meaning and they often encode a type of change that is common across a language like plurality or gender. Similarly to the word embedding space, common directions between embedding in the phonemic space might uncover consistent morphophonemic changes. Points are then initially connected to each other using graph matching where the embedding acts as a node and its cluster participation acts as an edge. This is necessary because often there will be a root that can be modified in many common ways, with some of those ways stacking upon one another. This graph matching allows clusters from the word embedding space to be assigned to clusters from the phonemic embedding space. Because it is unlikely that clusters have the same amount of points between spaces, matching starts from the center of the cluster and goes outward. After this matching process, loose words are then assigned to embeddings using a Zipf ordering. This uses the observed fact that more frequently used words in languages tend to be shorter. 

# Translation

A constructed language is fairly useless if it is untranslatable other than acting as decoration in the setting you are designing. Equipped with word-embeddings, translation becomes a task of both matching words in a source language to words in a target language and arranging those words in the correct grammatical order for that language. 

# Retrieving the Correct Embedding for a Word

This translation method relies on retrieving the correct embedding for a word. Because words have different senses, it is important to retrieve the sense of the word that is most appropriate for the given context. While this process is trivial for English, where you can just input your phrase as a string into the mBERT model and retrieve embeddings, it is impossible for the constructed language because the mBERT model has not been trained on that data. Therefore, I use the following method instead. In the get_embeddings() method of my translation class, if a word exists in the saved vocabulary of the class for a given source, all of its relevant senses are retrieved. If the word does not exist in the saved vocabulary (which is only allowable when the source is not the constructed language), it is generated via mBERT inference. Often these words will have been separated into WordPieces by the BERT Tokenizer with each piece having its own embedding (formally will be split into "form" and "##ally"). These embeddings are averaged to represent the entire word. 

In order to determine which words fit with each other, I use a method of relieving "tension" on the mBERT model given different choices of word embeddings. Because word embeddings are generated by retrieving the output of a hidden layer of the mBERT model in the first place, different combinations of the candidate embeddings are injected into the mBERT model mid-inference stream to determine which combination results in the least change when these values are projected into the next layer of the model. Inference requires an input to act upon, so this process is seeded with a completely masked phrase of the same length as the input phrase. Then the initial choice of best candidates is determined by which vectors initially match the projected vectors the best. This process continues iteratively until improvement stalls. This method then returns the context-optimal embeddings for each word. 

# Matching Words via Orthogonal Residuals

With the embeddings for each source word in-hand, an orthogonal residual search then selects a bag-of-words to match the source word. The orthogonal residual algorithm functions by first selecting the target word embedding that has the closest similarity to the source. It then subtracts a projection of the target word onto the source word to retrieve the orthogonal component of the target word to the source word. This orthogonal component is now set to the vector that the algorithm needs to find the most similar match to in the target language set. This process continues until the target word found until the residual vector accounts for 99% of the length of the original source (unit) vector. This value is kept high in order to encourage a large bag-of-words during the orthogonal residual search. 

Importantly, during this search, the similarity metric is calculated via a method called Cross-domain Similarity Local Scaling (CSLS) that measures how similar two items are while correcting for an issue called "hubness". Hubness is when a single item looks similar to everything else in the space. This is a problem that plagues word embeddings in particular because very common words like "the" or "a" are extremely central to many different concepts and have vectors that reflect this. CSLS is necessary to ensure that the orthogonal residual search does not just return these hub words alone. 

The orthogonal residual search relies on the assumption that the direction of the sum of two word embeddings preserves the meaning of those words. Therefore, if I were to translate a word from Spanish like "quiero" to English, the orthogonal residual search should return both "I" and "want". This is a key component in allowing for linguistic expansion from a source phrase to a target phrase. 

![Orthogonal Residual Search Visualization](/assets/images/OrthogonalResidualVisual.png)

# Matching Words via the Earth-Mover's Algorithm

The orthogonal residual search then passes every word that it matched to the source word list to the Earth Mover's algorithm. The Earth Mover's algorithm functions by assigning each source word a "mass" that must be distributed to each target word according to a cost matrix such that the total cost of mass movement is minimized. The cost matrix is defined by the cosine similarity between words so that a source word is more inclined to move its mass to a word that it is most similar to. If a source word shares similarities among multiple words, it will split its mass between those words. Similarly, if two source words both combine to send their mass to a single word, that might be the only word chosen for those two words (e.g. "I" and "want" to "quiero"). In the end, words are only accepted as valid translations if they receive an amount of the source word's mass greater than a threshold value. 

### Ordering Words 

Word ordering only happens in one direction: from English to the ConLang. Before ordering, each English word is categorized via the 'stanza' package. This package determines the part of speech for a given word in a sentence. By tracking which ConLang words come from which English words via the Earth Mover's method, the ordering method infers that these ConLang 'inheritors' must correspond to the same part of speech as the English words. Then using Dryer's [^dryer] Branching Dependent Theory for grammar that assumes that ordering of non-phrasal (NP) and phrasal (P) parts of speech is consistent for a grammatical system, the ordering method references a table of which parts of speech are non-phrasal or phrasal. Then based on the defined ordering of Object-Verb or Verb-Object for the language (and any language-specific exceptions that are built into the table), it begins to build the elements into a syntactic tree which is then flattened into a single sentence.

Translating from the ConLang to English is left to future work. Because the ConLang is generated, there is no method to determine parts of speech of the ConLang words without any English reference. Training can be done to infer labels for these words, but training models on grammatical structure is incredibly noisy and leads to very unstable inferred grammatical systems. 

# Some Next Steps!

Equipped with a language generation pipeline, I can now turn to using the generated language to start creating names and texts to use in a fictional world. To start building a history of the world, it will be helpful to have a framework that helps control language evolution. Most of my focus will now be on building this framework on top of the foundation I have already built. Having dials that allow a user to control things like phonemic and syntactic shift of a language will allow one to create rich worlds with rich histories. 

# Citations

[^ladefoge]: Ladefoged, P. (n.d.). A course in phonetics: Chapter 1. UCLA Phonetics Lab. Retrieved July 21, 2026, from https://www.phonetics.ucla.edu/course/chapter1/chapter1.html

[^dryer]: Dryer, Matthew S. "The Greenbergian word order correlations." Language 68.1 (1992): 81-138.