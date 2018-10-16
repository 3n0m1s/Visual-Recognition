# Visual-Recognition
Face/Eyes/Hands recognition with Node Js


Install windows build tool from an administrative shell

    npm install --global windows-build-tools
         
Install GLOBAL ENVIROMENT TO USE [face-recognition & opencv4nodejs]:

Prerequisites:
    
    GIT - https://gitforwindows.org/
    CMAKE - https://cmake.org/download/
    OPENCV3.4 - https://sourceforge.net/projects/opencvlibrary/files/latest/download
    
Setting up Enviroment variable & system variable :

    1- Copy openCV3.4 in C:/opencv
    2- Set ENVIROMENT VARIABLE   
        name: OPENCV_INCLUDE_DIR path: C:\opencv\build\include
        name: OPENCV_LIB_DIR path: C:\opencv\build\x64\vc15\lib
    3- Set SYSTEM VARIABLE
        name: OPENCV_BIN_DIR
        path: C:\opencv\build\x64\vc15\bin
    4- Add to system PATH
        Path: add to the end -----> %OPENCV_BIN_DIR%
        
AutoBuild Opencv4nodejs:

    npm install opencv-build            
        
After that you can install the nodes module as you know:

    npm install face-recognition
    npm install opencv4nodejs


    
