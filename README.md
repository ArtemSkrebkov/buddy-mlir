# BUDDY MLIR

An MLIR-Based Ideas Landing Project.

## Getting Started

### LLVM/MLIR Dependencies

This project uses LLVM/MLIR as an external library. Please make sure [the dependencies](https://mlir.llvm.org/getting_started/) are available
on your machine.

### Clone and Initialize


```
$ git clone git@github.com:buddy-compiler/buddy-mlir.git
$ cd buddy-mlir
$ git submodule update --init
```

### Build and Test LLVM/MLIR

```
$ cd buddy-mlir
$ mkdir llvm/build
$ cd llvm/build
$ cmake -G Ninja ../llvm -DLLVM_ENABLE_PROJECTS="mlir" \
-DLLVM_TARGETS_TO_BUILD="host" \
-DLLVM_ENABLE_ASSERTIONS=ON \
-DCMAKE_BUILD_TYPE=RELEASE
$ ninja
$ ninja check-mlir
```

### Build buddy-mlir

```
$ cd buddy-mlir
$ mkdir build
$ cd build
$ cmake -G Ninja .. \
    -DMLIR_DIR=$PWD/../llvm/build/lib/cmake/mlir \
    -DLLVM_DIR=$PWD/../llvm/build/lib/cmake/llvm \
    -DLLVM_ENABLE_ASSERTIONS=ON \
    -DCMAKE_BUILD_TYPE=RELEASE
$ ninja
```

## Dialect

### Image Dialect

[Image dialect](./include/Dialect/Img) is designed for digital image processing workloads.
## Tools

### conv-opt

The conv-opt is an MLIR convolution optimizer. 

So far, we provide the 2D convolution vectorization pass `conv-vectorization`. The pass implements the Coefficients Broadcasting algorithm with Strip Mining strategy, and the strip mining size is configurable. Take the size of 256 as an example, you can use the tool with the following configuration.

```
$ conv-opt <input> -conv-vectorization="strip-mining=256"
```

**Conversion example**

We provide a function with `linalg.conv_2d` operation. You can use the following commands to print the conversion result.

```
$ cd buddy-mlir/build/bin
$ ./conv-opt ../../examples/conv-opt/conv2d.mlir -conv-vectorization="strip-mining=256"
```

**Edge detection example**

We also provide an edge detection example with the `conv-opt` tool. The `conv-opt` is responsible for lowering the `linalg.conv_2d` to LLVM IR dialect.
And then we use `mlir-translate` and `llc` tools to generate the object file. At last, we call the MLIR convolution function in a C++ program.

Please use a Linux machine with OpenCV installed to play around.
You should specify the `<strip mining size>` (e.g. 256) and `<ISA vector extension>` (e.g. avx512f).

```
$ cd buddy-mlir/build
$ cmake -G Ninja .. \
    -DBUDDY_CONV_OPT_EXAMPLES=ON \
    -DBUDDY_CONV_OPT_STRIP_MINING=<strip mining size> \
    -DBUDDY_CONV_OPT_ATTR=<ISA vector extension>
$ ninja edge-detection
```

We provide an image at `buddy-mlir/examples/conv-opt/images/YuTu.png`, which is the robotic lunar rover that formed part of the Chinese Chang'e 3 mission.
You can detect the edge of the image with `edge-detection`.

```
$ cd bin
$ ./edge-detection ../../examples/conv-opt/images/YuTu.png result.png
```

Note: the edge detection example currently only supports 1026x1026 images.

### img-opt

The img-opt is the driver for image dialect. 

```
$ img-opt <input> -lower-img
```

**Conversion example**

Take prewitt operation as an example. 
If every element of the input is 1, it means there are no edges in the image.
As a result, the convolution with the prewitt operation is 0.

```
$ ./build/bin/img-opt ./examples/img-opt/prewitt.mlir -lower-img |\
  ./llvm/build/bin/mlir-opt -convert-linalg-to-loops -lower-affine -convert-linalg-to-llvm -convert-scf-to-std -convert-vector-to-llvm -convert-std-to-llvm |\
  ./llvm/build/bin/mlir-cpu-runner -entry-point-result=void -shared-libs=./llvm/build/lib/libmlir_runner_utils.so

Unranked Memref base@ = 0x55e62feb6be0 rank = 2 offset = 0 sizes = [7, 7] strides = [7, 1] data = 
[[0,   0,   0,   0,   0,   0,   0], 
 [0,   0,   0,   0,   0,   0,   0], 
 [0,   0,   0,   0,   0,   0,   0], 
 [0,   0,   0,   0,   0,   0,   0], 
 [0,   0,   0,   0,   0,   0,   0], 
 [0,   0,   0,   0,   0,   0,   0], 
 [0,   0,   0,   0,   0,   0,   0]]
```
