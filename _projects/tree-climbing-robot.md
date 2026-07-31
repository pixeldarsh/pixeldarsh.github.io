---
layout: page
title: Tree Climbing Robot
description: A pneumatic inchworm-inspired robot that autonomously climbs tree trunks using form closure and sequential cylinder actuation.
img: assets/img/projects/tree-climbing-robot-thumb.jpg
importance: 2
category:
youtube_id: kiXCCa_L0Ws
---

<div class="row justify-content-center mb-4">
  <div class="col-md-10">
    {% include figure.liquid loading="eager" path="assets/img/projects/tree-climbing-robot-thumb.jpg" title="Tree Climbing Robot" class="img-fluid rounded z-depth-1" %}
  </div>
</div>

{% if page.youtube_id %}
<div class="row justify-content-center mb-4">
  <div class="col-md-10">
    <div class="ratio ratio-16x9">
      <iframe
        src="https://www.youtube.com/embed/{{ page.youtube_id }}"
        title="Tree Climbing Robot Demo"
        allowfullscreen
      ></iframe>
    </div>
  </div>
</div>
{% endif %}

## Overview

When I was visiting the Andamans, I saw some farmers harvesting coconuts. It seemed intriguing and very tedious and dangerous to me. We had wondered as family if there was a better way to do the whole harvesting. When the opportunity to work on this project arose at the YTS summer school, I immediately decided to work on this project. Pneumatics were selected over electric motors to achieve a high torque-to-weight ratio while remaining highly cost-effective.

## Design Evolution

**Wheel-based design:** Initially explored a wheel-based design that provided force on the tree from a 360° angle, but this approach lost traction and was abandoned.

**Force closure — clamping mechanism:** Next, we looked at a clamping mechanism based on the inchworm climbing principle: one clamp grips while the other releases, with a vertical pneumatic cylinder driving vertical movement. This led to a Y-clamp design using 3 pneumatic cylinders. It was dropped due to budget constraints — 3 cylinders were too costly.

**Form closure — final approach:** We shifted to form closure using just 2 vertical pneumatic cylinders.

- **First prototype (5.5mm MDF):** Failed because the mechanism maintained a continuous, uniform grip on the trunk, preventing the alternating release needed to climb.
- **Second prototype (4mm MDF):** Switched to thinner, more flexible sheets. We also discovered that adding weight to the back side of the plate (farthest from the trunk) caused it to lock much more effectively.
- **Electronics placement:** When building the automation circuit — ESP32, two solenoids, air hoses, and a relay — we strategically placed the components on the back side to naturally provide that locking weight.
- **Sequenced actuation:** Extending both pneumatic cylinders simultaneously failed to create the angle required for the self-locking mechanism. We shifted to a sequence where one cylinder extends first, followed by the second after a short delay — this produces the correct geometry for climbing.
