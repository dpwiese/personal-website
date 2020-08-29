---
title: "MIMO Adaptive Control with Python"
date: 2020-08-26T19:12:00-04:00
draft: true
toc: false
images: ["img/posts/adaptive-mimo-python/og-image.png"]
tags: 
  - adaptive learning
  - control theory
  - python
keywords: [adaptive learning, control theory, python]
---

# Introduction

In a previous post, [Adaptive PI Control with Python](/posts/adaptive-pi-python/), an example of an adaptive PI controller was presented and the [Python Control Systems Library](https://python-control.org/) used to simulate the closed-loop response.
This was a simple academic problem well suited to demonstrating the abilities of the library.
Since then I've been re-implementing various other simulation examples as well.
The current post presents a more complex example of a classical multi-input multi-output adaptive system.
No attempt is made to explain the steps in the open-loop analysis or control synthesis, or prove stability - they are simply presented as-is.
For more details on the design of such a controller see Reference [^narendra.stable.2005].
This post is meant only as a quick walkthrough to share another example of the Python Control Systems Library.

# Open-Loop System

Consider the following LTI plant
\begin{equation*}
  \begin{split}
    \dot{x}&=Ax+Bu \\\\
    y&=Cx
  \end{split}
\end{equation*}
where $x\in\mathbb{R}^{n}$, $y\in\mathbb{R}^{p}$, and $u\in\mathbb{R}^{m}$ and the system matrices are given by
\begin{equation}\label{eqn.adaptive.plant_matrices}
  A=
  \begin{bmatrix}
    -2 & -1 & 0 & 0 & 0 \\\\
    1 & 0 & 0 & 0 & 0 \\\\
    0 & 0 & -4 & -2 & 0 \\\\
    0 & 0 & 2 & 0 & 0 \\\\
    0 & 0 & 0 & 0 & 3 
  \end{bmatrix}
  \qquad
  B=
  \begin{bmatrix}
    1 & 0 \\\\
    0 & 0 \\\\
    0 & 1 \\\\
    0 & 0 \\\\
    0 & 1 
  \end{bmatrix}
  \qquad
  C= 
  \begin{bmatrix}
    1 & 1 & 0 & 1 & 0 \\\\
    0 & 1 & 0 & 0 & 1
  \end{bmatrix}
\end{equation}
The system is fifth order with two inputs and two outputs.
The system transfer matrix $W_{p}(s)$ is defined as follows
\begin{equation*}
  W_{p}(s)\triangleq C(sI-A)^{-1}B\in\mathbb{R}_{p}^{p\times m}(s)
\end{equation*} 
and for the system with matrices in \eqref{eqn.adaptive.plant_matrices} gives
\begin{equation*}
  W_{p}(s)=
  \begin{bmatrix}
    \frac{1}{s+1}& \frac{2}{(s+2)^{2}} \\\\
    \frac{1}{(s+1)^{2}} & \frac{1}{s-3}
  \end{bmatrix}
\end{equation*}
In the subsections that follow we will design a classical adaptive controller for this system.

## Requirements

First, several requirements must be checked to determine whether the intended classical adaptive controller is applicable to the system in \eqref{eqn.adaptive.plant_matrices}.
These requirements are:

1. Plant must be square, that is $m=p$.
2. The plant has no unstable transmission zeros.
    * This is to ensure pole-zero cancellations do not occur in the right half plane.
3. The Hermite form $H_{p}(s)$ of $W_{p}(s)$ is diagonal.
    * The only information needed to determine whether the Hermite form $H_{p}(s)$ of $W_{p}(s)$ is diagonal is the relative degree of each entry of $W_{p}(s)$.
4. The sign of the high frequency gain satisfies a sign definite condition.
5. An upper bound on the observability index is known.

In what follows we will verify that these requirements are satisfied, select a suitable reference model, define the control filters and finally assembled these various pieces resulting in the completed controller.

## Evaluate System Transmission Zeros

This is accomplished by determining the system's Smith-McMillan form, which useful in obtaining the poles and zeros (with their multiplicities) from a given transfer matrix.
The Smith-McMillan form is found by dividing the system's [Smith Form](https://en.wikipedia.org/wiki/Smith_normal_form) by the minimum polynomial $d(s)$.

First calculate the minimum polynomial $d(s)$ of $W_{p}(s)$ as:
\begin{equation*}
  d(s)=(s+1)^{2}(s+2)^2(s-3)
\end{equation*}
and express $W_{p}(s)$ as
\begin{equation*}
  W_{p}(s)=\frac{1}{d(s)}P(s)
\end{equation*}
giving
\begin{equation*}
  P(s)=
  \begin{bmatrix}
    (s+1)(s+2)^{2}(s-3) & 2(s+1)^{2}(s-3) \\\\
    (s+2)^{2}(s-3) & (s+1)^{2}(s+2)^{2}
  \end{bmatrix}
\end{equation*}

Find the Smith form of $P(s)$ by finding the determinantal devisors as follows, with $D_{0}\triangleq1$.
The $1\times1$ minors of $P(s)$ are:
\begin{equation*}
  (s+1)(s+2)^{2}(s-3)
  \qquad
  2(s+1)^{2}(s-3)
  \qquad
  (s+2)^{2}(s-3)
  \qquad
  (s+1)^{2}(s+2)^{2}
\end{equation*}
The greatest common devisor of these matrices is 1, so we set 
\begin{equation*}
  D_{1}(s)=1
\end{equation*}
The $2\times2$ minor of $W_{p}(s)$ is:
\begin{equation*}
  (s+1)^{3}(s+2)^{4}(s-3)-2(s+1)^{2}(s+2)^{2}(s-3)^{2} = (s+1)^{2}(s+2)^{2}(s-3)[(s+1)(s+2)^{2}-2(s-3)]
\end{equation*}

The greatest common devisor of this is itself, so we set
\begin{equation*}
  D_{2}(s)=(s+1)^{2}(s+2)^{2}(s-3)[(s+1)(s+2)^{2}-2(s-3)]
\end{equation*}
We then calculate $\epsilon_{i}^{\prime}$ as
\begin{equation*}
  \epsilon_{i}'(s)=\frac{D_{i}(s)}{D_{i-1}(s)}
\end{equation*}
giving
\begin{equation*}
  \begin{split}
    \epsilon_{1}^{\prime}(s)&=1 \\\\
    \epsilon_{2}^{\prime}(s)&=(s+1)^{2}(s+2)^{2}(s-3)[(s+1)(s+2)^{2}-2(s-3)]
  \end{split}
\end{equation*}
So the Smith form of $P(s)$ is
\begin{equation*}
  S_{P}(s)=
  \begin{bmatrix}
  1 & 0 \\\\
  0 & (s+1)^{2}(s+2)^{2}(s-3)[(s+1)(s+2)^{2}-2(s-3)]
  \end{bmatrix}
\end{equation*}
To get the Smith-McMillan form of $W_{p}(s)$ we divide the Smith form $S_{P}(s)$ by the minimum polynomial $d(s)$.
\begin{equation*}
  \begin{split}
    SM_{W_{p}}(s)&=
    \frac{1}{(s+1)^{2}(s+2)^2(s-3)}
    \begin{bmatrix}
    1 & 0 \\\\
    0 & (s+1)^{2}(s+2)^{2}(s-3)[(s+1)(s+2)^{2}-2(s-3)]
    \end{bmatrix} \\\\
    &=
    \begin{bmatrix}
    \frac{1}{(s+1)^{2}(s+2)^2(s-3)} & 0 \\\\
    0 & (s+1)(s+2)^{2}-2(s-3)
    \end{bmatrix}
  \end{split}
\end{equation*}
Each of the diagonal entries of the Smith-McMillan form can be written as
\begin{equation*}
  \frac{\epsilon_{i}(s)}{\psi_{i}(s)}
\end{equation*}
with 
\begin{equation*}
  \begin{split}
    \epsilon_{1}&=1 \\\\
    \epsilon_{2}&=(s+1)(s+2)^{2}-2(s-3)
  \end{split}
\end{equation*}
and
\begin{equation*}
  \begin{split}
    \psi_{1}&=(s+1)^{2}(s+2)^2(s-3) \\\\
    \psi_{2}&=1
  \end{split}
\end{equation*}
And so the characteristic polynomial $p(s)$ can be calculated as 
\begin{equation*}
  \begin{split}
    p(s)&=\psi_{1}(s)\psi_{2}(s) \\\\
    &=(s+1)^{2}(s+2)^2(s-3)
  \end{split}
\end{equation*}
The zero polynomial is given by
\begin{equation*}
  \begin{split}
    z(s)&=\epsilon_{1}(s)\epsilon_{2}(s) \\\\
    &=(s+1)(s+2)^{2}-2(s-3) \\\\
    &=s^{3}+5s^{2}+6s+10
  \end{split}
\end{equation*}
We can see that this system is fifth order $n=5$, with a pole with multiplicity two at $s=-1$, a pole with multiplicity two at $s=-2$, and a pole at $s=3$.
It also has three <b>stable transmission zeros</b>, as verified using the [Routh–Hurwitz Stability Criterion](https://en.wikipedia.org/wiki/Routh–Hurwitz_stability_criterion).
The exact location of the transmission zeros is not important.

## Check Structure of Plant's Hermite Form

Check the matrix $E$ to see if it is nonsingular.
If so, the Hermite form $H_{p}(s)$ of $W_{p}(s)$ is diagonal, which will make control design easier.
To do this, we first find the minimum relative degree $n_{i}$ of the elements in each row of $W_{p}(s)$.
For our plant, this is
\begin{equation*}
  \begin{split}
    n_{1}&=1 \\\\
    n_{2}&=1
  \end{split}
\end{equation*}
Evaluate $E$ as in Reference[^narendra.stable.2005], p.396
\begin{equation*}
  E_{i}=\lim_{s\rightarrow\infty}s^{r_{i}}G_{i}(s)
\end{equation*}
where $G_{i}(s)$ corresponds to the $i^{\text{th}}$ row of $G(s)$.
This gives
\begin{equation*}
  E_{1}=\lim_{s\rightarrow\infty}s
  \begin{bmatrix}
    \frac{1}{s+1} & \frac{2}{(s+2)^{2}}
  \end{bmatrix}=
  \begin{bmatrix}
    1 & 0
  \end{bmatrix}
\end{equation*}
and
\begin{equation*}
  E_{2}=\lim_{s\rightarrow\infty}s
  \begin{bmatrix}
    \frac{1}{(s+1)^{2}} & \frac{1}{s-3}
  \end{bmatrix}=
  \begin{bmatrix}
    0 & 1
  \end{bmatrix}
\end{equation*}
So
\begin{equation*}
  E=
  \begin{bmatrix}
    1 & 0 \\\\
    0 & 1
  \end{bmatrix}
\end{equation*}
and since $E$ is nonsingular, the plant's Hermite form $H_{p}(s)$ is diagonal.

## Expressing the Plant's Hermite Form

We know the plant has a diagonal Hermite form.
It is given by
\begin{equation*}
  H_{p}(s)=
  \begin{bmatrix}
    \frac{1}{\pi^{n_{1}}(s)} & 0 &  \cdots & 0 \\\\
    0 & \frac{1}{\pi^{n_{2}}(s)} & & \vdots \\\\
    \vdots & & \ddots & 0 \\\\
    0 & \cdots & 0 & \frac{1}{\pi^{n_{m}}(s)}
  \end{bmatrix}
\end{equation*}
where $\pi(s)$ is any monic polynomial of degree 1 and $n_{i}$ is the minimum relative degree of the elements of $W_{p}(s)$ in the $i^{\text{th}}$ row.

<div class="rem-dan">
  Since $\pi(s)$ is any monic polynomial of degree 1, and since the class of reference models that we can use consists essentially of those asymptotically stable transfer matrices that are generated by the Hermite normal form of the plant, we will pick $\pi(s)=s+a$ where $a>0$.
</div>

With $n_{1}=1$ and $n_{2}=2$ this gives

\begin{equation}\label{eqn.adaptive.hermite_form}
  H_{p}(s)=
  \begin{bmatrix}
    \frac{1}{(s+a)} & 0 \\\\
    0 & \frac{1}{(s+a)}
  \end{bmatrix}
\end{equation}

# Controller Synthesis

## Find the High Frequency Gain

To find $K_{p}$, use $K_{p}=\lim_{s\rightarrow\infty}H_{p}^{-1}(s)W_{p}(s)$, which for diagonal Hermite forms is the same as $K_{p}=E[W_{p}(s)]$.

## Select the Reference Model

Pick the reference model transfer matrix $W_{m}(s)$ as
\begin{equation*}
  W_{m}(s)=H_{p}(s)Q_{m}(s)
\end{equation*}
where $Q_{m}(s)$ is an asymptotically stable unimodular matrix.
For purposes of simplicity we can assume that $Q_{m}=\gamma I$, where $\gamma$ is picked so that the DC gain of the components of the diagonal Hermite form, and thus reference model, have unity DC gain.
With $H_{p}(s)$ in \eqref{eqn.adaptive.hermite_form} and setting $\gamma=a$ this gives
\begin{equation*}
  W_{m}(s)=
  \begin{bmatrix}
    \frac{a}{(s+a)} & 0 \\\\
    0 & \frac{a}{(s+a)}
  \end{bmatrix}
\end{equation*}

## Calculate an Upper Bound on the Observability Index

Calculate an upper bound $\nu$ on the observability index using the following from Reference [^narendra.stable.2005], p.406.
It says an upper bound $\nu$ on the observability index can be obtained by knowing an upper bound $n_{ij}$ on the order of the $ij^{\text{th}}$ scalar entry in $W_{p}(s)$.
\begin{equation*}
  \nu=\left[\frac{1}{m}\sum_{i,j}n_{ij}\right]
\end{equation*}
For this example, with $m=2$ and $n_{ij}$ determined by inspection $\nu$ is calculated as
\begin{equation*}
  \nu=\frac{1}{2}(1+2+2+1)=3
\end{equation*}

## Design Controller Filters

Using the upper bound on the observability index $\nu$, we design $\nu-1$ control input filters, and $\nu$ output filters as follows, where $r_{q}(s)$ is a Hurwitz, monic polynomial of degree $\nu-1$.
\begin{align*}
  \mbox{Control signal filter} \qquad & \omega_{i}=\frac{s^{i-1}}{r_{q}(s)} \qquad i=1,\dots\nu-1 \\\\
  \mbox{Output filter} \qquad & \omega_{j}=\frac{s^{j-1}}{r_{q}(s)} \qquad j=\nu,\dots2\nu-1 \\\\
\end{align*}
Each filter has a scalar denominator $r_{q}(s)$, and there are $m$ components to each filter, and a total of $2\nu-1$ filter, so the total number of integrations (i.e. the number of controller states) to generate the $\omega$ signals is $m(2\nu-1)$.
There is a parameter matrix corresponding to each $\omega$ signal, giving $m^{2}(2\nu-1)$ parameters.
See the following image from [^narendra.stable.2005].
This block diagram provides a great way to see the structure of the control which otherwise might be a bit difficult given the construction and number of filters, and their corresponding adaptive elements.

<img src="/img/posts/adaptive-mimo-python/block.jpeg" width="800" />

With $\nu=$ this means there will be $2$ control input filters, and $3$ output filters.
The denominator $r_{q}(s)$ can be selected as

\begin{equation*}
r_{q}(s)=s^{2}+s+1
\end{equation*}



# Controller Summary

Reference Model

\begin{equation*}
  W_{m}(s)=
  \begin{bmatrix}
    \frac{a}{(s+a)} & 0 \\\\
    0 & \frac{a}{(s+a)}
  \end{bmatrix}
\end{equation*}

# Simulation Result

Repository here: [https://github.com/dpwiese/control-examples/tree/master/classical-mimo](https://github.com/dpwiese/control-examples/tree/master/classical-mimo)

<img src="/img/posts/adaptive-mimo-python/plot.png" width="700" />

[^narendra.stable.2005]: Narendra, K. S. and Annaswamy, A. M., Stable Adaptive Systems, Dover Books on Electrical Engineering Series, Dover, 2005, [http://books.google.com/books?id=CRJhmsAHCUcC](http://books.google.com/books?id=CRJhmsAHCUcC).

<script>
  const refs = document.getElementsByClassName("footnote-ref");
  const sups = document.querySelectorAll('*[id^="fnref:"]');

  Array.from(refs).forEach(function(ref) { ref.parentElement.parentElement.insertBefore(ref, ref.parentElement) });
  Array.from(sups).forEach(function(sup) { sup.remove() });
</script>
