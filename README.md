# Locality Sensitive Hashing

## Motivation

At some point in every persons life they hear an awesome song for the first time and are curious to know what it's name is. I was in this situation more often than I'd like to admit. Not anymore!! [Shazam](https://www.shazam.com/) has been a great friend of mine since I first learnt about it in 2014. I was amazed by how easy identifying a new song has become. At the same time I wondered how they are able to achieve such speedy identification with a massive database of songs (>11 million songs) last I checked). Shazam uses some advanced music information retrieval techniques to achieve this but one can implement such music recognition for fun using Audio fingerprinting and Locality Sensitive Hashing.

Audio finger printing is the process of identifying unique characteristics from a fixed duration audio stream. Such unique characteristics can be identified for all existing songs and stored in a database. When we hear a new song, we can extract similar characteristics from the recorded audio and compare against the database to identify the song. However, in practice there will be two challenges with this approach:

1. High dimensionality of the unique characteristic/feature vector required to identify songs
2. Comparison of the recorded audio features against features of all songs in the database is expensive in terms of time and memory

The fist challenge can be addressed using a dimensionality reduction technique like PCA and the second using a combination of clustering and nearest neighbor search. Locality Sensitive Hashing (hereon referred to as LSH) can address both the challenges by

1. reducing the high dimensional features to smaller dimensions while preserving the differentiability
2. grouping similar objects (songs in this case) into buckets

## Applications of LSH

Before jumping into understanding LSH, it is worth noting that the application areas are varied, including:

- Recommendation systems
- Near-duplicate detection
- Hierarchical clustering
- Genome-wide association study
- Image similarity identification (VisualRank)
- Audio similarity identification
- Audio fingerprint
- Digital video fingerprinting

Uber used LSH to detect [platform abuses](https://eng.uber.com/lsh/) (fake accounts, payment fraud, etc.). A Shazam styled app or Youtube sized recommender system can be built using LSH.

## What is LSH?

LSH is a hashing based algorithm to identify approximate nearest neighbors. In the normal nearest neighbor problem, there are a bunch of points (let's refer to these as training set) in space and given a new point, objective is to identify the point in training set closest to the given point. Complexity of such process is linear [O(N), where N is the size of training set] . An approximate nearest neighboring algorithm tries to reduce this complexity to sub-linear (less than linear but can be anything). Sub-linear complexity is achieved by reducing the number of comparisons needed to find similar items.

LSH works on the principle that if there are two points in  feature space closer to each other, they are very likely to have same hash (reduced representation of data). LSH primarily differs from conventional hashing (aka cryptographic) in the sense that cryptographic hashing tries to avoid collisions but LSH aims to maximize collisions for similar points. In cryptographic hashing a minor perturbation to the input can alter the hash significantly but in LSH, slight distortions should be ignored so that the main content can be identified easily. The hash collisions make it possible for similar items to have a high probability of having the same hash value.

> Locality Sensitive Hashing (LSH) is a generic hashing technique that aims, as the name suggests, to preserve the local relations of the data while significantly reducing the dimensionality of the dataset.

Now that we have established LSH is a hashing function that aims to maximize collisions for similar items, let's formalize the definition:

> A hash function h is Locality Sensitive if
> for given two points a, b in a high dimensional feature space,
> 1. Pr(h(a) == h(b)) is high if a and b are near
> 2. Pr(h(a) == h(b)) is low if a and b are far
> 3. Time complexity to identify close objects is sub-linear

## Implementing LSH

Having learnt what LSH is, it's time to understand how to implement it. Implementing LSH is down to understanding how to generate hash values. Some popular approaches to construct LSH are

- [Min-wise independent permutations](https://medium.com/engineering-brainly/locality-sensitive-hashing-explained-304eb39291e4)
- [Nilsimsa Hash (Anti-Spam focused)](https://wikivisually.com/wiki/Nilsimsa_Hash)
- [TLSH](https://github.com/trendmicro/tlsh/blob/master/TLSH_CTC_final.pdf) (For security and digital forensic applications)
- Random Projection aka SimHash

In this article, I'll give a walkthrough of Implementing LSH using random projection method. Curious readers can learn about the other methods from the linked urls.

### Random Projection Method

Random projection is a technique for representing high-dimensional data in low-dimensional feature space (dimensionality reduction). It gained traction for its ability to approximately preserve relations (pairwise distance or cosine similarity) in low-dimensional space while being computationally less expensive.

> The core idea behind random projection is that if points in a vector space are of sufficiently high dimension, then they may be projected into a suitable lower-dimensional space in a way which approximately preserves the distances between the points.
 
Above statement is an interpretaion of the [Johnson-Lindenstrauss lemma](https://en.wikipedia.org/wiki/Random_projection).

Consider a high-dimensional data represented as a matrix $D$ with `n` observations (columns of matrix) and `d` features (rows of the matrix). It can be projected onto a lower dimensional space with `k` dimensions, where `k<<d`, using a random projection matrix $R$. Mathematically, the lower dimensional representation $P$ can be obtained as

<img src="https://latex.codecogs.com/gif.latex?\begin{bmatrix}&space;&&space;&&space;\\&space;&&space;Projected&space;(P)&space;&&space;\\&space;&&space;&&space;\end{bmatrix}_{k&space;\times&space;n}&space;=&space;\begin{bmatrix}&space;&&space;&&space;\\&space;&&space;Random&space;(R)&space;&&space;\\&space;&&space;&&space;\end{bmatrix}_{k&space;\times&space;d}&space;\begin{bmatrix}&space;&&space;&&space;\\&space;&&space;Original&space;(D)&space;&&space;\\&space;&&space;&&space;\end{bmatrix}_{d&space;\times&space;n}" title="matrix_lsh"/>

Columns of the random projection matrix $R$ are called random vectors and the elements of these random vectors are drawn independently from gaussian distribution (zero mean, unit variance).

### LSH using Random Projection Method


Hash Function - Random projections - For the input feature vector (q of dimension L) take inner products with random vectors (p, k in number) and take the sign of the inner product (is p aligned with q on one side of space?). What are random projections - Gaussian random vectors with independent elements (zero mean and unit variance). L and k are tuned to adjust the tradeoff between recall and precision.

Why Gaussian and Why Random - If two points are aligned completely, have perfect correlation from origin, this hash construction imply they'll be in same hash bin. Similarly, two points separated by $$180^{\circ}$$ will be in different bins and two points $$90^{\circ}$$ apart have 50% probability to be in same bins. k-bit hash requires k projections. We can also have several hash tables (multiple k-bit hashes) => we define two hashes to be same if all hashes matches for at least one table. Multiple tables generalizes the space better and amortizes the contribution of bad random vectors.
*Does gaussian mean orthogonal? Low probability they are correlated*


## Resources
- [Random Projection](https://nbviewer.jupyter.org/github/lindarliu/blog/blob/master/Random%20Projection%20and%20its%20application.ipynb)
- [LSH applied to Music Information Retrieval](https://www.youtube.com/watch?v=SghMq1xBJPI)
- [LSH applied to document similarity detection](http://joyceho.github.io/cs584_s16/slides/lsh-11.pdf)
- [Music Search using LSH](https://github.com/stevetjoa/musicsearch)
- [List of Audio File Databases](http://www.audiocontentanalysis.org/data-sets/)
- [Free Music Archive](https://github.com/mdeff/fma)
