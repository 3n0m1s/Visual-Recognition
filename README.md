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

    const mat = cv.imread('./data/Lenna.png')
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
Extended example using util.js and commons from video file

```js
    const {
      cv,
      getDataFilePath
    } = require('./utils');

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
```
Extended example face and eye detection on image

```js
    const {
      cv,
      getDataFilePath,
      drawBlueRect,
      drawGreenRect
    } = require('./utils');

    const image = cv.imread(getDataFilePath('Lenna.png'));

    const faceClassifier = new cv.CascadeClassifier(cv.HAAR_FRONTALFACE_DEFAULT);
    const eyeClassifier = new cv.CascadeClassifier(cv.HAAR_EYE);

    // detect faces
    const faceResult = faceClassifier.detectMultiScale(image.bgrToGray());

    if (!faceResult.objects.length) {
      throw new Error('No faces detected!');
    }

    const sortByNumDetections = result => result.numDetections
      .map((num, idx) => ({ num, idx }))
      .sort(((n0, n1) => n1.num - n0.num))
      .map(({ idx }) => idx);

    // get best result
    const faceRect = faceResult.objects[sortByNumDetections(faceResult)[0]];
    console.log('faceRects:', faceResult.objects);
    console.log('confidences:', faceResult.numDetections);

    // detect eyes
    const faceRegion = image.getRegion(faceRect);
    const eyeResult = eyeClassifier.detectMultiScale(faceRegion);
    console.log('eyeRects:', eyeResult.objects);
    console.log('confidences:', eyeResult.numDetections);

    // get best result
    const eyeRects = sortByNumDetections(eyeResult)
      .slice(0, 2)
      .map(idx => eyeResult.objects[idx]);

    // draw face detection
    drawBlueRect(image, faceRect);

    // draw eyes detection in face region
    eyeRects.forEach(eyeRect => drawGreenRect(faceRegion, eyeRect));

    cv.imshowWait('face detection', image);
```
Extended example hands gesture recognition on video and or webcam

```js
    const cv = require('opencv4nodejs');
    const { grabFrames } = require('./utils');

    // segmenting by skin color (has to be adjusted)
    const skinColorUpper = hue => new cv.Vec(hue, 0.8 * 255, 0.6 * 255);
    const skinColorLower = hue => new cv.Vec(hue, 0.1 * 255, 0.05 * 255);

    const makeHandMask = (img) => {
      // filter by skin color
      const imgHLS = img.cvtColor(cv.COLOR_BGR2HLS);
      const rangeMask = imgHLS.inRange(skinColorLower(0), skinColorUpper(15));

      // remove noise
      const blurred = rangeMask.blur(new cv.Size(10, 10));
      const thresholded = blurred.threshold(200, 255, cv.THRESH_BINARY);

      return thresholded;
    };

    const getHandContour = (handMask) => {
      const mode = cv.RETR_EXTERNAL;
      const method = cv.CHAIN_APPROX_SIMPLE;
      const contours = handMask.findContours(mode, method);
      // largest contour
      return contours.sort((c0, c1) => c1.area - c0.area)[0];
    };

    // returns distance of two points
    const ptDist = (pt1, pt2) => pt1.sub(pt2).norm();

    // returns center of all points
    const getCenterPt = pts => pts.reduce(
        (sum, pt) => sum.add(pt),
        new cv.Point(0, 0)
      ).div(pts.length);

    // get the polygon from a contours hull such that there
    // will be only a single hull point for a local neighborhood
    const getRoughHull = (contour, maxDist) => {
      // get hull indices and hull points
      const hullIndices = contour.convexHullIndices();
      const contourPoints = contour.getPoints();
      const hullPointsWithIdx = hullIndices.map(idx => ({
        pt: contourPoints[idx],
        contourIdx: idx
      }));
      const hullPoints = hullPointsWithIdx.map(ptWithIdx => ptWithIdx.pt);

      // group all points in local neighborhood
      const ptsBelongToSameCluster = (pt1, pt2) => ptDist(pt1, pt2) < maxDist;
      const { labels } = cv.partition(hullPoints, ptsBelongToSameCluster);
      const pointsByLabel = new Map();
      labels.forEach(l => pointsByLabel.set(l, []));
      hullPointsWithIdx.forEach((ptWithIdx, i) => {
        const label = labels[i];
        pointsByLabel.get(label).push(ptWithIdx);
      });

      // map points in local neighborhood to most central point
      const getMostCentralPoint = (pointGroup) => {
        // find center
        const center = getCenterPt(pointGroup.map(ptWithIdx => ptWithIdx.pt));
        // sort ascending by distance to center
        return pointGroup.sort(
          (ptWithIdx1, ptWithIdx2) => ptDist(ptWithIdx1.pt, center) - ptDist(ptWithIdx2.pt, center)
        )[0];
      };
      const pointGroups = Array.from(pointsByLabel.values());
      // return contour indeces of most central points
      return pointGroups.map(getMostCentralPoint).map(ptWithIdx => ptWithIdx.contourIdx);
    };

    const getHullDefectVertices = (handContour, hullIndices) => {
      const defects = handContour.convexityDefects(hullIndices);
      const handContourPoints = handContour.getPoints();

      // get neighbor defect points of each hull point
      const hullPointDefectNeighbors = new Map(hullIndices.map(idx => [idx, []]));
      defects.forEach((defect) => {
        const startPointIdx = defect.at(0);
        const endPointIdx = defect.at(1);
        const defectPointIdx = defect.at(2);
        hullPointDefectNeighbors.get(startPointIdx).push(defectPointIdx);
        hullPointDefectNeighbors.get(endPointIdx).push(defectPointIdx);
      });

      return Array.from(hullPointDefectNeighbors.keys())
        // only consider hull points that have 2 neighbor defects
        .filter(hullIndex => hullPointDefectNeighbors.get(hullIndex).length > 1)
        // return vertex points
        .map((hullIndex) => {
          const defectNeighborsIdx = hullPointDefectNeighbors.get(hullIndex);
          return ({
            pt: handContourPoints[hullIndex],
            d1: handContourPoints[defectNeighborsIdx[0]],
            d2: handContourPoints[defectNeighborsIdx[1]]
          });
        });
    };

    const filterVerticesByAngle = (vertices, maxAngleDeg) =>
      vertices.filter((v) => {
        const sq = x => x * x;
        const a = v.d1.sub(v.d2).norm();
        const b = v.pt.sub(v.d1).norm();
        const c = v.pt.sub(v.d2).norm();
        const angleDeg = Math.acos(((sq(b) + sq(c)) - sq(a)) / (2 * b * c)) * (180 / Math.PI);
        return angleDeg < maxAngleDeg;
      });

    const blue = new cv.Vec(255, 0, 0);
    const green = new cv.Vec(0, 255, 0);
    const red = new cv.Vec(0, 0, 255);

    // main
    const delay = 20;
    //per video
    //grabFrames('../data/hand-gesture.mp4', delay, (frame) => {

    //per webcam
    //const webcamPort = 0;
    //grabFrames(webcamPort , 1, (frame) => {

    grabFrames('./data/hand-gesture.mp4', delay, (frame) => {
      const resizedImg = frame.resizeToMax(640);

      const handMask = makeHandMask(resizedImg);
      const handContour = getHandContour(handMask);
      if (!handContour) {
        return;
      }

      const maxPointDist = 25;
      const hullIndices = getRoughHull(handContour, maxPointDist);

      // get defect points of hull to contour and return vertices
      // of each hull point to its defect points
      const vertices = getHullDefectVertices(handContour, hullIndices);

      // fingertip points are those which have a sharp angle to its defect points
      const maxAngleDeg = 60;
      const verticesWithValidAngle = filterVerticesByAngle(vertices, maxAngleDeg);

      const result = resizedImg.copy();
      // draw bounding box and center line
      resizedImg.drawContours(
        [handContour],
        blue,
        { thickness: 2 }
      );

      // draw points and vertices
      verticesWithValidAngle.forEach((v) => {
        resizedImg.drawLine(
          v.pt,
          v.d1,
          { color: green, thickness: 2 }
        );
        resizedImg.drawLine(
          v.pt,
          v.d2,
          { color: green, thickness: 2 }
        );
        resizedImg.drawEllipse(
          new cv.RotatedRect(v.pt, new cv.Size(20, 20), 0),
          { color: red, thickness: 2 }
        );
        result.drawEllipse(
          new cv.RotatedRect(v.pt, new cv.Size(20, 20), 0),
          { color: red, thickness: 2 }
        );
      });

      // display detection result
      const numFingersUp = verticesWithValidAngle.length;
      result.drawRectangle(
        new cv.Point(10, 10),
        new cv.Point(70, 70),
        { color: green, thickness: 2 }
      );

      const fontScale = 2;
      result.putText(
        String(numFingersUp),
        new cv.Point(20, 60),
        cv.FONT_ITALIC,
        fontScale,
        { color: green, thickness: 2 }
      );

      const { rows, cols } = result;
      const sideBySide = new cv.Mat(rows, cols * 2, cv.CV_8UC3);
      result.copyTo(sideBySide.getRegion(new cv.Rect(0, 0, cols, rows)));
      resizedImg.copyTo(sideBySide.getRegion(new cv.Rect(cols, 0, cols, rows)));

      cv.imshow('handMask', handMask);
      cv.imshow('result', sideBySide);
    });
```
