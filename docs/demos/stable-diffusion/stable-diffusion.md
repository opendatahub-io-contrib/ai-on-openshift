# Text to Image using Stable Diffusion with DreamBooth

Stable Diffusion is a generative model that creates high-quality images by gradually denoising from random noise. DreamBooth fine-tuning customizes this model by training it on specific examples, allowing it to generate personalized images based on unique tokens and descriptive prompts.

![alt text](img/dreambooth_example.png)

Credit: DreamBooth

See the original DreamBooth [project](https://dreambooth.github.io/) homepage for more details on what this fine-tuning method achieves.

## Requirements

- An OpenShift cluster with RHOAI or ODH deployed.
- CSI driver capable of providing RWX volumes.
- A node with a GPU card. The example has been tested with a AWS p3.8xlarge node.

!!!info
    The full source and instructions for this demo are available on [this repo](https://github.com/opendatahub-io/distributed-workloads/tree/main/examples/stable-diffusion-dreambooth) and is based on this sample [code](https://docs.ray.io/en/latest/train/examples/pytorch/dreambooth_finetuning.html).