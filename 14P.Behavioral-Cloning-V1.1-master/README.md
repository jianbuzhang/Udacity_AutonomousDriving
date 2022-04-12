# 14P.Behavioral-Cloning-V1.1

- time:2018/05/12
- Author:Bob King

---
## The goal /steps of this project are the following:
- Drive the car manually with the simulator to collect enough data to test;
- Build a convolutional neural network in Keras that predicts steering angles from the collected data;
- Train and validate the model with a training and validation set;
- Test the model successfully around track one without going out of the road in aotunomous mode.

---
## Files to hand in &Code quality
- `model.py` contain the script to create and train the autonomous driving model;
- `drive.py` to drive the car in autonomous mode;
- `model.h5` contain a trained convolutional neural network;
- `Readme.md` summarizing this peoject;

---
## Detailed introduction of the behavioral cloning project
- 1.I run this project on Win10 OS with Nvidia GPU Geforce GTX900.In order to get enough training data fast,I set the simulator figures with 1024*768 screen and fastest graphics quality. When I drive the car in manual mode ,do not turn the steering wheel drastically.
- 2.The structure of the model.py is as following:
---
1. At first, we load the anual driving data which includes the images captured by the left ,middle and right cameras, steering angle, throttle ,break and speed.
2. Import keras library to build the deep convolutional neural network.
3. c.Use the model to train the data and save the model as model.h5.
---
- 3.model.h5 contains the CNN network we have just built;
- 4.At last ,we would openthe ipython terminal to start the autonomous driing stage.

> 1. Run the code **python drive.py model.h5**;

> 2. Open the simulator and choose autonomous mode;

> 3. Record the autonomous driving movie.