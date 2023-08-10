# Jupyter PyTorch

Run Jupyter lab or notebook with a Python, iPython and Pytorch included.

## About PyTorch

PyTorch is an open-source deep learning framework developed by Facebook's AI Research lab (FAIR). It provides a flexible and dynamic computational graph, allowing developers to build and train neural networks. Unlike some other frameworks, PyTorch enables defining and modifying network architectures on-the-fly, making experimentation and debugging easier.

One of PyTorch's essential features is automatic differentiation, which is crucial for training neural networks using gradient-based optimization algorithms like backpropagation. Additionally, PyTorch supports GPU acceleration, enabling faster computation during model training.

The framework has gained popularity among researchers and developers due to its ease of use and extensive community support. With a large and active community, users can find abundant resources, tutorials, and libraries to support their deep learning projects.


## Pre-built Images

Docker images are built automatically through a GitHub Actions workflow and hosted at the GitHub Container Registry.

#### Version Tags

The `:latest` tag points to `:latest-cuda`

Tags follow these patterns:

##### _CUDA_
- `:[pytorch-version]-py[python-version]-cuda-[x.x.x]-base-[ubuntu-version]`

- `:latest-cuda` -> `:2.0.1-py3.10-cuda-11.8.0-base-22.04`

##### _ROCm_
- `:[pytorch-version]-py[python-version]-rocm-[x.x.x]-runtime-[ubuntu-version]`

- `:latest-rocm` -> `:2.0.1-py3.10-rocm-5.4.2-runtime-22.04`

##### _CPU_
- `:[pytorch-version]-py[python-version]-ubuntu-[ubuntu-version]`

- `:latest-cpu` -> `:2.0.1-py3.10-cpu-22.04` 

Browse [here](https://github.com/ai-dock/jupyter-pytorch/pkgs/container/jupyter-pytorch) for an image suitable for your target environment.

You can also self-build from source by editing `.env` and running `docker compose build`.

Supported Python versions: `3.11`, `3.10`, `3.9`, `3.8`

Supported Pytorch versions: `2.0.1`, `1.13.1` 

Supported Platforms: `NVIDIA CUDA`, `AMD ROCm`, `CPU`

## Run Locally

A 'feature-complete' docker-compose.yaml file is included for your convenience. All features of the image are included - Simply edit the environment variables, save and then type `docker compose up`.

If you prefer to use the standard `docker run` syntax, the command to pass is `init.sh`.

## Run in the Cloud

This image should be compatible with any GPU cloud platform. You simply need to pass environment variables at runtime. 

>[!NOTE]  
>Please raise an issue on this repository if your provider cannot run the image.

__Container Cloud__

Container providers don't give you access to the docker host but are quick and easy to set up. They are often inexpensive when compared to a full VM or bare metal solution.

All images built for ai-dock are tested for compatibility with both [vast.ai](https://cloud.vast.ai/?ref_id=62897&template_id=7c6e86d5035f93c0d1a11c9e3f73be09) and [runpod.io](https://runpod.io/gsc?template=qcnhzcoflg&ref=m0vk9g4f).

Images that include Jupyter are also tested to ensure compatibility with [Paperspace Gradient](https://console.paperspace.com/signup?R=FI2IEQI)

See a list of pre-configured templates [here](#pre-configured-templates)

>[!WARNING]  
>Container cloud providers may offer both 'community' and 'secure' versions of their cloud. If your usecase involves storing sensitive information (eg. API keys, auth tokens) then you should always choose the secure option.

__VM Cloud__

Running docker images on a virtual machine/bare metal server is much like running locally.

You'll need to:
- Configure your server
- Set up docker
- Clone this repository
- Edit `.env`and `docker-compose.yml`
- Run `docker compose up`

Find a list of compatible VM providers [here](#compatible-vm-providers).

### Connecting to Your Instance

All services listen for connections at [`0.0.0.0`](https://en.m.wikipedia.org/wiki/0.0.0.0). This gives you some flexibility in how you interact with your instance:

_**Expose the Ports**_

This is fine if you are working locally but can be **dangerous for remote connections** where data is passed in plaintext between your machine and the container over http.

_**SSH Tunnel**_

This is the preferred method. You will only need to expose `port 22` (SSH) which can then be used with port forwarding to allow **secure** connections to your services.

If you are unfamiliar with port forwarding then you should read the guides [here](https://www.digitalocean.com/community/tutorials/ssh-essentials-working-with-ssh-servers-clients-and-keys?refcode=405a7b000d31#setting-up-ssh-tunnels) and [here](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-tunneling-on-a-vps?refcode=405a7b000d31).

## Environment Variables

| Variable              | Description |
| --------------------- | ----------- |
| `GPU_COUNT`           | Limit the number of available GPUs |
| `JUPYTER_MODE`        | `lab` (default), `notebook` |
| `JUPYTER_TOKEN`       | Manually set your password |
| `PROVISIONING_SCRIPT` | URL of a remote script to execute on init. See [note](#provisioning-script). |
| `RCLONE_*`            | Rclone configuration - See [rclone documentation](https://rclone.org/docs/#config-file) |
| `SKIP_ACL`            | Set `true` to skip modifying workspace ACL |
| `SSH_PUBKEY`          | Your public key for SSH |
| `WORKSPACE`           | A volume path. Defaults to `/workspace/` |

Environment variables can be specified by using any of the standard methods (`docker-compose.yaml`, `docker run -e...`). Additionally, environment variables can also be passed as parameters of `init.sh`.

Passing environment variables to init.sh is usually unnecessary, but is useful for some cloud environments where the full `docker run` command cannot be specified.

Example usage: `docker run -e STANDARD_VAR1="this value" -e STANDARD_VAR2="that value" init.sh EXTRA_VAR="other value"`

## Provisioning script

It can be useful to perform certain actions when starting a container, such as creating directories and downloading files.

You can use the environment variable `PROVISIONING_SCRIPT` to specify the URL of a script you'd like to run.

If you are running locally you may instead opt to mount an executable script at `/opt/ai-dock/bin/provisioning.sh`.

>[!NOTE]  
>`supervisord` will not spawn any processes until the provisioning script has completed.

>[!WARNING]  
>Only use scripts that you trust and which cannot be changed without your consent.

## Software Management

A small software collection is installed by apt-get. This is mostly to provide basic functionality, but also includes `openssh-server` as the OS vendor is likely to be first to patch any security issues.

All other software is installed into its own environment by `micromamba`, which is a drop-in replacement for conda/mamba. Read more about it [here](https://mamba.readthedocs.io/en/latest/user_guide/micromamba.html).

Micromamba environments are particularly useful where several software packages are required but their dependencies conflict. 

### Installed Micromamba Environments

| Environment    | Packages / Rationale |
| -------------- | ----------------------------------------- |
| `base`         | micromamba's base environment |
| `system`       | `supervisord`, `rclone` - latest versions |
| `jupyter`      | `jupyter` |
| `python_[ver]` | `python` |

If you are extending this image or running an interactive session where additional software is required, you should almost certainly create a new environment first. See below for guidance.

### Useful Micromamba Commands

| Command                              | Function |
| -------------------------------------| --------------------- |
| `micromamba env list`                | List available environments |
| `micromamba activate [name]`         | Activate the named environment |
| `micromamba deactivate`              | Close the active environment |
| `micromamba run -n [name] [command]` | Run a command in the named environment without activating |

All ai-dock images create micromamba environments using the `--experimental` flag to enable hardlinks which can save disk space where multiple environments are available.

To create an additional micromamba environment, eg for python, you can use the following:

`micromamba --experimental create -y -c conda-forge -n [name] python=3.10`

## Volumes

Data inside docker containers is ephemeral - You'll lose all of it when the container is destroyed.

You may opt to mount a data volume at `/workspace` - This is a directory that ai-dock images will look for to make downloaded data available outside of the container for persistence. 

This is usually of importance where large files are downloaded at runtime or if you need a space to save your work. This is the ideal location to store Jupyter notebooks(.ipynb) and datasets.

You can define an alternative path for the workspace directory by passing the environment variable `WORKSPACE=/my/alternative/path/` and mounting your volume there. This feature will generally assist where cloud providers enforce their own mountpoint location for persistent storage.

The provided docker-compose.yaml will mount the local directory `./workspace` at `/workspace`.

As docker containers generally run as the root user, new files created in /workspace will be owned by uid 0(root).

To ensure that the files remain accessible to the local user that owns the directory, the docker entrypoint will set a default ACL on the directory by executing the commamd `setfacl -d -m u:${WORKSPACE_UID}:rwx /workspace`.

If you do not want this, you can set the environment variable `SKIP_ACL=true`.

## Running Services

This image will spawn multiple processes upon starting a container because some of our remote environments do not support more than one container per instance.

All processes are managed by [supervisord](https://supervisord.readthedocs.io/en/latest/) and will restart upon failure until you either manually stop them or terminate the container.

>[!NOTE]  
>*Some of the included services would not normally be found **inside** of a container. They are, however, necessary here as some cloud providers give no access to the host; Containers are deployed as if they were a virtual machine.*

### Jupyter

The jupyter server will launch a `lab` instance unless you specify `JUPYTER_MODE=notebook`.

Jupyter server will listen on port `8888`.

A python kernel will be installed coresponding with the python version of the image.

Jupyter's official documentation is available at https://jupyter.org/ 

### SSHD

A SSH server will be started if at least one valid public key is found inside the running container in the file `/root/.ssh/authorized_keys`

There are several ways to get your keys to the container.

- If using docker compose, you can paste your key in the local file `config/authorized_keys` before starting the container.
 
- You can pass the environment variable `SSH_PUBKEY` with your public key as the value.

- Cloud providers often have a built-in method to transfer your key into the container

If you choose not to provide a public key then the SSH server will not be started.

To make use of this service you should map `port 22` to a port of your choice on the host operating system.

See [this guide](https://www.digitalocean.com/community/tutorials/ssh-essentials-working-with-ssh-servers-clients-and-keys?refcode=405a7b000d31#) by DigitalOcean for an excellent introduction to working with SSH servers.

>[!NOTE]  
>_SSHD is included because the end-user should be able to know the version prior to deloyment. Using a providers add-on, if available, does not guarantee this._

### Rclone mount

Rclone allows you to access your cloud storage from within the container by configuring one or more remotes. If you are unfamiliar with the project you can find out more at the [Rclone website](https://rclone.org/).

Any Rclone remotes that you have specified, either through mounting the config directory or via setting environment variables will be mounted at `/workspace/remote/[remote name]`. For this service to start, the following conditions must be met:

- Fuse3 installed in the host operating system
- Kernel module `fuse` loaded in the host
- Host `/etc/passwd` mounted in the container
- Host `/etc/group` mounted in the container
- Host device `/dev/fuse` made available to the container
- Container must run with `cap-add SYS_ADMIN`
- Container must run with `securiry-opt apparmor:unconfined`
- At least one remote must be configured

The provided docker-compose.yaml includes a working configuration (add your own remotes).

In the event that the conditions listed cannot be met, `rclone` will still be available to use via the CLI - only mounts will be unavailable.

If you intend to use the `rclone create` command to interactively generate remote configurations you should ensure `port 53682` is accessible. See https://rclone.org/remote_setup/ for further details.

>[!NOTE]  
>_Rclone is included to give the end-user an opportunity to easily transfer files between the instance and their cloud storage provider._

>[!WARNING]  
>You should only provide auth tokens in secure cloud environments.

### Logtail

This script follows and prints the log files for each of the above services to stdout. This allows you to follow the progress of all running services through docker's own logging system.

If you are logged into the container you can follow the logs by running `logtail.sh` in your shell.

## Open Ports

Some ports need to be exposed for the services to run or for certain features of the provided software to function


| Open Port             | Service / Description |
| --------------------- | --------------------- |
| `22`                  | SSH server            |
| `8888`                | Jupyter server |
| `53682`               | Rclone interactive config |

## Pre-Configured Templates

**Vast.​ai**

[jupyter-pytorch:latest](https://cloud.vast.ai/?ref_id=62897&template_id=7c6e86d5035f93c0d1a11c9e3f73be09)

---

**Runpod.​io**

[jupyter-pytorch:latest](https://runpod.io/gsc?template=qcnhzcoflg&ref=m0vk9g4f)

---

**Paperspace**

- Create a [new notebook](https://console.paperspace.com/signup?R=FI2IEQI) with the `Start from Scratch` template.
- Select `Advanced options`
- In Container Name enter `ghcr.io/ai-dock/jupyter-pytorch:latest`
- In Registry Username enter `x` (Paperspace bug)
- In Command enter `init.sh WORKSPACE=/notebooks`

---

>[!NOTE]  
>These templates are configured to use the `latest` tag but you are free to change to any of the available CUDA tags listed [here](https://github.com/ai-dock/jupyter-pytorch/pkgs/container/jupyter-pytorch)

## Compatible VM Providers

Images that do not require a GPU will run anywhere - Use an image tagged `:*-cpu-xx.xx`

Where a GPU is required you will need either `:*cuda*` or `:*rocm*` depending on the underlying hardware.

A curated list of VM providers currently offering GPU instances:

- [Akami/Linode](https://www.linode.com/lp/refer/?r=d49cf667cec6bcbb6c2d7d70665c5c9b9a5a4b95)
- [Amazon Web Services](https://aws.amazon.com)
- [Google Compute Engine](https://cloud.google.com)
- [Vultr](https://www.vultr.com/?ref=9519053-8H)

---

_The author ([@robballantyne](https://github.com/robballantyne)) may be compensated if you sign up to services linked in this document. Testing multiple variants of GPU images in many different environments is both costly and time-consuming; This helps to offset costs_