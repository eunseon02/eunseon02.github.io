---
layout: post
title: CUDA code tutorial
categories: [paper review]
tags: [slim, scene representation]
description: Sample placeholder post.
---

 CUDA Tutorial 페이지와 Alpha campus를 보고 정리한 내용이다!


## NVIDIA GPU Architecture

**type**

Tesla: for supercomputer(GPGPU)

Quadro :  for workstation(rendering, CAD)

Geforce : for Desktop(gaming) 

cc(compute capability), cuda 지원 버전이 출시년도가 다르다.

cc버전을 사용하기 위한 cuda 버전을 정해줘야 하는데 꼭 맞출 필요없이 cuda를 상위 버전 쓰면 된당!

**CPU**

-serial 

-OpenMP, MPI : 메모리 구조, 캐시 구조, 통신 고려

**GPU**

Multi core

hierachical structure

-Stream Processor(SM) : GPU core

-SP

GPU structure를 이해해야 성능이 좋은 코드를 만들 수 있다.

**Tesla V 100**



![Untitled](https://github.com/eunseon02/eunseon02.github.io/assets/108911413/4f72e37e-96af-4781-8c32-ab05a0938061)






NVLink 2.0/Tensor core

16GB HBM (High Bandwidth Memory)

6144 GB L2 Cache

SM

64 single-precision CUDA cores

32 double precision units

8 tensor cores(딥러닝의 정밀도를 계산하는 유닛)

Peak DP Perf : 7.0Tflops

single precision에 대한 초당 플로팅 연산

Peak Tensor Perf : 112 Tflops

**A 100**

**CUDA Kernel**

- **cudaDeviceSynchronize**

커널 호출은 스레드에 대해서 비동기적이다! cudaDeviceSynchronize 이를 동기화 시켜주기 위해서 모든 커널이 완료될 때까지 host application을 blocking하는 함수이다.

- **cudaDeviceReset**

## Code tutorial



### 01 : one GPU thread.

**CUDA Programming model**
![Untitled (1)](https://github.com/eunseon02/eunseon02.github.io/assets/108911413/7c6a5aa7-8e7a-4b2d-8f52-f7af426a8886)


kernel : GPU function “printHelloGpu” 

thead : 

block : 

grid : 

---

gridDim.x : number of blocks in grid “2”

blockIdx.x: block index “0, 1”

blockDim.x : number of threads in block “5”

threadIdx.x : thread index “0, 1, 2, 3, 4”

**CUDA Execution Model**

스레드 블록 단위를 SM에 분배하여 실행됨

데이터의 개수만큼 스레드가 생성되고 스레드를 코어에 스케줄링하게 된다!

**함수 정의**

```c
__global__ void func(){}
```

return 값을 가지지 않음

**함수 호출** 

```c
func<<<1,1>>>()
```

*kernel launch*

**Compiling CUDA**

**nvcc** : CUDA compiler

.cu 확장자를 가진 CUDA 소스 코드를 컴파일하기 위해 이용

C compiler, C++ compiler + $\alpha$  기능

```bash
nvcc hello.cu -o hello
```

-o : output file name

-arch : GPU architecture (arch=sm_70 ⇒ V100)

성능을 내기 위해서 필요하다!

Device Query Code를 통해서 CUDA 프로그래밍을 위한 정보들을 확인 가능하다

```c
nvcc DeviceQuery.cu -o DeviceQuery
```

**Action**

vector addition

```bash
cp vector_add.c vector_add.cu
nvcc vector_add.c -o vector_add
./vector_add
```

**Device memory management**

```c
cudaMalloc(void **devPtr, size_t count);
cudaFree(void *devPtr);
```

devPtr : device pointer

count : device memory of size

**Memory transfer**

```c
cudaMemcpy(void *dst, void *src, size_t count, cudaMemcpyKind kind)
```

kind : transfer direction (cudaMemcpyHostToDevice /cudaMemcpyDeviceToHost)

count : memory size

transfer from src to dst

**workflow of CUDA programs.**

1. Allocate host memory and initialized host data
2. Allocate device memory
3. Transfer input data from host to device memory
4. Execute kernels
5. Transfer output from device memory to host

```c
void main(){
    float *a, *b, *out;
    float *d_a;

    a = (float*)malloc(sizeof(float) * N);

    // Allocate device memory for a
    cudaMalloc((void**)&d_a, sizeof(float) * N);

    // Transfer data from host to device memory
    cudaMemcpy(d_a, a, sizeof(float) * N, cudaMemcpyHostToDevice);

    …
    vector_add<<<1,1>>>(out, d_a, b, N);
    …

    // Cleanup after kernel execution
    cudaFree(d_a);
    free(a);
}
```



### 02 : GPU parallelism.

***“thread block***"

```c
<<<M, T>>>
```

M : thread blocks 

T : parallel threads in each thread block

```c
func <<<threadIdx.x, blockDim.x>>>
```

threadIdx.x : the index of the thread within the block

blockDim.x : the size of thread block (number of threads in the thread block).

![Untitled (2)](https://github.com/eunseon02/eunseon02.github.io/assets/108911413/ad6bf088-5383-4208-8afc-4338ee985322)

n th iteration : **threadIdx.x + n*256**

### **Parallelizing idea**

**CUDA GPU**

***parallel processors : Streaming Multiprocessors*(*SMs)***

multiple parallel processors ⇒ multiple concurrent blocks

blockIdx.x : the index of block with in the grid

gridDim.x : the size of the grid

![Untitled (2)](https://github.com/eunseon02/eunseon02.github.io/assets/108911413/ab4fc2c6-1f78-476f-ad5a-edddfecda8a4)

block들 간의 coordinate를 잘 이해함으로써 GPU thread에 data를 적절하게 할당해야한다.

따라서 인덱스를 다음과 같이 할당할 수 있다.

**global index**

```c
int tid = blockIdx.x * blockDim.x + threadIdx.x;
```





