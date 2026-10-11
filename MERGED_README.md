# CPU Scheduling Simulator - Complete Setup & Guide

A browser-based CPU scheduling simulator with an integrated job queue. Build workloads, explore classic and modern scheduling policies, step through execution, compare results, and inspect process timelines and performance metrics.

---

## Table of Contents

1. [Quick Start](#quick-start)
2. [Features](#features)
3. [Requirements](#requirements)
4. [Installation & Setup](#installation--setup)
5. [Using the Simulator](#using-the-simulator)
6. [Scheduler Configuration](#scheduler-configuration)
7. [Workload Setup](#workload-setup)
8. [Reading the Output Gantt Chart](#reading-the-output-gantt-chart)
9. [Backend Integration](#backend-integration)
10. [Data Storage with MySQL](#data-storage-with-mysql)
11. [Git Workflow](#git-workflow)
12. [Project Structure](#project-structure)
13. [Tests](#tests)
14. [Troubleshooting](#troubleshooting)

---

## Quick Start

### 1. Run the Server
```bash
python Backend/server.py
# Or on systems where Python is named python3:
python3 Backend/server.py
```

### 2. Open the Simulator
Visit [http://127.0.0.1:8000/](http://127.0.0.1:8000/) in your browser. The landing page opens the workspace. Choose **Start simulating** to open the simulator.

### 3. Start Simulating
1. Open **Process Queue** and add processes
2. Choose a scheduling algorithm
3. Use **Play**, **Pause**, and step controls to explore execution
4. Inspect the Gantt chart and metrics

---

## Features

- Simulate 8 CPU scheduling algorithms, including preemptive and non-preemptive policies
- Create and edit process workloads, or queue jobs with long-term scheduling policies
- Inspect CPU Gantt chart, job admission timeline, and process lifecycle swimlanes
- Play, pause, and step through tick-by-tick simulation snapshots
- Compare algorithm results against the current workload
- Review per-process completion data and aggregate scheduling metrics
- Import workloads from JSON and CSV; export results in either format
- Generate uniform, Poisson, and exponential workloads
- Save settings and workloads in browser storage, separated by profile and workspace
- Use light or dark themes and accessibility controls
- Real-time job admission and process scheduling visualization

---

## Requirements

- **Python 3.9 or newer** for the included local server and workload API
- **A modern browser** with JavaScript and `localStorage` enabled
- No Python packages or JavaScript package installation required
- Plain HTML, CSS, and JavaScript (some icons and fonts load from CDNs when online)

---

## Installation & Setup

### Initial Clone
```bash
git pull  # Download all files to your local desktop folder
```

### Running the Server
From the repository root:
```bash
python Backend/server.py
```

The server:
- Binds to `127.0.0.1` on port `8000`
- Serves files from `Frontend/`
- Provides `GET /api/workload` for workload generation
- Falls back to browser-based generator if API is unreachable
- Press `Ctrl+C` to stop

### Direct Browser Access
You can also open `Frontend/index.html` directly in a browser for basic simulation, but running through the local server is recommended for full functionality.

---

## Using the Simulator

### Basic Workflow
1. **Add Processes:** Open Process Queue and add, edit, or remove processes
   - Each process needs: ID, arrival time, CPU burst, priority, and optional deadline
2. **Select Algorithm:** Choose a CPU scheduling algorithm and configure its parameters
3. **Run Simulation:** The simulation updates automatically from the current workload
4. **Inspect Results:** Use Play, Pause, and step controls to explore execution over time
5. **View Charts:** Switch Gantt chart views or scale to focus on CPU execution or process lifecycle
6. **Compare:** Use Compare Algorithms feature to test multiple policies on the same workload
7. **Job Queue (Optional):** Add jobs to the long-term scheduler and admit them using a policy
8. **Import/Export:** Load workloads from JSON/CSV or export results for analysis

### Benchmark Presets
Three presets are available to demonstrate different scenarios:
- **Balanced workload:** Mix of arrival times, priorities, and burst lengths
- **Burst-heavy workload:** Long process alongside short processes
- **Interactive workload:** Processes arriving over time with varied bursts

Select a preset to replace the current process workload and recalculate the simulation.

---

## Scheduler Configuration

### Scheduler Types

#### 1. CPU Scheduling Algorithm
Choose the short-term CPU scheduling policy:

| Algorithm | Selection Rule |
| --- | --- |
| **FCFS** | Earliest arrival first; running process not preempted |
| **SJF** | Shortest total CPU burst among arrived processes |
| **SRTF** | Shortest remaining CPU time; newly arrived process can preempt |
| **RR** | Processes take turns using configured time quantum |
| **Priority (Non-Preemptive)** | Highest priority among arrived processes; runs to completion |
| **Priority (Preemptive)** | Higher-priority arrival can preempt running process |
| **MLFQ** | Multiple feedback queues with configurable quanta and aging |
| **EDF** | Earliest deadline first |
| **CFS** | Lowest virtual runtime first |

#### 2. Algorithm Settings

- **Time Quantum (Q):** Time slice used by Round Robin (RR uses this; MLFQ exposes per-queue quanta)
- **Context Switch (`t_cs`):** Simulated overhead in time units when CPU changes processes (zero = no delay)
- **Priority Convention:** Choose whether lower or higher numeric value represents greater priority
- **MLFQ Quanta:** Time slices for feedback queues (defaults: 2, 4, 8 units)

Changing settings recalculates the run. Use playback controls to inspect snapshots.

---

## Workload Setup

### Process Queue
The **Process Queue** is the short-term CPU scheduler's workload. Each row shows:
- Process identifier
- Arrival time
- CPU burst
- Priority
- Optional deadline
- Current state

Edit any row to update the process; use remove control to delete it.

### Long-term Job Scheduler

Use the **Job Scheduler** to add work before it becomes a CPU process. A job has:
- ID (e.g., `J1`)
- Arrival time
- Burst
- Priority
- Deadline

#### Admitting Jobs to CPU Queue

1. **Choose an Admission Policy:** FCFS, SJF, Priority, or EDF
2. **Set Admission Time:** Only jobs with arrival time ≤ this time are eligible
3. **Set Batch Capacity:** Limit number of jobs admitted per operation
4. **Select "Admit jobs to CPU queue"**

Each admitted job:
- Receives a process ID and PCB
- Retains job ID as parent (e.g., `J1` → `P6`)
- Enters Process queue as `New`
- Transitions to `Ready` when arrival time is reached
- Can then be selected by CPU scheduler

Unadmitted jobs remain in long-term queue.

### Workload Tools

- **Generate:** Creates random workload (5 processes, uniform arrivals, lambda 2, max burst 8)
  - API endpoint: `/api/workload?count=5&distribution=uniform&lambda=2&maxBurst=8`
- **Import:** Loads process workload from JSON or CSV (replaces current workload)
- **JSON/CSV Export:** Downloads current simulation results

#### Workload Generator Parameters

| Parameter | Default | Range/Values |
| --- | --- | --- |
| `count` | 5 | 1-20 processes |
| `distribution` | `uniform` | `uniform`, `poisson`, or `exponential` |
| `lambda` | 2 | ≥ 0.1 |
| `maxBurst` | 8 | ≥ 1 |

### Import/Export Formats

#### JSON Import
```json
[
  { "id": "P1", "arrival": 0, "burst": 5, "priority": 2, "deadline": 10 },
  { "id": "P2", "arrival": 1, "burst": 3, "priority": 1, "deadline": 7 }
]
```

Or wrapped form:
```json
{
  "processes": [
    { "id": "P1", "arrival": 0, "burst": 5 }
  ]
}
```

#### CSV Import
```csv
id,arrival,burst,priority,deadline
P1,0,5,2,10
P2,1,3,1,7
```

Supported column names (case-insensitive):
- ID: `id` or `PID`
- Arrival: `arrival` or `AT`
- Burst: `burst` or `BT`
- Priority: `priority` or `Priority`
- Deadline: `deadline` or `Deadline`

Requirements:
- Burst must be ≥ 1
- Process IDs must be unique
- Arrival times must be ≥ 0

---

## Reading the Output Gantt Chart

### View Tabs

- **Unified (Dual-Tier):** Shows long-term job track and short-term CPU track together
- **CPU Execution:** Focuses on CPU activity (hides job track)
- **Process Swimlanes:** Shows one lifecycle row per process (hides scheduler tracks)

### Scale Options

- **Fit to width:** Timeline fits available chart width
- **Normal:** Standard time-unit width
- **Expanded:** More screen space per time unit for detailed inspection

### Reading Tracks (Unified View)

Read from top to bottom:

1. **Job Admission & Spooling Track (long-term scheduler)**
   - Hatched **Pool** block: Job queued from submission until admission
   - Colored block: Job in system through process completion
   - Label shows job-to-process mapping (e.g., `J1 → P6`)

2. **CPU Execution Timeline (short-term scheduler)**
   - Colored blocks: Which process uses CPU and for how long
   - Adjacent intervals for same process combined
   - **IDLE**: No process running
   - **CS**: Context-switch overhead

3. **Process Swimlanes (in dedicated view)**
   - One row per process
   - Shows waiting, running, and context-switch periods
   - Vertical deadline marker: Green = met, Red = missed

### Time Axis & Playhead

- Shared axis with event boundaries (arrivals, admissions, completions, deadlines)
- Playhead highlights selected snapshot
- Clock and step counter show current time and position

### Playback Controls

- **Reset playback:** Return to first snapshot
- **Previous/Next:** Move one snapshot backward/forward
- **Play/Pause:** Animate snapshots in order
- **Speed slider:** Control playback speed
- **Seek/Go:** Jump to nearest snapshot for entered time
- **Hover blocks:** Show PCB tooltip with process details

### Example: Follow One Job

1. Add `J1` in Job scheduler (appears in table and job-pool track)
2. Admit `J1` using admission controls (assigned process ID, e.g., `P6`)
3. Start at first playback snapshot (see `P6` as `New`)
4. Step to arrival/admission (becomes `Ready`)
5. Continue playback (watch `Running` → `Terminated`)
6. PCB tooltip and process row follow selected snapshot

---

## Simulation How It Works

The simulator models two scheduling stages:

### 1. Long-term Scheduling (Job Admission)
- Jobs begin in job queue
- Jobs become eligible when arrival time ≤ admission time
- Choose policy: FCFS, SJF, Priority, or EDF
- Admit up to batch capacity
- Admitted jobs enter CPU process queue; unadmitted remain queued

### 2. Short-term Scheduling (CPU Execution)
- Selected CPU algorithm chooses from ready processes
- Simulation advances clock and updates states/burst
- Preemptive algorithms can switch when rules require
- Round Robin uses time quantum
- Context-switch cost adds overhead when CPU changes

### Process States
- `New`: Just created, not yet ready
- `Ready`: Admitted and eligible for CPU
- `Running`: Currently executing on CPU
- `Terminated`: Completed execution

Use **Play**, **Pause**, and step controls to inspect saved execution snapshots.

**Reset Schedulers** button clears both queues and displayed run (settings remain).

---

## Metrics

The simulator reports:
- Average waiting time
- Average turnaround time
- Average response time
- CPU utilization
- Throughput

### Per-Process Results
- **AT:** Arrival time
- **BT:** Burst time
- **CT:** Completion time
- **TAT:** Turnaround time = CT − AT
- **WT:** Waiting time = TAT − BT
- **RT:** Response time = first CPU start − AT
- **Throughput:** Completed processes per time unit
- **CPU Utilization:** CPU execution time as % of elapsed time

### Job Queue Metrics
- Delay between job submission and admission
- Total turnaround from job submission to process completion

---

## Keyboard Shortcuts

Shortcuts work when focus is not in an input, select, or textarea:

| Key | Action |
| --- | --- |
| `R` | Run or refresh simulation |
| `Space` | Play or pause playback |
| `→` | Advance one snapshot |
| `←` | Go back one snapshot |

---

## Backend Integration

To bridge the gap between simulator and real system, three main approaches:

### 1. User-Space Scheduler (Actively Scheduling Real Processes)

Instead of simulating numbers, spawn real processes and control CPU time:

**Implementation (C/C++/Python):**
- **Spawn:** Use `fork()` or `subprocess`/`multiprocessing` to create child processes running actual tasks
- **Pause (Context Switch Out):** Send `SIGSTOP` signal to freeze process
- **Resume (Context Switch In):** Send `SIGCONT` signal to unfreeze process
- **Time Quantum:** Sleep for time slice, then send `SIGSTOP` again
- **Completion:** Use `waitpid()` to detect when process finishes

**Architecture:**
- Master: Your simulator (controls scheduling)
- Workers: Real processes (execute tasks)

### 2. Using Real OS Data (Trace-Driven Simulation)

Capture real OS data and feed into simulator:

**Linux/macOS:**
- **Read `/proc` (Linux):** `/proc/[pid]/stat` contains real-time process data
- **Use `ps` command:** Extract live processes with burst times and priorities
  ```bash
  ps -eo pid,etimes,time,ni
  ```
- **Pipeline:** Export `ps` output to CSV, have simulator read it

### 3. Creating Synthetic Real Workloads

Write small programs mimicking actual process behaviors:

1. **CPU-Bound Process:** Fibonacci calculation or hash mining (long CPU burst)
2. **I/O-Bound Process:** Large file read/write with sleeps (short bursts, frequent I/O waits)
3. **Interactive Process:** Wait for keyboard input

Measure exact execution times:
```bash
time ./cpu_bound_script
```

Hardcode profiles into simulator for realistic burst times.

---

## Data Storage with MySQL

### Current System
- System logs currently stored only in browser cache
- To persist data, use MySQL or similar database

### Implementation
1. Set up MySQL database for storing:
   - User profiles
   - Simulator configurations
   - Workload history
   - Simulation results and metrics
   - System logs

2. Optional Features:
   - **SIGN UP/LOGIN** feature for multi-user support
   - User-specific saved workloads and settings
   - Historical simulation data and comparisons
   - Performance tracking across sessions

### Database Schema (Suggested)
```sql
-- Users Table
CREATE TABLE users (
  id INT PRIMARY KEY AUTO_INCREMENT,
  username VARCHAR(255) UNIQUE NOT NULL,
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL
);

-- Profiles Table
CREATE TABLE profiles (
  id INT PRIMARY KEY AUTO_INCREMENT,
  user_id INT NOT NULL,
  profile_name VARCHAR(255),
  settings JSON,
  FOREIGN KEY (user_id) REFERENCES users(id)
);

-- Workloads Table
CREATE TABLE workloads (
  id INT PRIMARY KEY AUTO_INCREMENT,
  profile_id INT NOT NULL,
  workload_name VARCHAR(255),
  workload_data JSON,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (profile_id) REFERENCES profiles(id)
);

-- Simulation Results Table
CREATE TABLE simulation_results (
  id INT PRIMARY KEY AUTO_INCREMENT,
  workload_id INT NOT NULL,
  algorithm VARCHAR(50),
  metrics JSON,
  gantt_data JSON,
  completed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (workload_id) REFERENCES workloads(id)
);

-- System Logs Table
CREATE TABLE system_logs (
  id INT PRIMARY KEY AUTO_INCREMENT,
  user_id INT,
  log_message TEXT,
  log_level VARCHAR(20),
  timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(id)
);
```

---

## Git Workflow

### Daily Working Loop (99% of the time)

Run these three commands in order when saving work:

```bash
git add .
    # 📸 STAGE: Gathers all changed and new files

git commit -m "Your description here"
    # 💾 COMMIT: Takes permanent local snapshot

git push
    # 🚀 PUSH: Uploads to GitHub
```

### Setting Up New Project from Scratch

```bash
git init
    # Turn folder into Git repository

git remote add origin <YOUR_GITHUB_URL>
    # Link local folder to GitHub

git branch -M main
    # Ensure default branch is 'main'

git push -u origin main
    # Push first commit and link branch
```

### Check-Up Commands

```bash
git status
    # Show modified, staged, untracked files

git remote -v
    # Show linked GitHub URL

git log --oneline
    # Clean list of previous commits
```

### Emergency Fixes

```bash
git push -u origin main --force
    # Force local code to overwrite GitHub (use carefully!)

git checkout -- <filename>
    # Discard local changes, restore from last commit
```

### Pulling Updates

```bash
git pull
    # Download all files to local desktop folder
    # (Don't do manual file transfers)
```

---

## Project Structure

```
.
├── Backend/
│   └── server.py              # Static file server & workload API
├── Frontend/
│   ├── index.html             # CPU scheduling simulator
│   ├── app.js                 # Simulation engine & interactions
│   ├── styles.css             # Shared styles
│   ├── workspace.html         # Workspace landing page
│   ├── workspace.css          # Workspace styles
│   ├── profile.html           # Profile management
│   ├── profile.js             # Profile state & interactions
│   ├── profile.css            # Profile styles
│   ├── logs.html              # System log view
│   ├── about.html             # Project information
│   └── accessibility.js       # Theme & accessibility preferences
├── tests/
│   └── profile.test.js        # Profile reset behavior test
└── README FIRST/
    ├── README.md              # Main documentation
    ├── README2.md             # Scheduler & Gantt guide
    ├── InstructionsForBackend.md  # Backend integration
    ├── DataStorageInstructions.md  # MySQL setup
    ├── GithubUsage.md         # Git workflow
    └── MERGED_README.md       # This file
```

---

## Tests

Node.js test for profile reset behavior:

```bash
node tests/profile.test.js
```

Requirements:
- Node.js installed
- Run from repository root

---

## Profiles and Saved Data

The application stores in browser `localStorage`:
- Simulator state
- Profiles
- Workspace selection
- Theme preferences
- Accessibility settings

**Storage Scope:** Scoped to selected profile and workspace
**Persistence:** Local to browser only (not synced to remote accounts)
**Important:** Clearing site data or using different browser profile/device won't carry state across

**Profile Controls:**
- Create or rename profile
- Switch profiles
- Reset profile data
- Reset view
- Clear logs

---

## Troubleshooting

| Issue | Solution |
| --- | --- |
| Page doesn't load at port 8000 | Check server is running; verify port available; visit http://127.0.0.1:8000/ |
| Workload API fails | Browser generator used as fallback; confirm Python server running for API |
| Saved data missing | Ensure browser storage enabled; using same browser, profile, workspace |
| Icons/fonts missing | Load from external CDNs; won't appear without network access; simulator is local |
| Imports fail | Check JSON structure, CSV headers, unique IDs, non-negative arrivals, positive bursts |
| Server won't start | Verify Python 3.9+ installed; check port 8000 not in use; try `python3 Backend/server.py` |

---

## Academic Information

This project was developed for course **21CSC203P** at **SRM Institute of Science and Technology**, Ramapuram.

Project team and academic details listed in the in-app About page.

---

## Summary

This CPU Scheduling Simulator provides a comprehensive platform for:
✅ Learning OS scheduling algorithms
✅ Experimenting with different policies
✅ Visualizing process execution
✅ Comparing algorithm performance
✅ Integrating with real systems
✅ Persisting data with MySQL

Start by running `python Backend/server.py` and visiting http://127.0.0.1:8000/.
