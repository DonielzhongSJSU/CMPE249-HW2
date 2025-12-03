# Project Report — Failure Analysis and Summary of Unsuccessful Completion
## Project Goal

The objective of this task was to install MMDetection3D on two separate hardware environments:

1. An external NVIDIA Jetson Orin Nano device, and

2. San José State University’s HPC system

The goal was to:

* Install PyTorch, OpenMMLab libraries (MMCV, MMEngine, MMDetection3D)

* Configure CUDA toolchains

* Download models and datasets (nuScenes / Waymo)

* Run 3D object detection inference on sample data

However, the installation repeatedly failed due to CPU architecture incompatibilities, prebuilt wheel limitations, CUDA compiler errors, and restricted HPC network policies.

This report documents each attempt, what failed, and why the project could not be completed.

## 1. Orin Nano Installation Attempts
### 1.1 Repository Cloning

Successfully cloned the project onto the Orin Nano.
```
git clone <repo>
```

No issues here.

## 2. Creating Virtual Environment & Installing Dependencies

Environment created:

```
python3 -m venv venv
source venv/bin/activate
```

Installed basic Python libraries (open3d, pillow), PyTorch (CUDA 12.6), openmim, and mmengine — these steps succeeded.

## 3. CUDA Verification on the Orin Nano

nvcc --version returned:

> Cuda compilation tools, release 12.6, V12.6.68


CUDA configuration was correct.

## 4. Installing MMCV — First Major Point of Failure
### Attempt 1: Install wheel

```pip install /data/cmpe249-fa25/mmcv-2.1.0*-linux_x86_64.whl```


Failed — The prebuilt wheel was compiled for x86_64-v3, but the Orin Nano’s host CPU only supports x86_64-v2.

### Why this failed

* Newer MMCV wheels require AVX2 / BMI2 / x86_64-v3 instructions.

* The Orin Nano host CPU cannot meet that requirement.

* This caused repeated errors such as:

> nvc-Error-x86_64-v2 not supported - x86_64-v3 is minimum supported


This architecture mismatch is unfixable for the device.

## 5. Attempting to Build MMCV from Source

I repeatedly attempted to compile MMCV:

> MMCV_WITH_OPS=1 python setup.py build_ext --inplace


This required:

* CUDA headers

* Correct CUDA_HOME

* A supported CPU architecture

Even after fixing the incorrect CUDA path and starting the build, it failed with multiple compiler errors.

### Root Causes

1. CUDA 12.6 is not fully supported by MMCV 2.1.0

2. MMCV requires x86_64-v3 instructions during compilation

3. The Orin Nano’s host CPU is ARM + x86_64-v2, leading to:

  * Missing intrinsics

  * NVCC architecture failures

  * mmcv._ext never being built

Even after a successful “install”, the extension module stayed missing:

> has mmcv._ext ? False


Because of this, MMDetection3D cannot run, even though Python imports succeeded.

## 6. Attempting to Run MMDetection3D on Orin Nano

Once the repo was cloned and models were downloaded, running inference failed with:

> RuntimeError: MMCV ops are not compiled


or

> ImportError: mmcv._ext not found


This made the pipeline unusable.

## 7. Reattempting Installation on SJSU HPC

Due to incompatibilities on the Orin Nano, you attempted to move the project to SJSU’s HPC cluster.

### 7.1 Initial Conda Setup Failed

curl downloads for Miniconda failed because:

* The HPC blocks outbound HTTPS, and

* Cannot resolve DNS for external hosts (“ping: unknown host www.google.com”
).

Eventually succeeded by retrying via another mirror.

### 7.2 Installing PyTorch & Waymo TF Dataset

These steps succeeded with CUDA 12.4 modules loaded.

## 8. Installing MMCV on HPC — Final Point of Failure

Even on the HPC, MMCV installation failed.

### Reason

The HPC CPU nodes also only support x86_64-v2, but NVIDIA’s NVCC toolchain (nvc++) requires x86_64-v3 as the minimum target.

This produced repeated fatal build errors:

> nvc-Error-x86_64-v2 not supported - x86_64-v3 is minimum supported


This made it impossible to build MMCV with CUDA ops.

Thus mmcv._ext never built, and MMDetection3D could not run.

## 9. Final Failure Summary
### Why the project could not be completed
### (1) MMCV No Longer Supports x86_64-v2 CPUs

* Both the Orin Nano development machine and SJSU HPC login nodes use CPUs compatible only with x86_64-v2.

* Modern MMCV wheels require x86_64-v3 CPU instruction sets.

* NVCC (nvc++ backend used by MMCV) has deprecated x86_64-v2.

* As a result:

  * Prebuilt wheels fail

  * Source compilation fails

  * mmcv._ext never builds

This is a hardware-level incompatibility, not a software misconfiguration.

### (2) CUDA 12.6 is not fully supported by MMCV / MMDetection3D

* MMCV’s compiled CUDA ops do not support CUDA 12.6.

* This created additional compiler errors.

### (3) HPC External Network Restrictions

* Conda, pip, git mirrors, and model downloads initially failed.

* HTTPS access is blocked by default.

This prevented installation workflows until workarounds were found.

### (4) Even after setting up PyTorch + MMEngine, mmcv._ext remained missing

Without the native extension, MMDetection3D cannot run any operations involving:

* 3D voxelization

* Sparse convolution ops

* Data pipelines

* Collision detection kernels

## 10. Final Conclusion

After multiple installation attempts across two separate computing platforms (Orin Nano + SJSU HPC), the project could not be completed because of fundamental incompatibilities between:

* The hardware CPU architecture (x86_64-v2)

* The required MMCV version (which now requires x86_64-v3)

* The CUDA toolchain (CUDA 12.6), which MMCV does not support

* The HPC network limitations blocking package retrieval

Since MMCV cannot be installed with working CUDA ops, the entire MMDetection3D pipeline fails, making the task technically impossible on the available hardware.
