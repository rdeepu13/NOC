# Network on Chip (NoC) Simulator Metrics Calculation

This README provides pseudocode for calculating average latency and bandwidth using the output from a Network on Chip (NoC) simulator monitor, as described in the problem statement.

## Pseudocode: Calculate Average Latency and Bandwidth

**Definitions:**
- Latency is calculated for read transactions from the timestamp of a "Read Address" to the timestamp where data for that address is received.
- Bandwidth is calculated as the total number of bytes transferred divided by the total time elapsed.

```plaintext
Initialize:
    readRequests = {}  // Dictionary to store start times of read requests by address
    totalReadLatency = 0
    totalReadCount = 0
    lastTimestamp = 0
    totalBytesTransferred = 0

For each entry in the simulator monitor output:
    timestamp = entry.Timestamp
    txnType = entry.TxnType
    data = entry.Data

    // Update the last timestamp encountered
    Update lastTimestamp to the maximum value of current timestamp

    If txnType starts with "Rd" (Read request):
        // Extract address from the txnType (assuming format "Rd AddrX")
        address = extractAddress(txnType)
        readRequests[address] = timestamp

    Else If txnType starts with "Data" and extractAddress(txnType) in readRequests:
        startTimestamp = readRequests[extractAddress(txnType)]
        latency = timestamp - startTimestamp
        totalReadLatency += latency
        totalReadCount += 1
        // Remove the address from readRequests after processing
        delete readRequests[extractAddress(txnType)]

    If txnType starts with "Wr" (Write) or "Rd" (Read):
        totalBytesTransferred += 32  // Each transaction transfers 32B

Calculate metrics:
    If totalReadCount > 0:
        averageLatency = totalReadLatency / totalReadCount
    Else:
        averageLatency = 0

    If lastTimestamp > 0:
        bandwidth = (totalBytesTransferred / lastTimestamp) * frequency of the NOC
    Else:
        bandwidth = 0

Output:
    Print "Average Latency: ", averageLatency, " cycles"
    Print "Bandwidth: ", bandwidth, " bytes/cycle"
