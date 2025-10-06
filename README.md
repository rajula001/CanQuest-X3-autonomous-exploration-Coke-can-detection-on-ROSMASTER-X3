# CanQuest-X3 — Autonomous Exploration & Coke-Can Detection on ROSMASTER X3

A modular ROS system that **maps, localizes, detects Coke cans in 3D, and navigates to them** on the **ROSMASTER X3** platform (Jetson Nano + RGB-D). It combines **YOLOv8** (TensorRT) with the ROS Navigation Stack (move_base + AMCL) and custom nodes for depth fusion and RViz visualization.

---

## Goals
- **Mapping & Navigation:** Build a 2D occupancy map (GMapping), localize with **AMCL**, and navigate safely using **move_base** with layered costmaps.  
- **Real-Time Detection:** Run **YOLOv8** on the Jetson Nano; publish annotated images and bounding boxes.  
- **3D Localization & Nav-to-Object:** Fuse RGB-D depth with detections to estimate 3D can poses and send goals to the navigation stack.

---

## System Overview
1. **Perception (YOLOv8)**  
   - Subscribes to `/camera/rgb/image_raw`.  
   - Publishes `/yolo/bboxes` and `/yolo/annotated_image`.  
   - Optimized with **TensorRT** and deployed in **Docker** (`--net=host`).

<img width="382" height="267" alt="image" src="https://github.com/user-attachments/assets/9f264417-e5ac-4788-8845-2048b17ad638" />
<img width="611" height="339" alt="image" src="https://github.com/user-attachments/assets/47955ea4-ef41-4afd-9ef1-b4b640a1f9aa" />


2. **Depth Fusion & 3D Pose**  
   - For each bbox, sample depth from `/camera/depth/*` and back-project to 3D using camera intrinsics.  
   - Publish TF frames `coke_can_i` (in `odom` / `map`) and **RViz** markers.
<img width="359" height="271" alt="image" src="https://github.com/user-attachments/assets/dfb2275f-542c-4a7f-bcb2-46ee927b66cd" />

3. **Localization & Planning**  
   - **AMCL** localizes the robot on the map.  
   - **move_base** plans a path to the nearest `coke_can_i` frame with obstacle/cost layers and recovery behaviors.
<img width="699" height="450" alt="image" src="https://github.com/user-attachments/assets/b12db5ec-de23-4bb8-a67f-a6c5669a143c" />
<img width="908" height="689" alt="image" src="https://github.com/user-attachments/assets/28587e9f-4c2c-446a-8c11-4a7377540d09" />


4. **Visualization & Debugging**  
   - RViz shows annotated camera feed, can markers, TF tree, global/local costmaps, and planned paths.

---

## ROS Interfaces (key topics)
- **Input:** `/camera/rgb/image_raw`, `/camera/depth/image_raw`, `/tf`, `/odom`  
- **Perception Output:** `/yolo/bboxes` (detections), `/yolo/annotated_image`  
- **3D Pose Output:** TF frames `coke_can_i`, markers on `/visualization_marker`  
- **Navigation:** `/move_base/goal`, `/move_base/status`, `/cmd_vel`

---

## Dataset & Training
- **Model:** YOLOv8 (s/tiny/nano variants depending on speed/accuracy needs).  
- **Data:** Mixed set — Roboflow cans, custom lab captures (including **far-view**), **Isaac Sim** synthetic images, and strong augmentations (scale, illumination, blur).  
- **Rationale:** Domain randomization improves robustness to distance and lighting changes.
<img width="668" height="668" alt="image" src="https://github.com/user-attachments/assets/4097ae2a-22fe-408d-a54c-8bdece0622ee" />

---

## Deployment
- Convert YOLOv8 weights to **TensorRT** engine.  
- Run inside **Docker** on Jetson Nano; the container subscribes/publishes ROS topics and exposes annotated video + detections.  
- The rest of the ROS graph (AMCL, move_base, depth-fusion node) runs on the robot and/or a companion PC.

---

## Performance & Validation
- **Metrics:** detection precision/recall, FPS on Nano, navigation success rate and time-to-reach.  
- **Optimizations:** input resolution tuning, engine precision (FP16/INT8), topic throttling, and node isolation to reduce latency.  
- **Safety:** inflation + obstacle layers in costmaps, conservative recovery behaviors.
<img width="630" height="272" alt="image" src="https://github.com/user-attachments/assets/dc8b560a-f742-49e0-aaae-7cd35521584a" />

---

## Demo
- **Video:** `video/demo.mp4` (download to view; stored with Git LFS)

---

## Notes
- This is a documentation-only repo; source code and training assets are private.
- The architecture and interfaces are provided to guide reproduction on similar platforms.

## License
MIT
