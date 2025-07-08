
# RViz and ROS!

#### How to reset all packages
```bash
ros2 topic pub /packing_planner/reset_packages -1 std_msgs/msg/String '{data: "True"}'
```

#### Useful Docker commands
```bash
# Start container without immediately booting launchfile
docker compose -f docker-compose-sim.yml run vehicle /bin/bash

# Start the program
ros2 launch py_cargo cargo_sim.launch.py
```

#### How to un-reverse capslock
```bash 
sudo systemctl restart gdm
```

```bash
# Terminal 1: Launch
ros2 launch py_cargo cargo_sim.launch.py

# Terminal 2: Start package spawner node (can also uncomment it in the launch file)
python3 vehicle/ros/src/py_cargo/py_cargo/sim_package_spawner.py

# Terminal 3: Stow script
bash vehicle/backend/stow.sh 10 0 10 10 10
```


# Vision!

#### How to access GPU Node (again)
```shell
# 1. SSH into login node
ssh lujust@gl-login1.arc-ts.umich.edu

# 2. Check your actively running jobs
squeue -u lujust

# 3. Connect to the job
srun --jobid=JOBID_NUMBER --pty bash

#4. Test to ensure you're in the GPU node
nvidia-smi
```

#### Start training and demo
```shell
# Start training with 64 batch size and caching
python tools/train.py \
    -f exps/example/custom/cargo_yolox_s.py \
    -d 1 \
    -b 64 \
    --fp16 \
    -o \
    --cache \
    -c pretrained_models/yolox_s.pth

# Demo
python tools/cargo_demo.py \ 
	image \ 
	-f exps/example/custom/cargo_yolox_s.py \ 
	-c YOLOX_outputs/cargo_yolox_s/best_ckpt.pth \ 
	--path assets/duartest.jpg \ 
	--conf 0.25 \ 
	--nms 0.45 \ 
	--tsize 640 \ 
	--save_result \ 
	--device gpu # or cpu
```

#### Inline versions
```shell
# Train
python tools/train.py -f exps/example/custom/cargo_yolox_s.py -d 1 -b 64 --fp16 -o --cache -c pretrained_models/yolox_s.pth

# Demo
python tools/cargo_demo.py image -f exps/example/custom/cargo_yolox_s.py -c YOLOX_outputs/cargo_yolox_s/best_ckpt.pth --path assets/duartest.jpg --conf 0.25 --nms 0.45 --tsize 640 --save_result --device cpu
```


#### Start Tensorboard with port forwarding
```shell
# STEP 1: ssh on separate terminal w/ port forwarding
ssh -L 6006:localhost:6006 lujust@gl-login1.arc-ts.umich.edu

# STEP 2: Go into the directory
cd src/YOLOX/YOLOX_outputs/<YOUR_MODEL>

# STEP 3: Start tensorboard (while in the directory)
tensorboard --logdir=tensorboard/
```

**Note:** This terminal instance doesn't have to connect to the GPU to work, but the (yolox) venv must be activated for tensorboard

#### Use an export script
```shell
# Make sure you're in YOLOX directory
python tools/export_onnx.py \
	-f exps/example/custom/<YOUR_EXPERIMENT>.py \
	-c /path/to/your/best_ckpt.pth \
	--output-name <INSERT_NAME>.onnx

# You can also verify using:
python -m onnxruntime.tools.check_model cargo_detector.onnx
```

**Note:*** The ```export_onnx.py``` script will automatically apply ONNX simplification unless you specify the ```--no-onnxsm``` flag!

# Jetson Orin

#### ssh into Jetson Orin
```shell
# ssh into the jetson nano
ssh orin@orin-van1.local # Password: 123456
```