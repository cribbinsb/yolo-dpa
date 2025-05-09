# yolo-dpa : Yolo11 + Simultaneous Detection, Pose, Attributes
Contents: [Introduction](#introduction) | [Test Results](#test-results) | [Usage](#usage) | [License](#license)

---

### Introduction

<table>
  <tr>
    <td><img src="https://github.com/user-attachments/assets/a63b2f93-aeda-402e-9f2a-a619800d9730" width="400"></td>
    <td><img src="https://github.com/user-attachments/assets/a8b9072b-ea04-417e-9cda-548684f24d7b" width="400"></td>
  </tr>
  <tr>
    <td><img src="https://github.com/user-attachments/assets/fcd2a18e-b4aa-4beb-a777-a554455d57c8" width="400"></td>
    <td><img src="https://github.com/user-attachments/assets/2e7fe043-9ef5-4db2-a041-ef9272e598dc" width="400"></td>
  </tr>
</table>

<b>Yolo-DPA</b> is a set of proof of concept models derived from Ultralytics yolo11 to investigate the performance of combining <b>object detection</b>, <b>pose/keypoint detection</b> and <b>binary attribute detection</b> together into a single model, done in one pass. Remarkably, adding all the extra capabilities does not seem to make the model too much worse at the basic object detection versus the original object detection only model, with only a  tiny increase in parameter/flop count.

It was also intended to test the [dataset processing pipeline](https://github.com/cribbinsb/dataset_processor) which attempts things like automatic labelling, combining multiple datasets together, and using vision-LLMs

These models and technologies are intended as a proof of concept only. Please check out the [license](#licence) and also be mindful of the licenses of the datasets on which these models were trained.

#### Developed/trained using
- [Ultralytics Yolo11 (slightly) modified for multi-label detection support](https://github.com/cribbinsb/ultralytics/tree/multilabel)
- [DatasetProcessor](https://github.com/cribbinsb/dataset_processor)
- [AzureML](https://github.com/cribbinsb/azureml)

#### Test models (weights provided) has
- <b>Detection</b> of 5 basic classes *Person, Face, Vehicle, Animal, Weapon*
- <b>Pose</b> Face boxes have 5 face points *2 x eyes, nose, 2 x mouth*
- <b>Pose</b> Person boxes have 17 pose points *same as coco-pose*
- <b>Attributes</b> Person boxes have 35 binary attributes detected *male, female, wearing hat, wearing glasses,...*
- <b>Dataset</b> Trained on an ensemble dataset of around 350K images from Coco,Openimages,Objects365 & others, re-labelled using DatasetProcessor, with GPT-4o-V for attribute labelling. The dataset/DatasetProcessor config file is not provided, please contact me if you are interested to get hold of it.
- <b>Weights</b> can be downloaded from links in the [Test Results](#test-results) section

---

### Test results

Using map.py from https://github.com/cribbinsb/dataset_processor

These results are the geometric mean of results run on from 5 "val" datasets, three of which are the val sets of Coco, OpenImages, and Objects365

<small>
  
| Model | size<br><sup>(pixels)  | params<br><sup>(M) | mAP50 Person | mAP50 Face  | mAP50 Vehicle | mAP50 Pose | mAP50 Face KP | mAP50 Attr <br>(Main) | mAP50 Attr <br>(colour) |
| ---------- | --------- | ----------- | --------------- | ----------- | --------- | --------- |--------- |-------- | ------------ |
|[Yolo-dpa-l](https://drive.google.com/file/d/1DwRpgS53MtQYM4G7Rm1K7OBxHhguaiI5/view?usp=drive_link)   | 640 | 26.2 | 0.874 | 0.691 | 0.778 | 0.828 | 0.707 | 0.603 | 0.556 |
|[Yolo-dp-l](https://drive.google.com/file/d/1veVJ9y6Set3oIDtZ47_Zpz6cnYqyMauy/view?usp=drive_link)    | 640 | 26.2 | 0.883 | 0.740 | 0.732 | 0.822 | 0.733 |       |       | 
|yolo11l      | 640 | 25.3 | 0.850 |       | 0.813 |       |       |       |       |
|yolo11l-pose | 640 | 26.2 | 0.718 |       |       | 0.845 |       |       |       |
|[yolo-dpa-s](https://drive.google.com/file/d/1FUK6x26Z8Dz0gqw-20IHrvnUIKl8lLhk/view?usp=drive_link)   | 640 | 10.1 | 0.845 | 0.675 | 0.662 | 0.788 | 0.710 | 0.593 | 0.522 |
|yolo11-s     | 640 | 9.4  | 0.820 |       | 0.653 |       |       |       |       | 
|[yolo-dpa-n](https://drive.google.com/file/d/1YDbFnwfd_xvlm4kkRiXCs_FMCPPOTfXP/view?usp=drive_link)   | 640 | 3.0  | 0.798 | 0.658 | 0.545 | 0.718 | 0.691 | 0.494 | 0.449 |
|yolo11-n     | 640 | 2.6  | 0.758 |       | 0.678 |       |       |       |       | 

</small>

---

### Usage


#### Test app 


- check out the repository: <b> git clone git@github.com:cribbinsb/yolo-dpa.git  </b>
- create conda environment: <b> conda env create -f environment.yml </b>
- activate conda environment: <b> conda activate yolo-dpa </b>
- run: <b> python yolo-dpa-test.py --video webcam --model yolo-dpa-l </b>
- "video" can we 'webcam' or path to an mp4 file; 'model' can be one of yolo-dpa-l, yolo-dp-l, yolo-dpa-s, yolo-dpa-n
- the weights should be automatically downloaded if not already present
- press "P" to pause the video; click on a person box to see the attributes
  
![Screenshot from 2025-02-02 10-47-36](https://github.com/user-attachments/assets/f967c14d-e6c1-4938-9045-542131423c6e)

### Inference / how the model differs from standard ultralyics Yolo11 model

The basic model is a standard ultralytics yolo11-pose model. However because the object detector is used to detect attributes some changes are needed for both train and inference. Thr problem is that by default yolo11 will predict only one class label per box, this need to be improved to allow multiple labels (i.e. the 'attributes' are just also detecting 'male' class for the same box as 'person') The code for this was based on [this fork of ultralytics](https://github.com/Danil328/ultralytics.git), modified to also work for pose models.

Changes are needed both to the loss function during training and also during inference. The inference changes are purely to the postprocessing/interpretation of the model results. By default each "detection" row will have a score for each class and ultralytics will pick the highest scoring class only, prior to NMS. Instead there are a couple of different approaches

- (A) "expand" the detection to multiple detections, one for each class where the score is greater than the threshold, then produce boxes as normal using NMS. This will result in separate output boxes for "person", "person_male", "person_with_beard" etc which can then be combined together in a post-processing step

- (B) Remove the "attribute classes" before NMS. Post NMS add an attribute vector to the detection where the corresponding score for each attribute is the "max" of all the boxes that were combined/removed in the NMS step for that detection.

In my opinion (B) is the better option. However note that the modified ultralytics / test app used here are doing (A).

---

### License

Weights provided under <b>AGPL</b>, and for <b> noncommercial use only</b>.

Additional restrictions may be imposed by licenses of the datasets on which these models were trained.

Please contact [me](mailto:bernandocribbenza@gmail.com?subject=yolo-dpa%20question&body=Your-code-is-rubbish!) if any questions.


  
