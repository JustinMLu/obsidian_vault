####  USB Camera Formant Section

```bash
# List USB cameras
ls -l /dev/v4l/by-id

# Test usb script thingy
python3 scripts/test_usb_camera.py --mode web --port 5000 --device /dev/v4l/by-id/<insert camera id here>

# for example
python3 scripts/test_usb_camera.py --mode web --port 5000 --device /dev/v4l/by-id/usb-4K_USB_CAMERA_HD_USB_CAMERA_01.00.00-video-index0
```

#### SSH into Jetsons:

```bash
ssh orin@192.168.1.65 # Jetson Nano (orin@orin-van2)
ssh agx2@192.168.1.207 # Jetson AGX (agx2@agx2-desktop)
```

- Password is ```123456```

#### Specs:

**Orin Nano (Docker)** 
- Python: ```3.10``` (NOT 100% SURE)
- JetPack:  ```6.1``` (l4t version ```36.4.0```)
- CUDA: ```12.6```
- CuDNN: ```9.3```
- onnxruntime_gpu: ```1.20.0```

**Orin AGX**
- Python: ```3.8.10```
- JetPack: ```5.1.2``` (l4t version ```35.4.1```)
- CUDA: ```11.4```
- CuDNN: ```8.6.0```
- onnxruntime_gpu: ```1.15.1``` 

#### How to build Docker image with verbose
```bash
DOCKER_BUILDKIT=1 COMPOSE_DOCKER_CLI_BUILD=1 docker compose -f docker-compose-deploy.yml build --progress=plain vehicle
```

#### Useful commands
```bash
# Check JetPack version
cat /etc/nv_tegra_release

# Check current power mode
sudo nvpmodel -q

# Set to maximum performance (MAXN)
sudo nvpmodel -m 0

# See all available modes
sudo nvpmodel -q --verbose

# Maximize clock speeds
sudo jetson_clocks

# See if jetson_clocks is active
sudo jetson_clocks --show
```

#### SEARCH DOCKER IMAGES (HOLY MOLY)
CAN U BELIEVE U CAN DO THIS

```bash
# For example, see what L4T versions dustynv has:
docker search dustynv/l4t-pytorch 
```

## Jetson Inference Times (100 trials)

#### Jetson Nano
- Segmentation (CPU):
	- Average: 0.5390
	- Total: 53.9066

- Segmentation (GPU):
	- Average: 0.0677
	- Total: 6.7753

#### Jetson Orin AGX
- Segmentation (CPU):
	- Average: 0.1859
	- Total: 18.5936 

- Segmentation (GPU, non-FP16):
	- Average: 0.0296
	- Total: 2.9591

- YOLOX (CPU):
	- Average: 1.0062
	- Total: 100.6210

- YOLOX (GPU, non-FP16):
	- Average: 0.0225
	- Total: 2.2551

- YOLOX (GPU, FP16):
	- Average: 0.0218
	- Total: 2.1787


## Overall statistics

#### Jetson Orin Nano
- Segmentation speed-up: **7.96x**
- YOLOX speed-up: **N/A**

#### Jetson Orin AGX
- Segmentation speed-up: **6.28x**
- YOLOX speed-up: **44.7x**
- YOLOX speed-up (FP16): **46.2x**