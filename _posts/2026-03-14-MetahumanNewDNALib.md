---
layout: post
title: MetahumanNewDNALib Invoke
date: 2023-01-07 19:14:00 +8000
categories: [Coding]
tags: [UE5]
comments: true
math: true
image:
    path: ../assets/img/photos/DNA2_Standalone.png
---

## 1. Introduction

Epic's official open-source library [MetaHuman-DNA-Calibration](https://github.com/EpicGames/MetaHuman-DNA-Calibration) only supports old versions of DNA files.

In UE versions later than 5.6, DNA files can only be loaded with the [MetahumanForMaya](https://www.fab.com/listings/9e3bf55e-d4c3-44fc-a3d4-ec4cb772ec29) plugin.

If we want to use DNA files in standalone Python scripts without Maya dependencies, we need to dig into this plugin.
## 2. Analysis

Brief review of the installation of MetahumanForMaya:
1. Download the plugin from [MetahumanForMaya](https://www.fab.com/listings/9e3bf55e-d4c3-44fc-a3d4-ec4cb772ec29)
2. Add `MAYA_MODULE_PATH`  to your system environment variables with the value `C:\Program Files\Autodesk\Maya2025\modules`
3. Unzip the zip file provided in the downloaded plugin to the `C:\Program Files\Autodesk\Maya2025\modules`
4. Refer to [official documentation](https://dev.epicgames.com/documentation/en-us/metahuman/installing-in-maya) for installing the plugin into Maya.

The core scripts lie in the folder:
 `C:\Program Files\Autodesk\Maya2025\modules\MetaHumanForMaya\lib\PyDNA\9.4.7\platform-windows\.sanitizers-off\.json-0`

Since I found that Maya 2025 uses Python 3.11.4, I copied the lib folder from `C:\Program Files\Autodesk\Maya2025\modules\MetaHumanForMaya\lib\PyDNA\9.4.7\platform-windows\.sanitizers-off\.json-0\python-3.11` to a standalone folder for future development.

Directly import the DNA file with:
```python
import os
import sys

ROOT = os.path.dirname(__file__)
LIB  = os.path.join(ROOT, "lib")

sys.path.insert(0, LIB)
os.add_dll_directory(LIB)

import dna
```

The test script throws the error:
```sh
E:\Code\python\PyQT\lib>python test.py
Traceback (most recent call last):
  File "E:\Code\python\PyQT\lib\test.py", line 10, in <module>
    import dna
  File "E:\Code\python\PyQT\lib\dna.py", line 35, in <module>
    import _py3dna9_4_7
ImportError: DLL load failed while importing _py3dna9_4_7: 找不到指定的模块。
```

This means the _py3dna9_4_7 needs DLL dependencies.

So I used the third-party too [Dependencies](https://github.com/lucasg/Dependencies/releases) to detect which DLL files are needed.

![DllDependencies](../assets/img/photos/DNA1.png)

It is clear the `_py3dna9_4_7.pyd` uses `dna9_4_7.dll`, `polyalloc1_3_18.dll`,`statuscode1_2_12.dll`, `trio4_0_21.dll`. Then I copied all the files into the folder, and the `import dna` command worked fine.

The final structure of the DNA lib is shown below:

![DllDependencies](../assets/img/photos/DNA2_Standalone.png)