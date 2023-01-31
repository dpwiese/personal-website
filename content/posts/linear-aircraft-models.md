---
title: "Linear Aircraft Models"
date: 2023-01-28T19:25:59-05:00
draft: true
toc: false
images:
tags:
  - flight dynamics
  - control theory
  - python
keywords: [flight dynamics, control theory, python]
description: "This post presents some simple linear aircraft models and provides their implementation in Python for use with the Python Control Systems Library."
---

# Introduction

This post presents some simple linear aircraft models and provides their implementation in Python for use with the [Python Control Systems Library](http://python-control.org/).
Such models can be easily found across many references on aircraft flight mechanics.
However these models typically vary slightly depending on how they were derived and the simplifications made.
This post aims to provide a little bit more cohesion between these different models.

# Derivation from First Principles

The derivation of the models begins with nothing more than Newton's laws - how to describe the motion of a body acted upon by forces and moments.
In deriving these equations, some assumptions are commonly made for atmospheric flight vehicles - that the Earth is flat and nonrotating, and that the aircraft is a rigid body, thus neglecting the effects of rotating turbo machinery in the aircraft.
Ref. [^stevenslewis.flight.2015] presents these nonlinear equations in their matrix form in (eq. 1.7-18).
The result is twelve scalar nonlinear differential equations.
The state is the vehicle's position and orientation along with its linear and angular velocity, each in three dimensions.

## Simplification

These equations, while some simplifications have already been made, can be very reasonably further simplified for many aerospace applications.
The next steps in simplification involve expanding the matrix equation into scalar form to examine any quantities that can be neglected, and determining the functional dependence of the applied forces and moments (due to aerodynamic and propulsive forces).
Ref. [^stevenslewis.flight.2015] presents this expanded form well in Table 2.5-1.
A flight condition must be selected around which to linearize the equations, and the validity of the linearity assumption can be verified.
Finally, as a result of these simplifications, the linear equations of motion can be decoupled into their longitudinal and lateral-direction motion.

The resulting linear equations will take the following matrix form

\begin{equation*}
  E\dot{x} = \tilde{A}x + \tilde{B}u
\end{equation*}

where by left-multiplying both sides by $E^{-1}$ it can be written

\begin{equation*}
  \dot{x} = Ax + Bu
\end{equation*}

## Common Simplifications

The process of linearizing and simplifying the equations of motion is not presented here.
Rather, some of the common simplifications are stated, as they do vary somewhat between references.
In the equations that follow, some of the "small" terms are retained in some instances and in others they are dropped.
By listing at least some of the common simplifications here, we aim to at least make obvious which terms *should not* be dropped.

First, assuming symmetry with respect to the $x-z$ plane results in only the $J_{xz}$ cross-product of inertia being nonzero [^stevenslewis.flight.2015] (pg. 38).
However, the $J_{xz}$ is still generally very much smaller than $J_{xx}$, $J_{yy}$ and $J_{zz}$ and can often be neglected [^cook.flight.2012] (pg. 72).

We then consider the functional dependency of the forces and moments on the other variables, as this affects the presence of various partial derivative terms that result when linearizing.
For example, we can assume the aircraft the force $X$ is a function of the following

\begin{equation*}
X(u,w,q,\delta_{e},\delta_{\text{th}})
\end{equation*}

The dependence of $X$ on $u$, $w$, and $\delta_{\text{th}}$ are probably the most apparent ones, indicating essentially that the longitudinal force depends on airspeed, angle of attack, and thrust.
That $X$ also depends sufficiently on the pitch rate $q$ and the elevator deflection angle are less obvious, but perhaps still appear reasonable.
During linearization, partial derivative terms of $X$ will arise.

\begin{equation*}
  X_{i}=\frac{1}{m}\frac{\partial{}X}{\partial{}i}
\end{equation*}

Given the assumed functional dependence of $X$, this would result in the following terms.

\begin{equation*}
  X_{u}
  X_{w}
  X_{q}
  X_{\delta_{e}}
  X_{\delta_{\text{th}}}
\end{equation*}

By studying aircraft aerodynamic data it is found that many of the stability derivatives under most flight conditions can be neglected.
Some such stability derivatives which are commonly neglected are the following, some of which are indicated in Ref. [^nelson.flight.1998] (pg. 149) and Ref. [^mclean.flight.1990] (pg. 33).

\begin{equation*}
  X_{\dot{u}}
  X_{q}
  X_{\dot{w}}
  X_{\delta_{e}}
  Z_{q}
  Z_{\dot{u}}
  Z_{\dot{w}}
  M_{\dot{u}}
  Z_{\dot{\delta_{e}}}
  M_{\dot{\delta_{e}}}
\end{equation*}

This neglecting of stability derivatives is another statement that there is not a strong functional dependence of the forces or moments on particular state variables or inputs and their derivatives.

Finally, when linearizing the equations of motion, it is quite often the case that the equilibrium flight condition is steady, straight and level flight at cruise.
This implies, among other things, that the pitch angle, angle of attack, and flight path angle are small.

# Longitudinal

With some background presented on how, starting from Newton's laws, simplified linear aircraft equations of motion can be derived, some simple models are presented.

## Body-Fixed Axes System

The following, similar to that in Ref. [^cook.flight.2012] Eq. 4.65, is a standard 4-state longitudinal aircraft model.
For the purposes of implementation, this model as suitable as-is.
Values for a particular aircraft can be substituted into this form, and the inversion of $E$ can be performed numerically to achieve a model in standard state-space form.

\begin{equation}
  \label{eqn.exdotform.longitudinal}
  \begin{bmatrix}
    1 & 0 & 0 & 0 \\\\
    0 & 1-Z_{\dot{w}} & 0 & 0 \\\\
    0 & -M_{\dot{w}} & 1 & 0 \\\\
    0 & 0 & 0 & 1
  \end{bmatrix}
  \begin{bmatrix}
    \dot{u} \\\\
    \dot{w} \\\\
    \dot{q} \\\\
    \dot{\theta}
  \end{bmatrix}=
  \begin{bmatrix}
    X_{u} & X_{w} & X_{q}-w_{\text{eq}} & -g\cos(\theta_{\text{eq}}) \\\\
    Z_{u} & Z_{w} & Z_{q}+u_{\text{eq}} & -g\sin(\theta_{\text{eq}}) \\\\
    M_{u} &M_{w} & M_{q} & 0 \\\\
    0 & 0 & 1 & 0
  \end{bmatrix}
  \begin{bmatrix}
     u \\\\
     w \\\\
     q \\\\
    \theta
  \end{bmatrix}+
  \begin{bmatrix}
    X_{\delta_{\text{th}}} & X_{\delta_{e}} \\\\
    Z_{\delta_{\text{th}}} & Z_{\delta_{e}} \\\\
    M_{\delta_{\text{th}}} & M_{\delta_{e}} \\\\
    0 & 0
  \end{bmatrix}
  \begin{bmatrix}
    \delta_{\text{th}} \\\\
    \delta_{e}
  \end{bmatrix}
\end{equation}

### Analytical Simplification

For the purposes of comparing this model and its various terms to other models, it will be put in standard state-space form analytically.
$E^{-1}$ is given by the following.

\begin{equation*}
E^{-1}=
\begin{bmatrix}
  1 & 0 & 0 & 0 \\\\
  0 & \frac{1}{1-Z_{\dot{w}}} & 0 & 0 \\\\
  0 & \frac{M_{\dot{w}}}{1-Z_{\dot{w}}} & 1 & 0 \\\\
  0 & 0 & 0 & 1
  \end{bmatrix}
\end{equation*}

Performing the left multiplication by this inverse and assuming $Z_{\dot{w}}=0$ and $Z_{q}M_{\dot{w}}=0$ and $\sin(\theta_{\text{eq}})M_{\dot{w}}=0$ and $w_{\text{eq}}=0$ gives

\begin{equation*}
  \begin{bmatrix}
    \dot{u} \\\\
    \dot{w} \\\\
    \dot{q} \\\\
    \dot{\theta}
  \end{bmatrix}=
  \begin{bmatrix}
    X_{u} & X_{w} & X_{q}-w_{\text{eq}} & -g\cos(\theta_{\text{eq}}) \\\\
    Z_{u} & Z_{w} & Z_{q}+u_{\text{eq}} & -g\sin(\theta_{\text{eq}}) \\\\
    M_{u}+M_{\dot{w}}Z_{u} & M_{w}+M_{\dot{w}}Z_{w} & M_{q}+M_{\dot{w}}u_{\text{eq}} & 0 \\\\
    0 & 0 & 1 & 0
  \end{bmatrix}
  \begin{bmatrix}
     u \\\\
     w \\\\
     q \\\\
    \theta
  \end{bmatrix}+
  \begin{bmatrix}
    X_{\delta_{\text{th}}} & X_{\delta_{e}} \\\\
    Z_{\delta_{\text{th}}} & Z_{\delta_{e}} \\\\
    M_{\delta_{\text{th}}}+M_{\dot{w}}Z_{\delta_{\text{th}}} & M_{\delta_{e}}+M_{\dot{w}}Z_{\delta_{e}} \\\\
    0 & 0
  \end{bmatrix}
  \begin{bmatrix}
    \delta_{\text{th}} \\\\
    \delta_{e}
  \end{bmatrix}
\end{equation*}

When additional assumptions are made, such as $\theta_{\text{eq}}=0$ and $w_{\text{eq}}=0$ and $X_{q}=0$ and $Z_{q}=0$ gives the following form, as in Ref. [^nelson.flight.1998] Eq. 4.51.

\begin{equation}
  \label{eqn.ss.longitudinal.nelson}
  \begin{bmatrix}
    \dot{u} \\\\
    \dot{w} \\\\
    \dot{q} \\\\
    \dot{\theta}
  \end{bmatrix}=
  \begin{bmatrix}
    X_{u} & X_{w} & 0 & -g \\\\
    Z_{u} & Z_{w} & u_{\text{eq}} & 0 \\\\
    M_{u}+M_{\dot{w}}Z_{u} & M_{w}+M_{\dot{w}}Z_{w} & M_{q}+M_{\dot{w}}u_{\text{eq}} & 0 \\\\
    0 & 0 & 1 & 0
  \end{bmatrix}
  \begin{bmatrix}
     u \\\\
     w \\\\
     q \\\\
    \theta
  \end{bmatrix}+
  \begin{bmatrix}
    X_{\delta_{\text{th}}} & X_{\delta_{e}} \\\\
    Z_{\delta_{\text{th}}} & Z_{\delta_{e}} \\\\
    M_{\delta_{\text{th}}}+M_{\dot{w}}Z_{\delta_{\text{th}}} & M_{\delta_{e}}+M_{\dot{w}}Z_{\delta_{e}} \\\\
    0 & 0
  \end{bmatrix}
  \begin{bmatrix}
    \delta_{\text{th}} \\\\
    \delta_{e}
  \end{bmatrix}
\end{equation}

### Incorporation of Additional State Variables

Equations \eqref{eqn.exdotform.longitudinal} and \eqref{eqn.ss.longitudinal.nelson} are typical of linear longitudinal aircraft models.
The four scalar state variables which describe the longitudinal dynamics of the aircraft are retained from the original twelve, providing a significantly simplified model.
However, it may be the case that we wish to include altitude as a state in the model.
Linearizing the altitude equation gives the following

\begin{equation*}
  \dot{h}=u_{\text{eq}}\theta-w
\end{equation*}

Which can then be included in Equations \eqref{eqn.exdotform.longitudinal} as

\begin{equation*}
  \begin{bmatrix}
    1 & 0 & 0 & 0 & 0 \\\\
    0 & 1-Z_{\dot{w}} & 0 & 0 & 0 \\\\
    0 & -M_{\dot{w}} & 1 & 0 & 0 \\\\
    0 & 0 & 0 & 1 & 0 \\\\
    0 & 0 & 0 & 0 & 1
  \end{bmatrix}
  \begin{bmatrix}
    \dot{u} \\\\
    \dot{w} \\\\
    \dot{q} \\\\
    \dot{\theta} \\\\
    \dot{h}
  \end{bmatrix}=
  \begin{bmatrix}
    X_{u} & X_{w} & X_{q}-w_{\text{eq}} & -g\cos(\theta_{\text{eq}}) & 0 \\\\
    Z_{u} & Z_{w} & Z_{q}+u_{\text{eq}} & -g\sin(\theta_{\text{eq}}) & 0 \\\\
    M_{u} &M_{w} & M_{q} & 0 & 0 \\\\
    0 & 0 & 1 & 0 & 0 \\\\
    0 & -1 & 0 & u_{\text{eq}} & 0
  \end{bmatrix}
  \begin{bmatrix}
     u \\\\
     w \\\\
     q \\\\
    \theta \\\\
    h
  \end{bmatrix}+
  \begin{bmatrix}
    X_{\delta_{\text{th}}} & X_{\delta_{e}} \\\\
    Z_{\delta_{\text{th}}} & Z_{\delta_{e}} \\\\
    M_{\delta_{\text{th}}} & M_{\delta_{e}} \\\\
    0 & 0 \\\\
    0 & 0
  \end{bmatrix}
  \begin{bmatrix}
    \delta_{\text{th}} \\\\
    \delta_{e}
  \end{bmatrix}
\end{equation*}

and similarly in \eqref{eqn.ss.longitudinal.nelson} as

\begin{equation*}
  \begin{bmatrix}
    \dot{u} \\\\
    \dot{w} \\\\
    \dot{q} \\\\
    \dot{\theta} \\\\
    \dot{h}
  \end{bmatrix}=
  \begin{bmatrix}
    X_{u} & X_{w} & 0 & -g & 0 \\\\
    Z_{u} & Z_{w} & u_{\text{eq}} & 0 & 0 \\\\
    M_{u}+M_{\dot{w}}Z_{u} & M_{w}+M_{\dot{w}}Z_{w} & M_{q}+M_{\dot{w}}u_{\text{eq}} & 0 & 0 \\\\
    0 & 0 & 1 & 0 & 0 \\\\
    0 & -1 & 0 & u_{\text{eq}} & 0
  \end{bmatrix}
  \begin{bmatrix}
     u \\\\
     w \\\\
     q \\\\
    \theta \\\\
    h
  \end{bmatrix}+
  \begin{bmatrix}
    X_{\delta_{\text{th}}} & X_{\delta_{e}} \\\\
    Z_{\delta_{\text{th}}} & Z_{\delta_{e}} \\\\
    M_{\delta_{\text{th}}}+M_{\dot{w}}Z_{\delta_{\text{th}}} & M_{\delta_{e}}+M_{\dot{w}}Z_{\delta_{e}} \\\\
    0 & 0 \\\\
    0 & 0
  \end{bmatrix}
  \begin{bmatrix}
    \delta_{\text{th}} \\\\
    \delta_{e}
  \end{bmatrix}
\end{equation*}

### Short-Period Model

It is often the case that the four-state system in \eqref{eqn.ss.longitudinal.nelson} can be further separated into the short-period and phugoid modes.
This is often presented in the literature without much justification, only that such separation is reasonable for many flight vehicles.
In order to justify this separation for a particular flight vehicle, **modal analysis** is a helpful tool.

#### Modal Analysis

Modal analysis aims to determine which entries in a given eigenvector are small when the units of each state variable are not the same, so that the various modes may be decoupled.
Ref. [^durham.flight.2013] Ch. 9 has a helpful section describing this process.
Modal analysis allows \eqref{eqn.ss.longitudinal.nelson} to be decomposed into the short-period model below as in Ref. [^nelson.flight.1998] (Eq. 4-71).

\begin{equation*}
  \begin{bmatrix}
    \dot{w} \\\\
    \dot{q}
  \end{bmatrix}=
  \begin{bmatrix}
    Z_{w} & u_{\text{eq}} \\\\
    M_{w}+M_{\dot{w}}Z_{w} & M_{q}+M_{\dot{w}}u_{\text{eq}}
  \end{bmatrix}
  \begin{bmatrix}
     w \\\\
     q
  \end{bmatrix}+
  \begin{bmatrix}
    Z_{\delta_{e}} \\\\
    M_{\delta_{e}}+M_{\dot{w}}Z_{\delta_{e}}
  \end{bmatrix}
  \begin{bmatrix}
    \delta_{e}
  \end{bmatrix}
\end{equation*}

As described above, different terms can be retained or neglected resulting in

\begin{equation}
  \label{eqn.ss.shortperiod.body}
  \begin{bmatrix}
    \dot{w} \\\\
    \dot{q}
  \end{bmatrix}=
  \begin{bmatrix}
    Z_{w} & Z_{q}+u_{\text{eq}} \\\\
    M_{w} & M_{q}
  \end{bmatrix}
  \begin{bmatrix}
     w \\\\
     q
  \end{bmatrix}+
  \begin{bmatrix}
    Z_{\delta_{e}} \\\\
    M_{\delta_{e}}
  \end{bmatrix}
  \begin{bmatrix}
    \delta_{e}
  \end{bmatrix}
\end{equation}

## Stability Axes System

Ref. [^stengel.flight.2004] Eq. 4.1-111a on pg. 492 has good rotation from body-fixed reference frame to stability axis system and gives the small angle approximation in Eq. 4.1-111b.
Consider \eqref{eqn.ss.shortperiod.body} below.

\begin{equation*}
  \begin{split}
    \dot{w} &= Z_{w}w + (Z_{q}+u_{\text{eq}})q + Z_{\delta_{e}}\delta_{e} \\\\
    \dot{q} &= M_{w}w + M_{q}q + M_{\delta_{e}}\delta_{e}
  \end{split}
\end{equation*}

Using the following substitutions

\begin{equation*}
  u_{\text{eq}} = V_{\text{eq}} \qquad
  w = V_{\text{eq}}\alpha \qquad
  \dot{w} = V_{\text{eq}}\dot{\alpha} \qquad
  u = V_{T}
\end{equation*}

Gives

\begin{equation*}
  \dot{\alpha} = \frac{Z_{\alpha}}{V_{\text{eq}}}\alpha + (1+\frac{Z_{q}}{V_{\text{eq}}})q + \frac{Z_{\delta_{e}}}{V_{\text{eq}}}\delta_{e}
\end{equation*}

\begin{equation*}
  \dot{q} = M_{\alpha}\alpha + M_{q}q + M_{\delta_{e}}\delta_{e}
\end{equation*}

where

\begin{equation*}
  Z_{\alpha} = Z_{w}V_{\text{eq}} \qquad
  M_{\alpha} = M_{w}V_{\text{eq}}
\end{equation*}

Putting these into state-space form we get Ref. [^lavretsky.book.2012] Eq. 1.8

\begin{equation*}
  \begin{bmatrix}
    \dot{\alpha} \\\\
    \dot{q}
  \end{bmatrix}=
  \begin{bmatrix}
    \frac{Z_{\alpha}}{V_{\text{eq}}} & 1+\frac{Z_{q}}{V_{\text{eq}}} \\\\
    M_{\alpha} & M_{q}
  \end{bmatrix}
  \begin{bmatrix}
    \alpha \\\\
    q
  \end{bmatrix}+
  \begin{bmatrix}
    \frac{Z_{\delta_{e}}}{V_{\text{eq}}} \\\\
    M_{\delta_{e}}
  \end{bmatrix}
  \delta_{e}
\end{equation*}

Ref. [^nelson.flight.1998] Eq. 4.75 presents the short-period model as well, although neglecting and retaining a couple terms different than Ref. [^lavretsky.book.2012].

### Incorporation of Additional State Variables

The transformation of any of the higher order models (those including velocity, pitch angle, and altitude, for example) can also be converted to the stability axis system as in Ref. [^lavretsky.book.2012] Eq. 1.7 below.

\begin{equation*}
  \begin{bmatrix}
    \dot{V_{T}} \\\\
    \dot{\alpha} \\\\
    \dot{q} \\\\
    \dot{\theta}
  \end{bmatrix}=
  \begin{bmatrix}
    X_{V} & X_{\alpha} & 0 & -g\cos(\gamma_{\text{eq}}) \\\\
    \frac{Z_{V}}{V_{\text{eq}}} & \frac{Z_{\alpha}}{V_{\text{eq}}} & 1+\frac{Z_{q}}{V_{\text{eq}}} & \frac{-g\sin(\gamma_{\text{eq}})}{V_{\text{eq}}} \\\\
    M_{V} & M_{\alpha}+\frac{M_{\dot{\alpha}}Z_{\alpha}}{V_{\text{eq}}} & M_{q}+M_{\dot{\alpha}} & 0 \\\\
    0 & 0 & 1 & 0 \\\\
  \end{bmatrix}
  \begin{bmatrix}
    V_{T} \\\\
    \alpha \\\\
    q \\\\
    \theta
  \end{bmatrix}+
  \begin{bmatrix}
    X_{\delta_{\text{th}}}\cos(\alpha_{\text{eq}}) & X_{\delta_{e}} \\\\
    -X_{\delta_{\text{th}}}\sin(\alpha_{\text{eq}}) & \frac{Z_{\delta_{e}}}{V_{\text{eq}}} \\\\
    M_{\delta_{\text{th}}} & M_{\delta_{e}} \\\\
    0 & 0
  \end{bmatrix}
  \begin{bmatrix}
    \delta_{\text{th}} \\\\
    \delta_{e}
  \end{bmatrix}
\end{equation*}

This can be extended with altitude as in Ref. [^lavretsky.book.2012] Eq. 1.10

\begin{equation*}
\dot{h}=V_{\text{eq}}(\theta-\alpha)
\end{equation*}

Giving

\begin{equation*}
  \begin{bmatrix}
    \dot{V_{T}} \\\\
    \dot{\alpha} \\\\
    \dot{q} \\\\
    \dot{\theta} \\\\
    \dot{h}
  \end{bmatrix}=
  \begin{bmatrix}
    X_{V} & X_{\alpha} & 0 & -g\cos(\gamma_{\text{eq}}) & 0 \\\\
    \frac{Z_{V}}{V_{\text{eq}}} & \frac{Z_{\alpha}}{V_{\text{eq}}} & 1+\frac{Z_{q}}{V_{\text{eq}}} & \frac{-g\sin(\gamma_{\text{eq}})}{V_{\text{eq}}} & 0 \\\\
    M_{V} & M_{\alpha}+\frac{M_{\dot{\alpha}}Z_{\alpha}}{V_{\text{eq}}} & M_{q}+M_{\dot{\alpha}} & 0 & 0 \\\\
    0 & 0 & 1 & 0 & 0 \\\\
    0 & -V_{\text{eq}} & 0 & V_{\text{eq}} & 0
  \end{bmatrix}
  \begin{bmatrix}
    V_{T} \\\\
    \alpha \\\\
    q \\\\
    \theta \\\\
    h
  \end{bmatrix}+
  \begin{bmatrix}
    X_{\delta_{\text{th}}}\cos(\alpha_{\text{eq}}) & X_{\delta_{e}} \\\\
    -X_{\delta_{\text{th}}}\sin(\alpha_{\text{eq}}) & \frac{Z_{\delta_{e}}}{V_{\text{eq}}} \\\\
    M_{\delta_{\text{th}}} & M_{\delta_{e}} \\\\
    0 & 0 \\\\
    0 & 0
  \end{bmatrix}
  \begin{bmatrix}
  \delta_{\text{th}} \\\\
  \delta_{e}
  \end{bmatrix}
\end{equation*}

<!--
Vertical acceleration can also be included using the assumption $a_{z}\approx-\ddot{h}$ where $\ddot{h}$ is given by

\begin{equation*}
\begin{split}
  \ddot{h}&=-V_{\text{eq}}(\dot{\theta}-\dot{\alpha}) \\\\
  &=V_{\text{eq}}(\dot{\alpha}-q) \\\\
  &=Z_{V}V_{T}+Z_{\alpha}\alpha+(V_{\text{eq}}+Z_{q})q-g\sin(\gamma_{\text{eq}})\theta-V_{\text{eq}}X_{\delta_{\text{th}}}\sin(\alpha_{\text{eq}})\delta_{\text{th}}+Z_{\delta_{e}}\delta_{e}-V_{\text{eq}}q \\\\
  &=Z_{V}V_{T}+Z_{\alpha}\alpha+Z_{q}q-g\sin(\gamma_{\text{eq}})\theta-V_{\text{eq}}X_{\delta_{\text{th}}}\sin(\alpha_{\text{eq}})\delta_{\text{th}}+Z_{\delta_{e}}\delta_{e}
\end{split}
\end{equation*}
-->

# Lateral-Directional

## Body Axes System

\begin{equation*}
  \begin{bmatrix}
    1 & 0 & 0 & 0 \\\\
    0 & 1 & -\frac{J_{xz}}{J_{xx}} & 0 \\\\
    0 & -\frac{J_{xz}}{J_{zz}} & 1 & 0 \\\\
    0 & 0 & 0 & 1
  \end{bmatrix}
  \begin{bmatrix}
    \dot{v} \\\\
    \dot{p} \\\\
    \dot{r} \\\\
    \dot{\phi}
  \end{bmatrix}=
  \begin{bmatrix}
    Y_{v} & Y_{p} & Y_{r}-u_{\text{eq}} & -g\cos(\theta_{\text{eq}}) \\\\
    L_{v} & L_{p} & L_{r} & 0 \\\\
    N_{v} & N_{p} & N_{r} & 0 \\\\
    0 & 1 & 0 & 0
  \end{bmatrix}
  \begin{bmatrix}
    v \\\\
    p \\\\
    r \\\\
    \phi
  \end{bmatrix}+
  \begin{bmatrix}
    Y_{\delta_{a}} & Y_{\delta_{r}} \\\\
    L_{\delta_{a}} & L_{\delta_{r}} \\\\
    N_{\delta_{a}} & N_{\delta_{r}} \\\\
    0 & 0
  \end{bmatrix}
  \begin{bmatrix}
    \delta_{a} \\\\
    \delta_{r}
  \end{bmatrix}
\end{equation*}

See also McLean page 37, where he defines primed stability derivatives, and then makes the linear model as follows, as shown on page 49.
The primed notation just takes into account coupling.
Also use $\dot{\psi}=r$

## Stability Axes System

\begin{equation*}
  \begin{bmatrix}
    \dot{\beta} \\\\
    \dot{p} \\\\
    \dot{r} \\\\
    \dot{\phi} \\\\
    \dot{\psi}
  \end{bmatrix}=
  \begin{bmatrix}
    Y_{\beta} & 0 & -1 & \frac{g}{U_{\text{eq}}} & 0 \\\\
    L_{\beta} & L_{p} & L_{r} & 0 & 0\\\\
    N_{\beta} & N_{p} & N_{r} & 0 & 0 \\\\
    0 & 1 & 0 & 0 & 0 \\\\
    0 & 0 & 1 & 0 & 0
  \end{bmatrix}
  \begin{bmatrix}
    \beta \\\\
    p \\\\
    r \\\\
    \phi \\\\
    \psi
  \end{bmatrix}+
  \begin{bmatrix}
    0 & Y_{\delta_{r}} \\\\
    L_{\delta_{a}} & L_{\delta_{r}} \\\\
    N_{\delta_{a}} & N_{\delta_{r}} \\\\
    0 & 0 \\\\
    0 & 0
  \end{bmatrix}
  \begin{bmatrix}
    \delta_{a} \\\\
    \delta_{r}
  \end{bmatrix}
\end{equation*}

<!--
From Yechout [^yechout.flight.2003] pg. 291 has equations in stability axis system.

[^yechout.flight.2003]: Yechout, T. R., Introduction to Aircraft Flight Mechanics, AIAA, 2003, [https://books.google.com/books?id=a_c2V0zAFwcC](https://books.google.com/books?id=a_c2V0zAFwcC).
-->

[^mclean.flight.1990]: McLean, D., Automatic Flight Control Systems, Prentice Hall, 1990, [https://books.google.com/books?id=cJNTAAAAMAAJ](https://books.google.com/books?id=cJNTAAAAMAAJ).
[^nelson.flight.1998]: Nelson, R. C., Flight Stability and Automatic Control, 2nd Edition, McGraw-Hill Education, 1998 [https://books.google.com/books?id=Z4lTAAAAMAAJ](https://books.google.com/books?id=Z4lTAAAAMAAJ).
[^stengel.flight.2004]: Stengel, R. F., Flight Dynamics, Princeton University Press, 2004, [https://books.google.com/books?id=dWKYDwAAQBAJ](https://books.google.com/books?id=dWKYDwAAQBAJ).
[^lavretsky.book.2012]: Lavretsky, E. and Wise, K., Robust and Adaptive Control: With Aerospace Applications, Springer London, 2012 [https://books.google.com/books?id=cRefvQEACAAJ](https://books.google.com/books?id=cRefvQEACAAJ).
[^cook.flight.2012]: Cook, M. V., Flight Dynamics Principles: A Linear Systems Approach to Aircraft Stability and Control, Butterworth-Heinemann, 2012, [https://books.google.com/books?id=hgZDmoL4_DcC](https://books.google.com/books?id=hgZDmoL4_DcC)
[^durham.flight.2013]: Durham, W., Aircraft Flight Dynamics and Control, John Wiley & Sons, 2013, [https://books.google.com/books?id=dU4fAAAAQBAJ](https://books.google.com/books?id=dU4fAAAAQBAJ).
[^stevenslewis.flight.2015]: Stevens, B. L. and Lewis, F.L. and Johnson, E. N., Aircraft Control and Simulation: Dynamics, Controls Design, and Autonomous Systems, 3rd Edition, John Wiley & Sons, 2015, [https://books.google.com/books?id=lvhcCgAAQBAJ](https://books.google.com/books?id=lvhcCgAAQBAJ).

<script>
  const refs = document.getElementsByClassName("footnote-ref");
  const sups = document.querySelectorAll('*[id^="fnref:"]');

  Array.from(refs).forEach(function(ref) { ref.parentElement.parentElement.insertBefore(ref, ref.parentElement) });
  Array.from(sups).forEach(function(sup) { sup.remove() });
</script>
