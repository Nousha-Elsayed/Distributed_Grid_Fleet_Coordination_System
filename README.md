# Distributed Grid Fleet Coordination System (Simulation Only) 🚗🤖

## 1️⃣ Project Overview

This project is a **distributed multi-robot coordination system** implemented in **ROS1**, designed for simulation on a logical grid.  

The system simulates:

* An **8x8 grid world**
* **3 autonomous vehicles**
* **1 Task Manager**
* **1 Traffic Controller**
* **1 Monitor Node**

**Total Nodes:** 6  

**Movement:** Logical only (numbers, no sensors, cameras, or Gazebo).  

**Core Idea:**  
Each vehicle requests a delivery task, moves step-by-step on the grid, requests permission before entering a cell, avoids collisions, completes the task, and then requests a new task.

---

## 2️⃣ Core Idea / Vehicle Workflow

Each vehicle performs the following sequence:

1. Requests a delivery task from the **Task Manager**.  
2. Receives a task with:
   * Pickup location `(x, y)`
   * Drop-off location `(x, y)`  
3. Moves **step-by-step** on the grid.  
4. Requests **move permission** from the **Traffic Controller** before entering a cell.  
5. Avoids collisions with other vehicles.  
6. Completes the task and reports back.  
7. Requests a **new task** until tasks are finished.

---

## 3️⃣ Required Nodes

### 3.1 Vehicle Node (3 instances)

Each vehicle node must:

* Publish:
  * Current position (`/vehicle_position`)
  * Current state (`/vehicle_state`)
* Request task via service (`/request_task`)
* Request move permission via service (`/request_move`)  
* Implement an internal **state machine**:

**States:**

* IDLE
* REQUEST_TASK
* MOVING_TO_PICKUP
* MOVING_TO_DROPOFF
* WAITING
* FINISHED

**Note:** Each vehicle runs as a **separate node instance**.

---

### 3.2 Task Manager Node

Responsibilities:

* Generate **at least 10 tasks**
* Store tasks in a list
* Provide service `/request_task`:
  * Assign **one task at a time**
  * Ensure **no task is assigned twice**
  * Keep count of **completed tasks**

---

### 3.3 Traffic Controller Node (Critical)

Responsibilities:

* Subscribe to `/vehicle_position`
* Track **occupied cells**
* Provide service `/request_move`:
  * Approve or reject movement
  * Prevent **two vehicles in the same cell**
  * Prevent **direct position swap** (deadlock)
* Detect vehicles **waiting too long**
* Resolve deadlocks using **priority rules**

---

### 3.4 Monitor Node

Responsibilities:

* Subscribe to `/vehicle_state`
* Print:
  * Active tasks
  * Waiting vehicles
  * Completed tasks
* Detect if a vehicle is **stuck for more than 10 seconds**

---

## 4️⃣ System Rules

* **Grid size:** 8x8
* **Movement allowed:** Up, Down, Left, Right
* **Movement per step:** 1 cell only
* Movement must be **gradual**, with a small **delay** between steps

---

## 5️⃣ Required ROS Communication

**Topics:**

* `/vehicle_position` – Published by vehicles  
* `/vehicle_state` – Published by vehicles  

**Services:**

* `/request_task` – Provided by Task Manager  
* `/request_move` – Provided by Traffic Controller  

You may define **custom messages** or use multiple simple fields for communication.

---

## 6️⃣ Main Objectives

The system must handle:

A. **Collision Prevention** – Two vehicles must **never occupy the same cell**  
B. **Deadlock Prevention** – Example:  
   * Vehicle 1 at (2,3) wants (2,4)  
   * Vehicle 2 at (2,4) wants (2,3)  
   * The system prevents **infinite waiting**  
C. **Simultaneous Requests** – Only one vehicle receives a task if requested simultaneously  
D. **Fairness** – No vehicle should **starve** while others complete tasks

---

## 7️⃣ Team Division (Mandatory)

Each student is responsible for **one node**:

* **Student 1** – Vehicle A  
* **Student 2** – Vehicle B  
* **Student 3** – Vehicle C  
* **Student 4** – Task Manager  
* **Student 5** – Traffic Controller + Monitor  

> Each student must explain their node **line by line** in the video demonstration.

---

## 8️⃣ Execution Method

Because **launch files are not used**, nodes must be started **manually** in separate terminals:

```bash
roscore
rosrun your_package task_manager
rosrun your_package traffic_controller
rosrun your_package monitor
rosrun your_package vehicle_node vehicle1
rosrun your_package vehicle_node vehicle2
rosrun your_package vehicle_node vehicle3
