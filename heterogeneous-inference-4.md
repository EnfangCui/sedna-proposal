# Heterogeneous Inference
## Motivation
There is heterogeneous hardware and multiple AI platforms at the edge. Hardware such as Huawei Ascend, Baidu Kunlun, Cambrian, etc., AI platform such as MindSpore, Tensorflow, PaddlePaddle, etc. Different hardware adapts to different AI frameworks. This results in complicated model adaptation and heavy workload when deploying AI models. Heterogeneous inference is needed to automatically select AI serving platforms according to the hardware platform type, and automatically perform model format conversion and model serving.


### Goals
* Automatically select AI serving platforms (e.g., TFserving, PaddleServing, MindSpore Serving, Tensorflow lite, Paddle lite, MNN, OpenVINO) according to the hardware platform type, and automatically perform model format conversion and model serving.

## Proposal
We propose using Kubernetes Custom Resource Definitions (CRDs) to describe the heterogeneous inference specification/status and a controller to synchronize these updates between edge and cloud.
![image](https://raw.githubusercontent.com/EnfangCui/sedna-proposal/main/imgs/heterogeneous-inference-service-crd.png)

### Use Cases

* User can create a heterogeneous inference service with providing a raw DNN model.

* Users can get the model conversion status of heterogeneous inference service.

* Users can get the saved converted model. The model files are stored on the cloud.

## Design Details

### CRD API Group and Version
The `HeteroInferenceService` CRD will be namespace-scoped.
The tables below summarize the group, kind and API version details for the CRD.

* HeteroInferenceService

| Field                 | Description             |
|-----------------------|-------------------------|
|Group                  | sedna.io     |
|APIVersion             | v1alpha1                |
|Kind                   | HeteroInferenceService   |

## Controller Design
The heterogeneous inference controller starts one goroutine called  `hetero-inference` controller. 
- hetero-inference: watch the updates of hetero-inference-task crds, and create the workers to complete the task.

### Heterogeneous Inference Controller
![](./images/hetero-inference-controller.png)

The hetero-inference controller watches for the updates of hetero-inference tasks and the corresponding pods against the K8S API server.
Updates are categorized below along with the possible actions:

| Update Type                    | Action                                       |
|-------------------------------|---------------------------------------------- |
|New Hetero-inference-service Created             |Create the cloud/edge worker|
|Hetero-inference-service Deleted                 | NA. These workers will be deleted by GM.|
|The corresponding pod created/running/completed/failed                 | Update the status of hetero-inference task.|

### Details of api between GM(cloud) and Worker(cloud)
1. Model conversion worker sends model conversion info to GM:

    ```
    // POST /sedna/workers/<worker-name>/info
    ```
 
    ```json
   {
       "name": "worker-name",
       "namespace": "default",
       "ownerName": "heteroinferenceservice-name",
       "ownerKind": "heteroinferenceservice",
       "kind": "inference",
       "status": "completed/failed/running",
       "taskInfo": {
           "convesionState": "success",
           "startTime": "2020-11-03T08:39:22.517Z",
           "finishTime": "2020-11-03T08:50:22.517Z"
       }
   }
   ```

### Flow of Heterogeneous Inference
- The flow of heterogeneous inference service creation:

![image](https://raw.githubusercontent.com/EnfangCui/sedna-proposal/main/imgs/hetero-inference-flow-creation.png)
