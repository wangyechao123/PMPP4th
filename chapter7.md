# Chapter 7 Convolution

### 1
51

### 2
8, 21, 13, 20, 7

### 3
a. stay the same  
b. move to the left by one element  
c. move to the right by one element  
d. calculate the first derivative  
e. calculate the average

### 4
assume M is odd number, and let $r = (M - 1) / 2$  
a. $2r$  
b. $N * M$ multiplications. The size of output array is N, M multiplications for each element.  
c. $(N - 2r) * M$. The size of output array is N - 2r, M multiplications for each element.

### 5
assume M is odd number, and let $r = (M - 1) / 2$  
a. $4r(r + 1)$. The four corners.  
b. $N^2M^2$  
c. $(N-2r)^2M^2$

### 6
let $r_1 = (M_1 - 1) / 2$ and $r_2 = (M_2 - 1) / 2$  
a. $4r_{1}r_{2} + 2N_{1}r_{2} + 2N_{2}r_{1}$  
b. $N_{1}N_{2}M_{1}M_{2}$  
c. $(N_{1} - 2r_1)(N_{2} - 2r_2)M_{1}M_{2}$

### 7
let $r = (M - 1) / 2$  
a. $(\lfloor{\frac{N}{T}}\rfloor{} + 1) \times (\lfloor{\frac{N}{T}}\rfloor{} + 1)$  
b. $(T+2r) \times (T+2r)$  
c. $(T+2r) \times (T+2r) \times 4$ Byte  
d. num of blocks is the same, num of threads per block is $T \times T$, shared memory is $4T^2$ Byte  


### 8 
``` C++
__global__ void convolution_3D_basic_kernel(float *N, float *F, float *P, int r, int depth, int height, int width){
    int out_z = blockIdx.z * blockDim.z + threadIdx.z; 
    int out_y = blockIdx.y * blockDim.y + threadIdx.y; 
    int out_x = blockIdx.x * blockDim.x + threadIdx.x; 
    
    float Pvalue = 0.0f; 
    for(int i = 0; i < 2 * r + 1; i++){
        for(int j = 0; j < 2 * r + 1; j++){
            for(int k = 0; k < 2 * r + 1; k++){
                int in_z = out_z + i - r; 
                int in_y = out_y + j - r; 
                int in_x = out_x + k - r;
                if (0 <= in_z && in_z < depth && 0 <= in_y && in_y < height && 0 <= in_x && in_x < width){
                    Pvalue += F[i][j][k] * N[in_z * height * width + in_y * width + in_x]; 
                }
            }
        }
    }
    p[out_z][out_y][out_x] = Pvalue; 
}
```


### 9 
```c++
#define FILTER_RADIUS 2
__constant__ float F[2 * FILTER_RADIUS + 1][2 * FILTER_RADIUS + 1][2 * FILTER_RADIUS + 1];

// cudaMemcpyToSymbol(F, F_h, (2 * FILTER_RADIUS + 1) * (2 * FILTER_RADIUS + 1) * (2 * FILTER_RADIUS + 1) * sizeof(float)); 

__global__ void convolution_3D_const_mem_kernel(float *N, float *P, int r, int depth, int height, int width){
    int out_z = blockIdx.z * blockDim.z + threadIdx.z; 
    int out_y = blockIdx.y * blockDim.y + threadIdx.y; 
    int out_x = blockIdx.x * blockDim.x + threadIdx.x; 
    
    float Pvalue = 0.0f; 
    for(int i = 0; i < 2 * r + 1; i++){
        for(int j = 0; j < 2 * r + 1; j++){
            for(int k = 0; k < 2 * r + 1; k++){
                int in_z = out_z + i - r; 
                int in_y = out_y + j - r; 
                int in_x = out_x + k - r;
                if (0 <= in_z && in_z < depth && 0 <= in_y && in_y < height && 0 <= in_x && in_x < width){
                    Pvalue += F[i][j][k] * N[in_z * height * width + in_y * width + in_x]; 
                }
            }
        }
    }
    p[out_z][out_y][out_x] = Pvalue; 
}
```

### 10
```c++
#define IN_TILE 32
#define OUT_TILE (IN_TILE - 2 * FILTER_RADIUS)
__constant__ float F[2 * FILTER_RADIUS + 1][2 * FILTER_RADIUS + 1][2 * FILTER_RADIUS + 1]; 

__global__ void convolution_tiled_3D_const_mem_kernel(float *N, float *P, int depth, int height, int width){
    int out_z = blockIdx.z * OUT_TILE + threadIdx.z - FILTER_RADIUS;
    int out_y = blockIdx.y * OUT_TILE + threadIdx.y - FILTER_RADIUS;
    int out_x = blockIdx.x * OUT_TILE + threadIdx.x - FILTER_RADIUS;
    
    __shared__ N_s[IN_TILE][IN_TILE][IN_TILE]; 
    if (0 <= out_z && out_z < depth && 0 <= out_y && out_y < height && 0 <= out_x && out_x < width){
        N_s[threadIdx.z][threadIdx.y][threadIdx.x] = N[out_z * height * width + out_y * width + out_x];
    } else {
        N_s[threadIdx.z][threadIdx.y][threadIdx.x] = 0.0; 
    }
    __synthreads(); 
    
	
    if (0 <= out_z && out_z < depth && 0 <= out_y && out_y < height && 0 <= out_x && out_x < width){
        int tile_z = threadIdx.z - FILTER_RADIUS; 
        int tile_y = threadIdx.y - FILTER_RADIUS; 
        int tile_x = threadIdx.x - FILTER_RADIUS; 
        if (0 <= tile_z && tile_z < OUT_TILE && 0 <= tile_y && tile_y < OUT_TILE && 0 <= tile_x && tile_x < OUT_TILE){
            float Pvalue = 0.0f; 
            for(int i = 0; i < 2 * FILTER_RADIUS + 1; i++){
                for(int j = 0; j < 2 * FILTER_RADIUS + 1; j++){
                    for(int k=0; k < 2 * FILTER_RADIUS + 1; k++){
                        Pvalue += F[i][j][k] * N_s[tile_z + i][tile_y + j][tile_x + k]; 
                    }
                }
            }
            P[out_z * height * width + out_y * width + out_x] = Pvalue; 
        }
    }
}
```
