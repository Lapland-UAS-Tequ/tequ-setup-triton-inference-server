# tequ-setup-triton-inference-server
Configure NVIDIA Triton Inference Server on different platforms. Deploy object detection model in Tensorflow SavedModel format to server. Send images to server for inference with Node-RED. Triton Inference Server HTTP API is used for inference.

## NVIDIA Jetson Xavier NX

| Software      | Version       | Link |
| ------------- |:-------------:| :-------------:| 
| Jetpack       | 4.6           | https://developer.nvidia.com/embedded/downloads |
| Triton        | 2.14.0        | https://github.com/triton-inference-server/server/releases |

Other Jetson´s might work too, but haven´t been tested.

1. Install official Jetpack 4.6

https://developer.nvidia.com/embedded/learn/get-started-jetson-xavier-nx-devkit

2. Download and install Triton Inference Server package
```
cd /home/
mkdir triton_server
cd /home/triton_server
```

```
wget https://jetson-nodered-files.s3.eu.cloud-object-storage.appdomain.cloud/tritonserver2.14.0-jetpack4.6.tgz
```

```
tar xzvf tritonserver2.14.0-jetpack4.6.tgz
```

3. Create model repository
```
cd /home/triton_server
```
```
mkdir model_repository
```
```
cd model_repository
```
```
wget https://jetson-nodered-files.s3.eu.cloud-object-storage.appdomain.cloud/model_repository.zip
```






3. Install dependencies

```
apt-get update && 
apt-get install -y --no-install-recommends software-properties-common &&
apt-get install -y --no-install-recommends autoconf &&
apt-get install -y --no-install-recommends automake &&
apt-get install -y --no-install-recommends build-essential &&
apt-get install -y --no-install-recommends cmake &&
apt-get install -y --no-install-recommends git &&
apt-get install -y --no-install-recommends libb64-dev &&
apt-get install -y --no-install-recommends libre2-dev &&
apt-get install -y --no-install-recommends libssl-dev &&
apt-get install -y --no-install-recommends libtool &&
apt-get install -y --no-install-recommends libboost-dev &&
apt-get install -y --no-install-recommends libcurl4-openssl-dev &&
apt-get install -y --no-install-recommends libopenblas-dev &&
apt-get install -y --no-install-recommends rapidjson-dev &&
apt-get install -y --no-install-recommends patchelf &&
apt-get install -y --no-install-recommends zlib1g-dev
```

4. Download Triton Inference Server 

5. Download example model repository

6. 

## Windows 10 (TBD)



Sources:

https://developer.nvidia.com/embedded/learn/get-started-jetson-xavier-nx-devkit

https://github.com/triton-inference-server/server/releases
