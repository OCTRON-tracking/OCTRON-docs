# The file system 

During annotation, training and analysis, OCTRON creates various subfolders and files in the main project folder you selected.

## Annotation 
For every video you are annotating, OCTRON creates a subfolder that contains 
- `label_name masks.zarr` archives that contain the annotation mask data for every label and annotation you create
- `object_organizer.json` file that contains info on the label and various layers in your annotation project
- `video data.zarr` archive that contains (compressed) video frame data for every annotated frame 

The folder names are the abbreviated hashes of the video file that is loaded and OCTRON makes sure that whenever you re-load an annotation project from within the GUI, the hash of the loaded video and the one of the annotation subfolder match. 
Meaning, when you modify the video file in any way, OCTRON will throw an error, alerting you that the video file and loaded annotations do not match. There is also a `video_info.txt` that can be deleted without consequences and contains some basic info about the annotated video file itself. 

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

```json title="object_organizer.json example"
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
You can use the training data for multiple training runs (if `overwrite==False`). 

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
When new videos are analyzed, OCTRON creates a new folder `predictions` within the folder of the .mp4 files that are currently being analyzed. 
Within the predictions folder, subfolders for every video file are created that contain the video file name, followed by the tracker name (e.g. bytetrack). 
These are the actual prediction result folders, and those you can (after successful completion of analysis in OCTRON) drag and drop on OCTRON's main window to view them in OCTRON again. 

Each prediction result folder contains .csv files that contain a basic list of frames and position / features extracted for every detected label and zarr archhives alongside those csv files in `predictons.zarr`. 
Information about the actual prediction itself and parameters it was run with are saved in `predictions_metadata.json`. 
The label data is split by track ID since there can be multiple detections per label in OCTRON.

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

```csv title="Example tracking .csv excerpt"
video_name: Cteno16-2_MAH00984_cropped.mp4
frame_count: 3611
frame_count_analyzed: 3611
video_height: 720
video_width: 720
created_at: 2025-05-28 15:00:28.987197
frame_counter,frame_idx,track_id,label,confidence,pos_x,pos_y,area,eccentricity,orientation,solidity
2,2,3,inner,0.819916,218.9506172839506,597.8935185185185,648.0,0.9014825846802565,0.47584488052997526,0.9515418502202643
3,3,3,inner,0.8158086,218.83664122137404,597.7435114503817,655.0,0.9042791865211839,0.48847283909451655,0.9576023391812866
4,4,3,inner,0.8178996,218.7492211838006,597.5155763239875,642.0,0.9010777897427291,0.43718313189681607,0.9427312775330396
5,5,3,inner,0.8235048,219.1216848673947,597.5397815912637,641.0,0.900572515666512,0.4697154319141782,0.9510385756676558
6,6,3,inner,0.842097,219.00439882697947,597.407624633431,682.0,0.8960923467472833,0.4407903519156258,0.9445983379501385
7,7,3,inner,0.8389323,218.7737556561086,597.3122171945702,663.0,0.8898350709958005,0.4286921784414749,0.9417613636363636
8,8,3,inner,0.84204984,219.30917159763314,597.5784023668639,676.0,0.8896692313332966,0.4345940147711663,0.952112676056338
9,9,3,inner,0.84247273,219.27083333333334,597.9136904761905,672.0,0.8801244693405843,0.45840151444640026,0.9478138222849083
10,10,3,inner,0.8413821,219.02086230876216,596.4561891515995,719.0,0.8986274192278796,0.4069079572487939,0.9548472775564409
11,11,3,inner,0.8401129,218.85576923076923,595.8173076923077,728.0,0.9036298092176612,0.4176986289327898,0.9578947368421052
12,12,3,inner,0.844465,218.87262872628727,596.1151761517615,738.0,0.8959884611768869,0.44701113249833097,0.9659685863874345
13,13,3,inner,0.8539265,219.4295485636115,597.5075239398085,731.0,0.8725362387234515,0.47589255622996474,0.9555555555555556

...
```
