# Ernst Chladni, Chladni figures 
- explore the intersection of fractals and electronics, specifically focusing on recreating Chladni figures using electronic components and software. we can try simulations and hardware to visualize these patterns dynamically.
- Paul Bourke’s Geometry (Chladni Figures) looks at Chladni figures,
- which are patterns formed by vibrating surfaces covered with particles.
- It includes the mathematical and physical principles behind how these patterns emerge based on the frequency and mode of vibration.
- we can explore Chladni patterns, their mathematical modeling, and their application in electronics through the Fractansistor project. 

### TEC-1 mint code 
- warning - concept code
- need to add more ram, will not fit in 2k
- when the code fails then push yourself in the nuts
  


#### **Purpose**

This MINT program generates and displays a 16x16 grid pattern by simulating wave interference using integer arithmetic and precomputed sine and cosine values.

#### **Variables Used (All Single Letters)**

- `w`: Width of the grid.
- `h`: Height of the grid.
- `f`: Frequency of the wave patterns.
- `a`: Amplitude of the waves.
- `b`: Total grid size (`b = w * h`).
- `g`: Pointer to the allocated grid array.
- `s`: Sine lookup table (array of precomputed values).
- `c`: Cosine lookup table (array of precomputed values).
- `x`, `y`: Loop indices for grid coordinates.
- `m`, `n`: Indices for accessing the sine and cosine lookup tables.
- `p`, `q`: Sine and cosine values retrieved from the lookup tables.
- `t`, `u`: Temporary variables for intermediate calculations.
- `v`: Computed amplitude value at each grid point.
- `i`: Linear index in the grid array corresponding to coordinates `(x, y)`.

#### **Code Breakdown**

1. **Initialization**

   ```mint
   16 w !   // Set grid width to 16
   16 h !   // Set grid height to 16
   2 f !    // Set frequency to 2
   16384 a ! // Set amplitude to 16384 (mid-range for 16-bit signed integer)
   ```

   - Sets up the parameters for the grid size and wave properties.

2. **Memory Allocation**

   ```mint
   w h * b ! // Calculate total grid size b = w * h
   b /A g !  // Allocate memory for the grid array of size b
   ```

   - Calculates the total number of grid points and allocates memory for the grid array `g`.

3. **Lookup Tables**

   ```mint
   [0 12539 23170 30273 32767 30273 23170 12539 0 -12539 -23170 -30273 -32767 -30273 -23170 -12539] s !
   [32767 30273 23170 12539 0 -12539 -23170 -30273 -32767 -30273 -23170 -12539 0 12539 23170 30273] c !
   ```

   - Initializes the sine (`s`) and cosine (`c`) lookup tables with precomputed values scaled for 16-bit integer arithmetic.
   - These tables represent sine and cosine values at specific angles, avoiding the need for trigonometric calculations during runtime.

4. **Computing Grid Values**

   ```mint
   0 y !   // Initialize y to 0
   h (
     0 x !  // Initialize x to 0
     w (
       x f * 16 % m !   // m = (x * f) % 16
       y f * 16 % n !   // n = (y * f) % 16
       s m ? p !        // Retrieve sine value p from s at index m
       c n ? q !        // Retrieve cosine value q from c at index n
       a p * t !        // t = a * p
       t q * u !        // u = t * q
       u 1073741824 / v ! // Scale down u by dividing by 2^30 to keep within 16-bit range
       y w * x + i !    // Compute linear index i = x + y * w
       v g i ?!         // Store computed value v into grid array at index i
       1 x + x !        // Increment x
     )
     1 y + y !          // Increment y
   )
   ```

   - **Loops**: Nested loops iterate over each coordinate `(x, y)` in the grid.
   - **Index Calculations**:
     - `m` and `n` are indices for the lookup tables, ensuring they stay within the table size (0-15) using modulo 16.
   - **Value Retrieval**:
     - Retrieves sine and cosine values `p` and `q` from the lookup tables using indices `m` and `n`.
   - **Amplitude Calculation**:
     - Computes the product of amplitude `a`, sine value `p`, and cosine value `q`.
     - Scales down the result `u` to fit within 16-bit signed integer limits by dividing by `2^30` (since we multiplied three 16-bit numbers).
   - **Storing Values**:
     - Calculates the linear index `i` for the 1D grid array.
     - Stores the computed value `v` into the grid array `g` at index `i`.

5. **Displaying the Grid**

   ```mint
   0 y !   // Reset y to 0
   h (
     0 x !  // Reset x to 0
     w (
       y w * x + i !    // Compute linear index i = x + y * w
       g i ? v !        // Retrieve value v from grid array at index i
       v 1000 < ( ` ` ) /E (    // If v < 1000, print a space
         v 20000 > ( `#` ) /E ( `.` )  // If v > 20000, print '#', else print '.'
       )
       1 x + x !        // Increment x
     )
     /N                 // Newline after each row
     1 y + y !          // Increment y
   )
   ```

   - **Loops**: Nested loops iterate over each coordinate `(x, y)` to display the grid.
   - **Value Retrieval**:
     - Retrieves the computed value `v` from the grid array `g`.
   - **Character Mapping**:
     - **Condition 1**: If `v < 1000`, prints a space `' '`.
     - **Condition 2**: Else if `v > 20000`, prints a hash `'#'`.
     - **Else**: Prints a dot `'.'`.
   - **Output**:
     - Characters are printed sequentially to form the grid pattern.
     - A newline character is printed after each row to format the grid properly.

#### **What the Code Is Trying to Do**

- **Simulate Wave Patterns**: The program simulates wave interference patterns on a 2D grid using integer arithmetic.
- **Generate Visual Representation**: By mapping computed amplitude values to ASCII characters, it creates a visual representation of the wave patterns.
- **Use of Lookup Tables**: Precomputed sine and cosine values enable the program to perform trigonometric calculations without floating-point arithmetic, which MINT lacks.
- **Integer Arithmetic**: All calculations are performed using 16-bit signed integers to adhere to MINT's limitations.
- **Display Patterns**: The final output is a grid displaying spaces, dots, and hashes to represent different amplitude levels, effectively visualizing the wave interference.

#### **Notes**

- **Scaling Calculations**: Multiplying three 16-bit integers can result in values exceeding 16 bits. The division by `2^30` scales the result back into the 16-bit range.
- **Memory Management**: The grid size is limited to 16x16 to fit within MINT's 2KB memory constraint.
- **Variable Naming**: All variables are single lowercase letters (`a` to `z`), following MINT's variable naming rules.

### **How to Run the Code**

1. **Enter the Code into MINT**: Input the code exactly as shown into your MINT interpreter.

2. **Execute the Function**: At the MINT prompt, type:

   ```mint
   > P
   ```

3. **View the Output**: The program will display a pattern composed of spaces `' '`, dots `'.'`, and hashes `'#'`.

### **Understanding the Output**

- The pattern represents simulated wave interference on a grid.
- **Spaces** represent low amplitude areas.
- **Dots** represent medium amplitude areas.
- **Hashes** represent high amplitude areas.
- The visual pattern helps in understanding how the sine and cosine values interact across the grid.

### **Potential Modifications**

- **Adjust Frequency (`f`)**: Changing the frequency value will alter the wave patterns.

  ```mint
  3 f !   // Set frequency to 3
  ```

- **Adjust Amplitude (`a`)**: Modifying the amplitude affects the intensity of the patterns.

  ```mint
  20000 a !  // Set amplitude to 20000
  ```

- **Change Grid Size**: If memory allows, you can try smaller or slightly larger grid sizes.

  ```mint
  8 w !   // Set width to 8
  8 h !   // Set height to 8
  ```

- **Modify Character Mapping**: Adjust the threshold values and characters to change how the amplitude values are visualized.

### **Conclusion**

This MINT program demonstrates how to generate and display wave patterns using integer arithmetic and precomputed trigonometric values within the constraints of the MINT Programming Language. By understanding each line and variable, you can modify and extend the code to explore different patterns and behaviors.

 
 
  

### REF 
- https://github.com/SteveJustin1963/Fractansistor
- https://paulbourke.net/geometry/chladni/
- https://scholarscompass.vcu.edu/cgi/viewcontent.cgi?article=1098&context=jmsce_vamsc
- 
