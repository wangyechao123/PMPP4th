# Chapter 6

### 1
Fig6.4(A) and Fig6.4(B) represents different kernel functions. The kernel function of Fig6.4(A) is just the same as Fig5.9. Here I give the kernel function of Fig6.4(B) using corner turning.  
The matrix B in both Fig6.4(A) and (B) are column-major layout. But threads in Fig6.4(A) access data in a same row, so these access are not coalesced. while threads in Fig6.4(b) access data in a same column, and these access are coalesced.  
Both kernel functions are based on Fig5.9 Matrix Multiplication using Shared Memory. The main difference between these two kernel functions is the thread index when accessing data from matrix N to shared memory, which is Line 16 and Line 20 in the code below. **The key idea is exchanging the role of tx and ty.**  

```cpp {.line-numbers}
// Fig6.4(B) using corner turning, assume both tile and matrix are squared
#define TILE_WIDTH 16
__global__ void MatrixMulKernel(float * M, float * N, float * P, int Width){
    __shared__ float Mds[TILE_WIDTH][TILE_WIDTH];
    __shared__ float Nds[TILE_WIDTH][TILW_WIDTH];

    int tx = threadIdx.x; int ty = threadIdx.y; 
    int bx = blockIdx.x; int by = blockIdx.y; 
    int row = by * blockDim.y + ty;  // by * TILE_WIDTH + ty
    int col = bx * blockDim.x + tx; 

    float Pvalue = 0.0; 
    for (int ph = 0; ph < Width / TILE_WIDTH; ph++){

        Mds[ty][tx] = M[row * Width + (ph * TILE_WIDTH + tx)];  // M[by * TILE + ty][ph * TILE + tx]
        Nds[tx][ty] = N[(ph * TILE_WIDTH + tx) * Width + (bx * TILE_WIDTH + ty)];  // N[ph * TILE + tx][bx * TILE + ty]
        __synthreads__(); 

        for (int k = 0; k < TILE_WIDTH; k++){
            Pvalue += Mds[ty][k] * Nds[tx][k]; 
        }
        __synthreads__(); 
    }
    P[row * Width + col] = Pvalue;    
}
```


### 2
I don't understand this question.   

### 3
When you access to shared memory, coalescing is not applicable. And when you access to global memory, you need to specify whether coalesced or uncoalesced because global memory is DRAM. The key to specify whether global memory access is coalesced or uncoalesced is watch out the threadIdx.x.  
a. coalesced  
b. coalescing is not applicable  
c. coalesced   
d. uncoalesced  
e. coalescing is not applicable    
f. coalescing is not applicable   
g. coalesced  
h. coalescing is not applicable  
i. uncoalesced  


### 4
Firstly we copy the code below from P64 Fig3.11, P108 Fig 5.9, P140 Fig 6.13. Then we calculate the floating point to global memory access ratio (OP/B).

#### a
```cpp {.line-numbers}
__global__ void MatrixMulKernel(float * M, float * N, float * P, int Width){
    int row = blockIdx.y * blockDim.y + threadIdx.y; 
    int col = blockIdx.x * blockDim.x + threadIdx.x; 

    if ((row < Width) && (col < Width)){
        float Pvalue = 0.0; 
        for (int k = 0; k < Width; k++){
            // Pvalue += M[row][k] * N[k][col]; 
            Pvalue += M[row * Width + k] * N[k * Width + col]; 
        }
        P[row * Width + col] = Pvalue; 
    }
}
```
for each value in matrix P:  
  floating point: width multiplication and width add, total 2 * width  
  global meory access: width read from M, width read from N, and 1 write to P, total (2 * width + 1) * 4 B  
so ratio is $\frac{width}{4 * width + 2} = 0.25$

#### b
```cpp {.line-numbers}
#define TILE_WIDTH 32
__global__ void MatrixMulKernel(float * M, float * N, float * P, int Width){
    __shared__ float Mds[TILE_WIDTH][TILE_WIDTH];
    __shared__ float Nds[TILE_WIDTH][TILW_WIDTH];

    int tx = threadIdx.x; int ty = threadIdx.y; 
    int bx = blockIdx.x; int by = blockIdx.y; 
    int row = by * blockDim.y + ty;  // by * TILE_WIDTH + ty
    int col = bx * blockDim.x + tx; 

    float Pvalue = 0.0; 
    for (int ph = 0; ph < Width / TILE_WIDTH; ph++){

        Mds[ty][tx] = M[row * Width + (ph * TILE_WIDTH + tx)];  // M[row][ph * TILE_WIDTH + tx]
        Nds[ty][tx] = N[(ph * TILE_WIDTH + ty) * Width + col];  // N[ph * TILE_WIDTH + ty][col]
        __synthreads__(); 

        for (int k = 0; k < TILE_WIDTH; k++){
            Pvalue += Mds[ty][k] * Nds[k][tx]; 
        }
        __synthreads__(); 
    }
    P[row * Width + col] = Pvalue; 
}
```
for each tile in matrix P:  
  floating point: tile * tile * (width * 2) OP  
  global memory access: tile * tile * ph = tile * width for M, the same for N, and tile * tile * 1 for P   
so ratio is $\frac{2t^2w}{2tw+t^2}$, if $w >> t$, this is equal to $t = 32$ OP/B 

#### c 
```cpp {.line-numbers}
#define TILE_WIDTH 32 
#define COARSE_FACTOR 4
__global__ void matrixMulKernel(float * M, float * N, float * P, int Width){
    __shared__ float Mds[TILE_WIDTH][TILE_WIDTH]; 
    __shared__ float Nds[TILE_WIDTH][TILE_WIDTH]; 

    int bx = blockIdx.x; int by = blockIdx.y; 
    int tx = threadIdx.x; int ty = threadIdx.y; 

    int row = by * TILE_WIDTH + ty; 
    int colStart = bx * TILE_WIDTH * COARSE_FACTOR + tx; 

    float Pvalue[COARSE_FACTOR]; 
    for (int c = 0; c < COARSE_FACTOR; c++){
        Pvalue[c] = 0.0; 
    }

    for (int ph = 0; ph < Width / TILE_WIDTH; ph++){
        Mds[ty][tx] = M[row * Width + (ph * TILE_WIDTH + tx)]; 
        for (int c = 0; c < COARSE_FACTOR; c++){
            int col = colStart + c * TILE_WIDTH; 
            Nds[ty][tx] = N[(ph * TILE_WIDTH + ty) * Width + col]; 
            __synthreads__();

            for (int k = 0; k < TILE_WIDTH; k++){
                Pvalue[c] += Mds[ty][k] * Nds[k][tx]; 
            }
            __syncthreads__();
        }
    }

    for (int c = 0; c < COARSE_FACTOR; c++){
        int col = colStart + c * TILE_WIDTH; 
        P[row * Width + col] = Pvalue[c]; 
    }
}

```
the same as previous question, $32$ OP/B if $w >> t$
