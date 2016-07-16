# AWSforGPU

##Steps for creating ubuntu/keras enabled instance of AWS.

###Create ubuntu instance of AWS, with volume size of 20GB+

###Run the following (credit: https://github.com/BVLC/caffe/wiki/Caffe-on-EC2-Ubuntu-14.04-Cuda-7)

This installs the NVIDIA driver/cuda stuff
```
wget http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1404/x86_64/cuda-repo-ubuntu1404_7.5-18_amd64.deb
sudo dpkg -i cuda-repo-ubuntu1404_7.5-18_amd64.deb

sudo apt-get update
sudo apt-get upgrade -y
sudo apt-get install -y opencl-headers build-essential protobuf-compiler \
    libprotoc-dev libboost-all-dev libleveldb-dev hdf5-tools libhdf5-serial-dev \
    libopencv-core-dev  libopencv-highgui-dev libsnappy-dev libsnappy1 \
    libatlas-base-dev cmake libstdc++6-4.8-dbg libgoogle-glog0 libgoogle-glog-dev \
    libgflags-dev liblmdb-dev git python-pip gfortran

sudo apt-get clean

sudo apt-get install -y linux-image-extra-`uname -r` linux-headers-`uname -r` linux-image-`uname -r`

sudo apt-get install -y cuda
sudo apt-get clean
```

Verify your instance now recognizes your GPUs with the following:
```
nvidia-smi
```

###Install Anaconda, Keras and upgrade Theano to bleeding edge

###Add the cuda path file to your $PATH
```
vim .bashrc
```
Add the following to the bottom of the file
If you installed anaconda correctly, you should see a similar line for anaconda
```
export PATH="/usr/local/cuda/bin:$PATH"
export LD_LIBRARY_PATH="/usr/local/cuda/lib64:$LD_LIBRARY_PATH"
```
Run the following to execute the new .bashrc
```
source .bashrc
```

###Change the theano config file
```
vim ~/.theanorc
```
Add the following code and save/quit (credit: http://machinelearningmastery.com/develop-evaluate-large-deep-learning-models-keras-amazon-web-services/)
```
[global]
device = gpu
floatX = float32
allow_gc = False
 
[lib]
cnmem=.95
```

This will change the default parameters of calling Theano to autorun of the GPU.

###Check Installation

Open ipython and import theano to check. You should see something like:
"Using gpu device 0: GRID K520 (CNMeM is enabled with initial size: 95.0% of memory, cuDNN not available)"

You can now call scripts like normal, and it will default to the GPU if using Theano.
