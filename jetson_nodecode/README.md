# Detection with node-red on Jetson Xavier NX with BYO model

## prerequisite:

* Jetpack 4.6.1 with SDK components
* Jetson Xavier NX (these setup only works with Xavier NX)

## Setup Steps:

### Prepare and Flash Jetson Xavier NX with Jetpack 4.6.1 

Follow steps here: [flash jetson](https://wiki.seeedstudio.com/reComputer_J2021_J202_Flash_Jetpack#flashing-to-emmc-with-command-line)

### Training model on Local PC with gpu:

Follow steps here: [trainning model](https://wiki.seeedstudio.com/YOLOv5-Object-Detection-Jetson/#use-local-pc)

### Convert to Yolov5 TenserRT model

!!! note
    Following steps are preformed on Jetson Xavier NX

> STEP 1: Step up the yolov5 enviornment

```
sudo apt update

sudo apt install -y python3-pip

pip3 install --upgrade pip

git clone https://github.com/ultralytics/yolov5

cd yolov5

git checkout v6.2

```

Change the requirements.txt to avoid conflicts 

```
vi requirements.txt

```
Change the lib as following:

```
matplotlib==3.2.2
numpy==1.19.4
# torch>=1.7.0
# torchvision>=0.8.1

```

```
sudo apt install -y libfreetype6-dev

```

```
Pillow==7.1.2

pip3 install -r requirements.txt

```

install Torch

```
cd ~

sudo apt-get install -y libopenblas-base libopenmpi-dev

wget https://nvidia.box.com/shared/static/fjtbno0vpo676a25cgvuqc1wty0fkkg6.whl -O torch-1.10.0-cp36-cp36m-linux_aarch64.whl

pip3 install torch-1.10.0-cp36-cp36m-linux_aarch64.whl

```

Install torchvision

```
sudo apt install -y libjpeg-dev zlib1g-dev

git clone --branch v0.9.0 https://github.com/pytorch/vision torchvision

cd torchvision

sudo python3 setup.py install 

```

> STEP 2: convet to tenserrt model by using this [repo](https://github.com/wang-xinyu/tensorrtx)

```
cd ~

git clone https://github.com/wang-xinyu/tensorrtx

cd tensorrtx

git checkout yolov5-v6.2

cp yolov5/gen_wts.py ~/yolov5

```

Now please put the weight file `eg. best.pt` trained by local computer to Jetson Xavier NX yolov5 folder and using the following command to convert tersenrt weights file:

```
cd ~/yolov5

python3 gen_wts.py -w best.pt -o best.wts
```

setup environment for build the tensorrt model engine file from the weights file

```
cd ~/tensorrtx/yolov5

```

change the `CLASS_NUM` of the `yololayer.h` which aligns to your number of labels of the detection detection model

```
vi yololayer.h

```
find the `CLASS_NUM` parameter and change the number accordingly in these case it 2 labels which are "Healthy" & "Sick" so

```
CLASS_NUM = 2

```
exit the editor and make a build directory:

```
mkdir build 
cd build

```

Copy the previously generated best.wts file to the build directory:

```
cp ~/yolov5/best.wts .

```

build it

```
cmake ..
make

```

> STEP 3: Serialize the model please select the size according to your model training setup

```
sudo ./yolov5 -s [.wts] [.engine] [n/s/m/l/x/n6/s6/m6/l6/x6 or c/c6 gd gw]

#example
sudo ./yolov5 -s best.wts best.engine n6

```

now you should have `best.engine` and ` libxxxxx.so` file

### Setup the no-code-edge-ai-tool with these [wiki](https://wiki.seeedstudio.com/No-code-Edge-AI-Tool)

### Change the BYO model

>STEP 1: Copy the best.engine file to the node-red-contrib-ml-detection-1 container

```
sudo docker cp yolov5n.engine node-red-contrib-ml-detection-1:/data/build

```

>STEP 2: change the labels name accordingly 

```
sudo docker exec -it node-red-contrib-ml-detection-1 /bin/bash

nano python/yolov5_trt.py

```
change `self.categories` list to the labels according to your model. Save and exit Nano.

Exit docker container

```
exit

```

Restart  "node-red-contrib-ml-detection-1" container

```
sudo docker restart node-red-contrib-ml-detection-1

```
