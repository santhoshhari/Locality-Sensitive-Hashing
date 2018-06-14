# Locality Sensitive Hashing

## Motivation

At somepoint in every persons life they hear an awesome song for the first time and are curious to know what it's name is. I was in this situation more often than I'd like to admit. Not anymore!! [Shazam](https://www.shazam.com/) has been a great friend of mine since I first learnt about it in 2014. I was amazed by how easy identifying a new song has become. At the same time I wondered how they are able to achieve such speedy identification with a massive database of songs (>11 million songs) last I checked). From some google searches, I've learned that this is achieved through a combination of audio finger printing and hashing. Audio finger printing is the process of identifying unique characteristics of a fixed duration audio stream. Such unique characteristics identified for the recorded song are compared against the database of existing songs and the closest match is returned. In practice unique feature required to identify songs is very high dimensional and can be expensive in terms of space and time requirements. Hashing techniques address these concerns by reducing these high dimensional features to smaller dimensions while preserving the differentiability. Locality Sensitive Hashing (LSH) is one such hashing technique commonly used when identification of closest match (nearest neighbor, formally) is the end goal. This article is my attempt at demisifying LSH. 

## What is LSH?

LSH is a hashing based algorithm to identify approximate nearest neighbors. In the normal nearest neighbor problem, there a bunch of points (let's refer to these as training set) in space and given a novel point, objective is to identify the point in training set closest to the given point. Complexity of such process is linear [O(N), where N is the size of training set] . An approximate nearest neighboring algorithm tries to reduce this complexity to sub-linear (less than linear but can be anything). Sub-linear complexity is achieved by reducing the number of comparisons needed to find similar items.

> Locality Sensitive Hashing (LSH) is a generic hashing technique that aims, as the name suggests, to preserve the local relations of the data while significantly reducing the dimensionality of the dataset.

LSH works on the principle that if there are two points in  feature space closer to each other, they are very likely to have same hash (reduced representation of data). LSH primarily differes from conventional hashing (aka cryptographic) in the sense that cryptographic hashing tries to avoid collisions but LSH aims to maximize collisions for similar points. In cryptographic hashing a minor perturbation to the input can alter the hash significantly but in LSH, slight distortions should be ignored so that the main content can be identified easily. The hash collisions make it possible for similar items to have a high probability of having the same hash value.

Now that we have established LSH is a hashing function that aims to maximize collisions for similar items, let's formalize the defintion:

> A hash function h is Locality Sensitive if
> for given two points a, b in a high dimensional feature space,
> 1. Pr(h(a) == h(b)) is high if a and b are near
> 2. Pr(h(a) == h(b)) is low if a and b are far
> 3. Time complexity to identify close objects is sublinear

**General Process flow:**

Feature Vector -> Hash Function -> Bit String -> Lookup hash table and retreive bin corresponding to the bit string -> Aggregate information within a bin -> Return the closest items

## Implementation Details

Having understood what LSH does, it's time to understand how to implement LSH. Implementing an LSH is down to understanding how to generate hash values. Some popular approaches to construct LSH are

- Bit sampling for Hamming distance
- [Min-wise independent permutations](https://medium.com/engineering-brainly/locality-sensitive-hashing-explained-304eb39291e4)
- [Nilsimsa Hash (Anti-Spam focused)](https://wikivisually.com/wiki/Nilsimsa_Hash)
- TLSH (For security and digital forensic applications)
- Random Projections aka SimHash

I'll explain LSH using random projections.

### Random Projections Method

Hash Function - Random projections - For the input feature vector (q of dimension L) take inner products with random vecotrs (p, k in number) and take the sign of the inner product (is p aligned with q on one side of space?). What are random projections - Gaussian random vectors with independent elements (zero mean and unit variance). L and k are tuned to adjust the tradeoff between recall and precision.

Why Gaussian and Why Random - If two points are aligned completely, have perfect corelation from origin, this hash construction imply they'll be in same hash bin. SImilaryly, two points seperated by 180% will be in different bins and two points 90^0 apart have 50% probability to be in same bins. k-bit hash requires k projections. We can also have several hash tables (multiple k-bit hashes) => we define two hashes to be same if all hashes matches for atleast one table. Multiple tables generalizes the space better and amortizes the contribution of bad random vectors.
*Does gaussian mean orthogonal? Low probability they are correlated*


## Applicaitons

LSH has been applied to several problem domains including:

- Near-duplicate detection
- Hierarchical clustering
- Genome-wide association study
- Image similarity identification
    - VisualRank
- Audio similarity identification
- Nearest neighbor search
- Audio fingerprint
- Digital video fingerprinting

Uber used LSH to detect [platform abuses](https://eng.uber.com/lsh/) (fake accounts, payment fraud, etc.). A shazam styled app or Youtube sized recommender system can be built using LSH.

## Resources
- [LSH Applied to Music Information Retreival](https://www.youtube.com/watch?v=SghMq1xBJPI)
- [LSH Explained](http://joyceho.github.io/cs584_s16/slides/lsh-11.pdf)
- [Music Search using LSH](https://github.com/stevetjoa/musicsearch)
- [List of Audio File Databases](http://www.audiocontentanalysis.org/data-sets/)
- [Clean Database](https://github.com/mdeff/fma)
