
```python
#!/usr/bin/env python

# Import necessary modules
import argparse  # For parsing command-line arguments
import dns.name  # For handling DNS domain names

# Define the main function that takes two domain names as arguments
def main(domain1, domain2):
    # Convert the input domain strings into dns.name.Name objects
    domain1 = dns.name.from_text(domain1)
    domain2 = dns.name.from_text(domain2)
    
    # Check if domain1 is a subdomain of domain2 and print the result
    print("domain1 is subdomain of domain2: ", domain1.is_subdomain(domain2)) 
    
    # Check if domain1 is a superdomain of domain2 and print the result
    print("domain1 is superdomain of domain2: ", domain1.is_superdomain(domain2))

# Entry point of the script
if __name__ == '__main__':
    # Set up argument parsing
    parser = argparse.ArgumentParser(description='Check 2 domains with dns Python')
    
    # Define the first command-line argument: --domain1
    parser.add_argument('--domain1', action="store", dest="domain1",  default='python.org')
    
    # Define the second command-line argument: --domain2
    parser.add_argument('--domain2', action="store", dest="domain2",  default='docs.python.org')
    
    # Parse the command-line arguments
    given_args = parser.parse_args() 
    
    # Extract the values of domain1 and domain2 from the parsed arguments
    domain1 = given_args.domain1
    domain2 = given_args.domain2
    
    # Call the main function with the extracted domain names
    main(domain1, domain2)
```

### Explanation:

1. **Shebang Line (`#!/usr/bin/env python`)**:
   - This line specifies that the script should be executed using the Python interpreter found in the user's environment.

2. **Imports**:
   - `argparse`: This module is used to handle command-line arguments.
   - `dns.name`: This module from the `dnspython` library is used to manipulate and compare DNS domain names.

3. **`main` Function**:
   - This function takes two domain names as input.
   - It converts these domain names from strings into `dns.name.Name` objects using `dns.name.from_text()`.
   - It then checks if `domain1` is a subdomain of `domain2` using the `is_subdomain()` method and prints the result.
   - Similarly, it checks if `domain1` is a superdomain of `domain2` using the `is_superdomain()` method and prints the result.

4. **Argument Parsing**:
   - The script uses `argparse.ArgumentParser` to define and parse command-line arguments.
   - Two arguments are defined: `--domain1` and `--domain2`, both of which store their values and have default values (`'python.org'` and `'docs.python.org'`, respectively).

5. **Execution**:
   - The script checks if it is being run as the main program (`if __name__ == '__main__':`).
   - It parses the command-line arguments and extracts the values of `domain1` and `domain2`.
   - Finally, it calls the `main` function with these domain names.

### Example Usage:

If you run the script without any arguments, it will use the default values:

```bash
$ ./check_domains.py
domain1 is subdomain of domain2:  False
domain1 is superdomain of domain2:  True
```

You can also specify custom domains:

```bash
$ ./check_domains.py --domain1 example.com --domain2 sub.example.com
domain1 is subdomain of domain2:  False
domain1 is superdomain of domain2:  True
```

This script is useful for checking the relationship between two domain names, specifically whether one is a subdomain or superdomain of the other.
