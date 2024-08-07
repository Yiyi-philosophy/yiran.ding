# Summarize for some papers.

## MLsys

### Monarch: Expressive Structured Matrices for Efficient and Accurate Training

#### 1 Abstract

- Problem: To reduce their compute/memory requirements is to replace **dense weight matrices** with structured ones (e.g., sparse, low-rank, Fourier transform)

- Challenge

  - In end-to-end training due to unfavorable **efficiency-quality tradeoffs**

  - In dense-to-sparse fine-tuning due to lack of **tractable algorithms** to approximate a given dense weight matrix

- **Monarch**: **hardware-efficient** (two block-diagonal matrices for better hardware utilization) + **expressive** (represent many commonly used transforms).
  - New ways to train and fine-tune sparse and dense models
  
- Result

  - End-to-End: ViT, MLP-Mixer, and GPT-2  #**2x** 

  - PDE solving and MRI reconstruction tasks: **error down #40%**

  - Sparse-to-Dense: GPT-2 #**2x**; BERT pretraining #**23%>** Nvidia MLPerf

  - Dense-to-Sparse: BERT finetuning #**1.7x**

#### 1 Introduction

- Challenge:

  - E2E: unfavorable efficiency-quality tradeoffs

    - **Model quality**: expressive ability of encoding domain-specific knowledge

    <img src="Summarize.assets/image (8).png" alt="image (8)" width="60%" />

  - **Sparse training method**: slow down training + ✖ represent commonly used transforms

  - **Structured matrice**: difficult to use in E2E training

- -> **Monarch** (bridge between dense and sparse)

  - E2E: Parameterized as products of two block-diagonal matrices (up to permutation)

    - Optimized batch-matrix-multiply (BMM) routines on GPUs

  - S2D: reverse sparsification

    - Sparse training ✖

    - Fast intermediate representation to speed up the training process of the dense model.

  - D2S: fine-tuning

    - **Projection** problem: finding a matrix in a class of structured matrices that is the closest to a given dense matrix

    - Author found a optimal projection algorithm for Monarch parameterization  (***Theorem 1:*** Given an $n×n$ matrix $A$, there is an $O(n^{5/2})$-time algorithm that optimally $A= LR$.)

#### 2 Related Work

- Butterfly matrices

  - [https://dawn.cs.stanford.edu/2019/06/13/butterfly/](https://dawn.cs.stanford.edu/2019/06/13/butterfly/)

  - Encode the divide-and-conquer structure of many fast multiplication algorithms

#### 3 Monarch: Definition & Algorithms

- 3.1 Monarch Parametrization for Square Matrices

  - Definition 3.1. Let $n = m^2$. An $n × n$ Monarch matrix has the form:
    - $\mathbf{M}=\mathbf{P L P}^{\top} \mathbf{R}$


  <img src="Summarize.assets/image (11).png" alt="image (11)" width="50%" />

  - Monarch matrices $M$ = two block-diagonal matrices multiplication

  - Products of Monarch Matrices Class: $\mathcal{M} \mathcal{M}^*$ 

  - $\left(\mathcal{M} \mathcal{M}^{\ast}\right)^2$  for $M_1,M_2 \in \mathcal{M} \mathcal{M}^\ast$ 

  <img src="Summarize.assets/image (9).png" alt="image (9)" width="50%" />

- 3.2 Expressiveness and Efficiency

  - ***Proposition 3.2:*** The matrix class $\mathcal{M} \mathcal{M}^\ast$ can represent convolution, Hadamard transform, Toeplitz matrices, and AFDF matrices. The matrix class $(\mathcal{M} \mathcal{M}^\ast)^2$ can represent the Fourier transform, discrete sine and cosine transforms (DST/DCT), the $(HD)^3$ class, Fastfood, and ACDC matrices.

  - ***Parameters:*** Monarch matrix $\mathbf{M}=\mathbf{P L P}^{\top} \mathbf{R} \sim 2n\sqrt{n}$  parameters. Although total FLOPs is $O(n\sqrt{n}) > O(n\log n)$ (butterfly matrix), Monarch is easy to implement, **#2x** faster than dense multiply

- 3.3 Projection on the Set M of Monarch Matrices

  - Dense Matrix -> Monarch

  - Given matrix $A$, find $\underset{\mathbf{M} \in \mathcal{M}}{\operatorname{argmin}}\|\mathbf{A}-\mathbf{M}\|_F^2$

  - ***Theorem 1:*** $A=LR \sim O(n^{5/2})$

  <img src="Summarize.assets/image (10).png" alt="image (10)" width="50%" />

  - To convert a pretrained model into a model with Monarch weight matrices and speed up downstream fine-tuning.

- 3.4 Factorization of MM∗ Matrices

  - Under mild assumptions, factorization to store and apply M efficiently

#### 5 Experiments

- 5.2 Sparse-to-Dense Training (reverse sparsification)
  - train a GPT-2 model with Monarch weight matrices for 90% of the training iterations, then relax the constraint on the weight matrices and train them as **dense matrices** for the remaining 10% of the iterations.
  
- 5.3 Dense-to-Sparse Fine-tuning


#### 6 Conclusion

- By making **structured matrices practical**, our work is a first step towards unlocking tremendous performance improvements in applying sparse models to **wide-ranging ML applications** (including science and medicine)

- Inspire more future work on advancing machine learning models for interdisciplinary research with limited computational resources

----

### SLIDE : IN DEFENSE OF SMART ALGORITHMS OVER HARDWARE ACCELERATION FOR LARGE-SCALE DEEP LEARNING SYSTEMS

#### Abstract

- It's hard to maintain **enough capacity to memorize many** parameters **and** obtain state-of-the-art accuracy **when training** large neural networks.

- **Specialized hardware** for model training **is expensive** and hard to generalize**, compared to GPUs.**

- This paper propose SLIDE (Sub-Linear Deep learning Engine) that uniquely blends **smart randomized algorithms**, with **multi-core parallelism and workload optimization**. 

- Performance

  - 44 core CPU **=** **3.5*1** Tesla V100

  - **1** SLIDE **= 10x TF**

#### 1 Introduction

- the idea of **adaptive sparsity** or **adaptive dropouts**

- design a system that can effectively leverage the **computational advantage** and at the same time compensate for the **hash table overheads using limited (only a few cores) parallelisms**. 

- Our Contributions

  - This unique possibility is because the **parallelism** in SLIDE is **naturally asynchronous** by design. We have our code and benchmark scripts for reproducibility.

  - in designing the LSH based sparsification to minimize the computational overheads to a few **memory lookups** only (**truly O(1)**)

    - The implementation further takes advantage of the sparse gradient updates to achieve negligible update conflicts, which creates ideal settings for Asynchronous SGD

  - SLIDE is a memory-bound application

    - With careful workload and cache optimizations (eg. Transparent Hugepages) and a data access pattern (eg. SIMD instructions), we further speed up SLIDE by roughly 1.3x

#### 2 LOCALITY SENSITIVE HASHING

<img src="Summarize.assets/image (3).png" alt="image (3)" width="100%" />

  - **Initialization:**

  - **Sparse Feed-Forward Pass with Hash Table Sampling:**

    

<img src="Summarize.assets/image (4).png" alt="image (4)" width="50%" />

  - **Sparse Backpropagation or Gradient Update:**

    - **Here we use the classical backpropagation message passing type implementation rather than vector multiplication based.**

    - **take full advan-tage of sparsity.**

  - **Update Hash Tables after Weight Updates:**

    - Updating neurons typically involves deletion from the old bucket followed by an addition to the new bucket, which can be expensive.

  - **OpenMP Parallelization across a Batch:**

    - Each data instance in the batch runs in a separate thread and its gradients are computed in parallel.

  - **The extreme sparsity and randomness in gradient updates allow us to asynchronously parallelize the accumulation step of the gradient across different training data without leading to a considerable amount of overlapping updates.**

#### 4 REDUCING OVERHEAD

- 4.2 Updating Overhead

  - Dynamically change the update frequency of hash tables to reduce the overhead.

    -  $N_0 e^{\lambda i}$

  - The gradient updates in the initial stage of the training are larger than those in the later stage

  - Reservoir sampling algorithm

#### 6 CONCLUSION AND FUTURE WORK

- Our system SLIDE is a combination of carefully tailored randomized hashing algorithms with the right data structures that allow asynchronous parallelism.

- Our next steps are to extend SLIDE to include convolutional layers. 

- SLIDE has unique benefits when it comes to **random memory accesses** and **parallelism**. We anticipate that a **distributed implementation** of SLIDE would be very appealing because the communication costs are minimal due to sparse gradients.



---

### QUADRALIB: A PERFORMANT QUADRATIC NEURAL NETWORK LIBRARY FOR ARCHITECTURE OPTIMIZATION AND DESIGN EXPLORATION

#### Abstract

- DNNs' success depends on many supporting libraries.

- QDNNs ($(WX)^2+b$) show better **non-linearity** and **learning capability**

- In this paper, author proposed a new QDNN neuron architecture design, and further developed QuadraLib. 

- **good accuracy** and computation consumption

#### 1 Introduction

- the benefits of QDNNs stem from the unique characteristics of the second-order polynomial form: 

  - (1) **stronger non-linearity**, hence improved capability for feature extraction

  - (2) **higher model efficiency** as QDNN can approximate polynomial decision boundaries using smaller network depth/width

- Contribution

  - four types of QDNN

  - new quadratic neuron architecture

  - QuadraLib

  - experiment

#### 2 DRAWBACKS OF THE EXISTING QDNN NEURON ARCHITECTURE DESIGN

- P1 Approximation Capability Issue:

- P2 Computation Complexity Issue:

- P3 Converge Performance Issue:

  - second-order term in QDNNs will introduce critical gradient vanishing issue

- P4 Implementation Feasibility Issue:

  - need to rewrite the entire convolution operation to add extra multiplication between W and the second X.

- P5 Structure Design Issue:

- P6 Memory Usage Issue:

- 3 QDNN NEURON ARCHITECTURE OPTIMIZATION

  - 3.1 New Neuron Architecture Design

    <img src="Summarize.assets/image (5).png" alt="image (5)" width="50%" />

    - introduce extra trainable parameters on the second-order term

    - other term to prevent the gradient vanishing

    - Hardmard product

    - easily assembled

  - 3.2 Theoretical Performance Analysis

    - Extra Weights and Linear Term for Approximation Capa-
  bility (P1) Improvement:

    - Hadamard Product for Computation Complexity (P2) Optimization:

    - Linear Term for Converge Performance ( P3 ) Enhancement

    - First-order Neuron Combination for Implementation Feasibility ( P4 ) Improvement

  -  4 QUADRALIB FOR QDNN DESIGN EXPLORATION

      <img src="Summarize.assets/image (6).png" alt="image (6)" width="100%" />

    - In **Model** Level, QuadraLib defines a set of encapsulated layer modules

      - auto-builder for converting first-order model to a corresponding QDNN version

    - In **Training**/Inference Level, QuadraLib leverages a memory profiler to monitor the memory cost of the generated QDNN model 

    - In **Application** Level, QuadraLib also provides model analysis tools such as activation and weight/gradient distribution **visualization**.

    - 4.3 QuadraLib Training Optimization

      - Hybrid Back-propagation for Memory Efficiency

      <img src="Summarize.assets/image (7).png" alt="image (7)" width="50%" />
      
      
      - Quadratic Optimizer Implementation
      
    - 6 CONCLUSION AND DISCUSSION

      - QDNNs show a significant potential on learning tasks which highly depends on **extracted objects,** such as object detection, segmentation, and position recognition.

      - the recognition process will more focus on the important objects while ignoring the unimportant background



---

## ML 

### Why Globally Re-shuffle? Revisiting Data Shuffling in Large Scale Deep Learning

#### Abstract

- Challenge
  - SGD gives enormous pressure on the I/O subsystem.
  - In HPC environments, we often replicate the entire dataset to node local SSDs.
- Contribution
  - Investigate the viability of partitioning the dataset among workers performing only a **partial distributed exchange** of samples in each training epoch.
  - Demonstrate that in practice validation accuracy of global shuffling exchange $\approx$ partial distributed exchange when carefully tuning.


#### 1 INTRODUCTION

- Challenge
  - As datasets growing up, it is hard to store the **entire dataset** on compute node local storage, or by each node reading a subset of the samples from the **parallel file system** (PFS)
- Insight
  - **Random access** to the input samples has been in fact identified as one of the major contributors to **poor I/O performance**.
- Contribution
  - Revisit data shuffling strategies when scaling deep learning applications to a large number of workers.
  - Implement a dataset partitioning, shuffling, and redistribution solution for distributed training.
  - **Local shuffling achieves accuracy** $\approx$ **Global shuffling strategy** !!!
    - Average store **#0.03%** dataset
  - **Training time**: local shuffling **#5x** < global shuffling

#### 2 BACKGROUND AND MOTIVATION

#### 3 DESIGN AND IMPLEMENTATION

- To reduce I/O requirements: 
  - a) we split the dataset across workers, 
  - b) workers train on the local partition, and 
  - c) worker exchange a subset of the locals samples before each epoch
- A. Data Partitioning and Shuffling Scheme
  - Each worker exchanges globally a fraction Q of its local samples before each epoch. Q=1: full global shuffle, Q=0: pure local shuffle
  - In this paper, they use **partial local shuffling (PLS)**, which means 0<Q<1
  - PLS requires each worker **stores** up to $(1 + Q) N/M$ **samples**:
    - $N/M$ (LS) < $(1 + Q) N/M$ (PLS) < $M/2$ (GS)
- B. Exchanging Samples Between Workers
- C. Implementation
  - Two data primitives provided by PyTorch
    - A `Dataset` to store the samples and their corresponding labels
    -  A `DataLoader` to iterate over the (batches of) samples
  - `Scheduler` for managing the global exchange
    - Reduce the overhead: overlap communication with the forward and backward phases
    - <img src="Summarize.assets/image-20230109133337630.png" alt="image-20230109133337630" width="100%" />
    - E.g. A epoch with $I=N/(bM)$ iterations, $b$ is a batch size. In each iteration, $(Q\times N/M)/I = Q\times b$ samples are sent/received from one worker

#### 4 SHUFFLING IN DISTRIBUTED SGD

- **Understanding**: **partial local shuffling Accuracy** $\approx$  g**lobal shuffling Accuracy**

- A. Partial Local vs. Global Shuffling: Gradients

  -  partial local shuffling produces the **same** gradients as global shuffling.

- B. Convergence Rate and Shuffling Error

  - for insufficient global shuffling in the non-convex case, the convergence rate’s upper bound include three terms

  - $$O\left(\sqrt{\frac{1}{S|N|}}+\frac{\log |N|}{|N|}+\frac{|N| \epsilon(A, N)^2}{b|M|}\right)$$

    -  $S$ is the number of epochs, and $\epsilon(A, N)$ is the shuffling error of algorithm $A$ with the samples $N$
    -  And shuffling error can be simplified: $\epsilon(A, h, N)=1-\frac{\sigma}{|N| !}$
    
  - for practical dataset sizes and number of workers, the shuffling error $\epsilon(A, h, N)\rightarrow1$. 

#### 5 EVALUATION

- D. Equivalence of Local and Global Shuffling: **When Local is Enough**
  - while local shuffling starts to **converge slower** than its global counterpart, local partial shuffling provides almost identical accuracy trajectory with global sampling, which in turn with a feasible learning rate schedule could lead to **faster overall convergence** and thus a reduction in runtime.
  - On some larger datasets, **local shuffling accuracy == global shuffling**
    - Indicates that workers **do not** actually need to process a large portion of the whole dataset, and exchanging the gradient weights is **enough** to ensure convergence.
    - <img src="Summarize.assets/image-20230109155659742.png" alt="image-20230109155659742" width="40%;" />
  - Since global shuffling reads from the PFS, the cost of I/O is much higher than those of local and partial shuffling.
    - <img src="Summarize.assets/image-20230109160514415.png" alt="image-20230109160514415" width="40%;" />

#### 6 RELATED WORK

#### 7 CONCLUSION

- Eliminating **unnecessary shuffling** of data samples in distributed SGD can have profound implications on the I/O requirements of the overall training procedure.
- No need to replicate data everywhere, which reduces the cost of data staging in HPC environments. 
- **Smaller local data storage** suffices that could enable training comparable neural network models in **more** modest storage **environments**, such as over local tmpfs, opening up the potential for less powerful HPC systems to be utilized for deep learning workloads.



---









