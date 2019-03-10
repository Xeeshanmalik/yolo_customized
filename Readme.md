# How to train YOLOv2 to detect custom objects
## Installing Darknet

````
git clone https://github.com/AlexeyAB/darknet.git
cd darknet
````

(Not you want GPU please follow the steps. Important GPU is needed to training otherwise not needed)

````
vi Makefile
````

(Change the GPU 0 to 1 and save it. If you installed openCV set OPENCV 0 to 1 otherwise not needed)

````
make
````

If you have any errors, try to fix them? if everything seems to have compiled correctly, try running it!

````
./darknet
````

You should get the output:

````
usage: ./darknet <function>
````

## Yolo Testing

You already have the config file for YOLO in the cfg/ subdirectory. You will
have to download the pre-trained weight file
<a href="https://pjreddie.com/media/files/yolo.weights">here (258 MB)</a> or just run this:

````
wget https://pjreddie.com/media/files/yolo.weights
````

Then run the detector!

````
./darknet detect cfg/yolo.cfg yolo.weights data/dog.jpg
```` 

You will see some output like this:

````
layer     filters    size              input                output
    0 conv     32  3 x 3 / 1   416 x 416 x   3   ->   416 x 416 x  32
    1 max          2 x 2 / 2   416 x 416 x  32   ->   208 x 208 x  32
    .......
   29 conv    425  1 x 1 / 1    13 x  13 x1024   ->    13 x  13 x 425
   30 detection
Loading weights from yolo.weights...Done!
data/dog.jpg: Predicted in 0.016287 seconds.
car: 54%
bicycle: 51%
dog: 56%

````

## Training and Testing Data Preparation

Prepare train.txt and test.txt using below python file(process.py) and put your dataset path in line 5

````
import glob, os
# Current directory
current_dir = os.path.dirname(os.path.abspath(__file__))
print(current_dir)
current_dir = '<Your Dataset Path>'
# Percentage of images to be used for the test set
percentage_test = 10;
# Create and/or truncate train.txt and test.txt
file_train = open('train.txt', 'w')  
file_test = open('test.txt', 'w')
# Populate train.txt and test.txt
counter = 1  
index_test = round(100 / percentage_test)  
for pathAndFilename in glob.iglob(os.path.join(current_dir, "*.jpg")):  
    title, ext = os.path.splitext(os.path.basename(pathAndFilename))
if counter == index_test:
        counter = 1
        file_test.write(current_dir + "/" + title + '.jpg' + "\n")
    else:
        file_train.write(current_dir + "/" + title + '.jpg' + "\n")
        counter = counter + 1
````

## Preparing YOLOv2 configuration files

YOLOv2 needs certain specific files to know how and what to train. We’ll be creating these three files. I am using 1GB GPU. So i am used tiny-yolo.cfg:

* cfg/obj.data
* cfg/obj.names
* cfg/tiny-yolo.cfg

First let’s prepare the YOLOv2 .data and .names file. Let’s start by creating obj.data and filling it with this content. This basically says that we are training one class, what the train and validation set files are and what file contains the names for the categories we want to detect.

````
classes= 1  
train  = train.txt  
valid  = test.txt  
names = cfg/obj.names  
backup = backup/
````

The obj.names looks like this, plain and simple. Every new category should be on a new line, its line number should match the category number in the .txt label files we created earlier.

````
NFPA
````

A final file we have to prepare (I know, powerful GPU eagerly waiting to start crunching!), is the .cfg file. I just duplicated the tiny-yolo.cfg file, and made the following edits:

* Line 2: set batch=24, this means we will be using 64 images for every training step
* Line 3: set subdivisions=8, the batch will be divided by 8 to decrease GPU VRAM requirements. If you have a powerful GPU with loads of VRAM, this number can be decreased, or batch could be increased. The training step will throw a CUDA out of memory error so you can adjust accordingly.
* Line 120: set classes=1, the number of categories we want to detect
* Line 114: set filters=(classes + 5)*5 in our case filters=30

To start training, YOLOv2 requires a set of convolutional weights. To make things a little easier, Joseph offers a set that was pre-trained on Imagenet. 
This conv.23 file can be <a href="https://pjreddie.com/media/files/darknet19_448.conv.24">downloaded</a> from the official YOLOv2 website and provides
an excellent starting point. We'll need this file for the next step.

## Training

Time for the fun part! Enter the following command into your terminal and watch your GPU do what it does best (copy your train.txt and test.txt to yolo_darknet folder):

````
./darknet detector train cfg/obj.data cfg/yolo-obj.cfg darknet19_448.conv.23
````

Note: When completed 100 iteration it will automatically store weights file and kill the process once the average loss is less than 0.06 to get good a accuracy.

## Results

We should now have a .weights file that represents our trained model. Let’s use this on some images to see how well it can detect the NFPA 704 ‘fire diamond’ pictogram.

''''
./darknet detector test cfg/obj.data cfg/yolo-obj.cfg yolo-obj1000.weights data/image.jpg
''''
