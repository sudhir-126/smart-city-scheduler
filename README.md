# smart-city-scheduler
Zone Job-Scheduler, Deadlock-Safety Engine , Secure Cloud-IoT Deployment Blueprint containing Python execution scripts and architecture design documentation.
# Smart City Zone Job-Scheduler & Cloud-IoT Blueprint

## Repository Contents
- `jobs.py`: Task 1 fixed dataset with PCB field mapping annotations.
- `task2_scheduling.py`: Task 2 implementation of FCFS, SJF, and SRTF algorithms.
- `task3_round_robin.py`: Task 3 implementation of Round Robin (Quantum 3 & 6) with switch tracking.
- `task4_priority.py`: Task 4 implementation of Priority Scheduling with and without aging.
- `task5_peterson.py`: Task 5 Peterson's Algorithm multi-threading demo vs. unsynchronized race conditions.
- `task6_bankers.py`: Task 6 Banker's Algorithm safety check and independent request evaluation engine.
- `task7_memory.py`: Task 7 virtual memory paging and segmentation address translation engine.
- `docs/architecture_blueprint.md`: Part 2 Cloud, Security & IoT Deployment Blueprint.

## How to Run Part 1 Engine
Execute each script using Python 3 standard library:

```bash
python task2_scheduling.py
python task3_round_robin.py
python task4_priority.py
python task5_peterson.py
python task6_bankers.py
python task7_memory.py

Task 8: Justify Deployment Choice
Chosen Production Scheduling Family: Non-Preemptive Priority Scheduling with Aging
Selected Family: Priority Scheduling with Dynamic Aging is chosen for production deployment across the zone controllers.
Cited Reasons Rejecting Alternative Families:
Rejection of FCFS: FCFS exhibits a severe convoy effect on this dataset, producing the highest average waiting time of 13.12 ticks and an average turnaround time of 18.62 ticks, forcing critical low-burst safety tasks to wait behind heavy processing jobs.
Rejection of SJF / SRTF Family: Although SRTF achieved the lowest average waiting time (6.00 ticks), both SJF and SRTF require exact advance knowledge of CPU burst times (8, 4, 9, 5, \dots), which cannot be known beforehand for dynamic IoT sensor workloads.
Rejection of Round Robin Family: Round Robin introduces excessive context switching overhead. At quantum 3, the engine recorded 16 context switches across 17 dispatch slices, causing CPU cycles to be lost to OS switching overhead rather than processing real sensor data.
