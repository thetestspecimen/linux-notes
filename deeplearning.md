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

## Making a virtual environment

Open a terminal:

	conda create --name new-environment-name

Check it has been created by listing all environments:

	conda env list

Activate that environment:

	conda activate
	source activate new-environment-name

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

