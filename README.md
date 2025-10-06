# CanQuest-X3 — Autonomous Exploration & Coke-Can Detection on ROSMASTER X3

A modular ROS system that **maps, localizes, detects Coke cans in 3D, and navigates to them** on the **ROSMASTER X3** platform (Jetson Nano + RGB-D). It combines **YOLOv8** (TensorRT) with the ROS navigation stack (move_base + AMCL) and custom nodes for depth fusion and RViz visualization.

![Overview](docs/overview.png)

## Goals
- **Goal 1 — Mapping & Navigation:** Build a 2D map (GMapping), localize with **AMCL**, and navigate safely with **move_base** and costmaps.  
- **Goal 2 — Real-Time Detection:** Run **YOLOv8** on the Jetson Nano; publish annotated images and bboxes.  
- **Goal 3 — 3D Localization & Nav-to-Object:** Fuse **RGB-D** with detections to compute 3D can poses (TF frames) and send goals to the nav stack.

## Hardware & Software
- **Robot:** ROSMASTER X3 (mobile base, IMU) • **Compute:** Jetson Nano  
- **Sensors:** RGB-D camera (RealSense/OAK-D)  
- **ROS 1 Melodic:** `move_base`, AMCL, `costmap_2d`, TF • **Tools:** RViz  
- **Perception:** YOLOv8 (Ultralytics) fine-tuned on combined dataset (Roboflow + custom far-view + Isaac Sim + heavy augmentation)  
- **Deployment:** TensorRT engine inside **Docker** (`--net=host`), subscribing to `/camera/rgb/image_raw`, publishing `/yolo/bboxes` & `/yolo/annotated_image`.

![HW/SW Stack](docs/hw_sw_stack.png)
![YOLOv8 → TensorRT → Docker](docs/yolo_trt_docker.png)

## Custom ROS Nodes (high level)
- **`yolo_ros_inference.py`** — runs YOLOv8 (TensorRT), publishes bbox strings & annotated images.  
- **`coke_can_depth_processor.py`** — uses `/camera/depth/*` + camera intrinsics to back-project bbox centers → **3D points**; publishes TF frames `coke_can_i` in `odom`.  
- **`CokeCanMarkerPublisher`** — displays up to 10 red sphere markers in RViz at the TF-reported can poses.

![Nodes & Topics](docs/custom_nodes.png)

## Navigation to Detected Objects
- TF chain: `camera_color_optical_frame → odom → map`.  
- Detected can frames (`coke_can_i`) become targets; **move_base** plans paths in `map`.  
- Safety: obstacle layers + recovery behaviors.

![Localization & Navigation](docs/localization_nav.png)

## Dataset & Training
- **Model:** YOLOv8s (speed/accuracy trade-off) and tiny/nano variants for real-time.  
- **Data:** Roboflow + custom far-view photos + **Isaac Sim** synthetic + augmentations (zoom-out, lighting, blur).  
- **Why synthetic?** Domain randomization boosts robustness for small/far cans and lighting shifts.

## Optimization & Risk Mitigation
- TensorRT conversion and input-resolution tuning for FPS.  
- Isolated detection process (Docker) to avoid nav-node latency.  
- Reduced false positives via thresholds + hard negatives.  
- Logging & RViz for validation; metrics tracked: precision/recall/FPS; nav success/time.

## My Role
- Integrated YOLOv8 with ROS: camera bridge, bbox publication, annotated image topic.  
- Prepared multi-source dataset (incl. Isaac Sim) and tuned thresholds.  
- Converted `.pt` to TensorRT `.engine`, packaged Docker image, wired topics & TF.  
- Set up AMCL/move_base params and RViz layouts; conducted lab & real-world tests.

## Demo
- **Video:** `video/demo.mp4` (download to view; stored with Git LFS)

## Notes
- This is a docs-only case study. Source code and training assets remain private.
- Reproduction outline is provided for reference; adapt to your robot stack.

## License
MIT
