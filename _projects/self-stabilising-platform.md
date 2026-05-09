---
layout: page
title: Self-Stabilising Platform
description: A real-time stabilisation system built with Raspberry Pi, MPU-6050 IMU, and PID control.
img: assets/img/projects/stabilising-platform-thumb.jpg
importance: 1
category:
---

<div class="row justify-content-center mb-4">
  <div class="col-md-10">
    {% if page.youtube_id %}
    <div class="ratio ratio-16x9 mb-3">
      <iframe src="https://www.youtube.com/embed/{{ page.youtube_id }}" title="Project demo" allowfullscreen></iframe>
    </div>
    {% endif %}
  </div>
</div>

<div class="row justify-content-center mb-4">
  <div class="col-md-10">
    {% include figure.liquid loading="eager" path="assets/img/projects/stabilising-platform-thumb.jpg" title="Self-Stabilising Platform" class="img-fluid rounded z-depth-1" %}
  </div>
</div>

## Overview

This project is a self-stabilising platform built using a **Raspberry Pi**, an **MPU-6050 IMU** (Inertial Measurement Unit), and two servo motors. The platform keeps a surface level regardless of how the base is tilted — using a classic **PID (Proportional-Integral-Derivative) control loop** running in real-time.

## What I Built

- Read roll and pitch angles from the MPU-6050 over I²C
- Implemented a software PID controller in Python to correct for tilt
- Drove two servo motors to counteract disturbances in real time
- Tuned PID gains through iteration to achieve stable, oscillation-free response

## What I Learned

Building this project made abstract physics and engineering concepts tangible. Feedback loops, control theory, and signal processing stopped being textbook ideas and became real problems I had to debug and solve. This was the project that confirmed engineering as the path I want to pursue.

## Tech Stack

- **Hardware:** Raspberry Pi, MPU-6050 IMU, SG90 servo motors
- **Software:** Python, smbus2 (I²C), RPi.GPIO
- **Concepts:** PID control, sensor fusion, real-time systems
