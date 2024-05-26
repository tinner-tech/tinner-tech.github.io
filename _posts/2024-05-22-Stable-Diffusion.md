# Stable Diffusion

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

and now any images that are interpreted as NSFW will continue to be generated. 




