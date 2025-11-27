# Golf Swing Pose Analysis
### Pipeline Prototype
This repository summarizes a project I worked on during my internship at **Newruns.** A B2B client in the golf service industry (name not disclosed) requested an initial demonstration pipeline to evaluate the technical feasibility of converting a single 2D camera video into a 3D golf swing analysis.

Even though this was a Proof of Concept (PoC) project, it was carried out under a formal contract with a client company, so the actual code cannot be disclosed. Instead, I'm sharing the overall pipeline, key technical decisions, and challenges identified during development.

## Project Overview
### Objectives 
The goal of this project was to build a prototype pipeline capable of generating 3D golf swing poses from a single 2D video and comparing them against a professional golfer's swing. A B2B client aimed to introduce a new feature in their service where users could record their swing on mobile and receive feedback based on 3D pose comparison with professional reference swings.

### Scope of the PoC
The project focused on validating the working reconstruction pipeline itself, rather than model training, data collection, or production level optimization such as generating precise swing advice. 

## Pipeline Overview
### 1. Video Input
A user's golf swing video (single 2D camera from mobile) is loaded.

\* In this project, the client provided the sample input videos directly.

### 2. Swing Pose Detection (SwingNet)
SwingNet analyzes the full video to detect **eight key swing poses** and returns the corresponding frame indices. These indices are used to cut out unnecessary parts of the video and keep only the actual swing segment.

### 3. 2D Pose Estimation (HRNet)
HRNet estimates 2D keypoints for every frame within the swing segment identified by SwingNet.

### 4. 3D Pose Reconstruction (MHFormer)
The entire 2D key point sequence is passed into MHFormer, whihc reconstructs a temporally consistant 3D pose sequence using transformer based motion inference.

### 5. Event Based 3D Pose Extraction
From the reconstructed 3D sequence, the frmaes corresponding to the eight SwingNet events are extracted. These 3D poses are then used to compare the user's swing with professional reference swings.

## Tech Stack
**Models**: SwingNet (golf swing event detection), HRNet (2D human pose estimation), MHFormer (transformer based 3D reconstruction)
**Language & Libraries**: Python, PyTorch, OpenCV, NumPy, matplotlib
**Tools**: GitHub, HuggingFace transformer, KT cloud (VScode)

## My Key Contribution
I worked primarily on model research, defining the pipeline structure, and coordinating the interaction between models.

### 1. Defining What the Pipeline Neede to Detect
- Identified that SwingNet is essential for detecting key swing events but is not sufficient alone for comparison with professional swings.
- Established the rationale for combining SwingNet with HRNet and MHFormer, since pose models alone cannot determine when swing events occur.

### 2. Model Integration
- Integrated 3 pretrained models (SwingNet, HRNet, MHFormer) into a single working pipeline by aligning their required data formats and connecting them.

### 3. Evaluating Technical Feasibility
- Reviewed the outputs of the entire pipeline and identified technical constraints that impact single camera 3D reconstructuion, such as depth ambiguity and inhernet limitations of human pose models.
- Summarized these findings for the client, clarifying what the PoC can realistically support and what would require additional development.


## Challenges & Solutions
### Challenge 1: Video Length & Irrelevant Frames
**Problem**: \
User videos vary in length and often include unnecessarry frames, which increases processing time and lowers performance and effeciency.

**Solution**: \
Used SwingNet to detect the eight key swing events, then combined SwingNet's frame indices with the video's FPS to compute their timestamps.

Only the segemnt between the first and last swing events was extracted for further processing.

### Challenge 2: Low MHFormer Performance with Single Frame Input
**Problem**: \
SwingNet provides only the event frames, but MHFormer relies on continuous temporal sequences. Therefore, when only single key frames are used, MHFormer loses its temporal contect and produces inaccurate 3D poses.

**Solution**:\
Processed the entire swing segment (from the first event to the last) through HRNet and MHFormer, and then used the event timestamps to extract the specific 3D poses for each swing phase.


## Findings
### Limitations
#### 1. Single Camera Depth Ambiguity
Using only a single 2D camera makes depth difficult to infer, leading to potential inaccuracies in 3D reconstruction.

#### 2. No Golf Club Tracking
The pose estimation models (HRNet, MHFormer) detect only human joints and do not capture the golf club, limiting swing analysis.

## Future Improvements
- **Camera Guidelines for Consistency**
    Define recommended camera placement and angle to reduce depth ambiguity and improve 3D reconstruction stability.

- **Club Detection Model**
    Integrate an object detection model, such as YOLO, to detect the clubhead and shaft, enabling club path and shaft angle analysis.

- **Fine Tuning for Golf Specific Poses**
    Train and fine tune pose models on golf specific datasets to improve accuracy.