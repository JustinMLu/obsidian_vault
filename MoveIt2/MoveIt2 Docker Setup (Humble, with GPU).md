
# Setting Up Moveit2 with Docker

### 1. Enable non-sudo Docker

```bash 
sudo usermod -aG docker $USER   # add yourself to docker group
newgrp docker                   # or log out/in so the group takes effect
```

### 2. Install the NVIDIA Container Toolkit

```bash
# prerequisites
sudo apt update
sudo apt install -y ca-certificates curl gnupg lsb-release

# add NVIDIA GPG key
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey |
  sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit.gpg

# add the repo
curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list |
  sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit.gpg] https://#' |
  sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

# install toolkit
sudo apt update
sudo apt install -y nvidia-container-toolkit
```

### 3. Configure Docker to use NVIDIA runtime

```bash
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker # restart docker
```

### 4. Sanity check the GPU passthrough

- Make sure to specify the proper CUDA version! Here, I'm on Ubuntu 22.04 with  ```Driver Version: 550.144.03 | CUDA Version: 12.4```.

```bash
docker run --rm --gpus all nvidia/cuda:12.4.1-base-ubuntu22.04 nvidia-smi
# should print the familiar nvidia-smi table
```

### 6. Launch MoveIt2 container with GPU enabled

```bash
DOCKER_IMAGE=humble-humble-tutorial-source \
docker compose run --rm --name moveit2_gpu gpu
```

- Drops you into ```/ws_moveit``` inside the container.
- Workspace is already cloned and built.

### 7. Run RViz quickstart demo

```bash
source install/setup.bash
ros2 launch moveit2_tutorials demo.launch.py
```

- RViz will open up with the Panda arm. Don't bother trying to change it to the Kinova arm in this docker container - it's a massive pain in the ass.


# Useful tips

### 1. Re-enter the container (while it's still running) from another terminal:

```bash
docker exec -it moveit2_gpu bash
```

### 2. Rebuild after editing source:

```bash
cd /ws_moveit # or wherever your workspace is
colcon build --mixin release
source install/setup.bash # re-source rebuilt overlay (ws_moveit)
```

### 3. Omit ```--rm``` when launching MoveIt2 container if you want the container's changes to persist between sessions:

##### With ```--rm``` (default in the instructions):

```bash
DOCKER_IMAGE=humble-humble-tutorial-source \
docker compose run --rm --name moveit2_gpu gpu
```

- Container is deleted automatically upon exit.
- Every launch will start from a 'clean slate'.

##### Without ```--rm``` (persist changes):

```bash
DOCKER_IMAGE=humble-humble-tutorial-source \
docker compose run --name moveit2_gpu gpu
```

- Container **just stops** upon exiting - no deletions.
- Restart the container with ```docker start -ai moveit2_gpu```


