# Network on Chip (NoC) Simulator Metrics Calculation

This README provides pseudocode for calculating average latency and bandwidth using the output from a Network on Chip (NoC) simulator monitor, as described in the problem statement.

## Pseudocode: Calculate Average Latency and Bandwidth

**Definitions:**
- Latency is calculated for read transactions from the timestamp of a "Read Address" to the timestamp where data for that address is received.
- Bandwidth is calculated as the total number of bytes transferred divided by the total time elapsed.

```plaintext
# Function to process a single transaction entry
function process_transaction(entry, read_latency, write_latency):
    # Extract relevant information
    timestamp = entry["Timestamp"]
    txn_type = entry["TxnType"]
    data = entry.get("Data")  # Retrieve data if applicable

    # Update latency based on transaction type
    if txn_type.startswith("Rd"):  # Check for read transactions
        # Find corresponding data entry (robustness)
        data_entry = find_data_entry(timestamp + read_latency)
        if data_entry is not None:
            latency = data_entry["Timestamp"] - timestamp
            update_average_latency(latency, txn_type)
    elif txn_type.startswith("Wr"):  # Check for write transactions
        # Update average latency using write latency
        update_average_latency(write_latency, txn_type)

# Function to update average latency
function update_average_latency(latency, txn_type):
    if txn_type.startswith("Rd"):
        global total_read_latency
        global num_read_transactions
        total_read_latency += latency
        num_read_transactions += 1
    elif txn_type.startswith("Wr"):
        global total_write_latency
        global num_write_transactions
        total_write_latency += latency
        num_write_transactions += 1

# Main function to process monitor output
function analyze_monitor_output(monitor_output, read_latency, write_latency, cycle_time):
    global total_read_latency
    global num_read_transactions
    global total_write_latency
    global num_write_transactions

    total_read_latency = 0
    num_read_transactions = 0
    total_write_latency = 0
    num_write_transactions = 0
    total_bytes_transferred = 0

    for entry in monitor_output:
        process_transaction(entry, read_latency, write_latency)
        # Calculate total bytes transferred for bandwidth calculation
        if entry["TxnType"].startswith("Wr") or entry["TxnType"].startswith("Rd"):
            total_bytes_transferred += 32  # Assuming each transaction transfers 32B

    # Calculate average read latency
    average_read_latency = total_read_latency / num_read_transactions if num_read_transactions > 0 else 0

    # Calculate average write latency
    average_write_latency = total_write_latency / num_write_transactions if num_write_transactions > 0 else 0

    # Calculate bandwidth
    bandwidth = total_bytes_transferred / cycle_time

    # Print results
    print("Average Read Latency:", average_read_latency, "cycles")
    print("Average Write Latency:", average_write_latency, "cycles")
    print("Bandwidth:", bandwidth, "Bytes/sec")

# Sample monitor output data
monitor_output = [
    {"Timestamp": 0, "TxnType": "Rd Addr1", "Data": None},
    {"Timestamp": 2, "TxnType": "Wr Addr2", "Data": "hxxxxxxxx"},
    {"Timestamp": 4, "TxnType": "Wr Addr3", "Data": "hyyyyyyyy"},
    {"Timestamp": 10, "TxnType": "Data Addr1", "Data": "hzzzzzzzz"},
    # Add more entries as needed
]

# Call the main function with monitor output data and other parameters
analyze_monitor_output(monitor_output, read_latency=10, write_latency=5, cycle_time=100000)  # Assuming read and write latency and cycle time
```plaintext
Complexity Analysis
Time Complexity:
Processing a single transaction entry (process_transaction function) involves iterating through the monitor output once, resulting in a time complexity of O(n), where n is the number of entries in the monitor output.
Finding the corresponding data entry (find_data_entry function) within the monitor output also has a time complexity of O(n) in the worst case.
Overall, the time complexity for processing the entire monitor output and calculating average latency and bandwidth is O(n).

Space Complexity:
The space complexity primarily depends on the size of additional data structures used for storing intermediate results.
In this pseudocode, the primary additional space is used for storing total read and write latencies and the number of read and write transactions, which have a constant space requirement.
Therefore, the space complexity is O(1), indicating constant space usage regardless of the size of the monitor output.
