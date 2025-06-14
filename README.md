# Solving a $n^2 × n^2$ Sudoku using the Grover's Algorithm

- [Introduction](#introduction)
- [Grover Algorithm](#grover_algorithm)
- [Circuit Design](#circuit_design)
  - [Pairwise_Checker](#pairwise_checker)
  - [Rule_Checker](#rule_checker)
  - [Oracle_Design](#oracle_design)
  - [Diffuser](#diffuser)
  - [Grover_Circuit](#grover_circuit)
- [Hardware_Limitation](#hardware_limitation)

## Introduction

In a standard Sudoku puzzle, the goal is to complete a $3^2 × 3^2$ grid by placing digits so that every row, every column, and each of the nine $3 × 3$ blocks contains all the numbers from 1 to 9. The puzzle begins with a partially filled grid, and a properly designed puzzle will have one unique solution. Solving a Sudoku problem can be seen as a unstructured search. We know the rules are but not the solution(s). Suppose the problem to be solved is a specific $2^2 \times 2^2$ Sudoku:

![sudoku_001](https://drive.google.com/uc?id=1Ki0M0Rdc6TbuKKS6T-u14E0q8lFVCD3Z)

In an unstructured search, we can list all possibilities (both correct and wrong solutions) and start trying one by one. Since there are 16 numebers, each number can be 1, 2, 3 or 4, so there are $N=4^{16}=4294967296$ possibilities, then this search on a classical computer would take $\frac{N}{2}=2147483648$ (i.e. $O(N)$ ) trials on average. In contrast, if we use Grover's algorithm, which is made to solve unstructured search problems, we can reduce the number of trials to $O(\sqrt{N})$ (where $\sqrt{N}=65536$ ) on average.

## Grover_Algorithm

We will encode the 1, 2, 3, 4 by 00, 01, 10, 11 respectively. therefore, a solution would have 32 bits.

The Grover algorithm has the following steps:
1. A "list" to search: Create all possibilities based on the number of qubits and Hadamard gate. All the possibilities will be included in an equal superposition of $\left| x_0x_1...x_{31} \right\rangle$, which is created by $H^{\otimes 32}\left| 0 \right\rangle$.
2. The oracle, a gate acts as a solution checker, will check each possibility in the "list" created in step 1 and mark the solution. In quantum wordings, all basis state of $H^{\otimes 32}\left| 0 \right\rangle$ will go through the oracle gate and being phrase flipped only if it is a correct solution, unchanged otherwise.
3. The marked version of $H^{\otimes 32}\left| 0 \right\rangle$ will go through a diffuser such that it becomes a superposition with the correct solution emphasized.
4. Measure the final superposition, one would get the most emphasized basis state (in other words, the correct solution) most of the time. 

## Circuit_Design

Sudoku has some rules on the solution:
- The numbers in each row are distinct
- The numbers in each column are distinct
- The numbers in each $2 × 2$ non-overlapping block are distinct, where the first $2 × 2$ block is at the top-left.

A $2^2 × 2^2$ Sudoku has exactly $4$ rows, columns and blocks respectively. Fulfilling all the rules means a solution has been found. It is intuitive to create at least 12 ancillas as controls such that a multi-controlled Z gate on these ancillas could flip the phase (mark the state as the solution) when all ancillas are 1s (all rules fulfilled). In this project, we will only consider a specific $2^2 \times 2^2$ Sudoku problem given above. The circuit design is custom for this problem only. A general circuit design to solve an arbitrary Sudoku problem is doable but that would be a bigger project.

### Pairwise_Checker

This gate is the key component of the marker circuit (the oracle) as a solution checker. A oracle is a blackbox that takes all possible states and all ancillas as input. The output of the oracle would be almost identical to the input except the sign of the state that represents the correct solution will be flipped. Then this output (without the ancillas) will go through diffuser and the output of the diffuser is a superposition that is ready to be observed. The outcome of the observation is very likely to be the correct solution of the Sudoku.

Each ancillas is a check for duplicate numbers on either rows/columns/boxes, i.e. $n^2$ bits. Ancilla is set to 1 if a row/column/box passed a check.

Create the following gate, named as $G$, which compares two numbers:
- If the two numbers are the same, set the output to 0
- set the output to 1 otherwise

For each row/column/box, if there are $n^2$ numbers, the required number of check on two numbers is $(n^4-n^2)/2$. When $n=2$, 6 comparisons are required for each row/column/box.

#### Compare two numbers:
![image](https://github.com/user-attachments/assets/ba6e0d6e-26db-40e6-8235-e5d727dcbca4)

#### Inverse of Compare two numbers:
![image](https://github.com/user-attachments/assets/6e1d44d1-13ef-4980-9525-63004bba7e18)

Note that the inverse is needed after each pairwise check such that all altered numbers are recovered for subsequent checks.

### Rule_Checker

Use 6 pairwise check to create a rule check.

#### A general rule check, comparing 6 pairs of numbers:
![image](https://github.com/user-attachments/assets/0f3b08e7-ea40-4004-a8c1-885fa0de8760)

The above is a general rule check. In a specific Sudoku problem, some numbers are already given so we have to adjust the general check to incorporate number checks. One example is shown below:
#### A specific rule check for row 1, check whether 1st, 2nd, 3rd numbers are 1, 4, 3 respectively, and comparing 6 pairs of numbers:
![image](https://github.com/user-attachments/assets/79774ba4-e74c-4da0-acd9-22fd442b6cf7)

11 more (R2,R3,R4,C1,C2,C3,C4,B1,B2,B3,B4) specific checks for rows, columns and blocks are being developed in the Jupyter Notebook. Combining 12 of these checks, we created a gate call "Solution_Check" and its inverse "Inv_Solution_Check".

### Oracle_Design

The oracle consists of the "Solution_Check" gate, Control-Z gate, "Inv_Solution_Check" gate which illustrated below:

![image](https://github.com/user-attachments/assets/c5f5655b-6cea-45c2-8e5a-418d1125f3bf)

### Diffuser

The diffuser circuit performs the following to the marked superposition:\
\
$2\left| s \right\rangle\left\langle s \right|-I=H^{\otimes n}(2\left| 0 \right\rangle\left\langle 0 \right|-I)H^{\otimes n}$\
\
where $\left| s \right\rangle$ is the equal superposition.

Hence we can create the diffuser circuit using:

![image](https://github.com/user-attachments/assets/b24e4dec-8697-4d56-960c-f82ab93c7349)

### Grover_Circuit

The grover circuit is the combination of the above with the following structure:

![image](https://github.com/user-attachments/assets/099d04fb-ab21-476e-8273-b029c48b9a31)

## Hardware_Limitation

In theory, the above grover circuit could work and produce a state vector (superposition) with the correct answer being significantly emphasized. However, the number of qubits is large (32 qubits) and the number of ancillas is also large (23 ancillas), this makes the circuit impossible to run on a typical laptop. The error I got is "MemoryError: Unable to allocate 512. PiB for an array with shape (36028797018963968,) and data type complex128", means I need at least 512. PiB RAM to run this code. 
