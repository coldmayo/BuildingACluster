## Components

1. Central Scheduler
	1. Manages the cluster state + resource allocation
	2. Receives job submissions from users
	3. Assigns jobs to worker nodes based on resource availability and policies
	4. Tracks job status and node health
2. Worker Nodes
	1. Execute jobs assigned by the scheduler
	2. Report resource usage and job status back to the scheduler
	3. Handle job lifecycle
3. Job Queue
	1. Stores pending jobs in a priority queue
	2. Jobs are ordered based on priority, submission time, or whatever other policies
4. Resource Database
	1. Tracks available resources on each worker node
5. User interface
	1. Probably using CLI
	2. Allows users to submit jobs, check job status, and cancel jobs
6. Communication
	1. Use HTTP to communicate between nodes

## Running a Job

1. Submission: User submits job
2. Queuing: Job is added to the job queue
3. Scheduling: Scheduler assigns the job to a worker node based on resource availability
4. Execution: Worker node runs the job
5. Completion: Job finishes, and resources are released
6. Cleanup: Job output and logs are saved, and the job is removed from the system

## Implementation

1. Have a JSON file that holds all of the jobs and their data: ```{
  "job_id": "123",
  "command": "python script.py",
  "resources": {
    "cpu": 4,
    "memory": "8G",
    "gpu": 1
  },
  "priority": 1,
  "output": "/path/to/output.log"
}```
2. Build the Job Queue
3. Implement the Scheduler
4. Develop Worker Node Logic
5. Set up Communication
6. Add Monitoring and Logging
	1. Track job status
	2. Log events for debugging and auditing
7. Create User interface

## Useful C libraries

- I am building this in either C or Zig (uses C standard libraries, could be worth trying out...), here I will list some helpful C standard libraries
	- pthread.h
		- Use pthreads to create and manage threads for job execution
		- Assign threads to specific CPUs using CPU affinity (pthread_setaffinity_np)
	- sched.h
		- Use functions like sched_setaffinity to bind processes or threads to specific CPUs
		- ```#include <pthread.h>
			#include <sched.h>
			void assign_thread_to_cpu(pthread_t thread, int cpu_id) {
			    cpu_set_t cpuset;
			    CPU_ZERO(&cpuset);
				CPU_SET(cpu_id, &cpuset);
			    pthread_setaffinity_np(thread, sizeof(cpu_set_t), &cpuset);
			}```
	- unistd.h
		- Use fork() to create child processes for job execution
		- Use exec() family of functions to replace the child process with the job's executable
	- sys/sysinfo.h
		- use sysinfo() to get system-wide resource usage (total RAM, free RAM)
	- /proc filesystem
		- read /proc/cpuinfo to get CPU details
		- read /proc/meminfo to get memory usage details
	- Use sockets to communicate between nodes