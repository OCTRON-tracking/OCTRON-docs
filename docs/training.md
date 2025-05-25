# Train model
Once you have a decent number of annotated frames, you are ready to train your model. This is done in the **Train model** tab.

<img src="../assets/annotated_images/training.png"/>

## Generate training data
OCTRON needs to generate data to train the model on, i.e. it takes your annotations and splits them into a training dataset and a testing dataset. This enables it to evaluate how well the training is going by comparing its predictions against the ground truth. First, consider these options:

- **Prune:** select this if there are frames in which it is likely that not all of your objects were annotated despite them all being present. By selecting this option OCTRON will 'prune' the annotated frames so that only those where all labels are present are used. Otherwise you will be counteracting the training (the model will think that if one object isn't annotated in a certain frame, but the other objects are, this means the un-annotated object isn't there).
- **Overwrite:** if you've generated a training dataset before, selecting this option will overwrite the existing one (recommended).

Once you click *Generate*, you can observe the progress in the two progress bars:

- **label:** the progress for a given label.
- **label and split:** the progress of splitting up the annotated data into a training and testing dataset.

## Train
Once the training data has been generated, OCTRON is ready to train your model. There are a few settings to consider:

- **Choose model:** choose which model to use.

    ??? question "Which model should I choose?"
        The larger the model, the more accurate it may be, but the more time and GPU resources it needs too. Recommendation: start with the smallest model and move up from there if necessary.<br>
        
        - **YOLO11m-seg:** the 'medium' model (the smallest one) 
        - **YOLO11l-seg:** the 'large' model
        - **YOLO11x-seg:** the 'extra large' model (the largest one)

- **Img. size:** choose which image size OCTRON should work with.

    ??? question "Which image size should I choose?"
        - **x:** details 
        - **y:** details 

- **Epochs:** decide how many epochs the model should train for. The higher the number, the longer the training will take, but if no significant improvement is detected across x number of epochs then the training will automatically stop.

- **Save period:** decide how often (in number of ephocs) to save the training results.

- **Resume:** if you've previously started training a model but had to abort for some reason, you can continue from where the training stopped by selecting this option *[this feature has not been implemented yet]*.
- **Overwrite:** if you've previously trained a model and want to replace it, select this option.
- **Tensorboard:** select this if you want to follow the training progress live in your browser. Once you click *Train*, a new browser tab should open automatically, but if it does not you can copy the link that appears in your terminal window and open it manually.

When you're happy with your training settings, click *Train*.

While the model is training you can track its progress in the terminal window (and with graphs in your browser if you selected the *Tensorboard* option). Once OCTRON has finished one epoch, it will provide an estimate of how long the total training will take, based on how long the first epoch took to complete and how many epochs you've told it to train for. 