This repository is developed in Fish-IoT project

https://www.tequ.fi/en/project-bank/fish-iot/ 

---

# tequ-setup-triton-inference-server
Configure NVIDIA Triton Inference Server on different platforms. Deploy object detection model in Tensorflow SavedModel format to server. Send images to server for inference with Node-RED. Triton Inference Server HTTP API is used for inference.

# NVIDIA Jetson Xavier NX / Nano

| Software      | Version       | Link |
| ------------- |:-------------:| :-------------:|
| Jetpack       | 4.6           | https://developer.nvidia.com/embedded/downloads |
| Triton        | 2.17.0        | https://github.com/triton-inference-server/server/releases |

Other Jetson´s might work too, but haven´t been tested.

## 1. Install official Jetpack 4.6

https://developer.nvidia.com/embedded/learn/get-started-jetson-xavier-nx-devkit

## 2. Download and install Triton Inference Server package
```
mkdir /home/$USER/triton_server
cd /home/$USER/triton_server
```
```
wget https://jetson-nodered-files.s3.eu.cloud-object-storage.appdomain.cloud/tritonserver2.17.0-jetpack4.6.tgz
```

```
tar xzvf tritonserver2.17.0-jetpack4.6.tgz
```

## 3. Create model repository
```
wget https://jetson-nodered-files.s3.eu.cloud-object-storage.appdomain.cloud/model_repository.zip
```
```
unzip model_repository.zip
```

## 4. Install dependencies

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

## 5. Start Triton Inference Server
```
cd /home/$USER/triton_server/bin
```

```
./tritonserver --model-repository=/home/$USER/triton_server/model_repository --backend-directory=/home/$USER/triton_server/backends --backend-config=tensorflow,version=2 --strict-model-config=false
```

## 6. Autostart on boot

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

# Windows 10

## 1. Install Docker

https://www.docker.com/

## 2. Pull Triton Inference Server docker image
```
docker pull nvcr.io/nvidia/tritonserver:21.12-tf2-python-py3
```

## 3. Download and unzip model repository to c:\tritonserver\model_repository

https://jetson-nodered-files.s3.eu.cloud-object-storage.appdomain.cloud/model_repository.zip

## 4. Start Docker (without GPU support)

```
docker run --rm -p8000:8000 -p8001:8001 -p8002:8002 -v C:\tritonserver\model_repository:/models nvcr.io/nvidia/tritonserver:21.12-tf2-python-py3 tritonserver --model-repository=/models  --backend-config=tensorflow,version=2 --strict-model-config=false
```

# Use Triton Inference Server from Node-RED

Configure triton_server_url in "[AI] Detect - Triton" node to match your target Triton server. Client machine must have been prepared for Conmputer vision.

https://github.com/Lapland-UAS-Tequ/tequ-jetson-nodered-tensorflow

https://github.com/Lapland-UAS-Tequ/win10-nodered-tensorflow


Example flow: 
```
[{"id":"83a7a965.1808a8","type":"subflow","name":"[IMG] Annotate","info":"","category":"Tequ-API Client","in":[{"x":120,"y":140,"wires":[{"id":"d05bfd8e.a02e"}]}],"out":[{"x":1080,"y":140,"wires":[{"id":"4e5f5c6c.bcf214","port":0}]}],"env":[{"name":"box_colors","type":"json","value":"{\"fish\":\"#FFFFFF\",\"pike\":\"#006400\",\"perch\":\"#008000\",\"smolt\":\"#ADD8E6\",\"salmon\":\"#0000FF\",\"trout\":\"#0000FF\",\"cyprinidae\":\"#808080\",\"zander\":\"#009000\",\"bream\":\"#008800\"}","ui":{"type":"input","opts":{"types":["json"]}}},{"name":"image_settings","type":"json","value":"{\"quality\":0.8}","ui":{"type":"input","opts":{"types":["json"]}}},{"name":"image_type","type":"str","value":"image/jpeg","ui":{"type":"select","opts":{"opts":[{"l":{"en-US":"JPG"},"v":"image/jpeg"},{"l":{"en-US":"PNG"},"v":"image/png"}]}}},{"name":"bbox_lineWidth","type":"num","value":"5","ui":{"type":"spinner","opts":{"min":0,"max":10}}},{"name":"bbox_text_color","type":"str","value":"white","ui":{"type":"select","opts":{"opts":[{"l":{"en-US":"white"},"v":"white"},{"l":{"en-US":"black"},"v":"black"},{"l":{"en-US":"blue"},"v":"blue"},{"l":{"en-US":"green"},"v":"green"},{"l":{"en-US":"yellow"},"v":"yellow"},{"l":{"en-US":"red"},"v":"red"},{"l":{"en-US":"orange"},"v":"orange"}]}}},{"name":"bbox_font","type":"str","value":"30px Arial","ui":{"type":"select","opts":{"opts":[{"l":{"en-US":"5px Arial"},"v":"5 px Arial"},{"l":{"en-US":"10px Arial"},"v":"10px Arial"},{"l":{"en-US":"15px Arial"},"v":"15px Arial"},{"l":{"en-US":"20px Arial"},"v":"20px Arial"},{"l":{"en-US":"25px Arial"},"v":"25px Arial"},{"l":{"en-US":"30px Arial"},"v":"30px Arial"},{"l":{"en-US":"35px Arial"},"v":"35px Arial"},{"l":{"en-US":"40px Arial"},"v":"40px Arial"},{"l":{"en-US":"45px Arial"},"v":"45px Arial"},{"l":{"en-US":"50px Arial"},"v":"50px Arial"}]}}},{"name":"label_offset_x","type":"num","value":"0","ui":{"type":"input","opts":{"types":["num"]}}},{"name":"label_offset_y","type":"num","value":"30","ui":{"type":"input","opts":{"types":["num"]}}},{"name":"threshold","type":"num","value":"0.75","ui":{"type":"spinner","opts":{"min":0,"max":1}}},{"name":"labels","type":"json","value":"[\"fish\",\"perch\", \"pike\", \"rainbow trout\", \"salmon\", \"trout\", \"cyprinidae\", \"zander\", \"smolt\", \"bream\"]","ui":{"type":"input","opts":{"types":["json"]}}}],"meta":{"module":"[IMG] Annotate","version":"0.0.1","author":"juha.autioniemi@lapinamk.fi","desc":"Annotates prediction results from [AI] Detect subflows.","license":"MIT"},"color":"#87A980","icon":"font-awesome/fa-pencil-square-o","status":{"x":1080,"y":280,"wires":[{"id":"7fd4f6bf24348b12","port":0}]}},{"id":"c19ac6bd.2a9d08","type":"function","z":"83a7a965.1808a8","name":"Annotate with  canvas","func":"const img = msg.payload.image.buffer;\nconst image_type = env.get(\"image_type\");\nconst image_settings = env.get(\"image_settings\");\nconst bbox_lineWidth = env.get(\"bbox_lineWidth\");\nconst bbox_text_color = env.get(\"bbox_text_color\");\nconst label_offset_x = env.get(\"label_offset_x\");\nconst label_offset_y = env.get(\"label_offset_y\");\nconst bbox_font = env.get(\"bbox_font\");\nconst COLORS = env.get(\"box_colors\");\nconst objects = msg.payload.inference.result\nconst labels = env.get(\"labels\")\n\n//Define threshold\nlet threshold = 0;\n\nconst global_settings = global.get(\"settings\") || undefined\nlet thresholdType = \"\"\n\nif(global_settings !== undefined){\n    if(\"threshold\" in global_settings){\n        threshold = global_settings[\"threshold\"]\n        thresholdType = \"global\";\n    }\n}\n\nelse if(\"threshold\" in msg){\n    threshold = msg.threshold;\n    thresholdType = \"msg\";\n    if(threshold < 0){\n        threshold = 0\n    }\n    else if(threshold > 1){\n        threshold = 1\n    }\n}\n\nelse{\n    threshold = env.get(\"threshold\");\n    thresholdType = \"env\";\n}\n\nmsg.thresholdUsed = threshold;\nmsg.thresholdTypeUsed = thresholdType;\n\nasync function annotateImage(image) {\n  const localImage = await canvas.loadImage(image);  \n  const cvs = canvas.createCanvas(localImage.width, localImage.height);\n  const ctx = cvs.getContext('2d');  \n  ctx.drawImage(localImage, 0, 0); \n  \n  objects.forEach((obj) => {\n        if(labels.includes(obj.class) && obj.score >= threshold){\n            let [x, y, w, h] = obj.bbox;\n            ctx.lineWidth = bbox_lineWidth;\n            ctx.strokeStyle = COLORS[obj.class];\n            ctx.strokeRect(x, y, w, h);\n            ctx.fillStyle = bbox_text_color;\n            ctx.font = bbox_font;\n            ctx.fillText(obj.class+\" \"+Math.round(obj.score*100)+\" %\",x+label_offset_x,y+label_offset_y);\n        }\n      });\n  \n  return cvs.toBuffer(image_type, image_settings);\n}\n\nif(objects.length > 0){\n    msg.annotated_image = await annotateImage(img)\n    //node.done()\n    msg.objects_found = true\n}\nelse{\n    msg.objects_found = false\n}\n\nreturn msg;","outputs":1,"noerr":0,"initialize":"","finalize":"","libs":[{"var":"canvas","module":"canvas"}],"x":440,"y":140,"wires":[["a801355d.9f7ac8"]]},{"id":"d05bfd8e.a02e","type":"change","z":"83a7a965.1808a8","name":"timer","rules":[{"t":"set","p":"start","pt":"msg","to":"","tot":"date"}],"action":"","property":"","from":"","to":"","reg":false,"x":230,"y":140,"wires":[["c19ac6bd.2a9d08"]]},{"id":"a801355d.9f7ac8","type":"change","z":"83a7a965.1808a8","name":"end timer","rules":[{"t":"set","p":"payload.annotation.time_ms","pt":"msg","to":"$millis() - msg.start","tot":"jsonata"},{"t":"set","p":"payload.annotation.buffer","pt":"msg","to":"annotated_image","tot":"msg"},{"t":"set","p":"payload.annotation.objects_found","pt":"msg","to":"objects_found","tot":"msg"},{"t":"delete","p":"annotated_image","pt":"msg"},{"t":"delete","p":"start","pt":"msg"}],"action":"","property":"","from":"","to":"","reg":false,"x":640,"y":140,"wires":[["4e5f5c6c.bcf214","c20a6448.e6f218"]]},{"id":"4e5f5c6c.bcf214","type":"change","z":"83a7a965.1808a8","name":"delete useless","rules":[{"t":"delete","p":"annotated_image","pt":"msg"},{"t":"delete","p":"start","pt":"msg"},{"t":"delete","p":"objects_found","pt":"msg"}],"action":"","property":"","from":"","to":"","reg":false,"x":880,"y":140,"wires":[[]]},{"id":"c20a6448.e6f218","type":"switch","z":"83a7a965.1808a8","name":"objects found?","property":"objects_found","propertyType":"msg","rules":[{"t":"true"},{"t":"false"}],"checkall":"true","repair":false,"outputs":2,"x":660,"y":200,"wires":[["a9379cd1321a02da"],["0ec56ca8f000a540"]]},{"id":"a9379cd1321a02da","type":"function","z":"83a7a965.1808a8","name":"","func":"node.status({fill:\"green\",shape:\"dot\",text:msg.thresholdTypeUsed+\" \"+msg.thresholdUsed+\" in \"+msg.payload.annotation.time_ms+\" ms\"})","outputs":0,"noerr":0,"initialize":"","finalize":"","libs":[],"x":860,"y":180,"wires":[]},{"id":"0ec56ca8f000a540","type":"function","z":"83a7a965.1808a8","name":"","func":"node.status({fill:\"green\",shape:\"dot\",text:msg.thresholdTypeUsed+\" \"+msg.thresholdUsed+\" No objects to annotate\"})","outputs":0,"noerr":0,"initialize":"","finalize":"","libs":[],"x":860,"y":220,"wires":[]},{"id":"7fd4f6bf24348b12","type":"status","z":"83a7a965.1808a8","name":"","scope":null,"x":860,"y":280,"wires":[[]]},{"id":"9d7441713e5a169b","type":"subflow","name":"[AI] Detect - Triton","info":"Send image to NVIDIA Triton Inference server using HTTP API.\n\nModel should be configured and loaded to server before using.\n\nSupported models:\n - tequ-fish-species-detection\n","category":"Tequ-API Client","in":[{"x":80,"y":140,"wires":[{"id":"a27070060363f2df"}]}],"out":[{"x":1120,"y":300,"wires":[{"id":"ec5e76112f20b1cd","port":0},{"id":"2e4abd3abccee2f6","port":1}]},{"x":980,"y":420,"wires":[{"id":"df34ddea04c5826b","port":0}]}],"env":[{"name":"threshold","type":"num","value":"0.75","ui":{"type":"spinner","opts":{"min":0,"max":1}}},{"name":"model_name","type":"str","value":"ssd-example-model","ui":{"type":"input","opts":{"types":["str","env"]}}},{"name":"triton_server_url","type":"str","value":"tequ-jetson-nx-1:8000","ui":{"type":"input","opts":{"types":["str"]}}},{"name":"image_width_cm","type":"env","value":"image_width_cm","ui":{"type":"input","opts":{"types":["json","env"]}}}],"meta":{"module":"node-red-contrib-tequ-ai-triton","version":"0.0.1","author":"juha.autioniemi@lapinamk.fi","desc":"Inference image using NVIDIA Triton Inference server.","license":"MIT"},"color":"#FFCC66","inputLabels":["Image buffer in"],"outputLabels":["Inference result","Model configuration"],"icon":"node-red/status.svg","status":{"x":980,"y":620,"wires":[{"id":"5ac4fb48cdf1f282","port":0}]}},{"id":"18efd978252e44b4","type":"function","z":"9d7441713e5a169b","name":"Pre-process","func":"const imageBuffer = msg.payload;\nconst modelName = env.get(\"model_name\")\nconst server_url = env.get(\"triton_server_url\")\n\nasync function createImageTensor(input){\n    return tf.tidy(() => {\n        const tensor = tf.node.decodeJpeg(input, 3).expandDims(0);\n        const shape = tensor.shape;\n        return {\"tensor\":tensor,\"shape\":shape}\n    });\n}\n\nconst image_tensor = await createImageTensor(imageBuffer)\nconst image_tensor_buffer = Buffer.from(image_tensor[\"tensor\"].dataSync());\nconst image_tensor_buffer_length = image_tensor_buffer.length;\nimage_tensor[\"tensor\"].dispose()\n\nlet inference_header = {\n      \"model_name\" : \"test\",\n      \"inputs\" : [\n        {\n          \"name\" : \"input_tensor\",\n          \"shape\" : image_tensor[\"shape\"],\n          \"datatype\" : \"UINT8\",\n          \"parameters\" : {\n              \"binary_data_size\":image_tensor_buffer_length\n          }\n        }\n      ],\n      \"outputs\" : [\n        {\n          \"name\" : \"detection_scores\",\n          \"parameters\" : {\n            \"binary_data\" : false\n          }\n        },\n         {\n          \"name\" : \"detection_boxes\",\n          \"parameters\" : {\n            \"binary_data\" : false\n          }\n        },\n         {\n          \"name\" : \"detection_classes\",\n          \"parameters\" : {\n            \"binary_data\" : false\n          }\n        }\n      ]\n    }\n\nlet inference_header_buffer = Buffer.from(JSON.stringify(inference_header))\nlet inference_header_length = inference_header_buffer.length\n\nmsg.method = \"POST\";\nmsg.payload = Buffer.concat([inference_header_buffer,image_tensor_buffer])\nmsg.url = server_url+\"/v2/models/\"+modelName+\"/infer\";\nmsg.headers = {\n    \"Content-Type\":\"application/octet-stream\",\n    \"Inference-Header-Content-Length\":inference_header_length,\n    \"Content-Length\":inference_header_length+image_tensor_buffer_length\n};\n    \ntime_ms =  new Date().getTime() - msg.start;\nmsg.pre_process_time = time_ms;\nnode.status({fill:\"green\",shape:\"dot\",text:time_ms+\" ms\"});    \nreturn msg;","outputs":1,"noerr":0,"initialize":"","finalize":"// Code added here will be run when the\n// node is being stopped or re-deployed.\nnode.status({fill:\"blue\",shape:\"dot\",text:\"Stopping node...\"});  \ncontext.set(\"model\",undefined)\ntf.disposeVariables()","libs":[{"var":"fs","module":"fs"},{"var":"os","module":"os"},{"var":"zlib","module":"zlib"},{"var":"tf","module":"@tensorflow/tfjs-node-gpu"}],"x":210,"y":280,"wires":[["d0fddd20c79b5628"]]},{"id":"d0fddd20c79b5628","type":"http request","z":"9d7441713e5a169b","name":"","method":"use","ret":"obj","paytoqs":"ignore","url":"","tls":"","persist":true,"proxy":"","authType":"","senderr":false,"x":570,"y":280,"wires":[["2e4abd3abccee2f6"]]},{"id":"ec5e76112f20b1cd","type":"function","z":"9d7441713e5a169b","name":"Post-process","func":"let inference_result = msg.payload;\nlet results = [];\nconst modelName = env.get(\"model_name\")\nconst model = flow.get(modelName);\nconst labels = model.parameters.labels;\nconst metadata = model.parameters.metadata;\nconst ts = msg.ts;\n\nconst threshold = msg.threshold;\nconst image_width = msg.width;\nconst image_height = msg.height;\nconst originalImage = msg.image;\nlet scores;\nlet boxes;\nlet names;\n\nfunction chunk(arr, chunkSize) {\n  if (chunkSize <= 0) throw \"Invalid chunk size\";\n  var R = [];\n  for (var i=0,len=arr.length; i<len; i+=chunkSize)\n    R.push(arr.slice(i,i+chunkSize));\n  return R;\n}\n\nfor(let i=0;i<(inference_result.outputs).length;i++){\n    //node.warn(inference_result.outputs[i])\n    \n    if(inference_result.outputs[i][\"name\"] == \"detection_scores\"){\n        scores = inference_result.outputs[i].data\n    }\n    else if(inference_result.outputs[i][\"name\"] == \"detection_boxes\"){\n        boxes = inference_result.outputs[i].data\n        boxes = chunk(boxes,4)\n        \n    }\n    else if(inference_result.outputs[i][\"name\"] == \"detection_classes\"){\n        names = inference_result.outputs[i].data \n    }\n} \n\nfor (let i = 0; i < scores.length; i++) {\n    if (scores[i] > threshold) {\n        newObject = {\n            \"bbox\":[\n                    boxes[i][1] * image_width,\n                    boxes[i][0] * image_height,\n                    (boxes[i][3] - boxes[i][1]) * image_width,\n                    (boxes[i][2] - boxes[i][0]) * image_height\n            ],\n            \"class\":labels[names[i]-1],\n            \"label\":labels[names[i]-1],\n            \"score\":scores[i],\n            \"length_cm\":NaN\n            }\n        results.push(newObject)\n    }\n}\n        \n//Calculate object width if image_width_cm is given input message\n if(\"image_width_cm\" in msg){\n    const image_width_cm = msg.image_width_cm;    \n    for(let j=0;j<results.length;j++){\n        px_in_cm = image_width_cm / msg.width\n        object_size_cm = px_in_cm * results[j].bbox[2]\n        results[j].length_cm = Math.round(object_size_cm)\n    }\n}\n        \n// Create output message\nlet result_message = {\n    \"labels\":labels,\n    \"thresholdType\":msg.thresholdType,\n    \"threshold\": msg.threshold,\n    \"image_width_cm\":msg.image_width_cm,\n    \"image_width_cm_type\":msg.image_width_cm_type,\n    \"topic\":msg.topic,\n    \"payload\":{\n        \"inference\":{\n            \"metadata\":metadata,\n            \"time_ms\": new Date().getTime() - msg.start,\n            \"validated\":false,\n            \"result\":results,\n            \"type\":\"object detection\"\n        },\n        \"image\":{\n            \"buffer\":originalImage,\n            \"width\": msg.width,\n            \"height\": msg.height,\n            \"type\": msg.type,\n            \"size\": (originalImage).length,\n            \"exif\":{}\n            }\n    }\n}\n\n// Add exif information\nif(msg.exif){\n    result_message.payload.image.exif = msg.exif\n}\n        \ntotal_ms =  new Date().getTime() - msg.start;\n        \nnode.status({fill:\"blue\",shape:\"dot\",text: msg.pre_process_time+\" ms | \"+(result_message.payload.inference.time_ms-msg.pre_process_time)+\" ms | \"+result_message.payload.inference.time_ms+\" ms\"});           \n    \nreturn result_message;","outputs":1,"noerr":0,"initialize":"","finalize":"","libs":[],"x":910,"y":260,"wires":[[]]},{"id":"bdf244aaa74449f8","type":"function","z":"9d7441713e5a169b","name":"Set threshold & image_width_cm","func":"//Define threshold\nlet threshold = 0;\nconst global_settings = global.get(\"settings\") || undefined\nlet thresholdType = \"\"\n\nif(global_settings !== undefined){\n    if(\"threshold\" in global_settings){\n        threshold = global_settings[\"threshold\"]\n        thresholdType = \"global\";\n    }\n}\n\nelse if(\"threshold\" in msg){\n    threshold = msg.threshold;\n    thresholdType = \"msg\";\n    if(threshold < 0){\n        threshold = 0\n    }\n    else if(threshold > 1){\n        threshold = 1\n    }\n}\n\nelse{\n    try{\n        threshold = env.get(\"threshold\");\n        thresholdType = \"env\";\n    }\n    catch(err){\n        threshold = 0.5\n        thresholdType = \"default\";\n    }\n}\n\n\ntry{\n    image_width_cm_type = \"env\";\n    image_width_cm = JSON.parse(env.get(\"image_width_cm\"))[msg.topic];\n        \n}\ncatch(err){\n    image_width_cm = 130\n    image_width_cm_type = \"default\";\n}\n\nif(image_width_cm == undefined){\n    image_width_cm = 130\n}\n\n\nif(threshold == undefined){\n    threshold = 0\n}\n\nmsg.thresholdType = thresholdType;\nmsg.threshold = threshold;\nmsg.image_width_cm = image_width_cm;\nmsg.image_width_cm_type = image_width_cm_type;\n//node.status({fill:\"green\",shape:\"dot\",text:\"threshold: \"+threshold+\" | Image width: \"+image_width_cm});\nreturn msg;","outputs":1,"noerr":0,"initialize":"","finalize":"","libs":[],"x":980,"y":200,"wires":[["18efd978252e44b4"]]},{"id":"aac6b7c6db9f9a61","type":"function","z":"9d7441713e5a169b","name":"isBuffer?","func":"let timestamp = new Date().toISOString();\nmsg.start = new Date().getTime()\n\nif(Buffer.isBuffer(msg.payload)){\n    node.status({fill:\"green\",shape:\"dot\",text:timestamp + \" OK\"});  \n    msg.image = msg.payload;\n    return msg;\n}\nelse{\n    node.error(\"msg.payload is not an image buffer\",msg)\n    node.status({fill:\"red\",shape:\"dot\",text:timestamp + \" msg.payload is not an image buffer\"});  \n    return null;\n}","outputs":1,"noerr":0,"initialize":"","finalize":"","libs":[],"x":200,"y":200,"wires":[["a9c87fd163b03fe8"]]},{"id":"def713aaa3aa2726","type":"exif","z":"9d7441713e5a169b","name":"","mode":"normal","property":"payload","x":750,"y":200,"wires":[["bdf244aaa74449f8"]]},{"id":"6f0b51afb2815ede","type":"image-info","z":"9d7441713e5a169b","name":"","x":570,"y":200,"wires":[["def713aaa3aa2726"]]},{"id":"71ce551ba3faaef2","type":"http request","z":"9d7441713e5a169b","name":"http request","method":"use","ret":"obj","paytoqs":"ignore","url":"","tls":"","persist":false,"proxy":"","authType":"","x":590,"y":440,"wires":[["df34ddea04c5826b"]]},{"id":"e2b2f167d1e53859","type":"function","z":"9d7441713e5a169b","name":"request","func":"let modelName = env.get(\"model_name\")\nlet server_url = env.get(\"triton_server_url\")\n\nmsg.method = \"GET\";\nmsg.url = server_url+\"/v2/models/\"+modelName+\"/config\";\nreturn msg;","outputs":1,"noerr":0,"initialize":"","finalize":"","libs":[],"x":420,"y":440,"wires":[["71ce551ba3faaef2"]]},{"id":"df34ddea04c5826b","type":"function","z":"9d7441713e5a169b","name":"status","func":"let statusCode = msg.statusCode;\n\nif(statusCode == 200){\n    let model_data = msg.payload;\n    let modelName = env.get(\"model_name\")\n    parsed_value = JSON.parse(model_data.parameters.metadata.string_value)\n    model_data.parameters.labels = parsed_value.labels\n    model_data.parameters.metadata = parsed_value.metadata\n    flow.set(modelName,model_data)  \n    node.status({fill:\"green\",shape:\"dot\",text:\" Model configuration loaded.\"});   \n    flow.set(\"ready\",true)\n    return [msg,null]\n}\nelse{\n    node.status({fill:\"red\",shape:\"dot\",text:msg.statusCode+\":\"+msg.payload.error});   \n    node.error(msg.statusCode,msg.payload)\n    return [null,msg]\n}\n\nreturn msg;","outputs":2,"noerr":0,"initialize":"","finalize":"","libs":[],"x":750,"y":440,"wires":[[],["57c77c63c706ef2b"]]},{"id":"5ac4fb48cdf1f282","type":"status","z":"9d7441713e5a169b","name":"","scope":["18efd978252e44b4","d0fddd20c79b5628","ec5e76112f20b1cd","df34ddea04c5826b","2e4abd3abccee2f6","71ce551ba3faaef2"],"x":700,"y":620,"wires":[[]]},{"id":"2e4abd3abccee2f6","type":"function","z":"9d7441713e5a169b","name":"Status","func":"let statusCode = msg.statusCode;\nlet timestamp = new Date().toISOString();\n\nif(statusCode == 200){\n    return [msg,null]\n}\nelse{\n    node.status({fill:\"yellow\",shape:\"dot\",text:\"Request failed... checking server...\"});  \n    flow.set(\"ready\",false)\n    return [null,msg]\n}","outputs":2,"noerr":0,"initialize":"","finalize":"","libs":[],"x":750,"y":280,"wires":[["ec5e76112f20b1cd"],[]]},{"id":"a27070060363f2df","type":"delay","z":"9d7441713e5a169b","name":"","pauseType":"rate","timeout":"5","timeoutUnits":"seconds","rate":"10","nbRateUnits":"1","rateUnits":"second","randomFirst":"1","randomLast":"5","randomUnits":"seconds","drop":true,"allowrate":false,"outputs":1,"x":220,"y":140,"wires":[["aac6b7c6db9f9a61"]]},{"id":"48add00aafb4b737","type":"inject","z":"9d7441713e5a169b","name":"","props":[{"p":"payload"},{"p":"topic","vt":"str"}],"repeat":"","crontab":"","once":true,"onceDelay":"1","topic":"","payloadType":"date","x":130,"y":380,"wires":[["e2b2f167d1e53859"]]},{"id":"57c77c63c706ef2b","type":"delay","z":"9d7441713e5a169b","name":"","pauseType":"delay","timeout":"5","timeoutUnits":"seconds","rate":"1","nbRateUnits":"1","rateUnits":"second","randomFirst":"1","randomLast":"5","randomUnits":"seconds","drop":false,"allowrate":false,"outputs":1,"x":600,"y":540,"wires":[["e2b2f167d1e53859"]]},{"id":"a9c87fd163b03fe8","type":"moment","z":"9d7441713e5a169b","name":"ts","topic":"","input":"","inputType":"date","inTz":"Europe/Helsinki","adjAmount":0,"adjType":"days","adjDir":"add","format":"HH:mm:ss.SSS","locale":"fi-FI","output":"ts","outputType":"msg","outTz":"Europe/Helsinki","x":390,"y":200,"wires":[["6f0b51afb2815ede"]]},{"id":"3ff6e11b151051a1","type":"subflow:9d7441713e5a169b","z":"999f0bad9816935c","name":"","env":[{"name":"triton_server_url","value":"http://tequ-jetson-nano-1:8000","type":"str"}],"x":350,"y":1620,"wires":[["6162669bfea291c7","6e3e72cd784b2471"],["54a2c1a725ecd18a"]]},{"id":"6162669bfea291c7","type":"debug","z":"999f0bad9816935c","name":"","active":true,"tosidebar":true,"console":false,"tostatus":false,"complete":"false","statusVal":"","statusType":"auto","x":570,"y":1600,"wires":[]},{"id":"54a2c1a725ecd18a","type":"debug","z":"999f0bad9816935c","name":"","active":true,"tosidebar":true,"console":false,"tostatus":false,"complete":"false","statusVal":"","statusType":"auto","x":570,"y":1640,"wires":[]},{"id":"a862fae855735785","type":"image","z":"999f0bad9816935c","name":"","width":"480","data":"payload.annotation.buffer","dataType":"msg","thumbnail":false,"active":true,"pass":false,"outputs":0,"x":780,"y":1680,"wires":[]},{"id":"6e3e72cd784b2471","type":"subflow:83a7a965.1808a8","z":"999f0bad9816935c","name":"","env":[{"name":"box_colors","value":"{\"dog\":\"#0000FF\"}","type":"json"},{"name":"threshold","value":"0.50","type":"num"},{"name":"labels","value":"[\"person\",\"bicycle\",\"car\",\"motorcycle\",\"airplane\",\"bus\",\"train\",\"truck\",\"boat\",\"traffic light\",\"fire hydrant\",\"street sign\",\"stop sign\",\"parking meter\",\"bench\",\"bird\",\"cat\",\"dog\",\"horse\",\"sheep\",\"cow\",\"elephant\",\"bear\",\"zebra\",\"giraffe\",\"hat\",\"backpack\",\"umbrella\",\"shoe\",\"eye glasses\",\"handbag\",\"tie\",\"suitcase\",\"frisbee\",\"skis\",\"snowboard\",\"sports ball\",\"kite\",\"baseball bat\",\"baseball glove\",\"skateboard\",\"surfboard\",\"tennis racket\",\"bottle\",\"plate\",\"wine glass\",\"cup\",\"fork\",\"knife\",\"spoon\",\"bowl\",\"banana\",\"apple\",\"sandwich\",\"orange\",\"broccoli\",\"carrot\",\"hot dog\",\"pizza\",\"donut\",\"cake\",\"chair\",\"couch\",\"potted plant\",\"bed\",\"mirror\",\"dining table\",\"window\",\"desk\",\"toilet\",\"door\",\"tv\",\"laptop\",\"mouse\",\"remote\",\"keyboard\",\"cell phone\",\"microwave\",\"oven\",\"toaster\",\"sink\",\"refrigerator\",\"blender\",\"book\",\"clock\",\"vase\",\"scissors\",\"teddy bear\",\"hair drier\",\"toothbrush\",\"hair brush\"]","type":"json"}],"x":580,"y":1680,"wires":[["a862fae855735785"]]},{"id":"1f64ebf42699f276","type":"fileinject","z":"999f0bad9816935c","name":"","x":150,"y":1620,"wires":[["3ff6e11b151051a1"]]}]
```

![alt text](
https://github.com/Lapland-UAS-Tequ/tequ-setup-triton-inference-server/blob/main/images/triton-example-flow.JPG "Example")


## Sources:

https://github.com/triton-inference-server/server/releases

https://www.howtogeek.com/687970/how-to-run-a-linux-program-at-startup-with-systemd/
