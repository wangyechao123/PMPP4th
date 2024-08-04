# Chapter 3

### 1a
```C++
__global__ void MatrixMulRow(float * M, float * N, float * P, int width){
    int row = blockIdx.y * blockDim.y + threadIdx.y; 
    if (row < width){
        for(int i = 0; i < width; i++){
            float sum = 0.0; 
            for(int k = 0; k < width; k++){
                sum += M[row * width + k] * N[k * width + i]; 
            } 
            P[row * width + i] = sum; 
        }
    }
}


dim3 dimGrid(1, ceil(width / 64), 1);
dim3 dimBlock(1, 64, 1); 
//execution configuration parameters
MatrixMulRow<<<dimGrid, dimBlock>>>(M_d, N_d, P_d);
```

### 1b
just change little code from 1a, it's easy to do. 

### 1c
The differences between these two kernel designs are the read and write. Both kernels need to read from matrix M and N and write to matrix P. Both kernels read one matrix consecutively and another matrix nonconsecutively. But the kernel in 1a write to P consecutively, the kernel in 1b write to P nonconsecutively. 

### 2
  

### 3
a. 16 * 32 = 512  
b. blockDim (16, 32), gridDim (19, 5), so numbers of threads in the grid is 48640  
c. 19 * 5 = 95  
d. 150 * 300 = 45000

### 4
suppose the index is start from 0.  
a. 20 * 400 + 10 = 8010  
b. 10 * 500 + 20 = 5020

### 5
index = z * height * width + y * width + x = 5 * 500 * 400 + 20 * 400 + 10 = 1008010

