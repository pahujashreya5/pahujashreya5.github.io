##  Implementing Stochastic Turbulence for Fluids In Houdini

# Introduction

First off, it is important to clarify that stochastic turbulence is a **mathematical model**. It does not represent the true nature of fluid turbulence. To recreate real-life turbulence, Houdini uses Turbulence noise, which solves for every little swirl and eddy. _Stochastic_ turbulence, on the other hand, leaves uses randomness and probability. It estimates random fluid movement using statistics. But this is also used for the same overall visual effect (small swirls and eddies created due to interactions of the fluid with obstacles or boundary).
So, the unpredictable parts are treated as random noise. This is part of the reason why Houdini doesn't have this natively; it restricts creative choices. 
It is 'stochastic', which is basically saying that it the forces are random but can be analyzed. They cannot be predicted in a future time.
It is also known as transport noise. It adds random shifting directly to fluid particle positions.

>  A transport type noise in a fluid dynamic model may be seen, loosely speaking, as a simplified description of small-medium space-scales of motion.

> ...in general, this perturbation destabilizes large scales producing smaller eddies.

Above quoted from [https://www.researchgate.net/publication/374437022_Effect_of_Transport_Noise_on_Kelvin-Helmholtz_Instability](https://www.researchgate.net/publication/374437022_Effect_of_Transport_Noise_on_Kelvin-Helmholtz_Instability)

## Use Case?

The benefit of stochastic turbulence is that it saves a lot of time and computation power. Suppose there is a scene in which the fluid is not required to act in a particular manner but still requires realistic turbulence - stochastic noise could be the right choice for this use case.
But if, say, a small wave needs to hit an island at a particular frame, stochastic turbulence cannot be used for this amount of control over the movements of the fluid.

## Pseudo-Transport Noise?

This is a type of stochastic turbulence/transport noise in which the low frequency movements of the fluid are left as they are, and the high frequency movements are affected by transport noise. This leaves the overall flow and direction of the fluids intact, while creating more localized swirly motions. 

# Some Fluid Maths (Optional Read)


# How Is Noise Usually implemented Under The Hood In Houdini?

# Finally, Creating Transport Noise!

# Additional Reading
[On Fluid Noise Within Houdini - Curl, Flow and Turbulence](fluid-noise-houdini.md)

# Sources ♥️

1. [https://arxiv.org/pdf/2211.12918](https://arxiv.org/pdf/2211.12918)
2. [https://imstat.org/2024/02/15/youngstats-transport-noise-in-fluid-dynamics/](https://imstat.org/2024/02/15/youngstats-transport-noise-in-fluid-dynamics/)
3. [https://www.artivoxa.com/houdini-noise-vop-deep-dive-every-noise-type-and-when-to-use-it/](https://www.artivoxa.com/houdini-noise-vop-deep-dive-every-noise-type-and-when-to-use-it/)
4. [https://link.springer.com/article/10.1007/s40072-022-00249-7#Sec23](https://link.springer.com/article/10.1007/s40072-022-00249-7#Sec23)
5. [https://www.waseda.jp/fsci/mathphys/assets/uploads/2021/03/Chapter3_first_part_v1.pdf](https://www.waseda.jp/fsci/mathphys/assets/uploads/2021/03/Chapter3_first_part_v1.pdf)
