---
layout: post
title: 开源AR库ArUco的原理与位姿估计实战
categories:  AR
description: none
keywords: AR,  Marker, ARUco
---

近期在搭建SLAM-AR的过程中，需要用到Marker作为虚拟坐标系的参考，使用了开源库ArUco实现了该功能。

------

### 参考文献

[ArUco marker detection (aruco module)](https://docs.opencv.org/master/d9/d6d/tutorial_table_of_content_aruco.html)

### 1、ArUco和ChArUco介绍

ArUco标记是可用于相机姿态估计的二进制方形基准标记。它们的主要优点是检测功能强大，快速且简单。但是，ArUco标记的问题之一是，即使在应用亚像素细化之后，其拐角位置的精度也不太高。

另一种常见的图案是纯棋盘，由于每个角都被两个黑色正方形包围，因此可以更精确地细化。但是，找到棋盘图案并不像找到ArUco棋盘那样通用：它必须完全可见，并且不允许遮挡。

ChArUco功能将ArUco标记器与传统棋盘相结合，可以轻松，通用地检测角点。ChArUco的定义可以参考下面这个图：

![charucodefinition.png](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/charucodefinition.png)



aruco模块包括检测这些类型的标记以及将其用于姿势估计和相机校准的工具。作为Opencv-contrib里面的功能，需要`#include <opencv2/aruco/charuco.hpp>`

### 2、各类型Marker的创建

不管哪种类型，都需要创建字典，来规定使用何种精度和id的有效范围



#### 2.1 普通的Marker

```c++
cv::Mat markerImage;
cv::Ptr<cv::aruco::Dictionary> dictionary = cv::aruco::getPredefinedDictionary(cv::aruco::DICT_6X6_250);
cv::aruco::drawMarker(dictionary, 23, 200, markerImage, 1);
cv::imwrite("marker23.png", markerImage);
```

其中23代表id号，200表示输出像素

![marker23.png](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/marker23.png)

#### 2.2 Board

```C++
cv :: Ptr <cv :: aruco :: GridBoard> board = cv :: aruco :: GridBoard :: create（5，7，0.04，0.01，dictionary）;
cv :: Mat boardImage;
board-> draw（cv :: Size（600，500），boardImage，10，1）;
```

其中5，7，分别代表行列的Marker数，0.04，0.01代表单个Marker宽度和间隙宽度，单位任意，但是后面的位姿估计也会以此作为单位，我们默认用米。

输出的网格板中的坐标系位于板平面中，居中于板的左下角，如下图所示（X：红色，Y：绿色，Z：蓝色） 下图中的手来自opencv官方教程，与笔者无关(

![gbocclusion.png](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/gbocclusion.png)

将gridboard替换为CharucoBoard，可以创建另一种类型的板。

![chaxis.png](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/chaxis.png)

Board类型的好处是即使遮掉一部分Marker，坐标轴仍会被检测到。

#### 2.3 Diamond（钻石标记）

在AR中，有时希望基准在Marker的中心，此时可以使用Diamond标记，它是由3x3正方形和白色正方形内的4个ArUco标记组成的棋盘。Diamond标记的检测不是基于标识符，而是基于标记的相对位置。标记标识符可以在同一钻石中或在不同钻石之间重复，并且可以同时被检测而没有歧义。

```c++
cv::Mat diamondImage;
cv::Ptr<cv::aruco::Dictionary> dictionary = cv::aruco::getPredefinedDictionary(cv::aruco::DICT_6X6_250);
cv::aruco::drawCharucoDiamond(dictionary, cv::Vec4i(45,68,28,74), 200, 120, markerImage);
```

其中45,68,28,74代表顶部，左侧，右侧和底部的4个Marker id，正方形尺寸为200像素，标记尺寸为120像素。

![diamondsaxis.png](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/diamondsaxis.png)

### 3、以Diamond为例的检测和位姿估计

首先需要将marker存成图片，参考2.1节。然后打印Marker，量出正方形和标记的实际宽度。

连接摄像头，写入相机内参和畸变。

钻石标记的检测需要事先检测ArUco标记。在检测标记之后，使用`detectCharucoDiamond()`功能检测钻石。

由于ChArUco钻石由其四个角表示，因此可以用与单个ArUco标记相同的方式（即使用`estimatePoseSingleMarkers()`函数）估算其姿态。

```c++
void pose_test() {
	float squareLength = 0.053;
	float markerLength = 0.032;
	cv::VideoCapture inputVideo(0);
	cv::Mat cameraMatrix = (Mat_<double>(3, 3) << 619.281, 0, 327.429, 0, 619.36, 236.488, 0, 0, 1);
	cv::Mat distCoeffs = (Mat_<double>(4, 1)<<0,0,0,0 );
	//camera parameters are read from somewhere

	Ptr<aruco::Dictionary> dictionary = cv::aruco::getPredefinedDictionary(cv::aruco::DICT_4X4_50);
	while (1) {
		cv::Mat image, imageCopy;
		inputVideo>>image;
		image.copyTo(imageCopy);
		std::vector<int> ids;
		std::vector<std::vector<cv::Point2f>> corners;
		std::vector<cv::Vec4i> diamondIds;
		std::vector<std::vector<cv::Point2f>> diamondCorners;
		cv::aruco::detectMarkers(image, dictionary, corners, ids);
		
		// if at least one marker detected
		if (ids.size() > 0) {
			// detect diamon diamonds
			cv::aruco::detectCharucoDiamond(image, corners, ids, squareLength / markerLength, diamondCorners, diamondIds, cameraMatrix, distCoeffs);
			//cv::aruco::drawDetectedMarkers(imageCopy, corners, ids);
			if (diamondIds.size() > 0) {
				cv::aruco::drawDetectedDiamonds(imageCopy, diamondCorners, diamondIds);
				std::vector<cv::Vec3d> rvecs, tvecs;
				//cv::Vec3d rvec, tvec;
				//int valid = cv::aruco::estimatePoseBoard(corners, ids,board, cameraMatrix, distCoeffs, rvec, tvec);
				cv::aruco::estimatePoseSingleMarkers(diamondCorners, squareLength, cameraMatrix, distCoeffs, rvecs, tvecs);
				/*if(valid>0)*/
				cv::aruco::drawAxis(imageCopy, cameraMatrix, distCoeffs, rvecs[0], tvecs[0], 0.05);
				//cv::aruco::drawDetectedMarkers(imageCopy, corners, ids);
				cout << "r:" << rvecs[0] << endl;
				cout << "t:" << tvecs[0] << endl;
			}
			
		}
		cv::imshow("out", imageCopy);
		char key = (char)cv::waitKey(5);
		if (key == 27)
			break;
	}

}
```

