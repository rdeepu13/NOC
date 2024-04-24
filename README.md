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
            update_average_latency(latency)
    elif txn_type.startswith("Wr"):  # Check for write transactions
        # Update average latency using write latency
        update_average_latency(write_latency)

# Function to update average latency
function update_average_latency(latency, total_latency, num_transactions):
    total_latency += latency
    num_transactions += 1
    return total_latency, num_transactions

# Main function to process monitor output
function analyze_monitor_output(monitor_output, read_latency, write_latency, cycle_time):
    total_latency = 0
    num_transactions = 0
    total_bytes_transferred = 0

    for entry in monitor_output:
        process_transaction(entry, read_latency, write_latency)
        # Calculate total bytes transferred for bandwidth calculation
        if entry["TxnType"].startswith("Wr") or entry["TxnType"].startswith("Rd"):
            total_bytes_transferred += 32  # Assuming each transaction transfers 32B

    # Calculate average latency
    average_latency = total_latency / num_transactions if num_transactions > 0 else 0

    # Calculate bandwidth
    bandwidth = total_bytes_transferred / cycle_time

    # Print results
    print("Average Latency:", average_latency, "cycles")
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
