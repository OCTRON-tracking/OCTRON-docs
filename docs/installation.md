# Installation

!!! info "Important Update Information"
    Just a heads up that there will be changes accumulating in the repo in these days. 
    So the first thing that you should do before playing with OCTRON (after you installed it successfully once, see below), should be:

    1.  Pull latest changes from main (in the GitHub Desktop app for example).
    2.  In your terminal, browse to your cloned repository folder on disk.
    3.  Run `pip install . -U` to install the new code and update existing packages.

    If you ever mess up completely, do not despair! You can trash everything with:

    -   `conda deactivate` and then
    -   `conda env remove --name octron --yes`.

    Then start over by recreating the environment using the *.yaml* file again (see steps below).


1. Make sure **ffmpeg** is installed on the system. Some packages rely on it.
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

2. Download miniconda. <br>
    - Open your web browser and go to the official [Miniconda download page](https://www.anaconda.com/download/success). 
    - Download and execute the Miniconda Installer for your operating system (Windows, MacOS, or Linux). 
    - Restart your terminal.

3. Clone the [OCTRON-GUI repository](https://github.com/horsto/OCTRON-GUI) and in a terminal/command prompt browse to the folder that you cloned it to (e.g., `cd "YOUR/CLONED/FOLDER"`)

4. Create a new Conda environment called "octron" with python version 3.11 by entering:
    ```sh
    conda env create -f environment.yaml
    ```

    !!! important "CUDA Users"
        **If you have a CUDA compatible graphics card in your computer, do *instead***:

        ```sh
        conda env create -f environment_cuda.yaml
        ```

        This will install the right PyTorch version automatically on Windows systems.

5. Activate the new environment:
    ```sh
    conda activate octron
    ```
6. Check the accessibility of GPU resources on your computer:
    ```sh
    python test_gpu.py
    ```
    This should show your graphics card, if it is correctly installed and accessible by PyTorch. If this fails, you should correct this first, since OCTRON will not engage your GPU otherwise (and thus be much slower).
