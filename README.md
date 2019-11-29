# Lesion attributes segmentation for melanoma detection with multi-task U-Net 

Melanoma is the most deadly form of skin cancer worldwide. Many efforts have been made for early detection of melanoma. The International Skin Imaging Collaboration (ISIC) hosted the 2018 Challenges to help the diagnosis of melanoma based on dermoscopic images. We describe our solutions for the task 2 of ISIC 2018 Challenges. We present a multi-task U-Net model to automatically detect lesion attributes of melanoma. The network includes two tasks, one is the classification task to classify if the lesion attributes present, and the other is the seg-mentation task to segment the attributes in the images. Our multi-task U-Net model achieves a Jaccard index of 0.433 on official test data of ISIC 2018 Challenges task 2, which is the best single-model result and ranks the 5th place on the final leaderboard.


## Introduction

Deep learning models have also been applied toclassify skin diseases and segment whole lesion regions. However, lesion attribution segmentation for melanomadetection has not been well explored. Automatic detection of those skin lesion attributes can be a tremendous help forearly melanoma diagnosis.  Towards this goal, the Interna-tional Skin Imaging Collaboration (ISIC) hosts a competitionto predict the locations of five skin lesion patterns (i.e., dermoscopic attributes) in dermoscopic images. These five pat-terns are pigment network, negative network, streaks, milia-like cysts, and globules.  These morphological pat-terns have been associated with clinical criteria for detectingskin disease. Below are examples of dermoscopic skin images and corresponding lesion attribute topology (figures from [https://challenge2018.isic-archive.com/](https://challenge2018.isic-archive.com/))

<p align="center">
<img src="img/intro.png" width="600" align="center">
</p>


We noticed that not all attributes present in each skin image. Below is a summary of non-empty masks in the training data. N is the number of non-empty masks. 58.7%, 26.3% and 23.2% of the skin images have non-empty masks for pigment network, milia-like cysts, and globules, respectively. While only 7.3% 2.9% of the skin images have non-empty masks for negative network and streaks, respectively. Since segmentation is essentially a pixel-wise classification problem, a large numberof empty masks increase the number of samples in negative class.

<p align="center">
<img src="img/percentage.png" width="300" align="center">
</p>


Examples of skin images and corresponding lesion attributes. For each attribute, the top row is the ground truth masks and the bottom row is the predicted masks from multi-task U-Net model. Note that many of the ground truth masks are empty, indicating no corresponding lesion attributes in the skin images.

<p align="center">
<img src="img/skin.png" width="400" align="center">
</p>

We develop a multi-task U-Net model to automatically detect lesion attributes of melanoma.  In the U-Net model, we replace the encoder part of the U-Net with a pretrained VGG16 network.  We further extend the model based on multi-task learning.  In the middle layer and the last layer of the U-Net, we add two classification heads to classify the empty masks versus the non-empty masks.  One classification head is added to the middle layer with a 1×1 convolutional layer followed by a global average pooling layer and the other is added to the last layer with a global max pooling layer. The experiment results show that the extra ancillary classification task indeed improves the performance of the segmentation task.  Without any model ensemble, our multi-task U-Net model achieves a Jaccard index of 0.433 on official test data, which is the best single-model result. 

<p align="center">
<img src="img/vgg.png" width="700" align="center">
</p>

The loss is a combination of pixel-wise segmentation loss loss1, classification loss using the middle layer loss2 and classification loss using the last layer loss3.

loss = loss1+ 0.5×loss2 + 0.5×loss3

## How to run the pre-trained model on the ISIC2018 test data

### Step 1: Set up environment

#### If you want to use conda 

#### 1. Create a Python environment
```
conda create -n isic2018 python=3
source activate isic2018
```

#### 2. Install necessary packages
```
pip install -r docker/requirements.txt
```
Those are the packages installed in my environment and many packages are not necessarily needed to run the pretrained model. But for ease of use, I just dumped all the installed packages into one file.

#### 3. Clone this repo
```
git clone https://github.com/chvlyl/ISIC2018.git
cd ISIC2018
```

#### If you want to use docker

#### 1. Install docker
```
## make sure you have the latest nvidia driver

## for Ubuntu
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu  $(lsb_release -cs)  stable" 
sudo apt-get update
sudo apt-get install docker-ce

## start docker
sudo systemctl start docker
sudo systemctl enable docker

## check if docker is running
sudo systemctl status docker

## check docker version
docker --version

## add user to docker group 
## otherwise you may get the following error
## Got permission denied while trying to connect to the Docker daemon socket
sudo usermod -aG docker $USER
## may need to restart the system

# Add the package repositories
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list

sudo apt-get update && sudo apt-get install -y nvidia-container-toolkit nvidia-container-runtime
sudo systemctl restart docker
```

#### 2. Build docker image

```
## Clone this repo
git clone https://github.com/chvlyl/ISIC2018.git
cd ISIC2018

## build docker image
cd docker
docker  build -t isic2018 .

## create a docker container and mount the git folder
cd ..
docker run --gpus all --rm -v $(pwd):/home/ISIC2018/ --name isic2018 --ipc=host -it isic2018  bash
```

### Step 2: Download data and model weights

#### 4. Download the ISIC2018 test images
All the ISIC2018 test images are in jpg format. Save those images into a folder.

#### 5. Download the pretrained model weights
The trained model weights can be downloaded [here](https://drive.google.com/drive/folders/1oxA7AXwnIug2H91r_49qthekz6UP47rc?usp=sharing)


### Step 3: Run the model

#### 6. Run the pretrained model on test data
```
python3 submission.py --image-path test_image_path --model-weight model.pt
```
By default, the predicted masks will be saved in the prediction folder

## Some notes
1. I trained the model with multi-GPUs. If you run my code on a single GPU, you may get an error about the parameter name mismatch. I think this is a bug in Pytorch and currently I don't have a good solution rather than manually modifying the parameter names (remove the 'module' prefix)

2. When I developed the model, I tried many different things. I commented out some code and kept them just in case you may be interested in trying them out. 

3. The pickle file I used for train test split.

The data was not split based on task2. I was working with some friends and the original plan was to use the results from other tasks (1 and 3) to help our own task 2. So the data was split based on task1 (or task3) so that we had the same validation and test dataset. I could use the results from task1 or task3 to help my task. I would suggest you split the data by yourself according to this format in the pickle file. 

## Further improvement
1. I entered in this competition relatively late and I only had one month to work on it in part-time. Therefore, I believe many things can still be improved. Feel free to copy my code and work on it.

2. I probably still need to clean my code when I have time. 


## Reference    
1. For more details, please check our paper: Eric Z. Chen, Xu Dong, Junyan Wu, Hongda Jiang, Xiaoxiao Li, Ruichen Rong. Lesion Attributes Segmentation for Melanoma Detection with Deep Learning. ISBI 2019 [https://ieeexplore.ieee.org/document/8759483] bioRxiv 2018 [https://doi.org/10.1101/381855]
2. My poster presented at ISBI 2019 ([ISBI_EZC.pdf](ISBI_EZC.pdf))
3. Some of the code was adopted from this [repo](https://github.com/ternaus/robot-surgery-segmentation)

