# Chapter2
1. C
2. C
3. D
4. C  
There are num of blocks ceil(8000/1024) = 8, so the num of threads is 1024 * 8 = 8192
5. D  
Pay attention to the "v integer elements" in question, it's 'v' not 'n'
6. D
7. C
8. C
9. a. 128  
b. num of blocks = ceil(200000/128) = 1563, so num of threads in the grid = 1563 * 128 = 200614  
c. 1563  
d. all of the 200614 threads  
e. 200000 threads
10. Use both \_\_global\_\_ and \_\_device\_\_ for function declaration
