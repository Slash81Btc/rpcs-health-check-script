# Health Check Script for t3rn, Arbitrum, Base, Blast, and OP RPC Endpoints

This shell script performs a health check for multiple Ethereum RPC endpoints (t3rn, Arbitrum, Base, Blast, and OP) by making `eth_blockNumber` RPC calls to each endpoint. It tracks and prints detailed timing for each call, including the request start time, response time, and block number. Additionally, it indicates whether the call was a success or failure.

## Setup Shell Script

Follow these steps to set up the shell script:

1. **Open a terminal.**
2. **Create a new file, for example, `rpcs_health_check.sh`:**
    ```bash
    nano rpcs_health_check.sh
    ```
3. **In the script file, add the following content** (Ensure you **update the RPC endpoints** according to your needs):

    ```bash
    #!/bin/bash

    # Function to get timestamp with microseconds
    get_timestamp() {
        date +"%Y-%m-%d %H:%M:%S.%6N"
    }

    # Function to make RPC call with detailed timing
    make_rpc_call() {
        local endpoint=$1
        local start_timestamp=$(get_timestamp)
        echo -e "\033[1;34m[START]\033[0m${start_timestamp} - Making call to $endpoint"

        local start_time=$(date +%s.%N)
        response=$(curl -s -X POST -H "Content-Type: application/json" --data '{
            "jsonrpc":"2.0",
            "method":"eth_blockNumber",
            "params":[],
            "id":1
        }' "$endpoint")
        local end_time=$(date +%s.%N)
        local duration=$(echo "$end_time - $start_time" | bc)

        local block_hex=$(echo $response | grep -o '"result":"[^"]*"' | cut -d'"' -f4)
        if [ ! -z "$block_hex" ]; then
            local block_dec=$((16#${block_hex#0x}))
            local success_timestamp=$(get_timestamp)
            echo -e "\033[1;32m✅ [SUCCESS]\033[0m${success_timestamp}"
            echo "   Endpoint: $endpoint"
            echo "   Block Number: $block_dec"
            printf "   Response Time: %.3f seconds\n" $duration
        else
            local failure_timestamp=$(get_timestamp)
            echo -e "\033[1;31m❌ [FAILED]\033[0m${failure_timestamp}"
            echo "   Endpoint: $endpoint"
        fi
        echo "--------------------------------------------------"
    }

    # Print script start time
    echo "RPCs Health Check Started at $(get_timestamp)"
    echo "--------------------------------------------------"

    # List of RPC endpoints
    endpoints=(
        "https://brn.rpc.caldera.xyz" # L1RN RPC URL
        "https://brn.calderarpc.com/" # Alternative L1RN RPC URL
        "0000000000000000000000000000000000000000000000000000000000000000"  # Update with your Arbitrum Sepolia RPC URL
        "0000000000000000000000000000000000000000000000000000000000000000"  # Update with your Base Sepolia RPC URL
        "0000000000000000000000000000000000000000000000000000000000000000"  # Update with your Blast Sepolia RPC URL
        "0000000000000000000000000000000000000000000000000000000000000000"  # Update with your OP Sepolia RPC URL
    )

    # Make RPC calls sequentially
    for endpoint in "${endpoints[@]}"; do
        make_rpc_call "$endpoint" &
    done

    # Wait for all background processes to complete
    wait

    # Print script end time
    echo "All RPC calls completed at $(get_timestamp)"
    ```

4. **Make the shell script executable:**
    ```bash
    sudo chmod +x rpcs_health_check.sh
    ```

5. **Start the script:**
    ```bash
    ./rpcs_health_check.sh
    ```

## How It Works

1. The script defines two functions:
    - **`get_timestamp`**: Retrieves the current timestamp with microseconds.
    - **`make_rpc_call`**: Makes an `eth_blockNumber` RPC call to an endpoint and tracks the duration. It also parses the block number from the response and prints the results.

2. For each endpoint:
    - The script outputs the start time.
    - It sends an `eth_blockNumber` request.
    - If successful, it prints the block number, response time, and a success message.
    - If the call fails, it prints a failure message.

3. The script runs the RPC calls in parallel using background processes (`&`) and waits for all processes to complete before printing the end time.
