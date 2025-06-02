# Analyze (new) videos
Once you have a trained model, you can use it to analyze new videos. This is done in the **Analyse (new) videos** tab.
We recommend preparing some excerpts from your test videos (for example, some 60 seconds snippets) that you use initially to test parameters (see below), before you send long videos through analysis. 
OCTRON by default only accepts MP4 transcoded videos. You can use OCTRON to transcode a whole folder of videos of any format to .mp4 (check out [Create Project](create-project.md)).

??? note "Extracting a snippet from an existing .mp4 file"
    To extract a snippet from an existing video you can do<br>
    ```
    ffmpeg -ss 20 -i "your_video_path.mp4" -t 60 -c:v libx264 -preset superfast -crf 23 -an "your_video_path_snippet.mp4" 
    ```
    where<br>
    - `-ss` indicates the start of the snippet in seconds from the start of the video.<br>
    - `-t 60` indicates that you want to extract 60 seconds from the video.<br>
    - `-c:v libx264 -preset superfast -crf 23 -an` specifies the codec (**do not change this!**) and that you do not want any audio in the output.<br>


<img src="../assets/annotated_images/analysis.png"/>

## Add video files
Start by adding the video(s) you want to analyse to the *Add video files* section. Once you added videos you will see them listed in the videos dropdown list. 

## Create predictions from videos
Next, in the *Create predictions from videos* section you indicate how the videos you selected should be analyzed by considering the following options:

- **Choose model...:** choose which of the generated models you want to use from the drop-down menu. You can choose models from different stages of the training, e.g. the model that was saved after x number of epochs, or the best/last one that was saved (recommended).
- **Tracker...:** if you have annotated more than one object for a given label (e.g. *artemia 1*, *artemia 2*), a tracker needs to be used to help the model determine which is which across frames. Pick the one you prefer in this drop-down menu. If you only have one object per label, click the **1 subject** option.

    ??? question "Which tracker should I use?"
        Short answer: it probably doesn't matter which one you choose. `BoT-SORT` seems to give slightly better results, but is computationally much more intense than ByteTrack. If you want to dig into the differences then you can find detailed information from the creators of each tracker here: 

        - **[ByteTrack](https://github.com/FoundationVision/ByteTrack)** 
        - **[BoT-SORT](https://github.com/NirAharon/BoT-SORT)** 

        There is a nice module comparison of various trackers here: https://github.com/mikel-brostrom/boxmot/ - but only ByteTrack and BoT-SORT are integrated currently

- **Videos:** select which video(s) you want to analyze.
- **Polygon sigma:** select smoothing sigma for Gaussian smoothing of the detected region. This is optional and can be left at `0`, but if your regions have very jagged edges, you can increase it to smooth them a bit. 
- **Conf. thres. (confidence threshold, 0-1):** the confidence threshold that should be used to determine which analysed frames to keep. There will likely be frames where the model is more confident that it has identified the right object than others. If the confidence threshold is set to 0.8 it means the model will only keep frames where it is 80% certain that it has correctly identified a given object.
- **View results:** select this option if you want OCTRON to automatically open a new window where you can see the result of the analysis once it is complete (recommended).
- **Overwrite:** select this option if you've previously analysed the selected video and want to replace that analysis.
- **IOU (intersection over union, 0-1):** this threshold determines how much objects can overlap and still be considered separate objects. If this value is zero, then objects that have no overlap will be considered to be the same object, i.e. there is only one object.

Click *Predict*.

OCTRON will now analyze your video(s) and show its progress via the progress bars (if analyzing multiple videos at once, then the first progress bar displays the progress of a single video while the second bar updates with the completion of each video) along with an estimate for when the analysis will finish.

