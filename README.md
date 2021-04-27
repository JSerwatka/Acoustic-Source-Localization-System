## Table of contents
- [Project goals](#Project-goals)
- [Software - simplified description](#Software---simplified-description)
- [TDoA measurement method](#TDoA-measurement-method)
- [Angle calculation](#Angle-calculation)
- [Error handling](#Error-handling)
- [GUI](#GUI)

## Project
### Project goals
This project implements an _Acoustic source localization system_ using _LabVIEW_ and _sbRIO-9636_ (Single-Board Controller from _National Instruments_). 

The device is designed to show the direction of sound (_Direction of Arrival_ (DoA)) in  real-time, in the form of an angle deviation from the center of the device.

The whole system consists of three main components: 
- 4 microphones, 
- 4 microphone amplifiers,
- _sbRIO-9636_. 

The angle mentioned above is calculated based on the delay of the acoustic signals (TDoA) between successive pairs of microphones. Based on the data collected in this way, the position of the sound relative to the center of the device can be calculated.

An additional goal of this project is to use the cheapest possible components, but with a satisfactory accuracy of calculations in difficult acoustic conditions. The use of low-quality components creates additional complications but allows the system to be easily reproducible.

### Software - simplified description
![diagram](https://user-images.githubusercontent.com/33938646/71518613-0a48f800-28b4-11ea-8894-7f0e0f73baa9.png)
Below is a simplified description of the successive steps the system uses to calculate the _DoA_:
1. After amplifying the signal, it is sampled at the selected frequency (default - 50 kS/s).
2. The signal is filtered with a high-pass filter with a cut-off frequency of 2.5 kHz.
3. When the sound exceeds a certain threshold, a user-selected number of samples is collected
(default - 9000).
4. The collected audio signal is queued in the FIFO.
5. A deterministic loop, working in the real-time part of the program, receives the data from the FIFO and passes it to the internal queue (RT FIFO).
6. The data is received in a non-deterministic loop that analyzes the signal.
7. Using _Generalized Cross Correlation_ from _Phase Transform Weights_, _TDoA_ is measured.
8. Based on the data from the previous point, the angle of incidence of sound for each pair of microphones is calculated. 
9. The angles are analyzed to ensure that there is no measurement error.
10. The final angle is computed.
11. The angle data is sent to the user panel.
12. The following data is displayed on the user panel:
    * the final angle of incidence of sound,
    * graph of the tested signal for one of the microphones,
    * TDoA graph for one of the pairs of microphones.

### TDoA measurement method
The main task of the described system is to efficiently calculate the difference in time of arrival of sound between pairs of microphones. Many algorithms have been developed to solve this problem, among the most popular are _Maximum Likelihood (ML)_, _Multiple Signal Classification (MUSIC)_ and _Generalized
Cross-Correlation (GCC)_. The first two methods have a disadvantage which is crucial in the case of
the system described in this work; to work properly, they require a large number of microphones and complex calculations. Both of these disadvantages are impossible to solve with a simple
microphone array using a device that is not specialized for this task.
For the above reasons, the GCC algorithm was chosen.

To explain _Generalized Cross-Correlation (GCC), we must first create a mathematical model of the signals received by the microphones:

![](https://latex.codecogs.com/gif.latex?\dpi{120}&space;\bg_white&space;\begin{equation}&space;\color{black}&space;\begin{aligned}&space;&r_{1}(t)=s(t)&plus;n_{1}(t),&space;\\&space;&r_{2}(t)=s(t-D)&plus;n_{2}(t),&space;\end{aligned}&space;\qquad&space;0\leq&space;t\leq&space;T&space;\end{equation})

where _s(t)_ is an audio signal, _n1(t)_ and _n2(t)_ are additional, random noises independent of the original signal, D represents a delay of the sound signal reaching the next microphones, and T represents the observation time of the signal.

The main method of computing TDoA is _Cross-correlation (CC)_. The _Cross-correlation_ function shall be applied to the two received signals. Thanks to this, we are able to obtain a graph of the similarity of two signals for different time shifts. Then we just need to find the value at which the function is at its maximum, and we get the sought-after amount of delay between the signals. The _CC_ method is extremely simple, but at the same time very unreliable, especially with a low SNR value.

In order to increase the accuracy of the results, I use the appropriate filtering and _weighting function_ to enhance the important signal parts and suppress unnecessary noise. This way, we obtain the _Generalized Cross-Correlation_, which can be represented by the following equation:

![](https://latex.codecogs.com/gif.latex?\dpi{120}&space;\bg_white&space;\begin{equation}&space;\color{black}&space;R_{r_{1}r_{2}}(\tau)=\int_{-\infty}^{\infty}\psi_{p}(f)G_{r_{1}r_{2}}(f)e^{\frac{j2\pi}{\tau}}df,&space;\\&space;D_{p}=argmax[R_{r_{1}r_{2}}(\tau)]&space;\end{equation})

where _psi_ is our _weighting function_, G - _cross-correlation_ between received signals, and D - the value of the delay we are looking for.

The most frequently studied _weighting functions_ are: _Roth_, _Smoothed COherence Transform (SCOT)_, *Phase_
_Transform (PHAT)* and _Eckart Filter_. Many of these analyzes clearly show that the most effective method is GCC-PHAT. This approach is also most commonly used in similar projects. It is characterized by simplicity of calculations, with simultaneous high accuracy in TDoA estimation, which makes it extremely useful in sound localization systems operating in real time. PHAT can be represented by the following equation:

![](https://latex.codecogs.com/gif.latex?\dpi{120}&space;\bg_white&space;\psi_{p}(f)=\frac{1}{G_{r_{1}r_{2}}(f)})

Flowchart of the TDoA estimation method, using _generalized_cross-correlatio_:

![GCC](https://user-images.githubusercontent.com/33938646/71519207-d28f7f80-28b6-11ea-8b23-c52e10f75539.jpg)

Implementation of GCC-PHAT in LabVIEW:

![image](https://user-images.githubusercontent.com/33938646/116226830-c2f6a400-a753-11eb-84b6-1314100b52a0.png)

### Angle calculation
Having TDoA allows you to calculate the angle of incidence of an acoustic wave on a pair of microphones. To achieve this, a simple trigonometric operation is used.

![image](https://user-images.githubusercontent.com/33938646/116226921-d86bce00-a753-11eb-8656-58fa830cf6c0.png)

The figure shows a diagram of the microphone array, where _L_ is the distance between the microphones, _alpha_, _beta_, and _gamma_ are the angles at which acoustic waves hit the next pairs of microphones, and _theta_ is the final angle with respect to the center of the array.

The alpha, beta and gamma angles are calculated using the equation below:

![](https://latex.codecogs.com/gif.latex?\dpi{120}&space;\bg_white&space;\theta=\arccos(\frac{c\tau}{l}),)

where _c_ is the speed of sound, _tau_ - the delay between the microphones, and the distance between them is represented by _l_.

After calculating the alpha, beta, and gamma angles, you can estimate the final theta angle. Below is a possible solution to this problem in four steps:

![Bez tytułu](https://user-images.githubusercontent.com/33938646/116227784-c9d1e680-a754-11eb-8d47-13ad17652c5d.png)

The use of three different values of _a_ and _b_ to calculate the angle is required to average the errors of each of the angles.

### Error handling
Due to the poor quality of the components, angle error handling is required to protect a user from receiving false information about the direction of the sound source. In order not to lose data with each device error, two types of errors are being proposed:
1. _Single angle error_ - occurs when:
    - ![](https://latex.codecogs.com/gif.latex?\dpi{120}&space;\bg_white&space;\mid\alpha-\beta\mid>20\quad\&space;\land\quad&space;\mid\alpha-\gamma\mid>35&space;\&space;\longrightarrow\&space;\alpha&space;\text{&space;angle&space;error},)
    - ![](https://latex.codecogs.com/gif.latex?\dpi{120}&space;\bg_white&space;\mid\beta-\alpha\mid>20\quad\&space;\land\quad&space;\mid\beta-\gamma\mid>20&space;\&space;\longrightarrow\&space;\beta&space;\text{&space;angle&space;error},)
    - ![](https://latex.codecogs.com/gif.latex?\dpi{120}&space;\bg_white&space;\mid\gamma-\alpha\mid&space;>35\quad\&space;\land\quad&space;\mid\gamma-\alpha\mid>20&space;\&space;\longrightarrow\&space;\gamma\text{&space;angle&space;error},)
2. _Critical angles error_ - occurs when:
    - ![](https://latex.codecogs.com/gif.latex?\dpi{120}&space;\bg_white&space;\mid\alpha-\beta\mid>20\quad\&space;\land\quad&space;\mid\beta-\gamma\mid>20\quad\&space;\land\quad\mid\alpha-\gamma\mid>35)

When a _single angle error_ occurs, the result is displayed to the user, but also an error notification is sent, which is signaled by lighting the appropriate LED on the user panel. The _critical angles error_ causes the calculated result to be ignored and a red LED on the user panel lights up.

### GUI
The panel has been designed to be as intuitive as possible, but also to contain all the necessary information for the user of the application. For this reason, it has been divided into four segments, visually separated from each other:

![image](https://user-images.githubusercontent.com/33938646/116227061-005b3180-a754-11eb-90d2-161e5b335e8f.png)

1. _Settings_ - contains basic application settings that allow the user to choose between the precision and speed of the program. You can adjust:
    - _SamplesToSend_ - allows you to control the number of samples that will be taken for analysis for each microphone (default - 9000). The greater the value of this control, the more accurate the result of the angle calculation, but slower performance.
    - _AcqLoopFreq_ - allows you to select the signal sampling rate (default - 50kS/s). The greater the value of this control, the more precise representation of the input signal, but the shorter duration of the analyzed signal.
    - _Threshold_ - allows you to set the application sensitivity, i.e. the minimum signal strength that will be accepted for analysis (default - 0.1). The lower value of the control allows you to observe low-intensity signals but increases the chance of errors and accidental analysis.
    - _STOP_ - this control stops all VIs.
2. _Errors_ - contains error indicators:
    - _Single angle error_,
    - _Critical angles error_,
    - _FIFO overflow error_,
    - internal application errors.
3. _Angle indicator_ - this indicator shows where the sound is coming from and displays the calculated angle as a number.
4. _Signals_ - contains two graphs:
    - _Signal_ - draws a graph of the amplitude of the input signal for the first microphone in successive units of time (time is presented as a number of samples).
    - _TDOA_ - shows a graph of the correlation of signals from the first and second microphones analyzed with the _GCC-PHAT_ algorithm and their shifts in relation to each other.
