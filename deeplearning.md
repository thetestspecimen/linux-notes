# Deep Learning

The setup described in this file is based on the following:

Hardware:

- AMD RX560 2G GPU

Software:

- Fedora 31
- Anaconda
- TensorFlow
- Keras
- Plaidml

## Problems

Unfortunately, there is not really great support for AMD GPUs at the moment in software
required for deep learning. Even less so on Fedora.

If you happen to run Ubuntu, CentOS, RHEL or SLES you can take a look at [ROCm](https://github.com/RadeonOpenCompute/ROCm), which may help.

## Recommendations

You may note that my GPU is certainly not top of the line, but the fact is it is a **lot** better than using a CPU
for the same task. This is something worth bearing in mind. So even though it was a hassle to find out how to make 
the GPU work, it was definitely worth it.

I have run a simple test to hammer home the point:

I have a an i7 4790k overclocked to 4.6Ghz (this is an older processor with 4 physical cores (8 threads), but it is far from slow). I ran the same test of 
12 epochs on both the CPU and the GPU. Here are the times in seconds for each epoch:

- CPU: 46 seconds
- GPU: 16 seconds

So with the GPU it takes a third of the time, and it isn't even a good GPU! Imagine what your RX Vega or RX 5000 series could manage (or multiples of them).

## Install Anaconda

Anaconda is a great piece of software, and is relatively easy to install.

Some packages you will need before install:

	sudo dnf install libXcomposite libXcursor libXi libXtst libXrandr alsa-lib mesa-libEGL libXdamage mesa-libGL libXScrnSaver

Download the [installer](https://www.anaconda.com/download/#linux)

Enter the following to install once downloaded:

	bash ~/Downloads/Anaconda3-2020.02-Linux-x86_64.sh

Obviously adjust the above based on download location and actual file name.

Now go through the install process:

- Agree to the license
- Choose an install location or accept the default (default recommended)
- Do you wish the installer to initialise Anaconda (recommended: yes)

Once finished close and re-open the terminal window.

You will note that the terminal now has a "(base)" at the beginning. This means you are in the Anaconda virtual environment. However,
you may not want to be in the virtual environment every time you open a terminal window, so if you want to switch this off, paste the
following into the terminal:

	conda config --set auto_activate_base False

...and you are done.

One last thing. It may be worth updating, so run this in a terminal:

	conda update conda

## Install OpenCL (Optional)

If you already have OpenCL installed for your graphics card you can skip this step. To check whether you have do the following:

	sudo dnf install clinfo
	clinfo

If you see the output:

	Number of platforms                               0

Then you do not have the necessary OpenCL modules installed and should install the following:

	sudo dnf install mesa-libOpenCL

This will install all the necessary packages for you to use your graphics card with plaid-ml

## Install amdgpu-pro drivers for OpenCL

These are card drivers directly from AMD. It is possible that the OpenCL drivers included with Fedora will
not allow you to utilise plaidml later in this setup routine.

If this is the case, the best solution is to use the ammdgpu-pro drivers. 

Unfortunately, there is no easy option for Fedora, so it becomes a bit of a hack.

The information that follows has been taken from [here](https://ask.fedoraproject.org/t/guide-install-amdgpu-pro-opencl-in-fedora-32/7929).

### Download the drivers

Firstly, you will need to download the latest AMD drivers from the AMD website for your card. There will be no option for Fedora, so 
download the drivers for RedHat or CentOS. As of writing it is RHEL 8.2 / CentOS 8.2 (Revision number 20.40).

Go to [this](https://www.amd.com/en/support) webpage and go through the process of getting the correct drivers for your card.

### Extract the drivers

Now extract the drivers to the `/var/local` folder.

```
tar xf /path/to/amdgpu-pro-xx-xx-xxxxxx-rhel-x.tar.xz
sudo mv amdgpu-pro-xx-xx-xxxxxx-rhel-x /var/local/amdgpu
```
### Create local repo file

Create a local repo file at `/etc/yum.repos.d/amdgpu.repo` containing the following:

```
[amdgpu]
name=AMDGPU Packages
baseurl=file:///var/local/amdgpu/
enabled=1
skip_if_unavailable=1
gpgcheck=0
cost=500
metadata_expire=300
```
### Install packages

If you have a POLARIS card or older:

	sudo dnf install libdrm-amdgpu libdrm-amdgpu-common clinfo-amdgpu-pro opencl-amdgpu-pro-comgr amdgpu-pro-core opencl-orca-amdgpu-pro-icd libopencl-amdgpu-pro

If you have a VEGA card or newer:

	sudo dnf install libdrm-amdgpu libdrm-amdgpu-common clinfo-amdgpu-pro opencl-amdgpu-pro-comgr amdgpu-pro-core opencl-amdgpu-pro-icd libopencl-amdgpu-pro

In both cases the `amdgpu-core` dependency will fail to install, but it is not needed so it doesn't matter.

### Disable mesa runtime

If you have the Mesa runtime for OpenCL installed just rename it:

	sudo mv /etc/OpenCL/vendors/mesa.icd /etc/OpenCL/vendors/mesa.icd.bk

### Future upgrading

As the rpm files are locally stored they will not update automatically, so if you want to update the files you will need to download the new drivers from the 
website and replace those stored in ```/var/local/amdgpu``` with the new ones.

Then run a normal upgrade in Fedora.

## Making a virtual environment

Open a terminal:

	conda create --name new-environment-name

Check it has been created by listing all environments:

	conda env list

Activate that environment:

	conda activate new-environment-name

## Installing packages

Once you are in the new environment, you can install the packages you need:

	conda install PACKAGENAME

...and check what packages are installed:

	conda list


## Install TensorFlow and Keras

	conda install keras
	conda install tensorflow

## Install plaidml

Plaidml is what will allow the use of an AMD GPU

Within your conda virtual environment:

	pip install plaidml-keras plaidbench

Then we run the setup:

	plaidml-setup

Below is what appears when you run the setup.

There are three questions to answer in the text below:

- Enable experimental device support? (y,n)[n]:y
- Default device? (1,2)[1]:2 **(this may vary for you, depending on the devices in your computer. Select the number that represents your GPU)**
- Save settings to /home/testspecimen/.plaidml? (y,n)[y]:y

```
PlaidML Setup (0.7.0)

Thanks for using PlaidML!

The feedback we have received from our users indicates an ever-increasing need
for performance, programmability, and portability. During the past few months,
we have been restructuring PlaidML to address those needs.  To make all the
changes we need to make while supporting our current user base, all development
of PlaidML has moved to a branch — plaidml-v1. We will continue to maintain and
support the master branch of PlaidML and the stable 0.7.0 release.

Read more here: https://github.com/plaidml/plaidml 

Some Notes:
  * Bugs and other issues: https://github.com/plaidml/plaidml/issues
  * Questions: https://stackoverflow.com/questions/tagged/plaidml
  * Say hello: https://groups.google.com/forum/#!forum/plaidml-dev
  * PlaidML is licensed under the Apache License 2.0
 

Default Config Devices:
   llvm_cpu.0 : CPU (via LLVM)

Experimental Config Devices:
   llvm_cpu.0 : CPU (via LLVM)
   opencl_amd_baffin.0 : Advanced Micro Devices, Inc. Baffin (OpenCL)

Using experimental devices can cause poor performance, crashes, and other nastiness.

Enable experimental device support? (y,n)[n]:y

Multiple devices detected (You can override by setting PLAIDML_DEVICE_IDS).
Please choose a default device:

   1 : llvm_cpu.0
   2 : opencl_amd_baffin.0

Default device? (1,2)[1]:2

Selected device:
    opencl_amd_baffin.0

Almost done. Multiplying some matrices...
Tile code:
  function (B[X,Z], C[Z,Y]) -> (A) { A[x,y : X,Y] = +(B[x,z] * C[z,y]); }
Whew. That worked.

Save settings to /home/testspecimen/.plaidml? (y,n)[y]:y
Success!
```
The final thing to do to make sure it uses the GPU is to include the following lines
in your python file that contains your actual code:

```python
import plaidml.keras
plaidml.keras.install_backend()
```

That's it!

