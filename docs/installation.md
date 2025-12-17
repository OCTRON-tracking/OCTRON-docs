# Installation

1. Install **ffmpeg** on your system. Some packages rely on it and we generally recommend ffmpeg because it is very useful for dealing with video data.
    - Open a terminal window/command prompt

        ??? note "Opening a terminal window/command prompt"
            === "Windows"
                1. Click the Windows key + R
                2. Type `cmd` and click *OK* to open a terminal window.
            === "MacOS"
                1. Click the Launchpad icon.
                2. Type `Terminal` in the search field and click on *Terminal* to open it.
            === "Linux"
                You know what to do :)

    - Type `ffmpeg -version` and click *Enter* <br>
        *If this command fails for some reason, make sure you install ffmpeg first*

        ??? note "Installing ffmpeg"
            === "Windows"
                1. Open the [ffmpeg download page](https://ffmpeg.org/download.html).
                2. Under the "Get packages & executable files" section, click on the Windows logo.
                3. You will be redirected to a page with various builds. Click on the link for the "Windows builds from gyan.dev".
                4. Scroll down to the "Release builds" section and download the `ffmpeg-release-essentials.zip` file.
                5. Extract the downloaded zip file and copy the `bin` subfolder to, for example, `C:\ffmpeg\bin`.
                6. Open the Start menu in Windows, search for "Environment Variables", and select "Edit the system environment variables".
                7. In the System Properties window, click on the "Environment Variables..." button.
                8. In the Environment Variables window, find the "Path" variable under the "System variables" section and select it. Click "Edit...".
                9. In the Edit Environment Variable window, click "New" and paste the path to the `bin` directory (e.g., `C:\ffmpeg\bin`). Click "OK" to close all windows.
                10. Verify that the installation is complete: Open a new (!) command prompt (cmd) and type `ffmpeg -version` and press `Enter`.
                11. If ffmpeg is installed correctly, you should see the version information for ffmpeg.
            === "MacOS"
                On MacOS you can use [homebrew](https://formulae.brew.sh/formula/ffmpeg) and type `brew install ffmpeg` into your terminal
            === "Linux"
                Instructions depend on your system, but please do not hesitate to reach out if you run into issues. 

2. Install miniconda. <br>
   Conda is a package manager and we recommend using miniconda to create an environment that OCTRON runs in.  

    - Open your web browser and go to the official [Miniconda download page](https://www.anaconda.com/download/success). 
    - Download and execute the Miniconda Installer for your operating system (Windows, MacOS, or Linux). During installation make sure you click on the check box **"Add to PATH"** during installation. This way you will be able to execute `conda` commands in your terminal. You can also click "register python as default python". 
    - Restart your terminal.

3. Create a new Conda environment called "octron" with python version 3.11 by entering:
    ```
    conda create -n octron python=3.11
    ```

4. Activate the new environment:
    ```sh
    conda activate octron
    ```
5. You can now install OCTRON into your new conda environment. If you have an NVIDIA (*CUDA compatible*) graphics card in your machine, do 
    ```
    pip install --extra-index-url https://download.pytorch.org/whl/cu126 git+https://github.com/OCTRON-tracking/OCTRON-GUI.git
    ```
    otherwise it suffices to do 
    ```
    pip install git+https://github.com/OCTRON-tracking/OCTRON-GUI.git
    ```
    
    !!! question "How do I update OCTRON?"
        OCTRON is undergoing a lot of development. So if you haven't used in a while or just want to make sure you run the latest version, you should update it.<br>To update OCTRON make sure you conda activated your environment (step 4) and just run the above `pip install` command (step 5) again. In most cases that is all you need. If you want to update all underlying libaries to the most up-to-date versions, append a `-U` at the end of the command. In some rare cases updates might not work as expected, and you can then try to add an additional `--force-reinstall` at the end of the command to give it all a fresh start.

6. Check the accessibility of GPU resources on your computer:
    ```sh
    octron-test-gpu
    ```
    This should show your graphics card, if it is correctly installed and accessible by PyTorch. 
    If your are using OCTRON on a modern mac this should show: 
    ```
    CUDA GPU is not available.
    MPS (Metal Performance Shaders) GPU is available.
    MPS GPU: Apple Silicon GPU
    ```
    and under windows or linux using a NVIDIA GPU this should show something similar to: 
    ```
    CUDA GPU is available.
    Number of CUDA GPUs: 1
    CUDA GPU 0: NVIDIA GeForce RTX 3070 Ti
    MPS GPU is not available.
    ```

    If this fails, you should correct this first, since OCTRON will not engage your GPU otherwise (and thus be much slower). A common issue is that CUDA dependencies were not correctly installed. Check [this issue](https://github.com/horsto/OCTRON-GUI/issues/11#issuecomment-2954125899) for a potentially quick fix.
