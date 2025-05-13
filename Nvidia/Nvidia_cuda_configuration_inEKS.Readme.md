Step-1: install Nvidia Plugin in Bastion server

Step-2: Install Gpu operator in Bastion server

  **1. Add NVIDIA Helm Repository**

                  helm repo add nvidia https://nvidia.github.io/gpu-operator
        
                  helm repo update

  **2.namespace created**

                 kubectl create namespace gpu-operator-resources

  **3.installing gpu-operator**

                 helm install nvidia-gpu-operator nvidia/gpu-operator --namespace gpu-operator-resources

  **4.verify gpu availability**

                 kubectl describe node <your-node-name> | grep nvidia

Step-3: Go inside the pod and run the below command and verify nvidia is installed

           kubectl exec -it <podname> /bin/bash
           nvidia-smi

Step-4: For cuda toolkit in dockerfile below base image and commands should include and build the dockerfile

# base image
FROM nvidia/cuda:12.2.0-devel-ubuntu22.04
# Install pip & Python dependencies
RUN apt-get update && apt-get install -y python3-pip
# Install PyTorch with CUDA
RUN pip3 install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu121

# for different base image install below commands in dockerfile
sudo apt update
sudo apt install -y software-properties-common
sudo add-apt-repository ppa:graphics-drivers/ppa
sudo apt-get install -y nvidia-cuda-toolkit

Step-5: After build the image in manifest update below lines




