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


______________________________________________________________________________________________________________________________________
Q2 : Answer



### **Step 1: Verify DNS Resolution**
#### **1. Check current DNS servers**  
```bash
cat /etc/resolv.conf
```


![1](https://github.com/user-attachments/assets/d687dcbc-2f9a-45e7-8f8d-edecccd44d62)


#### **2. Test DNS resolution**  
```bash
nslookup internal.example.com          # Using local DNS
nslookup internal.example.com 8.8.8.8  # Using Google DNS
```

![2](https://github.com/user-attachments/assets/ab45f343-d6da-44e3-9be9-fcf469cb8eac)

---

### **Step 2: Diagnose Service Reachability**
#### **1. Get the IP address**  
```bash
dig +short internal.example.com
# Or use the IP directly if known (e.g., 192.168.1.100)
IP=192.168.1.100  # Replace with actual IP
```
Show blank as unresolved  
![3](https://github.com/user-attachments/assets/e2d06980-d135-4df6-be76-7921b8d8bc29)


#### **2. Test HTTP/HTTPS connectivity**  
```bash
curl -v http://$IP              # Check HTTP (port 80)
curl -vk https://$IP            # Check HTTPS (port 443, ignore cert errors)
telnet $IP 80                   # Test raw TCP connection
```
output example:  
- `curl: (7) Failed to connect` → Network/firewall issue.  
- `HTTP/1.1 200 OK` → Service is reachable.  

#### **3. Check if the service is running locally**  
```bash
sudo ss -tulnp | grep -E ':80|:443'  # Modern alternative to netstat
```
output example:  
- Empty output → Service not running.  
- `nginx/apache2` listed → Service is up but may be blocked.  

---

### **Step 3: Trace the Issue (Common Causes & Fixes)**
#### **1. DNS Misconfiguration**  
**Confirm**:  
```bash
ping internal.example.com  # Fails if DNS is broken
```  
**Fix**:  
```bash
# Temporary fix (edit resolv.conf)
echo "nameserver 192.168.1.1" | sudo tee /etc/resolv.conf
```  


#### **2. Firewall Blocking**  
**Confirm**:  
```bash
sudo ufw status  # Check if UFW is active
iptables -L -n   # Check for blocked ports
```  
**Fix**:  
```bash
sudo ufw allow 80/tcp   # Allow HTTP
sudo ufw allow 443/tcp  # Allow HTTPS
```  
 

#### **3. Service Not Running**  
**Confirm**:  
```bash
systemctl status apache2  # Or nginx
```  
**Fix**:  
```bash
sudo systemctl start apache2
```  


#### **4. /etc/hosts Override (Bonus)**  
**Add a manual entry**:  
```bash
echo "192.168.1.100 internal.example.com" | sudo tee -a /etc/hosts
```  
**Verify**:  
```bash
ping -c 2 internal.example.com
```  

---

### **Step 4: Persist DNS Settings (Bonus)**  
#### **Using `systemd-resolved` (Ubuntu default)**:  
```bash
sudo systemctl restart systemd-resolved
sudo ln -sf /run/systemd/resolve/resolv.conf /etc/resolv.conf
```  
 

#### **Using `NetworkManager` (GUI)**:  
```bash
nmcli con show | grep -i ethernet  # Find connection name
nmcli con mod "Wired Connection 1" ipv4.dns "192.168.1.1"
nmcli con up "Wired Connection 1"
```  





