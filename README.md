# Jetraw Image Compression
This repository hosts releases of the Jetraw image compression software. For more information about Jetraw, please visit https://jetraw.com/.

## Using Jetraw
To achieve its high compression ratio, Jetraw relies on knowledge of the exact statistics given by the camera profile. These depend on the camera's unique serial number, as well as the settings that were used to acquire the image. For Jetraw to work, this information must have been embedded within the image. This is done by the instrument manufacturer, who can ensure, at runtime, that the right parameters are used.
‍
**Apply dpcore** to the image, using the **Windows PowerShell**. For example with te command below:
```powershell
dpcore -d prepared -i 20600089_1_23333334_0 .\c1.tif
Hint: type dpcore --help to see the full list of options
Hint: the -d option allows you to give the destination folder. If it does not exist, it will be created.
Hint: if the identifier (-i 20600089_1_23333334_0) is wrong, a list of available identifiers will be printed.
Hint: if you want to directly compress your file, not just prepare it, add the flag -c to the command line
Hint: you can prepare and/or compress entire image folders
```

Good to know: 
dpcore has two functions, it replaces the original camera noise which is random and unpredictable, with noise that is pseudorandom, with the same statistical profile as a the camera. This operation is not bit-for-bit lossless, as the process reduces the SNR by ~1dB. However, no other artefacts or biases are introduced. To give an intuitive parallel, with respect to consumer cameras, is as if you would increase the ISO setting of the camera from e.g. ISO 100 to ISO 115. It is up to the user to decide whether this sligh loss in SNR is acceptable, in exchange for a compression ratio and speed both 5x better than other technologies.

Prepared images can then be compressed and decompressed losslessly with jetraw:
```powershell
jetraw compress -d compressed c1.tif
jetraw decompress -d decompressed c1.p.tif
```

Use your compressed images directly: 
jetraw compressed (.p.tiff) files are standard tiff files, however the image data requires the tiff library to be aware of the installed jetraw codec. Currently, we have **plugins that allow you to read .p.tiff files** directly from **Fiji/ImageJ**, from **Python**, and from any language able to use a C library. Please let us know if you require integration with other software. **Matlab** and **LabView** and **Micro-Manager** are already in our development pipeline.

## Installation
Download the [latest release](https://github.com/Jetraw/Jetraw/releases/latest), or browse [previous releases](https://gtihub.com/Jetraw/Jetraw/releases).
Once downloaded proceed installation using the .msi (Windows), .dmg (macOS) or .tar.gz (Linux) file. 

## Contact
Feel free to use the [issues section](https://github.com/Jetraw/Jetraw/issues) to report bugs or request new features. You can also ask questions and give comments by visiting the [discussions](https://github.com/Jetraw/Jetraw/discussions)
