---
layout: page
title: Mars Rover with Autonomous Obstacle Navigation and Avoidance Classification
description: A low-cost Mars rover prototype that autonomously classifies obstacles and plans routes using stereo vision, IMU, and ultrasonic sensors.
img: assets/img/projects/mars-rover-thumb.webp
importance: 4
category:
youtube_id: _6h9HucQGHs
---

<div class="row justify-content-center mb-4 g-3">
  <div class="col-12 col-md-6">
    <div class="ratio ratio-16x9">
      <img src="{{ '/assets/img/projects/mars-rover-thumb.webp' | relative_url }}" alt="Mars Rover" class="rounded z-depth-1" style="object-fit:cover;width:100%;height:100%;">
    </div>
  </div>
  {% if page.youtube_id %}
  <div class="col-12 col-md-6">
    <div class="ratio ratio-16x9">
      <iframe
        src="https://www.youtube.com/embed/{{ page.youtube_id }}"
        title="Mars Rover Demo"
        allowfullscreen
      ></iframe>
    </div>
  </div>
  {% endif %}
</div>

## Overview

A 70cm × 50cm Mars rover prototype built on the rocker-bogie suspension mechanism — inspired by NASA JPL rovers — capable of navigating rough terrain and autonomously classifying obstacles as passable or impassable. The rover was built for ₹56,950 (~US$590), with autonomous navigation, stereo vision, IMU, and a 6-DoF arm.

The rover is controlled by a Raspberry Pi 3B+ running Ubuntu and communicating over Wi-Fi with a laptop that handles perception and route planning. All 6 wheels are independently driven; the 4 front and rear wheels have independent steering (Ackermann geometry). A pan-tilt stereo camera, MPU6050 IMU, and 4 ultrasonic sensors provide sensing. A 6-DoF robotic arm with a wrist camera enables pick-and-place.

## Autonomous Navigation

The navigation stack uses the SLAM RTAB architecture, combining data from the stereo camera, IMU, and ultrasonic sensors to build an odometry estimate and a terrain map. The rover follows a "Gather → Re-Plan → Move → Stop → Gather" loop: because obstacle measurement accuracy degrades beyond 3m, the rover re-plans at each step as new terrain comes into view.

The LAS2-S stereo model was chosen over SGBM for significantly better depth coverage on textureless surfaces — cardboard-coloured terrain, repetitive textures — which SGBM struggles with. 3D point cloud estimation achieves less than 10% dimensional variance from ground truth. Servo jitter in the pan-tilt assembly degrades depth and terrain mapping; a dedicated IMU-based PID control loop compensates for this in real time.

## Mechanical Design

The rocker-bogie suspension passively keeps all six wheels in ground contact over severely uneven terrain, distributing pressure evenly to prevent sinking in soft surfaces. The chassis is built from aluminium extrusions with 3D-printed connectors. Rubber strips applied to the 3D-printed wheels improve grip on steep slopes — demonstrated safe navigation on slopes up to 40° on smooth glass-like surfaces, and over step obstacles up to 80% of wheel diameter.

## Robotic Arm

A 6-DoF arm mounted on the front of the chassis uses a wrist-mounted camera for object detection and pick-and-place. Automated detection identifies target objects; the arm positions itself based on visual feedback.

## Tech Stack

- **Hardware:** Raspberry Pi 3B+, AR0144 stereo camera, MPU6050 IMU, HC-SR04 ultrasonic sensors, Feetech servo motors (16 total)
- **Perception:** SLAM RTAB, LAS2-S stereo depth, 3D point cloud estimation
- **Software:** Ubuntu, Python, ROS 2 (Docker on laptop)
- **Mechanical:** Rocker-bogie suspension, Ackermann steering, 3D-printed wheels with rubber grip
