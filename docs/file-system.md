# The file system
During annotation, training and analysis, OCTRON creates various subfolders and files in the main project folder you selected.

## Annotation 
For every video you are annotating, OCTRON creates a subfolder that contains:

- `label_name masks.zarr` archives that contain the annotation mask data for every label and annotation you create.
- `object_organizer.json` file that contains info on the label and various layers in your annotation project.
- `video data.zarr` archive that contains (compressed) video frame data for every annotated frame.

The folder names are the abbreviated hashes of the video file that is loaded and OCTRON makes sure that whenever you re-load an annotation project from within the GUI, the hash of the loaded video and the one of the annotation subfolder match. 
Meaning, when you modify the video file in any way, OCTRON will throw an error, alerting you that the video file and loaded annotations do not match. There is also a `video_info.txt` that can be deleted without consequences and contains some basic info about the annotated video file itself. 

OCTRON uses the [zarr > 3.0](https://zarr.readthedocs.io/en/stable/index.html) python libary for reading and writing annotation, frame and prediction mask data. 

```
Your project folder
├─ hash_of_video_1
│  └─ zarr label folder A 
│  └─ zarr label folder B
│  └─ zarr video data
│  └─ ...
│  └─ object_organizer.json
│  └─ video_info.txt
├─ hash_of_video_2
│  └─ zarr label folder A 
│  └─ zarr label folder B 
│  └─ zarr label folder C 
│  └─ zarr video data
│  └─ ...
│  └─ object_organizer.json
│  └─ video_info.txt
└─ ...
```

```json title="Example object_organizer.json"
{
  "entries": {
    "0": {
      "label": "body",
      "suffix": "",
      "label_id": 0,
      "color": [
        0.5647058823529412,
        0.054901960784313725,
        0.6470588235294118,
        1.0
      ],
      "annotation_layer_metadata": {
        "name": "body points",
        "type": "Points",
        "visible": true,
        "opacity": 0.6
      },
      "prediction_layer_metadata": {
        "name": "body masks",
        "type": "Labels",
        "num_predicted_indices": 29,
        "data_shape": [
          302,
          300,
          300
        ],
        "ndim": 3,
        "visible": true,
        "opacity": 0.4,
        "zarr_path": "0c883c88/body masks.zarr",
        "video_file_path": "videos/20150311_EC_light_snippet3.mp4",
        "video_hash": "0c883c88",
        "colormap_name": "custom"
      }
    },
    "1": {
      "label": "tentacles",
      "suffix": "",
      "label_id": 1,
      "color": [
        0.1411764705882353,
        0.9411764705882353,
        0.7529411764705882,
        1.0
      ],
      "annotation_layer_metadata": {
        "name": "tentacles points",
        "type": "Points",
        "visible": true,
        "opacity": 0.6
      },
      "prediction_layer_metadata": {
        "name": "tentacles masks",
        "type": "Labels",
        "num_predicted_indices": 29,
        "data_shape": [
          302,
          300,
          300
        ],
        "ndim": 3,
        "visible": true,
        "opacity": 0.4,
        "zarr_path": "0c883c88/tentacles masks.zarr",
        "video_file_path": "videos/20150311_EC_light_snippet3.mp4",
        "video_hash": "0c883c88",
        "colormap_name": "custom"
      }
    }
  },
  "time_last_changed": "2025-05-28T10:53:16.154549"
}
```

## Training
With the creation of training output (in OCTRON's training tab), a `model` subfolder is created in your project folder.
Within it you will find a `training_data` subfolder that contains the actual training data for YOLO and a `yolo_config.yaml`, that contains all parameters that the model is being trained with. 
You can use the training data for multiple training runs (if you unticked `Overwrite` in the training tab of the GUI). 

Once you start training the YOLO model, a `training` subfolder is created. It will gradually fill up with info during training and contain evaluation metrics and figures after training has finished. 
If you want to read more about these files, check out ultralytic's [own documentation](https://www.ultralytics.com/). In that same folder you will find a subfolder `weights` that contains the 
trained model files. For the minimum those will be `best.pt` and `last.pt`, but depending on how many epochs the model trained for and how you set the `Save period` parameter in the training tab, 
it will contain snapshots that were saved during training.

```
Your project folder
├─ ...
├─ model
│  └─ training_data
│     └─ train
│     └─ test
│     └─ val
│     └─ yolo_config.yaml
│  └─ training
│     └─ results.csv
│     └─ weights
│        └─ last.pt
│        └─ best.pt
```

## Analysis
When new videos are analyzed OCTRON creates a new folder `octron_predictions` within the folder of the .mp4 files that are currently being analyzed. 
Within the predictions folder, subfolders for every video file are created that contain the video file name, followed by the tracker name (e.g. `videoname_HybridSort`). 
These are the actual prediction result folders and you can (after successful completion of analysis in OCTRON) drag and drop these on napari's main window to view them in napari again. A successfully completed prediction folder will have `.csv` files in it, while one that is currently processing only contains `.zarr` subfolders.

Each prediction result folder contains .csv files that contain a basic list of positions and features extracted for every detected label across frames, and predicted masks in zarr archives in `predictons.zarr`. 
Information about the actual prediction itself and parameters it was run with are saved in `predictions_metadata.json`. 
The output data is split by track ID since there can be multiple detections per label in OCTRON.

```
video folder
├─ video_file_1.mp4
├─ video_file_2.mp4
├─ video_file_3.mp4
├─ ...
├─ predictions
│   └─ video_file_1_tracker
│      └─ label_track_id_1.csv
│      └─ label_track_id_2.csv
│      └─ label_track_id_3.csv
│      └─ prediction_metadata.json
│      └─ predictions.zarr
│         └─ track_id_1_masks
│         └─ track_id_2_masks
│         └─ track_id_3_masks
│   └─ video_file_2_tracker
│      └─ ...
│   └─ video_file_3_tracker
│      └─ ...
```

If you want to learn more about how to load OCTRON prediction results programmatically in python, check out the section on [how to access output data](access-data.md). We created a reader class to make it easy for you to load and play with those data and we recommend using that one. However, if you wanted to, you could load for example the .csv files yourself using [pandas](https://pandas.pydata.org/docs/reference/api/pandas.read_csv.html) or other libraries. OCTRON uses the [zarr > 3.0](https://zarr.readthedocs.io/en/stable/index.html) python libary for reading and writing annotation, frame and prediction mask data. 


##### Explanation of .csv data 

The .csv files contain four (primary) index columns:

- `frame_counter`: a running counter from 0 to last analyzed frame. *This does not match the actual frame index in the original video if you skipped frames during analysis!*
- `frame_idx`: the actual frame number (index) in the video file. 
- `track_id`: YOLO creates track_ids, which are unique. There can be multiple track_ids per label (if multiple subjects per label were tracked). 
- `label`: label name, matching your original choices during annotation.

The data columns are:

- `confidence`: how confident was the model about this region in this frame? Can be used to filter out bad segmentation results later on. 
- `pos_x`, `pos_y`: centroid of mask in frame coordinate system if "Detailed" was clicked in analysis tab, otherwise bounding box center. 
- `bbox_area`: Area of bounding box 
- `bbox_x_min`: Bounding box minimum x coordinate
- `bbox_x_max`: Bounding box maximum x coordinate
- `bbox_y_min`: Bounding box minimum y coordinate
- `bbox_y_max`: Bounding box maximum y coordinate

If "Detailed" was clicked in analysis tab you will get additional columns in the csv file:
- `area`: total number of pixels in mask.
- `eccentricity`: from [scikit-image](https://scikit-image.org/docs/stable/api/skimage.measure.html): Eccentricity of the ellipse that has the same second-moments as the region. The eccentricity is the ratio of the focal distance (distance between focal points) over the major axis length. The value is in the interval [0, 1). When it is 0, the ellipse becomes a circle.
- `solidity`: from [scikit-image](https://scikit-image.org/docs/stable/api/skimage.measure.html): Ratio of pixels in the region to pixels of the convex hull image.
- `orientation`: from [scikit-image](https://scikit-image.org/docs/stable/api/skimage.measure.html): Angle between the 0th axis (rows) and the major axis of the ellipse that has the same second moments as the region, ranging from -pi/2 to pi/2 counter-clockwise.


```csv title="Example .csv excerpt"
video_name: my_cool_video_of_a_worm.mp4
frame_count: 420
frame_count_analyzed: 420
video_height: 1000
video_width: 1000
created_at: 2025-10-04 09:06:31.546592

frame_counter,frame_idx,track_id,label,confidence,pos_x,pos_y,bbox_area,bbox_x_min,bbox_x_max,bbox_y_min,bbox_y_max,area,eccentricity,solidity,orientation
0,0,1,worm,0.91695845,319.9365778200857,292.5203474535935,16876.645,265.0909,364.5334,190.18854,359.90118,8404.0,0.9178007984069679,0.8893121693121693,-0.45843770544723256
1,1,1,worm,0.91518414,313.97980943738656,304.4472549909256,18039.29,258.3096,360.2941,202.73657,379.61923,8816.0,0.9287892159764167,0.8692565568921318,-0.5209959253546441
2,2,1,worm,0.91567045,308.8360366713681,318.38763516690176,17867.014,251.09196,357.89276,217.1332,384.4261,8508.0,0.9243025571818851,0.9110183103115965,-0.526705837700152
3,3,1,worm,0.90073955,302.5342702453634,331.1230273010022,18212.67,244.86223,350.6416,227.8413,400.0173,8681.0,0.9274125685818969,0.8915477046318168,-0.5571711039155764
4,4,1,worm,0.915146,298.6263222632226,343.29384993849936,16546.568,238.14746,344.79773,248.87978,404.0277,8130.0,0.9123922490246262,0.9063545150501672,-0.5434600903882498
5,5,1,worm,0.915768,292.0182905786265,355.8093868629932,17815.31,232.9051,342.2042,258.8585,421.85446,8693.0,0.9204012448318362,0.8599267979028589,-0.605617361365298
6,6,1,worm,0.92256516,285.9585199151544,368.61878387933064,18140.51,223.81511,337.50482,271.95493,431.5165,8486.0,0.9198769041829229,0.8919487071683835,-0.6122092126370358
7,7,1,worm,0.9252587,279.3139147009769,380.8497736478437,17971.914,216.84714,332.79538,285.53268,440.53214,8394.0,0.9118294299207745,0.8699347082599234,-0.6703245262237776
8,8,1,worm,0.9157298,272.24632178452777,392.7733744660655,18048.926,207.68143,327.19724,300.30136,451.31842,8428.0,0.9111193683524245,0.8660980372006988,-0.6976541840294108
9,9,1,worm,0.91899484,265.83014354066984,405.87523923444974,17710.676,200.23694,320.2725,315.80124,463.3465,8360.0,0.8996716238646716,0.877782444351113,-0.7017803083673717
10,10,1,worm,0.9200833,256.42858913959833,419.76803868088274,16941.69,189.33072,314.85196,336.50397,471.47467,8066.0,0.899845033040744,0.8591819343843204,-0.8129853986054455
11,11,1,worm,0.9144628,237.1053937573462,447.87488572547994,12945.019,169.37561,296.0545,389.3012,491.48886,7657.0,0.8756011684103597,0.929473173100267,-0.9911368715276544
12,12,1,worm,0.91048104,231.09452987143936,457.201159566423,14109.16,161.46072,287.42816,393.36517,505.37158,7934.0,0.8517008100127463,0.9074688322086241,-0.928434608972939
13,13,1,worm,0.9047041,220.54919085538145,467.89725147701,13775.395,150.33623,277.6463,405.62286,513.82635,7786.0,0.8572078674568909,0.9038774088692826,-0.9896874861291551
14,14,1,worm,0.9120327,216.02685219089466,473.4855364335408,14255.165,141.52246,277.48285,412.4609,517.30884,8193.0,0.8802462457878328,0.914193260432939,-1.0712996972561315
15,15,1,worm,0.89594054,208.0970363288719,478.71438814531547,15825.623,130.25069,273.0127,415.57236,526.42554,8368.0,0.9046674306318826,0.9059218360939699,-1.0674198457149662
16,16,1,worm,0.89766216,201.20757413361915,485.64618316065264,16351.414,122.52483,270.83633,420.90582,531.1563,8397.0,0.9116038219562306,0.8933929141398022,-1.1340528948900301
17,17,1,worm,0.8805878,193.74640522875816,489.79001782531196,15640.947,114.77721,269.16022,428.24365,529.5563,8415.0,0.9204615938100952,0.8869097807757167,-1.221151943366744
18,18,1,worm,0.8930524,185.90940435144455,495.01200808465103,15823.722,105.01762,269.54987,439.55527,535.72925,8411.0,0.9247136247671065,0.9088060507833603,-1.2659752898018573
19,19,1,worm,0.8980207,179.17675573421613,497.7461574840388,14893.042,98.84409,268.26993,448.30237,536.2054,8458.0,0.9249512229854449,0.896069498887594,-1.3103173210041945
...
```
