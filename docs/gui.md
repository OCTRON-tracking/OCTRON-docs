# Using the GUI

## Opening the GUI
1. Activate the *octron* environment:
    ```sh
    conda activate octron
    ```
2. Start the GUI:
    ```sh
    octron-gui
    ```
    ... and enjoy! 
    On first start this will take a long time to load, since all available models are also downloaded. Subsequent startups will be much quicker. 

## The GUI explained
In the GUI there are four main panels:

- **Project navigation:** this is where you navigate through the different steps of the process, from [opening your project](create-project.md) (*Manage project* tab), [annotate videos](annotating.md) (*Generate annotation data* tab), [train a model](training.md) on your annotated videos (*Train model* tab), and use your trained model to [analyze new videos](analysing.md) (*Analyze (new) videos* tab).
- **Video panel:** videos you are annotating will show up here.
- **Layer list:** the annotations you create will be shown as individual layers in this section.
- **Layer controls:** the tools to create annotations will be found here once you've selected a specific layer in the *Layer list*.

![an annotated image of the GUI upon startup](assets/annotated_images/gui.png)