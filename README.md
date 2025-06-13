# Solving a $n^2 × n^2$ Sudoku using the Grover's Algorithm

- [Introduction](#introduction)
- [Grover Algorithm](#grover_algorithm)
- [Circuit Design](#circuit_design)

## Introduction

In a standard Sudoku puzzle, the goal is to complete a $3^2 × 3^2$ grid by placing digits so that every row, every column, and each of the nine $3 × 3$ subgrids contains all the numbers from 1 to 9. The puzzle begins with a partially filled grid, and a properly designed puzzle will have one unique solution. Solving a Sudoku problem can be seen as a unstructured search. We know the rules are but not the solution(s). Suppose the problem to be solved is a specific $2^2 \times 2^2$ Sudoku:

![sudoku_001](https://drive.google.com/uc?id=1Ki0M0Rdc6TbuKKS6T-u14E0q8lFVCD3Z)

In an unstructured search, we can list all possibilities (both correct and wrong solutions) and start trying one by one. Since there are 4 blanks, each blank can be 1, 2, 3 or 4, so there are $N=4^4=256$ possibilities, then this search on a classical computer would take $\frac{N}{2}=128$ (i.e. $O(N)$ ) trials on average. In contrast, if we use Grover's algorithm, which is made to solve unstructured search problems, we can reduce the number of trials to $O(\sqrt{N})$ on average.

## Grover_Algorithm

The Grover algorithm has the following steps:
1. A "list" to search: Create all possibilities based on the number of qubits and Hadamard gate. All the possibilities will be included in an equal superposition of $\left| x_0x_1...x_{7} \right\rangle$, which is created by $H^{\otimes 8}\left| 0 \right\rangle$.
2. The oracle, a gate acts as a solution checker, will check each possibility in the "list" created in step 1 and mark the solution. In quantum wordings, all basis state of $H^{\otimes 8}\left| 0 \right\rangle$ will go through the oracle gate and being phrase flipped only if it is a correct solution, unchanged otherwise.
3. The marked version of $H^{\otimes 8}\left| 0 \right\rangle$ will go through a diffuser such that it becomes a superposition with the correct solution emphasized.
4. Measure the final superposition, one would get the most emphasized basis state (in other words, the correct solution) most of the time. 

## Circuit_Design

We will encode the 1, 2, 3, 4 by 00, 01, 10, 11 respectively. 

Sudoku has some rules on the solution:
- The numbers in each row are distinct
- The numbers in each column are distinct
- The numbers in each $2 × 2$ non-overlapping subgrid are distinct, where the first $2 × 2$ subgrid is at the top-left.

A $2^2 × 2^2$ Sudoku has exactly $4$ rows, columns and blocks respectively. Fulfilling all the rules means a solution has been found. It is intuitive to create at least 12 ancillas as controls such that a multi-controlled Z gate on these ancillas could flip the phase (mark the state as the solution) when all ancillas are 1s (all rules fulfilled). In this project, we will only consider a specific $2^2 \times 2^2$ Sudoku problem given above. The circuit design is custom for this problem only. A general circuit design to solve an arbitrary Sudoku problem is doable but that would be a bigger project.

### Create a gate that checks if two numbers are the same
