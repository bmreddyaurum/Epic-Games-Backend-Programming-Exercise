
# Requirements and Assumptions

# Requirements
Write a function that, given a matrix of integers, builds a string with the entries of that matrix
appended in clockwise order.
String matrix(int[][] input)
For instance, the 3x4 matrix below:
2, 3, 4, 8
5, 7, 9, 12
1, 0, 6, 10
would make the string “2, 3, 4, 8, 12, 10, 6, 0, 1, 5, 7, 9”.

# Assumptions

Based on the requirements. Need to implement spiral traversal algorithm.

## Implementation

Wrote a program to implement the spiral traversal algorithm. The algorithm is as follows:

1. Starting from the top-left corner
2. Moving right, down, left, and up in succession
3. Narrowing boundaries with each complete pass

## Technical Requirements

- Java Runtime: 17+
- Testing Framework: JUnit 5

## Testing
- Unit Testing: JUnit 5

## How can you verify that your solution is correct?
    Created a JUnit test case to verify the correctness of the implementation.


## Useful Maven Commands

- **Run Tests:**  
  `mvn test`
- **Build JAR:**  
  `mvn clean package`

