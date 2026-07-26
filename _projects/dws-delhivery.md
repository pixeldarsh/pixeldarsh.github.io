---
layout: page
title: DWS — Automated Dimensioning, Weighing & Scanning
description: Internally developed parcel dimensioning system for Delhivery warehouses — several orders of magnitude cheaper to build than third-party alternatives, and more accurate in testing.
img: assets/img/projects/dws/image3.png
importance: 3
category:
---

<div class="row justify-content-center mb-4">
  <div class="col-md-10">
    {% include figure.liquid loading="eager" path="assets/img/projects/dws/image2.png" title="DWS Station at Delhivery Warehouse" class="img-fluid rounded z-depth-1" %}
  </div>
</div>

## Why Delhivery

I chose to intern at **Delhivery** because it is India's number one company in logistics technology — the organisation that, over the last 15 years, built the infrastructure that made the entire Indian e-commerce revolution possible. Almost every package ordered online in India today moves through Delhivery's network. Working inside its Automation & Infrastructure (A&I) team meant working on systems that operate at a scale few other companies in the country reach, and on problems that directly affect millions of shipments.

## Overview

A **DWS (Dimensioning, Weighing & Scanning)** station measures a parcel's length, width, height, and weight, and captures its barcode — automatically, without the operator entering any values manually. The operator places the parcel on the scale and scans its barcode; the system does the rest within seconds. This directly replaces slow, manual, error-prone measurement at induction, which is a significant source of billing inaccuracy (volumetric weight is billed from L×W×H) and throughput loss at warehouse scale. **The system built here is intended for deployment across Delhivery warehouses**, replacing a paid third-party tool with an internally owned and maintained alternative that is several orders of magnitude cheaper to build and, in live testing, more accurate.

## System Architecture

The system is a distributed web application, not a desktop program. A **host node** at the station collects everything locally — depth frames from the Intel RealSense D455, weight readings from a serial-interfaced scale, barcode scans, and captured images — and forwards them over the network to a **backend server**, which performs all the heavy processing: point-cloud reconstruction, dimension computation, database writes, and API sync. The operator interacts with the station entirely through a **browser-based HMI** served by the backend — no separate install is needed on the operator's side. All transaction, calibration, and error history is stored in a local SQLite database on the server, with an external API sync layer that classifies connectivity failures and recovers automatically via the operator's next barcode scan.

## Why the Intel RealSense D455 (and not a stereo camera)

A conventional stereo RGB camera relies on matching visual texture between two frames to compute depth. In a warehouse environment, plain cardboard parcels have almost no texture — large surfaces of uniform brown or white give stereo matching almost nothing to work with, producing noisy or missing depth in exactly the regions that matter most for dimensioning. The **RealSense D455 is an active infrared depth camera**: it projects its own structured light pattern onto the scene, so depth is computed from that pattern rather than from the parcel's own surface texture. This makes it robust to the low-texture surfaces, variable lighting, and fast-moving conveyors typical of a warehouse — and its depth accuracy at the ~1 metre mounting height needed for overhead dimensioning is well within the tolerance required for billing-grade volumetric weight computation.

## The Dual Measurement Engine

A single algorithm doesn't work well for every parcel shape. The system runs two independent measurement engines and automatically routes each capture to the one that fits best.

**Simple Engine** works directly in the camera's 2D image plane: height from depth-differencing against a calibrated empty-scale baseline, length and width from fitting a minimum-area rectangle to the 2D silhouette. It is fast, low-latency, and — for regular, flat-topped, rigid parcels like standard cardboard cartons — the *more accurate* of the two engines. Its flat-top assumption holds exactly when the parcel is genuinely flat-topped, so it avoids the additional sources of error (point-cloud noise, segmentation edge effects, RANSAC fitting variance) that the Perception Engine accumulates. If every parcel were a regular box, the Simple Engine alone would be sufficient.

**Perception Engine** reconstructs a full 3D point cloud from the depth frame, segments the parcel using connected-components and DBSCAN clustering, RANSAC-fits the top face as a plane, derives orientation via PCA, and fits the footprint from a rotated convex hull. When no reliable flat surface exists it falls back to a percentile-based height estimate. This engine is built specifically for irregular, non-planar, or soft-sided parcels — bubble wrap, poly bags, oddly shaped items — where the Simple Engine's flat-top assumption breaks down and its silhouette-based length/width measurement is thrown off by surface curvature or droop.

The router computes an orientation-invariant shape-regularity signal from each parcel's point-cloud covariance structure and routes automatically — validated against real captured data with visually confirmed ground truth, including a controlled test that tilted a known-regular box through several angles to confirm the signal stays "regular" under tilt rather than being fooled by orientation.

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

- Adapted the codebase to the production environment and fixed a calibration bug where camera intrinsics weren't being persisted, causing every measurement attempt to fail
- Built a dual-engine router using a validated shape-regularity signal derived from point-cloud covariance
- Reduced total measurement error from ~100mm to 7–8mm using per-axis trim factors and a hybrid measurement approach
- Root-caused a systematic ~56–59mm height error traced to a mismatch between a tape-measured reference and the camera's calibrated depth baseline
- Added stacked-parcel detection, soft-packaging support, and multi-frame depth averaging
- Built offline diagnostic tooling to re-run the shipped measurement code against any saved historical point cloud
- Implemented external API sync with automatic failure classification and recovery via the operator's next barcode scan
- Authored a complete Standard Operating Procedure covering the full operator workflow

## Tech Stack

- **Hardware:** Intel RealSense D455 (active IR depth), serial weighing scale, barcode scanner
- **Software:** Python, Open3D (point cloud), DBSCAN, RANSAC, PCA, SQLite, browser-based HMI
- **Concepts:** 3D computer vision, active infrared depth sensing, distributed systems, reliability engineering
