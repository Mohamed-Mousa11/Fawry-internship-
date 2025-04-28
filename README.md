# Fawry-internship
Q1-Answer:

### Step 1: Create the script file
```bash
touch mygrep.sh
chmod +x mygrep.sh  # Make it executable
```

### Step 2: Write the script
```bash
#!/bin/bash

# Initialize variables
show_line_numbers=0
invert_match=0
search_string=""
filename=""

# Handle options
while [[ "$1" == -* ]]; do
    case "$1" in
        -n) show_line_numbers=1; shift ;;
        -v) invert_match=1; shift ;;
        -nv|-vn) show_line_numbers=1; invert_match=1; shift ;;
        --help) 
            echo "Usage: $0 [options] search_string filename"
            echo "Options:"
            echo "  -n    Show line numbers"
            echo "  -v    Invert match"
            exit 0
            ;;
        *) echo "Invalid option: $1" >&2; exit 1 ;;
    esac
done

# Check for required arguments
if [ $# -lt 2 ]; then
    echo "Error: Missing arguments" >&2
    echo "Usage: $0 [options] search_string filename" >&2
    exit 1
fi

search_string="$1"
filename="$2"

# Check if file exists
if [ ! -f "$filename" ]; then
    echo "Error: File '$filename' not found" >&2
    exit 1
fi

# Search through the file
line_number=0
while IFS= read -r line; do
    ((line_number++))
    
    # Case-insensitive match
    if [[ "${line,,}" == *"${search_string,,}"* ]]; then
        match=1
    else
        match=0
    fi
    
    # Handle -v invert flag
    if ((invert_match)); then
        match=$((1 - match))
    fi
    
    # Print if matched
    if ((match)); then
        if ((show_line_numbers)); then
            printf "%d:" "$line_number"
        fi
        printf "%s\n" "$line"
    fi
done < "$filename"
```

### Step 3: Create the test file
```bash
cat > testfile.txt <<EOF
Hello world
This is a test
another test line
HELLO AGAIN
Don't match this line
Testing one two three
EOF
```

### Step 4: Testing

1-
./mygrep.sh hello testfile.txt

![1](https://github.com/user-attachments/assets/22e7f024-6465-41db-9c5f-e1c8644bc5d5)

2-
./mygrep.sh -n hello testfile.txt
![2](https://github.com/user-attachments/assets/ee8420db-01e6-4eac-9201-250e82b0ae23)

3-
./mygrep.sh -vn hello testfile.txt

![3](https://github.com/user-attachments/assets/09c9a775-ecb0-431b-ad15-6a256bd73bff)

4-
./mygrep.sh -v testfile.txt  
![4](https://github.com/user-attachments/assets/e5a873e4-d369-4654-9a13-b69721dfd4b1)

5-
./mygrep.sh --help  # Bonus

![bonus](https://github.com/user-attachments/assets/092d31ec-a02c-4337-b88c-48b0100d4900)


---

### How it works:
1. **Argument handling**: The script first processes options (-n, -v), then checks for required arguments
2. **Case-insensitive search**: Converts both line and search string to lowercase for comparison
3. **Option combinations**: Handles -n, -v, and their combinations
4. **Error handling**: Checks for missing arguments and file existence

### If adding more features:
To support regex or other options (-i, -c, -l), I would:
1. Use `getopts` for cleaner option parsing
2. Add flags for each new feature
3. Modify the matching logic to use `grep -E` for regex support

### Hardest part:
The inverted match logic was tricky because it required negating the match condition while keeping all other functionality working.

### Bonus:
The script already includes `--help` support and could be easily upgraded to use `getopts` for more robust option parsing.
