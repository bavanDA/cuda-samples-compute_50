# CUDA 12.8 Samples Setup Guide

This guide explains how to install CUDA Toolkit 12.8, configure GCC 14 as the default compiler, fix a header file compatibility issue, and build and run CUDA samples.

---

## 1. Install CUDA Toolkit 12.8

Download and install **CUDA Toolkit 12.8** from NVIDIA:

👉 [CUDA 12.8 Download (Fedora 41 x86_64)](https://developer.nvidia.com/cuda-12-8-0-download-archive?target_os=Linux&target_arch=x86_64&Distribution=Fedora&target_version=41)

Follow the installation steps provided for your distribution.

---

## 2. Install GCC 14 and Make It Default

CUDA 12.8 does not support GCC versions higher than 14.x.x. Install GCC 14 and configure it as the default compiler:

```bash
sudo dnf install gcc-14 g++-14
sudo alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-14 60
sudo alternatives --install /usr/bin/g++ g++ /usr/bin/g++-14 60
sudo alternatives --config gcc
sudo alternatives --config g++
```

Verify:

```bash
gcc --version
g++ --version
```

Both should show **GCC 14.x**.

---

## 3. Patch `math_functions.h`

CUDA 12.8 has a known issue with **glibc 2.41+** that causes errors like:

```
error: exception specification is incompatible
```

### Reference
- NVIDIA Developer Forum: [Exception specification incompatible for cospi/sinpi with glibc 2.41](https://forums.developer.nvidia.com/t/error-exception-specification-is-incompatible-for-cospi-sinpi-cospif-sinpif-with-glibc-2-41/323591/4?u=epk)

### Patch Instructions

1. Edit the file:

   ```
   /usr/local/cuda-12.8/targets/x86_64-linux/include/crt/math_functions.h
   ```

2. Apply the following patch (example shown for Gentoo ~amd64, works with CUDA 12.6.1 and 12.8.0):

```diff
--- a/builds.orig/cuda_nvcc/targets/x86_64-linux/include/crt/math_functions.h	2024-08-23 00:25:39.000000000 +0200
+++ b/builds/cuda_nvcc/targets/x86_64-linux/include/crt/math_functions.h	2025-02-17 01:19:44.270292640 +0100
@@ -2547,7 +2547,7 @@
  *
  * \note_accuracy_double
  */
-extern __DEVICE_FUNCTIONS_DECL__ __device_builtin__ double                 sinpi(double x);
+extern __DEVICE_FUNCTIONS_DECL__ __device_builtin__ double                 sinpi(double x) noexcept (true);
 /**
  * \ingroup CUDA_MATH_SINGLE
  * \brief Calculate the sine of the input argument 
@@ -2570,7 +2570,7 @@
  *
  * \note_accuracy_single
  */
-extern __DEVICE_FUNCTIONS_DECL__ __device_builtin__ float                  sinpif(float x);
+extern __DEVICE_FUNCTIONS_DECL__ __device_builtin__ float                  sinpif(float x) noexcept (true);
 /**
  * \ingroup CUDA_MATH_DOUBLE
  * \brief Calculate the cosine of the input argument 
@@ -2592,7 +2592,7 @@
  *
  * \note_accuracy_double
  */
-extern __DEVICE_FUNCTIONS_DECL__ __device_builtin__ double                 cospi(double x);
+extern __DEVICE_FUNCTIONS_DECL__ __device_builtin__ double                 cospi(double x) noexcept (true);
 /**
  * \ingroup CUDA_MATH_SINGLE
  * \brief Calculate the cosine of the input argument 
@@ -2614,7 +2614,7 @@
  *
  * \note_accuracy_single
  */
-extern __DEVICE_FUNCTIONS_DECL__ __device_builtin__ float                  cospif(float x);
+extern __DEVICE_FUNCTIONS_DECL__ __device_builtin__ float                  cospif(float x) noexcept (true);
 /**
  * \ingroup CUDA_MATH_DOUBLE
  * \brief  Calculate the sine and cosine of the first input argument
```

3. Save the file.

---

## 4. Install CMake (3.20+)

Check if you have CMake installed:

```bash
cmake --version
```

If version < 3.20, update:

```bash
sudo apt install cmake    # Ubuntu/Debian
# or
sudo dnf install cmake    # Fedora
```

---

## 5. Build CUDA Samples

1. Clone and navigate to your CUDA samples repository:

   ```bash
   git clone https://github.com/bavanDA/cuda-samples-compute_50.git
   cd cuda-samples-compute_50
   ```

2. Create and enter a build directory:

   ```bash
   mkdir build && cd build
   ```

3. Configure the project:

   ```bash
   cmake ..
   ```

4. Build using all cores:

   ```bash
   make -j$(nproc)
   ```

---

## 6. Run Samples

You can run any sample from the `build` folder. For example:

```bash
./Samples/1_Utilities/deviceQuery/deviceQuery
```

---

## 7. Run All Tests

From the **samples root**:

```bash
cd ..
python3 run_tests.py --output ./test --dir ./build/Samples --config test_args.json
```

⚠️ **Note:** Not all tests will pass, because some require higher GPU compute power.

---

## Troubleshooting

- If you see **incompatible exception specification errors**, confirm you patched `math_functions.h`.
- If compilation fails, verify **GCC 14** is active:  
  ```bash
  gcc --version
  ```
- Use `nvidia-smi` to confirm driver and CUDA runtime compatibility.

