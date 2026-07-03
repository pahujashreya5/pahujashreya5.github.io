##  Implementing Transport Noise and Pseudo-Transport Noise for Fluids In Houdini

# Introduction:

Transport Noise is used to create the effect of turbulence in fluids. Why is it different from other types of noise (Curl Noise, Flow Noise) used for fluids? Curl and Flow noises use external forces to create the movements that we see in a fluids. Transport Noise differs because it works directly with particle positions. It creates swirly turbulence by changing the positions of particles of the fluid itself. 

>  A transport type noise in a fluid dynamic model may be seen, loosely speaking, as a simplified description of small-medium space-scales of motion.
> ...in general, this perturbation destabilizes large scales producing smaller eddies.

Above quoted from [https://www.researchgate.net/publication/374437022_Effect_of_Transport_Noise_on_Kelvin-Helmholtz_Instability](https://www.researchgate.net/publication/374437022_Effect_of_Transport_Noise_on_Kelvin-Helmholtz_Instability)

## Pseudo-Transport Noise?

This is a type of transport noise in which the low frequency movements of the fluid are left as they are, and the high frequency movements are affected by transport noise. This leaves the overall flow and direction of the fluids intact, while creating more localized swirly motions. The effect is that overall disturbance of the fluid seems less compared to transport noise.


# Sources ♥️

1. [https://imstat.org/2024/02/15/youngstats-transport-noise-in-fluid-dynamics/](https://imstat.org/2024/02/15/youngstats-transport-noise-in-fluid-dynamics/)
2. [https://www.artivoxa.com/houdini-noise-vop-deep-dive-every-noise-type-and-when-to-use-it/](https://www.artivoxa.com/houdini-noise-vop-deep-dive-every-noise-type-and-when-to-use-it/)
