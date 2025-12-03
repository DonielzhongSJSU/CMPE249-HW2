# 1. Clone repo on to external orin nano device
> git clone <repo-url>

# 2. Create virtual environment and install dependencies

> python3 -m venv venv
> 
> source venv/bin/activate
> 
> pip install open3d pillow
> 
> pip3 install torch torchvision --index-url https://download.pytorch.org/whl/cu126
> 
> python -m pip install -U pip setuptools wheel
> 
> python -m pip install -U openmim
> 
> mim install mmengine

# 3. Check nvcc version

> (venv) root@serviceorin-desktop:~/CMPE249-HW2# nvcc --version

> nvcc: NVIDIA (R) Cuda compiler driver
> 
> Copyright (c) 2005-2024 NVIDIA Corporation
> 
> Built on Wed_Aug_14_10:14:07_PDT_2024
> 
> Cuda compilation tools, release 12.6, V12.6.68

# 4. Installing MMCV

> pip install /data/cmpe249-fa25/mmcv-2.1.0*-linux_x86_64.whl
> <img width="1119" height="92" alt="image" src="https://github.com/user-attachments/assets/f4d4164c-8d0c-4980-b621-e85b1d136821" />

Installation failed for MMCV, skipping this section.

# 5. Check for all required dependencies
Since mmcv install failed using the command, I had to install using `pip install mmcv`.

Then I ran `python - <<'PY'
import sys, numpy, matplotlib
print("Python     :", sys.version.split()[0])
print("NumPy      :", numpy.__version__)
print("Matplotlib :", matplotlib.__version__)
from matplotlib import pyplot as plt
print("pyplot OK")
import torch, mmengine, mmcv
print("Torch     :", torch.__version__, "| CUDA:", torch.version.cuda, "| is_available:", torch.cuda.is_available())
print("MMEngine  :", mmengine.__version__)
import pkgutil
try:
    import mmcv
    print("MMCV   :", mmcv.__version__, "at", mmcv.__file__)
    print("has mmcv._ext ? ", pkgutil.find_loader("mmcv._ext") is not None)
except Exception as e:
    print("MMCV import error:", repr(e))
PY` to check for all dependencies. I got the below error :
<img width="864" height="182" alt="image" src="https://github.com/user-attachments/assets/922272fa-a5c4-4d13-a397-992a90cdeff9" />

## 5.1 Attempting to resolve mmcv._ext = false

I ran the debugging commands `python -m pip install -U pip setuptools wheel
python -m pip install -U openmim
mim install mmengine
mim install 'mmcv<2.2.0,>=2.0.0rc4'`. However after doing everything, mmcv._ext was still False.

## 5.2 Attempting to rebuild mmcv

Running the debug instructions, I got a failure that said path not found for nvcc:

<img width="1109" height="489" alt="image" src="https://github.com/user-attachments/assets/19c81119-6f03-40d1-89ba-dcc5ecc91882" />

Looking at the path, the nvcc was directed in the wrong path:

`which nvcc`

>/usr/local/cuda-12.6/bin/nvcc

This time, i was able to get the build started using `MMCV_WITH_OPS=1 python setup.py build_ext --inplace` but got a bunch of errors:

<img width="1119" height="783" alt="image" src="https://github.com/user-attachments/assets/9c4811d1-a848-4004-8589-c1dc76a09a2a" />

<img width="1128" height="562" alt="image" src="https://github.com/user-attachments/assets/67c41d1d-6b6f-4c42-96e0-7a373bb05668" />

Installing mmcv worked this time, using `pip install .`

<img width="1284" height="750" alt="image" src="https://github.com/user-attachments/assets/d8534ba0-f83e-41f7-b022-fbe5b5245f2c" />

mmcv._ext is still False so I will continue without it. I have all other dependencies.

>MMCV   : 2.1.0 at /root/CMPE249-HW2/venv/lib/python3.10/site-packages/mmcv/__init__.py
has mmcv._ext ?  False

# 6. Clone Repo for mmdetect using `$ git clone https://github.com/open-mmlab/mmdetection3d.git`

<img width="743" height="133" alt="image" src="https://github.com/user-attachments/assets/b611ac4f-35b8-4bb7-8a5a-d513c15bed26" />

## 6.1 Download models using `mim download mmdet3d --config pointpillars_hv_secfpn_8xb6-160e_kitti-3d-car --dest ./modelzoo_mmdetection3d/`

<img width="741" height="151" alt="image" src="https://github.com/user-attachments/assets/44a247b6-64fe-44bb-a55c-623bf6f1891d" />

## 6.2 Link Nuscenes Data set to mmdetect data folder. 

Set path using `MMDETECTION3D=~/CMPE249-HW2/mmdetection3d`

Make directory `mkdir -p "$MMDETECTION3D/data"`

Symlink Nuscenes data to mmdetect folder `ln -snf ~/datasets/nuscenes/ "$MMDETECTION3D/data/nuScenes"`

Run detection on nuscenes data `python tools/test.py ./modelzoo_mmdetection3d/pointpillars_hv_secfpn_sbn-all_8xb4-2x_nus-3d.py ./modelzoo_mmdetection3d/hv_pointpillars_secfpn_sbn-all_4x8_2x_nus-3d_20210826_225857-f19d00a3.pth`

I get the following error due to mmcv._ext missing.
<img width="1121" height="483" alt="image" src="https://github.com/user-attachments/assets/713fc2b7-3414-4af6-9276-5bf70fadba1c" />

# 5.3 Attempting to resolve mmcv_.ext again

I consulted chatgpt and was instructed to attempt reinstalling everything

```
pip uninstall -y mmcv

python setup.py clean
rm -rf build

MMCV_WITH_OPS=1 CUDA_HOME=/usr/local/cuda-12.6 \
    pip install -v --no-build-isolation .
```

Again, I get multiple errors during compilation:

<img width="1135" height="591" alt="image" src="https://github.com/user-attachments/assets/400f29be-ebe8-475f-87b4-647c026b3aae" />
