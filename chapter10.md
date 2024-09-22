# Chapter 10

### 1
The number of elements is 1024, so there are 512 threads and 16 warps.  
During the 1st iteration, all threads are active.  
During the 2nd iteration, threadIdx.x % 2 == 0 threads are active.  
During the 3th iteration, threadIdx.x % 4 == 0 threads are active.  
During the 4th iteration, threadIdx.x % 8 == 0 threads are active.  
During the 5th iteration, threadIdx.x % 16 == 0 threads are active.  
So, **all the 16 warps** have divergence during the 5th iteration. 

### 2
During the 1st iteration, all threads are active.  
During the 2nd iteration, the first 1/2 threads are active.   
During the 3th iteration, the first 1/4 threads are active.  
During the 4th iteration, the first 1/8 threads are active.  
During the 5th iteration, the first 1/16 threads are active, say, the first 32 threads are active.    
The first warp have no divergence because all 32 threads are active, and the latter 5 warps also have no divergence because all threads are not active. So **no warps** have divergence.  

### 3
```C++
__global__ void Kernel(float * input, float * output){
	unsigned int i = blockDim.x + threadIdx.x; 
    for (unsigned int stride = blockDim.x; stride >= 1; stride /= 2){
        if (threadIdx.x >= blockDim.x - stride){
            input[i] += input[i - stride]; 
        }
        __synthreads(); 
    }
    if (threadIdx.x == blockDim.x){
        *output = input[threadIdx.x]; 
    }
}
```

### 4
```c++
__global__ void Kernel(float * input, float * output){
    __shared__ float input_s[BLOCK_DIM]; 
    unsigned int segment = COARSE_FACTOR * 2 * blockDim.x * blockIdx.x; 
    unsigned int i = segment + threadIdx.x; 
    unsigned int t = threadIdx.x; 
    
    float max = input[i];  
    for (unsigned int turn = 1; turn < 2 * COARSE_FACTOR; turn++){
        if max < input[i + turn * blockDim.x]{  // The difference is here
            max = input[i + turn * blockDim.x]; 
        }
    }
    input_s[t] = max; 
    
    for (unsigned int stride = blockDim.x / 2; stride >= 1; stride /= 2){
        __synthreads(); 
        if (t < stride){
            if (input_s[t] < input_s[t + stride]){  // The difference is here
                input_s[t] = input_s[t + stride];
            }
        }
    }
    
    if (t == 0){
        atomicMax(output, input_s[0]);  // The difference is here
    }
}
```

### 5
```c++
__global void Kernel(float * input, float * output, unsigned int N){
    __shared__ float input_s[BLOCK_DIM]; 
    unsigned int segment = COARSE_FACTOR * 2 * blockDim.x * blockIdx.x;
    unsigned int i = segment + threadIdx.x; 
    unsigned int t = threadIdx.x; 
    
    float sum = input[i];
    for (unsigned int tile = 1; tile < COARSE_FACTOR * 2; tile++){
        if (i + tile * blockDim.x < N){  // The difference is here
            sum += input[i + tile * blockDim.x]; 
        }
    }
    input_s[t] = sum; 
    
    for (unsigned int stride = blockDim.x / 2; stride >= 1; stride /= 2){
        __synthreads(); 
        if (t < stride){
            input_s[t] += input_s[t + stride]; 
        }
    }
    
    if (t == 0){
        atomicAdd(output, input_s[0]); 
    }
}
```

### 6
#### a
blockDim.x = 4, there are 4 threads.  
start:  6  2  7  4  5  8  3  1  
iter1:  8  2  11 4  13 8  4  1  
iter2:  19 2  11 4  17 8  4  1  
iter3:  36 2  11 4  17 8  4  1
#### b  
start:  6  2  7  4  5  8  3  1  
iter1:  11 10 10 5  5  8  3  1  
iter2:  21 15 10 5  5  8  3  1  
iter3:  36 15 10 5  5  8  3  1

