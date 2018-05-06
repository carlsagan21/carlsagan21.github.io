---
title: Gehr, Timon, et al. "AI 2 - Safety and Robustness Certification of Neural Networks with Abstract Interpretation."
---

[WARNING] NOT MY WRITING - just for save and translation

see: <https://www.sri.inf.ethz.ch/publications.php>

AI2: Safety and Robustness Certification of Neural Networks with Abstract Interpretation 
Timon Gehr, Matthew Mirman, Dana Drachsler-Cohen, Petar Tsankov, Swarat Chaudhuri, Martin Vechev
IEEE S&P 2018

## Abstract

We present AI2, the first sound and scalable an- alyzer for deep neural networks. Based on overapproximation, AI2 can automatically prove safety properties (e.g., robustness) of realistic neural networks (e.g., convolutional neural networks).

The key insight behind AI2 is to phrase reasoning about safety and robustness of neural networks in terms of classic abstract interpretation, enabling us to leverage decades of advances in that area. Concretely, we introduce abstract transformers that capture the behavior of fully connected and convolutional neural network layers with rectified linear unit activations (ReLU), as well as max pooling layers. This allows us to handle real-world neural networks, which are often built out of those types of layers.

We present a complete implementation of AI2 together with an extensive evaluation on 20 neural networks. Our results demonstrate that: (i) AI2 is precise enough to prove useful specifications (e.g., robustness), (ii) AI2 can be used to certify the effectiveness of state-of-the-art defenses for neural networks, (iii) AI2 is significantly faster than existing analyzers based on symbolic analysis, which often take hours to verify simple fully connected networks, and (iv) AI2 can handle deep convolutional networks, which are beyond the reach of existing methods.

Index Terms—Reliable Machine Learning, Robustness, Neural Networks, Abstract Interpretation

## I. INTRODUCTION

Recent years have shown a wide adoption of deep neural networks in safety-critical applications, including self-driving cars [2], malware detection [44], and aircraft collision avoid- ance detection [21]. This adoption can be attributed to the near-human accuracy obtained by these models [21], [23].

Despite their success, a fundamental challenge remains: to ensure that machine learning systems, and deep neural networks in particular, behave as intended. This challenge has become critical in light of recent research [40] showing that even highly accurate neural networks are vulnerable to adver- sarial examples. Adversarial examples are typically obtained by slightly perturbing an input that is correctly classified by the network, such that the network misclassifies the perturbed input. Various kinds of perturbations have been shown to suc- cessfully generate adversarial examples (e.g., [3], [12], [14], [15], [17], [18], [29], [30], [32], [38], [41]). Fig. 1 illustrates two attacks, FGSM and brightening, against a digit classifier. For each attack, Fig. 1 shows an input in the Original column, the perturbed input in the Perturbed column, and the pixels that were changed in the Diff column. Brightened pixels are marked in yellow and darkened pixels are marked in purple.

![Fig. 1: Attacks applied to MNIST images [25].]()

The FGSM [12] attack perturbs an image by adding to it a particular noise vector multiplied by a small number ε (in Fig. 1, ε = 0.3). The brightening attack (e.g., [32]) perturbs an image by changing all pixels above the threshold 1 − δ to the brightest possible value (in Fig. 1, δ = 0.085).

Adversarial examples can be especially problematic when safety-critical systems rely on neural networks. For instance, it has been shown that attacks can be executed physically (e.g., [9], [24]) and against neural networks accessible only as a black box (e.g., [12], [40], [43]). To mitigate these issues, recent research has focused on reasoning about neural network robustness, and in particular on local robustness. Local robust- ness (or robustness, for short) requires that all samples in the neighborhood of a given input are classified with the same label [31]. Many works have focused on designing defenses that increase robustness by using modified procedures for training the network (e.g., [12], [15], [27], [31], [42]). Others have developed approaches that can show non-robustness by underapproximating neural network behaviors [1] or methods that decide robustness of small fully connected feedforward networks [21]. However, no existing sound analyzer handles convolutional networks, one of the most popular architectures.

**Key Challenge: Scalability and Precision.** The main chal- lenge facing sound analysis of neural networks is scaling to large classifiers while maintaining a precision that suffices to prove useful properties. The analyzer must consider all possible outputs of the network over a prohibitively large set of inputs, processed by a vast number of intermediate neurons. For instance, consider the image of the digit 8 in Fig. 1 and suppose we would like to prove that no matter how we brighten the value of pixels with intensity above 1−0.085, the network will still classify the image as 8 (in this example we have 84 such pixels, shown in yellow). Assuming 64-bit floating point numbers are used to express pixel intensity, we obtain more than 101154 possible perturbed images. Thus, proving the property by running a network exhaustively on all possible input images and checking if all of them are classified as 8 is infeasible. To avoid this state space explosion, current methods (e.g., [18], [21], [34]) symbolically encode the network as a logical formula and then check robustness properties with a constraint solver. However, such solutions do not scale to larger (e.g., convolutional) networks, which usually involve many intermediate computations.

![Fig. 2: A high-level illustration of how AI2 checks that all perturbed inputs are classified the same way. AI2 first creates an abstract element A1 capturing all perturbed images. (Here, we use a 2-bounded set of zonotopes.) It then propagates A1 through the abstract transformer of each layer, obtaining new shapes. Finally, it verifies that all points in A4 correspond to outputs with the same classification.]()

**Key Concept: Abstract Interpretation for AI.** The key insight of our work is to address the above challenge by lever- aging the classic framework of abstract interpretation (e.g., [6], [7]), a theory which dictates how to obtain sound, computable, and precise finite approximations of potentially infinite sets of behaviors. Concretely, we leverage numerical abstract domains – a particularly good match, as AI systems tend to heavily manipulate numerical quantities. By showing how to apply abstract interpretation to reason about AI safety, we enable one to leverage decades of research and any future advancements in that area (e.g., in numerical domains [39]). With abstract interpretation, a neural network computation is overapproxi- mated using an abstract domain. An abstract domain consists of logical formulas that capture certain shapes (e.g., zonotopes, a restricted form of polyhedra). For example, in Fig. 2, the green zonotope A1 overapproximates the set of blue points (each point represents an image). Of course, sometimes, due to abstraction, a shape may also contain points that will not occur in any concrete execution (e.g., the red points in A2).

**The AI2 Analyzer.** Based on this insight, we developed a system called AI2 (Abstract Interpretation for Artificial Intelligence)1. AI2 is the first scalable analyzer that han- dles common network layer types, including fully connected and convolutional layers with rectified linear unit activations (ReLU) and max pooling layers.

To illustrate the operation of AI2, consider the example in Fig. 2, where we have a neural network, an image of the digit 8 and a set of perturbations: brightening with parameter 0.085. Our goal is to prove that the neural network classifies all perturbed images as 8. AI2 takes the image of the digit 8 and the perturbation type and creates an abstract element A1 that captures all perturbed images. In particular, we can capture the entire set of brightening perturbations exactly with a single zonotope. However, in general, this step may result in an abstract element that contains additional inputs (that is, red points). In the second step, A1 is automatically propagated through the layers of the network. Since layers work on concrete values and not abstract elements, this propagation requires us to define abstract layers (marked with #) that compute the effects of the layers on abstract elements. The abstract layers are commonly called the abstract transformers of the layers. Defining sound and precise, yet scalable abstract transformers is key to the success of an analysis based on abstract interpretation. We define abstract transformers for all three layer types shown in Fig. 2.

At the end of the analysis, the abstract output A4 is an overapproximation of all possible concrete outputs. This enables AI2 to verify safety properties such as robustness (e.g., are all images classified as 8?) directly on A4. In fact, with a single abstract run, AI2 was able to prove that a convolutional neural network classifies all of the considered perturbed images as 8.

We evaluated AI2 on important tasks such as verifying robustness and comparing neural networks defenses. For ex- ample, for the perturbed image of the digit 0 in Fig. 1, we showed that while a non-defended neural network classified the FGSM perturbation with ε = 0.3 as 9, this attack is provably eliminated when using a neural network trained with the defense of [27]. In fact, AI2 proved that the FGSM attack is unable to generate adversarial examples from this image for any ε between 0 and 0.3.

**Main Contributions.** Our main contributions are:

- A sound and scalable method for analysis of deep neural networks based on abstract interpretation (Section IV).
- AI2, an end-to-end analyzer, extensively evaluated on feed-forward and convolutional networks (computing with 53 000 neurons), far exceeding capabilities of current systems (Section VI).
- An application of AI2 to evaluate provable robustness of neural network defenses (Section VII).

## II. REPRESENTING NEURAL NETWORKS AS CONDITIONAL AFFINE TRANSFORMATIONS

In this section, we provide background on feedforward and convolutional neural networks and show how to transform them into a representation amenable to abstract interpretation. This representation helps us simplify the construction and description of our analyzer, which we discuss in later sections. We use the following notation: for a vector x ∈ Rn, xi denotes its ith entry, and for a matrix W ∈ Rn×m, Wi denotes its ith row and Wi,j denotes the entry in its ith row and jth column.

![Fig. 3: Definition of CAT functions.]()

**CAT Functions.** We express the neural network as a com- position of conditional affine transformations (CAT), which are affine transformations guarded by logical constraints. The class of CAT functions, shown in Fig. 3, consists of functions f: Rm → Rn for m,n ∈ N and is defined recursively. Any affine transformation f(x) = W · x + b is a CAT function, for a matrix W and a vector b. Given sequences of conditions E1,...,Ek and CAT functions f1,...,fk, we write:

```
f (x) = case E1 : f1 (x), . . . , case Ek : fk (x).
```

This is also a CAT function, which returns fi(x) for the first Ei satisfied by x. The conditions are conjunctions of constraints oftheformxi ≥xj,xi ≥0andxi <0.Finally,any composition of CAT functions is a CAT function. We often write f′′ ◦ f′ to denote the CAT function f(x) = f′′(f′(x)).

**Layers.** Neural networks are often organized as a sequence of layers, such that the output of one layer is the input of the next layer. Layers consist of neurons, performing the same function but with different parameters. The output of a layer is formed by stacking the outputs of the neurons into a vector or three-dimensional array. We will define the functionality in terms of entire layers instead of in terms of individual neurons.

**Reshaping of Inputs.** Layers often take three-dimensional inputs (e.g., colored images). Such inputs are transformed into vectors by reshaping. A three-dimensional array x ∈ Rm×n×r can be reshaped to xv ∈ Rm·n·r in a canonical way, first by depth, then by column, finally by row. That is, given x:

```
xv =(x1,1,1...x1,1,r x1,2,1...x1,2,r...xm,n,1...xm,n,r)T.
```

**Activation Function.** Typically, layers in a neural network perform a linear transformation followed by a non-linear activation function. We focus on the commonly used rectified linear unit (ReLU) activation function, which for x ∈ R is defined as ReLU(x) = max(0, x), and for a vector x ∈ Rm as ReLU(x)=(ReLU(x1), . . . , ReLU(xm)).

**ReLU to CAT.** WecanexpresstheReLUactivationfunction as ReLU = ReLUn ◦ . . . ◦ ReLU1 where ReLUi processes the ith entry of the input x and is given by:

```
ReLUi(x) = case (xi ≥ 0): x,
        case (xi < 0): Ii←0 · x.
```

I is the identity matrix with the ith row replaced by zeros.

**Fully Connected (FC) Layer.** An FC layer takes a vector of size m (the m outputs of the previous layer), and passes it to n neurons, each computing a function based on the neuron’s weights and bias, one weight for each component of the input. Formally, an FC layer with n neurons is a function FCW,b : Rm → Rn parameterized by a weight matrix W ∈Rn×m and a bias b∈Rn. For x∈Rm, we have:

```
FCW,b(x) = ReLU(W · x + b).
```

![Fig. 4a shows an FC layer computation for x = (2, 3, 1).]()

**Convolutional Layer.** A convolutional layer is defined by a series of t filters Fp,q = (Fp,q,..,Fp,q), parameterized by the same p and q, where p ≤ m and q ≤ n. A filter Fp,q is a function parameterized by a three-dimensional array of weightsW∈Rp×q×randabiasb∈R.Afiltertakesa three-dimensional array and returns a two-dimensional array:

```
F p,q : Rm×n×r → R(m−p+1)×(n−q+1).
```

The entries of the output y for a given input x are given by:

```
yi,j = ReLU( 􏰋 􏰋 􏰋 Wi′,j′,k′ ·x(i+i′−1),(j+j′−1),k′ +b).
```

Intuitively, this matrix is computed by sliding the filter along the height and width of the input three-dimensional array, each time reading a slice of size p×q×r, computing its dot product with W (resulting in a real number), adding b, and applying ReLU. The function ConvF , corresponding to a convolutional layer with t filters, has the following type:

```
ConvF : Rm×n×r → R(m−p+1)×(n−q+1)×t.
```

As expected, the function ConvF returns a three-dimensional array of depth t, which stacks the outputs produced by each filter. Fig. 4b illustrates a computation of a convolutional layer with a single filter. For example:

```
y1,1,1 =ReLU((1·0+0·4+(−1)·(−1)+2·0)+1)=2.
```

Here, the input is a three-dimensional array in R4×4×1. As the input depth is 1, the depth of the filter’s weights is also 1. The output depth is 1 because the layer has one filter.

**Convolutional Layer to CAT.** For a convolutional layer
ConvF,wedefineamatrixWF whoseentriesarethoseofthe
weight matrices for each filter (replicated to simulate sliding), and a bias b consisting of copies of the filters’ biases. We then treat the convolutional layer ConvF like the equivalent FCW F ,bF . We provide formal definitions of W F and bF in Appendix A. Here, we provide an intuitive illustration of the translation on the example in Fig. 4b. Consider the first entry y1,1 = 2 of y in Fig. 4b:

```
y1,1= ReLU(W1,1 ·x1,1 +W1,2 ·x1,2 +W2,1 ·x2,1 +W2,2 ·x2,2 +b).
```

When x is reshaped to a vector xv, the four entries x1,1, x1,2, x2,1 and x2,2 will be found in xv1, xv2, xv5 and xv6, respectively. Similarly, when y is reshaped to yv, the entry y1,1 will be found in y1v. Thus, to obtain y1v = y1,1, we define the first row in WF such that its 1st, 2nd, 5th, and 6th entries are W1,1 , W1,2 , W2,1 and W2,2 . The other entries are zeros.

![Fig. 4: One example computation for each of the three layer types supported by AI2.]()

We also define the first entry of the bias to be b. For similar reasons, to obtain y2v = y1,2, we define the second row in WF such that its 2nd , 3rd , 6th , and 7th entries are W1,1 , W1,2 , W2,1 and W2,2 (also b2 = b). By following this transformation, we
obtain the matrix WF ∈ R9 × R16 and the bias bF ∈ R9:

To aid understanding, we show the entries from W that appear in the resulting matrix WF in bold.

**Max Pooling (MP) Layer.** An MP layer takes a three-
   dimensional array x ∈ R
m×n×r
and reduces the height m of
   x by a factor of p and the width n of x by a factor of q (for p
and q dividing m and n). Depth is kept as-is. Neurons take as
input disjoint subrectangles of x of size p × q and return the
maximal value in their subrectangle. Formally, the MP layer
   is a function MaxPoolp,q : R
input x returns the three-dimensional array y given by:

```
yi,j,k =max({xi′,j′,k | p·(i−1)<i′ ≤p·i q·(j−1)<j′ ≤q·j}).
```

![Fig. 4c illustrates the max pooling computation for p = 2, q = 2 and r = 1. For example, here we have:]()

Max Pooling to CAT. Let MaxPool′
}) = 2.
: Rm·n·r → R p · q ·r
y
, x
, x
, x
1,1,1
1,1,1
1,2,1
2,1,1
p,q
2,2,1
0000000000000010 of CAT functions whose composition is MaxPool′ : 0000000000010000
mn
  MP 0000000100000000 W =0000000010000000 p,q 0000000001000000 its input and output: MaxPool′ (xv ) = MaxPool (x)v . To 0000000000001000 p,q p,q 0000000000000100 represent max pooling as a CAT function, we define a series 0000000000100000
be the function that is obtained from MaxPool by reshaping
  p,q 0000000000000001
MaxPool′p,q =fm·n·r ◦...◦f1 ◦fMP.

Fig. 5: The operation of the transformed max pooling layer.
The first function is fMP(xv) = WMP · xv, which reorders
its input vector xv to a vector xMP in which the values
of each max pooling subrectangle of x are adjacent. The
remaining functions execute standard max pooling. Concretely,
the function f ∈ {f ,...,fm n } executes max pooling i 1 p · q ·r
on the ith subrectangle by selecting the maximal value and removing the other values. We provide formal definitions of the CAT functions f MP and fi in Appendix A. Here, we illustrate them on the example from Fig. 4c, where r = 1. The CAT computation for this example is shown in Fig. 5. The computation begins from the input vector xv, which is the reshaping of x from Fig. 4c. The values of the first 2 × 2 subrectangle in x (namely, 0, 1, 2 and −4) are separated in xv by values from another subrectangle (3 and −2). To make them contiguous, we reorder xv using a permutation matrix WMP, yielding xMP. In our example, WMP is:

One entry in each row of W MP is 1, all other entries are zeros.

## III. BACKGROUND: ABSTRACT INTERPRETATION

We now provide a short introduction to Abstract Interpre- tation (AI). AI enables one to prove program properties on a set of inputs without actually running the program. Formally, givenafunctionf:Rm→Rn,asetofinputsX⊆Rm, and a property C ⊆ Rn, the goal is to determine whether the property holds, that is, whether ∀x ∈ X. f(x) ∈ C.

Fig. 6 shows a CAT function f : R2 → R2 that is defined as f(x) = 􏰁2 −1 􏰂·x and four input points for the function f, given as X = {(0, 1), (1, 1), (1, 3), (2, 2)}. Let the property be C = {(y1, y2) ∈ R | y1 ≥ −2}, which holds in this example. To reason about all inputs simultaneously, we lift the definition of f to be over a set of inputs X rather than a single input:

```
Tf:P(R )→P(R ),
Tf(X)={f(x)|x∈X}.
```

The function Tf is called the concrete transformer of f. With Tf, our goal is to determine whether Tf(X) ⊆ C for a given input set X. Because the set X can be very large (or infinite), we cannot enumerate all points in X to com- pute Tf (X ). Instead, AI overapproximates sets with abstract elements (drawn from some abstract domain A) and then defines a function, called an abstract transformer of f, which works with these abstract elements and overapproximates the effect of Tf. Then, the property C can be checked on the resulting abstract element returned by the abstract transformer. Naturally, because AI employs overapproximation, it is sound, but may be imprecise (i.e., may fail to prove the property when it holds). Next, we explain the ingredients of AI in more detail.

**Abstract Domains.** Abstract domains consist of shapes expressible as a set of logical constraints. A few popular numerical abstract domains are: Box (i.e., Interval), Zonotope, and Polyhedra. These domains provide different precision versus scalability trade-offs (e.g., Box’s abstract transformers are significantly faster than Polyhedra’s abstract transformers, but polyhedra are significantly more precise than boxes). The Box domain consists of boxes, captured by a set of constraints oftheforma≤xi ≤b,fora,b∈R∪{−∞,+∞}anda≤b. A box B contains all points which satisfy all constraints in B. In our example, X can be abstracted by the following box:

```
B={0≤x1 ≤2,1≤x2 ≤3}.
```

Note that B is not very precise since it includes 9 integer points (along with other points), whereas X has only 4 points. The Zonotope domain [10] consists of zonotopes. A zono- tope is a center-symmetric convex closed polyhedron Z ⊆ Rn
that can be represented as an affine function:

```
z : [a1, b1] × [a2, b2] × · · · × [am, bm] → Rn.
```

In other words, z has the form z(ε) = M ·ε+b where ε is a vector of error terms satisfying interval constraints εi ∈ [ai, bi] for 1 ≤ i ≤ m. The bias vector b captures the center of the zonotope, while the matrix M captures the boundaries of the zonotope around b. A zonotope z represents all vectors in the image of z (i.e., z[[a1,1 ] × · · · × [am, bm]]). In our example, X can be abstracted by the zonotope z : [−1, 1]3 → R2 :

```
z(ε1,ε2,ε3)=(1+0.5·ε1 +0.5·ε2,2+0.5·ε1 +0.5·ε3).
```

Zonotope is a more precise domain than Box: for our example, z includes only 7 integer points.
The Polyhedra [8] domain consists of convex closed poly- hedra, where a polyhedron is captured by a set of linear constraints of the form A · x ≤ b, for some matrix A and a vector b. A polyhedron C contains all points which satisfy the constraints in C. In our example, X can be abstracted by the following polyhedron:

```
C ={x2 ≤2·x1 +1,x2 ≤4−x1,x2 ≥1,x2 ≥x1}.
```

Polyhedra is a more precise domain than Zonotope: for our example, C includes only 5 integer points.
To conclude, abstract elements (from an abstract domain) represent large, potentially infinite sets. There are various abstract domains, providing different levels of precision and scalability.

**Abstract Transformers.** To compute the effect of a func- tion on an abstract element, AI uses the concept of an abstract transformer. Given the (lifted) concrete transformer Tf : P(Rm) → P(Rn) of a function f : Rm → Rn, an abstract transformer of Tf is a function over abstract domains, denoted by Tf# : Am → An. The superscripts denote the number of components of the represented vectors. For example, elements in Am represent sets of vectors of dimension m. This also determines which variables can appear in the constraints associated with an abstract element. For example, elements inAm constrainthevaluesofthevariablesx1,...,xm.
Abstract transformers have to be sound. To define sound- ness, we introduce two functions: the abstraction function α and the concretization function γ. An abstraction function αm : P(Rm) → Am maps a set of vectors to an abstract element in Am that overapproximates it. For example, in the Box domain:

```
α2({(0,1),(1,1),(1,3),(2,2)}) = {0 ≤ x1 ≤ 2,1 ≤ x2 ≤ 3}.
```

A concretization function γm : Am → P(Rm) does the opposite: it maps an abstract element to the set of concrete vectors that it represents. For example, for Box:

```
γ2({0 ≤ x1 ≤ 2,1 ≤ x2 ≤ 3}) ={(0,1),(0,2),(0,3), (1, 1), (1, 2), (1, 3),
(2,1),(2,2),(2,3),...}.
```

This only shows the 9 vectors with integer components. We can now define soundness. An abstract transformer Tf# is sound if for all a ∈ Am, we have Tf (γm(a)) ⊆ γn(Tf#(a)), where Tf is the concrete transformer. That is, an abstract transformer has to overapproximate the effect of a concrete transformer. For example, the transformers illustrated in Fig. 6 are sound. For instance, if we apply the Box transformer on the box in Fig. 6a, it will produce the box in Fig. 6b. The box in Fig. 6b includes all points that f could compute in principle when given any point included in the concretization of the box in Fig. 6a. Analogous properties hold for the Zonotope and Polyhedra transformers. It is also important that abstract transformers are precise. That is, the abstract output should include as few points as possible. For example, as we can see in Fig. 6b, the output produced by the Box transformer is less precise (it is larger) than the output produced by the Zonotope transformer, which in turn is less precise than the output produced by the Polyhedra transformer.

**Property Verification.** After obtaining the (abstract) output, we can check various properties of interest on the result. In general, an abstract output a = Tf#(X) proves a property Tf(X) ⊆ C if γn(a) ⊆ C. If the abstract output proves a property, we know that the property holds for all possible concrete values. However, the property may hold even if it cannot be proven with a given abstract domain. For example, in Fig. 6b, for all concrete points, the property C = {(y1 , y2 ) ∈ R2 | y1 ≥ −2} holds. However, with the Box domain, the abstract output violates C, which means that the Box domain is not precise enough to prove the property. In contrast, the Zonotope and Polyhedra domains are precise enough to prove the property.
In summary, to apply AI successfully, we need to: (a) find a suitable abstract domain, and (b) define abstract transformers that are sound and as precise as possible. In the next section, we introduce abstract transformers for neural networks that are parameterized by the numerical abstract domain. This means that we can explore the precision-scalability trade-off by plugging in different abstract domains.

## IV. AI2: AI FOR NEURAL NETWORKS

In this section we present AI2, an abstract interpretation framework for sound analysis of neural networks. We begin by defining abstract transformers for the different kinds of neural network layers. Using these transformers, we then show how to prove robustness properties of neural networks.

### A. Abstract Interpretation for CAT Functions

In this section, we show how to overapproximate CAT functions with AI. We illustrate the method on the example in Fig. 7, which shows a simple network that manipulates two-dimensional vectors using a single fully connected layer of the form f (x) = ReLU2 􏰁ReLU1 􏰁􏰁 2 −1 􏰂 · x􏰂􏰂. Recall that

```
ReLU (x) = case (x ≥ 0): x, ii
case (xi < 0): Ii←0 · x,
```

where Ii←0 is the identity matrix with the ith row replaced by the zero vector. We overapproximate the network behavior on an abstract input. The input can be obtained directly (see Sec. IV-B) or by abstracting a set of concrete inputs to an abstract element (using the abstraction function α). For our example, we use the concrete inputs (the blue points) from Fig 6. Those concrete inputs are abstracted to the green zonotope z0 : [−1, 1]3 → R2, given as:

```
z0(ε1,ε2,ε3) = (1+0.5·ε1 +0.5·ε2,2+0.5·ε1 +0.5·ε3).
```

Due to abstraction, more (spurious) points may be added. In this example, except the blue points, the entire area of the zonotope is spurious. We then apply abstract transformers to the abstract input. Note that, if a function f can be written as f = f ′′ ◦ f ′ , the concrete transformer for f is Tf = Tf ′′ ◦ Tf ′ . Similarly, given abstract transformers Tf ′ and Tf′′ , an abstract transformer for f is Tf′′ ◦ Tf′ . When
a neural network N = fl′ ◦ ··· ◦ f1′ is a composition of multiple CAT functions fi′ of the shape fi′(x) = W · x + b or fi(x) = case E1 : f1(x),...,case Ek : fk(x), we only have to define abstract transformers for these two kinds of functions. We then obtain the abstract transformer T = T ′ ◦···◦T ′ . Nfl f1

**Abstracting Affine Functions.** To abstract functions of the form f(x) = W · x + b, we assume that the underlying ab- stract domain supports the operator Aff that overapproximates such functions. We note that for Zonotope and Polyhedra, this operation is supported and exact. Fig. 7 demonstrates Aff as the first step taken for overapproximating the effect of the fully connected layer. Here, the resulting zonotope z1 : [−1,1]3 → R2 is:

```
z1(ε1, ε2, ε3) =
(2·(1+0.5·ε1 +0.5·ε2)−(2+0.5·ε1 +0.5·ε3),
2+0.5·ε1 +0.5·ε3) = (0.5·ε1 +ε2 −0.5·ε3,2+0.5·ε1 +0.5·ε3).
```

**Abstracting Case Functions.** To abstract functions of the form f(x) = case E1 : f1(x),...,case Ek : fk(x), we first split the abstract element a into the different cases (each defined by one of the expressions Ei), resulting in k abstract elements a1,...,ak. We then compute the result of T#(ai) fi for each ai. Finally, we unify the results to a single abstract element. To split and unify, we assume two standard operators for abstract domains: (1) meet with a conjunction of linear constraints and (2) join. The meet (⊓) operator is an abstract transformer for set intersection: for an inequality expression E from Fig. 3, γn(a)∩{x ∈ Rn | x |= E} ⊆ γn(a⊓E). The join (⊔) operator is an abstract transformer for set union: γn(a1) ∪ γn(a2) ⊆ γn(a1 ⊔ a2). We further assume that the abstract domain contains an element ⊥, which satisfies γn(⊥) = {}, ⊥ ⊓ E = ⊥ and a ⊔ ⊥ = a for a ∈ A.

For our example in Fig. 7, abstract interpretation continues on z1 using the meet and join operators. To compute the effect of ReLU1, z1 is split into two zonotopes z2 = z1 ⊓ (x1 ≥ 0) and z3 = z1 ⊓ (x1 < 0). One way to compute a meet between a zonotope and a linear constraint is to modify the intervals of the error terms (see [11]). In our example, the resulting zonotopes are z : [−1,1] × [0,1] × [−1,1] → R2 such that z2(ε) = z1(ε) and z3 : [−1,1]×[−1,0]×[−1,1] → R2 such that z3(ε) = z1(ε) for ε ̄ common to their respective domains. Note that both z2 and z3 contain small spurious areas, because the intersections of the respective linear constraints with z1 are not zonotopes. Therefore, they cannot be captured exactly by the domain. Here, the meet operator ⊓ overapproximates set intersection ∩ to get a sound, but not perfectly precise, result.

Then, the two cases of ReLU1 are processed separately. We apply the abstract transformer of f1(x) = x to z2 and we apply the abstract transformer of f2(x) = I0←0 · x to z3. The resulting zonotopes are z4 = z2 and z5 : [−1, 1]2 → R2 such that z5(ε1, ε3) = (0, 2+0.5·ε1 +0.5·ε3). These are then joined to obtain a single zonotope z6. Since z5 is contained in z4, we get z6 = z4 (of course, this need not always be the case). Then, z6 is passed to ReLU2. Because z6 ⊓(x1 < 0) = ⊥, this results in z7 = z6. Finally, γ2(z7) is our overapproximation of the network outputs for our initial set of points. The abstract element z7 is a finite representation of this infinite set.

```
For f(x)=W ·x+b, Tf#(a) = Aff(a,W,b). For f(x) = case E1 : f1(x),...,case Ek : fk(y),
Tf#(a) = 􏰊 fi#(a ⊓ Ei). 1≤i≤k
For f(x) = f2(f1(x)), T#(a) = T#(T#(a)). f f2f1
Fig. 8: Abstract transformers for CAT functions.
```

In summary, we define abstract transformers for every kind of CAT function (summarized in Fig. 8). These definitions are general and are compatible with any abstract domain A which has a minimum element ⊥ and supports (1) a meet operator between an abstract element and a conjunction of linear constraints, (2) a join operator between two abstract elements, and (3) an affine transformer. We assume that the operations are sound. We note that these operations are standard or definable with standard operations. By definition of the abstract transformers, we get soundness:

**Theorem 1.** For any CAT function f with transformer Tf : P(Rm) → P(Rn) and any abstract input a ∈ Am,

```
Tf (γm(a)) ⊆ γn(Tf#(a)).
```

Theorem 1 is the key to sound neural network analysis with our abstract transformers, as we explain in the next section.

### B. Neural Network Analysis with AI

In this section, we explain how to leverage AI with our ab- stract transformers to prove properties of neural networks. We focus on robustness properties below, however, the framework can be applied to reason about any safety property.
For robustness, we aim to determine if for a (possibly un- bounded) set of inputs, the outputs of a neural network satisfy a given condition. A robustness property for a neural network N:Rm →Rn isapair(X,C)∈P(Rm)×P(Rn)consisting ofarobustnessregionXandarobustnessconditionC.We say that the neural network N satisfies a robustness property (X,C) if N(x)∈C for all x∈X.

**Local Robustness.** This is a property (X,CL) where X is a robustness region and CL contains the outputs that describe the same label L:

```
n
CL= y∈R 􏰃􏰃argmax(yi)=L .
􏰃 i∈{1,...,n}
```

For example, Fig. 7 shows a neural network and a robustness property (X,C2) for X = {(0,1),(1,1),(1,3),(2,2)} and C2 = {y | argmax(y1,y2) = 2}. In this example, (X,C2) holds. Typically, we will want to check that there is some label L for which (X, CL) holds.
We now explain how our abstract transformers can be used to prove a given robustness property (X,C).

**Robustness Proofs using AI.** Assume we are given a neural network N : Rm → Rn, a robustness property (X, C) and an abstract domain A (supporting ⊔, ⊓ with a conjunction of linear constraints, Aff, and ⊥) with an abstraction function α and a concretization function γ. Further assume that N can be written as a CAT function. Denote by TN# the abstract transformer of N, as defined in Fig. 8. Then, the following condition is sufficient to prove that N satisfies (X,C):

```
γn(TN#(αm(X))) ⊆ C.
```

This follows from Theorem 1 and the properties of α and γ. Note that there may be abstract domains A that are not precise enough to prove that N satisfies (X,C), even if N in fact satisfies (X,C). On the other hand, if we are able to show that some abstract domain A proves that N satisfies (X,C), we know that it holds.

**Proving Containment.** To prove the property (X,C) given the result a = TN# (αm (X )) of abstract interpretation, we need to be able to show γn(a) ⊆ C. There is a general method if C is given by a CNF formula 􏰍i 􏰎j li,j where all literals li,j are linear constraints: we show that the negated formula 􏰎i 􏰍j ¬li,j is inconsistent with the abstract element
a by checking that a⊓􏰄􏰍 ¬l 􏰅=⊥ for all i.

For our example in Fig. 7, the goal is to check that all inputs are classified as 2. We can represent C using the formula y2 ≥ y1. Its negation is y2 < y1, and it suffices to show that no point in the concretization of the abstract output satisfies this negated constraint. As indeed z7 ⊓ (y2 < y1) = ⊥, the property is successfully verified. However, note that we would not be able to prove some other true properties, such as y1 ≥ 0. This property holds for all concrete outputs, but some points in the concretization of the output z7 do not satisfy it.

## V. IMPLEMENTATION OF AI2

The AI2 framework is implemented in the D programming language and supports any neural network composed of fully connected, convolutional, and max pooling layers.

**Properties.** AI2supportsproperties(X,C)whereXisspeci- fied by a zonotope and C by a conjunction of linear constraints over the output vector’s components. In our experiments, we illustrate the specification of local robustness properties where the region X is defined by a box or a line, both of which are precisely captured by a zonotope.

**Abstract Domains.** The AI2 system is fully integrated with all abstract domains supported by Apron [20], a popular library for numerical abstract domains, including: Box [7], Zonotope [10], and Polyhedra [8].

**BoundedPowerset.** Wealsoimplementedboundedpowerset domains (disjunctive abstractions [33], [36]), which can be instantiated with any of the above abstract domains. An ab- stract element in the powerset domain P(A) of an underlying abstract domain A is a set of abstract elements from A, con- cretizing to the union of the concretizations of the individual elements (i.e., γ(A) = 􏰌a∈A γ(a) for A ∈ P(A)).

The powerset domain can implement a precise join oper- ator using standard set union (potentially pruning redundant elements). However, since the increased precision can become prohibitively costly if many join operations are performed, the bounded powerset domain limits the number of abstract elements in a set to N (for some constant N).

We implemented bounded powerset domains on top of stan- dard powerset domains using a greedy heuristic that repeatedly replaces two abstract elements in a set by their join, until the number of abstract elements in the set is below the bound N. For joining, AI2 heuristically selects two abstract elements that minimize the distance between the centers of their bounding boxes. In our experiments, we denote by ZonotopeN or ZN the bounded powerset domain with bound N ≥ 2 and underlying abstract domain Zonotope.

## VI. EVALUATION OF AI2

In this section, we present our empirical evaluation of AI2. Before discussing the results in detail, we summarize our three most important findings:

- AI2 can prove useful robustness properties for convo- lutional networks with 53000 neurons and large fully connected feedforward networks with 1 800 neurons.
- AI2 benefits from more precise abstract domains: Zono- tope enables AI2 to prove substantially more properties over Box. Further, ZonotopeN, with N ≥ 2, can prove stronger robustness properties than Zonotope alone.
- AI2 scales better than the SMT-based Reluplex [21]: AI2 is able to verify robustness properties on large networks with ≥ 1200 neurons within few minutes, while Reluplex takes hours to verify the same properties.

In the following, we first describe our experimental setup. Then, we present and discuss our results.

### A. Experimental Setup

We now describe the datasets, neural networks, and robust- ness properties used in our experiments.

**Datasets.** We used two popular datasets: MNIST [25] and CIFAR-10 [22] (referred to as CIFAR from now on). MNIST consists of 60000 grayscale images of handwritten digits, whose resolution is 28 × 28 pixels. The images show white digits on a black background.

CIFAR consists of 60 000 colored photographs with 3 color channels, whose resolution is 32 × 32 pixels. The images are partitioned into 10 different classes (e.g., airplane or bird). Each photograph has a different background (unlike MNIST).

**Neural Networks.** We trained convolutional and fully con- nected feedforward networks on both datasets. All networks were trained using accelerated gradient descent with at least 50 epochs of batch size 128. The training completed when each network had a test set accuracy of at least 0.9.

For the convolutional networks, we used the LeNet ar- chitecture [26], which consists of the following sequence of layers: 2 convolutional, 1 max pooling, 2 convolutional, 1 max pooling, and 3 fully connected layers. We write np×q to denote a convolutional layer with n filters of size p × q, and m to denote a fully connected layer with m neurons. The hidden layers of the MNIST network are 83×3 , 83×3 , 143×3 , 143×3 , 50, 50, 10, and those of the CI- FAR network are 243×3, 243×3, 323×3, 323×3, 100, 100, 10. The max pooling layers of both networks have a size of 2 × 2. We trained our networks using an open-source implementa- tion [37].

We used 7 different architectures of fully connected feed- forward networks (FNNs). We write l × n to denote the FNN architecture with l layers, each consisting of n neurons. Note that this determines the network’s size; e.g., a 4 × 50 network has 200 neurons. For each dataset, MNIST and CIFAR, we trained FNNs with the following architectures: 3 × 20, 6 × 20, 3×50, 3×100, 6×100, 6×200, and 9×200.

**Robustness Properties.** In our experiments, we consider local robustness properties (X,CL) where the region X cap- tures changes to lighting conditions. This type of property is inspired by the work of [32], where adversarial examples were found by brightening the pixels of an image.

Formally, we consider robustness regions Sx,δ that are parameterized by an input x ∈ Rm and a robustness bound δ ∈ [0, 1]. The robustness region is defined as:

```
Sx,δ ={x′ ∈Rm |∀i∈[1,m].1−δ≤xi ≤x′i ≤1∨x′i =xi}.
```

For example, the robustness region for x = (0.6,0.85,0.9) and bound δ = 0.2 is given by the set:

```
{(0.6,x,x′) ∈ R3 | x ∈ [0.85,1],x′ ∈ [0.9,1]}.
```

Note that increasing the bound δ increases the region’s size. In our experiments, we used AI2 to check whether all inputs in a given region Sx,δ are classified to the label assigned to x. We consider 6 different robustness bounds δ, which are drawn
from the set ∆ = {0.001, 0.005, 0.025, 0.045, 0.065, 0.085}. We remark that our robustness properties are stronger than those considered in [32]. This is because, in a given robustness region Sx,δ, each pixel of the image x is brightened indepen- dently of the other pixels. We remark that this is useful to capture scenarios where only part of the image is brightened
(e.g., due to shadowing).

**Other perturbations.** Note that AI2 is not limited to cer- tifying robustness against such brightening perturbations. In general, AI2 can be used whenever the set of perturbed inputs can be overapproximated with a set of zonotopes in a precise way (i.e., without adding too many inputs that do not capture actual perturbations to the robustness region). For example, the inputs perturbed by an l∞ attack [3] are captured exactly by a single zonotope. Further, rotations and translations have low-dimensional parameter spaces, and therefore can be overapproximated by a set of zonotopes in a precise way.

**Benchmarks.** We selected 10 images from each dataset. Then, we specified a robustness property for each image and each robustness bound in ∆, resulting in 60 properties per dataset. We ran AI2 to check whether each neural network satisfies the robustness properties for the respective dataset. We compared the results using different abstract domains, including Box, Zonotope, and ZonotopeN with N ranging between 2 and 128.
We ran all experiments on an Ubuntu 16.04.3 LTS server with two Intel Xeon E5-2690 processors and 512GB of memory. To compare AI2 to existing solutions, we also ran the FNN benchmarks with Reluplex [21]. We did not run convolutional benchmarks with Reluplex as it currently does not support convolutional networks.

### B. Discussion of Results

In the following, we first present our results for convolu- tional networks. Then, we present experiments with different abstract domains and discuss how the domain’s precision affects AI2’s ability to verify robustness. We also plot AI2’s running times for different abstract domains to investigate the trade-off between precision and scalability. Finally, we compare AI2 to Reluplex.

**Proving Robustness of Convolutional Networks.** We start with our results for convolutional networks. AI2 terminated within 1.5 minutes when verifying properties on the MNIST network and within 1 hour when verifying the CIFAR network.

In Fig. 9, we show the fraction of robustness properties verified by AI2 for each robustness bound. We plot separate bars for Box and Zonotope to illustrate the effect of the domain’s precision on AI2’s ability to verify robustness.

For both networks, AI2 verified all robustness properties for the smallest bound 0.001 and it verified at least one property for the largest bound 0.085. This demonstrates that AI2 can verify properties of convolutional networks with rather wide robustness regions. Further, the number of verified properties converges to zero as the robustness bound increases. This is expected, as larger robustness regions are more likely to contain adversarial examples.

In Fig. 9a, we observe that Zonotope proves significantly more properties than Box. For example, Box fails to prove any robustness properties with bounds at least 0.025, while Zonotope proves 80% of the properties with bounds 0.025 and 0.045. This indicates that Box is often imprecise and fails to prove properties that the network satisfies.

Similarly, Fig. 9b shows that Zonotope proves more robust- ness properties than Box also for the CIFAR convolutional net- work. The difference between these two domains is, however, less significant than that observed for the MNIST network. For example, both Box and Zonotope prove the same properties for bounds 0.065 and 0.085.

**PrecisionofDifferentAbstractDomains.** Next,wedemon- strate that more precise abstract domains enable AI2 to prove stronger robustness properties. In this experiment, we consider our 9 × 200 MNIST and CIFAR networks, which are our largest fully connected feedforward networks. We evaluate the Box, Zonotope, and the ZonotopeN domains for exponentially increasing bounds of N between 2 and 64. We do not report results for the Polyhedra domain, which takes several days to terminate for our smallest networks.

In Fig. 10, we show the fraction of verified robustness properties as a function of the abstract domain used by AI2. We plot a separate line for each robustness bound. All runs of AI2 in this experiment completed within 1 hour.

The graphs show that Zonotope proves more robustness properties than Box. For the MNIST network, Box proves 11 out of all 60 robustness properties (across all 6 bounds), failing to prove any robustness properties with bounds above 0.005. In contrast, Zonotope proves 43 out of the 60 properties and proves at least 50% of the properties across the 6 robustness bounds. For the CIFAR network, Box proves 25 out of the 60 properties while Zonotope proves 35.

The data also demonstrates that bounded sets of zonotopes further improve AI2’s ability to prove robustness properties. For the MNIST network, Zonotope64 proves more robustness properties than Zonotope for all 4 bounds for which Zonotope fails to prove at least one property (i.e., for bounds δ ≥ 0.025). For the CIFAR network, Zonotope64 proves more properties than Zonotope for 4 out of the 5 the bounds. The only exception is the bound 0.085, where Zonotope64 and Zonotope prove the same set of properties.

**Trade-off between Precision and Scalability.** In Fig. 11, we plot the running time of AI2 as a function of the abstract domain. Each point in the graph represents the average running time of AI2 when proving a robustness property for a given MNIST network (as indicated in the legend). We use a log-log plot to better visualize the trade-off in time.
The data shows that AI2 can efficiently verify robustness of large networks. AI2 terminates within a few minutes for all MNIST FNNs and all considered domains. Further, we observe that AI2 takes less than 10 seconds on average to verify a property with the Zonotope domain.
As expected, the graph demonstrates that more precise domains increase AI2’s running time. More importantly, AI2’s running time is polynomial in the bound N of ZonotopeN, which allows one to adjust AI2’s precision by increasing N.

**Comparison to Reluplex.** The current state-of-the-art system for verifying properties of neural networks is Reluplex [21]. Reluplex supports FNNs with ReLU activation functions, and its analysis is sound and complete. Reluplex would eventually either verify a given property or return a counterexample.
To compare the performance of Reluplex and AI2, we ran both systems on all MNIST FNN benchmarks. We ran AI2 using Zonotope and Zonotope64. For both Reluplex and AI2, we set a 1 hour timeout for verifying a single property.

Fig. 12 presents our results: Fig. 12a plots the average running time of Reluplex and AI2 and Fig. 12b shows the fraction of robustness properties verified by the systems. The data shows that Reluplex can analyze FNNs with at most 600 neurons efficiently, typically within a few minutes. Overall, both system verified roughly the same set of properties. However, Reluplex crashed during verification of some of the properties. This explains why AI2 was able to prove slightly more properties than Reluplex on the smaller FNNs.

For large networks with more than 600 neurons, the running time of Reluplex increases significantly and its analysis often times out. In contrast, AI2 analyzes the large networks within a few minutes and verifies substantially more robustness properties than Reluplex. For example, Zonotope64 proves 57 out of the 60 properties on the 6 × 200 network, while Reluplex proves 3. Further, Zonotope64 proves 45 out of the 60 properties on the largest 9 × 200 network, while Reluplex proves none. We remark that while Reluplex did not verify any property on the largest 9 × 200 network, it did disprove some of the properties and returned counterexamples.

We also ran Reluplex without a predefined timeout to investigate how long it would take to verify properties on the large networks. To this end, we ran Reluplex on properties that AI2 successfully verified. We observed that Reluplex often took more than 24 hours to terminate. Overall, our results indicate that Reluplex does not scale to larger FNNs whereas AI2 succeeds on these networks.

## VII. COMPARING DEFENSES WITH AI2

In this section, we illustrate a practical application of AI2: evaluating and comparing neural network defenses. A defense is an algorithm whose goal is to reduce the effectiveness of a certain attack against a specific network, for example, by retraining the network with an altered loss function. Since the discovery of adversarial examples, many works have sug- gested different kinds of defenses to mitigate this phenomenon (e.g., [12], [27], [42]). A natural metric to compare defenses is the average “size” of the robustness region on some test set. Intuitively, the greater this size is, the more robust the defense.

We compared three state-of-the-art defenses:

- GSS [12] extends the loss with a regularization term encoding the fast gradient sign method (FGSM) attack.
- Ensemble [42] is similar to GSS, but includes regular- ization terms from attacks on other models.
- MMSTV [27] adds, during training, a perturbation layer before the input layer which applies the FGSMk attack. FGSMk is a multi-step variant of FGSM, also known as projected gradient descent.

All these defenses attempt to reduce the effectiveness of the FGSM attack [12]. This attack consists of taking a network N and an input x and computing a vector ρN,x in the input space along which an adversarial example is likely to be found. An adversarial input a is then generated by taking a step ε along this vector: a=x+ε·ρN,x.

We define a new kind of robustness region, called line, that captures resilience with respect to the FGSM attack. The line robustness region captures all points from x to x + δ · ρN,x for some robustness bound δ:

```
LN,x,δ ={x+ε·ρN,x |ε∈[0,δ]}.
```

This robustness region is a zonotope and can thus be precisely captured by AI2.

We compared the three state-of-the-art defenses on the MNIST convolutional network described in Section VI; we call this the Original network. We trained the Original network with each of the defenses, which resulted in 3 additional networks: GSS, Ensemble, and MMSTV. We used 40 epochs for GSS, 12 epochs for Ensemble, and 10 000 training steps for MMSTV using their published frameworks.



## References
