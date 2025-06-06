# Access output data
To access the data that OCTRON has generated (the coordinates of all your labelled objects for each frame, masks, etc), you can either drag and drop the prediction output folder on the napari window and click "Open with OCTRON", or use python directly to access the data. We have compiled [this tutorial notebook](https://github.com/horsto/OCTRON-GUI/blob/main/octron/notebooks/OCTRON_results_loading.ipynb) for that purpose. 

To understand more about what output OCTRON is saving for each analysed video file see [File System - Analysis](file-system.md#analysis) section.

## Access OCTRON prediction results 

```python
# To access any prediction results programmatically, 
# you can use the YOLO_results class.
from octron import YOLO_results

results_dir = 'path_to_folder/predictions/some_output_bytetrack'
yolo_results = YOLO_results(results_dir, verbose=True)

# yolo_results contains quite a bit of information now ... 
print(yolo_results.height, yolo_results.width, yolo_results.num_frames)
# The track_id_label is a dictionary that contains the 
# track id as key and the label as value
print(yolo_results.track_id_label)

# to access tracking data
tracking = yolo_results.get_tracking_data(interpolate=False)
# to access mask data 
masks = yolo_results.get_mask_data()
```
Check out the [tutorial notebook](https://github.com/horsto/OCTRON-GUI/blob/main/octron/notebooks/OCTRON_results_loading.ipynb) for a more comprehensive guide.


## Load OCTRON prediction results into napari
You can either drag and drop the folder onto the napari main window, or trigger the same operation like this: 

```python
# You can access the yolo_octron class directly if you 
# want to use it for your own purposes
# This allows you to plot things in napari like you would when 
# drag-and-dropping the results into napari
# This is just a convenience function to load the results, 
# but should not be used for any serious analysis.
# In fact, YOLO_octron accesses YOLO_results itself to load the results! 
from octron import YOLO_octron
results_dir = 'path_to_folder/predictions/some_output_bytetrack'
yolo_octron = YOLO_octron()

# Loop over all results and add them to napari one by one
for frame_results in yolo_octron.load_predictions(save_dir=results_dir, 
                                                  sigma_tracking_pos=2, 
                                                  open_viewer=True
                                                  ):
    # you can access the tracking_df and features_df here if you want to ...
    # label, trackid, color, tracks, feats, masks = frame_results
    # ... 
    # ... 
    # ... or just continue 
    continue

```



