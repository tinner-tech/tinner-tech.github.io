# How to Install Stable Diffusion via CLI on Ubuntu 22.04

This guide demonstrates how to create a Virtual Private Server (VPS) using [vpsai.io](https://vpsai.io), a provider that allows you to quickly spin up a VPS with GPU resources. You only pay for the time you use, so terminating the VPS instance before its expiration date will only incur charges for the actual usage period.

## How to Deploy a VPS with VPS AI

Before starting, ensure you've connected a wallet to https://cloud-beta.vpsai.io/, added credits to your account, and uploaded a valid SSH key to your dashboard. For more details on these steps, please refer to the following resources:
- [Connecting Your Wallet to the VPS AI Dashboard](https://docs.vpsai.io/vps-ai/product-guides/connecting-your-wallet-to-the-vps-ai-dashboard)
- [Adding Credit to your VPS AI Dashboard](https://docs.vpsai.io/vps-ai/product-guides/adding-credit-to-your-vps-ai-dashboard)
- [Generate SSH Keys](https://docs.vpsai.io/vps-ai/product-guides/generate-ssh-keys)

### Deploying a new VPS with GPU

Stable Diffusion requires substantial resources to generate images effectively. For optimal performance, I recommend configuring your VPS with the following specifications:
- vCPUs: 6
- Memory: 24 GiB
- Storage: 168 GiB
- GPU: Any Nvidia GPU (I recommend the NVIDIA V100)
- Operating System: Ubuntu 22.04 + NVIDIA Drivers + Docker

This setup should cost approximately $12 USD for 24 hours of operation. Terminating the VPS within this time frame will reduce your charge to only the duration used. If the VPS remains active past 24 hours and sufficient credits are available, it will automatically renew.

### Accessing Your VPS

```sh
ssh root@your.vps.ip.address
```
Upon connection, you'll see a prompt regarding the authenticity of the host, which you can confirm as follows:
```sh
yes
```

### Installing Anaconda

First, download and install Anaconda using the commands below. The installer script URL provided is subject to change, so ensure it's up to date.
```sh
wget https://repo.anaconda.com/archive/Anaconda3-2022.05-Linux-x86_64.sh
chmod +x Anaconda3-2022.05-Linux-x86_64.sh
./Anaconda3-2022.05-Linux-x86_64.sh -b
```

### Downloading the Model file from HuggingFace

```sh
wget -O sd-v1-4.ckpt https://huggingface.co/CompVis/stable-diffusion-v-1-4-original/resolve/main/sd-v1-4.ckpt
```

### Initialize Anaconda (conda)

After installing Anaconda, configure it using:
```sh
~/anaconda3/bin/conda config --set auto_activate_base false
~/anaconda3/bin/conda init
```
Note: Restart your terminal to apply these changes.

### Clone the Stable Diffusion Repository

```sh
git clone https://github.com/CompVis/stable-diffusion
cd stable-diffusion
```
Move the previously downloaded model file:
```sh
mkdir -p models/ldm/stable-diffusion-v1/
mv ../sd-v1-4.ckpt models/ldm/stable-diffusion-v1/model.ckpt
```

### Setting Up Your Conda Environment

Create and activate your conda environment:
```sh
conda env create -f environment.yaml
conda activate ldm
```

### Generating Images

Run the txt2img script:
```sh
python scripts/txt2img.py --prompt "a photograph of an astronaut riding a horse" --plms
```

Monitor GPU usage:
```sh
watch -n 1 nvidia-smi
```

### Transferring Images

To transfer images to your local machine:
```sh
scp username@your-vps-ip:/root/stable-diffusion/outputs/txt2img-samples/grid-0000.png /local/path
```
Image filenames increment with each new image, e.g., `grid-0001.png`.

## Bonus: Disabling the NSFW Safety Checker

Modify the NSFW safety checker in the `txt2img.py` script:
```sh
nano ./scripts/txt2img.py
```
In nano, use `ctrl + w` to search for `check_safety`. Replace the found method with:
```python
def check_safety(x_image):
    # Returns the images directly without checking for NSFW content
    return x_image, [False] * len(x_image)  # Assuming no images are flagged NSFW
```
Save and exit nano with `ctrl + x`, then `y`, and `Enter`.

This modification will allow the generation of images flagged as NSFW.
