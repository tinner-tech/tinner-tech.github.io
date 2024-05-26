# How to Install Stable Diffusion via CLI on Ubuntu 22.04

This guide will create a Virtual Private Server (VPS) using [vpsai.io](https://vpsai.io) as the provider. You can easily and quickly spin up a VPS AI instance with GPU resources, and the best part is you will only be charged for the resources used. If you terminate the VPS instance before it's expirey date, it will only charge you for the time used. 

## How to Deploy a VPS with VPS AI

For the purposes of simplicity, this guide will assume you have connected a wallet to https://cloud-beta.vpsai.io/, have added credits to your account, and have a valid SSH key added to your dashboard. If you need more information on how to do these things, please see:
- [Connecting Your Wallet to the VPS AI Dashboard](https://docs.vpsai.io/vps-ai/product-guides/connecting-your-wallet-to-the-vps-ai-dashboard)
- [Adding Credit to your VPS AI Dashboard](https://docs.vpsai.io/vps-ai/product-guides/adding-credit-to-your-vps-ai-dashboard)
- [Generate SSH Keys](https://docs.vpsai.io/vps-ai/product-guides/generate-ssh-keys)

### Deploying a new VPS with GPU

Stable Diffusion is a fairly resource-intensive application. In order for your VPS to properly generate images, you will need to be sure to allocate the proper resouces. 

To deploy a new GPU-enabled VPS to your dashboard, choose the following settings. It's possible you could get away with fewer vCPUs and less RAM, but to ensure this works the first time I would recommend the following setup.
- vCPUs: 6
- Memory: 24 GiB
- Storage: 168 GiB
- GPU: Any Nvidia GPU (I choose the NVIDIA V100)
- Operating System: Ubuntu 22.04 + NVIDIA Drivers + Docker
  - The NVIDIA Drivers are important here. They enable us to work with the GPU out of the box wihtout extra setup. 

As you can see, this should cost ~$12 USD for 1 Day (24 Hours) of operation.

```sh
ssh root@your.vps.ip.address
```

- The authenticity of host 'your.vps.ip.address (your.vps.ip.address)' can't be established.
ED25519 key fingerprint is {unique fingerprint}.
This key is not known by any other names.
Are you sure you want to continue connecting

```sh
yes
```

Download the Anaconda installer script from their website and install it. The download URL may change over time

```sh
wget https://repo.anaconda.com/archive/Anaconda3-2022.05-Linux-x86_64.sh
```

```sh
chmod +x Anaconda3-2022.05-Linux-x86_64.sh
```

```sh
# Install Anaconda without prompts
./Anaconda3-2022.05-Linux-x86_64.sh -b
```

Get the model file

```sh
wget -O sd-v1-4.ckpt https://huggingface.co/CompVis/stable-diffusion-v-1-4-original/resolve/main/sd-v1-4.ckpt
```

Initialise conda, but tell it not to activate each time the shell starts.

```sh
~/anaconda3/bin/conda config --set auto_activate_base false
~/anaconda3/bin/conda init
```

You will need to restart your terminal for these changes to take effect. Before moving on, close your current terminal and open a new one.

Get the Stable Diffusion repository

```sh
git clone https://github.com/CompVis/stable-diffusion
cd stable-diffusion
```

Move previously downloaded model

```sh
mkdir -p models/ldm/stable-diffusion-v1/
mv ../sd-v1-4.ckpt models/ldm/stable-diffusion-v1/model.ckpt
```

Setup your conda environment
```sh
conda env create -f environment.yaml
```

Activate your conda environment
```sh
conda activate ldm
```

Run the txt2img script with a prompt:
```sh
python scripts/txt2img.py --prompt "a photograph of an astronaut riding a horse" --plms 
```

To see the GPU usage while stable diffusion is creating your image you can open a new terminal and use the command:
```sh
watch -n 1 nvidia-smi
```

To transfer the generated image from the VPS to your local machine, run this command from your local machine
```sh
scp username@your-vps-ip:/root/stable-diffusion/outputs/txt2img-samples/grid-0000.png /local/path
```

each subsequent image generated will increment the grid name. So the second Image will be
`grid-0001.png`

## Bonus: Disabling the NSFW Safety Checker

To disable the safety checker you need to modify a method in the `txt2img.py` script.

```sh
nano ./scripts/txt2img.py 
```

Once in the nano editor, click `ctrl + w` to search. Type `check_safety` and press `Enter`.

It should take you to a method that looks like the following:

```python
def check_safety(x_image):
    safety_checker_input = safety_feature_extractor(numpy_to_pil(x_image), return_tensors="pt")
    x_checked_image, has_nsfw_concept = safety_checker(images=x_image, clip_input=safety_checker_input.pixel_values)
    assert x_checked_image.shape[0] == len(has_nsfw_concept)
    for i in range(len(has_nsfw_concept)):
        if has_nsfw_concept[i]:
            x_checked_image[i] = load_replacement(x_checked_image[i])
    return x_checked_image, has_nsfw_concept
```

you can replace that with the following code:

```python
def check_safety(x_image):
    # Returns the images directly without checking for NSFW content
    return x_image, [False] * len(x_image)  # Assuming no images are flagged NSFW
```

to save and exit the nano editor, click `ctrl + x`, then `y`, and then `Enter`.

and now any images that are interpreted as NSFW will continue to be generated. 




