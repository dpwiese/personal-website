---
title: "Adaptive Control in the Presence of Input Constraints"
date: 2020-12-12T19:13:00-04:00
draft: false
toc: false
images: ["img/posts/adaptive-control-input-constraints/og-image.png"]
tags: 
  - adaptive learning
  - control theory
  - python
keywords: [adaptive learning, control theory, python]
description: ""
---

# Introduction

In two earlier posts, [Adaptive PI Control with Python](/posts/adaptive-pi-python/) and [MIMO Adaptive Control with Python](/posts/adaptive-mimo-python/), I presented some simple adaptive control examples with simulations generated using the [Python Control Systems Library](https://python-control.org/).
This post is a third.
Here I present a controller from a paper by S.P. Karason and A.M. Annaswamy, [Adaptive Control in the Presence of Input Constraints](https://doi.org/10.1109/9.333787) published in IEEE Transactions on Automatic Control.
This adaptive controller accommodates input constraints, and after presenting the controller the simulations from the paper are reproduced.

# Problem Statement

Consider the following plant, reference model, and control law, which we have seen many times before, but note the difference between the control law which produces the **desired control** $u_{c}$ and the <strong>control input</strong> $u$ to the plant.
As before $a_{m}<0$, and the sign of $b_{p}$ is known.

\begin{align}
  &\mbox{Plant:} &\hspace{0.5in} \dot x_{p}&=a_{p}x_{p}+b_{p}u\label{eqn.plant} \\\\
  &\mbox{Reference model:} &\hspace{0.5in} \dot x_{m}&=a_{m}x_{m}+b_{m}r\label{eqn.refmodel} \\\\
  &\mbox{Control law:} &\hspace{0.5in} u_{c}&=\theta{} x_{p}+kr\label{eqn.adaptive.saturation.uc}
\end{align}

Depending on the values of the adaptive parameters $\theta$ and $k$, the state $x_{p}$ and the reference input $r$, the desired control $u_{c}$ might be larger than what the actuator is actually capable of as given by $u$.
This is known as <strong>actuator saturation</strong>, which all actuators will have.

This limit is a _known constraint_, for example how much torque a motor can apply, or how much a control surface on an aircraft can deflect.
We model this limit as $u_{\text{max}}>0$ such that $|u|\leq u_{\text{max}}$, after which the actuator experiences saturation.
The control law with this saturation model can be expressed as follows.

\begin{align}
\label{eqn.adaptive.saturation.usat}
  &\mbox{Control input:} &\hspace{0.5in}
  u&=
  \begin{cases}
    u_{c} &\mbox{if } |u_{c}|\leq{} u_{\text{max}} \\\\
    u_{\text{max}}\text{sgn}(u_{c}) & \mbox{if } |u_{c}|>u_{\text{max}}
  \end{cases}
\end{align}

Before considering a control solution which accommodates the actuator saturation in \eqref{eqn.adaptive.saturation.usat}, _the case where the actuator has no saturation limits will be quickly reviewed first_.
That is set $u=u_{c}$ and design a controller, resulting in the standard adaptive system outlined below.
The tracking and parameter error are given by

\begin{equation}\label{eqn.adaptive.saturation.e}
e=x_{p}-x_{m}
\end{equation}

and parameter error as

\begin{equation}\label{eqn.adaptive.saturation.param_errors}
  \tilde{\theta~}=\theta-\theta^{*}\hspace{0.5in}\tilde{k~}=k-{k}^{ *}
\end{equation}

with matching conditions

\begin{equation}\label{eqn.adaptive.saturation.matching}
  a_{m}=a_{p}+b_{p}\theta^{*}
  \hspace{0.5in}
  b_{m}=b_{p}k^{*}
\end{equation}

and error dynamics

\begin{equation*}
  \dot{e}=a_{m}e+b_{p}\tilde{\theta}x_{p}+b_{p}\tilde{k}r
\end{equation*}

Using the following standard update laws in \eqref{standard.update.laws}, stability can easily be proved.

\begin{equation}
\label{standard.update.laws}
\mbox{Update laws:}\hspace{0.5in}\
\begin{array}{l}
\dot{\tilde{k~}}=-\text{sgn}(b_{p})er \\\\
\dot{\tilde{\theta~}}=-\text{sgn}(b_{p})ex_{p}
\end{array}
\end{equation}

But when these standard adaptive laws are used in the _presence_ of the actuator constraints in \eqref{eqn.adaptive.saturation.usat} the system is no longer globally stable and behaves undesirable.
Clearly an adaptive controller that accommodates the actuator limitations in \eqref{eqn.adaptive.saturation.usat} is desired.

# The Adaptive Controller

In what follows, modifications to the adaptive system presented above will be proposed so as to provide stability in the presence of the actuator saturation model in \eqref{eqn.adaptive.saturation.usat}.
The control input can be expressed in two parts: the total _desired control_ as computed by the control law, and a component _which subtracts off the portion of this control signal which the actuator is unable to produce_.
We call these components $u$ and $\Delta u$, respectively, where $\Delta u$ is the <strong>control deficit</strong>.
Because the desired control and control input are known quantities, so is the control deficit.

\begin{align}\label{eqn.adaptive.saturation.u}
  u=u_{c}+\Delta{}u
\end{align}

Substituting \eqref{eqn.adaptive.saturation.u} into the plant equation \eqref{eqn.plant} with control law \eqref{eqn.adaptive.saturation.uc} we get

\begin{equation}\label{eqnxpdot}
  \dot x_{p}=(a_{p}+b_{p}\theta)x_{p}+b_{p}kr+b_{p}\Delta u
\end{equation}

Differentiating the tracking error \eqref{eqn.adaptive.saturation.e} and using \eqref{eqnxpdot} and \eqref{eqn.refmodel} along withthe matching conditions \eqref{eqn.adaptive.saturation.matching} and parameter errors \eqref{eqn.adaptive.saturation.param_errors} gives the following error dynamics

\begin{align}\label{eqn.adaptive.saturation.edot}
  \dot{e}=a_{m}e+b_{p}\tilde{\theta~}x_{p}+b_{p}\tilde{k~}r+b_{p}\Delta{}u
\end{align}

_The portion of the control signal which the actuator is unable to accommodate, $\Delta u$, can be looked at like a disturbance to the system which we have seen before._
We would like to design the adaptive scheme to deal with the presence of this $\Delta u$.
Because our actuator has a known saturation limit, essentially what we want to do is measure the error, but only try to reduce the portion of the error which we actually have the actuator authority to do so.
That is, we define the <strong>controllable error</strong> $e_{u}$ as

\begin{equation}\label{eqn.controllable_error}
  e_{u}=e-e_{\Delta}
\end{equation}

We then obtain the controllable error $e_{u}$ by subtracting the <strong>deficit error</strong> $e_{\Delta}$ from the measurable tracking error.
The deficit error due to $\Delta u$ is determined from the following differential equation, where the input $\beta_{\Delta}$ is unknown.

\begin{equation}\label{eqn.adaptive.saturation.edeltadot}
  \dot e_{\Delta}=a_{m}e_{\Delta}+\beta_{\Delta}\Delta{}u
\end{equation}

When the control input is saturating, the controller cannot achieve any higher level of performance.
That is, the controller should not seek to minimize an error signal which it is unable to do because of the limitation of the actuator.
Instead we define the error $e_{u}$ which takes the state error (the actual error we would like to minimize) and subtracts off the portion of the error due to the "disturbance" $\Delta u$ which the controller input can do nothing about.
This is the error we would like to use to drive adaptation.
If the controller is causing the input to saturate, there is no sense using an error signal which the controller cannot reduce to drive adaptation.
and define the last parameter

\begin{equation}\label{eqn.adaptive.saturation.beta_tilde}
  \tilde{\beta~}=b_{p}-\beta_{\Delta}
\end{equation}

The dynamics describing this error are given by differentiating \eqref{eqn.controllable_error} and plugging in \eqref{eqn.adaptive.saturation.edot} and \eqref{eqn.adaptive.saturation.edeltadot} gives the following controllable error dynamics

\begin{equation}\label{eqn.adaptive.saturation.eudot}
  \dot e_{u}=a_{m}e_{u}+b_{p}\tilde{\theta~}x_{p}+b_{p}\tilde{k~}r+\tilde{\beta~}\Delta{}u
\end{equation}

This is the error we want to try to minimize.
Propose the following candidate Lyapunov function

\begin{equation*}
  V(e_{u},\tilde{\theta~},\tilde{k~},\tilde{\beta~})=
  \frac{1}{2}e_{u}{}^{2}+
  \frac{1}{2}|b_{p}|\gamma_{1}^{-1}\tilde{\theta~}^{2}+
  \frac{1}{2}|b_{p}|\gamma_{2}^{-1}\tilde{k~}^{2}+
  \frac{1}{2}\gamma_{3}^{-1}\tilde{\beta~}^{2}
\end{equation*}

Differentiating and using the following update laws

\begin{align*}
  \dot{\tilde{\theta~}}&=-\gamma_{1}\text{sgn}(b_{p})e_{u}x_{p} \\\\
  \dot{\tilde{k~}}&=-\gamma_{2}\text{sgn}(b_{p})e_{u}r \\\\
  \dot{\tilde{\beta~}}&=-\gamma_{3}e_{u}\Delta u
\end{align*}

gives

\begin{equation*}
  \dot{V}=a_{m}e_{u}{}^{2}\leq0
\end{equation*}


# Simulation Result

As usual, the code to create these simulations is available on Github: [dpwiese/control-examples/saturation-protection](https://github.com/dpwiese/control-examples/tree/master/saturation-protection).
None of the code is presented here as the implementation is quite straightforward.
Note that in the paper the values for the update law gains $\gamma_{1}$, $\gamma_{2}$, and $\gamma_{3}$ are not given, so the values used in the simulations below were selected to as to produce similar results to those in the paper.

The reference input in both plots below is $r(t)=\text{sin}(0.5t)$.
The first plot shows the result of the method presented above.
The controller, while saturating the actuator for the first six seconds or so, handles this saturation well - it does not increase the desired control too much beyond the actuators capabilities.
As the tracking error is reduced and the desired control is within the actuators limits, the controller quickly and smoothly brings the tracking error near zero by about ten seconds in and maintains a good response for the duration of the simulation.

<img src="/img/posts/adaptive-control-input-constraints/saturation_protection.png" width="600" />

The second plot illustrates what happens if the method presented above is _not used_.
Here we can see for the first six seconds or so, while the actuator is saturating, the desired control continues to increase, _winding up_ as the tracking error remains nonzero until around seven seconds.
As the tracking error decreases towards zero, the desired control abrubtly decreases, and for the duration of the simulation never manages to really track the reference.

<img src="/img/posts/adaptive-control-input-constraints/no_saturation_protection.png" width="600" />
