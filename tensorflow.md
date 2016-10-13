## Installing GPU enabled tensorflow
This is a concentrated guide for installing GPU enabled Tensorflow. This quick guide uses the google binaries instead of building your own source for simplicity. If you want to use a custom combination of CUDA/cuDNN you'll have to follow their instructions to compile from sources.

### Launch instance
Follow the standard procedure to launch a g2/p2 instance with ubuntu.

### Install conda
Go to the conda website. <br>
https://www.continuum.io/downloads <br>

Go to linux and get the download url for 64 bit version of python 2/3. <br>

Download the shell script. Then install it using bash. <br>
Make sure to say yes when it asks if you want to add it to your bash profile. <br>

```
wget https://repo.continuum.io/archive/Anaconda2-4.2.0-Linux-x86_64.sh
bash Anaconda2-4.2.0-Linux-x86_64.sh
```
Source your bash profile so that it uses your conda install.
```
source .bashrc
```

### Determine the CUDA/CuDNN version
Next, go to the tensorflow install page. Look for the section under pip install and figure out what versions of CUDA/cuDNN are currently supported in the binaries. Look for ubuntu + GPU support and look for which versions it needs. <br>

This is the example at the time of writing, notice it needs CUDA 7.5 and cuDNN v5.
```
# Ubuntu/Linux 64-bit, GPU enabled, Python 2.7
# Requires CUDA toolkit 7.5 and CuDNN v5. For other versions, see "Install from sources" below.
$ export TF_BINARY_URL=https://storage.googleapis.com/tensorflow/linux/gpu/tensorflow-0.11.0rc0-cp27-none-linux_x86_64.whl
```
### Download/install the CUDA
Next, find the appropriate CUDA toolkit from nvidia's website. You might have to look in the archives. At the time of writing, CUDA 8 is the one featured, so to find 7.5 we had to look in the cuda archives.

Once you find the site, you click the following according to the OS, which for our ubuntu aws instace is:<br>
* Linux
* x86_64
* Ubuntu
* 14.04

Get the url for the deb (local) button. <br>
Now we use wget similar as above to download the file on our ubuntu machine. Then enter the code as directed on the nvidia site. This block downloads the code, unpackages it, and uses ubuntu's apt-get to install the cuda. <br>

```
wget http://developer.download.nvidia.com/compute/cuda/7.5/Prod/local_installers/cuda-repo-ubuntu1404-7-5-local_7.5-18_amd64.deb
sudo dpkg -i cuda-repo-ubuntu1404-7-5-local_7.5-18_amd64.deb
sudo apt-get update
sudo apt-get install cuda
```

Next, we have a few more cleanups to make sure our isntance knows where our cuda is located. <br>
Open up .bashrc with your favorite text editor and add the two lines to the bottom.
```
export LD_LIBRARY_PATH="$LD_LIBRARY_PATH:/usr/local/cuda/lib64:/usr/local/cuda/extras/CUPTI/lib64"
export CUDA_HOME=/usr/local/cuda
```
Then source the bash file again like above.
```
source .bashrc
```
Finally, there's a line that for some reason makes the kernel recognize the cuda.
```
sudo apt-get install -y linux-image-extra-`uname -r` linux-headers-`uname -r` linux-image-`uname -r`
```

Now, if you type `nvidia-smi` in the terminal, it should display your graphics card.

### Install the cuDNN
Next google nvidia cuDNN. It should take you the appropriate webpage. You'll have to create an account to be able to access it. <br>
Once you're at the downloads page, select the version of the cuDNN that corresponds to the info above. In our case, v5 and cuda 7.5. <br>
I don't think you can wget this file, so you'll have to download it locally, then `scp` it up to your instance.<br>
Once you've done that, you copy the following lines from the tensorflow site. <br>
Change the tar line to match the filename of your cuDNN.
```
tar xvzf cudnn-7.5-linux-x64-v5.1-ga.tgz
sudo cp cuda/include/cudnn.h /usr/local/cuda/include
sudo cp cuda/lib64/libcudnn* /usr/local/cuda/lib64
sudo chmod a+r /usr/local/cuda/include/cudnn.h /usr/local/cuda/lib64/libcudnn*
```

### Install Tensorflow from the predone wheel
Now that you have all the gpu stuff installed, the pip install is ready to be run. <br>
Use the version you found above, then pip install. <br>

```
export TF_BINARY_URL=https://storage.googleapis.com/tensorflow/linux/gpu/tensorflow-0.11.0rc0-cp27-none-linux_x86_64.whl
sudo pip install --ignore-installed --upgrade $TF_BINARY_URL
```

### Test the install
To test your install, run the following in `ipython`

```python
import tensorflow as tf

# Creates a graph.
a = tf.constant([1.0, 2.0, 3.0, 4.0, 5.0, 6.0], shape=[2, 3], name='a')
b = tf.constant([1.0, 2.0, 3.0, 4.0, 5.0, 6.0], shape=[3, 2], name='b')
c = tf.matmul(a, b)
# Creates a session with log_device_placement set to True.
sess = tf.Session(config=tf.ConfigProto(log_device_placement=True))
# Runs the op.
print sess.run(c)
```

If your instance is configured correctly, you should see some gibberish about loading cuda libraries after the import, then a bunch of log information specifying that it's using the GPU after you do the sess.run.
