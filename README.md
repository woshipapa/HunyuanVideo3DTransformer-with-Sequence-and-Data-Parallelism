# HunyuanVideo-2DParallel
This repository provides an implementation of HunyuanVideoTransformer with both single-stream and dual-stream architectures. It incorporates sequence parallelism using Ulysses' method and data parallelism based on a customized framework of accelerate and deepspeed. The implementation supports **both pre-training and inference stages.**



## Transformer_Hunyuan_Video
transformer_hunyuan_video.py is an extension of the original Diffusers library, which introduces support for both single-stream and dual-stream SP processing.


### Features
- Single-stream SP: Text and video inputs are processed separately before the attention mechanism. Specifically, the QKV matrices for text and video are computed independently before being passed into the attention block. This approach maintains the individual characteristics of each modality during processing.

- Dual-stream SP: Text and video inputs are concatenated as soon as they enter the block. This combined input is then used for the joint computation of the QKV matrices within the attention mechanism. This method allows for a more integrated representation of both modalities, leveraging their combined features throughout the attention process.
- Extended Utility Functions: Utilizes a custom utils library for distributed operations, including:
  -  Padding: Ensures input consistency across different tensor sizes.
  -  All2all: Distributes data across multiple devices or workers.
  -  Split and Gather: Efficient data splitting and collection operations, useful for parallel processing.


## Environment
The environment for this project is provided through a Docker image, ensuring that all dependencies are encapsulated for ease of deployment and consistency across different systems. The image is still being uploaded and will be available for use shortly.


## Accelerate and Deepspeed Support
This project supports integration with both accelerate and deepspeed for distributed training and inference.

- Data Parallelism: The **data_parallel_size** is configurable, allowing for parallel processing across multiple devices, speeding up training and inference.

- Sequence Parallelism: The **sequence_parallel_size** is also supported, enabling fine-grained parallelism across different sequence lengths and improving the- handling of large-scale data.

### BatchSamplerShard for DP+SP
A key modification in the accelerate framework is the enhancement of the BatchSamplerShard. This custom sampler now supports distributed data sampling for both Data Parallelism (DP) and Sequence Parallelism (SP). Specifically:

- DP+SP Distributed Sampling: This version ensures that data within the same SP group is consistent across devices. Previously, the original implementation only supported independent data on each device. With this change, all devices within the same SP group will sample the same batch, maintaining data consistency across the group.


## DeepSpeed Integration
In this project, DeepSpeed is further integrated through the mesh_device parameter used in deepspeed_initialize. This allows the initialization of device_mesh and the retrieval of the ProcessGroup, which is essential for handling distributed communication during training and inference.

- device_mesh: Initialized using the **mesh_device** parameter in **deepspeed_initialize**, enabling efficient communication across multiple devices.

- ProcessGroup: Once the **device_mesh** is initialized, the ProcessGroup is obtained, which is crucial for the distributed communication layer in training.

Moreover, in the accelerate pipeline, the **DeepSpeedEngine (wrapped model)** is extracted and encapsulated into a **ParallelManager** class, which simplifies the management of parallel tasks.

- ParallelManager: This class wraps the DeepSpeedEngine and handles the communication and synchronization tasks required for efficient distributed processing.

## Implementation Results
For a **720p, 49 fps video** processed with torch.checkpoint, we observed significant improvements in memory usage:

- SP=1: During forward propagation, the activation values used up to 15GB of memory.
- SP=4: By setting SP=4, the activation values during forward propagation dropped to 3GB, achieving a substantial memory reduction.
This demonstrates the efficiency of sequence parallelism in managing memory usage, making it easier to process larger videos without running into memory bottlenecks.

## References
- [HunyuanVideo](https://github.com/Tencent/HunyuanVideo): The official repository for Tencent's HunyuanVideo, a foundation for multi-modal video processing.
- [Diffusers Library](https://github.com/huggingface/diffusers): The original library on which `transformer_hunyuan_video.py` is based.
- [Accelerate Library](https://github.com/huggingface/accelerate): The Hugging Face library for distributed training.
- [DeepSpeed](https://github.com/microsoft/deepspeed): A deep learning optimization library for scaling models efficiently.


## Contribution
Feel free to fork this repository and submit pull requests with improvements or bug fixes.

