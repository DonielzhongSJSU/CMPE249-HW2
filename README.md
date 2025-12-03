# CMPE249-HW2

Student Name: Daniel Xinye Zhang
SJSU ID: 017471818
email: xinye.zhang@sjsu.edu

## Project report and set up is in report.md: https://github.com/DonielzhongSJSU/CMPE249-HW2/blob/main/report.md 

# Homework Instructions
## Goal

### Run 3D object detection with ≥2 models on ≥2 datasets, produce a video + screenshots, and save all inference artifacts.

### Starter code
Inference: [3d_detection_inferenceLinks to an external site.] (modify/extend as needed)

### Local viz: 
[local_open3d_visualizationLinks to an external site.] (install Open3D locally)

### HPC and mmdetection3d setup tutorial: [hpc3Links to an external site.]
### New: [mmdet3d step-by-step tutorialLinks to an external site.]
### Lab server access: student@10.31.81.42, password: cmpeeng276# (sudo account, do not do any system modifications)
### Jetson Orin nano access: student@10.31.52.143, password: sjsujetson01qa (no sudo access)
### These IPs are SJSU internal IPs, you need to VPN back to campus. There is a high chance that VPN is also not work (some students' VPN group cannot access labs), you need to ssh into HPC1 first, then hop to these IPs.
### Put all your work in Container or Conda virtual environment for isolation.
### Two TAs for infrastructure and ROS support: Ambarish Govindarajulu Kaliamurthi <ambarish.govindarajulukaliamurthi@sjsu.edu>, Deven Jaimin Desai <devenjaimin.desai@sjsu.edu>
### Student ROS tutorial: https://ambarishgk.me/ROS2-Tutorial/Links to an external site.

## Tasks
### Inference, Saving, and Visualization (combined)

Run detection on ≥2 datasets using ≥2 models; save .png frames, .ply point clouds (with predictions), and .json metadata; export a short demo video (stitched from frames). Then install Open3D and run [local_open3d_visualization] to view .ply files; include screenshots of detected objects.

## Comparison & Analysis

Compare models across datasets using ≥2 metrics (e.g., mAP/AP, precision/recall, IoU, FPS/latency, memory).

## Excellent option
Train and evaluate a model (via dataset metrics). Fine-tune or train from scratch on one dataset (other than KITTI); report validation/test metrics, include training config (hyperparameters, epochs, batch size), learning curves/logs, and compare with your pretrained baselines. Discuss generalization to the new dataset.

## Deliverables
Repo/folder link containing:

report.md (1–2 pages): setup (env + exact commands), models & datasets, metrics/results figure/table, screenshots, takeaways/limitations.

results/: demo video, ≥4 labeled screenshots.

Modified code (clearly commented) and a README with reproducible steps (env, commands, seeds).
