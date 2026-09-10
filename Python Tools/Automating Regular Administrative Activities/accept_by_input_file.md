
```python
# File Handling in Python: Reading from One File and Writing to Another

# Open the input file 'sample.txt' in read mode ('r')
i = open('sample.txt', 'r')

# Open the output file 'sample_output.txt' in write mode ('w')
o = open('sample_output.txt', 'w')

# Read the entire content of 'sample.txt'
a = i.read()

# Write the content to 'sample_output.txt'
o.write(a)

# Close both files to free resources
i.close()
o.close()
```

### Explanation:
1. `open('sample.txt', 'r')`: Opens the file `sample.txt` in **read mode**.
2. `open('sample_output.txt', 'w')`: Opens (or creates) `sample_output.txt` in **write mode**.
3. `i.read()`: Reads the entire content of `sample.txt` and stores it in variable `a`.
4. `o.write(a)`: Writes the content stored in `a` to `sample_output.txt`.
5. `i.close()`: Closes the input file.
6. `o.close()`: Closes the output file.

### How to Run:
1. Make sure `sample.txt` exists in the same directory as your Python script.
2. Save the above code as `copy_file.py`.
3. Open a terminal or command prompt.
4. Run the script using:
   ```
   python copy_file.py
   ```
5. The content of `sample.txt` will be copied to `sample_output.txt`.

#### Best Practice:
Instead of manually closing files, use the `with` statement, which automatically handles closing:
```python
# Using 'with' for automatic file closing
with open('sample.txt', 'r') as i, open('sample_output.txt', 'w') as o:
    o.write(i.read())  # Read from input file and write to output file
```
