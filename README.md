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
```


### Complexity Analysis
**Time Complexity:**
Processing a single transaction entry (process_transaction function) involves iterating through the monitor output once, resulting in a time complexity of O(n), where n is the number of entries in the monitor output.
Finding the corresponding data entry (find_data_entry function) within the monitor output also has a time complexity of O(n) in the worst case.
Overall, the time complexity for processing the entire monitor output and calculating average latency and bandwidth is O(n).

**Space Complexity:**
The space complexity primarily depends on the size of additional data structures used for storing intermediate results.
In this pseudocode, the primary additional space is used for storing total read and write latencies and the number of read and write transactions, which have a constant space requirement.
Therefore, the space complexity is O(1), indicating constant space usage regardless of the size of the monitor output.


## Describing the RL framework consisting of states/behaviors, actions and rewards. 

```plaintext
# Define Actor and Critic neural networks
Initialize actor_network and critic_network with random weights

# Define learning rate, discount factor, and other hyperparameters
learning_rate_actor = 0.001
learning_rate_critic = 0.001
discount_factor = 0.99

# Initialize state, action, and reward buffers
state_buffer = []
action_buffer = []
reward_buffer = []

# Define exploration strategy
epsilon = 0.1

# Main training loop
for episode in range(num_episodes):
    # Reset environment and initialize state
    state = initialize_environment()

    # Reset episode-specific buffers
    episode_states = []
    episode_actions = []
    episode_rewards = []

    # Interaction loop
    while not environment_terminated:
        # Choose action using epsilon-greedy policy
        if random() < epsilon:
            action = random_action()
        else:
            action = actor_network.predict(state)

        # Take action and observe next state and reward
        next_state, reward, environment_terminated = take_action(action)
        
        # Store transition in episode buffers
        episode_states.append(state)
        episode_actions.append(action)
        episode_rewards.append(reward)

        # Update current state
        state = next_state

    # Calculate discounted returns (reward-to-go)
    returns = calculate_returns(episode_rewards)

    # Update actor and critic networks using episode buffers
    update_actor_critic(actor_network, critic_network, episode_states, episode_actions, returns)

# Define functions for actor and critic updates
function update_actor_critic(actor_network, critic_network, states, actions, returns):
    for i in range(len(states)):
        state = states[i]
        action = actions[i]
        return_i = returns[i]

        # Compute TD-error for critic update
        critic_target = return_i
        critic_prediction = critic_network.predict(state)
        td_error = critic_target - critic_prediction

        # Update critic network
        critic_network.update_parameters(state, critic_target)

        # Compute advantages for actor update
        advantages = returns - critic_prediction

        # Update actor network
        actor_network.update_parameters(state, action, advantages)

# Function to initialize the environment for each episode
function initialize_environment():
    # Reset the state of the environment to its initial state
    # This may involve resetting buffer sizes, arbitration weights, etc.
    return initial_state

# Function to take action and observe next state and reward
function take_action(action):
    # Apply the chosen action to the environment and observe the effects
    # This may involve adjusting buffer sizes, arbitration weights, etc.
    next_state = environment.get_next_state_after_action(action)
    reward = environment.get_reward_for_action(action)
    environment_terminated = environment.is_episode_finished()
    return next_state, reward, environment_terminated

# Function to calculate the returns (reward-to-go) for each time step
function calculate_returns(rewards):
    # Calculate the discounted sum of rewards from each time step to the end of the episode
    # This may involve applying a discount factor and summing up the rewards
    returns = []
    cumulative_reward = 0
    for reward in reversed(rewards):
        cumulative_reward = reward + discount_factor * cumulative_reward
        returns.insert(0, cumulative_reward)
    return returns

# Function to choose a random action 
function random_action():
    # Choose a random action from the action space
    return random_action_from_action_space

```

### Complexity Analysis

**Time Complexity:** 
- The time complexity of the main training loop depends on the number of episodes (num_episodes) and the length of each episode. Let's denote the length of each episode as T. 
- Within each episode, the time complexity is determined by the number of interactions with the environment, which involves selecting actions, taking actions, and updating the actor and critic networks. 
- Assuming the time complexity of these operations is O(1), the overall time complexity of the main training loop is O(num_episodes * T). 
- Additionally, the time complexity of updating the actor and critic networks depends on the size of the neural networks and the number of parameters being updated. Let's denote this as O(N), where N represents the total number of parameters in the networks.
  
**Space Complexity:**
- The space complexity primarily depends on the storage required for maintaining the state, action, and reward buffers and the neural network parameters.
- Assuming the size of these buffers is proportional to the length of each episode (T) and the size of the neural networks is proportional to the number of parameters (N), the overall space complexity can be expressed as O(T + N). 

