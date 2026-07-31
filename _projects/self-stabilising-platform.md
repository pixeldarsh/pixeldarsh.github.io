---
layout: page
title: Self-Stabilising Platform
description: A real-time stabilisation system built with Raspberry Pi, MPU-6050 IMU, and PID control.
img: assets/img/projects/stabilising-platform-thumb.jpg
importance: 1
category:
youtube_id: M7jIpGaIvR4
---

<div class="row justify-content-center mb-4 g-3">
  <div class="col-12 col-md-6">
    <div class="ratio ratio-16x9">
      <img src="{{ '/assets/img/projects/stabilising-platform-thumb.jpg' | relative_url }}" alt="Self-Stabilising Platform" class="rounded z-depth-1" style="object-fit:cover;width:100%;height:100%;">
    </div>
  </div>
  {% if page.youtube_id %}
  <div class="col-12 col-md-6">
    <div class="ratio ratio-16x9">
      <iframe
        src="https://www.youtube.com/embed/{{ page.youtube_id }}"
        title="Self-Stabilising Platform Demo"
        allowfullscreen
      ></iframe>
    </div>
  </div>
  {% endif %}
</div>

## Overview

This project is a self-stabilising platform built using a **Raspberry Pi**, an **MPU-6050 IMU** (Inertial Measurement Unit), and two servo motors. 
I got inspired to build this when I visited IIT Delhi for a Research exhibition. They had built a magnetic levitation platform and it had a built in control loop for stablization. To understand the concept in practice, I started building this project. 

The platform keeps a surface level regardless of how the base is tilted — using a classic **PID (Proportional-Integral-Derivative) control loop** running in real-time.
 The aim of the project was to demonstrate how feedback control systems used in robotics and aerospace can be applied to create a compact stabilizing mechanism.
The platform used an MPU6050 sensor, which combines a gyroscope and accelerometer to continuously measure tilt angle and motion. This data was processed by a Raspberry Pi, which acted as the main controller. The Raspberry Pi ran a PID algorithm that compared the current tilt angle with the desired balanced position and calculated the correction needed in real time.
To physically stabilize the platform, MG90S metal gear servo motors were used as actuators. Based on the PID output, the servos adjusted the platform’s position to counteract disturbances and restore balance. This control loop repeated continuously at high speed, allowing the platform to react dynamically when pushed or tilted.
## What I Built

- Read roll and pitch angles from the MPU-6050 over I²C
- Implemented a software PID controller in Python to correct for tilt
- Drove two servo motors to counteract disturbances in real time
- Tuned PID gains through iteration to achieve stable, oscillation-free response

## What I Learned

Building this project made abstract physics and engineering concepts tangible. Feedback loops, control theory, and signal processing stopped being textbook ideas and became real problems I had to debug and solve. This was the project that confirmed engineering as the path I want to pursue.

## Tech Stack

- **Hardware:** Raspberry Pi, MPU-6050 IMU, MG90S servo motors
- **Software:** Python, smbus2 (I²C), RPi.GPIO
- **Concepts:** PID control, sensor fusion, real-time systems