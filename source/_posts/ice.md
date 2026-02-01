---
title: Ice Cramer
---
## Ice Cramer CTF Writeup

This challenge involves solving a system of linear equations where the unknown variables are the ASCII values of the flag characters. The challenge title and description hint at **Cramer's Rule**, though standard matrix solvers are more efficient for automation.

### 1. Challenge Overview

The server provides a set of equations in the format:
`k_0*x_0 + k_1*x_1 + ... + k_n*x_n = constant`

* 
**Variables ():** Represent the integer ASCII values of the flag characters (with the `pascalCTF{}` wrapper removed).


* 
**Coefficients ():** Randomly generated integers between -100 and 100.


* 
**Result ():** The sum of the products of the coefficients and the ASCII values.



### 2. The Math

The problem is represented by the linear equation :

* **** is the matrix of coefficients.
* **** is the vector of unknown ASCII values.
* **** is the vector of constants (the solutions on the right side of the equations).

Because the system is square ( in this instance), it can be solved by calculating .

### 3. Exploitation Strategy

The solution requires automating three steps:

1. 
**Connection:** Use `pwntools` to connect to the remote server via netcat.


2. 
**Parsing:** Extract the integers () and the solution () for every line using Regular Expressions (regex).


3. 
**Linear Algebra:** Use `numpy.linalg.solve` to calculate the values of .



### 4. Solution Script

```python
import numpy as np
import re
from pwn import *

# Connection details
io = remote('cramer.ctf.pascalctf.it', 5002)

# Receive output until the prompt
raw_data = io.recvuntil(b'Solve the system of equations to find the flag!').decode()

lines = raw_data.strip().split('\n')
A = []
B = []

for line in lines:
    if '=' not in line or 'x_' not in line:
        continue
    
    # Extract the constant after '='
    parts = line.split('=')
    B.append(int(parts[1].strip()))
    
    # Extract coefficients (k) before each '*x_'
    coeffs = re.findall(r'(-?\d+)\*x_', parts[0])
    A.append([int(c) for c in coeffs])

# Convert to arrays and solve
matrix_A = np.array(A)
vector_B = np.array(B)
solution = np.linalg.solve(matrix_A, vector_B)

# Convert ASCII floats to characters
flag = "".join([chr(int(round(val))) for val in solution])
print(f"pascalCTF{{{flag}}}")

```

### 5. Conclusion

By treating the flag characters as variables in a linear system, we can use matrix operations to recover them. The script successfully parsed a  matrix to reveal the flag:

**`pascalCTF{0h_My_G0DD0_too_much_m4th_:O}`**
