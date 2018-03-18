# My Goals
  - Setup CUDA and Tensorflow so that I can train some models with GPU accleration
  - Should be able to switch to Nouveau since I want the reverse PRIME to work so that I can use my external monitor through HDMI. (Though most of the time I don't need CUDA. So switch by rebooting is fine)
# Environment Description
  - Hardware:
      - CPU: Xeon E3-1505M v6 @ 3.00GHz
      - GPU: Quadro M2200
      - Screen: FHD w/o touch
  - Software:
      - Distro: Arch Linux
      - Kernel: 4.15.9-1-ARCH
      - X version: 1.19.6
      - KDE Plasma 5.12.3
      - Qt: 64-bit, 5.10.1
      - With Nouveau installed
# Process
  - Install Nvidia Driver:
      - Download the run file from [here](http://www.nvidia.com/download/driverResults.aspx/131853/en-us). (You may be able to find a newer version though)
      - run with `sudo ./NVIDIA-Linux-x86_64-390.42.run --no-opengl-files`.
      - When asked to generate blacklist files, choose no.
      - We can't use the nvidia driver from the repository, since that will install the nvidia OpenGL files which will mess up Nouveau.
  - Install CUDA Toolkit and CUDA driver:
      - Download the run file from [here](https://developer.nvidia.com/cuda-downloads?target_os=Linux&target_arch=x86_64&target_distro=Fedora&target_version=25&target_type=runfilelocal). Note that there is no official support for Arch. But the run file from other distros will work.
      - First execute: `sudo ./cuda_9.1.85_387.26_linux.run --no-opengl-libs`. 
        - When prompted with question about whether to install NVIDIA Accelerated Graphics Driver, choose no. This will install everything except CUDA driver and the OpenGL files.
        - Choose y for `Install the CUDA 9.1 Toolkit?`
        - Use default location for Toolkit Location.
        - Choose y for `Do you want to install a symbolic link at /usr/local/cuda?`
        - Answer to other questions according to your needs.
        - After done, follow the printout instructions to add PATH and LD_LIBRARY_PATH values.
      - Then execute: `sudo ./cuda_9.1.85_387.26_linux.run --silent --driver` to install the driver.
  - Install cuDNN
      - Download from [here](https://developer.nvidia.com/cudnn).
      - Follow the instructions about installing from a tar file from [here](http://docs.nvidia.com/deeplearning/sdk/cudnn-install/index.html#installlinux). Make sure the location of cuda is correct. (You should have symlinks to CUDA in `/usr/local/cuda/`).
  - Install Tensorflow
      - Follow the [instructions](https://www.tensorflow.org/install/install_sources) from Tensorflow to build from source.
        - The CUDA version: The cuda version you have downloaded. Mine is 9.1.
        - The cuDNN version: The number in `/usr/local/cuda/lib64/libcudnn.so.*`. Mine is 7.
        - The compute capability is 5.2.
      - **NOTE**:If the glibc version is 2.26 (check with `ldd --version`), then you need this "dirty hack" before compiling:
        ```
        sed -i "1 i#define _BITS_FLOATN_H" "/usr/local/cuda/include/host_defines.h"
        ```
        according to [here](https://git.archlinux.org/svntogit/community.git/commit/trunk?h=packages/cuda&id=ae90e4d243510e9565e66e9e8e08c509f5719fe0)
      
# Switch the drivers
  - Create a new directory and create three files in it: `disable-nouveau.conf`, `disable-nvidia.conf`, `toggle-gpu.sh`.
  - Copy the following into each file:
      - Copy these into disable-nouveau.conf.
        ``` 
        blacklist nouveau
        options nouveau modeset=0
        ```
      - Copy this into disable-nvidia.conf
        ```
        blacklist nvidia
        ```
      - Copy this into toggle-gpu.sh, don't forget to change the `<dir_path>` to the actual path.
        ```bash
        #!/usr/bin/bash
        function to_nvidia {
            sudo rm /etc/modprobe.d/disable-nouveau.conf -f
            sudo rm /etc/modprobe.d/disable-nvidia.conf -f
            sudo cp <dir_path>/disable-nouveau.conf /etc/modprobe.d/
            sudo rm /usr/lib/modprobe.d/disable-nouveau.conf -f
            sudo rm /usr/lib/modprobe.d/disable-nvidia.conf -f
            sudo cp <dir_path>/disable-nouveau.conf /usr/lib/modprobe.d/
        }
                                                                                                                                                                                                                     
        function to_nouveau {                                                                                                                                                                                                
            sudo rm /etc/modprobe.d/disable-nouveau.conf -f                                                                                                                                                                  
            sudo rm /etc/modprobe.d/disable-nvidia.conf -f                                                                                                                                                                   
            sudo cp <dir_path>/disable-nvidia.conf /etc/modprobe.d/                                                                                                                                
            sudo rm /usr/lib/modprobe.d/disable-nouveau.conf -f                                                                                                                                                              
            sudo rm /usr/lib/modprobe.d/disable-nvidia.conf -f                                                                                                                                                               
            sudo cp <dir_path>/disable-nvidia.conf /usr/lib/modprobe.d/                                                                                                                            
        }  
        ```
   - Lastly add this to your .bashrc.
     ```
     source <dir_path>/toggle-gpu.sh
     ```
   - To change the kernel module, either type `to_nvidia` or `to_nouveau` in the terminal and then reboot.
   - When using the Nvidia module, the graphic is handled by the intel GPU and Nvidia module is only used for CUDA.
