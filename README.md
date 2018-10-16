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

Run example with face-recognition:

```js

    const fr = require('face-recognition')

    fr.winKillProcessOnExit()
    const detector = fr.FaceDetector()
    const img = fr.loadImage('./data/got.jpg')

    console.log('detecting faces')
    const faceSize = 150
    const faces = detector.detectFaces(img, faceSize)

    const win = new fr.ImageWindow()
    win.setImage(fr.tileImages(faces))
    fr.hitEnterToContinue()
```    
Run example with opencv4nodejs:

```js
    const cv = require('opencv4nodejs')
    const fr = require('face-recognition').withCv(cv)

    fr.winKillProcessOnExit()

    const detector = fr.FaceDetector()

    const mat = cv.imread('./../data/Lenna.png')
    const cvImg = fr.CvImage(mat)

    console.log('detecting faces')
    const faceRects =  detector.locateFaces(cvImg)

    const faces = faceRects
      .map(mmodRect => fr.toCvRect(mmodRect.rect))
      .map(cvRect => mat.getRegion(cvRect).copy())

    faces.forEach((face, i) => {
      cv.imshow(`face_${i}`, face)
    })
    cv.waitKey()
```

Extended example using util.js and commons from webcam

```js
    const { cv } = require('./utils');
    const { runVideoFaceDetection } = require('./commons');

    const classifier = new cv.CascadeClassifier(cv.HAAR_FRONTALFACE_ALT2);

    const webcamPort = 0;

    function detectFaces(img) {
      // restrict minSize and scaleFactor for faster processing
      const options = {
        minSize: new cv.Size(100, 100),
        scaleFactor: 1.2,
        minNeighbors: 10
      };
      return classifier.detectMultiScale(img.bgrToGray(), options).objects;
    }

    runVideoFaceDetection(webcamPort, detectFaces);
```
Extended example using util.js and commons from webcam

```js
    const {
      cv,
      getDataFilePath
    } = require('../utils');

    const { runVideoFaceDetection } = require('./commons');

    const videoFile = getDataFilePath('people.mp4');

    const classifier = new cv.CascadeClassifier(cv.HAAR_FRONTALFACE_ALT2);

    function detectFaces(img) {
      // restrict minSize and scaleFactor for faster processing
      const options = {
        // minSize: new cv.Size(40, 40),
        // scaleFactor: 1.2,
        scaleFactor: 1.1,
        minNeighbors: 10
      };
      return classifier.detectMultiScale(img.bgrToGray(), options).objects;
    }

    runVideoFaceDetection(videoFile, detectFaces);

