---
layout: post
title:  "Reading A Paper on Faster Top-k Document Retrieval"
date:   2020-11-29 10:20:00 -0800
categories: indexing paper-reading
---

# Reading A Paper on Faster Top-k Document Retrieval

Recently a friend shared a paper with me on document retrieval: [Faster Top-k Document Retrieval Using Block-Max Indexes](http://engineering.nyu.edu/~suel/papers/bmw.pdf). I wasn’t able to follow it as I wasn’t familiar with the WAND algorithm. So I went back and read that original paper on [WAND](https://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.365.2939&rep=rep1&type=pdf). It was quite an interesting read.

WAND stands for Weighted And. It is an operator that  applies weights to each operand (either 1 or 0), sums them up, and then, evaluates to true or false depending on whether the sum is no less than a threshold. Essentially, WAND allows for representing OR or AND, or anywhere in between using a summation and a comparison to a threshold value at the end. The original explanation in the paper is as follows:

![WAND Operator Definition](/assets/wand-operator.jpg)

Having defined the WAND operator, the WAND paper sets out to find an efficient way to traverse through a set of documents, one document at a time, and return the top-k documents that are ranked via an additive scoring model. The key insight in the paper is that we can define a upper bound for the score of a term for any document in the set. Once we have the upper bounds for each term, we can skip any documents that we know for certain cannot make it into the set of top-k documents. The algorithm for going to the next document to promote to top-k is as follows:

![WAND Algorithm](/assets/wand-algorithm.jpg)

What makes the pivot document so special? Why can we skip ahead to that document for all terms? The observation is that once we have sorted all postings by their current document ID. The first term whose current document we have arrived upon as the pivot indicates that no documents before the pivot can possibly contain any other terms that could make it part of the top-k docs. The sorting the in the beginning by document ID has guaranteed this aspect.  Following are the two invariants maintained by the WAND operator, as explained in the paper:

> The WAND iterator maintains two invariants during its execution:
> 1. All documents with DID ≤ curDoc have already been considered as candidates.
> 2. For any term t, any document containing t, with DID < posting[t].DID, has already been considered as a candidate.

Now, having understood the original WAND algorithm, I re-read the newer paper on improving it. The observation in this paper is that the original WAND algorithm has limited skipping potential as the upper bound for the entire posting list for a term can be too high, which means we end up with way too many false positives. The innovation this paper proposes is that we can instead partition the posting list into blocks, and assign a *block max value* for each block. We also keep the global upper bound for each term. Then, we follow the original WAND algorithm and when we come across a potential candidate (pivot document), we examine its block max upper bound which we know is most likely lower than the global upper bound. We likely will find out this document’s local upper bound does not make the cut, and thus we’re able to skip ahead to the first document in the next block for this term. An example in the paper is as follows:

![BMW Algorithm](/assets/bmw-algorithm.jpg)

We landed upon a pivot document with the term monkey. However, after examining its block max, we realize it’s less than the threshold. Thus, instead of incrementing the pivot by just 1, we can skip straight to the first document in the next block (d3). We then re-sort, and repeat the original WAND algorithm again.

These two papers were a fun read for me. They weren’t too difficult to understand. They got me interested in the various indexing techniques in general, and inspired me to start reading [Introduction to Information Retrieval](https://nlp.stanford.edu/IR-book/information-retrieval-book.html).
