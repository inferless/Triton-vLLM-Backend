# Steps to use Triton with vLLM-Backend
The Triton backend for vLLM is designed to run supported models on a vLLM engine.

## STEP 1: Install the Triton Server with vLLM backend
To install and deploy the vLLM backend, the recommended method is using the Pre-Built Docker Container. Follow these steps:

1. Pull the `tritonserver:<xx.yy>-vllm-python-py3` container from the [NGC registry](https://catalog.ngc.nvidia.com/orgs/nvidia/containers/tritonserver), where \<xx.yy\> represents the version of Triton you want to use. Note that Triton's vLLM container is available starting from the 23.10 release.
```bash
docker pull nvcr.io/nvidia/tritonserver:24.02-vllm-python-py3
```

## STEP 2: Model Repository
Setting up the model repository for vLLM is akin to setting up Triton Server. You can do this by copying the provided [model repository](./vllm_model) and modifying the model configuration in the [`model.json`](./vllm_model/1/model.json) file.

In the [`model.json`](./vllm_model/1/model.json) file, you'll find a key-value dictionary format. Modify the `model` value to specify the model you want to use. This file is essential as it provides the necessary configuration information to vLLM's AsyncLLMEngine during model initialization.

These files have been copied from [here](https://github.com/triton-inference-server/vllm_backend/blob/main/samples/model_repository/).


## STEP 3: START the Server
Start the Triton server in explicit mode. These commands start Triton server in explicit mode, allowing you to manage model loading and unloading according to your needs. You can load the model using the provided endpoint, perform inference, and then unload the model when it's no longer needed. you can use the following commands:

```bash
docker run --gpus all --rm -p 8000:8000 --shm-size=1g -v ${PWD}/model_repository:/models nvcr.io/nvidia/tritonserver:24.02-vllm-python-py3 bash -c "tritonserver --model-repository=/models --model-control-mode=explicit"
```
### Load the Model:
```bash
curl -X POST localhost:8000/v2/repository/models/vllm_model/load
```
### Inference from the Model:
```bash
curl -X POST localhost:8000/v2/models/vllm_model/generate -d '{"text_input": "What is Triton Inference Server?", "parameters": {"stream": false, "temperature": 0, "max_tokens": 200}}'
```
### Unload the Model:
```bash
curl -X POST localhost:8000/v2/repository/models/vllm_model/unload
```
