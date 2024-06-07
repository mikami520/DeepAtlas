<h2 align="center"> [OTO–HNS2024] A Label-Efficient Framework for Automated Sinonasal CT Segmentation in Image-Guided Surgery </h2>
<p align="center">
<a href="https://aao-hnsfjournals.onlinelibrary.wiley.com/doi/10.1002/ohn.868"><img src="https://img.shields.io/badge/Wiley-Paper-red"></a>
<a href="https://pubmed.ncbi.nlm.nih.gov/38686594/"><img src="https://img.shields.io/badge/PubMed-Link-blue"></a>
<a href="https://github.com/mikami520/AutoSeg4SinonasalCT"><img src="https://img.shields.io/badge/Code-Page-magenta"></a>
</p>
<h5 align="center"><em>Manish Sahu<sup>*</sup>, Yuliang Xiao<sup>*</sup>, Russell Taylor, Mathias Unberath, Bryan K. Ward, Deepa Galaiya, Masaru Ishii, Francis X. Creighton</em></h5>
<p align="center"> * Indicates Equal Contribution </p>
<p align="center">
  <a href="#news">News</a> |
  <a href="#abstract">Abstract</a> |
  <a href="#installation">Installation</a> |
  <a href="#preprocess">Preprocess</a> |
  <a href="#train">Train</a> |
  <a href="#inference">Inference</a>
</p>

## News 

**2024.06.03** - Our paper is accepted to **American Academy of Otolaryngology–Head and Neck Surgery 2024 (OTO-HNS2024)**.

**2024.06.07** - The data preprocessing, training, inference, and evaluation code are released.

## Installation
### Step 0: Fork This GitHub Repository 
```bash
git clone https://github.com/mikami520/AutoSeg4SinonasalCT.git
```

### Step 1: Set Up Two Environments Using requirements.txt Files (virtual environment is recommended)
```bash
pip install -r requirements.txt
```
## Preprocess
### Step 2: Preprocess Datasets
### Step 2.1: Co-align the data (make sure scan and segmentation are co-aligned)
Activate scripting environment
```bash
cd <path to repo>/deepatlas/preprocess
```
Co-align the scans and segmentations (recommendation: Similarity registration)
```bash
python3 registration_training.py 
-bp <full path of base dir> 
-ip <relative path to nifti images dir> 
-sp <relative path to segmentations dir>
-ti <task id> 
```
If you want to make sure correspondence of the name and value of segmentations, you can add the following commands after above command (**Option for nrrd format**)
```bash
-sl LabelValue1 LabelName1 LabelValue2 LabelName2 LabelValue3 LabelName3 ...
```
For example, if I have two labels for maxillary sinus named ```L-MS``` and ```R-MS``` and I want ```L-MS``` matched to ```label 1``` and ```R-MS``` to ```label 2``` (**Pay attention to the order**)
```bash
python3 registration_training.py -bp /Users/mikamixiao/Desktop -ip images -sp labels -sl 1 L-MS 2 R-MS
```
Final output of registered images and segmentations will be saved in 
```text
base_dir/deepatlas_raw_data_base/task_id/Training_dataset/images && base_dir/deepatlas_raw_data_base/task_id/Training_dataset/labels
```
### Step 2.2: Crop Normalize and Flip Data (if needed)
Crop，normalize and flip data to extract region of interest (ROI). **Notice: the images and segmentations should be co-registered. We recommend to use the outputs of Step 2.1**
```bash
python3 crop_flip_training.py 
-fp <if need to flip data, use flag for true and not use for false> 
-ti <task id> 
-rs <customized resized shape>
``` 
**Pay attention to the resized dimension which should not be smaller than cropped dimension**\
Final output of ROI will be saved in
```text
base_dir/deepatlas_preprocessed/task_id/Training_dataset/images && base_dir/deepatlas_preprocessed/task_id/Training_dataset/labels
```

## Train
Activate scripting environment
```bash
cd <path to repo>/deepatlas/scripts
```
Start training, **Note: please use the outputs of Step 2.2**
```bash
python3 deep_atlas_train.py
--config <configuration file of network parameters>
--continue_training <check if need to resume training>
--train_only <only training or training plus test>
--plot_network <whether to plot the network>
```
**For detailed information, use ```-h``` to see more instructions**
Before training, a folder named ```deepatlas_results``` is created automatically under the repository directory. All training results are stored in this folder. A clear structure is shown below:
```text
DeepAtlas/deepatlas_results/
    ├── Task001_ET
    |   └── results
    |       └── RegNet
    |           |── anatomy_loss_reg.txt
    |           |── anatomy_reg_losses.png
    |           |── reg_net_best.pth
    |           |── reg_net_training_losses.png
    |           |── regularization_loss.txt
    |           |── regularization_reg_losses.png
    |           |── similarity_loss_reg.txt
    |           |── similarity_reg_losses.png
    |       └── SegNet
    |           |── anatomy_loss_seg.txt
    |           |── anatomy_seg_losses.png
    |           |── seg_net_best.pth
    |           |── seg_net_training_losses.png
    |           |── supervised_loss_seg.txt
    |           |── supervised_seg_losses.png
    |       └── training_log.txt
    |   └── dataset.json
    ├── Task002_Nasal_Cavity
```
## Inference
```
python3 deep_atlas_test.py
-gpu <id of gpu device to use>
-op <relative path of the prediction result directory>
-ti <task id and name>
```
The final prediction results will be saved in the ```DeepAtlas_dataset/Task_id_and_Name``` directory. For example,
```text
DeepAtlas/DeepAtlas_dataset/
    ├── Task001_ET
    |   └── results
    |       └── RegNet
    |           |── anatomy_loss_reg.txt
    |           |── anatomy_reg_losses.png
    |           |── reg_net_best.pth
    |           |── reg_net_training_losses.png
    |           |── regularization_loss.txt
    |           |── regularization_reg_losses.png
    |           |── similarity_loss_reg.txt
    |           |── similarity_reg_losses.png
    |       └── SegNet
    |           |── anatomy_loss_seg.txt
    |           |── anatomy_seg_losses.png
    |           |── seg_net_best.pth
    |           |── seg_net_training_losses.png
    |           |── supervised_loss_seg.txt
    |           |── supervised_seg_losses.png
    |       └── training_log.txt
    |   └── prediction
    |       └── RegNet
    |           |── reg_img_losses.txt
    |           |── reg_seg_dsc.txt
    |           |── figures containing fixed, moving, warped scans deformation field and jacobian determinant
    |           |── warped scans and labels in nifti format
    |       └── SegNet
    |           |── seg_dsc.txt
    |           |── predicted labels in nifti format
    |   └── dataset.json
    ├── Task002_Nasal_Cavity
```

