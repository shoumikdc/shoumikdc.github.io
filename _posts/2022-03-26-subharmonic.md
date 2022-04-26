---
layout: post
title:  Driven Transmon Physics
date: 2022-03-26
description: Notes on superconducting qubit control via resonant and subharmonic drives
tags: physics qubits quantum

authors:
  - name: Shoumik Chowdhury
    affiliations:
      name: MIT

bibliography: 2022-03-26-subharmonic.bib

toc:
  - name: Introduction
  - name: Basics of Transmon Physics
  - name: Resonantly Driven Transmon
  - name: Subharmonic Driving Scheme
---

**This write-up is still work in progress. Expect breaking changes!**


## 0. Introduction
It is well-known that driving a nonlinear quantum system can lead to interesting _effective_ dynamical behavior that is absent in the undriven case. One area of physics and quantum information where this manifests quite beautifully is in the control of superconducting qubits; here it is possible to use a drive to engineer a desired effective interaction in the system. In this article, we will focus on reviewing some basic methods to describe the effects of a driving field on a superconducting transmon qubit. As a preliminary case study, we will investigate how to treat a resonantly driven transmon. Furthermore, we will extend the analysis to rederive a newer proposal for fast superconducting qubit control via subharmonic drives (cf. talk [Q38.8](https://meetings.aps.org/Meeting/MAR22/Session/Q38.8) at APS March Meeting 2022).

## 1. Basics of Transmon Physics
The _transmon_ is a widely adopted type of superconducting qubit, and is considered a key building block for scalable quantum processors constructed from superconducting circuits. The Hamiltonian of the transmon is given by:

$$ 
\begin{equation}
\hat{H}_{\rm tr} = 4E_C \big(\hat{n} - n_g\big)^2 - E_J\cos(\hat{\varphi})
\end{equation}
$$

where $$E_C$$ and $$E_J$$ are the charging and Josephson energies of the transmon respectively, and $$n_g$$ is the _gate offset charge_. The system is described by conjugate variables $$\hat{\varphi}$$ and $$\hat{n}$$ with $$[\hat{\varphi}, \hat{n}] = i$$, describing the superconducting phase and charge number. In the so-called _transmon regime_ when $$E_J/E_C \gg 1$$, the energy spectrum of the above Hamiltonian becomes exponentially insensitive to the offset charge, and thus we can drop $$n_g$$ from $$H_{\rm tr}$$ without too much loss of generality. Thus $$H_{\rm tr} \to 4E_C \hat{n}^2 - E_J\cos(\hat{\varphi})$$. For more details on the transmon, I would recommend reading the original [transmon paper](https://journals.aps.org/pra/abstract/10.1103/PhysRevA.76.042319), as well as Lev Bishop's [thesis](https://arxiv.org/abs/1007.3520). For beginners, an more digestible version with code can be found in the Qiskit [textbook](https://qiskit.org/textbook/ch-quantum-hardware/transmon-physics.html). 


#### 1.1. Transmon as a Weakly-Anharmonic Oscillator
In the transmon-regime, the Hamiltonian $$H_{\rm tr}$$ describes a weakly nonlinear oscillator. To see this, we can introduce mode operators $$\hat{a}, \hat{a}^\dagger$$ such that
$$
\begin{equation}
\hat{\varphi} = \varphi_{\rm zpf}(\hat{a} + \hat{a}^\dagger) \quad \mathrm{and} \quad 
\hat{n} = in_{\rm zpf}(\hat{a}^\dagger - \hat{a}).
\end{equation}
$$

Here $$\varphi_{\mathrm{zpf}} = (2E_C / E_J)^{1/4}$$ and $$n_{\rm zpf} = (E_J / 32 E_C)^{1/4}$$ are the _zero-point fluctuations_ in charge and phase, resulting in the commutation relation $$[\hat{a}, \hat{a}^\dagger] = 1$$. In analogy with the quantum harmonic oscillator ([QHO](https://en.wikipedia.org/wiki/Quantum_harmonic_oscillator)), the operators $$\hat{\varphi}$$ and $$\hat{n}$$ can be thought of as the generalized coordinate and momentum of the system. Now, in the transmon regime $$E_C \ll E_J$$, we have a small parameter $$\varphi_{\mathrm{zpf}} \ll 1$$; thus we can Taylor expand the "potential energy" term:

$$
\begin{equation}
U(\hat{\varphi}) = -E_J\cos(\hat{\varphi}) \approx -E_J \bigg[1 - \frac{\hat{\varphi}^2}{2!} + \frac{\hat{\varphi}^4}{4!} + \cdots\bigg]
\end{equation}
$$

The first term $$-E_J$$ is a constant, and thus we can drop it by scaling the energies suitably. If we then keep only upto the quartic term, we get the Hamiltonian of a _Duffing oscillator_: $$H_{\rm tr} \approx 4E_C \hat{n}^2 + E_J \hat{\varphi}^2 / 2 - E_J\hat{\varphi}^4 / 24$$. Finally, if we rewrite this using the mode operators and simplify, we get:

$$
\begin{equation}
\hat{H}_{\rm tr} = \omega_{\rm o} \hat{a}^\dagger \hat{a} + g_4 (\hat{a} + \hat{a}^\dagger)^4
\end{equation}
$$

Here, we have defined the 'oscillator' frequency $$\omega_{\rm o} = \sqrt{8 E_C E_J}$$ and 4th order nonlinearity $$g_4 = -E_C / 12$$ (also note $$\hbar = 1$$ throughout). In the transmon limit, we expect $$g_4 \ll \omega_{\rm o}$$, so that the Hamiltonian indeed describes a weakly nonlinear oscillator within excellent approximation. Still, for a more rigorous treatment, however, we can consider even higher order nonlinearities, e.g. $$g_6$$, from the Taylor expansion. 

Finally, since we wish to treat the transmon as an approximate two-level qubit, it will be helpful to define the $$0\to 1$$ transition frequency as well as the anharmonicity of $$\hat{H}_{\rm tr}$$.  Setting $$E_n = \langle n \vert \hat{H}_{\rm tr} \vert n \rangle$$ and $$E_{nm} = E_m - E_n$$, we get $$E_{01} = \hbar\omega_{01} = \omega_{\rm o} - \alpha$$ and $$\hbar\alpha = E_0 + E_2 - 2E_1 = -E_C$$. 


## 2. Resonantly Driven Transmon
In this section, we incorporate a drive into our Hamiltonian formulation of the transmon qubit. Here we focus on the case of a time-periodic _classical_ field $$f(t) = f(t + T)$$, and represent the drive coupling via the Hamiltonian $$\hat{H}_d(t) = \hat{Q} f(t)$$. For the typical case of a transmon that is capacitively coupled to the drive line, the operator $$\hat{Q}$$ is chosen to be the charge $$\hat{n}$$. In the Duffing model, however, this leads to the mild inconvenience of having to keep track of a minus sign in $$\hat{n} \propto i(\hat{a}^\dagger - \hat{a})$$. Therefore, without loss of generality, we can simplify the analysis here by choosing $$\hat{Q}$$ to be the _flux_ $$\hat{\varphi} \propto (\hat{a}+\hat{a}^\dagger)$$. As an aside, we remark that this choice of gauge bears semblance to the _classical_ driven nonlinear oscillator, where an external force $$F(t)$$ couples to the position $$x$$ via $$H_d = xF(t)$$. Now, with our assumptions in place, let us specify a drive field of the form $$f(t) = \Omega_d \cos(\omega_d t)$$ with period $$T = 2\pi / \omega_d$$. In this case, the full drive Hamiltonian reads

$$
\begin{equation}
\hat{H}(t) = \omega_{\rm o} \hat{a}^\dagger \hat{a} + g_4 (\hat{a} + \hat{a}^\dagger)^4 + \Omega_d (\hat{a}+\hat{a}^\dagger) \cos(\omega_d t).
\end{equation}
$$


#### 2.1. Rotating Wave Approximation and Rotating Frame Transformation
In the case of a near-resonant drive, we have $$\omega_d = \omega_{01} + \delta$$. That is, the drive is detuned from the $$0\to 1$$ transition frequency of the transmon by only a _small_ detuning $$\delta \ll \alpha$$. In this case, it is very well motivated to move to a frame of reference that rotates at the drive frequency $$\omega_d$$. This can be accomplished via the unitary $$\hat{U}(t) = \exp[-i\omega_d \hat{a}^\dagger\hat{a}]$$, and results in $$\hat{a} \to \hat{a}e^{-i\omega_d t}$$. Under this transformation, the resulting Hamiltonian is

$$
\hat{H}_{\rm rot} = \hat{U}\hat{H}\hat{U} + i[\partial_t \hat{U}] \hat{U}^\dagger = (\omega_{\rm o} - \omega_d)\hat{a}^\dagger \hat{a} + g_4 (\hat{a} e^{-i\omega_d t} + \hat{a}^\dagger e^{i\omega_d t})^4 + \Omega_d (\hat{a} e^{-i\omega_d t} + \hat{a}^\dagger e^{i\omega_d t}) \cos(\omega_d t),
$$

where the derivative term $$i[\partial_t \hat{U}] \hat{U}^\dagger$$ shifts the effective frequency to $$\omega_{\rm o} - \omega_d$$ in the rotating frame. While the equation above may appear unwieldy, it is at this stage that we can perform the so-called _rotating wave approximation_ (RWA). Expanding $$\hat{H}_{\rm rot}$$, we keep only the static terms under the assumption that all rotating terms time-average to zero near resonance. Under the RWA, we get

$$
\hat{H}_{\rm RWA}(t) = (\omega_{\rm o} - \omega_d)\hat{a}^\dagger \hat{a} + 12g_4 {\hat{a}{}^\dagger}^2\hat{a}^2 + \frac{\Omega_d}{2} \big[\hat{a}+\hat{a}^\dagger\big].
$$











## 3. Subharmonic Driving Scheme

#### 3.1. Displacement Transformations
Under the assumption of a weak nonlinearity $$g_4$$, it is often convenient to move into the displaced frame of the drive. The reason for this is as follows. In the limit $$g_4 \to 0$$, the Hamiltonian above describes a forced QHO. Thus, starting from a vacuum state $$|0\rangle$$, the action of $$\hat{H}(t)$$ is to produce a displaced state $$|\alpha_{\rm lin}(t)\rangle$$, in steady state, whose trajectory in phase space tracks the phase of the drive. As before, our intuition from the _classical_ driven harmonic oscillator carries over nicely to the quantum domain. Our objective here is to move into a frame of reference that follows the linear response trajectory of the oscillator, and thus 'subtract out' the uninteresting linear dynamics of the drive. Following the calculation below, we find $$\alpha_{\rm lin}(t) = \zeta e^{i\omega_d t} + \eta e^{-\omega_d t}$$, where $$\zeta, \eta$$ are proportional to $$\Omega_d$$. 

Now, applying the displacement operator $$\hat{D}[\alpha_{\rm lin}(t)] = \exp[\hat{a}^\dagger\alpha _{\rm lin} - \hat{a}\alpha _{\rm lin}^\ast]$$ to the Hamiltonian results in a _displaced frame_ Hamiltonian $$\hat{H}_{\rm disp} = \hat{D} \hat{H}\hat{D}^\dagger +i [\partial_t \hat{D}]\hat{D}^\dagger$$. After carrying out this time-dependent unitary transformation, we see that the derivative term fully cancels out the original drive term in $$\hat{H}(t)$$, giving:

$$
\begin{equation}
\hat{H}_{\rm disp}(t) = \omega_{\rm o} \hat{a}^\dagger \hat{a} + g_4 (\hat{a} + \hat{a}^\dagger + \Pi e^{i\omega_d t} + \Pi e^{-\omega_d t})^4
\end{equation}
$$




##### 3.1.1. Aside: Calculation of Linear Response
The calculation of the linear oscillator response can be done as follows. In the $$g_4 \to 0$$ limit, we have

$$
\dot{\hat{a}}(t) = i[\hat{H}(t), \hat{a}] \to -i\omega_{\rm o} \hat{a} + \frac{i\Omega_d}{2}\Big[e^{i\omega_d t} + e^{-i\omega_d t}\Big]
$$

To solve for the steady state response of the above equation, we employ a trick of adding in a canonical dissipative term $$-\gamma\hat{a}$$ to the problem. Since this term has a real coefficient, it leads to an exponential decay characteristic of dissipation. One might observe that this equation of motion can be obtained by replacing $$\omega_{\rm o}$$ by a complex frequency $$\widetilde{\omega} = \omega_{\rm o} - i\gamma$$ in $$\hat{H}(t)$$, so that $$\dot{\hat{a}}(t) = -i\widetilde{\omega} \hat{a} + i\Omega_d[e^{i\omega_d t} + e^{-i\omega_d t}] / 2$$. We thus have a first order differential equation of the form $$y'(t) = by + f(t)$$, and this has a solution $$y(t) = e^{-bt} \int^t e^{b \tau} f(\tau) d\tau + c_1 e^{-bt}$$ with integration constant $$c_1$$. Matching terms, we find

$$
\hat{a}(t) = e^{-i\widetilde{\omega} t} \frac{i\Omega_d}{2} \int_1^t  e^{i\widetilde{\omega} \tau} \Big[e^{i\omega_d \tau} + e^{-i\omega_d \tau}\Big] d\tau + c_1 e^{-i\widetilde{\omega} t} = \frac{\Omega_d / 2}{\widetilde{\omega} + \omega_d}e^{i\omega_d t} + \frac{\Omega_d / 2}{\widetilde{\omega} - \omega_d}e^{-i\omega_d t} + ic_1 e^{-i(\omega_{\rm o} - i\gamma) t}
$$

Finally, if we take the limit that $$\gamma \to 0$$, we see that the exponentially decaying $$c_1$$ boundary term vanishes, leaving a periodic steady state response. Of course, this would require an 'infinite' time to reach steady state, and in practice we must always content with finite dissipation. Nonetheless, to focus on the steady state behavior, we take the above limit $$\widetilde{\omega} \to \omega_{\rm o}$$ to get

$$
\hat{a}(t) = \frac{\Omega_d / 2}{\omega_{\rm o} + \omega_d}e^{i\omega_d t} + \frac{\Omega_d / 2}{\omega_{\rm o} - \omega_d}e^{-i\omega_d t} \equiv \zeta e^{i\omega_d t} + \eta e^{-\omega_d t}
$$

Since the coherent state amplitude is $$\alpha_{\rm lin}(t) = \langle \hat{a}(t)\rangle$$, we can capture the linear response via the equation above. 
