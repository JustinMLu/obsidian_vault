#### SSH into Jetsons:

```bash
ssh orin@192.168.1.65 # Jetson Nano (orin@orin-van2)
ssh agx2@192.168.1.207 # Jetson AGX (agx2@agx2-desktop)
```

- Password is ```123456```

#### Specs:

**Orin Nano (Docker)** 
- Python: ```3.10``` (NOT SURE)
- JetPack:  ```6.1``` (l4t version ```36.4.0```)
- CUDA: ```12.6```
- CuDNN: ```9.3```
- onnxruntime_gpu: ```1.20.0```

**Orin AGX**
- Python: ```3.8.10```
- JetPack: ```5.1.2``` (l4t version ```35.4.1```)
- CUDA: ```11.4```
- CuDNN: ```8.6.0```
- onnxruntime_gpu: 

#### How to build Docker image with verbose
```bash
DOCKER_BUILDKIT=1 COMPOSE_DOCKER_CLI_BUILD=1 docker compose -f docker-compose-deploy.yml build --progress=plain vehicle
```

#### Useful commands
```bash
# Check JetPack version
cat /etc/nv_tegra_release
```