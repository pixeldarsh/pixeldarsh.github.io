---
layout: page
title: DWS — Automated Dimensioning, Weighing & Scanning
description: Built an in-house parcel dimensioning station at Delhivery, replacing ₹2.5L/year commercial systems at a fraction of the cost.
img: assets/img/projects/dws/image2.png
importance: 3
category:
---

<div class="row justify-content-center mb-4">
  <div class="col-md-10">
    {% include figure.liquid loading="eager" path="assets/img/projects/dws/image2.png" title="DWS Station at Delhivery Warehouse" class="img-fluid rounded z-depth-1" %}
  </div>
</div>

## Overview

During a one-month internship with the **Automation & Infrastructure (A&I) team at Delhivery**, I built and productionised an automated **Dimensioning, Weighing & Scanning (DWS)** station — a system that measures a parcel's length, width, height, and weight, and captures its barcode, all without any manual input from the operator.

The station runs on an **Intel RealSense D455** depth camera, a serial-interfaced weighing scale, and a barcode scanner — packaged into a single-click Windows application. It replaces manual measurement at induction, which is a major source of billing inaccuracy (volumetric weight is billed from L×W×H) and throughput loss.

## Cost Comparison

Commercial DWS systems like V-Measure cost **~₹2.5 lakh upfront plus ₹2.5 lakh/year** in recurring subscription fees. This station cost **~₹70,000–80,000 to build with no recurring cost** — roughly 70% cheaper upfront, and several times cheaper over any multi-year horizon. Extensive live testing at the Delhivery Sector 17 warehouse showed it to be **more accurate than V-Measure** on the same package types.

## The Dual Measurement Engine

A single algorithm doesn't work well for every parcel shape, so I built two independent measurement engines and an automatic router between them.

**Simple Engine** works directly in the camera's 2D image plane: height from depth-differencing against a calibrated baseline, length and width from fitting a bounding rectangle to the 2D silhouette. Fast and highly accurate for regular, flat-topped, rigid parcels.

**Perception Engine** reconstructs a full 3D point cloud, segments the parcel using DBSCAN clustering, fits the top face via RANSAC, and derives orientation via PCA. Built specifically for irregular, non-planar, or soft-sided parcels where the Simple Engine's flat-top assumption breaks down.

The router computes an orientation-invariant shape-regularity signal from each parcel's point cloud and automatically routes to whichever engine fits — validated against real captured data with visually confirmed ground truth.

<div class="row justify-content-center mb-2 mt-4">
  <div class="col-md-10">
    {% include figure.liquid loading="eager" path="assets/img/projects/dws/image11.jpeg" title="Simple Engine — top-down camera capture with bounding box and measurements" class="img-fluid rounded z-depth-1" %}
    <p class="text-center text-muted mt-2" style="font-size: 0.85rem;">Simple Engine: top-down camera capture with bounding box and live measurements overlaid</p>
  </div>
</div>

<div class="row justify-content-center mb-4">
  <div class="col-md-10">
    {% include figure.liquid loading="eager" path="assets/img/projects/dws/image3.png" title="Perception Engine — 3D point cloud of an irregular parcel" class="img-fluid rounded z-depth-1" %}
    <p class="text-center text-muted mt-2" style="font-size: 0.85rem;">Perception Engine: 3D point cloud reconstruction of an irregular parcel — isometric, top-down, and side profile views</p>
  </div>
</div>

## What I Built

- Ported the codebase from Linux to Windows and packaged it as a one-click executable
- Fixed a calibration bug where camera intrinsics weren't being persisted, causing every measurement attempt to fail
- Built a dual-engine router using a validated shape-regularity signal derived from point-cloud covariance
- Reduced total measurement error from ~100mm to 7–8mm using per-axis trim factors and a hybrid measurement approach
- Root-caused a systematic ~56–59mm height error traced to a mismatch between a tape-measured reference and the camera's calibrated depth baseline
- Added stacked-parcel detection, soft-packaging support, and multi-frame depth averaging
- Built offline diagnostic tooling to re-run the shipped measurement code against any saved historical point cloud
- Implemented external API sync with automatic failure classification and recovery via the operator's next barcode scan
- Authored a complete Standard Operating Procedure covering the full operator workflow

## Tech Stack

- **Hardware:** Intel RealSense D455, serial weighing scale, barcode scanner
- **Software:** Python, Open3D (point cloud), DBSCAN, RANSAC, PCA, SQLite, Windows packaging
- **Concepts:** 3D computer vision, depth sensing, PID-style calibration, real-time systems, reliability engineering
