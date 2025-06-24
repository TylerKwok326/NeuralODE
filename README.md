# NeuralODE

## Data Preprocessing and Quality Assessment
The original dataset was processed by selecting one representative subject (s01) with 180 valid trials and 5 OFC electrodes for initial model development. The neural data consisted of 3,001 timepoints per trial, spanning -1 to +2 seconds relative to game presentation, sampled at 1 kHz.

### Data Quality Assessment: 

A comprehensive quality control pipeline was implemented including: (1) detection of NaN and infinite values, (2) signal quality assessment per electrode through variance and range analysis, (3) outlier detection using z-score thresholding (>5 standard deviations), (4) temporal consistency verification, and (5) behavioral-neural alignment validation.

Outlier Handling: Given the presence of 7,916 extreme outliers across electrodes (1,352-3,918 per electrode), conservative clipping at 6 standard deviations was applied to preserve signal integrity while removing artifacts.

Behavioral-Neural Alignment: A mismatch between 200 behavioral trials and 180 neural trials was addressed by truncating the behavioral data to match the neural recordings, ensuring proper temporal alignment.

### Neural ODE Architecture
A compact Neural ODE architecture was designed and optimized for the 5-electrode OFC dataset:

## ODE Function: 

The dynamics were parameterized by a neural network taking latent state (8 dimensions), behavioral context (4 dimensions), and time as inputs, outputting the derivative of the latent state:

dz/dt = f(z(t), behavioral_context, t; θ)

### Encoder-Decoder Framework:

- Encoder: Mapped 5-electrode neural observations to latent initial conditions using a 2-layer MLP with variational parameterization (mean and log-variance)
- Decoder: Reconstructed 5-electrode neural activity from latent states using a 2-layer MLP with dropout regularization
- Behavioral Encoder: Processed 4 behavioral features (choice type, reaction time, trial progression, outcome) into contextual embeddings
Model Architecture: The complete model contained 5,201 parameters, with 32 hidden units in the ODE function and 16-32 hidden units in encoder/decoder networks.

### Behavioral Feature Engineering
Four key behavioral features were extracted for each trial:

- Choice Type: Binary indicator (0=safebet, 1=gamble)
- Reaction Time: Normalized to [0,1] range based on 2-second maximum
- Trial Progression: Normalized trial number (0-1)
- Outcome Feature: Binary outcome indicator (0=loss/would-have-lost, 1=win/would-have-won)

## Training Procedure
Data Splitting: An 80-20 train-validation split (144 training, 36 validation trials) was used with random assignment.

Loss Function: A composite loss was employed combining:

Reconstruction Loss: MSE between predicted and actual neural trajectories
KL Divergence: Regularization of latent space (weight=0.1)
Prediction Loss: MSE on randomly masked trajectory portions (weight=0.5)
Optimization: Training used Adam optimizer (learning rate=1e-3, weight decay=1e-5) with gradient clipping (max norm=1.0) for stability. Early stopping was implemented with patience=10 epochs.

Temporal Subsampling: For computational efficiency, timestamps were subsampled by factor of 10 (300 points instead of 3,001) during training while preserving temporal resolution.
