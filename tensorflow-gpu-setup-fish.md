# How to setup a Conda environment in fish shell that uses a NVIDIA GPU

As I primarily use fish shell rather than bash, I thought it might be useful to detail how to setup a Conda environment that can utilise a NVIDIA GPU from within fish shell, as all instructions tend to be for bash.

The general guidance that the following information is taken from can be found [here](https://www.tensorflow.org/install/pip#step-by-step_instructions), which details how to do the same thing for bash terminal.

## Install Miniconda

In general I would always recommend [Miniconda](https://docs.conda.io/en/latest/miniconda.html) over a full Anaconda install, but it is not essential. However, it should be noted that even the instructions from the official [TensorFlow guide](https://www.tensorflow.org/install/pip#step-by-step_instructions) recommend [Miniconda](https://docs.conda.io/en/latest/miniconda.html).

```bash
curl https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -o Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh
```

Follow the prompts.

After install you will need to initialise miniconda for fish so run:

```bash
conda init fish
```

Update conda:

```bash
conda update conda
```

You may also want to stop conda from automatically activating the base environment every time you open a new terminal (this is optional):

```bash
conda config --set auto_activate_base false
```

## Set conda channel and flexible priority

### Conda channel

This sets the conda-forge channel to the highest priority, which should help keep packages up to date:

```bash
conda config --add channels conda-forge
```

### Channel priority

As per the documentation for conda:

> As of version 4.6.0, Conda has a strict channel priority feature. Strict channel priority can dramatically speed up conda operations and also reduce package incompatibility problems. We recommend setting channel priority to "strict" when possible.
>
> -[conda.io](https://conda.io/projects/conda/en/latest/user-guide/tasks/manage-channels.html)

In our particular case we cannot set the ```channel_priority ``` as strict. This is because later in this tutorial only flexible channel priority will successfully install and setup  ```cuda-nvcc```. So please set the ```channel_priority``` as follows: 

```bash
conda config --set channel_priority flexible
```

Other options are as follows:

```bash
channel_priority (ChannelPriority)

Accepts values of 'strict', 'flexible', and 'disabled'. The default
value is 'flexible'. With strict channel priority, packages in lower
priority channels are not considered if a package with the same name
appears in a higher priority channel. With flexible channel priority,
the solver may reach into lower priority channels to fulfill
dependencies, rather than raising an unsatisfiable error. With channel
priority disabled, package version takes precedence, and the
configured priority of channels is used only to break ties. In
previous versions of conda, this parameter was configured as either
True or False. True is now an alias to 'flexible'.

channel_priority: flexible
```

## Create a new conda environment

We will create a new environment with the name ```tf``` using Python version 3.11:

```bash
conda create --name tf python=3.11
```

After creation activate the environment:

```bash
conda activate tf
```

## GPU Setup

You will of course need to have NVIDIA drivers installed on your system before you can proceed through this section.

In my case (Manjaro Linux) they are already installed. How this is done will vary depending on your distro. In my case this command will suffice with a reboot afterwards:

```bash
sudo mhwd -a pci nonfree 0300
```

You could get the drivers from [source](https://www.nvidia.com/Download/index.aspx), but in general I would only recommend this if you **really** know what you are doing. Distro supplied drivers are usually sufficient.

You can check if everything is as it should be by running:

```bash
nvidia-smi
```

You should get an output something like this:

```bash
+---------------------------------------------------------------------------------------+
| NVIDIA-SMI 530.41.03              Driver Version: 530.41.03    CUDA Version: 12.1     |
|-----------------------------------------+----------------------+----------------------+
| GPU  Name                  Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf            Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                                         |                      |               MIG M. |
|=========================================+======================+======================|
|   0  NVIDIA GeForce GTX 1070         Off| 00000000:01:00.0  On |                  N/A |
|  0%   41C    P0               32W / 185W|    570MiB /  8192MiB |      0%      Default |
|                                         |                      |                  N/A |
+-----------------------------------------+----------------------+----------------------+
                                                                                         
+---------------------------------------------------------------------------------------+
| Processes:                                                                            |
|  GPU   GI   CI        PID   Type   Process name                            GPU Memory |
|        ID   ID                                                             Usage      |
|=======================================================================================|
|    0   N/A  N/A      3552      G   /usr/lib/Xorg                               293MiB |
|    0   N/A  N/A      3663      G   /usr/bin/gnome-shell                         84MiB |
|    0   N/A  N/A      4518      G   ...7368340,11203893294388247584,262144      122MiB |
|    0   N/A  N/A     47432      G   /usr/bin/nautilus                            32MiB |
|    0   N/A  N/A     48307      G   ...ures=SpareRendererForSitePerProcess       33MiB |
+---------------------------------------------------------------------------------------+
```

Now we can install CUDA and cuDNN (the versions below may need updating, please check the [source](https://www.tensorflow.org/install/pip#step-by-step_instructions)):

```bash
conda install -c conda-forge cudatoolkit=11.8.0
pip install nvidia-cudnn-cu11==8.6.0.163
```

We now need to set some paths. The paths will be set in an auto executed file, so that whenever you activate the conda environment these paths will be automatically set:

```bash
mkdir -p $CONDA_PREFIX/etc/conda/activate.d
echo 'set CUDNN_PATH (dirname (python -c "import nvidia.cudnn;print(nvidia.cudnn.__file__)"))' >> $CONDA_PREFIX/etc/conda/activate.d/env_vars.fish
echo 'set -gx LD_LIBRARY_PATH $LD_LIBRARY_PATH $CONDA_PREFIX/lib/ $CUDNN_PATH/lib' >> $CONDA_PREFIX/etc/conda/activate.d/env_vars.fish
```

Then we  check the file runs automatically:

```bash
conda deactivate
conda activate tf
```

The above should cause the file ```env_vars.fish``` to be run automatically on activation of the environment. You will know if this is successful if the tests in the "Verify the install" section complete successfully.

## Install TensorFlow

Apparently TensorFlow requires a quite recent version of pip, so we will update it here:

```bash
pip install --upgrade pip
```

Then we install TensorFlow. 

**Note: it is advised that TensorFlow should be installed with pip and NOT conda**

```bash
pip install tensorflow=="2.12.*"
```

## Verify the install

Verify that TensorFlow is installed:

```bash
python3 -c "import tensorflow as tf; print(tf.reduce_sum(tf.random.normal([1000, 1000])))"
```

If the last line is like the following then it worked:

```bash
tf.Tensor(-100.2298, shape=(), dtype=float32)
```

Then we can check if the GPU is set correctly:

```bash
python3 -c "import tensorflow as tf; print(tf.config.list_physical_devices('GPU'))"
```

The last line should return something similar to the following:

```bash
[PhysicalDevice(name='/physical_device:GPU:0', device_type='GPU')]
```

## Potential additional settings

The [source](https://www.tensorflow.org/install/pip#step-by-step_instructions) states that you may need some additional settings in some circumstances. Specifically it mentions Ubuntu 22.04, but I can confirm Manjaro also has the same issue.

The error will be thrown in your code and will involve 'ptxas'.

If you encounter this error then please execute the following (again update versions from [source](https://www.tensorflow.org/install/pip#step-by-step_instructions) as required):

```bash
# Install NVCC
conda install -c nvidia cuda-nvcc=11.3.58
# set flag
echo 'set -gx XLA_FLAGS --xla_gpu_cuda_data_dir=$CONDA_PREFIX/lib/' >> $CONDA_PREFIX/etc/conda/activate.d/env_vars.fish
# Copy libdevice file to the required path
mkdir -p $CONDA_PREFIX/lib/nvvm/libdevice
cp $CONDA_PREFIX/lib/libdevice.10.bc $CONDA_PREFIX/lib/nvvm/libdevice/
```

## Final steps

Everything is now set. All that is left to do is to 'reboot' the conda environment to run the ```env_vars.fish``` file with the additional parameters for XLA_FLAGS:

```bash
conda deactivate
conda activate tf
```

## Run the code

Now you can run your code and the GPU should be used in tensorflow with no problem, even within things like a Jupyter Notebook or Jupyter Lab.

You can install and launch Jupyter Lab as follows:

```bash
conda install -c conda-forge jupyterlab
jupyter-lab
```



