---
title: "Adaptive PI Control with Python"
date: 2020-05-08T23:08:36-04:00
draft: true
toc: false
images:
tags: 
  - adaptive learning
  - control theory
  - python
---

# Introduction

During graduate school I spent years studying, learning and researching dynamic systems and adaptive learning and control theory.
The majority of my work was theoretical - deriving new algorithms to learn about and control unique systems in new ways and prove that these algorithms would work.
Theory was applied to determine how well such algorithms would work and understand the situations when they would fail.
However, it was always interesting and useful to simulate the systems to actually see how they responded over time and gain additional insight.
Almost every simulation was done in <a href="https://www.mathworks.com/products/matlab.html" target="_blank">MATLAB</a>/<a href="https://www.mathworks.com/products/simulink.html" target="_blank">Simulink</a>.
Overall, I had a positive experience using these tools with expensive licenses provided to me for free during my tenure as a graduate student.
Now several years out of school I thought it'd be nice to dig up some of the more interesting academic/classroom examples and make them available.
However, I didn't want to pay for even a discounted license and the various addons that would cost several hundreds of dollars.
Additionally, I didn't want to impose the same restriction on someone else who might be interested in running the examples themselves.

After a bit of searching around I found the <a href="http://python-control.org/" target="_blank">Python Control Systems Library</a> and was able to quickly implement a simple adaptive control example.
I've been long meaning to write a post (or series of posts) introducing some interesting bits of system dynamics and control to the non-control engineer.
This is not that post.
I'm not going to attempt to introduce adaptive control or provide an accessible walkthrough of stability theory.
This post will simply introduce a simple academic control problem, quickly work through the solution, and present a short code to simulate the system with the <a href="http://python-control.org/" target="_blank">Python Control Systems Library</a>.

# Speed Control of a DC Motor

Consider the following equation desribing the angular velocity $x$ of a DC motor, where the input $u$ is the motor torque.
This model assumes motor torque is directly proportional to voltage and the fast electrical dynamics can be neglected.

\begin{equation}
  \label{eqn.adaptive.adaptivepi.gp}
  J\dot{x}+Bx=u
\end{equation}

Note that because this is a motor, the rotational inertia $J>0$ and viscous damping $B>0$, giving the plant a stable pole at $−B/J$ and no zeros.
This is a very simple system to consider for this control problem: first order and stable.
<b>The control goal is: have $x\rightarrow x_{d}$ when the parameters $J$ and $B$ are unknown</b>, where $x_{d}$ is the desired angular velocity.
The controller will be connected with the plant as shown by the following block diagram

<img src="http://localhost:1313/img/posts/adaptive-pi-python/block1.png" width="500" />

In the next section the control architecture is proposed.

# Adaptive PI Control

The first step in the design of the adaptive PI controller is to first design the nominal PI controller: the PI controller that would designed if $J$ and $B$ were known.
This step is called the algebraic part.
An adaptive version of this nominal controller is then created by replacing the parameter values within the controller with parameter estimates.
This step is called the analytic part.
This procedure is called the <i>certainty equivalence principle</i>: develop solution when parameters are known, then replace parameters by estimates.
As these parameter estimates are learned adaptively in real time, stable update laws that govern their evolution must be determined.

## Algebraic Solution

Following control architecture proposed

<img src="http://localhost:1313/img/posts/adaptive-pi-python/block2.png" width="600" />

A standard PI controller is written in transfer function form as

\begin{equation*}
  G_{c}(s)=\frac{u}{e_{r}}=k_{p}+\frac{k_{i}}{s}
\end{equation*}

where the control gains $k_{p}>0$ and $k_{i}>0$.
From this transfer function representation, we can see that this controller has a pole at the origin and a zero in the LHP at $-k_{i}/k_{p}$.
The reference error is given by

\begin{equation*}
e_{r}=r-x
\end{equation*}

A precompensator is proposed to suitably generate $r$ from $x_{d}$.
This is done by determining the closed-loop transfer function from $r$ to $x$ and defining the precompensator as the inverse.
Thus, with the precompensator the transfer function between $x_{d}$ and $x$ will be unity.

<img src="http://localhost:1313/img/posts/adaptive-pi-python/block3.png" width="600" />

Evaluating $W_{cl}(s)$ gives

\begin{equation*}
W_{cl}(s)=\frac{k_{p}s+k_{i}}{Js^{2}+k_{p}s+k_{i}}
\end{equation*}

From which the inverse can be found, and the reference input $r$ found in terms of the desired output $x_{d}$ such that the transfer function between $x_{d}$ and $x$ will be unity.

<img src="http://localhost:1313/img/posts/adaptive-pi-python/block4.png" width="700" />

The reference input is given by

\begin{equation*}
r=x_{d}+[JG_{c}^{-1}(s)]\dot{x}_{d}
\end{equation*}

In the block diagram

<img src="http://localhost:1313/img/posts/adaptive-pi-python/block5.png" width="700" />

Equivalently

<img src="http://localhost:1313/img/posts/adaptive-pi-python/block6.png" width="700" />

The following parameterization is proposed

\begin{align*}
  k_{p}&=K+J\lambda{} \\\\
  k_{i}&=K\lambda{}
\end{align*}

The control input can be expressed

\begin{equation}
  \label{eqn.adaptive.adaptivepi.alglaw1}
  u=J(\dot{x}_{d}+\lambda e)+Bx+K\left(e+\lambda\int edt\right)
\end{equation}

Defining the following errors

\begin{align}
  \label{eqn.adaptive.adaptivepi.e1}
  e_{1}&=\dot{x}_{d}+\lambda e \\\\
  \label{eqn.adaptive.adaptivepi.e2}
  e_{2}&=e+\lambda\int edt
\end{align}

Allowing the control law \eqref{eqn.adaptive.adaptivepi.alglaw1} to be written

\begin{equation}
  \label{eqn.adaptive.adaptivepi.alglaw2}
  u=Je_{1}+Bx+Ke_{2}
\end{equation}

In block diagram

<img src="http://localhost:1313/img/posts/adaptive-pi-python/block7.png" width="500" />

Again looking at the closed-loop transfer function using the PI controller with positive damping feedback

\begin{align*}
  W_{cl}(s)&=\frac{k_{p}s+k_{i}}{Js^{2}+k_{p}s+k_{i}}
\end{align*}

Using the parameterization from before, the characteristic polynomial is

\begin{equation*}
  Js^{2}+(K+J\lambda)s+K\lambda
\end{equation*}

which, using Routh-Hurwitz criterion is stable if all of the coefficients have the same sign.
So stable for

\begin{equation*}
  J>0
  \quad
  K>0
  \quad
  \lambda>0
\end{equation*}

and then

\begin{equation*}
  r=W_{c}^{-1}(s)x_{d}\quad\Rightarrow\quad x\rightarrow x_{d}
\end{equation*}

Completes algebraic part.
Now go to analytic part.

## Analytic Solution

Now in the nominal control law in \eqref{eqn.adaptive.adaptivepi.alglaw2} that was proposed as the algebraic solution the unknown parameters $J$ and $B$ are replaced with their estimates, $\hat{J~}$ and $\hat{B~}$, respectively.

\begin{equation}
  \label{eqn.adaptive.adaptivepi.bettercontrollaw}
  u=\hat{J~}(t)e_{1}(t)+\hat{B~}(t)x(t)+Ke_{2}(t)
\end{equation}

Block diagram

<img src="http://localhost:1313/img/posts/adaptive-pi-python/block8.png" width="500" />

Rearranging the plant dynamics from \eqref{eqn.adaptive.adaptivepi.gp} and substituting in the adaptive control law \eqref{eqn.adaptive.adaptivepi.bettercontrollaw} gives

\begin{equation*}
  \dot{x}=\frac{1}{J}\left(\tilde{B~}x+\hat{J~}e_{1}+Ke_{2}\right)
\end{equation*}

where damping coefficient parameter error is

\begin{equation*}
  \tilde{B~}=\hat{B~}-B
\end{equation*}

Time differentiating the error $e$ gives

\begin{equation*}
  \dot{e}=\dot{x}_{d}-\frac{1}{J}\left(\tilde{B~}x+\hat{J~}e_{1}+Ke_{2}\right)
\end{equation*}

Time differentiating $e_{2}$

\begin{equation*}
  \dot{e}_{2}=-\frac{K}{J}e_{2}-\frac{1}{J}\left(\tilde{B~}x+\tilde{J~}e_{1}\right)
\end{equation*}

## Stability

Since the plant is a DC motor we know $\text{sgn}(J)>0$.
Goal now is to drive $e_{2}\rightarrow0$.
We attempt to find stable update laws using the following proposed Lyapunov function

\begin{equation*}
  V=\frac{1}{2}\left(Je_{2}^{2}+\frac{1}{\gamma_{1}}\tilde{J~}^{2}+\frac{1}{\gamma_{2}}\tilde{B~}^{2}\right)
\end{equation*}

Time differentiating

\begin{equation*}
  \dot{V}=-Ke_{2}^{2}-\tilde{B~}xe_{2}-\tilde{J~}e_{1}e_{2}+\frac{1}{\gamma_{1}}\tilde{J~}\dot{\tilde{J~}}+\frac{1}{\gamma_{2}}\tilde{B~}\dot{\tilde{B~}}
\end{equation*}

And we can see that if we choose the following adaptive laws

\begin{align}
  \label{eqn.adaptive.adaptivepi.update1}
  \dot{\tilde{J~}}&=\gamma_{1}e_{2}e_{1} \\\\
  \label{eqn.adaptive.adaptivepi.update2}
  \dot{\tilde{B~}}&=\gamma_{2}e_{2}x
\end{align}

and substitute them into the $\dot{V}$ equation

\begin{equation*}
  \dot{V}=-Ke_{2}^{2}
\end{equation*}

Thus $\dot{V}\leq0$.
With $e_{2}$, $\tilde{J~}$, $\tilde{B~}\in\mathcal{L}_{\infty} \Rightarrow \dot{e}_{2} \in\mathcal{L}_{\infty}$.
And $e_{2}\in\mathcal{L}_{\infty} \Rightarrow e\in\mathcal{L}_{\infty}$ and $e\in\mathcal{L}_{\infty} \Rightarrow e_{1}\in\mathcal{L}_{\infty}$.
Therefore $\dot e_{2} \in\mathcal{L}_{\infty}$ and $e_{2}\in\mathcal{L}_{2} \Rightarrow \lim_{t\rightarrow\infty} e_{2}(t)=0 \Rightarrow \lim_{t\rightarrow\infty} e(t)=0$.

## System Summary


# Simulation with the Python Control Systems Library

With the simple DC motor example above, the adaptive PI controller proposed along with the update laws, and stability proved, the system can now be implemented in a simulation to show how it actually responds.
For this, the <a href="http://python-control.org/" target="_blank">Python Control Systems Library</a> is used.
The implementation of the above system will be presented without great detail.
The full code is available on Github: <a href="https://github.com/dpwiese/control-examples/tree/master/adaptive-pi" target="_blank">dpwiese/control-examples/adaptive-pi</a>.

The first step is defining the plant as a `LinearIOSystem` class, which, as the name suggests, represents a linear system.
The parameters of the state-space representation of the plant described by \eqref{eqn.adaptive.adaptivepi.gp} are used.
In creating the linear system the inputs, outputs, and state are described.
The system is given a name to use when refering to its inputs and outputs.

```python
SS_PLANT = control.StateSpace(-B/J, 1/J, 1, 0)

IO_DC_MOTOR = control.LinearIOSystem(
    SS_PLANT,
    inputs=('u'),
    outputs=('x'),
    states=('x'),
    name='plant'
)
```

Next, the controller is defined.
As the controller is nonlinear, the `NonlinearIOSystem` is used to describe it.
The usage is very similar to that of linear systems, but requires defining an `updfcn` describing the state dynamics and an `outfcn` to return the output.
These are `adaptive_pi_state` and `adaptive_pi_output` below.

```python
IO_ADAPTIVE_PI = control.NonlinearIOSystem(
    adaptive_pi_state,
    adaptive_pi_output,
    inputs=3,
    outputs=('u', 'j_hat', 'b_hat'),
    states=3,
    name='control',
    dt=0
)
```

The state dynamics implements the integral error state in \eqref{eqn.adaptive.adaptivepi.e2} and the update laws in \eqref{eqn.adaptive.adaptivepi.update1} and \eqref{eqn.adaptive.adaptivepi.update2}.
The output from this function are the equations which describe the evolution of the state via it's derivatives.

```python
def adaptive_pi_state(_t, x_state, u_input, _params):
    """Internal state of adpative PI controller"""

    # Controller inputs
    x_d = u_input[0]
    x_d_dot = u_input[1]
    x = u_input[2]

    # Controller state
    e_i = x_state[2]

    # Algebraic relationships
    e = x_d - x
    e_1 = x_d_dot + LAMBDA * e
    e_2 = e + LAMBDA * e_i

    # Dynamics
    d_j_hat = GAMMA_1 * e_2 * e_1
    d_b_hat = GAMMA_2 * e_2 * x
    e_i_dot = e

    return [d_j_hat, d_b_hat, e_i_dot]
```

Next, the output function is defined, giving the output from the nonlinear adaptive controller.
In this case, the controller output is scalar - just the control $u$ as given in \eqref{eqn.adaptive.adaptivepi.bettercontrollaw}.


```python
def adaptive_pi_output(_t, x_state, u_input, _params):
    """Algebraic output from adaptive PI controller"""

    # Controller inputs
    x_d = u_input[0]
    x_d_dot = u_input[1]
    x = u_input[2]

    # Controller state
    j_hat = x_state[0]
    b_hat = x_state[1]
    e_i = x_state[2]

    # Algebraic relationships
    e = x_d - x
    e_1 = x_d_dot + LAMBDA * e
    e_2 = e + LAMBDA * e_i

    # Control law
    u = j_hat * e_1 + b_hat * x + K * e_2

    return [u, x_state[0], x_state[1]]
```

With the plant and controller defined, they only need to be connected and then the closed-loop system can be simulated.
This is accomplished with the `InterconnectedSystem` class, that is explained well in the <a href="https://python-control.readthedocs.io/en/0.8.3/generated/control.iosys.InterconnectedSystem.html?highlight=InterconnectedSystem#control.iosys.InterconnectedSystem" target="_blank">docs</a>:

> The InterconnectedSystem class is used to represent an input/output system that consists of an interconnection between a set of subystems.
> The outputs of each subsystem can be summed together to to provide inputs to other subsystems.
> The overall system inputs and outputs can be any subset of subsystem inputs and outputs.

In this example these interconnections are simple: the output of the controller `control.u` connects to the input of the plant `plant.u`, and the output of the plant `plant.x` connects to (one of) the controller inputs `control.u[2]`.
The other two control inputs `control.u[0]` and `control.u[1]` are for the reference input and its derivative, $x_{d}$ and $\dot{x}_{d}$, respectively.

```python
IO_CLOSED = control.InterconnectedSystem(
    (IO_DC_MOTOR, IO_ADAPTIVE_PI),
    connections=(
        ('plant.u', 'control.u'),
        ('control.u[2]', 'plant.x')
    ),
    inplist=('control.u[0]', 'control.u[1]'),
    outlist=('plant.x', 'control.j_hat', 'control.b_hat'),
    dt=0
)
```

With the system configured, running the simulation is straightforward, only requiring the time steps `T` at which the input is defined, along with the input `U` at each time step, and the initial conditions `X0`.

```python
T_OUT, Y_OUT = control.input_output_response(IO_CLOSED, T, U, X0)
```

## Simulation Results

(TBD)
