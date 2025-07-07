This GCS application has a fully designed and implemented multi-threaded architecture that achieves the following:

✅ Multi-Threaded Parallel Processing: The system uses Python's threading module to run multiple critical processes concurrently, ensuring that no single task blocks another.

✅ Autonomous Drone Control: Each drone is managed by its own dedicated DroneController thread, which handles low-level MAVSDK commands, mission execution, and control of the payload delivery mechanism (dual servos).

✅ Dynamic Mission Planning: A central MissionPlanner thread acts as the strategic brain. It ingests data, makes decisions, and dispatches high-level commands to the drone controllers. Its logic includes:

Area Division: Splitting the main operational area for efficient parallel scouting.

Emergency Tasking: Identifying the four most critical targets post-scouting and assigning them greedily to the nearest drones.

Clustering for Efficiency: Using K-Means clustering to group remaining survivors and assign them to drones for optimized, multi-stop delivery routes.

✅ Real-time Safety Monitoring: A dedicated BatteryMonitor thread runs independently, constantly checking the battery levels of all drones and triggering a Return-to-Launch (RTL) command if any drone reaches a critical threshold (e.g., 25%), overriding its current task to ensure vehicle safety.

✅ Asynchronous Data Handling: An independent DetectionListener thread is designed to watch for incoming survivor location data (as KML files), parsing them and feeding them into a shared queue for the planner to process.

Role of Each Thread:

main_gcs.py (The Conductor): Sets up the shared resources (queues, state variables) and starts all other threads.

MissionPlanner (Thread 3): Decides what to do. It analyzes the overall situation and sends high-level commands like "deliver_payloads_to_cluster_A".

DroneController (Threads 4 & 5): Knows how to do it. It receives commands from the planner and translates them into specific MAVSDK actions (goto_location, set_actuator, etc.).

BatteryMonitor (Thread 1): The safety override. It can interrupt any other process by sending an RTL command directly to a drone.

DetectionListener (Thread 2): The data pipeline. It ensures that new information from the field is consistently made available to the planner.

Technology Stack

Language: Python 3.9+
Drone Control: MAVSDK-Python
Concurrency: threading, asyncio, queue
Data Science / AI:
scikit-learn: For K-Means clustering of survivor locations.
numpy: For efficient numerical operations.
OpenCV (for the computer vision module on the drone)
Geospatial Data: fastkml for parsing survivor coordinates.
