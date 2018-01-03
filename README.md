# Direct Sparse Odometry with Stereo Cameras 

Stereo DSO

---
Stereo DSO is a real-time stereo SLAM system based on DSO. It is developed by members of Autonomous Driving Group at Horizon Robotics, Inc. It runs on laptops with CPU and provides localization and mapping services for self-driving cars. A demonstration is provides to showcase its capability.
### **Authors:** [Jiatian WU](https://github.com/JiatianWu), [Degang YANG](https://github.com/yangdegang), [Qinrui YAN](https://github.com/castoryan), [Shixin LI](https://github.com/MaidouPP)
### **Videos:** https://youtu.be/JZ0JRrGA6Kc
### **Related Papers:** 
* **Direct Sparse Odometry**, *J. Engel, V. Koltun, D. Cremers*, In arXiv:1607.02565, 2016
* **Large-scale direct SLAM with stereo cameras**, *J. Engel, J. Stückler, D. Cremers*, IROS, 2015
### 1. Installation
Please follow https://github.com/JakobEngel/dso.
### 2. Usage
See https://github.com/JakobEngel/dso for how to run on a dataset. 
Run on a dataset from [http://www.cvlibs.net/datasets/kitti/eval_odometry.php](http://www.cvlibs.net/datasets/kitti/eval_odometry.php) using:

		bin/dso_dataset \
			files=XXXXX/sequence_XX \
			calib=XXXXX/sequence_XX/para/camera.txt \
			gamma=XXXXX/sequence_XX/para/pcalib.txt \
			vignette=XXXXX/sequence_XX/para/vignette.png \
			preset=0 \
			mode=1
Under sequence_XX, there should be two image datasets called image_0 and image_1, which are left and right image sets. Note that mode is set to 1 because we do not have photometric calibration. Besides, gamma and vignette are not required to run the code. That is, a minimal exampel to run on a kitti dataset is:

		bin/dso_dataset \
			files=XXXXX/sequence_XX \
			calib=XXXXX/sequence_XX/para/camera.txt \
			preset=0 \
			mode=1
#### 2.1 Dataset Format
All the dataset format is same as DSO except the calib file. The format of  geometric calibration file is slightly different because of the involved stereo baseline.
##### Geometric Calibration File.
###### Calibration File for Pre-Rectified Images

    Pinhole fx fy cx cy 0
    in_width in_height
    "crop" / "full" / "none" / "fx fy cx cy 0"
    out_width out_height
    baseline

###### Calibration File for FOV camera model:

    FOV fx fy cx cy omega
    in_width in_height
    "crop" / "full" / "fx fy cx cy 0"
    out_width out_height
    baseline


###### Calibration File for Radio-Tangential camera model
    
    RadTan fx fy cx cy k1 k2 r1 r2
    in_width in_height
    "crop" / "full" / "fx fy cx cy 0"
    out_width out_height
    baseline


###### Calibration File for Equidistant camera model

    EquiDistant fx fy cx cy k1 k2 r1 r2
    in_width in_height
    "crop" / "full" / "fx fy cx cy 0"
    out_width out_height
    baseline
note: `baseline` is in meters.
#### 2.2 Run Mode 
Two modes `MODE_SLAM` and `MODE_STEREOMATCH` can be set in `main_dso_pangolin.cpp`. If `MODE_STEREOMATCH` is true, it will do stereo matching and output the idepth map given a pair of stereo images.
#### 3. Pipeline
This work is mostly inspired by **Large-scale direct SLAM with stereo cameras**. Temporal and static stereo are combine in a direct, real-time capable SLAM method.
Key frame and nonkey frame are processed differently in depth map estimation. When a new Keyframe is initialized, we perform static stereo to update and prune the propagated depth map. The nonkey frame is used to refine the depth map of key frame.
See `ImmaturePoint::traceStereo` for Static Stereo Depth Estimation.
See `CoarseTracker::makeCoarseDepthL0` and  `FullSystem::traceNewCoarseNonKey` for depth propogation.
See `CoarseInitialzer::setFirstStereo` and  `FullSystem::InitializeFromInitialzier` for stereo initialization.
#### 4. Experiments 
We tested stereo DSO on kitti datasets and datasets we collect including highway, park and garage. It performs better than DSO in degrees of scale, accuracy, robustness and speed.
Below is the trajectory that DSO runs on Kitti 05. We can see that the scale of DSO is much smaller than groud truth.

![05dso](http://ovms74foj.bkt.clouddn.com/05.png?imageView2/0/w/400/h/400)

Stereo DSO can achieve much better accuracy and faster speed:

![05sdso](http://ovms74foj.bkt.clouddn.com/05sdso.png?imageView2/0/w/402/h/400)

It is evaluated that stereo DSO achieves about 1.1% ~ 4.2% translation error, 0.001deg/m ~ 0.0053deg/m rotation error, with running time of 53ms per frame.

![05tl](http://ovms74foj.bkt.clouddn.com/05_tl.png?imageView2/0/w/402/h/400)

![05rl](http://ovms74foj.bkt.clouddn.com/05_rl.png?imageView2/0/w/402/h/400)

We also tested stereo DSO on Kitti 00.
It achieves about 1.3% ~ 3.7% translation error, 0.002 deg/m ~ 0.007deg/m rotation error, with running time of 56ms per frame. 

![00sdso](http://ovms74foj.bkt.clouddn.com/00.png?imageView2/0/w/402/h/402)

![00tl](http://ovms74foj.bkt.clouddn.com/00_tl.png?imageView2/0/w/402/h/400)

![00rl](http://ovms74foj.bkt.clouddn.com/00_rl.png?imageView2/0/w/402/h/400)

In conlcusion, stereo DSO have several advantages comparing with DSO:

- No need to initialize. Stereo DSO initializes immediately.
- Much better scale and accuracy. DSO performs bad on kitti dataset, especially in scale measurements. Stereo DSO reduced the translation and rotation error largely.
- Real-time speed. Stereo DSO runs at typically 20 frames per sec.
- Robustness. Stereo DSO seldomly gets lost, but DSO usually fails to initialize if the movement during initialization is not large enough.
#### 5. Acknowledgements
Stereo DSO is developed at Horizon Robotics, Inc. Our work is based on [DSO](https://github.com/JakobEngel/dso).
We are still working for improving the performance. Welcome to contribute to Stereo DSO or ask any issues via Github or contacting jiatianwuwork@gmail.com.
