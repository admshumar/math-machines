---
layout: post
title:  "Steerable Convolutional Neural Networks"
date:   2019-04-01 14:11:25 +0200
categories: jekyll update
---
(Notes from a reading of [Cohen and Welling: Steerable CNNs][steerable-cnn].)

Consider $$\mathbb{R}^2$$ as an affine hyperplane in $$\mathbb{R}^3$$, embedded via the map $$x\mapsto (x,1)$$. Then the Euclidean motion group $$\tilde G = \mathbb{R}^2\ltimes O(2)$$ has a convenient matrix representation. Let $$r$$ be a rotation and $$t$$ a translation. Then

\\[
\begin{split}
r &= \begin{pmatrix} R & 0 \\\\ 0 & 1 \\\\ \end{pmatrix} \\\\\\\\
t &= \begin{pmatrix} I & T \\\\ 0 & 1 \\\\ \end{pmatrix}
\end{split}
\\]	

where $$R\in O(2)$$ and $$T\in \mathbb{R}^2$$. Given $$x\in \mathbb{R}^2$$, we may identify it with a translation map in $$\tilde G$$ via

$$\bar x = \begin{pmatrix} I & x \\ 0 & 1 \\ \end{pmatrix}.$$

In image recognition, $$\mathbb{Z}^2$$ parametrizes the pixels of an image with infinite width and height, and a discrete subgroup $$G$$ of $$\tilde G$$ acts on this parametrization. In particular, $$G = \mathbb{Z}^2\ltimes D_4$$, so that our parametrization is the homogeneous space $$\mathbb{Z}^2 = G/D_4$$. Let’s call this parametrization the *pixel space*.

Images are described by *feature maps*, which are functions  $$f:G/D_4 \rightarrow \mathbb{R}^K$$, with each dimension of the target interpreted as a color channel. For example, $$K=3$$ may correspond to an RGB image, and $$K=4$$ may correspond to a CMYK image. A feature map typically takes non-negative rational values, and these values correspond to pixel intensities. A representation of a real-world image would thus be a compactly supported feature map.

Over the pixel space, we have a homogeneous vector bundle $$G\times_{D_4} \mathbb{R}^K$$ with an action by the discrete motion group given by $$ g'\cdot(g,v) = (g'g, v)$$. Let $$\mathcal{F} = \Gamma(G\times_{D_4} \mathbb{R}^K)$$ be the space of all feature maps. $$G$$ acts on it by left translation:
$$ \pi(tr) f(xD_4) = (tr)\cdot f((tr)^{-1}\cdot xD_4).$$

Let $$\Psi: \mathcal{F}\rightarrow \mathbb{R}^{K’}$$ be a *filter bank*. This can be thought of as a collection of $$K'$$ linear functionals on the space of feature maps. Each linear functional is an operation that is performed on the value of a fixed pixel (which we may as well assume is the origin). Each linear functional outputs a weighted sum of the value associated to the fixed pixel and its neighbors. By translating pixels to the origin, we can construct a feature map $$\Psi\ast f\in \mathcal{F}’$$, where $$\mathcal{F}’ = \Gamma(G\times_{D_4} \mathbb{R}^{K’})$$ is another space of feature maps. This is defined as follows:

$$(\Psi\ast f)(x) = \Psi(\pi(\bar x)^{-1} f).$$

Thus we get a map $$\Phi:\mathcal{F}\rightarrow \mathcal{F}’$$ via $$\Phi(f) = \Psi\ast f$$. This map is called a *convolutional neural network*.

Let $$(\mathbb{R}^{K’},\rho)$$ be a representation of $$D_4$$ on the model fiber of $$\mathcal{F}’$$. Our objective is to construct a representation $$(\mathcal{F}’,\pi’)$$ such that if the filter bank $$\Psi$$ intertwines the dihedral group representations $$(\mathcal{F},\pi)$$ and $$(\mathbb{R}^{K'},\rho)$$, then the convolutional neural network $$\Phi$$ intertwines the discrete motion group representations $$(\mathcal{F},\pi)$$ and $$(\mathcal{F}',\pi')$$. First we need some algebra:

**Lemma.** Let $$r$$ be a rotation, $$t$$ a translation, and $$\bar x$$ the translation corresponding to the position $$x\in\mathbb{R}^2$$. Then

$$ (tr)^{-1} \bar x^{-1} r = \overline{(tr)^{-1}\cdot x}. $$

The proof is a straightforward application of the given matrix representation. This result leads to the following equivariance rule.

**Proposition.** Let $$r$$ be a rotation, $$t$$ a translation, and $$\bar x$$ the translation corresponding to the position $$x\in\mathbb{Z}^2$$. If $$\Psi\pi(r) = \rho(r)\Psi$$ for all $$r\in D_4$$, then

$$ (\Psi\ast \pi(tr) f)(x) = \rho(r)(\Psi\ast f)((tr)^{-1}\cdot x).$$

*Proof.* By using an identity trick, we can exploit our two versions of the same map to produce the equivariance law:

\\[ 
\begin{split}
(\Psi\ast \pi(tr) f)(x) &= \Psi(\pi(\bar x)^{-1}\pi(tr)f)\\\\ &= \rho(r)\Psi(\pi(r)\pi(r)^{-1}\pi(\bar x)^{-1}\pi(tr)f) \\\\ &= \rho(r)\Psi(\pi(r^{-1}\bar x^{-1}tr)f)\\\\ &= \rho(r)\Psi(\pi((tr)^{-1}\bar xr)^{-1}f)\\\\ &= \rho(r)\Psi(\pi(\overline{(tr)^{-1}\cdot x})^{-1}f)\\\\ &= \rho(r)(\Psi\ast f)((tr)^{-1}\cdot x)
\end{split} 
\\]

With this calculation in mind, we are now in a position to define a representation of $$G$$ on $$\mathcal{F}'$$:

$$ \pi'(tr)(\Psi\ast f)(x) = \rho(x)(\Psi\ast f)((tr)^{-1}\cdot x). $$

To verify this, we note that one checks the relation $$f(t_1r_1t_2r_2) = f(t_1r_1)f(t_2r_2)$$ by using the fact that the conjugate of a translation by a rotation is again a translation, and that

$$ t_1r_1t_2r_2 = t_1r_1t_2r_1^{-1}r_1r_2. $$

As a consequence, we get an *intertwining property*:

**Corollary.** If $$\Psi\pi(r) = \rho(r)\Psi$$ for all $$r\in D_4$$, then $$ \Phi\, \pi(g) = \pi'(g)\, \Phi $$ for all $$g\in G$$.

Whenever we can find representations $$(\mathcal{F},\pi)$$ and $$(\mathcal{F}',\pi')$$ for which $$\Phi$$ is an intertwiner, we say that $$\Phi$$ is a *steerable convolutional neural network*.

To determine the homogeneous vector bundle of which  $$\Psi\ast f$$ is a section, it is enough to calculate the action of $$D_4$$ on $$(\Psi\ast f)(0)$$:

\\[\pi'(r)(\Psi\ast f)(0) = \rho(r)(\Psi\ast f)(\pi(r)^{-1}\cdot 0) = \rho(r)(\Psi\ast f)(0).\\]

This means that $$ \Psi\ast f\in \Gamma(G\times_\rho \mathbb{R}^{K'})$$. From representation theory, we know that

$$\Gamma(G\times_\rho \mathbb{R}^{K'}) \cong Ind_{D_4}^G(\rho)$$

thus we may interpret $$ \Phi$$ as a map into an induced representation of the discrete motion group. Moreover, we can treat $$ \Gamma(G\times_{D_4} \mathbb{R}^K)$$ as an induced representation as well. 

On $$ \mathcal{F}$$, the action of $$ D_4$$ should only rotate the pixel of the image; there should be no linear transformations within fibers (i.e. no transformations of color channels). This means that $$ D_4$$ has a trivial action on the value of $$ f$$ at the origin. Hence we are regarding $$ \mathcal{F}$$ as $$ \Gamma(G\times_{\rho_0} \mathbb{R}^K)$$, where $$ \rho_0$$ denotes the trivial representation of $$ D_4$$. To summarize,
\\[\Gamma(G\times_{\rho_0} \mathbb{R}^K)\cong Ind_{D_4}^G(\rho_0)\cong C(\mathbb{Z}^2).\\]
We can now restate this picture in a representation theoretic context.

**Theorem.** Let $$ (\mathbb{R}^K,\rho_0)$$ be a trivial representation of $$ D_4$$ and $$ Ind_{D_4}^G(\rho_0)$$ a space of feature maps. Let $$ \Psi : Ind_{D_4}^G(\rho_0) \rightarrow \mathbb{R}^{K'}$$ be a $$ D_4$$-equivariant filter with respect to $$ (\mathbb{R}^{K'},\rho)$$. Then \\[\Phi: Ind_{D_4}^G(\rho_0) \rightarrow Ind_{D_4}^G(\rho)\\] is a steerable convolutional neural network.

[steerable-cnn]: https://arxiv.org/abs/1612.08498