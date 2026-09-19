# 📡 UWB Localization & Sensor Fusion

Low-cost indoor localization system for multi-robot applications using **ESP32 UWB boards**, **DWM3000 transceivers**, **IMU sensors** and **ROS 2 / Micro-ROS**.

The system combines **UWB trilateration** with **IMU data through a Kalman Filter**, while using the **Channel Impulse Response (CIR)** of the DW3000 to detect potentially degraded **NLOS (Non-Line-of-Sight)** measurements and reduce their influence on the localization estimate.

<img width="966" height="582" alt="Capture d&#39;écran 2026-09-19 144622" src="https://github.com/user-attachments/assets/d56bd0e6-a7c3-418b-9b7d-3c7851d0beb4" />
Visualization of the finale of the sytem working.

---

## 🚀 Overview

The objective of this project was to develop a low-cost localization system for robots operating in **GPS-denied indoor environments**.

The system uses:

* 📡 **UWB** for precise distance measurements
* 📍 **Trilateration** to estimate the robot's 2D position
* 🧭 **IMU** for high-rate motion information
* 🧠 **Kalman Filter** to fuse UWB and IMU measurements
* 📶 **CIR-based NLOS detection** to identify unreliable UWB measurements
* 🤖 **Micro-ROS / ROS 2** for communication, processing and visualization

---

## 🏗️ System Architecture

```text
             ┌─────────────────┐
             │   UWB Anchors   │
             │ ESP32 + DWM3000 │
             └────────┬────────┘
                      │
                 DS-TWR ranging
                      │
                      ▼
             ┌─────────────────┐
             │   UWB Tag       │
             │ ESP32 + DWM3000 │
             └────────┬────────┘
                      │
          ┌───────────┴───────────┐
          │                       │
      UWB ranges                CIR data
          │                       │
          ▼                       ▼
   ┌──────────────┐       ┌──────────────┐
   │ Trilateration│       │ NLOS Detection│
   └──────┬───────┘       └──────┬───────┘
          │                       │
          └──────────┬────────────┘
                     │
                     ▼
              ┌─────────────┐
              │ Kalman Filter│
              │  UWB + IMU   │
              └──────┬──────┘
                     │
                     ▼
                ┌─────────┐
                │  ROS 2  │
                │ /Micro-ROS│
                └─────────┘
```

---

## 📡 UWB Localization

The UWB system uses **Double-Sided Two-Way Ranging (DS-TWR)** between the mobile tag and fixed anchors.

Three anchors are used for 2D trilateration. Each measured distance defines a circle around an anchor, allowing the robot's position to be estimated from the intersection of the three measurements.

Per-device **antenna-delay calibration** was performed to reduce systematic ranging bias.

---

## 🚧 NLOS Mitigation

Indoor environments can introduce multipath and **Non-Line-of-Sight (NLOS)** conditions, resulting in biased UWB measurements.

The **DW3000 Channel Impulse Response (CIR)** is used as an indicator of signal quality and potential NLOS conditions.

When a measurement is identified as potentially degraded, its contribution to the localization estimate is reduced rather than treating it as equally reliable as a clear line-of-sight measurement.

This provides a more robust localization estimate in indoor environments affected by reflections and obstructions.

---

## 🧭 UWB + IMU Sensor Fusion

The UWB trilateration output is fused with IMU measurements using a **Kalman Filter**.

The IMU provides high-rate motion information, while UWB provides an absolute position reference that limits accumulated inertial drift.

```text
             IMU
              │
              ▼
        ┌─────────────┐
        │             │
        │   Kalman    │──────► Filtered Position
        │   Filter    │
        │             │
        └──────▲──────┘
               │
        UWB Trilateration
               │
        NLOS-dependent
         measurement
           weighting
```

The filter therefore combines:

**High-rate IMU motion information + absolute UWB position measurements + NLOS reliability information**

---

## 🤖 ROS 2 / Micro-ROS

The ESP32 communicates with a ROS 2 system through **Micro-ROS over Wi-Fi**.

Main topics include:

| Topic                | Description                            |
| -------------------- | -------------------------------------- |
| `/uwb/distances`     | UWB distances to the anchors           |
| `/uwb/nlos`          | NLOS / measurement quality information |
| `/uwb/anchors_age`   | Time since each anchor responded       |
| `/imu/data`          | IMU measurements                       |
| `/uwb/odom`          | UWB-based odometry                     |
| `/odometry/filtered` | Filtered localization estimate         |

The ROS 2 pipeline handles trilateration, sensor fusion and visualization.

---

## 🔧 Hardware

* **ESP32 UWB boards**
* **Qorvo DWM3000 / DW3000**
* **IMU**
* 3 × UWB anchors
* Wi-Fi network

---

## 💻 Software & Technologies

**Programming:**
C/C++ · Python

**Embedded:**
ESP32 · DWM3000 · PlatformIO

**Robotics:**
ROS 2 · Micro-ROS · Sensor Fusion · Localization

**Algorithms:**
DS-TWR · Trilateration · Kalman Filtering · NLOS Detection

**Communication:**
UWB · Wi-Fi · SPI

---

## 📊 Results

After antenna-delay calibration and filtering, the system achieved approximately **10–20 cm indoor positioning accuracy** in the tested environment.

Key observations:

* Three anchors removed the ambiguity encountered with two-anchor trilateration.
* Calibration significantly reduced systematic ranging bias.
* UWB + IMU fusion produced a smoother localization estimate.
* CIR-based NLOS detection allowed degraded UWB measurements to have a reduced influence on the filter.
* Sequential DS-TWR limited the overall update rate.
* Anchor geometry had a significant impact on localization accuracy.

The system was successfully demonstrated as an end-to-end pipeline from **UWB ranging → trilateration → NLOS detection → IMU/UWB sensor fusion → ROS 2 visualization**.

---

## ⚠️ Limitations/Possible Improvements

The main limitations of the current implementation are:

* Limited number of anchors
* Sequential ranging between anchors
* Indoor multipath and NLOS effects
* Calibration uncertainty
* Limited ground-truth instrumentation
* Trade-off between filtering and responsiveness

---

## 🔮 Future Work

Potential improvements include:

* Increasing the number of anchors
* Improving CIR-based NLOS classification
* Further tuning the Kalman Filter
* Validation against a motion-capture or other ground-truth system
* Integration with wheel odometry
* Integration with LiDAR / SLAM
* Extension toward multi-robot peer-to-peer localization

---

## 📚 References

The project is based on research and technical work covering UWB localization, trilateration, sensor fusion and NLOS mitigation.

See the project report for the complete bibliography and theoretical background.

---

## 👤 Author

**Mateo Gomes**

Engineering Degree — ENSEA
M.S. Electrical Engineering — Illinois Institute of Technology
