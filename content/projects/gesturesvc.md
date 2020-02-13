+++
date = "2019-11-30"
title = "Gesture Classification of sEMG Signals for Prosthesis Control"
slug = "gesture-classifier"
+++

### Overview 
A major hurdle for building effective prostheses is the control interface. Patients will not find the device useful unless they are able to accurately and intuitively control the device. The current industry standard is to capture the muscle signals in the residual limb using surface electromyography (sEMG) electrodes. These signals must be accurately deciphered by the device for the prosthesis to perform the intended action. 

This project attempts to define a supervised machine learning model called a Support Vector Classifier to accurately classify gestures based on sEMG signals. The classifier was trained on a labelled dataset consisting of four classes. Each class represents a hand gesture (rock, paper, scissors, ok). 

The time series sEMG data was preprocessed using the Daubechies discrete wavelet transform to characterize the waveform before fitting the data to the classifier. After fitting the model and optimizing the parameters, 92% accurate classification was achieved on the test dataset. This is on par with current prosthetic device classification accuracy. Further testing is required to determine if the program can be executed in real-time (less than 75ms) on a typical embedded processor that would be used in a prosthesis today.

### Implementation
A Support Vector Machine or Support Vector Classifier (SVC) will be created to solve the problem of classifying gestures based on forearm sEMG data in real-time. An SVC is a method of supervised machine learning which means it is trained using labelled data. The SVC finds a line or hyperplane which separates data into categories. Then new data can be categorized based on where it falls relative to the separating line. For this project, the dataset has much higher dimensionality and the dividing line will be a hyperplane. 

{{< figure src="/images/svc/svc.PNG" caption="SVC Hyperplane" >}}

The [dataset](https://www.kaggle.com/kyr7plus/emg-4) to be utilized in this report is based on sEMG data collected from the [Myo armband](https://neurosciencenews.com/shop/neuroscience-clothing/myo-gesture-control-armband-black/). The use of this device is prevalent in prosthetic control research due to its 8 non-invasive sensors and the fact that it is commercially available. The Myo armband sensors are dry and the device can be easily slipped on by patients. This does have performance drawbacks as dry electrodes have worse performance than gel-based electrodes. 

The dataset was created by taking readings using the Myo armband while the patients hold gestures. The dataset consists of four classes, labelled ‘rock’, ‘paper’, ‘scissors’, ‘ok’.
{{< figure src="/images/svc/rps.jpg" caption="Gestures for Classification" >}}

### Results 
The Support Vector Classifier described in this document was able to classify gestures based on forearm sEMG signals with a high degree of accuracy (92%). Optimizing the tuning parameters of the model were essential to achieving high accuracy classification. 
{{< figure src="/images/svc/optimized.png" caption="Confusion Matrix" >}}

The full detailed report can be read [here](/images/svc/report.pdf)


