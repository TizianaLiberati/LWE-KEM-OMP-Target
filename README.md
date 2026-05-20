# LWE-KEM GPU Offloading

This repository contains a GPU-parallel implementation of a lattice-based LWE-KEM scheme.  
The code provides two GPU offloading backends:

- **OpenACC**
- **OpenMP Target Offloading**

Randomness on GPU is generated through **RNGonGPU**, included as a Git submodule.

---

# Clone

Clone the repository together with all submodules:

```bash
git clone --recurse-submodules https://github.com/TizianaLiberati/LWE-KEM-OMP-Target.git
cd LWE-KEM-OMP-Target
```

If the repository was already cloned without submodules:

```bash
git submodule update --init --recursive
```

---

# Requirements

The code requires:

- NVIDIA GPU
- NVIDIA HPC SDK (`nvc++`)
- CUDA toolkit (`nvcc`)
- OpenSSL development libraries
- CMake (required to build RNGonGPU)

---

# Build RNGonGPU

RNGonGPU must be compiled before building the main code.

## GH200 / H100 nodes

```bash
cd RNGonGPU

cmake -S . -B build \
  -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_CUDA_ARCHITECTURES=90

cmake --build build -j
```

## A100 nodes

```bash
cd RNGonGPU

cmake -S . -B build \
  -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_CUDA_ARCHITECTURES=80

cmake --build build -j
```

---

# Build the LWE-KEM code

```bash
cd ../CodiceBoth
make clean
make GPU_ARCH="-gpu=cc90,mem:managed"
```

For A100 nodes:

```bash
make GPU_ARCH="-gpu=cc80,mem:managed"
```

The build generates two executables:

```text
lwe_kem_acc   # OpenACC implementation
lwe_kem_omp   # OpenMP Target implementation
```

---

# Run

The executables take two input parameters:

```bash
./executable N n
```

where:

- `N` is the number of KEM executions
- `n` is the LWE dimension

---

# Examples

## OpenACC version

```bash
./lwe_kem_acc 10 4096
```

## OpenMP Target version

```bash
./lwe_kem_omp 10 4096
```

---

# Complete Example Workflow

```bash
git clone --recurse-submodules https://github.com/TizianaLiberati/LWE-KEM-OMP-Target.git

cd LWE-KEM-OMP-Target

cd RNGonGPU

cmake -S . -B build \
  -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_CUDA_ARCHITECTURES=90

cmake --build build -j

cd ../CodiceBoth

make clean
make GPU_ARCH="-gpu=cc90,mem:managed"

./lwe_kem_acc 10 4096
./lwe_kem_omp 10 4096
```

---

# GPU Architectures

| GPU | Architecture |
|---|---|
| NVIDIA A100 | `cc80` / `CMAKE_CUDA_ARCHITECTURES=80` |
| NVIDIA H100 / GH200 | `cc90` / `CMAKE_CUDA_ARCHITECTURES=90` |

---

# Notes

- RNGonGPU is included as a Git submodule.
- RNGonGPU must be built before compiling the main code.
- The final executables link against both RNGonGPU and GPU-NTT libraries.
- The `mem:managed` option is used for CUDA Unified Memory support.
