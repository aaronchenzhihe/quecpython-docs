# VSCode Plugin User Guide

To facilitate developers using [VSCode](https://code.visualstudio.com/) for [QuecPython](https://developer.quectel.com/en/) development, the QuecPython team has introduced a VSCode plugin called [`QuecPython`](https://marketplace.visualstudio.com/items?itemName=Quectel.qpy-ide).

![overview.png](../../media/dev-tools/VSCode-Plugin-quecpython/overview.png)

The plugin currently supports the following features:

- Firmware flashing
- REPL command interaction
- File transfer
- Filesystem directory tree
- Run specific script files

It currently all QuecPython series modules.

## Plugin Installation

1. **Click on the `Extensions` icon in the primary sidebar ![extensions-icon.png](../../media/dev-tools/VSCode-Plugin-quecpython/extensions-icon.png).**
2. **Type `QuecPython` in the plugin search box.**
3. **Find the QuecPython plugin in the search list, click the `install` button to complete the plugin installation.**

![installation](../../media/dev-tools/VSCode-Plugin-quecpython/installation.png)

## Plugin Usage Guide

> Before you start, make sure you have downloaded the appropriate QuecPython firmware package for your module. [Click here to view how to download firmware packages](../../media/dev-tools/VSCode-Plugin-quecpython/download-fw-pkg.png).

The following steps are demonstrated based on the U-235 development board

### Firmware Flashing

1. **Access the Firmware Flashing Interface on the Primary Sidebar**

    There are two methods:

    - Click on the QuecPython plugin icon on the primary sidebar ![quecpython-plugin-icon.png](../../media/dev-tools/VSCode-Plugin-quecpython/quecpython-plugin-icon.png).

    - Use the shortcut `Ctrl+Shift+P` to open the command palette, type `> QuecPython` in it, and select `QuecPython: Focus on Firmware Manager View` from the options, as shown below:

    ![cmd-palette.png](../../media/dev-tools/VSCode-Plugin-quecpython/cmd-palette.png)
        
    
    The firmware flashing interface on the primary sidebar is shown in the following image:

    ![fw-flash-ui.png](../../media/dev-tools/VSCode-Plugin-quecpython/fw-flash-ui.png)

2. **Select Firmware**

    Click the `Select Firmware` button ![select-fw-button.png](../../media/dev-tools/VSCode-Plugin-quecpython/select-fw-button.png) on the firmware flashing interface, and choose the firmware package for the U-235 development board module.

    The path to the firmware package will be displayed on the interface:

    ![selected-fw-path.png](../../media/dev-tools/VSCode-Plugin-quecpython/selected-fw-path.png)

3. **Flash Firmware**

    i. Click the `Flash` button ![fw-flash.png](../../media/dev-tools/VSCode-Plugin-quecpython/fw-flash.png) on the firmware flashing interface.

    ii. In the pop-up confirmation window, click the `Yes` button ![flash-confirm-yes.png](../../media/dev-tools/VSCode-Plugin-quecpython/flash-confirm-yes.png).

    ![flash-confirm.png](../../media/dev-tools/VSCode-Plugin-quecpython/flash-confirm.png)

    Now you can see the firmware flashing progress bar in the lower right corner of the VSCode interface:

    ![flash-progress.png](../../media/dev-tools/VSCode-Plugin-quecpython/flash-progress.png)

    When the progress bar shows `Flashing Firmware: 8/8 100%`, the firmware flashing is complete, and the module will restart.

### REPL Command Interaction

> Before starting the REPL command interaction for QuecPython, it is  necessary to flash the firmware that supports QuecPython functionality.

1. **Find the command to connect to the terminal in the command palette**

    Use the shortcut `Ctrl+Shift+P` to open the command palette. Enter `> QuecPython` in the palette and select `QuecPython: Connect to COM Port` from the options, as shown below:

    ![connect-to-com-port-cmd.png](../../media/dev-tools/VSCode-Plugin-quecpython/connect-to-com-port-cmd.png)

2. **Connect to Terminal**

    Follow the instructions in the image below:

    ![connect-to-com-port-flow.gif](../../media/dev-tools/VSCode-Plugin-quecpython/connect-to-com-port-flow.gif)

    Once the operation is completed, the module will be connected to the terminal in VSCode.

    At the same time, the terminal window will pop up with the content as shown in the image below:

    ![repl-terminal](../../media/dev-tools/VSCode-Plugin-quecpython/repl-terminal.png)

3. **REPL Command Interaction**

    After pressing Enter in the terminal, you will see the QuecPython command prompt `>>>`.

    Enter the following code in the terminal:

    ```python
    print("Hello, QuecPython!")
    ```

    After pressing Enter, you can see the execution result as shown in the following image:

    ![repl-print-hello.png](../../media/dev-tools/VSCode-Plugin-quecpython/repl-print-hello.png)

    Now, we have completed the simplest QuecPython REPL interaction.

<!-- Needs to be updated, for now it's hidden

### Code Completion

1. **Create a new Python script file `test.py` and open it in VSCode.**

2. **In the editing page of the `test.py` file, enter the following code:****

    ```python
    import utime
    
    for i in range(100):
        print("Hello, QuecPython!")
        utime.sleep(1)
    ```

    This code snippet is used to print the string `"Hello, QuecPython!"` in a loop every 1 second.

    As shown in the figure below, when entering the unique `utime` module of QuecPython, the VSCode interface will prompt for the methods it contains.

    Press the up or down arrow keys to move the cursor to the desired interface, and press the `Tab` or `Enter` key to complete the code in the `test.py` file.

    ![code-auto-completed.gif](../../media/dev-tools/VSCode-Plugin-quecpython/code-auto-completed.gif)
-->

### File Transfer

The edited script file `test.py` needs to be imported into the module's file system to run.

1. **In the editor page of the file to be transferred (i.e., `test.py`), right-click the mouse.**
2. **Click on the `QPY: Download File` command to start file transfer.**

![download-file.png](../../media/dev-tools/VSCode-Plugin-quecpython/download-file.png)

During the file transfer process, a pop-up window appears in the bottom right corner of the VSCode interface, as shown below:

![downloading-file.png](../../media/dev-tools/VSCode-Plugin-quecpython/downloading-file.png)

### File System Directory Tree

1. **After the file transfer is complete, click on the `Explorer` button ![explorer-icon.png](../../media/dev-tools/VSCode-Plugin-quecpython/explorer-icon.png) on the primary sidebar.**

2. **Once the `Explorer` interface is open, look at the bottom and find `QPY: FILE SYSTEM (USR)`. It shows the directory tree of the module's file system `/usr`.**

    The imported file `test.py` is shown in the following image:

![directory-list.png](../../media/dev-tools/VSCode-Plugin-quecpython/directory-list.png)

### Run Script File

1. Select the script file `test.py` in the directory tree.
2. Right-click on the script file `test.py` and choose `Run Script` to execute the script file.

![run-script.png](../../media/dev-tools/VSCode-Plugin-quecpython/run-script.png)

In the terminal window, you can see the script periodically outputting the string `"Hello, QuecPython!"`.

![test-script-running.png](../../media/dev-tools/VSCode-Plugin-quecpython/test-script-running.png)

### Package Manager

1. From QuecPython menu -> Quick Access -> Quectel -> Projects + Components

<p>
  <img src="https://raw.githubusercontent.com/QuecPython/vscode-extension-issues/refs/heads/main/images/menu.png" alt="Select Menu">
</p>

2. List of Projects + Components will show in the panel
<p>
  <img src="https://raw.githubusercontent.com/QuecPython/vscode-extension-issues/refs/heads/main/images/main-panel.png" alt="Main Panel">
</p>

3. From the panel, you can view usig view button for a project of component, which will show readme + submodules for project or component.
<p>
  <img src="https://raw.githubusercontent.com/QuecPython/vscode-extension-issues/refs/heads/main/images/view-button.png" alt="Ciew Button">
</p>
<p>
  <img src="https://raw.githubusercontent.com/QuecPython/vscode-extension-issues/refs/heads/main/images/readme-info.png" alt="View Info">
</p>

<b>Note:</b> View button will always show the info of project, component or submodule.

4. From Main Panel, Import button will clone the project to your machine.

  - click import
<p>
  <img src="https://raw.githubusercontent.com/QuecPython/vscode-extension-issues/refs/heads/main/images/import-button.png" alt="Import Button">
</p>

  - choose where to save the project
  - please wait while the extension clone the project
  - when done, the extension will open the new project

<p>
  <img src="https://raw.githubusercontent.com/QuecPython/vscode-extension-issues/refs/heads/main/images/clone-done.png" alt="Clone Done">
</p>

5. From Main Panel, Add to project button will clone the submodule to your current open project.

<p>
  <img src="https://raw.githubusercontent.com/QuecPython/vscode-extension-issues/refs/heads/main/images/add-to-project.png" alt="Add to Project">
</p>

<b>Note 1:</b> Add to project will work only when there's an open project already.
If there's no project open, the button will be disabled.

<b>Note 2:</b> Add to project and these submodules can be used and cloned with any of client QuecPython projects or with our projects from our github.

6. Banner buttons, and search fields are give a better experience while using the extension

<p>
  <img src="https://raw.githubusercontent.com/QuecPython/vscode-extension-issues/refs/heads/main/images/banner-buttons.png" alt="Banners Buttons">
</p>

  - Show all / Hide All: Show or Hide all the items from projects + components list.
  - New Project: Create a new project.
  - Home: Redirect to main page (Not active when user is alraedy at main page).
  - Back: go back to previouse page (active only when previouse page is avilable).
  - Search Projects / Components: search the list of Projects or Components.
  - Show More:show the list of Projects or Components.