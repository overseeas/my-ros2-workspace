
# Analysis of ROS 2 QoS Policies: Reliability and History

## 1. Introduction
In ROS 2, **Quality of Service (QoS)** settings allow developers to tune the communication between nodes based on network conditions and data importance. This report documents an empirical analysis of how different QoS profiles affect data transmission and system behavior.

## 2. Theoretical Background

### A. Reliability
- **Best Effort**: Provides non-guaranteed message delivery. It focuses on low latency by not attempting to retransmit lost packets.
- **Reliable**: Ensures that messages are delivered successfully. The middleware will retransmit data until the subscriber acknowledges receipt, which may increase latency in poor network conditions.

### B. History & Depth
- **History (Keep Last)**: Only stores up to a specified number of samples (Depth).
- **History (Keep All)**: Storing all samples until the resource limits of the underlying middleware are reached.
- **Depth**: A queue size that determines how many messages are buffered before the oldest ones are discarded.

## 3. Experimental Test Cases

| Scenario | Reliability | History | Depth | Ideal Use Case |
| :--- | :--- | :--- | :--- | :--- |
| **Case 1** | **Best Effort** | Keep Last | 1 | Sensor streams (LiDAR, IMU) where the latest data is prioritized. |
| **Case 2** | **Reliable** | Keep Last | 10 | Control commands (cmd_vel) where losing a command could be critical. |
| **Case 3** | **Reliable** | Keep All | N/A | Configuration changes or system-critical state transitions. |

## 4. Observations and Analysis

### 4.1 Throughput vs. Latency
- **Observation**: Under high-load scenarios, **Reliable** settings showed an increase in end-to-end latency compared to **Best Effort**. This is due to the overhead of ACKs (Acknowledgments) and potential retransmissions at the DDS layer.
- **Insight**: For high-bandwidth data like Camera images, *Best Effort* is preferred to prevent "Head-of-Line Blocking" where a single lost packet stalls the entire stream.

### 4.2 Queue Overflow Behavior
- **Observation**: With **Keep Last** and a **Depth of 1**, only the most recent message was processed by the subscriber even if the publisher sent messages at a higher frequency.
- **Insight**: A small Depth is effective for maintaining real-time performance in navigation tasks, while a larger Depth is necessary for data logging or sequential state updates.

### 4.3 QoS Compatibility (Mismatch)
- During the experiment, I verified that a **Reliable Subscriber** cannot receive data from a **Best Effort Publisher**. 
- **Middleware Rule**: The "Offered" reliability (Publisher) must be greater than or equal to the "Requested" reliability (Subscriber). 
- *Offered (Best Effort) < Requested (Reliable) → **Incompatible***

## 5. Conclusion
Selecting appropriate QoS policies is a trade-off between **reliability, latency, and resource consumption**. As a middleware engineer, I recommend using *Best Effort* for high-frequency state updates and *Reliable* for critical control loops and one-time event notifications.

---
*This analysis was conducted as part of a ROS 2 middleware performance study.*
