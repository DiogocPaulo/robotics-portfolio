---
title: "Autonomous Navigation & Manipulation"
date: 2026-02-26
draft: false
author: "Diogo Paulo"
tags: ["ROS2", "LiDAR", "OpenCV", "Python", "Robotics", "YOLOv5", "Behavior Trees"]
description: "Autonomous Search, Map & Collect Robotics Project"
---


**KTH Royal Institute of Technology | DD2419 Project Course**

> **The Mission:** To design a mobile robot capable of autonomously exploring an unknown workspace, detecting targets and executing a coordinated pick-and-place sequence.

## System Architecture & Integration
The project was a deep integration of modular hardware and ROS 2 software.

<div style="display: flex; gap: 20px; align-items: flex-start; flex-wrap: wrap; margin: 2rem 0;">
    <!-- make the table column a fixed/limited width -->
    <div style="flex: 0 0 360px; max-width: 360px; min-width: 280px;">
        <div style="border-radius: 12px; overflow: hidden; box-shadow: 0 4px 15px rgba(0,0,0,0.08); border: 1px solid rgba(128,128,128,0.2);">
            <!-- card content (removed height:100% so it won't stretch) -->
            <div style="background: #4A90E2; color: white; padding: 12px 16px; font-weight: bold; font-size: 1.1rem; display: flex; align-items: center; gap: 8px;">
                Hardware
            </div>
            <table style="width: 100%; border-collapse: collapse; margin: 0; border: none !important; background: transparent !important;">
                <tr style="background: transparent !important;">
                    <td style="padding: 14px 16px; font-weight: bold; color: inherit; width: 35%; background: transparent !important; border: none !important;">Computer</td>
                    <td style="padding: 14px 16px; color: inherit; opacity: 0.8; background: transparent !important; border: none !important;">Intel NUC i7</td>
                </tr>
                <tr style="background: transparent !important;">
                    <td style="padding: 14px 16px; font-weight: bold; color: inherit; background: transparent !important; border: none !important;">Inference</td>
                    <td style="padding: 14px 16px; color: inherit; opacity: 0.8; background: transparent !important; border: none !important;">NVIDIA GT 1030 (GPU)</td>
                </tr>
                <tr style="background: transparent !important;">
                    <td style="padding: 14px 16px; font-weight: bold; color: inherit; background: transparent !important; border: none !important;">3D Vision</td>
                    <td style="padding: 14px 16px; color: inherit; opacity: 0.8; background: transparent !important; border: none !important;">Intel RealSense D435i</td>
                </tr>
                <tr style="background: transparent !important;">
                    <td style="padding: 14px 16px; font-weight: bold; color: inherit; background: transparent !important; border: none !important;">LiDAR</td>
                    <td style="padding: 14px 16px; color: inherit; opacity: 0.8; background: transparent !important; border: none !important;">RPLIDAR A1 (SLAM)</td>
                </tr>
                <tr style="background: transparent !important;">
                    <td style="padding: 14px 16px; font-weight: bold; color: inherit; background: transparent !important; border: none !important;">Power</td>
                    <td style="padding: 14px 16px; color: inherit; opacity: 0.8; background: transparent !important; border: none !important;">14.8V 4-Cell LiPo</td>
                </tr>
            </table>
        </div>
    </div>

<div style="flex: 1 1 0; min-width: 280px; text-align: center; display: flex; flex-direction: column;">
        <img src="/images/physical-robot.png" style="border-radius: 12px; width: 100%; height: auto; min-height: 350px; object-fit: cover; box-shadow: 0 10px 25px rgba(0,0,0,0.12); border: 1px solid rgba(128,128,128,0.2);" alt="Physical Robot Hardware">
        <p style="font-size: 0.85rem; color: inherit; opacity: 0.7; margin-top: 12px; font-style: italic;">
            Fully integrated mobile platform with a 5-DOF manipulator.
        </p>
    </div>
</div>

## Phase 1: Autonomous Exploration & Mapping
During the first phase, the robot autonomously navigated the workspace to build an occupancy grid while localizing objects and boxes. This process relied on an **Exploration** algorithm, which identifies the boundaries between known and unknown space to determine the next optimal navigation goal.

By integrating 360° LiDAR data with onboard computer, the robot effectively mapped obstacles while maintaining its own position in real-time with only a small drift. Below, we can see a video illustrating this scenario, showcasing live the occupancy grid being built as the robot systematically discovers its environment to ensure every object (cube, sphere or plushie) and box is accounted for before the collection phase begins.

<div style="border-radius: 16px; overflow: hidden; box-shadow: 0 10px 30px rgba(0,0,0,0.1); margin: 2rem 0;">
    <div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden;">
        <iframe 
            src="https://www.youtube.com/embed/c-SVYSbCK9Y?rel=0" 
            style="position: absolute; top: 0; left: 0; width: 100%; height: 100%; border:0;" 
            allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" 
            allowfullscreen>
        </iframe>
    </div>
    <p style="text-align: center; font-size: 0.85rem; color: #666; padding: 10px; margin: 0; background: #f8f9fa;">
        <strong>Exploration Workflow.</strong>
    </p>
</div>

### Perception, Mapping & Reliability
My primary focus was the perception pipeline, particularly for the autonomous exploration phase. A critical challenge was translating raw, noisy sensor data into object to locate in the occupancy grid. To bridge the gap between 3D spatial data and semantic understanding, I developed a multi-stage pipeline:

1.  **LiDAR Clustering**: I first used a LiDAR clustering to segment raw point clouds into distinct spatial groups, identifying potential targets in the 3D environment, as we can see the figures below.
2.  **Hybrid Geometric & Color Filtering**: Then, I design a custom filter so that by analyzing both the geometric dimensions and the color profile of each cluster, the system could reliably classify them as targets (objects), boxes or static environmental obstacles.
3.  **Multi-Frame Voting System**: To achieve a **~95% detection accuracy**, I implemented a temporal reliability filter. Detections only registered as "confirmed" on the global map after appearing consistently across multiple frames, effectively eliminating false positives caused by motion blur, light changes or sensor noise during high-speed mapping.



<div style="display: flex; gap: 15px; margin: 2rem 0; flex-wrap: wrap;">
    <div style="flex: 1; min-width: 280px; text-align: center;">
        <img src="/images/robot-view.png" style="border-radius: 12px; border: 1px solid #ddd; width: 100%;" alt="Camera View">
        <p style="font-size: 0.8rem; color: #777; margin-top: 8px;"><strong>Stage 1:</strong> Real-World View of the Robot looking at the box</p>
    </div>
    <div style="flex: 1; min-width: 280px; text-align: center;">
        <img src="/images/robot-cluster.png" style="border-radius: 12px; border: 1px solid #ddd; width: 100%;" alt="Rviz Cluster">
        <p style="font-size: 0.8rem; color: #777; margin-top: 8px;"><strong>Stage 2:</strong> Rviz Spatial Clustering Output</p>
    </div>
</div>


## Phase 2: Autonomous Collection & Sorting
Once the mapping was finalized, the robot transitioned to the collection phase. Using a **Collection Behavior Tree**, the system planned an optimal sequence to retrieve targets and transport them to their designated sorting boxes. 

This phase required a shift from high-speed navigation to high-precision manipulation. In the following video, we can see the robot executing this mission, using real-time feedback to adjust its approach and successfully perform the pick-and-place operation autonomously.

<div style="border-radius: 16px; overflow: hidden; box-shadow: 0 10px 30px rgba(0,0,0,0.1); margin: 2rem 0;">
    <div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden;">
        <iframe 
            src="https://www.youtube.com/embed/b1CjsHVQaUs?rel=0" 
            style="position: absolute; top: 0; left: 0; width: 100%; height: 100%; border:0;" 
            allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" 
            allowfullscreen>
        </iframe>
    </div>
    <p style="text-align: center; font-size: 0.85rem; color: #666; padding: 10px; margin: 0; background: #f8f9fa;">
        <strong>Collection Workflow.</strong>
    </p>
</div>

### YOLO-Based Precision & Dataset Training
To ensure reliable grasping, I trained a custom YOLOv5 model using a dataset managed via **Roboflow**. This model allowed the system to re-estimate object coordinates in the manipulator frame, correcting for localization drift accumulated during exploration.

<style>
/* --- YOLO & Dataset Section: Theme Adaptive --- */

/* 1. Main Flex Container */
.yolo-showcase-container {
    display: flex; 
    gap: 20px; 
    align-items: center; 
    flex-wrap: wrap; 
    margin: 2rem 0;
}

/* 2. Left Column: Media Box & Caption */
.yolo-media-box {
    flex: 1.5; 
    min-width: 320px;
}

.yolo-image {
    border-radius: 12px; 
    width: 100%; 
    border: 1px solid #ddd; /* Default Light Mode Border */
    box-shadow: 0 4px 15px rgba(0,0,0,0.1);
}

.yolo-caption {
    font-size: 0.8rem; 
    color: #777 !important; /* Muted Grey for Light Mode */
    text-align: center; 
    margin-top: 8px;
}

/* 3. Right Column: Dataset Card */
.dataset-split-card {
    flex: 1; 
    min-width: 280px; 
    background: #fcfcfc; /* Default Light Mode Background */
    padding: 20px; 
    border-radius: 12px; 
    border: 1px solid #eee; /* Default Light Mode Border */
}

/* Metadata Colors for Light Mode */
.dataset-card-title {
    margin-top: 0; 
    font-size: 1rem; 
    color: #333 !important;
}

.dataset-label,
.dataset-count {
    font-size: 0.85rem; 
    color: inherit; 
    opacity: 0.9;
}

/* Progress Bar Backgrounds for Light Mode */
.progress-bar-container {
    height: 8px; 
    background: #eee; 
    border-radius: 4px;
}

/* Progress Bar Fills: Authentic Roboflow Colors */
.progress-fill-train { background: #f39c12; width: 91%; }
.progress-fill-valid { background: #3498db; width: 5%; }
.progress-fill-test { background: #9b59b6; width: 4%; }

/* Unified Progress Fill Style */
.progress-fill-common { height: 100%; border-radius: 4px; }

/* ---------------------------------------------- */
/* --- DARK MODE OVERRIDES (.dark selector) --- */
/* ---------------------------------------------- */

/* Theme assumes .dark class is on body or html */

/* Adaptive Media & Captions */
.dark .yolo-image {
    border-color: rgba(128,128,128,0.2) !important;
}

.dark .yolo-caption {
    color: inherit !important; /* Flips to theme text color */
    opacity: 0.7;
}

/* Adaptive Dataset Card */
.dark .dataset-split-card {
    background: var(--bg-card, rgba(255, 255, 255, 0.05)) !important;
    border-color: rgba(128, 128, 128, 0.2) !important;
}

/* Adaptive Typography */
.dark .dataset-card-title,
.dark .dataset-label,
.dark .dataset-count {
    color: inherit !important; /* Switches to theme text color (white/grey) */
}

/* Progress Bar Background for Dark Mode */
.dark .progress-bar-container {
    background: rgba(255, 255, 255, 0.1) !important;
}
</style>

<div class="yolo-showcase-container">
    <div class="yolo-media-box">
        <img src="/images/yolo-detection.png" class="yolo-image" alt="YOLOv5 Detection Output">
        <p class="yolo-caption">YOLOv5 inference on the arm camera for high-precision grasping.</p>
    </div>

<div class="dataset-split-card">
        <h4 class="dataset-card-title">📊 Dataset Split</h4>
        
<div style="margin-bottom: 15px;">
            <div style="display: flex; justify-content: space-between; margin-bottom: 5px;">
                <span class="dataset-label"><strong>Train Set (91%)</strong></span> 
                <span class="dataset-count">4,515 Images</span>
            </div>
            <div class="progress-bar-container">
                <div class="progress-fill-common progress-fill-train"></div>
            </div>
        </div>

<div style="margin-bottom: 15px;">
            <div style="display: flex; justify-content: space-between; margin-bottom: 5px;">
                <span class="dataset-label"><strong>Valid Set (5%)</strong></span> 
                <span class="dataset-count">233 Images</span>
            </div>
            <div class="progress-bar-container">
                <div class="progress-fill-common progress-fill-valid"></div>
            </div>
        </div>

<div>
            <div style="display: flex; justify-content: space-between; margin-bottom: 5px;">
                <span class="dataset-label"><strong>Test Set (4%)</strong></span> 
                <span class="dataset-count">209 Images</span>
            </div>
            <div class="progress-bar-container">
                <div class="progress-fill-common progress-fill-test"></div>
            </div>
        </div>
    </div>
</div>

<style>
/* --- Challenges & Resources: Theme Adaptive Fix --- */

/* 1. Challenge Cards: White (Light) vs Adaptive (Dark) */
.challenge-card {
    background: #ffffff !important; /* Forced White for Light Mode */
    padding: 20px;
    border-radius: 12px;
    border: 1px solid #eee !important;
    box-shadow: 0 4px 12px rgba(0,0,0,0.05);
    transition: transform 0.3s ease;
}

.dark .challenge-card {
    background: var(--bg-card, rgba(255, 255, 255, 0.05)) !important;
    border-color: rgba(128, 128, 128, 0.2) !important;
    box-shadow: 0 4px 12px rgba(0,0,0,0.2);
}

.challenge-card h4 {
    margin-top: 0; 
    color: #4A90E2 !important; /* Brand Blue */
}

.challenge-card p {
    font-size: 0.95rem;
    color: #666 !important; /* Grey for Light Mode */
}

.dark .challenge-card p {
    color: inherit !important; /* White/Grey for Dark Mode */
    opacity: 0.85;
}

/* 2. Resources Box: Light Grey (Light) vs Adaptive (Dark) */
.resource-box {
    flex: 1; 
    min-width: 280px; 
    background: #f8f9fa !important; /* Light Grey for Light Mode */
    padding: 20px; 
    border-radius: 12px;
    border: 1px solid #eee !important;
}

.dark .resource-box {
    background: var(--bg-card, rgba(255, 255, 255, 0.05)) !important;
    border-color: rgba(128, 128, 128, 0.2) !important;
}

.resource-box li {
    color: #666 !important;
}

.dark .resource-box li {
    color: inherit !important;
    opacity: 0.9;
}

/* 3. Software Stack: Solid Blue Brand Anchor */
.stack-box-blue {
    flex: 1.5; 
    min-width: 300px; 
    background: #4A90E2 !important; 
    color: white !important; 
    padding: 20px; 
    border-radius: 12px; 
    display: flex; 
    flex-direction: column; 
    justify-content: center; 
    box-shadow: 0 4px 15px rgba(74, 144, 226, 0.2);
}

.dark .stack-box-blue {
    box-shadow: 0 4px 15px rgba(0, 0, 0, 0.4); /* Deeper shadow for dark mode */
}
</style>

---
## Challenges & Takeaways
Developing a robust pipeline meant moving beyond "perfect" simulations to handle the unpredictable nature of hardware.

<div style="display: grid; grid-template-columns: repeat(auto-fit, minmax(300px, 1fr)); gap: 20px; margin: 2rem 0;">
    <div class="challenge-card">
        <h4>Localization Drift</h4>
        <p>Mitigated using <strong>Inflation Layers</strong> in the costmap and implementing a secondary verification pass via the manipulator camera.</p>
    </div>

<div class="challenge-card">
        <h4>Perception Reliability</h4>
        <p>Achieved a <strong>95% Detection Rate</strong> by using temporal voting to filter out noise and motion blur during exploration.</p>
    </div>
</div>

---

## Project Resources & Stack

<div style="display: flex; gap: 20px; flex-wrap: wrap; margin-top: 2rem;">
    <div class="resource-box">
        <ul style="list-style: none; padding: 0; margin: 0;">
            <li style="margin-bottom: 12px;">
                <strong>💻 Source Code:</strong> <a href="https://github.com/DiogocPaulo/project-workspace-dd2419" style="color: #4A90E2; text-decoration: none; font-weight: bold;">View on GitHub</a>
            </li>
            <li style="margin-bottom: 12px;">
                <strong>📊 Managed Dataset:</strong> Managed via <a href="#" style="color: #4A90E2; text-decoration: none; font-weight: bold;">Roboflow</a> (4,957 Images)
            </li>
        </ul>
    </div>

<div class="stack-box-blue">
        <p style="margin: 0; font-size: 0.85rem; text-transform: uppercase; letter-spacing: 1px; opacity: 0.9;">Software Stack</p>
        <p style="margin: 8px 0 0 0; font-size: 1.1rem; font-weight: bold;">
            ROS 2 • YOLOv5 • OpenCV • Nav2 • BehaviorTree.CPP
        </p>
    </div>
</div>