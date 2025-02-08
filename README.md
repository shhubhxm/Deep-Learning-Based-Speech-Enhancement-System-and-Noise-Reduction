# Deep Learning Based Noise Reduction and Speech Enhancement System

## Introduction
People with hearing loss depend heavily on hearing aid to hear properly. However, traditional hearing aid also amplify disturbing environmental noise. This project implements **two deep learning models, one can classify the type of noise, the other can retain human voice and reduce environmental noise**. This proposal has the advantage of adapting to environmental noise compared to the traditional hearing aid.

In this repo, Our team at Frugal only provide codes of data preprocessing, model training and model validation that runs on PC.

## Structure
![](https://i.imgur.com/XduRFpP.png)

## Total works
- Audio to Spectrogram Preprocessing & Inversion Algorithm (STFT & MFCC)
- Dataset Generator: Human Voice & Environmental Noise Mixing
- Noise Classifier (NC) Model
- Noise Reduction Model (DDAE)
- Preprocessing, NC and DDAE Integration on PC
- Model conversion: Implement quantization and convert into tflite model


## Datasets
The goal of this project is to recognize the noise in the input audio and clean them out with deep learning techniques. We mix clean voice and noises together as training input, and clean voice as output.

As we all know, the data you use is more important than the model itself. We choose TIMIT as clean voice dataset. And according to [*Sounds perceived as annoying by hearing-aid users in their daily soundscape*](https://www.researchgate.net/publication/260093127_Sounds_perceived_as_annoying_by_hearing-aid_users_in_their_daily_soundscape), we select four categories of noise that respondents mentioned most, which are verbal human sounds, TV/radio, vehicles and household appliances as the goal that we want to clean out from the input audio.


## Models
### Noise Classifier, NC
<img src="images/structure.jpg" width=80% height=80%>


This section tells setup of Noise Classifier. We use another 
<!-- [repo](https://github.com/qqq89513/NOISE_CLASSIFIER) 
for it, so please `cd` another folder before continues.
```
git clone https://github.com/qqq89513/NOISE_CLASSIFIER
``` -->
The repo itself does not contain dataset and the trained weights. The weights will be published as soon as the training stage is finalized.
Here are some public sources that we used for Noise Classifier:
  - Dataset: TIMIT mixed with AudioSet
  - Preprocessing reference: [VGGish](https://github.com/tensorflow/models/tree/master/research/audioset/vggish)
File desciption:
  - `utils/data_paser.py`: This script generates the list of dataset file paths as json. You can execute the script directly:
      ```
      python ./utils/data_paser.py
      ```
  - `requirement.txt`: The dependencies of this repo, will be available soon.
  - `vggish_params.py`: Hyperparameters.
  - `vggish_input.py`: Converter from audio waveform into input examples.
  - `vggish_mel_features.py`: Audio feature extraction helpers. This is a pure nummpy implementation, which can be referenced for embedded implementation.
  - `NC_training.py`
    1. Loads dataset from the disk
    2. Preprocesses via `vggish_input.py`
    3. Build the classifier model
    4. Train the model
    5. Inference

### Deep Denoising Autoencoder, DDAE
<img src="images/structure.jpg" width=80% height=80%>

DDAE is one of the derivatives of the autoencoder. Autoencoders are neural networks trained in an unsupervised way to attempt to copy inputs to outputs. And denoise autoencoder(DAE) is trained to reduce the noise from the input. In this project, we add multiple layers to the DAE and makes it DDAE.

Our preprocessing will output 0.5 sec STFT spectrogram, and we accumulate 1 sec of it as the input of DDAE. We feed it to the trained DDAE model, and the model will reduce the noise from the input, then the program will convert the clean spectrogram to `.wav` files.


- Dataset: TIMIT mixed with AudioSet
- Preprocessing reference: VGGish but without converting to Mel spectrogram
- `config_params.py`: Configuration of the whole DDAE training process
- `prepare_data.py`: Used to prepare data in `.h5` format
- `DDAE.py`: The model itself
- `prediction_denoise.py`: Used to validate the model

<!-- # How to prepare the dataset? 
- NC: Please check [NC's repo](https://github.com/qqq89513/NOISE_CLASSIFIER).
- DDAE: Download our pre-prepared dataset from [Google Drive](https://drive.google.com/file/d/1eiRYFSOqBTPAJabmzAV5s0pQaqCE-OVg/view?usp=sharing) and unzip it to the root directory of the repo. Then run the `prepare_data.py`. -->


# Results
### Noise Classifier, NC
Our original purpose is to classify 4 classes of noise. We found that the "TV/radio" is too similar to "Human noise", which cause accuracy down to 51%, so we had to abandon it. Finally, the accuracy of the 3 classes of noise was raised to 71%.

Every picture with 3 subplots in the table below shows the result of classifying different noises.
- Subplot 1: The waveform of the input
- Subplot 2: The spectrogram of the input
- Subplot 3: The probability of every kind of noises. Our NC classify the noise every 0.5 sec. The darker the colour the higher the probability.

|        Class         |               Results                |
|:--------------------:|:------------------------------------:|
|     Human noise     | ![](https://i.imgur.com/iqorlX3.png) |
|       Vehicles       | ![](https://i.imgur.com/BVOs7x9.png) |
| Household appliances | ![](https://i.imgur.com/0ERrEBV.png) |

### Deep Denoising Autoencoder, DDAE
![](https://i.imgur.com/v45r9Mc.png)

The following pictures are the spectrograms of clean voice, mixed audio and denoised audio. The XY axis represents time and frequency. The red colour shows higher energy at the specific time and frequency and the blue colour vice versa.

Take household appliances as an example—you may notice that different SNR levels lead to varying outcomes. A higher SNR generally produces better results.

| SNR |             Clean Voice              |                Mixed                 |               Denoised               |
|:---:|:------------------------------------:|:------------------------------------:|:------------------------------------:|
| -10 | ![](https://i.imgur.com/kfS5rWq.png) | ![](https://i.imgur.com/OkkhpRU.png) | ![](https://i.imgur.com/RgtUtfz.png) |
| -5  | ![](https://i.imgur.com/p5aiv8P.png) | ![](https://i.imgur.com/YLUbwpy.png) | ![](https://i.imgur.com/EOkwwrq.png) |
|  0  | ![](https://i.imgur.com/OJ7aGnv.png) | ![](https://i.imgur.com/d0dNDgu.png) | ![](https://i.imgur.com/BsdyVOX.png) |
|  5  | ![](https://i.imgur.com/LA527kY.png) | ![](https://i.imgur.com/hnOHWkz.png) | ![](https://i.imgur.com/hgVe0ey.png) |
| 10  | ![](https://i.imgur.com/P0eWIzy.png) | ![](https://i.imgur.com/EQDW0BS.png) | ![](https://i.imgur.com/GRmGLsB.png) |
| 15  | ![](https://i.imgur.com/cX0rkID.png) | ![](https://i.imgur.com/zsVBgnu.png) | ![](https://i.imgur.com/X5Why2f.png) |

Also, we found that those clean areas become dirty after denoising. That's because the model learned lots of different data, and the spectrogram of those data is arbitrary and has randomness and diversity.
