# Visual-Recognition
Face/Eyes/Hands recognition with Node Js


Install windows build tool from an administrative shell

    npm install --global windows-build-tools
         
Install GLOBAL ENVIROMENT TO USE [face-recognition & opencv4nodejs]:

Prerequisites:
    GIT - https://gitforwindows.org/
    CMAKE - https://cmake.org/download/
    OPENCV3.4 - https://sourceforge.net/projects/opencvlibrary/files/latest/download
    
Setting up Enviroment variable on system global variable enviroment:
    1- Copy openCV3.4 in C:/opencv
    2- Set variable 
                
    name: OPENCV_INCLUDE_DIR path: C:\opencv\build\include
    name: OPENCV_LIB_DIR path: C:\opencv\build\x64\vc15\lib
  
After that you can install the nodes module as you know:

    npm install face-recognition
    npm install opencv4nodejs
    
    
