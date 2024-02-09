# SHARP
SHARP (Algorithms for Human Activity Recognition with IEEE 802.11ac Wi-Fi Router) is a repository containing Python code for processing and analyzing Wi-Fi channel frequency response (CFR) data collected in an IEEE 802.11ac network for human activity recognition. The repository provides scripts for phase sanitization, Doppler computation, dataset creation, training learning algorithms for human activity recognition (HAR), inference, and performance evaluation. Algorithms for human activity recognition with a commercial IEEE 802.11ac router @ 5 GHz, 80 MHz of bandwidth.

## Table of Contents
- [How to Use](#how-to-use)
- [Phase Sanitization](#phase-sanitization)
- [Doppler Computation](#doppler-computation)
- [Dataset Creation](#dataset-creation)
- [Understanding Doppler Plot Values](#understanding-doppler-plot-values)
- [Train the Learning Algorithm for HAR](#train-the-learning-algorithm-for-har)
- [Parameters](#parameters)
- [Python and Relevant Libraries Version](#python-and-relevant-libraries-version)

# How to use
Clone the repository and enter the folder with the python code:
```bash
cd <your_path>
git clone git.url
```

Download the input data from http://researchdata.cab.unipd.it/id/eprint/624 and unzip the file.
For your convenience, you can use the ```input_files``` inside this project folder to place the files but the scripts work whatever is the source folder.

The dataset contains Wi-Fi channel frequency response (CFR) data collected in an IEEE 802.11ac network through [NEXMON CSI](https://github.com/seemoo-lab/nexmon_csi). 
The information is collected by a monitor node (ASUS RT-AC86U router) while two terminals are exchanging traffic in channel 42 (5.21 GHz for the center frequency and 80 MHz of bandwidth) and a person acts as an obstacle for the transmission by performing different activities. 
The considered movements are the following: walking (W) or running (R) around, jumping (J) in place, sitting (L) or standing (S) somewhere in the room, sitting down and standing up (C) continuously, and doing arm gym (H).
The CFR data for the empty room (E) is also provided. We obtained data from three volunteers, a male, and two females.
The complete description of the dataset can be found in the reference paper.

The code for SHARP is implemented in Python and can be found in the ```Python_code``` folder inside this repository. The scripts to perform the processing are described in the following, together with the specific parameters.

# Phase sanitization
### 1. CSI Phase Sanitization Signal Preprocessing

The `CSI_phase_sanitization_signal_preprocessing.py` script processes Wi-Fi CSI (Channel State Information) data, focusing on phase information, specifically the process of cleaning the phase information.

Phase sanitization in Wi-Fi CSI data represents the phase shift of the wireless signal as it travels from the transmitter to the receiver. This phase information can be affected by various factors such as noise and environmental conditions, and it directly impacts the quality of the received signal.

#### Parameters

- **Directory of the input data**: The directory path where the input data is located.
- **Process all the files in subdirectories**: Boolean value (1 or 0) indicating whether to process all the files in subdirectories or not.
- **Name of the file to process**: Name of the file to process (only applicable if the previous field is 0).
- **Number of spatial streams**: The number of independent data streams that can be transmitted simultaneously from multiple transmit antennas to multiple receive antennas (MIMO).
- **Number of cores**: The number of CPU cores to use for processing.
- **Index where to start the processing for each stream**: Index where to start the processing for each stream.

### Hampel Filter

This script utilizes the Hampel filter, which is a noise removal technique applied to the CSI data. It processes a .mat data file and removes unwanted data points.

```bash
python CSI_phase_sanitization_signal_preprocessing.py <'directory of the input data'> <'process all the files in subdirectories (1) or not (0)'> <'name of the file to process (only if 0 in the previous field)'> <'number of spatial streams'> <'number of cores'> <'index where to start the processing for each stream'> 
```
e.g., python CSI_phase_sanitization_signal_preprocessing.py ../input_files/S1a/ 1 - 1 4 0

### 2. CSI Phase Sanitization H Estimation

`CSI_phase_sanitization_H_estimation.py` is used to perform optimization on the signal using a lasso regression technique. It is designed to refine signals and generate time-frequency representations, clean unwanted noise, enhance signal quality, and make it more clear and reliable.
#### Purpose
- Perform optimization on signals.
- Use lasso regression technique.
- Refine signals.
- Generate time-frequency representations.
- Clean unwanted noise.
- Enhance signal quality.
- Make signals more clear and reliable.

```bash
python CSI_phase_sanitization_H_estimation.py <'directory of the input data'> <'process all the files in subdirectories (1) or not (0)'> <'name of the file to process (only if 0 in the previous field)'> <'number of spatial streams'> <'number of cores'> <'index where to start the processing for each stream'> <'index where to stop the processing for each stream'> 
```
e.g., python CSI_phase_sanitization_H_estimation.py ../input_files/S1a/ 0 S1a_E 1 4 0 -1

### 3. CSI Phase Sanitization Signal Reconstruction

The `CSI_phase_sanitization_signal_reconstruction.py` script is designed to enhance the amplitude of the signal, improve accuracy by connecting errors, determine the strength and direction of strong signals, and correct any mistakes in the data related to signal strength.

#### Purpose

- **Amplitude Enhancement**: The script aims to boost the amplitude of the signal, which can enhance the quality of the data and make it more suitable for further analysis.
  
- **Accuracy Improvement**: By connecting errors in the data, the script helps to improve the overall accuracy of the signal reconstruction process. This is crucial for ensuring that the reconstructed signal reflects the true underlying data as accurately as possible.

- **Strength and Direction Analysis**: It analyzes the strength and direction of strong signals within the data. Understanding these aspects can provide valuable insights into the characteristics of the signal and help in making informed decisions during the reconstruction process.

- **Error Correction**: The script also focuses on correcting any mistakes or inconsistencies in the data related to signal strength. This ensures that the reconstructed signal is reliable and free from errors or artifacts.



```bash
python CSI_phase_sanitization_signal_reconstruction.py <'directory of the processed data'> <'directory to save the reconstructed data'> <'number of spatial streams'> <'number of cores'> <'index where to start the processing for each stream'> <'index where to stop the processing for each stream'> 
```
e.g., python CSI_phase_sanitization_signal_reconstruction.py ./phase_processing/ ./processed_phase/ 1 4 0 -1

# Doppler computation

The `CSI_doppler_computation.py` script processes Wi-Fi data to understand human movement through the Doppler effect. It analyzes changes in the Wi-Fi signal caused by human motion.

#### Parameters

- **Noise Parameter:** 
  - This parameter sets a threshold, with signals below it considered as noise.

- **Number of Packets in Sample (Size of Each Sample):** 
  - This parameter represents the length of time or duration of signal measurement.

- **Number of Packets for Sliding Operation:** 
  - This parameter represents the step size when moving from one sample to another. It controls the amount of overlap between samples.

```bash
python CSI_doppler_computation.py <'directory of the reconstructed data'> <'sub-directories of data'> <'directory to save the Doppler data'> <'starting index to process data'> <'end index to process data (samples from the end)'> <'number of packets in a sample'> <'number of packets for sliding operations'> <'noise level'> <--bandwidth 'bandwidth'>
```
e.g., python CSI_doppler_computation.py ./processed_phase/ S1a,S1b,S1c,S2a,S2b,S3a,S4a,S4b,S5a,S6a,S6b,S7a ./doppler_traces/ 800 800 31 1 -1.2

To plot the Doppler traces use (first to plot all the antennas, second single antenna for all the activities)
`CSI_doppler_plots_antennas.py` apply log transformation to the Spectrogram.
```bash
python CSI_doppler_plots_antennas.py <'directory of the reconstructed data'> <'sub-directory of data'> <'length along the feature dimension (height)'> <'sliding length'> <'labels of the activities to be considered'> <'last index to plot'>
```
e.g., python CSI_doppler_plots_antennas.py ./doppler_traces/ S7a 100 1 E,L1,W,R,J1 20000

`CSI_doppler_plots_activities.py` plot multiple activities on single plot.

```bash
python CSI_doppler_plots_activities.py <'directory of the reconstructed data'> <'sub-directory of data'> <'length along the feature dimension (height)'> <'sliding length'> <'labels of the activities to be considered'> <'first index to plot'> <'last index to plot'>
```
e.g., python CSI_doppler_plots_activities.py ./doppler_traces/ S7a 100 1 E,L1,W,R,J1 570 1070

#### Pre-computed Doppler traces
If you want to skip the above processing steps, you can find the Doppler traces [in this Google Drive folder](https://drive.google.com/drive/folders/1SilO6VD73Lz8sjZ-KQgFnQ2IKRvggqPg?usp=sharing). In the same folder, the sanitized channel measurements for S2a and S7a are uploaded as examples in ```processed_phase```. Exaples of plots of the Doppler traces are also included.

### Dataset creation
- Create the datasets for training and validation
```bash
python CSI_doppler_create_dataset_train.py <'directory of the Doppler data'> <'sub-directories, comma-separated'> <'number of packets in a sample'> <'number of packets for sliding operations'> <'number of samples per window'> <'number of samples for window sliding'> <'labels of the activities to be considered'> <'number of streams * number of antennas'>
```
  e.g., python CSI_doppler_create_dataset_train.py ./doppler_traces/ S1a,S1b,S1c 31 1 340 30 E,L,W,R,J 4

- Create the datasets for test
```bash
python CSI_doppler_create_dataset_test.py <'directory of the Doppler data'> <'sub-directories, comma-separated'> <'number of packets in a sample'> <'number of packets for sliding operations'> <'number of samples per window'> <'number of samples for window sliding'> <'labels of the activities to be considered'> <'number of streams * number of antennas'>
```
  e.g., python CSI_doppler_create_dataset_test.py ./doppler_traces/ S2a,S2b,S3a,S4a,S4b,S5a,S6a,S6b,S7a 31 1 340 30 E,L,W,R,J 4

### Understanding Doppler Plot Values

![CSI Doppler Activity S2a Running](https://i.ibb.co/2kZTZzX/csi-doppler-activity-logs-S2a-R.png)

When analyzing Doppler plots generated from radar or sensor data, it's essential to interpret the velocity and power values correctly. Here's how to read the plot values with velocity and power represented in dB:

#### Velocity Axis:
- The velocity axis represents the Doppler shift, which corresponds to the movement of objects in the environment.
- Positive values indicate motion towards the radar/antenna, while negative values indicate motion away from the radar/antenna.

#### Power Axis (in dB):
- The power axis represents the signal strength or power level of the received signal.
- Power levels are represented in logarithmic scale, such as decibels (dB), to compress the dynamic range of the data.
- Higher values on the power axis indicate stronger signal power, while lower values indicate weaker signal power.

#### Interpretation:
- **Peak Analysis**: Look at the intersection of a certain velocity (or frequency shift) value on the x-axis with the corresponding power level on the y-axis.
- **Peak Magnitude**: Higher power levels indicate stronger reflections or signal returns at that particular velocity, suggesting significant movement or activity associated with that frequency shift.
- **Noise Floor**: Lower power levels indicate weaker signal returns, which may correspond to less significant movements or noise in the environment.

In summary, positive peaks on the plot represent movement towards the radar/antenna with higher signal power, while negative peaks represent movement away from the radar/antenna. The power values (in dB) indicate the strength of the reflected signals at different velocities (or frequency shifts).


### Train the learning algorithm for HAR
```bash
python CSI_network.py <'directory of the datasets'> <'sub-directories, comma-separated'> <'length along the feature dimension (height)'> <'length along the time dimension (width)'> <'number of channels'> <'number of samples in a batch'> <'name prefix for the files'> <'activities to be considered, comma-separated'> <--bandwidth 'bandwidth'> <--sub-band 'index of the sub-band to consider (for 20 MHz and 40 MHz)'> 
```
e.g., python CSI_network.py ./doppler_traces/ S1a 100 340 1 32 4 single_ant E,L,W,R,J

### Use the trained algorithm for inference
- Run the algorithm with the test data 
```bash
python CSI_network_test.py <'directory of the datasets'> <'sub-directories, comma-separated'> <'length along the feature dimension (height)'> <'length along the time dimension (width)'> <'number of channels'> <'number of samples in a batch'> <'name prefix for the files'> <'activities to be considered, comma-separated'> <--bandwidth 'bandwidth'> <--sub-band 'index of the sub-band to consider (for 20 MHz and 40 MHz)'> 
```
  e.g., python CSI_network_test.py ./doppler_traces/ S7a 100 340 1 32 4 single_ant E,L,W,R,J

- Compute the performance metrics using the output file of the test
```bash
python CSI_network_metrics.py <'name of the output file containing the metrics'> <'activities to be considered, comma-separated'>
```
  e.g., python CSI_network_metrics.py complete_different_E,L,W,R,J_S7a_band_80_subband_1 E,L,W,R,J 

- Plot the performance metrics
```bash
python CSI_network_metrics_plot.py <'sub-directories, comma-separated'>
```
  e.g., python CSI_network_metrics_plot.py complete_different_E,L,W,R,J_S7a_band_80_subband_1 E,L,W,R,J

Some examples of confusion matrices can be found [in this Google Drive folder](https://drive.google.com/drive/folders/1SilO6VD73Lz8sjZ-KQgFnQ2IKRvggqPg?usp=sharing).

### Parameters
The results of the article are obtained with the parameters reported in the examples. For convenience, the repository also contains two pre-trained networks, i.e., ``single_ant_E,L,W,R,J_network.h5`` and ``single_ant_E,L,W,R,J_C_H_S_network.h5`` respectively for 5-classes and 8-classes classification problems.

### Python and relevant libraries version
Python >= 3.7.7  
TensorFlow >= 2.6.0  
Numpy >= 1.19.5  
Scipy = 1.4.1  
Scikit-learn = 0.23.2  
OSQP >= 0.6.1
Latex
Texlive

