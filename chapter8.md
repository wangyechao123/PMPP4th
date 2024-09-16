# Chapter 8

### 1
Let us assume order of question 1 is 1.  
a. $(120 - 2 * order)^3$, if the order is 1, then output grid points is $118^3$  
b. $\lceil{}\frac{120}{8}\rceil{} = 15 $, so the number of blocks is $15^3 = 3375$  
c. the input tile size is 8, so the output tile size is 8 - 2 * order = 6, $\lceil{}\frac{120}{6}\rceil{} = 20$, so the number of blocks is $20^3 = 8000$  
d.  the input tile size is 32, so the output tile size is 30, $\lceil{}\frac{120}{30}\rceil{} = 4$, so the number of blocks is $4^3 = 64$  


### 2 
a. 32 * 32 * 18  
b. 30 * 30 * 16  
c. $\frac{30 * 30 * 16 * 13}{32 * 32 * 18 * 4} = 2.539 OP/B$  
d. 3 * 32 * 32 * 4 B = 12288 B  
e. 32 * 32 * 4 B = 4096 B  
