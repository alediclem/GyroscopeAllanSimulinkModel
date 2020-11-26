# Allan variance analysis and modeling of a MEMS gyroscope

End-course project for *Misure ed Analisi Dati* (Metrology and Data Analysis) @ University of Rome Tor Vergata.

## Project summary
This project aims to produce a Simulink model for a MEMS angular speed sensor to be used in simulation of autonomous vehicle algorithms. The model must be *parametrized* to be adapted to the datasheet of the chosen sensor to approximate its behavior in terms of *noise*, *non linearities* and *non idealities*.
The project follows this schedule:
- Mathematical modeling of sensor output;
- *Allan variance* method for noise contributions classification;
- Data collection and model fitting of real sensor output;
- Comparison of model output and real output.

A low cost MARG sensor (InvenSense MPU9250) was used to collect data and to verify the model.
The data is collected from the sensor using an NXP LPC microcontroller with I2C serial protocol and streamed to a target computer via USB 2.0 Full-Speed protocol.


## Mathematical modeling
Gyroscope output is modeled as follows:

<!---
\boldsymbol{y}_{\omega}(t) = M\boldsymbol{\omega}^b_{b/i}(t)+\boldsymbol\beta_\omega^b(t) + \boldsymbol\nu_\omega(t)
-->

![\boldsymbol{y}_{\omega}(t) = M\boldsymbol{\omega}^b_{b/i}(t)+\boldsymbol\beta_\omega^b(t) + \boldsymbol\nu_\omega(t)
](https://render.githubusercontent.com/render/math?math=%5Cdisplaystyle+%5Cboldsymbol%7By%7D_%7B%5Comega%7D%28t%29+%3D+M%5Cboldsymbol%7B%5Comega%7D%5Eb_%7Bb%2Fi%7D%28t%29%2B%5Cboldsymbol%5Cbeta_%5Comega%5Eb%28t%29+%2B+%5Cboldsymbol%5Cnu_%5Comega%28t%29%0A)

where ![M
](https://render.githubusercontent.com/render/math?math=%5Cdisplaystyle+M%0A) is the *disalignment matrix*, ![\boldsymbol\nu_\omega(t)](https://render.githubusercontent.com/render/math?math=%5Cdisplaystyle+%5Cboldsymbol%5Cnu_%5Comega%28t%29)
models the *random fluctuations* of the sensor output; it is tipically assumed as a random variable with *normal distribution* with *zero mean* and diagonal covariance matrix.

![\boldsymbol\beta_\omega^b(t)](https://render.githubusercontent.com/render/math?math=%5Cdisplaystyle+%5Cboldsymbol%5Cbeta_%5Comega%5Eb%28t%29)
is a *bias* term which can be assumed as a *Wiener process* (random walk process):

<!---
\dot{\boldsymbol\beta}_\omega^b(t) = \mathcal{N}\left(0,\boldsymbol\sigma_{\beta\omega}^2\right)
-->

![\dot{\boldsymbol\beta}_\omega^b(t) = \mathcal{N}\left(0,\boldsymbol\sigma_{\beta\omega}^2\right)](https://render.githubusercontent.com/render/math?math=%5Cdisplaystyle+%5Cdot%7B%5Cboldsymbol%5Cbeta%7D_%5Comega%5Eb%28t%29+%3D+%5Cmathcal%7BN%7D%5Cleft%280%2C%5Cboldsymbol%5Csigma_%7B%5Cbeta%5Comega%7D%5E2%5Cright%29)

Bias contains, however, a **static contribution** which can be calculated from the still gyroscope output.
This static contribution is evaluated as an example with 100 samples from the still sensor at a sampling frequency of 100Hz. The y-axis histogram for this data sample is shown

<p align="center">
<img src="/images/bias_y_hist.png" width="512">
</p>

Mean and variance are evaluated as -0.3421 dps and 0.1395 dps, then it is possible to evaluate the standard deviation for the means distribution as

<!---
s\left(\bar{\beta}_{\omega,y}\right)=\frac{s\left({\beta}_{\omega,y}\right)}{\sqrt N}=0.014\text{dps}
-->

![s\left(\bar{\beta}_{\omega,y}\right)=\frac{s\left({\beta}_{\omega,y}\right)}{\sqrt N}=0.014\text{dps}](https://render.githubusercontent.com/render/math?math=%5Cdisplaystyle+s%5Cleft%28%5Cbar%7B%5Cbeta%7D_%7B%5Comega%2Cy%7D%5Cright%29%3D%5Cfrac%7Bs%5Cleft%28%7B%5Cbeta%7D_%7B%5Comega%2Cy%7D%5Cright%29%7D%7B%5Csqrt+N%7D%3D0.014%5Ctext%7Bdps%7D)

Considering a 95.45% *confidence level* related to a 2 *coverage factor+, the expanded uncertainty is 0.0279 dps.
The static bias contribution on the y-axis is then, within a ![2\sigma](https://render.githubusercontent.com/render/math?math=%5Cdisplaystyle+2%5Csigma)
confidence level,

![\beta_{\omega,y}=\left(-0.3421\pm 0.0279\right) \text{dps}](https://render.githubusercontent.com/render/math?math=%5Cdisplaystyle+%5Cbeta_%7B%5Comega%2Cy%7D%3D%5Cleft%28-0.3421%5Cpm+0.0279%5Cright%29+%5Ctext%7Bdps%7D)

The same procedure can be repeated on the other axes to obtain the entire **static bias vector**.
To reduce the confidence interval, it is necessary to increase the dimension of the sample.

## Allan variance
**Integration** of gyroscope output for attitude estimation **drifts** tipically because of bias; *output noise* is also an important factor of performance degradation of estimation algorithms, especially for long integration times.
The noise of an inertial sensor can be characterized in the *frequency domain* with as a **noise power spectral density** (PSD), or in the **time domain** through the **Allan variance**.

Allan variance is a method to analyze the phase and frequency stability of oscillators and atomic clocks introduced by David W. Allan in 1966.
This method converges even in presence of types of noises for which traditional variance does not converge to finite values, like *flicker noise* and *random walk noise*.
For this reason, Allan variance has been adopted to **identify the origin of noise contributions** on the inertial sensors output, first for **linear accelerometers**, then for **angular speed sensors**.

For the latter type sensors the Allan variance is calculated from a sequence ![\Omega[n]](https://render.githubusercontent.com/render/math?math=%5Cdisplaystyle+%5COmega%5Bn%5D) of  ![L](https://render.githubusercontent.com/render/math?math=%5Cdisplaystyle+L)
 output values, sampled at a frequency ![\frac1{\tau_0}](https://render.githubusercontent.com/render/math?math=%5Cdisplaystyle+%5Cfrac1%7B%5Ctau_0%7D).
 From this sequence, a set of ![L](https://render.githubusercontent.com/render/math?math=%5Cdisplaystyle+L) angles is calculated

 <!---
\theta[i]=\tau_0 \sum_{j=1}^i \Omega[j]
 -->

 ![\theta[i]=\tau_0 \sum_{j=1}^i \Omega[j]](https://render.githubusercontent.com/render/math?math=%5Cdisplaystyle+%5Ctheta%5Bi%5D%3D%5Ctau_0+%5Csum_%7Bj%3D1%7D%5Ei+%5COmega%5Bj%5D)

which is equivalent to a weighted cumulative sum of the output vector.
The Allan variance is then calculated from these angles

![\sigma_y^2(\tau)=\frac1{2\tau^2\left(N-2m\right)}\sum_{k=1}^{L-2m}\left(\theta[k+2m]-2\theta[k+m]+\theta[k]\right)^2](https://render.githubusercontent.com/render/math?math=%5Cdisplaystyle+%5Csigma_y%5E2%28%5Ctau%29%3D%5Cfrac1%7B2%5Ctau%5E2%5Cleft%28N-2m%5Cright%29%7D%5Csum_%7Bk%3D1%7D%5E%7BL-2m%7D%5Cleft%28%5Ctheta%5Bk%2B2m%5D-2%5Ctheta%5Bk%2Bm%5D%2B%5Ctheta%5Bk%5D%5Cright%29%5E2)


<!---
\sigma_y^2(\tau)=\frac1{2\tau^2\left(N-2m\right)}\sum_{k=1}^{L-2m}\left(\theta[k+2m]-2\theta[k+m]+\theta[k]\right)^2
-->

Using a square root it is possible to evaluate the **Allan deviation**. It is also possible to calculate the *confidence interval* for the estimated Allan variance assuming a ![\chi^2](https://render.githubusercontent.com/render/math?math=%5Cdisplaystyle+%5Cchi%5E2)
 distribution for the sample variance distribution.

 ## Noise terms identification
 It is shown that Allan variance is related to the *two-sided power spectral density* of the ![\Omega(t)](https://render.githubusercontent.com/render/math?math=%5Cdisplaystyle+%5COmega%28t%29)
 signal, in particular it is the noise PSD of the output signal from a **filter** with transfer function

 <!---
H(f)=\frac{\sin^4(\pi f) \tau}{(\pi f \tau)^2}
 -->

 ![H(f)=\frac{\sin^4(\pi f \tau)}{(\pi f \tau)^2}](https://render.githubusercontent.com/render/math?math=%5Cdisplaystyle+H%28f%29%3D%5Cfrac%7B%5Csin%5E4%28%5Cpi+f+%5Ctau%29%7D%7B%28%5Cpi+f+%5Ctau%29%5E2%7D)

that is a **band-pass filter** with **pass-band determined by the cluster dimension** of the Allan variance expression. It is then possible to separate the noise contributions by varying the  time of integration, if the **noise terms are supposed to be dominant in different frequency bands** with respect to others.

Three main noise contributions are deduced from the logarithmic plot of the **Allan deviation**, depending on the **slope** and **location** of the curve sectors:
- **Angle Random Walk** ![N^2](https://render.githubusercontent.com/render/math?math=%5Cdisplaystyle+N%5E2)
(ARW) represents the spectrum of the white noise and is constant with frequency; it is the **noise density** and it is tipically specified by the manufacturer. This contribution is represented as a -1/2 slope line on the plot, so the random walk coefficient ![N](https://render.githubusercontent.com/render/math?math=%5Cdisplaystyle+N) can be read on the plot as the value of the Allan deviation in ![\tau=1](https://render.githubusercontent.com/render/math?math=%5Cdisplaystyle+%5Ctau%3D1). It can be related to the ![\boldsymbol\nu_\omega(t)](https://render.githubusercontent.com/render/math?math=%5Cdisplaystyle+%5Cboldsymbol%5Cnu_%5Comega%28t%29) variance.

- **Bias instability** ![B](https://render.githubusercontent.com/render/math?math=%5Cdisplaystyle+B) is a low-frequency noise component caused by *flicker noise* on the sensor output. It is shown on the Allan deviation plot as a 0-slope line.

- **Rate Random Walk** ![K](https://render.githubusercontent.com/render/math?math=%5Cdisplaystyle+K) is produced by brownian noise on the gyroscope output and can be related to bias slow variations.


Supposing that the noise contributions are **statistically independent**, Allan variance is their sum:

![\sigma^2(\tau)=\sigma_N^2(\tau)+\sigma_B^2(\tau)+\sigma_K^2(\tau)](https://render.githubusercontent.com/render/math?math=%5Cdisplaystyle+%5Csigma%5E2%28%5Ctau%29%3D%5Csigma_N%5E2%28%5Ctau%29%2B%5Csigma_B%5E2%28%5Ctau%29%2B%5Csigma_K%5E2%28%5Ctau%29)

## Real gyroscope Allan variance
The **MPU9250** gyroscope has been configured disabling the low-pass filter and with a full-scale range of 500 dps; the sampling frequency was set to 100 Hz.
The x-axis output for a **12-hours sample collection** is the following:

<p align="center">
<img src="/images/test_timeplot.png" width="480">
</p>

Considering a 68% confidence level, the **Allan deviation plot** is the following:

<p align="center">
<img src="/images/test_allandev_x_degs.png" width="480">
</p>

From this plot it is possible to evaluate all the **noise contributions**:

<p align="center">
<img src="/images/test_allandev_x_arw.png" width="480">
</p>

<p align="center">
<img src="/images/test_allandev_x_binst.png" width="480">
</p>

<p align="center">
<img src="/images/test_allandev_x_rrw.png" width="480">
</p>

Repeating the same calculations, it is possible to evaluate the noise coefficients N, B, K for all sensor axes and characterize the sensor with the measured Allan deviation plot:

<p align="center">
<img src="/images/mpu9250_allan.png" width="480">
</p>

With these parameters it is possible to implement a Simulink model which can simulate the real sensor noise output. The resultant block also includes temperature instability:

<p align="center">
<img src="/images/block.png" width="480">
</p>

Allan noise terms and other block parameters can be deduced from device datasheet or measurement analysis and configured in the block mask to emulate the real sensor:

<p align="center">
<img src="/images/blockMask.png" width="480">
</p>

## Output Comparison
The model output fits well the output of the real sensor and is suitable for sensor simulation in a model based control design approach.

<p align="center">
<img src="/images/simulation_comparison.png" width="480">
</p>

<p align="center">
<img src="/images/simulation_comparison_allan.png" width="480">
</p>
