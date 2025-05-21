# LM-LSS Notes 

## Chap 0

A sequence of characters is *document*.

A set of documents is a *corpus*, which we call $D$.

Sample of documents, each $n_L$ long, drawn from vocab of $n_V$ words, then the
dimension of the representation of the document would be $n_V^{n_L}$.

This indicates text is high-dimension data.

Tokenization and Dimension Reduction focuses on this problem: we throw away the
useless information. We transform unstructured corpus $D$ to a useful matrix $X$.

NLP can now handle local language well, so we can identify which pronoun indicates
what part of the sentence.

Dictionary-based method: we use regular expression to capture a pre-list of words
to analyse the corpus.

## Chap 1

LDA is short for Latent Dirichlet Allocation, which is an unsupervised model capturing
topics in a document. The input is several documents, each represented as word-bag, i.e.
TF-IDF matrix, and the output is each topic, represented as a prob. distribution on words.
Then we can calc the similarity between documents using cos-distance between topic representation,
like document $A$ contains 30% topic $a$, 70% topic $b$; document $B$ contains ... etc..

We can also use pretrained S-Bert to get sentence vector, then use clustering technique to get 
topics. The advantages are: in LDA, worker and employer are two independent words.. Also it gives 
a topic for each sentence. So LDA is more suitable for long documents.

Once we have a topic, we can sample some sentences from this topic, and ask ChatGPT to 
summarize a topic for these sentences.
