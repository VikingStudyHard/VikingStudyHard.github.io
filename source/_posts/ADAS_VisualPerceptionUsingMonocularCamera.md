---

title: 【ADAS】Visual Perception Using Monocular Camera

date: 2018.5.8

categories: 自动驾驶仿真测试

tags: 
 - Matlab
 - CATARC

description: Automated Driving System Toolbox 提供了一套通过摄像机的数据检测并追踪车道、机动车、行人的计算机视觉算法：Visual Perception Using Monocular Camera，使用单目照相机的视觉感知

---


# 简介
单目摄像头系统，传感器可以完成：
1. 车道边界检测
2. 车、人识别
3. 本车到障碍物之间的距离估计

发出车道偏离警告、碰撞警告或设计车道保持辅助控制系统。也可以配合其他传感器实现紧急制动系统和其他安全功能。

# 代码流程
## 定义摄像头配置
掌握相机本身和外部的校准参数对准确转换像素和机动车坐标来说非常关键。

### 定义相机本身校准参数

以下参数是使用棋盘校准模式(checkerboard calibration pattern)的相机校准程序较早确定的。使用 cameraCalibrator app 从摄像头获得数据。

```matlab
focalLength = [309.4362, 344.2161]; % [fx, fy] in pixel units
principalPoint = [318.9034, 257.5352]; % [cx, cy] optical center in pixel coordinates 像素坐标中的光学中心
imageSize = [480, 640]; % [nrows, mcols]
```

镜头失真系数 (lens distortion coefficients)被忽略，因为数据中有小的失真。参数被储存到`cameraIntrinsics`对象里。
```matlab
camIntrinsics = cameraIntrinsics(focalLength, principalPoint, imageSize);
```
### 定义相对于车辆底盘的摄像机方向
使用此信息建立摄像机外部定位系统，来定义3-D摄像机坐标系相对于车辆坐标系的位置。

```matlab
height = 2.1798; % mounting height in meters from the ground 离地面距离的安装高度（米） 
pitch = 14; % pitch of the camera in degrees 相机的pitch的百分比度数
```

上述量可以从 `extrinsics` 函数返回的旋转和平移矩阵中导出。Pitch指定相机水平位置倾斜度。 对于本例中使用的摄像机，传感器的横摇和横摆均为零。 定义内部函数和外部函数的整个配置存储在 `monoCamera` 对象中。

```matlab
sensor = monoCamera(camIntrinsics, height, 'Pitch', pitch);
```

`monoCamera` 对象X轴指向机动车行驶方向，Y轴指向机动车左侧。

![IMAGE](/images/ADAS_VisualPerceptionUsingMonocularCamera/557A7853C421B5C93AD046C18D3D24DC.jpg)

坐标系的原点位于地面上，正下方是由相机焦点定义的相机中心。 

`monoCamera` 还提供 `imageToVehicle` 和 `vehicleToImage` 方法，用于图像和车辆坐标系之间的转换。
注意：坐标系之间的转换采用平坦的道路。它基于建立单应矩阵，将成像平面上的位置映射到路面上的位置。 非平坦的道路会在距离计算中引入误差，特别是在距离车辆很远的地方。

## 加载一帧视频

首先创建一个打开视频文件的 `VideoReader` 对象。为了提高内存效率，`VideoReader` 一次加载一个视频帧。

```matlab
videoName = 'caltech _ cordova1.avi'; 
videoReader = VideoReader(videoName);
```
读取包含车道和机动车的视频：

```matlab
timeStamp = 0.06667; % time from the beginning of the video 
videoReader.CurrentTime = timeStamp; % point to the chosen frame

frame = readFrame(videoReader); % read frame at timeStamp seconds 
imshow(frame) % display frame
```
例子中忽略了镜头失真。若要考虑镜头失真引入的距离测量的错误，要用 `undistortImage`函数移除镜头失真。

## 建立鸟瞰视角图

分割和检测车道标记方法之一：使用鸟瞰图像变换。

缺点：带来计算成本
优势：
鸟瞰图中的车道标记厚度均匀——简化了分割过程。
属于相同车道的车道标记变得平行——便于进一步的分析。

对于给定相机的设置，`birdEyeView` 对象将原图转为鸟瞰图。它允许使用车辆坐标指定要变换的区域。

当摄像机安装高度以米为单位指定时，车辆坐标单位由 `monoCamera` 对象建立。例如，如果高度以毫米为单位指定，则模拟的其余部分将使用毫米。

```matlab
% Using vehicle coordinates, define area to transform
distAheadOfSensor = 30; % in meters, as previously specified inmonoCamera ... 
height input
spaceToOneSide = 6; % all other distance quantities are also in meters 
bottomOffset = 3;

outView = [bottomOffset, distAheadOfSensor, -spaceToOneSide, ...
spaceToOneSide]; % [xmin, xmax, ymin, ymax]
imageSize = [NaN, 250]; % output image width in pixels; height is ... 
chosen automatically to preserve units per pixel ratio

birdsEyeConfig = birdsEyeView(sensor, outView, imageSize);
```
生成鸟瞰图：
```matlab
birdsEyeImage = transformImage(birdsEyeConfig, frame); 
figure
imshow(birdsEyeImage)
```

距传感器更远的区域更模糊，因为具有较少的像素并需要更大量的插值。
只要可以在车辆坐标中找到车道边界候选像素，就可以在不使用鸟瞰视图的情况下完成后面的处理步骤。

## 在车辆坐标中查找车道标记

有了鸟瞰图，可用 `segmentLaneMarkerRidge` 方法从路面分割车道标记候选像素。
参数要用世界通用单位。


```matlab
% Convert to grayscale
birdsEyeImage = rgb2gray(birdsEyeImage);

% Lane marker segmentation ROI in world units
vehicleROI = outView - [-1, 2, -3, 3]; % look 3 meters to left and right, ... 
and 4 meters ahead of the sensor
approxLaneMarkerWidthVehicle = 0.25; % 25 centimeters

% Detect lane features
laneSensitivity = 0.25;
birdsEyeViewBW = segmentLaneMarkerRidge(birdsEyeImage, birdsEyeConfig, approxLaneMarkerWidthVehicle,...
    ‘ROI’, vehicleROI, ‘Sensitivity’, laneSensitivity);

figure
imshow(birdsEyeViewBW)
```

适用于车辆路径上的车道标记。穿过道路的道路标记或涂在沥青上的道路标志标记不了。

## 确定道路的边界
上一步中找到的一些曲线可能仍然无效。 例如，当曲线适合人行横道标记时。 使用额外的启发式来回避许多这样的曲线。

```matlab
% Establish criteria for rejecting boundaries based on their length
maxPossibleXLength = diff(vehicleROI(1:2));
minXLength = maxPossibleXLength * 0.60; % establish a threshold

% Reject short boundaries 回避短线
isOfMinLength = arrayfun(@(b)diff(b.XExtent) > minXLength, boundaries); 
boundaries = boundaries(isOfMinLength);
```

根据 `findParabolicLaneBoundaries` 函数计算的强度指标移除其他边界。根据ROI和图像大小设置车道强度阈值。

```matlab
% To compute the maximum strength, assume all image pixels within the ROI 
% are lane candidate points
birdsImageROI = vehicleToImageROI(birdsEyeConfig, vehicleROI); 
[laneImageX,laneImageY] = meshgrid(birdsImageROI(1):birdsImageROI(2),birdsImageROI(3):birdsImageROI(4));

% Convert the image points to vehicle points
vehiclePoints = imageToVehicle(birdsEyeConfig,[laneImageX(:),laneImageY(:)]);

% Find the maximum number of unique x-axis locations possible for any lane 
% boundary
maxPointsInOneLane = numel(unique(vehiclePoints(:,1)));

% Set the maximum length of a lane boundary to the ROI length
maxLaneLength = diff(vehicleROI(1:2));

% Compute the maximum possible lane strength for this image size/ROI size 
% specification
maxStrength = maxPointsInOneLane/maxLaneLength;

% Reject weak boundaries
isStrong = [boundaries.Strength] > 0.4*maxStrength; 
boundaries = boundaries(isStrong);

```
将车道标记类型分为单/双、实线/虚线的启发式方法包含在本例底部列出的辅助函数中。
了解车道标记类型对于自动驾驶车辆至关重要。如：禁止车辆穿过双实线。

```matlab
% Locate two ego lanes if they are present
xOffset = 0; % 0 meters from the sensor 
distanceToBoundaries = boundaries.computeBoundaryModel(xOffset);

% Find candidate ego boundaries
leftEgoBoundaryIndex = [];
rightEgoBoundaryIndex = [];
minLDistance = min(distanceToBoundaries(distanceToBoundaries>0));

minRDistance = max(distanceToBoundaries(distanceToBoundaries<=0)); 
if ~isempty(minLDistance)
  leftEgoBoundaryIndex = distanceToBoundaries == minLDistance;
end
if ~isempty(minRDistance)
  rightEgoBoundaryIndex = distanceToBoundaries == minRDistance;
end
leftEgoBoundary = boundaries(leftEgoBoundaryIndex); 
rightEgoBoundary = boundaries(rightEgoBoundaryIndex);
```
在鸟瞰图像和常规视图中显示检测到的车道标记。
```matlab
xVehiclePoints = bottomOffset:distAheadOfSensor;
birdsEyeWithEgoLane = insertLaneBoundary(birdsEyeImage, leftEgoBoundary , birdsEy Config, xVehiclePoints, ‘Color’,’Red’);
birdsEyeWithEgoLane = insertLaneBoundary(birdsEyeWithEgoLane, rightEgoBoundary, birdsEyeConfig, xVehiclePoints, ‘Color’,’Green’);

frameWithEgoLane = insertLaneBoundary(frame, leftEgoBoundary, sensor, xVehiclePoints, ‘Color’,’Red’);
frameWithEgoLane = insertLaneBoundary(frameWithEgoLane, rightEgoBoundary, sensor, xVehiclePoints, ‘Color’,’Green’);

figure
subplot(‘Position’, [0, 0, 0.5, 1.0]) % [left, bottom, width, height] in normalized units
imshow(birdsEyeWithEgoLane)
subplot(‘Position’, [0.5, 0, 0.5, 1.0])
imshow(frameWithEgoLane)
```
## 在车辆坐标中查找车辆
在前方碰撞警告（FCW,front collision warning）和自动紧急制动（AEB,automated emergency braking）系统中，检测和跟踪车辆至关重要。

加载经过预训练的聚集信道特征（ACF,aggregate channel features）检测器，以检测车辆的前部和后部。 像这样的探测器可以处理发出碰撞警告的场景。

```matlab
detector = vehicleDetectorACF();
% Width of a common vehicles is between 1.5 to 2.5 meters
vehicleWidth = [1.5, 2.5];
```

使用 `configureDetectorMonoCamer` 函数来专用通用ACF检测器，以考虑典型汽车应用的几何结构。 通过在这个摄像头进行配置，新探测器只搜索道路表面的车辆，因为没有点能在高于消失点的地方寻找车辆。这节省了计算时间并减少了误报的数量。

```matlab
monoDetector = configureDetectorMonoCamera(detector, sensor, vehicleWidth); 
[bboxes, scores] = detect(monoDetector, frame);
```

上述处理单个帧，追踪请看：
[Tracking Multiple Vehicles From a Camera example](https://www.mathworks.com/help/driving/examples/track-multiple-vehicles-using-a-camera.html)


接下来，将车辆检测转换为车辆坐标。 

`computeVehicleLocations` 函数包含在本例最后，它给出了一个由图像坐标中的检测算法返回的边界框在车辆坐标系中的车辆位置，返回车辆坐标中边界框底部的中心位置。

由于我们使用的是单眼相机传感器和简单的单应矩阵，因此只有**沿道路表面的距离**才能精确计算。计算三维空间中的任意位置需要使用立体相机或其他能够进行三角测量的传感器。


```matlab
locations = computeVehicleLocations(bboxes, sensor);

% Overlay the detections on the video frame 追踪多辆车
imgOut = insertVehicleDetections(frame, locations, bboxes); 
figure;
imshow(imgOut);

```
## 用视频输入模拟完整的传感器

将视频倒回到开头，然后处理视频。下面的代码缩短了，因为所有关键参数都是在前面的步骤中定义的。 。


```matlab
videoReader.CurrentTime = 0;
isPlayerOpen = true;
snapshot = [];

while hasFrame(videoReader) && isPlayerOpen
  % Grab a frame of video
  frame = readFrame(videoReader);
  
  % Compute birdsEyeView image
  birdsEyeImage = transformImage(birdsEyeConfig, frame); 
  birdsEyeImage = rgb2gray(birdsEyeImage);
  
  % Detect lane boundary features
  birdsEyeViewBW = segmentLaneMarkerRidge(birdsEyeImage, birdsEyeConfig, ... 
    approxLaneMarkerWidthVehicle, ‘ROI’, vehicleROI, ...
    ‘Sensitivity’, laneSensitivity);
    
  % Obtain lane candidate points in vehicle coordinates
  [imageX, imageY] = find(birdsEyeViewBW);
  xyBoundaryPoints = imageToVehicle(birdsEyeConfig, [imageY, imageX]);
  
  % Find lane boundary candidates
  [boundaries, boundaryPoints] = findParabolicLaneBoundaries(xyBoundarPoints,boundaryWidth, ...
    ‘MaxNumBoundaries’, maxLanes, ‘validateBoundaryFcn’, @validateBoundaryFcn);
  

  % Reject boundaries based on their length and strength
  isOfMinLength = arrayfun(@(b)diff(b.XExtent) > minXLength, boundaries);
  boundaries = boundaries(isOfMinLength);
  isStrong = [boundaries.Strength] > 0.2*maxStrength; 
  boundaries = boundaries(isStrong);
  
  % Classify lane marker type
  boundaries = classifyLaneTypes(boundaries, boundaryPoints);
  
  % Find ego lanes
  xOffset = 0; 
  % 0 meters from the sensor 
  distanceToBoundaries = boundaries.computeBoundaryModel(xOffset); 
  % Find candidate ego boundaries
  
  leftEgoBoundaryIndex = [];
  rightEgoBoundaryIndex = [];
  minLDistance = min(distanceToBoundaries(distanceToBoundaries>0)); 
  minRDistance = max(distanceToBoundaries(distanceToBoundaries<=0)); 
  if ~isempty(minLDistance)
    leftEgoBoundaryIndex = distanceToBoundaries == minLDistance;
  end
  if ~isempty(minRDistance)
    rightEgoBoundaryIndex = distanceToBoundaries == minRDistance;
  end
  leftEgoBoundary = boundaries(leftEgoBoundaryIndex); 
  rightEgoBoundary = boundaries(rightEgoBoundaryIndex);
  
  
  % Detect vehicles
  [bboxes, scores] = detect(monoDetector, frame); 
  locations = computeVehicleLocations(bboxes, sensor);
  
  % Visualize sensor outputs and intermediate results. Pack the core 
  % sensor outputs into a struct.
  sensorOut.leftEgoBoundary = leftEgoBoundary; 
  sensorOut.rightEgoBoundary = rightEgoBoundary; 
  sensorOut.vehicleLocations = locations;
  
  sensorOut.xVehiclePoints = bottomOffset:distAheadOfSensor; 
  sensorOut.vehicleBoxes = bboxes;
  
  % Pack additional visualization data, including intermediate results
  intOut.birdsEyeImage = birdsEyeImage; 
  intOut.birdsEyeConfig = birdsEyeConfig;
  intOut.vehicleScores = scores;
  intOut.vehicleROI = vehicleROI;
  intOut.birdsEyeBW = birdsEyeViewBW;
  
  closePlayers = ~hasFrame(videoReader);
  isPlayerOpen = visualizeSensorResults(frame, sensor, sensorOut, ...
    intOut, closePlayers);
    
  timeStamp = 7.5333; % take snapshot for publishing at timeStamp seconds 
  if abs(videoReader.CurrentTime - timeStamp) < 0.01
    snapshot = takeSnapshot(frame, sensor, sensorOut);
  end
end
```
显示视频帧,在timeStamp秒处拍摄快照。
```matlab
if ~isempty(snapshot) 
  figure
  imshow(snapshot)
end
```
## 在不同的视频上尝试传感器设计
`helperMonoSensor` 类将安装程序和所有必要的步骤组装到一起，以将单眼相机传感器模拟为可应用于任何视频的完整套件。由于传感器设计所使用的大多数参数均基于世界单位，因此该设计对于相机参数（包括图像尺寸）的变化具有很强的鲁棒性。请注意，辅助MonoSensor类中的代码与前一节中的循环不同，后者用于说明基本概念。

除了提供新视频之外，您还必须提供与该视频相对应的摄像机配置。该过程如下所示。试试你自己的视频。

```matlab
% Sensor configuration
focalLength = [309.4362, 344.2161]; 
principalPoint = [318.9034, 257.5352];
imageSize = [480, 640];
height = 2.1798; % mounting height in meters from the ground 
pitch = 14; % pitch of the camera in degrees

camIntrinsics = cameraIntrinsics(focalLength, principalPoint, imageSize); 
sensor = monoCamera(camIntrinsics, height, ‘Pitch’, pitch);
videoReader = VideoReader(‘caltech _ washington1.avi’);
```

创建 `helperMonoSensor` 对象并将其应用于视频。

```matlab
monoSensor = helperMonoSensor(sensor);
monoSensor.LaneXExtentThreshold = 0.5;
% To remove false detections from shadows in this video, we only return 
% vehicle detections with higher scores. 
monoSensor.VehicleDetectionThreshold = 20;

isPlayerOpen = true;
snapshot = [];
while hasFrame(videoReader) && isPlayerOpen
  frame = readFrame(videoReader); % get a frame
  sensorOut = processFrame(monoSensor, frame);
  closePlayers = ~hasFrame(videoReader);
  isPlayerOpen = displaySensorOutputs(monoSensor, frame, sensorOut, closePlayers);
  timeStamp = 11.1333; % take snapshot for publishing at timeStamp seconds 
  if abs(videoReader.CurrentTime - timeStamp) < 0.01
    snapshot = takeSnapshot(frame, sensor, sensorOut);
  end
end
```
## 支持功能
`visualizeSensorResults` 显示单眼相机传感器模拟的核心信息和中间结果。

```matlab
function isPlayerOpen = visualizeSensorResults(frame, sensor, sensorOut,...
  intOut, closePlayers)
  % Unpack the main inputs
  leftEgoBoundary = sensorOut.leftEgoBoundary; 
  rightEgoBoundary = sensorOut.rightEgoBoundary; 
  locations = sensorOut.vehicleLocations;
  
  xVehiclePoints = sensorOut.xVehiclePoints; 
  bboxes = sensorOut.vehicleBoxes;
  
  % Unpack additional intermediate data
  birdsEyeViewImage = intOut.birdsEyeImage;
  birdsEyeConfig = intOut.birdsEyeConfig; 
  vehicleROI = intOut.vehicleROI;
  birdsEyeViewBW = intOut.birdsEyeBW;
  
  % Visualize left and right ego-lane boundaries in bird’s-eye view
  birdsEyeWithOverlays = insertLaneBoundary(birdsEyeViewImage, leftEgoBoundary , birdsEyeConfig, xVehiclePoints, ‘Color’,’Red’);
  birdsEyeWithOverlays = insertLaneBoundary(birdsEyeWithOverlays, rightEgoBoundary, birdsEyeConfig, xVehiclePoints, ‘Color’,’Green’);
  
  % Visualize ego-lane boundaries in camera view
  frameWithOverlays = insertLaneBoundary(frame, leftEgoBoundary, sensor, xVehicl Points, ‘Color’,’Red’);
  frameWithOverlays = insertLaneBoundary(frameWithOverlays, rightEgoBoundary, sensor, xVehiclePoints, ‘Color’,’Green’);
  
  frameWithOverlays = insertVehicleDetections(frameWithOverlays, locations, bboxes);
  imageROI = vehicleToImageROI(birdsEyeConfig, vehicleROI);
  ROI = [imageROI(1) imageROI(3) imageROI(2)-imageROI(1) imageROI(4)-imageROI(3)];
  
  % Highlight candidate lane points that include outliers
  birdsEyeViewImage = insertShape(birdsEyeViewImage, ‘rectangle’, ROI); % show detection ROI
  birdsEyeViewImage = imoverlay(birdsEyeViewImage, birdsEyeViewBW, ‘blue’);
  
  % Display the results
  frames = {frameWithOverlays, birdsEyeViewImage, birdsEyeWithOverlays};
  persistent players; 
  if isempty(players)
    frameNames = {‘Lane marker and vehicle detections’, ‘Raw segmentation’, ‘Lane marker detections’};
    players = helperVideoPlayerSet(frames, frameNames);
  end
  update(players, frames);
  
  % Terminate the loop when the first player is closed
  isPlayerOpen = isOpen(players, 1);
  if (~isPlayerOpen || closePlayers) % close down the other players 
    clear players;
  end 
end
```

`computeVehicleLocations`计算车辆在车辆坐标中的位置，给定由图像坐标中的检测算法返回的边界框。它返回车辆坐标中边界框底部的中心位置。
由于使用单眼相机传感器和简单的单应矩阵，因此只能计算沿道路表面的距离。计算三维空间中的任意位置需要使用立体相机或其他能够进行三角测量的传感器。

```matlab
function locations = computeVehicleLocations(bboxes, sensor) locations = zeros(size(bboxes,1),2);
  for i = 1:size(bboxes, 1)
    bbox = bboxes(i, :);
    % Get [x,y] location of the center of the lower portion of the
    % detection bounding box in meters. bbox is [x, y, width, height] in 
    % image coordinates, where [x,y] represents upper-left corner. 
    yBottom = bbox(2) + bbox(4) - 1;
    xCenter = bbox(1) + (bbox(3)-1)/2; % approximate center
    
    locations(i,:) = imageToVehicle(sensor, [xCenter, yBottom]);
  end 
end

```
`insertVehicleDetections` 插入边界框并显示与返回车辆检测相对应的[x，y]位置。

```matlab
function imgOut = insertVehicleDetections(imgIn, locations, bboxes)
  imgOut = imgIn;
  for i = 1:size(locations, 1) 
    location = locations(i, :); 
    bbox = bboxes(i, :);
    
    label = sprintf(‘X=%0.2f, Y=%0.2f’, location(1), location(2));
    imgOut = insertObjectAnnotation(imgOut, ... 
      ‘rectangle’, bbox, label, ‘Color’,’g’);
  end
end
```

`vehicleToImageROI` 将车辆坐标中的ROI转换为鸟瞰图像中的图像坐标。
```matlab
function imageROI = vehicleToImageROI(birdsEyeConfig, vehicleROI)
  vehicleROI = double(vehicleROI);
  loc2 = abs(vehicleToImage(birdsEyeConfig, [vehicleROI(2) vehicleROI(4)])); 
  loc1 = abs(vehicleToImage(birdsEyeConfig, [vehicleROI(1) vehicleROI(4)])); 
  loc4 = vehicleToImage(birdsEyeConfig, [vehicleROI(1) vehicleROI(4)]); 
  loc3 = vehicleToImage(birdsEyeConfig, [vehicleROI(1) vehicleROI(3)]);
  
  [minRoiX, maxRoiX, minRoiY, maxRoiY] = deal(loc4(1), loc3(1), loc2(2), loc1(2)); 
  imageROI = round([minRoiX, maxRoiX, minRoiY, maxRoiY]);
end
```
`validateBoundaryFcn` 拒绝使用RANSAC算法计算出的一些车道边界曲线。

```matlab
function isGood = validateBoundaryFcn(params) 
  if ~isempty(params)
    a = params(1);
    % Reject any curve with a small ‘a’ coefficient, which makes it highly
    % curved.
    isGood = abs(a) < 0.003; % a from ax^2+bx+c
  else
    isGood = false;
  end 
end
```

classifyLaneTypes determines lane marker types as solid, dashed, etc.
```matlab
function boundaries = classifyLaneTypes(boundaries, boundaryPoints)
  for bInd = 1 : numel(boundaries)
    vehiclePoints = boundaryPoints{bInd};
    
    % Sort by x
    vehiclePoints = sortrows(vehiclePoints, 1);
    xVehicle = vehiclePoints(:,1); 
    xVehicleUnique = unique(xVehicle);
    
    % Dashed vs solid
    xdiff = diff(xVehicleUnique);
    % Sufficiently large threshold to remove spaces between points of a 
    % solid line, but not large enough to remove spaces between dashes 
    xdifft = mean(xdiff) + 3*std(xdiff);
    largeGaps = xdiff(xdiff > xdifft);
    
    % Safe default
    boundaries(bInd).BoundaryType= LaneBoundaryType.Solid; 
    if largeGaps>2
      % Ideally, these gaps should be consistent, but you cannot rely
      % on that unless you know that the ROI extent includes at least 3 dashes. 
      boundaries(bInd).BoundaryType = LaneBoundaryType.Dashed;
    end 
  end
end
```
用`takeSnapshot` 捕获HTML发布报告的输出。

```matlab
function I = takeSnapshot(frame, sensor, sensorOut)
  % Unpack the inputs
  leftEgoBoundary = sensorOut.leftEgoBoundary; rightEgoBoundary = sensorOut.rightEgoBoundary;
  locations = sensorOut.vehicleLocations; 
  xVehiclePoints = sensorOut.xVehiclePoints;
  bboxes = sensorOut.vehicleBoxes;
  
  frameWithOverlays = insertLaneBoundary(frame, leftEgoBoundary, sensor, xVehicle Points, ‘Color’,’Red’);
  frameWithOverlays = insertLaneBoundary(frameWithOverlays, rightEgoBoundary, sensor, xVehiclePoints, ‘Color’,’Green’);
  frameWithOverlays = insertVehicleDetections(frameWithOverlays, locations, bboxes);
  I = frameWithOverlays;
end
```




# 实践
运行 /Applications/MATLAB_R2017b.app/toolbox/driving/drivingdemos/ 下的 MonoCameraExample.m

程序首先设置了相机参数以及相机相对汽车的位置，导入视频caltech_cordova1.avi，截取一帧得到下图：

![IMAGE](/images/ADAS_VisualPerceptionUsingMonocularCamera/8B0AC551304894E1B042876F3D2E5DCE.jpg)

然后将原图转换为鸟瞰图：

![IMAGE](/images/ADAS_VisualPerceptionUsingMonocularCamera/B525B730D2235B93AC65A2061922C87E.jpg)

在二进制鸟瞰图中找出车道：

![IMAGE](/images/ADAS_VisualPerceptionUsingMonocularCamera/E2B16EEAF6367D72A6CCAE237F31A3F6.jpg)

区分出左右两条车道：

![IMAGE](/images/ADAS_VisualPerceptionUsingMonocularCamera/F53B95EA034F12E20A580B149FE09413.jpg)

标记汽车，x代表行车方向，y代表汽车左侧：

![IMAGE](/images/ADAS_VisualPerceptionUsingMonocularCamera/FE913FC0F560E17D42B2CE342CB41B0B.jpg)

从头放映视频，并实时标记出车道和汽车：

![IMAGE](/images/ADAS_VisualPerceptionUsingMonocularCamera/946852F1410E1EB0163453C4D3EED636.jpg)

![IMAGE](/images/ADAS_VisualPerceptionUsingMonocularCamera/339330D8C2CBB830FD868D04B17B2108.jpg)

![IMAGE](/images/ADAS_VisualPerceptionUsingMonocularCamera/81CDCECCC3A36FDA5DDD6F1BD33D10CE.jpg)

快照：

![IMAGE](/images/ADAS_VisualPerceptionUsingMonocularCamera/4C6D4DCA6AC3CC4F2DC92592C0C57A7E.jpg)

另一个视频做实验：

![IMAGE](/images/ADAS_VisualPerceptionUsingMonocularCamera/E98E990E0104C238BAEA845B794BC961.jpg)

![IMAGE](/images/ADAS_VisualPerceptionUsingMonocularCamera/056B4F0CE8CEF9CF21D2AD172D86DA24.jpg)


快照：

![IMAGE](/images/ADAS_VisualPerceptionUsingMonocularCamera/D1558F50A4E5B85F7BD53F3243A4B5EE.jpg)






# 附
monocular [mɒ'nɒkjʊlə]  
adj. 单眼的;单眼用的;单目镜的;单筒的
monocular camera sensor

tailor
n. 裁缝，成衣工
vi.做裁缝
vt.调整使适应;为…裁制衣服;为…做衣服;剪裁，制作

pedestrian 英 [pəˈdestriən]   美 [pəˈdɛstriən]  
n.行人;步行者
adj.徒步的;平淡无奇的;一般的

calibration
英 [ˌkælɪˈbreɪʃn]   美 [ˌkæləˈbreʃən]  
n.校准，标准化;刻度，标度;测量口径 
calibration parameters

departure
英 [dɪˈpɑ:tʃə(r)]   美 [dɪˈpɑ:rtʃə(r)]  
n.背离;离开，离去;起程;东西距离
issue lane departure warnings 发出车道偏离警告

distortion
英 [dɪ'stɔ:ʃn]   美 [dɪˈstɔrʃən]  
n.扭曲，变形;失真，畸变;[心理学] 扭转
lens distortion 镜头失真



