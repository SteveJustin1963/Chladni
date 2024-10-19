# Chladni

### **Pseudocode**

#### **1. PI Approximation**

- **Issue:** The value of `PI_16` is set to `20589`, which is supposed to be an approximation of π scaled to fit the 16-bit range (`PI * 2^13`).
- **Correction:** Calculate the correct scaled value.
  
  \[
  \text{PI\_16} = \pi \times 2^{13} \approx 3.1416 \times 8192 \approx 25736
  \]
  
- **Action:** Update `PI_16` to `25736`.

#### **2. Trigonometric Lookup Tables**

- **Issue:** The scaling factor used in `populate_trig_tables` is `(2^15 - 1)`, which is `32767`. When multiplying multiple 16-bit integers, this can cause overflow.
- **Correction:** Adjust the scaling to prevent overflow. Use a smaller scaling factor or adjust the arithmetic to accommodate the larger intermediate results.
- **Action:** Use fixed-point arithmetic with proper scaling (Q15 format). When multiplying two Q15 numbers, the result is in Q30 format, so we need to shift right by 15 bits to keep it in Q15.

#### **3. Grid Value Calculations**

- **Issue:** Multiplying three 16-bit signed integers (`grid[x][y]`, `SIN_LOOKUP[angle_x]`, and `COS_LOOKUP[angle_y]`) can result in values exceeding 16 bits.
- **Correction:** Break down the multiplication into steps and include right shifts to maintain values within 16 bits.
- **Action:** Perform sequential multiplications with appropriate right shifts after each multiplication.

#### **4. Threshold Comparison**

- **Issue:** The comparison `grid[x][y] < threshold_value` doesn't account for negative values that may result from the sine and cosine multiplications.
- **Correction:** Use the absolute value of `grid[x][y]` when comparing to the threshold.
- **Action:** Update the condition to `ABS(grid[x][y]) < threshold_value`.

#### **5. ASCII Bar Graph Scaling**

- **Issue:** The `scale_factor` in `display_ascii_bar_graph` may not accurately represent the scaled values of `grid[x][y]`.
- **Correction:** Adjust `scale_factor` based on the new scaling of sine and cosine values.
- **Action:** Recalculate `scale_factor` and adjust the amplitude ranges in the ASCII representation accordingly.

---

### **  Pseudocode**

```plaintext
// Define grid with 16-bit signed integers
DEFINE grid[WIDTH][HEIGHT] AS matrix to hold vibration amplitudes (16-bit signed)

// Initialize parameters for vibration
SET frequency = initial_frequency (16-bit signed)
SET amplitude = initial_amplitude (16-bit signed)
SET fractal_pattern AS predefined 16-bit signed fractal array (e.g., scaled Mandelbrot set)

// Define constants for integer-based trigonometric calculations
SET PI_16 = 25736 // Approximation of PI scaled to fit Q13 format (PI * 2^13)

// Initialize sine and cosine lookup tables
DEFINE SIN_LOOKUP[0..359] AS array of 16-bit signed integers
DEFINE COS_LOOKUP[0..359] AS array of 16-bit signed integers
CALL populate_trig_tables(SIN_LOOKUP, COS_LOOKUP) // Populate with scaled values

// Loop through each grid cell to apply fractal properties
FOR x FROM 0 TO WIDTH - 1
    FOR y FROM 0 TO HEIGHT - 1
        // Apply fractal function to generate base vibration pattern (returns 16-bit signed value)
        grid[x][y] = FRACTAL_FUNCTION(x, y, fractal_pattern)

        // Modify grid values based on scaled Chladni equations using lookup tables
        SET angle_x = (frequency * x) % 360 // Ensure angle stays within 0-359 degrees
        SET angle_y = (frequency * y) % 360

        // Perform fixed-point multiplication with proper scaling
        temp = (grid[x][y] * SIN_LOOKUP[angle_x]) >> 15 // Q15 * Q15 -> Q30, shift to Q15
        grid[x][y] = (temp * COS_LOOKUP[angle_y]) >> 15 // Q15 * Q15 -> Q30, shift to Q15
    END FOR
END FOR

// Display Chladni patterns using ASCII bar graph
CALL display_ascii_bar_graph(grid)

// Analyze Chladni nodes to determine transistor behavior
DEFINE nodes AS LIST of (x, y) coordinates
FOR x FROM 0 TO WIDTH - 1
    FOR y FROM 0 TO HEIGHT - 1
        IF ABS(grid[x][y]) < threshold_value THEN
            ADD (x, y) TO nodes
        END IF
    END FOR
END FOR

// Process nodes to simulate how transistor properties are affected
FOR EACH node IN nodes
    // Calculate conductivity or other properties using integer math
    transistor_property = CALCULATE_PROPERTY(node, fractal_pattern) // Returns 16-bit signed value
    OUTPUT transistor_property
END FOR

// Adjust frequency and amplitude to explore different modes
WHILE frequency < max_frequency (16-bit signed)
    INCREMENT frequency by small value
    // Optionally adjust amplitude
    REPEAT entire process
END WHILE

END

// Function to populate sine and cosine lookup tables
FUNCTION populate_trig_tables(SIN_LOOKUP, COS_LOOKUP):
    FOR angle FROM 0 TO 359
        // Scale sine and cosine values to fit Q15 format (-32768 to +32767)
        SIN_LOOKUP[angle] = SIN(angle * PI / 180) * 32768 // Scale sine value
        COS_LOOKUP[angle] = COS(angle * PI / 180) * 32768 // Scale cosine value
    END FOR
END FUNCTION

// Function to display the grid as an ASCII bar graph
FUNCTION display_ascii_bar_graph(grid):
    DEFINE scale_factor = 32768 / 5 // Divide range into 5 levels for ASCII representation
    FOR y FROM 0 TO HEIGHT - 1
        FOR x FROM 0 TO WIDTH - 1
            // Determine ASCII character based on grid value scaled to 5 levels
            SET value = grid[x][y]
            IF value < -scale_factor * 2 THEN
                PRINT " " // Very low amplitude (space)
            ELSE IF value >= -scale_factor * 2 AND value < -scale_factor THEN
                PRINT "-" // Low amplitude
            ELSE IF value >= -scale_factor AND value < 0 THEN
                PRINT "=" // Medium-low amplitude
            ELSE IF value >= 0 AND value < scale_factor THEN
                PRINT "#" // Medium-high amplitude
            ELSE IF value >= scale_factor THEN
                PRINT "@" // High amplitude
            END IF
        END FOR
        PRINT NEW_LINE // Move to next row in the graph
    END FOR
END FUNCTION
```

---

 

#### **1. Corrected PI Approximation**

- **Updated Value:** `PI_16` is now set to `25736`, accurately representing π scaled to `2^13`.

#### **2. Fixed-Point Arithmetic in Trigonometric Tables**

- **Scaling Factor:** Changed to `32768` to use Q15 fixed-point format, allowing sine and cosine values to range from `-32768` to `+32767`.
- **Benefit:** Prevents overflow when performing multiplications, as shifting right by 15 bits keeps the result within 16 bits.

#### **3. Adjusted Grid Calculations**

- **Sequential Multiplication and Shifting:**
  - First Multiplication:
    \[
    \text{temp} = \frac{\text{grid}[x][y] \times \text{SIN\_LOOKUP}[angle\_x]}{2^{15}}
    \]
  - Second Multiplication:
    \[
    \text{grid}[x][y] = \frac{\text{temp} \times \text{COS\_LOOKUP}[angle\_y]}{2^{15}}
    \]
- **Explanation:** By shifting right by 15 bits after each multiplication, we keep the values within the 16-bit signed integer range.

#### **4. Absolute Value in Threshold Comparison**

- **Update:** Used `ABS(grid[x][y])` to ensure that both positive and negative amplitudes are considered when detecting nodes.
- **Reasoning:** Since sine and cosine functions can produce negative values, taking the absolute value allows for consistent node detection.

#### **5. Adjusted ASCII Bar Graph Scaling**

- **Scaling Factor:** Recalculated `scale_factor` as `32768 / 5` to match the new range of `grid[x][y]`.
- **Amplitude Ranges:**
  - Very Low Amplitude: `value < -scale_factor * 2`
  - Low Amplitude: `-scale_factor * 2 ≤ value < -scale_factor`
  - Medium-Low Amplitude: `-scale_factor ≤ value < 0`
  - Medium-High Amplitude: `0 ≤ value < scale_factor`
  - High Amplitude: `value ≥ scale_factor`
- **Benefit:** Provides a more accurate visual representation of the amplitude variations in the grid.

---

### **Additional Notes**

- **Data Types and Overflow Prevention:**
  - **Intermediate Variables:** Use 32-bit integers for intermediate calculations to prevent overflow before shifting.
  - **Consistency:** Ensure that all variables used in calculations are of the appropriate data type.

- **Frequency and Angle Calculations:**
  - **Overflow Check:** When calculating `angle_x` and `angle_y`, ensure that `frequency * x` and `frequency * y` do not exceed 32-bit integer limits.
  - **Modulo Operation:** Using `% 360` keeps angles within the valid range for the lookup tables.

- **Function Definitions:**
  - **FRACTAL_FUNCTION:** Should return a 16-bit signed integer based on the fractal pattern.
  - **CALCULATE_PROPERTY:** Must be defined to simulate how the nodes affect transistor properties.

- **Loop Structure:**
  - **Frequency Adjustment Loop:** Added a comment to optionally adjust amplitude alongside frequency in the exploration loop.
  - **Process Repetition:** Clearly indicated that the entire process is repeated when frequency is incremented.

---


 

### **Next Steps**

- **Implementation:** Translate the pseudocode into the target programming language, ensuring that all data types and operations are correctly defined.
- **Testing:** Validate the code with known inputs and compare the output to expected results to ensure correctness.
- **Optimization:** If necessary, optimize the code for performance or memory usage, especially if running on hardware with strict limitations.
- **Visualization:** Enhance the `display_ascii_bar_graph` function or implement additional visualization methods for better representation of the patterns.

Feel free to ask if you need further assistance with any part of this process!



### REF 
- https://github.com/SteveJustin1963/Fractansistor
- https://paulbourke.net/geometry/chladni/
- https://scholarscompass.vcu.edu/cgi/viewcontent.cgi?article=1098&context=jmsce_vamsc
- 
