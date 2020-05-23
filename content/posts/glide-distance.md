---
title: "Weight and Glide Distance of an Aicraft"
date: 2020-05-23
draft: true
toc: false
tags:
  - aviation
  - flying
keywords: [aviation, flying]
---

# Introduction

Flying is governed by the laws of physics, from aerodynamics to structural dynamics, propulsion, and more.
These laws, theories, and equations are critical to understand exactly how aircraft and flying works.
However *piloting* generally requires learning, memorizing, and being able to quickly and reliably apply some relatively simple rules of thumb that have been distilled from theory.
The safety of the pilot and their passengers depends on it.

A significant part being a safe pilot is knowing the correct response when encountering emergency scenarios, a common one being engine failure during flight.
In this scenario the procedures generally dictate that an attempt to restart the engine be made.
In a single engine aircraft, if an engine restart is not possible, the plane and its occupants will inevitably return to the ground.
The concern of the pilot then becomes how to execute an emergency landing as safely as possible.

One of the main considerations when executing an emergency landing is *finding a suitable landing spot within the maximum glide distance of the aircraft.*
While the wind, terrain, and many other factors are important considerations, it is critical to understand the behavior of the gliding aircraft to be able to safely pilot it to the ground.
The remainder of this post uses some simple equations from physics to give insight into how fast *and how far* a gliding aircraft will go as it glides.
**Specifically, I wanted to use these equations to help give some understanding to the perhaps counterintuitive fact that the glide distance of an aircraft is independent of its weight.**

# Maximum Glide Distance

Consider following chart from the emergency procedures section of the aircraft information manual
for the Cessna 172.

<img src="http://localhost:1313/img/posts/glide-distance/c172-glide-distance.jpeg" width="600"/>

This chart gives the maximum glide distance based on the height of the aircraft in feet above ground level, or AGL.
It is linear, and from this the pilot can see the slope: **this Cessna 172 can glide a maximum of 1.5 nm per 1000 feet AGL.**
This glide ratio is one of two critical numbers from this chart that the pilot should to commit to memory and be to recall immediately should an engine-out arise.
The second important number on this chart is best glide speed which tells the pilot **to glide at 68 knots indicated airspeed in order to achieve the maximum glide distance**.

So if the aircraft is at 6,000 feet AGL and the engine dies, the pilot can quickly and effortlessly calculate that they need to fly the aircraft at 68 KIAS and will glide up to 9 nm.
**Taking into account wind and other environmental factors this gives clear guidance to the pilot where to look for a suitable place to execute an emergency landing.**

Note that the glide distance and best glide speed *do not depend on the aircraft weight* even though many other performance characteristics, such as rate of climb, do.
It may be tempting to assume this is just an approximation for such a small aircraft with a narrow operating envelope, and that's partially true.

Let's check out information manual from slightly larger Cessna 182.
This is very similar to that for the 172, except that the glide ratio gives closer to 1.4 nm per thousand feet.

<img src="http://localhost:1313/img/posts/glide-distance/c182-glide-distance.jpeg" width="600"/>

More importantly though note now that the best glide speed *does* depend on weight even though the maximum glide distance *does not*.
And this dependence is quite large---the best glide speed increases 31% between 2100 lb and 3100 lb.
So what's going on?
Why is there not a corresponding change of 31% in glide distance as the weight goes up?
And does it not seem unexpected that the glide distance does not get worse as the aircraft gets heavier?

This is a question that maybe isn't so important to understand as the directive to the pilot does not change---calculate the maximum glide distance and fly at the best glide speed to a location within that radius.
But it is an interesting question, and the following derivation is very simple to help provide some insight into the answer.

# Derivation

Consider following free-body diagram of a glider (the aircraft with failed engine) from <a href="https://www.google.com/books/edition/Airplane_Aerodynamics_and_Performance/bSq-cEf0EWsC" target="_blank">Airplane Aerodynamics and Performance</a> by Jan Roskam.

<img src="http://localhost:1313/img/posts/glide-distance/roskam-free-body.jpeg" width="600"/>

_Note: $RC$ is rate of climb, $RD$ is rate of descent, and $RC=-RD$.
The angle of attack is given by $\alpha$, the pitch angle by $\theta$, and the flight path angle by $\gamma$.
The lift, drag, and weight are denoted $L$, $D$, $W$, respectively.
$X$ and $Z$ are the aircraft axes, with $b$ representing the body-axes, and $s$ the stability axes.
Most of these won't be required in what follows._

As this diagram represents a gliding aircraft thrust is zero.
The only forces acting on the aircraft are lift, drag, and weight, and as the aircraft glides to the ground these forces must balance.
This balance in the *stability-axis* system is given by

\begin{align}
  \label{eqn.glidedistance.L}
  L&=W\cos(\gamma) \\\\
  \label{eqn.glidedistance.D}
  D&=W\sin(\gamma)
\end{align}

With the aircraft at altitude above ground given by $h$, the relationship between the flight path angle glide distance $d$ is given by

\begin{equation}
  \label{eqn.glidedistance.tangamma}
  \tan(\gamma)=\frac{h}{d}
\end{equation}

Using \eqref{eqn.glidedistance.L} and \eqref{eqn.glidedistance.D} the equation in \eqref{eqn.glidedistance.tangamma} can be rearranged to obtain the following expression for the glide distance

\begin{equation}
  \label{eqn.glidedistance.d}
  d=h\frac{L}{D}
\end{equation}

Lift and drag are related to their corresponding coefficients as follows, where $\rho$ is air density and $S$ is the planform area of the wing.

\begin{align}
  \label{eqn.glidedistance.CL}
  C_{L}&=\frac{L}{\frac{1}{2}\rho V^{2}S} \\\\
  \label{eqn.glidedistance.CD}
  C_{D}&=\frac{D}{\frac{1}{2}\rho V^{2}S}
\end{align}

Using \eqref{eqn.glidedistance.CL} and \eqref{eqn.glidedistance.CD} and **maximizing the glide distance** in \eqref{eqn.glidedistance.d} gives

\begin{equation}
  \label{eqn.glidedistance.dmax}
  d_{\max}=h\left(\frac{C_{L}}{C_{D}}\right)_{\max}
\end{equation}

So now it can be easily see from \eqref{eqn.glidedistance.dmax} that the glide distance is not dependent on $W$ at all.
Rather it depends only on the aerodynamic properties of the aircraft---lift and drag specifically.
For a small sub-sonic general aviation aircraft like a Cessna, operating only at low altitudes in Earth's atmosphere, $C_{L}$ and $C_{D}$ depend only on angle of attack $\alpha$.
The relationship between them and thus the maximization of the lift versus the drag coefficient can be found from the aircraft's <a href="https://en.wikipedia.org/wiki/Drag_polar" target="_blank">Drag Polar</a>.
The drag polar is not provided in the Aircraft Information Manual, but it is something very well characterised by the aircraft manufactuer, and used to presribe $d_{\max}$.

## Best Glide Speed

The maximization of $C_{L}$ with respect to $C_{D}$ fixes both values of these coefficients, and by looking at \eqref{eqn.glidedistance.CL} the remaining unknowns in the best glide scenario are $L$ and $V$.
As lift depends on the weight, it is thus expected that the best glide speed depends on weight.
The following will show this relationship.

In the *earth-axis* system the forces on the aicraft can be described by the balanced as follows.

\begin{align*}
  W &= L\cos(\gamma)+D\sin(\gamma) \\\\
  D\cos(\gamma) &= L\sin(\gamma)
\end{align*}

Using some trig identities or use the Pythagorean theorem given the relationship in the diagram above, the following equation relating weight, lift, and drag can be found.

\begin{equation*}
  W^{2} = L^{2}+D^{2}
\end{equation*}

Using the nondimensional coefficients \eqref{eqn.glidedistance.CL} and \eqref{eqn.glidedistance.CD} the weight can be expressed as

\begin{equation*}
  W = \frac{1}{2}\rho V^{2}S\sqrt{C_{L}^{2}+C_{D}^{2}}
\end{equation*}

From this expression the velocity $V$ can be solved for as

\begin{equation}
  \label{eqn.glidedistance.V}
  V = \sqrt{\frac{2W}{\rho S\sqrt{C_{L}^{2}+C_{D}^{2}}}}
\end{equation}

From \eqref{eqn.glidedistance.V} it is clear to see that the glide speed varies with the square root of the weight $W$.
