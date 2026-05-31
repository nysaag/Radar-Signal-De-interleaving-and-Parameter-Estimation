# Blind Radar Emitter Deinterleaving using 4-Channel I/Q Data

## Overview

This project implements a completely blind radar emitter deinterleaving pipeline in Python using complex baseband I/Q data collected from a 4-channel antenna array.

The objective is to detect radar pulses, extract pulse descriptor words (PDWs), separate interleaved emitters, and estimate key radar parameters including:

* Carrier Frequency (RF)
* Pulse Repetition Interval (PRI)
* Direction of Arrival (DoA)
* Pulse Width (PW)

The entire pipeline operates without prior knowledge of:

* Number of emitters
* Emitter frequencies
* PRI values
* Direction of arrival

making it a blind emitter analysis system.

---

## Input Data

Each dataset consists of:

* 4-channel complex I/Q baseband data
* Shape: `(4, N)`
* 4 antenna channels
* N time-domain samples

Each complex sample contains:

* In-phase (I) component
* Quadrature (Q) component

which preserve both signal amplitude and phase information.

---

## Processing Pipeline

### 1. Signal Loading

The complex I/Q datasets are loaded and basic statistics are generated:

* Number of channels
* Number of samples
* Power statistics
* Signal visualization

---

### 2. Pulse Detection using Median CFAR

Adaptive thresholding is performed using a robust Median-CFAR detector.

Advantages:

* Noise adaptive
* Resistant to strong outliers
* No manually chosen global threshold

Output:

* Detection mask
* Candidate pulse regions

---

### 3. Pulse Extraction

Detected regions are converted into pulse objects containing:

* Start sample
* End sample
* Time of Arrival (ToA)
* Pulse Width
* Pulse Energy

---

### 4. Pulse Fragment Merging

Closely spaced detections are merged to prevent:

* Pulse fragmentation
* Duplicate detections
* Incorrect PRI estimation

Final pulse counts:

| Dataset | Pulses |
| ------- | ------ |
| A       | 37     |
| B       | 16     |

---

### 5. Carrier Frequency Estimation

Carrier frequency is estimated using:

* FFT
* Peak-centered analysis window
* Quadratic peak interpolation

This improves frequency resolution beyond a single FFT bin.

Estimated RF regions:

#### Dataset A

* ~120 kHz
* ~300 kHz
* ~450 kHz

#### Dataset B

* ~80 kHz
* ~210 kHz
* ~380 kHz

---

### 6. Spatial Feature Extraction

The 4-channel antenna array is used to extract spatial information.

For each pulse:

* Phase01
* Phase12
* Phase23

are computed using adjacent antenna phase differences.

These spatial signatures help discriminate emitters occupying similar RF regions.

---

### 7. Pulse Descriptor Word (PDW) Generation

For every pulse a PDW is generated containing:

```text
ToA
Carrier Frequency
Pulse Width
Pulse Energy
Phase01
Phase12
Phase23
```

---

### 8. RF-Based Emitter Grouping

PDWs are clustered using:

* Carrier Frequency similarity
* Pulse Width similarity

Tracks are formed representing candidate emitters.

Singleton tracks are removed as outliers.

---

### 9. PRI Estimation

For each emitter track:

```text
PRI = median(diff(ToA))
```

where ToA values are sorted within the track.

This produces robust PRI estimates while minimizing the effect of outliers.

---

### 10. Final Emitter Report

For each recovered emitter:

* Carrier Frequency
* PRI
* DoA
* Pulse Width
* Pulse Count

are reported.

---

## Results

### Dataset A

| Emitter | Carrier Frequency | PRI       | DoA     |
| ------- | ----------------- | --------- | ------- |
| E1      | 299.98 kHz        | 23.994 ms | -41.12° |
| E2      | 119.94 kHz        | 23.995 ms | -44.36° |
| E3      | 450.06 kHz        | 19.019 ms | -1.65°  |

---

### Dataset B

| Emitter | Carrier Frequency | PRI       | DoA     |
| ------- | ----------------- | --------- | ------- |
| E1      | 380.03 kHz        | 98.992 ms | -43.33° |
| E2      | 79.99 kHz         | 19.971 ms | -47.80° |
| E3      | 210.04 kHz        | 54.069 ms | -36.78° |

---

## Key Features

* Blind radar pulse detection
* Adaptive Median-CFAR thresholding
* Pulse extraction and cleaning
* Pulse fragment merging
* Carrier frequency estimation
* 4-channel spatial feature extraction
* PDW generation
* RF-based emitter grouping
* PRI estimation
* Final emitter reporting

---

## Technologies Used

* Python
* NumPy
* SciPy
* Matplotlib
* Pandas
* Jupyter Notebook

---

## Notes

The reported DoA values are estimated using inter-element phase differences from the 4-channel antenna array.

Since calibrated antenna geometry was not provided, the DoA values should be interpreted as relative arrival angles useful for emitter discrimination rather than absolute calibrated bearings.

---

## Future Improvements

Possible enhancements include:

* PRI-transform based deinterleaving
* Sequential track growth algorithms
* Density-based clustering (DBSCAN/HDBSCAN)
* MUSIC-based DoA estimation
* Kalman filter based emitter tracking
* Real-time streaming implementation
