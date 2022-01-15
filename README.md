# tequ-setup-triton-inference-server
Configure NVIDIA Triton Inference Server on different platforms. Deploy object detection model in Tensorflow SavedModel format to server. Send images to server for inference with Node-RED. Triton Inference Server HTTP API is used for inference.

## NVIDIA Jetson Xavier NX

| Software      | Version       | Link |
| ------------- |:-------------:| :-------------:|
| Jetpack       | 4.6           | https://developer.nvidia.com/embedded/downloads |
| Triton        | 2.14.0        | https://github.com/triton-inference-server/server/releases |

Other Jetson´s might work too, but haven´t been tested.

### 1. Install official Jetpack 4.6

https://developer.nvidia.com/embedded/learn/get-started-jetson-xavier-nx-devkit

### 2. Download and install Triton Inference Server package
```
mkdir /home/$USER/triton_server
cd /home/$USER/triton_server
```
```
wget https://jetson-nodered-files.s3.eu.cloud-object-storage.appdomain.cloud/tritonserver2.14.0-jetpack4.6.tgz
```

```
tar xzvf tritonserver2.14.0-jetpack4.6.tgz
```

### 3. Create model repository
```
wget https://jetson-nodered-files.s3.eu.cloud-object-storage.appdomain.cloud/model_repository.zip
```
```
unzip model_repository.zip
```

### 4. Install dependencies

```
sudo apt-get update &&
sudo apt-get install -y --no-install-recommends software-properties-common &&
sudo apt-get install -y --no-install-recommends autoconf &&
sudo apt-get install -y --no-install-recommends automake &&
sudo apt-get install -y --no-install-recommends build-essential &&
sudo apt-get install -y --no-install-recommends cmake &&
sudo apt-get install -y --no-install-recommends git &&
sudo apt-get install -y --no-install-recommends libb64-dev &&
sudo apt-get install -y --no-install-recommends libre2-dev &&
sudo apt-get install -y --no-install-recommends libssl-dev &&
sudo apt-get install -y --no-install-recommends libtool &&
sudo apt-get install -y --no-install-recommends libboost-dev &&
sudo apt-get install -y --no-install-recommends libcurl4-openssl-dev &&
sudo apt-get install -y --no-install-recommends libopenblas-dev &&
sudo apt-get install -y --no-install-recommends rapidjson-dev &&
sudo apt-get install -y --no-install-recommends patchelf &&
sudo apt-get install -y --no-install-recommends zlib1g-dev
```

### 5. Start Triton Inference Server
```
cd /home/$USER/triton_server/bin
```

```
./tritonserver --model-repository=/home/$USER/triton_server/model_repository --backend-directory=/home/$USER/triton_server/backends --backend-config=tensorflow,version=2 --strict-model-config=false
```

### 6. Autostart on boot

Create file 'triton.service' to folder /etc/systemd/system
```
sudo nano /etc/systemd/system/triton.service
```

Example file content, correct your paths if needed.
```
[Unit]
Description=NVIDIA Triton Inference Server

Wants=network.target
After=syslog.target network-online.target

[Service]
Type=simple
ExecStart=/home/tequ/triton_server/bin/tritonserver --model-repository=/home/tequ/triton_server/model_repository --backend-directory=/home/tequ/triton_server/backends --backend-config=tensorflow,version=2 --strict-model-config=false
Restart=on-failure
RestartSec=10
KillMode=process

[Install]
WantedBy=multi-user.target
```

Enable service
```
sudo systemctl enable triton.service
```
Start service
```
sudo systemctl start triton.service
```

### 7. Send image to server from Node-RED

Example flow: https://github.com/Lapland-UAS-Tequ/tequ-api-client/blob/master/flows/example-ai-detect-triton.json

![alt text](
https://github.com/Lapland-UAS-Tequ/tequ-setup-triton-inference-server/blob/main/images/example-ai-detect-triton.JPG "Example")


## Windows 10

### 1. Install Docker

https://www.docker.com/

### 2. Pull Triton Inference Server docker image
```
docker pull nvcr.io/nvidia/tritonserver:21.12-tf2-python-py3
```

### 3. Download and unzip model repository to c:\tritonserver\model_repository

https://jetson-nodered-files.s3.eu.cloud-object-storage.appdomain.cloud/model_repository.zip

### 4. Start Docker (without GPU support)

```
docker run --rm -p8000:8000 -p8001:8001 -p8002:8002 -v C:\tritonserver\model_repository:/models nvcr.io/nvidia/tritonserver:21.12-tf2-python-py3 tritonserver --model-repository=/models  --backend-config=tensorflow,version=2 --strict-model-config=false
```


## Sources:

https://developer.nvidia.com/embedded/learn/get-started-jetson-xavier-nx-devkit

https://github.com/triton-inference-server/server/releases

https://www.howtogeek.com/687970/how-to-run-a-linux-program-at-startup-with-systemd/
