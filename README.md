
This tutorial will use Kalibr calibration tool, for more details:
Kalibr on github: https://github.com/ethz-asl/kalibr

## In short

If using Kalibr https://github.com/ethz-asl/kalibr you can follow the youtube video https://www.youtube.com/watch?app=desktop&v=puNXsnrYWTY or the article https://support.stereolabs.com/hc/en-us/articles/360012749113-How-can-I-use-Kalibr-with-the-ZED-Mini-camera-in-ROS- to do the calibration.
Then refer to https://github.com/ethz-asl/kalibr/wiki/yaml-formats to read the yaml file.
Note that the translation matrices are: T_camX_imu and need to be inverted to obtain: T_imu_camX for use in VINS_Fusion

## Detailed tutorial

You can use your device's own factory calibration files (for zed can be found in /usr/local/zed/settings/) which should be more accurate since they use sophisticated equipment (in theory)
(more info about zed calibration file: https://support.stereolabs.com/hc/en-us/articles/360007497173-What-is-the-calibration-file-)

Otherwise, Kalibr calibration is a great tool for calibration, and is compatible with ROS which makes it really easy to use.

First, you need to print the april grid https://github.com/ethz-asl/kalibr/wiki/downloads on a sheet of paper (A4 is good), and then change line 4 in the yaml file to the size of apriltag (the big squares) you printed
tagSize: 0.055             #size of apriltag, edge to edge [m]

Second, you need to record a ROS bag with a appropriate data, follow the instructions in the youtube video https://www.youtube.com/watch?app=desktop&v=puNXsnrYWTY for what type of data (footage) you need to record, and refer to the article https://support.stereolabs.com/hc/en-us/articles/360012749113-How-can-I-use-Kalibr-with-the-ZED-Mini-camera-in-ROS- to copy command line commands from.

Third, Perform the calibration, firstly for the camera(s) only, then for the camera(s) and imu. This may take a while :)

example using ZED mini:

$ kalibr_calibrate_cameras --bag Kalibr_data.bag --topics /zed/zed_node/left/image_rect_color /zed/zed_node/right/image_rect_color --models pinhole-radtan pinhole-radtan --target april_grid.yaml

$ kalibr_calibrate_imu_camera --bag Kalibr_data.bag --cam camchain-Kalibr_data.yaml --imu imu-params.yaml --target april_grid.yaml

You can find the ROS topic of your ZED camera when launching ZED wrapper since it may be different.

Finally, you need to read the calibration file to use it for your VINS-Fusion calibration.

For cameras' intrinsic and parameters (camX.yaml file, or camX_calib):
intrinsics vector: [fx fy px py]
distortion_coeffs: [k1 k2 r1 r2]

For main config.yaml file:
VINS-Fusion requires the matrix T_imu_cam for each camera, which means where each camera if the we set the imu to be at point (0, 0, 0)
But Kalibr provides us with T_cam_imu.
To find T_imu_cam, we need to take the T_cam_imu matrices and inverse them.
You can do that by inputing the array as a 4x4 numpy array and using numpy.linalg.inv() function in python.

Then replace the matrices body_T_camX in the config file with the inversed matrix. (note that the matrix should be in 1D array format for that to work)

For other imu parameters you don't need to change them much for the system to work fine.


Don't forget to change the resolution in both the main config file and imageX_topic files.
