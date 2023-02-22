# Passing Images to Seldon Core

In this tutorial we are going to try to pass a image to a yolov5 model from torch hub inside a seldon core server.

## Installation

To order to execute the following test you need to install the following dependencies and use Python > 3.7.0:

```
pip install -qr https://raw.githubusercontent.com/ultralytics/yolov5/master/requirements.txt 
```
```
pip install seldon-core opencv-python
```

## Server

Using the documentation used in the Seldon Core Python Component we are going to create the following server in the file `YoloServer.py`:

```python
import ast

import numpy as np
import torch


class YoloServer(object):
    """
    Yolov5 server
    """

    def __init__(self):
        """
        Add any initialization parameters. These will be passed at runtime from the graph definition
        parameters defined in your seldondeployment kubernetes resource manifest.
        """
        print("Initializing")
        self.ready = False

    # this function is called automaticly after the __init__ when using seldon
    def load(self):
        print("load")
        # be carreful to not load the model twice, can cause deadlocks
        self._model = torch.hub.load("ultralytics/yolov5", "yolov5s")
        self.ready = True

    def predict(self, features, names=[], meta=[]):

        try:
            if not self.ready:
                self.load()
            print("Calling predict")
            # convert to string
            features_ = str(features[0])

            # separate nparray and shape
            features_ = ast.literal_eval(features_)

            # create nparry and convert to float
            im0 = np.array(features_["nparray"])
            im0 = im0.reshape(features_["shape"]).astype("float32")

            # inference
            result = self._model(im0)
            return [str(result)]
        except Exception as e:
            print("EXCEPTION")
            print(e)


if __name__ == "__main__":
    y = YoloServer()
    y.load()

```
To execute the server you can create a new Docker image as used in the [Custom Server Using Docker](docs/custom_server_using_docker.md) or just executing:

```bash
seldon-core-microservice YoloServer --service-type MODEL
```

Seldon will first execute the `__init__` function where we are setting our readyness status as `False`. Next, the server will automatic call the `load` function where our model is being loaded into the atribute `_model`.

The predict function we reiceve the the input in the `features` parameter, we first need to transform `features` from `nparray` to `string`. Next we separate the values of the nparray and shape and convert the output to float. Finally we execute the inference using the pytorch model and return the output.

## Client

To test your server we can use the following script, where we read the image, transform the nparray to a string and make a CURL request to our server:

```python
import json
from subprocess import PIPE, Popen, run

import cv2

# read image
im0 = cv2.imread("maxresdefault.jpg")

# create payload
a = {"nparray": im0.tolist(), "shape": list(im0.shape)}
payload = '{"data": { "ndarray":' + f"{[str(a)]}" + "}}"
# payload is too large, need to save in a file
with open("arg.json", "w") as jSON:
    jSON.write(payload)
payload = '"@arg.json"'

# call CURL
cmd = f"""curl -d {payload} \
   http://0.0.0.0:9000/predict \
   -H "Content-Type: application/json" 
"""
ret = Popen(cmd, shell=True, stdout=PIPE)
raw = ret.stdout.read().decode("utf-8")

# print result
res = json.loads(raw)
results = res["data"]["ndarray"][0]
print(results)
```

# References

https://pytorch.org/hub/ultralytics_yolov5/

https://docs.seldon.io/projects/seldon-core/en/latest/python/python_component.html


https://docs.seldon.io/projects/seldon-core/en/latest/examples/triton_examples.html

https://docs.seldon.io/projects/seldon-core/en/latest/examples/triton_mnist_e2e.html

https://docs.seldon.io/projects/seldon-core/en/latest/reference/integration_nvidia_link.html

