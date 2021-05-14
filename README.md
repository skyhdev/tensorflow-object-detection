#TensorFlow Object Detection

## Introduction

The repository also includes the Object_detection_picamera.py script, which is a Python script that loads an object detection model in TensorFlow and uses it to detect objects in a Picamera video feed. The guide was written for TensorFlow v1.8.0 on a Raspberry Pi Model 3B running Raspbian Stretch v9. It will likely work for newer versions of TensorFlow.

## Steps
### 1. Install TensorFlow

we’ll install TensorFlow. The download is rather large (over 100MB), so it may take a while. Issue the following command:

```
pip3 install tensorflow
```

TensorFlow also needs the LibAtlas package. Install it by issuing the following command. (If this command doesn't work, issue "sudo apt-get update" and then try again).
```
sudo apt-get install libatlas-base-dev
```
While we’re at it, let’s install other dependencies that will be used by the TensorFlow Object Detection API. These are listed on the [installation instructions](https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/installation.md) in TensorFlow’s Object Detection GitHub repository. Issue:
```
sudo pip3 install pillow lxml jupyter matplotlib cython
sudo apt-get install python-tk
```
Alright, that’s everything we need for TensorFlow! Next up: OpenCV.

### 2. Install OpenCV
TensorFlow’s object detection examples typically use matplotlib to display images, but I prefer to use OpenCV because it’s easier to work with and less error prone. The object detection scripts in this guide’s GitHub repository use OpenCV. So, we need to install OpenCV.

To get OpenCV working on the Raspberry Pi, there’s quite a few dependencies that need to be installed through apt-get. If any of the following commands don’t work, issue “sudo apt-get update” and then try again. Issue:
```
sudo apt-get install libjpeg-dev libtiff5-dev libjasper-dev libpng12-dev
sudo apt-get install libavcodec-dev libavformat-dev libswscale-dev libv4l-dev
sudo apt-get install libxvidcore-dev libx264-dev
sudo apt-get install qt4-dev-tools libatlas-base-dev
```
Now that we’ve got all those installed, we can install OpenCV. Issue:
```
sudo pip3 install opencv-python
```
Alright, now OpenCV is installed!

### 3. Compile and Install Protobuf
The TensorFlow object detection API uses Protobuf, a package that implements Google’s Protocol Buffer data format. You used to need to compile this from source, but now it's an easy install! I moved the old instructions for compiling and installing it from source to the appendix of this guide.

```sudo apt-get install protobuf-compiler```

Run `protoc --version` once that's done to verify it is installed. You should get a response of `libprotoc 3.6.1` or similar.

### 4. Set up TensorFlow Directory Structure and PYTHONPATH Variable
Now that we’ve installed all the packages, we need to set up the TensorFlow directory. Move back to your home directory, then make a directory called “tensorflow1”, and cd into it.
```
mkdir tensorflow1
cd tensorflow1
```
Download the tensorflow repository from GitHub by issuing:
```
git clone --depth 1 https://github.com/tensorflow/models.git
```
Next, we need to modify the PYTHONPATH environment variable to point at some directories inside the TensorFlow repository we just downloaded. We want PYTHONPATH to be set every time we open a terminal, so we have to modify the .bashrc file. Open it by issuing:
```
sudo nano ~/.bashrc
```
Move to the end of the file, and on the last line, add:
```
export PYTHONPATH=$PYTHONPATH:/home/pi/tensorflow1/models/research:/home/pi/tensorflow1/models/research/slim
```
Then, save and exit the file. This makes it so the “export PYTHONPATH” command is called every time you open a new terminal, so the PYTHONPATH variable will always be set appropriately. Close and then re-open the terminal.

Now, we need to use Protoc to compile the Protocol Buffer (.proto) files used by the Object Detection API. The .proto files are located in /research/object_detection/protos, but we need to execute the command from the /research directory. Issue:
```
cd /home/pi/tensorflow1/models/research
protoc object_detection/protos/*.proto --python_out=.
```
This command converts all the "name".proto files to "name_pb2".py files. Next, move into the object_detection directory:
```
cd /home/pi/tensorflow1/models/research/object_detection
```
Now, we’ll download the SSD_Lite model from the [TensorFlow detection model zoo](https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/detection_model_zoo.md). The model zoo is Google’s collection of pre-trained object detection models that have various levels of speed and accuracy. The Raspberry Pi has a weak processor, so we need to use a model that takes less processing power. Though the model will run faster, it comes at a tradeoff of having lower accuracy. For this tutorial, we’ll use SSDLite-MobileNet, which is the fastest model available. 

Google is continuously releasing models with improved speed and performance, so check back at the model zoo often to see if there are any better models.

Download the SSDLite-MobileNet model and unpack it by issuing:
```
wget http://download.tensorflow.org/models/object_detection/ssdlite_mobilenet_v2_coco_2018_05_09.tar.gz
tar -xzvf ssdlite_mobilenet_v2_coco_2018_05_09.tar.gz
```
Now the model is in the object_detection directory and ready to be used.

### 6. Detect Objects!
Okay, now everything is set up for performing object detection on the Pi! The Python script in this repository, Object_detection_picamera.py, detects objects in live feeds from a Picamera or USB webcam. Basically, the script sets paths to the model and label map, loads the model into memory, initializes the Picamera, and then begins performing object detection on each video frame from the Picamera. 

If you’re using a Picamera, make sure it is enabled in the Raspberry Pi configuration menu.

Download the Object_detection_picamera.py file into the object_detection directory by issuing:
```
wget https://raw.githubusercontent.com/EdjeElectronics/TensorFlow-Object-Detection-on-the-Raspberry-Pi/master/Object_detection_picamera.py
```
Run the script by issuing: 
```
python3 Object_detection_picamera.py 
```
The script defaults to using an attached Picamera. If you have a USB webcam instead, add --usbcam to the end of the command:
```
python3 Object_detection_picamera.py --usbcam
```
Once the script initializes (which can take up to 30 seconds), you will see a window showing a live view from your camera. Common objects inside the view will be identified and have a rectangle drawn around them. 

<p align="center">
  <img src="doc/kitchen.png">
</p>

With the SSDLite model, the Raspberry Pi 3 performs fairly well, achieving a frame rate higher than 1FPS. This is fast enough for most real-time object detection applications.

You can also use a model you trained yourself [(here's a guide that shows you how to train your own model)](https://www.youtube.com/watch?v=Rgpfk6eYxJA) by adding the frozen inference graph into the object_detection directory and changing the model path in the script. You can test this out using my playing card detector model (transferred from ssd_mobilenet_v2 model and trained on TensorFlow v1.5) located at [this dropbox link](https://www.dropbox.com/s/27avwicywbq68tx/card_model.zip?dl=0). Once you’ve downloaded and extracted the model, or if you have your own model, place the model folder into the object_detection directory. Place the label_map.pbtxt file into the object_detection/data directory.

<p align="center">
  <img src="doc/directory.png">
</p>

Now, when you run the script, it will use your model rather than the SSDLite_MobileNet model. If you’re using my model, it will detect and identify any playing cards dealt in front of the camera.

**Note: If you plan to run this on the Pi for extended periods of time (greater than 5 minutes), make sure to have a heatsink installed on the Pi's main CPU! All the processing causes the CPU to run hot. Without a heatsink, it will shut down due to high temperature.**

<p align="center">
  <img src="doc/cards.png">
</p>

Thanks for following through this guide, I hope you found it useful. Good luck with your object detection applications on the Raspberry Pi!

## Bonus: Pet Detector!

<p align="center">
  <img src="doc/pet_detector_demo.png">
</p>

### Description
The Pet_detector.py script is an example application of using object detection on the API to alert users when a certain object is detected. I have two indoor-outdoor pets at my parents' home: a cat and a dog. They frequently stand at the door and wait patiently to be let inside or outside. This pet detector uses the TensorFlow MobileNet-SSD model to detect when they are near the door. It defines two regions in the image, an "inside" region and an "outside" region. If the pet is detected in either region for at least 10 consecutive frames, the script uses Twilio to send my phone a text message.

### Usage
Run the pet detector by downloading Pet_detector.py to your /object_detection directory and issuing:
```
python3 Pet_detector.py
```

Using the Pet_detector.py program requires having a Twilio account set up [(see tutorial here)](https://www.twilio.com/docs/sms/quickstart/python). It also uses four environment variables that have to be set before running the program: TWILIO_ACCOUNT_SID, TWILIO_AUTH_TOKEN, MY_DIGITS, and TWILIO_DIGITS. These can be set using the "export" command, as shown below. More information on setting environment variables for Twilio is given [here](https://www.twilio.com/blog/2017/01/how-to-set-environment-variables.html).
```
export TWILIO_ACCOUNT_SID=[sid_value]
export TWILIO_AUTH_TOKEN=[auth_token]
export MY_DIGITS=[your cell phone number]
export TWILIO_DIGITS=[phone number of the Twilio account]
```
The sid_value, auth_token, and phone number of the Twilio account values are all provided when a Twilio account is set up.

If you don't want to bother with setting up Twilio so the pet detector can send you texts, you can just comment out the lines in the code that use the Twilio library. The detector will still display a message on the screen when your pet wants inside or outside.

Also, you can move the locations of the "inside" and "outside" boxes by adjusting the TL_inside, BR_inside, TL_outside, and BR_outside variables.

