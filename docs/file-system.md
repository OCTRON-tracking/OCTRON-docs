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
You can use the training data for multiple training runs (if you unticked `Overwrite` in the GUI). 

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
When new videos are analyzed OCTRON creates a new folder `predictions` within the folder of the .mp4 files that are currently being analyzed. 
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
##### Explanation of .csv data 

The .csv files contain four (primary) index columns

- `frame_counter`: a running counter from 0 to last analyzed frame. *This does not match the actual frame index in the original video if you skipped frames during analysis!*
- `frame_idx`: the actual frame number (index) in the video file. 
- `track_id`: YOLO creates track_ids, which are unique. There can be multiple track_ids per label (if multiple subjects per label were tracked). 
- `label`: label name, matching your original choices during annotation.

The data columns are 

- `confidence`: How confident was the model about this region in this frame? Can be used to filter out bad segmentation results later on. 
- `pos_x`, `pos_y`: centroid of mask in frame coordinate system
- `area`: total number of pixels in mask 
- `eccentricity`: from [scikit-image](https://scikit-image.org/docs/stable/api/skimage.measure.html): Eccentricity of the ellipse that has the same second-moments as the region. The eccentricity is the ratio of the focal distance (distance between focal points) over the major axis length. The value is in the interval [0, 1). When it is 0, the ellipse becomes a circle.
- `orientation`: from [scikit-image](https://scikit-image.org/docs/stable/api/skimage.measure.html): Angle between the 0th axis (rows) and the major axis of the ellipse that has the same second moments as the region, ranging from -pi/2 to pi/2 counter-clockwise.
- `solidity`: from [scikit-image](https://scikit-image.org/docs/stable/api/skimage.measure.html): Ratio of pixels in the region to pixels of the convex hull image.
- `mask_l_mean`,`mask_a_mean`,`mask_b_mean`: CIE LAB space average values of masked frame. `l` gives you brightness, whereas `a` and `b` give you color information.
- `frame_l_mean`,`frame_a_mean`,`frame_b_mean`: CIE LAB space average values of whole frame (not masked).


```csv title="Example .csv excerpt"
video_name: bbf6e863928e0a16_segment_38.86s-56.31s.mp4
frame_count: 349
frame_count_analyzed: 22
video_height: 1544
video_width: 2064
created_at: 2025-06-06 11:04:05.085877
frame_counter,frame_idx,track_id,label,confidence,pos_x,pos_y,area,eccentricity,orientation,solidity,mask_l_mean,mask_a_mean,mask_b_mean,frame_l_mean,frame_a_mean,frame_b_mean
0,0,1,mantle,0.9549847,1347.9634156775983,814.3706225324223,81811.0,0.7405340653261191,-0.8526591252359514,0.9823960997634401,26.676253804500618,3.5404163254330103,11.436420530246544,47.04124838082902,1.2360588123067036,-0.9179883620516528
1,16,1,mantle,0.9433561,1370.8768908040893,790.8794687424968,79133.0,0.7387458817319026,-0.8442717817653299,0.9751928622482932,27.199347933226342,3.4205830690104,11.733094916153817,47.06251161033458,1.2213450666746997,-0.9007319531469655
2,32,1,mantle,0.94711256,1391.5591196019695,760.8456089376856,77783.0,0.7279324344046391,-0.8367155979800265,0.9738456530448718,27.001517040998674,3.4671072085159995,11.715631950426186,47.05470946549785,1.228286791581315,-0.9113899264971683
3,48,1,mantle,0.95881116,1411.6508743789366,723.7351173010967,76683.0,0.7334769439452283,-0.8565298743266302,0.9752012513830072,27.02511638824772,3.3766414981156188,11.750048902625092,47.04806772653332,1.2312985751295338,-0.9139012104872073
4,64,1,mantle,0.95045257,1434.837653478854,676.5479549023141,77698.0,0.7325826554622429,-0.8333463294677883,0.9745139846983569,27.871515354320575,3.132294267548714,11.9704239491364,47.04043848154396,1.2314790060047396,-0.894443858697835
5,80,1,mantle,0.9490872,1436.9215640607613,645.307067256683,77286.0,0.7330306292246384,-0.7715441592172607,0.9678049513505391,27.547887068809356,3.1808736381750897,11.982247755091478,47.00310843173877,1.2424953307627424,-0.8939706591155561
6,96,1,mantle,0.9530784,1431.4939176201963,593.7592848204956,77519.0,0.7170495922779244,-0.7284046882360108,0.9672826643041639,27.156490666804267,3.2903159225480203,11.611720997432887,46.99171210386793,1.2551706781941598,-0.9098532830260674
7,112,1,mantle,0.9566795,1419.9704924918617,549.3996639714376,76184.0,0.7034603622051787,-0.724098294875588,0.9699037531191119,26.76039588365011,3.2726294235009976,11.24272813189121,47.00908492991123,1.2608051421858055,-0.9338920100815359
8,128,1,mantle,0.9537202,1413.5321253582324,502.06567704275017,75719.0,0.6956896804050031,-0.6661989763287154,0.9699605451936872,27.160686221423948,3.173483537817457,11.06131882354495,47.05788002821625,1.258206623790015,-0.9512861740370325
9,144,1,mantle,0.95769113,1420.1058638175055,453.75624577987844,75531.0,0.7005498352491896,-0.731067532286296,0.9776842922788169,26.29893686036197,3.2555109822457005,10.66129139029008,47.04174354590914,1.2604066252962205,-0.9746728395991485
10,160,1,mantle,0.95733744,1413.9914485445183,412.2876811014168,74958.0,0.7085000554327338,-0.6690426727760964,0.9700663897192996,26.525841137703782,3.0553643373622563,10.468235545238667,47.

...
```
