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
- `pos_x`, `pos_y`: centroid of mask in frame coordinate system.
- `area`: total number of pixels in mask.
- `eccentricity`: from [scikit-image](https://scikit-image.org/docs/stable/api/skimage.measure.html): Eccentricity of the ellipse that has the same second-moments as the region. The eccentricity is the ratio of the focal distance (distance between focal points) over the major axis length. The value is in the interval [0, 1). When it is 0, the ellipse becomes a circle.
- `orientation`: from [scikit-image](https://scikit-image.org/docs/stable/api/skimage.measure.html): Angle between the 0th axis (rows) and the major axis of the ellipse that has the same second moments as the region, ranging from -pi/2 to pi/2 counter-clockwise.


```csv title="Example .csv excerpt"
video_name: 20221205_3PL_Ctrl1.mp4
frame_count: 6574
frame_count_analyzed: 6574
video_height: 2048
video_width: 2048
created_at: 2025-09-27 14:52:52.363542
frame_counter,frame_idx,track_id,label,confidence,pos_x,pos_y,area,eccentricity,orientation
38,38,1,mantle,0.7551425,2027.3234575442884,1694.942883323152,3274.0,0.8313205865350272,0.07209660649053543
39,39,1,mantle,0.5012319,1960.1668184301584,1578.5427062465853,5491.0,0.7220735697073672,0.7093078001056766
40,40,1,mantle,0.686898,1730.276359472272,1417.3980688142617,1369.0,0.7258227318232526,0.04543474711027081
42,42,1,mantle,0.5193799,1478.4483306836248,1279.4908585055643,7548.0,0.7884716377340559,0.23025266168127095
44,44,1,mantle,0.8494755,1382.7353560893384,912.8278550358197,4746.0,0.4469251323499112,-1.4728752817769104
45,45,1,mantle,0.8324638,1399.2999318336742,916.0902067711884,4401.0,0.31346838725001264,1.5641413260528338
46,46,1,mantle,0.6398426,1611.76725,941.37325,4000.0,0.7318063263015527,1.1615708302162984
47,47,1,mantle,0.8172138,1479.1186071817192,915.7098295248459,5514.0,0.3602943568492482,0.10137142162631338
48,48,1,mantle,0.75389636,1405.7252349963846,860.8732224632441,4149.0,0.6840367469384251,0.4756388254410816
49,49,1,mantle,0.7728026,1355.533661417323,812.280905511811,5080.0,0.4347150882617974,0.27426227517244695
50,50,1,mantle,0.82223874,1300.270232371795,783.3778044871794,4992.0,0.670370877670398,1.2359856002310008
56,56,1,mantle,0.82897145,1219.86590389016,637.9162471395881,4370.0,0.39596783226218746,-0.5059858313300662
57,57,1,mantle,0.8612444,1226.0353170731707,600.4394146341464,5125.0,0.4495708402040449,-1.188856962899363
...
```
