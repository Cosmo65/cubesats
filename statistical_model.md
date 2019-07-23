## Table of contents

1. [Link Budget](https://nbviewer.jupyter.org/github/kirlf/cubesats/blob/master/LinkBudget/LB.ipynb)
2. Statistical channel model survey
3. [Modulation and coding \(FEC\) survey](https://github.com/kirlf/cubesats/blob/master/fec.md)

# 2. Statistical channel model

## 2.1. SISO (Single Input Single Output) configuration

According to [\[1\]](https://www.csie.ntu.edu.tw/~b92b02053/printing/summer/Materials/channel%20model/CHN_A%20statistical%20model%20for%20land%20mobile%20satellite%20channels%20and%20itsapplication%20to%20nongeostationary.pdf) the most appropriate channel model for LEO satellites is the mixture of the Rician and lognormal independent fading processes with two ultimate conditions: light shadowing and strong shadowing \(tab.2.1\). Moreover, shadowing \(lognormal part\) is negligible on elevation angles larger than 60 degrees \(and smaller than 120 degree\) \(fig. 2.1\) [\[1\]](https://www.csie.ntu.edu.tw/~b92b02053/printing/summer/Materials/channel%20model/CHN_A%20statistical%20model%20for%20land%20mobile%20satellite%20channels%20and%20itsapplication%20to%20nongeostationary.pdf).

<img src="https://raw.githubusercontent.com/kirlf/cubesats/master/.gitbook/assets/image%20(6).png" alt="params" width="600" />

*Fig. 2.1. Model parameters, sigma, mu and K as a function of the elevation angle, a, in a mal tree-shadowed environment. \[1\]*

 Table 2.1. Statistical characteristics of the LEO channel

| Parameter | Light shadowing | Strong shadowing |
| :--- | :--- | :--- |
|  Rician factor \(linear scale\) | 4.0 | 0.6 |
|  Lognormal mean \(linear scale\) | 0.13 | -1.08 |
|  Lognormal variance \(linear scale\) | 1.0 | 2.5 |


> **NOTE**: 
>
>Possibly, it makes scense to model [Rician flat fading channel](https://nbviewer.jupyter.org/gist/kirlf/4328eb389b3ddc9a0c350eaed468f870) to estimate BER performance primaraly.

### 2.1.1. Generalized block scheme of the model

Corazza-Vatalaro model (the mixture of the Rician fast fading channel and Log-normal slow fading channel) should be considered for more precise consideration (fig. 2.2).

![Figure 2.2.  Circuit implementation of the C&V model with Jakes multipath Doppler shaping..](https://raw.githubusercontent.com/kirlf/cubesats/master/.gitbook/assets/cvm.png)
*Figure 2.2.  Circuit implementation of the C&V model with Jakes multipath Doppler shaping.\[2\]*

> Our suggestion is that implementation of this model in MatLab or Python is good student project. This allows to learn more about Doppler spread, [Doppler shifts](https://en.wikipedia.org/wiki/Doppler_effect#Satellite_communication), Log-normal fading, Rate conversion and Interpolation.

The following variables are used in the upper rail:

- <img src="https://tex.s2cms.ru/svg/%20G(0%2C%201)" alt=" G(0, 1)" />  is a random variable with a normal (Gaussian) distribution with zero mean and unit variance
- <img src="https://tex.s2cms.ru/svg/K" alt="K" />  is the Rician factor: the ratio between the power of the direct path of electromagnetic wave propagation and the total power of the other paths
-  <img src="https://tex.s2cms.ru/svg/%5Cphi_%7Binitial%7D" alt="\phi_{initial}" />  is the initial phase of the direct signal 
- <img src="https://tex.s2cms.ru/svg/%5CDelta%5Cphi" alt="\Delta\phi" />  is the Doppler shift of the direct signal
- <img src="https://tex.s2cms.ru/svg/%5Bn%5D" alt="[n]" /> - sample number

Lower rail belongs to slow fading ("log-normal series").
Variables:
- <img src="https://tex.s2cms.ru/svg/M" alt="M" /> - mean of the Gaussian process
- <img src="https://tex.s2cms.ru/svg/%5CSigma" alt="\Sigma" /> - variance of the Gaussian process

The multiplication of these rails makes complex envelop of the impulse responce of the considered channel.

### 2.1.2. Doppler shapping

The Butterworth filter based approach is proposed in \[2\] and \[3\] to simulate Doppler shaping.

The main characteristics \[3\]:
- filter order: 10
- filter type: passband
- cut-off frequencies: 30 and 300 Hz

Python implementation:

```python
from scipy import signal

b, a = signal.butter(10, [30 , 300], 'bandpass', analog=True)
w, h = signal.freqs(b, a)
```

### 2.1.3. Doppler shift

### 2.1.4. Slow fading

The lower rail of the fig. 2.2 can be reformulated as fig. 2.3.

![](https://raw.githubusercontent.com/kirlf/cubesats/master/.gitbook/assets/slow_fad_new.PNG)

*Fig. 2.3. Alternative low-pass filtering method for generating slower varying shadowed signals at the same rate as the multipath rail. \[2\]*

Where: 

- <img src="https://tex.s2cms.ru/svg/b%20%3D%20%5CSigma%5Csqrt%7B1%20-%20c%5E2%7D" alt="b = \Sigma\sqrt{1 - c^2}" />, 
- <img src="https://tex.s2cms.ru/svg/c%20%3D%20%5Cexp%7B%5Cleft(-%5Cfrac%7BvT_s%7D%7Bl_%7Bcorr%7D%7D%5Cright)%7D" alt="c = \exp{\left(-\frac{vT_s}{l_{corr}}\right)}" />,
- <img src="https://tex.s2cms.ru/svg/T_s" alt="T_s" /> is the sampling period, 
- <img src="https://tex.s2cms.ru/svg/v" alt="v" /> is the velocity of the mobile terminal, and 
- <img src="https://tex.s2cms.ru/svg/l_%7Bcorr%7D" alt="l_{corr}" /> is the correlation length (3-5 m \[4\]).

### 2.1.5. Markov chains based model 

Interesting research can be obtained in [\[5\]](https://www.db-thueringen.de/receive/dbt_mods_00026568), where both single-state and multi-state models are considered. 

<img alt="Markov" src="https://raw.githubusercontent.com/kirlf/cubesats/master/.gitbook/assets/SatMarkov.png" width="600"/>

*Figure 2.4. Semi-Markov model for two satellites \[5\].*

## 2.2. MIMO (Multiple Input Multiple Output) configuration

Moreover, the MIMO channel be considered \[6-9\]. Several frameworks are developed to model this kind of channels:
- [MIMOSA](https://artes.esa.int/projects/mimosa-characterisation-mimo-channel-mobile-satellite-systems)
- [SATCOM Spatial Geometrical Optimization](https://www.researchgate.net/profile/A_Knopp/publication/4323825_Optimum-capacity_MIMO_satellite_link_for_fixed_and_mobile_services/links/55701f2b08aeab77722897ad.pdf)

Additionaly, special MIMO techniques such as spatial diversity, spatial multiplexing or multi-user MIMO can be applied. Cooperative schemes should be taken into account (fig. 2.5).

<img src="https://upload.wikimedia.org/wikipedia/commons/a/af/Cooperative_Sat.png" width="600">

*Fig. 2.5. System model for the cooperative MIMO case over satellite communicactions.*

## References

\[1\] Giovanni E. Corazza and Francesco Vatalaro, A Statistical Model for Land Mobile Satellite Channels and Its Application to Nongeostationary Orbit Systems, Transac- tions on Vehicular Technology, vol. 43, no. 3. August 1994 

\[2\] Fontan, F. P., Mayo, A., Marote, D., Prieto‐Cerdeira, R., Mariño, P., Machado, F., & Riera, N. (2008). Review of generative models for the narrowband land mobile satellite propagation channel. International Journal of Satellite Communications and Networking, 26(4), 291-316.

\[3\] Loo, C. "Further results on the statistics of propagation data at L-band (1542 MHz) for mobile satellite communications." [1991 Proceedings] 41st IEEE Vehicular Technology Conference. IEEE, 1991.

\[4\] Perez-Fontan, Fernando, et al. "S-band LMS propagation channel behaviour for different environments, degrees of shadowing and elevation angles." IEEE transactions on broadcasting 44.1 (1998): 40-76.

\[5\] [Arndt, D. (2015). On Channel Modelling for Land Mobile Satellite Reception (Doctoral dissertation).](https://www.db-thueringen.de/receive/dbt_mods_00026568)

\[6\] Arapoglou, P.D.; Liolis, K.; Bertinelli, M.; Panagopoulos, A.; Cottis, P.; De Gaudenzi, R. (2011). "MIMO over satellite: A review". IEEE Communications Surveys & Tutorials. 13 (1): 27–51. doi:10.1109/SURV.2011.033110.00072

\[7\] Kyröläinen, J.; Hulkkonen, A.; Ylitalo, J.; Byman, A.; Shankar, B.; Arapoglou, P.D.; Grotz, J. (2014). "Applicability of MIMO to 
satellite communications". International Journal of Satellite Communications and Networking. 32 (4): 343–357. doi:10.1002/sat.1040

\[8\] Zang, Guo-zhen; Huang Bao-hua; Mu Jing (2010). One scheme of cooperative diversity with two satellites based on the alamouti code. IET 3rd International Conference on Wireless, Mobile and Multimedia Networks (ICWMMN 2010). pp. 151–4. doi:10.1049/cp.2010.0640. ISBN 978-1-84919-240-8.

\[9\] Pérez-Neira, A.I.; Ibars, C.; Serra, J.; Del Coso, A.; Gómez-Vilardebó, J.; Caus, M.; Liolis, K.P. (2011). "MIMO channel modeling and transmission techniques for multi-satellite and hybrid satellite-terrestrial mobile networks". Physical Communication. 4 (2): 127–139. doi:10.1016/j.phycom.2011.04.001

