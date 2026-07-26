---
layout: page
title: Automated Parcel Dimensioning System
description: Full-stack industrial measurement system built for Delhivery — replacing a paid third-party tool with an in-house system that is several orders of magnitude cheaper and more accurate in live testing.
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

I chose to intern at **Delhivery** because it is India's number one company in logistics technology — the organisation that, over the last 15 years, built the infrastructure that made the entire Indian e-commerce revolution possible. Almost every package ordered online in India today moves through Delhivery's network. Working inside its Automation & Infrastructure (A&I) team meant working on systems that operate at a scale few other companies in the country reach, on problems that directly affect millions of shipments.

## What I Built

I designed and built a complete **industrial parcel measurement system** from the ground up — a production-grade station that measures the length, width, height, and weight of every parcel passing through a warehouse, links each measurement to its barcode, and archives the full transaction automatically, in under 3 seconds, with no manual input from the operator. The system is intended to replace a paid third-party dimensioning tool across Delhivery warehouses; it is several orders of magnitude cheaper to build, and in live warehouse testing it outperformed the commercial tool on measurement accuracy.

The entire system — from hardware drivers to vision pipeline to database to operator interface — was built in one internship. This is not a script or a prototype; it is a multi-subsystem distributed application running in active industrial deployment.

## System Architecture

The system is built as a distributed web application. A set of specialised nodes run on a central server, each responsible for one concern: the **VisionNode** streams depth frames from an Intel RealSense D455 overhead camera; the **ScaleNode** polls a serial-connected industrial weighing scale; the **BarcodeNode** receives AWB scans from a USB scanner. All of these feed into an **OrchestratorNode** — a finite state machine that coordinates the full transaction pipeline, ensuring the right things happen in the right order with no race conditions. The **DatabaseNode** persists every transaction to SQLite. The **HmiBridgeNode** serves live updates to operators over WebSockets and MJPEG video. A separate **FastAPI REST server** exposes transaction history, analytics, and PDF report generation to external systems. The operator interacts with everything through a browser-based HMI — no software to install on the operator's side.

```
[Operator places parcel → scans barcode]
            ↓
[Orchestrator FSM: wait for weight to stabilise]
            ↓
[Operator clears the camera's field of view]
            ↓
[Camera captures depth frame + colour frame]
            ↓
[Dual vision engines measure L × W × H independently]
            ↓
[Automatic routing selects the better engine for this parcel]
            ↓
[Transaction saved: dimensions, weight, barcode, annotated snapshot, point cloud]
            ↓
[HMI updates live — operator sees result instantly]
            ↓
[Ready for next parcel]
```

## Why the Intel RealSense D455

A conventional stereo camera relies on matching visual texture between two frames to compute depth. Warehouse parcels are mostly plain cardboard — large, textureless surfaces that give stereo matching almost nothing to work with. The **RealSense D455 is an active infrared depth camera**: it projects its own structured-light pattern onto the scene, so depth is computed from that pattern rather than from the parcel's surface texture. This makes it robust to low-texture surfaces, variable warehouse lighting, and moving parcels — and its depth accuracy at the ~1.5 m overhead mounting height needed for dimensioning is well within billing-grade tolerance for volumetric weight calculation.

## The Dual Vision Pipeline

A single measurement algorithm does not work well across the full range of parcel shapes that move through a warehouse. The system runs two independent vision engines and automatically routes each capture to the one that fits best.

**Simple Engine** processes the camera's 2D depth image directly: height is computed by depth-differencing against a calibrated empty-scale baseline; length and width come from fitting a minimum-area bounding rectangle to the 2D silhouette. For regular, flat-topped, rigid parcels — the majority of warehouse volume — this engine is the *more accurate* of the two. Its flat-top assumption holds exactly for standard cartons, and it avoids the additional noise sources that accumulate in 3D reconstruction.

**Perception Engine (V7)** reconstructs a full 3D point cloud from the depth frame, segments the parcel using DBSCAN density-based clustering, RANSAC-fits the top surface as a plane, derives orientation via Principal Component Analysis, and fits the footprint from a rotated convex hull. When no reliable flat surface exists, it falls back to a percentile-based height estimate from the point cloud. This engine is built for irregular, non-planar, or soft-sided parcels — bubble wrap, poly bags, oddly shaped items — where the Simple Engine's silhouette-based measurement breaks down.

The router computes an orientation-invariant shape-regularity signal from each parcel's point-cloud covariance structure. The two engines are never blended — one engine's L/W/H is selected entirely for each transaction. This was validated against real warehouse captures with visually confirmed ground truth, including controlled experiments that tilted known-regular boxes through multiple angles to confirm the routing signal was not fooled by orientation.

<div class="row justify-content-center mb-2 mt-4">
  <div class="col-md-10">
    {% include figure.liquid loading="eager" path="assets/img/projects/dws/image11.jpeg" title="Simple Engine — top-down capture with bounding box and live measurements" class="img-fluid rounded z-depth-1" %}
    <p class="text-center text-muted mt-2" style="font-size: 0.85rem;">Simple Engine: top-down camera capture with bounding box and live measurements overlaid</p>
  </div>
</div>

<div class="row justify-content-center mb-4">
  <div class="col-md-10">
    {% include figure.liquid loading="eager" path="assets/img/projects/dws/image3.png" title="Perception Engine — 3D point cloud reconstruction" class="img-fluid rounded z-depth-1" %}
    <p class="text-center text-muted mt-2" style="font-size: 0.85rem;">Perception Engine: 3D point cloud of an irregular parcel — isometric, top-down, and side profile views</p>
  </div>
</div>

## Engineering Highlights

- **Finite state machine orchestrator** with deterministic transaction sequencing — weight stability gating, operator-clear pause, capture, measurement, and database write all happen in a guaranteed order with no race conditions
- **Dual vision pipeline with automatic per-capture routing**, validated against real warehouse data; measurement error reduced to 7–8mm against a known reference
- **Complete calibration system**: auto-calibration from an empty-scale depth baseline, per-axis trim factor derivation from a registered reference box, and a mandatory verification pass before the system accepts any measurements
- **WebSocket-powered HMI** with live dimension readout, MJPEG video stream, measurement logs, calibration logs, and session diagnostics — all served as a single-page web app
- **FastAPI REST API** for external system integration, analytics, and PDF report generation
- **Offline replay tooling** — the full measurement pipeline can be re-run against any saved historical point cloud without access to the physical hardware, enabling root-cause analysis from archived data
- **Production deployment** via systemd with health monitoring, disk retention, and graceful shutdown
- **External sync layer** with automatic failure classification (network vs. API server) and self-recovery via the operator's next barcode scan

## Tech Stack

- **Hardware:** Intel RealSense D455, industrial serial weighing scale, USB barcode scanner
- **Vision:** OpenCV, Open3D, DBSCAN, RANSAC, PCA
- **Backend:** Python, FastAPI, WebSockets, aiohttp, SQLite
- **Frontend:** Browser-based HMI (HTML/JS, WebSocket live updates, MJPEG stream)
- **Deployment:** systemd, headless Linux server
