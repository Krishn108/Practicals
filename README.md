# OS Practical
1. Write a program to implement CPU scheduling for first come first serve
THEORY:- First Come First Serve (FCFS) is one of the simplest scheduling algorithms. Processes are executed in the order they arrive in the ready queue. Once a process starts execution, it runs to completion.
In case of a tie, process with smaller process ID is executed first. It is always non-preemptive in nature. Its implementation is based on FIFO queue. 
It suffers from convoy effect. 

Convoy effect :- 
Convoy Effect is a situation where many processes, who need to use a resource for a short time, are blocked by one process holding that resource for a long time. This cause poor resource management.
CODE:-
#include <stdio.h>

int main()
{
    int n;
    printf("Enter the number of processes: ");
    scanf("%d", &n);
    printf("\n");

    int arrival_time[n];
    int burst_time[n];
    int completion_time[n];
    int turnaround_time[n];
    int waiting_time[n];
    int current_time = 0;
    float avg_wt = 0.0;
    float avg_tat = 0.0;

    for (int i = 0; i < n; i++)
    {
        printf("Enter Arrival time and Burst time for process P%d: ", i + 1);
        scanf("%d %d", &arrival_time[i], &burst_time[i]);
    }

    for (int i = 0; i < n; i++)
    {
        if (current_time < arrival_time[i])
        {
            current_time = arrival_time[i];
        }
        current_time += burst_time[i];
        completion_time[i] = current_time;
        turnaround_time[i] = completion_time[i] - arrival_time[i];
        waiting_time[i] = turnaround_time[i] - burst_time[i];
    }

    for (int i = 0; i < n; i++)
    {
        avg_wt += waiting_time[i];
        avg_tat += turnaround_time[i];
    }

    printf("\n");
    printf("Process No.\tArrival Time\tBurst Time\tCompletion Time\tTurnAround Time\tWaiting Time\n");

    for (int i = 0; i < n; i++)
    {
        printf("P%d\t\t%d\t\t%d\t\t%d\t\t%d\t\t%d\n", i + 1, arrival_time[i], burst_time[i], completion_time[i], turnaround_time[i], waiting_time[i]);
    }

    printf("\nAverage Turn-Around Time = %.2f units", avg_tat / n);
    printf("\nAverage Waiting Time = %.2f units", avg_wt / n);

    return 0;
}

2. Write a program to implement CPU scheduling for shortest job first.
THEORY:-
Shortest Job First (SJF) is a non-preemptive or preemptive CPU scheduling algorithm that selects the process with the smallest burst time to execute next. It aims to minimise the average waiting time and is optimal in that regard.
If two processes have the same burst time, then FCFS is used to break the tie. It is easy to implement in Batch systems where required CPU time is known in advance.
Pre-emptive mode of Shortest Job First is called as Shortest Remaining Time First (SRTF).

CODE:-
#include <stdio.h>
#include <stdlib.h>
#include <limits.h>
#include <stdbool.h> // Include this to use bool, true, and false

typedef struct process
{
    int pid;
    int arrival_time;
    int burst_time;
    int completion_time;
    int turn_around_time;
    int waiting_time;
} process;

void getProcessInfo(process p[], int n)
{
    for (int i = 0; i < n; i++)
    {
        printf("Enter arrival time and burst time of process %d: ", i + 1);
        scanf("%d %d", &p[i].arrival_time, &p[i].burst_time);
        p[i].pid = i + 1;
    }
}

void sortByArrivalTime(process p[], int n)
{
    for (int i = 0; i < n - 1; i++)
    {
        for (int j = 0; j < n - i - 1; j++)
        {
            if (p[j].arrival_time > p[j + 1].arrival_time)
            {
                process temp = p[j];
                p[j] = p[j + 1];
                p[j + 1] = temp;
            }
        }
    }
}

void calculateCompletionTime(process p[], int n)
{
    int time = 0;
    int completed = 0;
    bool is_completed[n]; // Use bool array

    for (int i = 0; i < n; i++)
    {
        is_completed[i] = false;
    }

    while (completed < n)
    {
        int idx = -1;
        int min_burst = INT_MAX;

        for (int i = 0; i < n; i++)
        {
            if (p[i].arrival_time <= time && !is_completed[i] && p[i].burst_time < min_burst)
            {
                min_burst = p[i].burst_time;
                idx = i;
            }
        }

        if (idx != -1)
        {
            time += p[idx].burst_time;
            p[idx].completion_time = time;
            is_completed[idx] = true; // Mark process as completed
            completed++;
        }
        else
        {
            time++;
        }
    }
}

void calculateTurnAroundTimeAndWaitingTime(process p[], int n)
{
    for (int i = 0; i < n; i++)
    {
        p[i].turn_around_time = p[i].completion_time - p[i].arrival_time;
        p[i].waiting_time = p[i].turn_around_time - p[i].burst_time;
    }
}

void sortByProcessID(process p[], int n)
{
    for (int i = 0; i < n - 1; i++)
    {
        for (int j = 0; j < n - i - 1; j++)
        {
            if (p[j].pid > p[j + 1].pid)
            {
                process temp = p[j];
                p[j] = p[j + 1];
                p[j + 1] = temp;
            }
        }
    }
}

void printProcessInfo(process p[], int n)
{
    float avg_wt = 0.0;
    float avg_tat = 0.0;

    for (int i = 0; i < n; i++)
    {
        avg_wt += p[i].waiting_time;
        avg_tat += p[i].turn_around_time;
    }

    printf("\nProcess No.\tArrival Time\tBurst Time\tCompletion Time\tTurnAround Time\tWaiting Time\n");
    for (int i = 0; i < n; i++)
    {
        printf("P%d\t\t%d\t\t%d\t\t%d\t\t%d\t\t%d\n", p[i].pid, p[i].arrival_time, p[i].burst_time, p[i].completion_time, p[i].turn_around_time, p[i].waiting_time);
    }

    printf("\nAverage Turn-Around Time = %.2f units", avg_tat / n);
    printf("\nAverage Waiting Time = %.2f units\n", avg_wt / n);
}

int main()
{
    int n;
    printf("Enter the number of processes: ");
    scanf("%d", &n);

    process *p = (process *)malloc(n * sizeof(process));
    if (p == NULL)
    {
        printf("Memory allocation failed.\n");
        return 1;
    }

    getProcessInfo(p, n);
    sortByArrivalTime(p, n);
    calculateCompletionTime(p, n);
    calculateTurnAroundTimeAndWaitingTime(p, n);
    sortByProcessID(p, n);
    printProcessInfo(p, n);

    free(p);
    return 0;
}

3. Write a program to implement CPU scheduling for Round Robin.
THEORY:-
Round Robin (RR) is a pre-emptive scheduling algorithm used in operating systems to allocate CPU time to processes in a time-sharing system. It is designed to share CPU time fairly among all processes, ensuring that no process monopolises the CPU, which makes it particularly well-suited for interactive systems such as time-sharing systems and real-time operating environments.

CPU is assigned to the process on the basis of FCFS for a fixed amount of time.
This fixed amount of time is called as time quantum or time slice.
After the time quantum expires, the running process is preempted and sent to the ready queue.
Then, the processor is assigned to the next arrived process.
It is always preemptive in nature.
CODE:-
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

typedef struct process{
    int arrival_time;
    int burst_time;
    int completion_time;
    int remaining_time;
    int turn_around_time;
    int waiting_time;
} process;

void getProcessInfo(process p[], int n){
    for (int i = 0; i < n; i++)
    {
        printf("Enter arrival time and burst time of process %d: ", i + 1);
        scanf("%d %d", &p[i].arrival_time, &p[i].burst_time);
        p[i].remaining_time = p[i].burst_time;
    }
}

void roundRobin(int n, process p[], int time_quantum){
    int current_time = 0;
    int completed = 0;
    bool executed_in_this_cycle;

    while (completed < n){
        executed_in_this_cycle = false;
        for (int i = 0; i < n; i++)
        {
            if (p[i].arrival_time <= current_time && p[i].remaining_time > 0)
            {
                executed_in_this_cycle = true;
                if (p[i].remaining_time > time_quantum)
                {
                    p[i].remaining_time -= time_quantum;
                    current_time += time_quantum;
                }
                else
                {
                    current_time += p[i].remaining_time;
                    p[i].completion_time = current_time;
                    p[i].remaining_time = 0;
                    completed++;
                }
            }
        }

        if (!executed_in_this_cycle)
        {
            current_time++;
        }
    }

    for (int i = 0; i < n; i++)
    {
        p[i].turn_around_time = p[i].completion_time - p[i].arrival_time;
        p[i].waiting_time = p[i].turn_around_time - p[i].burst_time;
    }
}

void printProcessInfo(process p[], int n)
{
    float avg_wt = 0.0;
    float avg_tat = 0.0;

    printf("Process No.\tArrival Time\tBurst Time\tCompletion Time\tTurn Around Time\tWaiting Time\n");
    for (int i = 0; i < n; i++)
    {
        avg_wt += p[i].waiting_time;
        avg_tat += p[i].turn_around_time;
        printf("P%d\t\t%d\t\t%d\t\t%d\t\t%d\t\t\t%d\n", i + 1, p[i].arrival_time, p[i].burst_time, p[i].completion_time, p[i].turn_around_time, p[i].waiting_time);
    }

    printf("\nAverage Turn-Around Time = %.2f units", avg_tat / n);
    printf("\nAverage Waiting Time = %.2f units\n", avg_wt / n);
}

int main()
{
    int n, time_quantum;

    printf("Enter number of processes: ");
    scanf("%d", &n);
    printf("Enter time quantum: ");
    scanf("%d", &time_quantum);

    process p[n];
    getProcessInfo(p, n);
    roundRobin(n, p, time_quantum);
    printProcessInfo(p, n);
    return 0;
}


4. Write a program for page replacement policy using a) LRU b) FIFO c) Optimal.
THEORY:-
Page replacement algorithms are used in operating systems to manage how pages are swapped in and out of memory when a process requires more memory than is available. When a page that is not currently in memory is needed, the operating system selects a page to replace, i.e., remove from memory, to make room for the new one. The goal of these algorithms is to minimise page faults, which occur when the required page is not in memory.
Common Page Replacement Algorithms:
First-In, First-Out (FIFO):
The oldest loaded page (the first one loaded into memory) is replaced.
Pages are loaded into memory in a queue. When a page needs to be replaced, the page at the front of the queue (the one that has been in memory the longest) is removed.
It is simple to implement. However, it does not consider how often or how recently a page was used, leading to possible poor performance (e.g., Belady’s Anomaly).

Optimal Page Replacement (OPT):
It replaces the page that will not be needed for the longest period in the future.
The algorithm looks ahead to see which pages will be used and replaces the page that will not be needed for the longest time.
It guarantees the lowest number of page faults. However, it is not practical for real systems since it requires future knowledge of memory accesses.

Least Recently Used (LRU):
It replaces the page that has not been used for the longest time.
The algorithm keeps track of the order in which pages are used. When a replacement is needed, it chooses the page that was least recently used.
It approximates the performance of the optimal algorithm and does not suffer from Belady's Anomaly. However, it requires extra overhead to track the order of usage.
FIRST IN FIRST OUT : -
CODE:-
#include <stdio.h>

void getRefString(int *ref_string, int n)
{
    printf("Enter reference string: ");
    for (int i = 0; i < n; i++)
    {
        scanf("%d", &ref_string[i]);
    }
}

void fifoPageReplacement(int *ref_string, int n, int frame_size)
{
    int hits = 0, page_faults = 0, frame_ptr = 0;
    int frame[frame_size];
    double hit_ratio, page_fault_ratio;

    // Initialize frame with -1
    for (int i = 0; i < frame_size; i++)
    {
        frame[i] = -1;
    }

    // Process reference string
    for (int i = 0; i < n; i++)
    {
        int found = 0;

        // Check if page is already in frame
        for (int j = 0; j < frame_size; j++)
        {
            if (frame[j] == ref_string[i])
            {
                hits++;
                found = 1;
                break;
            }
        }

        // Page fault handling
        if (!found)
        {
            frame[frame_ptr] = ref_string[i];
            page_faults++;
            frame_ptr = (frame_ptr + 1) % frame_size;
        }
    }

    // Calculate and display results
    hit_ratio = (double)hits / n;
    page_fault_ratio = (double)page_faults / n;
    printf("Hits: %d\n", hits);
    printf("Page Faults: %d\n", page_faults);
    printf("Hit Ratio: %.2f\n", hit_ratio);
    printf("Page Fault Ratio: %.2f\n", page_fault_ratio);
}

int main()
{
    int n, frame_size;

    printf("Enter size of reference string: ");
    scanf("%d", &n);
    int ref_string[n];

    getRefString(ref_string, n);

    printf("Enter frame size: ");
    scanf("%d", &frame_size);

    fifoPageReplacement(ref_string, n, frame_size);

    return 0;
}

OPTIMAL PAGE REPLACEMENT :-
CODE:-
#include <stdio.h>

void getRefString(int *ref_string, int n)
{
    printf("Enter reference string: ");
    for (int i = 0; i < n; i++)
    {
        scanf("%d", &ref_string[i]);
    }
}

int findOptimalPage(int *frame, int frame_size, int *ref_string, int n, int current_index)
{
    int farthest = current_index;
    int replace_index = -1;

    for (int i = 0; i < frame_size; i++)
    {
        int j;
        for (j = current_index + 1; j < n; j++)
        {
            if (frame[i] == ref_string[j])
            {
                if (j > farthest)
                {
                    farthest = j;
                    replace_index = i;
                }
                break;
            }
        }

        if (j == n)
        {
            return i; // Page not found in future, should replace this page
        }

        if (replace_index == -1)
        {
            replace_index = i;
        }
    }

    return replace_index;
}

int main()
{
    int n;
    printf("Enter size of reference string: ");
    scanf("%d", &n);

    int ref_string[n];
    getRefString(ref_string, n);

    int frame_size;
    printf("Enter frame size: ");
    scanf("%d", &frame_size);

    int frame[frame_size];
    int hits = 0, page_faults = 0;

    for (int i = 0; i < frame_size; i++)
    {
        frame[i] = -1; // Initialize frames to -1 (empty)
    }

    for (int i = 0; i < n; i++)
    {
        int found = 0;

        // Check if page is already in frame
        for (int j = 0; j < frame_size; j++)
        {
            if (frame[j] == ref_string[i])
            {
                hits++;
                found = 1;
                break;
            }
        }

        if (!found)
        {
            int empty_found = 0;

            // Check if there's an empty frame available
            for (int j = 0; j < frame_size; j++)
            {
                if (frame[j] == -1)
                {
                    frame[j] = ref_string[i];
                    page_faults++;
                    empty_found = 1;
                    break;
                }
            }

            if (!empty_found)
            {
                // Find the page to replace using the optimal policy
                int pos = findOptimalPage(frame, frame_size, ref_string, n, i);
                frame[pos] = ref_string[i];
                page_faults++;
            }
        }
    }

    double hit_ratio = (double)hits / n;
    double page_fault_ratio = (double)page_faults / n;

    printf("Hits: %d\n", hits);
    printf("Page Faults: %d\n", page_faults);
    printf("Hit Ratio: %f\n", hit_ratio);
    printf("Page Fault Ratio: %f\n", page_fault_ratio);

    printf("Final frame state: ");
    for (int i = 0; i < frame_size; i++)
    {
        printf("%d ", frame[i]);
    }
    printf("\n");

    return 0;
}

LRU PAGE REPLACEMENT :-
CODE:-
#include <stdio.h>

void getRefString(int *ref_string, int n)
{
    printf("Enter reference string: ");
    for (int i = 0; i < n; i++)
    {
        scanf("%d", &ref_string[i]);
    }
}

int findLRUPage(int *last_used, int frame_size)
{
    int min = last_used[0];
    int pos = 0;

    for (int i = 1; i < frame_size; i++)
    {
        if (last_used[i] < min)
        {
            min = last_used[i];
            pos = i;
        }
    }

    return pos;
}

int main()
{
    int n;
    printf("Enter size of reference string: ");
    scanf("%d", &n);

    int ref_string[n];
    getRefString(ref_string, n);

    int frame_size;
    printf("Enter frame size: ");
    scanf("%d", &frame_size);

    int frame[frame_size];
    int last_used[frame_size];
    int hits = 0, page_faults = 0, time = 0;

    // Initialize frames and last_used arrays
    for (int i = 0; i < frame_size; i++)
    {
        frame[i] = -1;
        last_used[i] = -1;
    }

    // Process each page in the reference string
    for (int i = 0; i < n; i++)
    {
        int found = 0;

        // Check if the page is already in the frame
        for (int j = 0; j < frame_size; j++)
        {
            if (frame[j] == ref_string[i])
            {
                hits++;
                found = 1;
                last_used[j] = time++; // Update last used time
                break;
            }
        }

        if (!found)
        {
            int empty_found = 0;

            // Check for an empty frame
            for (int j = 0; j < frame_size; j++)
            {
                if (frame[j] == -1)
                {
                    frame[j] = ref_string[i];
                    last_used[j] = time++; // Update last used time
                    page_faults++;
                    empty_found = 1;
                    break;
                }
            }

            // If no empty frame is found, replace the least recently used page
            if (!empty_found)
            {
                int pos = findLRUPage(last_used, frame_size);
                frame[pos] = ref_string[i];
                last_used[pos] = time++; // Update last used time
                page_faults++;
            }
        }
    }

    // Calculate and display hit and page fault ratios
    double hit_ratio = (double)hits / n;
    double page_fault_ratio = (double)page_faults / n;

    printf("Hits: %d\n", hits);
    printf("Page Faults: %d\n", page_faults);
    printf("Hit Ratio: %.2f\n", hit_ratio);
    printf("Page Fault Ratio: %.2f\n", page_fault_ratio);

    // Display final frame state
    printf("Final frame state: ");
    for (int i = 0; i < frame_size; i++)
    {
        printf("%d ", frame[i]);
    }
    printf("\n");

    return 0;
}

5. Write a program to perform priority scheduling. 
THEORY:-
Priority scheduling is a CPU scheduling algorithm used in operating systems where each process is assigned a priority. In this algorithm, the CPU is allocated to the process with the highest priority. If two processes have the same priority, they are scheduled based on other criteria, such as first-come, first-served (FCFS).

Priority of processes depends on some factors such as:
Time limit
Memory requirements of the process
Ratio of average I/O to average CPU burst time

Types of Priority Scheduling Algorithms
There are two types of priority scheduling algorithms in OS:

Non-Preemptive Scheduling
In this type of scheduling:
If during the execution of a process, another process with a higher priority arrives for execution, even then the currently executing process will not be disturbed.
The newly arrived high priority process will be put in next for execution since it has higher priority than the processes that are in a waiting state for execution.
All the other processes will remain in the waiting queue to be processed. Once the execution of the current process is done, the high-priority process will be given the CPU for execution.

Preemptive Scheduling
Preemptive Scheduling as opposed to non-preemptive scheduling will preempt (stop and store the currently executing process) the currently running process if a higher priority process enters the waiting state for execution and will execute the higher priority process first and then resume executing the previous process.

PREEMPTIVE PRIORITY SCHEDULING
CODE:-
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>
#include <limits.h>

typedef struct process
{
    int pid;
    int arrival_time;
    int burst_time;
    int remaining_time;
    int completion_time;
    int turn_around_time;
    int waiting_time;
    int priority;
    bool completed;
} process;

void getProcessInfo(process p[], int n)
{
    for (int i = 0; i < n; i++)
    {
        printf("Enter the Arrival time, Burst time, and Priority of process %d: ", i + 1);
        scanf("%d %d %d", &p[i].arrival_time, &p[i].burst_time, &p[i].priority);
        p[i].pid = i + 1;
        p[i].remaining_time = p[i].burst_time;
        p[i].completed = false;
    }
}

int main()
{
    printf("\nLOWER PRIORITY VALUE INDICATES HIGHER PRIORITY\n");
    int n;
    float avg_tat = 0.0, avg_wt = 0.0;

    printf("Enter number of processes: ");
    scanf("%d", &n);
    process p[n];
    getProcessInfo(p, n);

    int current_time = 0;
    int completed = 0;
    int last_process = -1;

    while (completed < n)
    {
        int highest_priority = INT_MAX;
        int next_process = -1;

        // Find the highest-priority process that has arrived and is not completed
        for (int i = 0; i < n; i++)
        {
            if (p[i].arrival_time <= current_time && !p[i].completed &&
                p[i].priority < highest_priority && p[i].remaining_time > 0)
            {
                highest_priority = p[i].priority;
                next_process = i;
            }
        }

        if (next_process != -1)
        {
            if (last_process != next_process)
            {
                printf("Switching to process %d at time %d\n", p[next_process].pid, current_time);
                last_process = next_process;
            }

            p[next_process].remaining_time--;
            current_time++;

            // Process completion handling
            if (p[next_process].remaining_time == 0)
            {
                p[next_process].completed = true;
                p[next_process].completion_time = current_time;
                p[next_process].turn_around_time = p[next_process].completion_time - p[next_process].arrival_time;
                p[next_process].waiting_time = p[next_process].turn_around_time - p[next_process].burst_time;
                completed++;
            }
        }
        else
        {
            current_time++;
        }
    }

    // Calculate average turnaround time and waiting time
    for (int i = 0; i < n; i++)
    {
        avg_wt += p[i].waiting_time;
        avg_tat += p[i].turn_around_time;
    }

    printf("\n--------------------------------------------------------------------------------------------\n");
    printf("Process ID\tArrival Time\tBurst Time\tPriority\tCompletion Time\tTurnaround Time\tWaiting Time\n");
    for (int i = 0; i < n; i++)
    {
        printf(" %d\t\t%d\t\t%d\t\t%d\t\t%d\t\t%d\t\t%d\n",
               p[i].pid, p[i].arrival_time, p[i].burst_time, p[i].priority,
               p[i].completion_time, p[i].turn_around_time, p[i].waiting_time);
    }

    printf("\nAverage Turnaround Time = %.2f units\n", avg_tat / n);
    printf("Average Waiting Time = %.2f units\n", avg_wt / n);

    return 0;
}

NON-PREEMPTIVE PRIORITY SCHEDULING
CODE:-
#include <stdio.h>
#include <stdbool.h>
#include <limits.h>

typedef struct process
{
    int pid;
    int arrival_time;
    int burst_time;
    int completion_time;
    int turn_around_time;
    int waiting_time;
    int priority;
    bool completed;
} process;

void getProcessInfo(process p[], int n)
{
    for (int i = 0; i < n; i++)
    {
        printf("Enter the Arrival time, Burst time, and Priority of process %d: ", i + 1);
        scanf("%d %d %d", &p[i].arrival_time, &p[i].burst_time, &p[i].priority);
        p[i].pid = i + 1;
        p[i].completed = false;
    }
}

int main()
{
    printf("\nLOWER PRIORITY VALUE HAS HIGHER PRIORITY\n");
    int n;
    float avg_tat = 0.0, avg_wt = 0.0;

    printf("Enter number of processes: ");
    scanf("%d", &n);
    process p[n];
    getProcessInfo(p, n);

    int current_time = 0, completed = 0;

    while (completed < n)
    {
        int highest_priority_index = -1;
        int highest_priority = INT_MAX;

        // Find the process with the highest priority that has arrived and is not completed
        for (int i = 0; i < n; i++)
        {
            if (p[i].arrival_time <= current_time && !p[i].completed)
            {
                if (p[i].priority < highest_priority ||
                    (p[i].priority == highest_priority && p[i].arrival_time < p[highest_priority_index].arrival_time))
                {
                    highest_priority = p[i].priority;
                    highest_priority_index = i;
                }
            }
        }

        if (highest_priority_index != -1)
        {
            int idx = highest_priority_index;
            current_time += p[idx].burst_time;
            p[idx].completion_time = current_time;
            p[idx].turn_around_time = p[idx].completion_time - p[idx].arrival_time;
            p[idx].waiting_time = p[idx].turn_around_time - p[idx].burst_time;
            p[idx].completed = true;
            completed++;
        }
        else
        {
            current_time++;
        }
    }

    for (int i = 0; i < n; i++)
    {
        avg_wt += p[i].waiting_time;
        avg_tat += p[i].turn_around_time;
    }

    printf("\n---------------------------------------------------------------------------------------------------------\n");
    printf("Process ID\tArrival Time\tBurst Time\tPriority\tCompletion Time\tTurnAround Time\tWaiting Time\n");
    for (int i = 0; i < n; i++)
    {
        printf(" %d\t\t%d\t\t%d\t\t%d\t\t%d\t\t%d\t\t%d\n",
               p[i].pid, p[i].arrival_time, p[i].burst_time, p[i].priority,
               p[i].completion_time, p[i].turn_around_time, p[i].waiting_time);
    }

    printf("\nAverage Turn-Around Time = %.2f units ", avg_tat / n);
    printf("\nAverage Waiting Time = %.2f units ", avg_wt / n);

    return 0;
}

6. Write a program to implement first fit, best fit and worst fit algorithm for memory management.
THEORY:-
First Fit, Best Fit, and Worst Fit are memory management algorithms used in operating systems to allocate memory blocks to processes. Each approach has unique strategies to allocate free memory, balancing between minimising fragmentation and maximising resource utilisation.
First Fit Algorithm
The first-fit algorithm searches for the first free partition that is large enough to accommodate the process. The operating system starts searching from the beginning of the memory and allocates the first free partition that is large enough to fit the process.

Best Fit Algorithm
The best-fit algorithm searches for the smallest free partition that is large enough to accommodate the process. The operating system searches the entire memory and selects the free partition that is closest in size to the process.

Worst Fit Algorithm
The worst-fit algorithm searches for the largest free partition and allocates the process to it. This algorithm is designed to leave the largest possible free partition for future use.

CODE:-
#include <stdio.h>

void firstFit(int blockSize[], int m, int processSize[], int n)
{
    int allocation[n];
    for (int i = 0; i < n; i++)
        allocation[i] = -1;

    for (int i = 0; i < n; i++)
    {
        for (int j = 0; j < m; j++)
        {
            if (blockSize[j] >= processSize[i])
            {
                allocation[i] = j;
                blockSize[j] -= processSize[i];
                break;
            }
        }
    }

    printf("\nFirst Fit Allocation:\nProcess No.\tProcess Size\tBlock No.\n");
    for (int i = 0; i < n; i++)
    {
        printf("%d\t\t%d\t\t", i + 1, processSize[i]);
        if (allocation[i] != -1)
            printf("%d\n", allocation[i] + 1);
        else
            printf("Not Allocated\n");
    }
}

void bestFit(int blockSize[], int m, int processSize[], int n)
{
    int allocation[n];
    for (int i = 0; i < n; i++)
        allocation[i] = -1;

    for (int i = 0; i < n; i++)
    {
        int bestIdx = -1;
        for (int j = 0; j < m; j++)
        {
            if (blockSize[j] >= processSize[i])
            {
                if (bestIdx == -1 || blockSize[j] < blockSize[bestIdx])
                    bestIdx = j;
            }
        }

        if (bestIdx != -1)
        {
            allocation[i] = bestIdx;
            blockSize[bestIdx] -= processSize[i];
        }
    }

    printf("\nBest Fit Allocation:\nProcess No.\tProcess Size\tBlock No.\n");
    for (int i = 0; i < n; i++)
    {
        printf("%d\t\t%d\t\t", i + 1, processSize[i]);
        if (allocation[i] != -1)
            printf("%d\n", allocation[i] + 1);
        else
            printf("Not Allocated\n");
    }
}

void worstFit(int blockSize[], int m, int processSize[], int n)
{
    int allocation[n];
    for (int i = 0; i < n; i++)
        allocation[i] = -1;

    for (int i = 0; i < n; i++)
    {
        int worstIdx = -1;
        for (int j = 0; j < m; j++)
        {
            if (blockSize[j] >= processSize[i])
            {
                if (worstIdx == -1 || blockSize[j] > blockSize[worstIdx])
                    worstIdx = j;
            }
        }

        if (worstIdx != -1)
        {
            allocation[i] = worstIdx;
            blockSize[worstIdx] -= processSize[i];
        }
    }

    printf("\nWorst Fit Allocation:\nProcess No.\tProcess Size\tBlock No.\n");
    for (int i = 0; i < n; i++)
    {
        printf("%d\t\t%d\t\t", i + 1, processSize[i]);
        if (allocation[i] != -1)
            printf("%d\n", allocation[i] + 1);
        else
            printf("Not Allocated\n");
    }
}

int main()
{
    int m, n;

    // Input for block sizes
    printf("Enter the number of blocks: ");
    scanf("%d", &m);
    int blockSize[m];
    printf("Enter the sizes of the blocks:\n");
    for (int i = 0; i < m; i++)
        scanf("%d", &blockSize[i]);

    // Input for process sizes
    printf("Enter the number of processes: ");
    scanf("%d", &n);
    int processSize[n];
    printf("Enter the sizes of the processes:\n");
    for (int i = 0; i < n; i++)
        scanf("%d", &processSize[i]);

    // Create copies of block sizes for each allocation algorithm
    int blockSize1[m], blockSize2[m], blockSize3[m];
    for (int i = 0; i < m; i++)
    {
        blockSize1[i] = blockSize[i];
        blockSize2[i] = blockSize[i];
        blockSize3[i] = blockSize[i];
    }

    // Perform memory allocation with different algorithms
    firstFit(blockSize1, m, processSize, n);
    bestFit(blockSize2, m, processSize, n);
    worstFit(blockSize3, m, processSize, n);

    return 0;
}


7. Write a program to implement reader/writer problem using semaphore.
THEORY:-

The Reader-Writer Problem is a classic synchronization problem in computer science, which deals with allowing multiple processes to access a shared resource (like a database) while preventing data inconsistency. The problem is particularly useful in operating systems and databases where processes (or threads) need to read or write shared data.
In the Reader-Writer problem, there are two types of processes:

Readers: Processes that only read the shared data. Multiple readers can access the data simultaneously without causing inconsistencies.
Writers: Processes that modify (write to) the shared data. Only one writer can access the data at a time to avoid data corruption.

Problem Statement
The challenge in the Reader-Writer Problem is to design a system that:
Allows multiple readers to read simultaneously.
Restricts writers so that only one writer can access the shared data at a time.
Prevents readers from accessing the data while a writer is modifying it.

There are two main variations of the problem:
First Reader-Writers Problem (No Starvation for Readers): This version gives priority to readers, allowing them to read as long as there is no writer waiting.
Second Reader-Writers Problem (No Starvation for Writers): This version gives priority to writers, ensuring that once a writer is waiting, no new readers can start reading until the writer has finished.

Solution Using Semaphores
Semaphores are used to implement synchronisation and avoid race conditions in the Reader-Writer Problem. Here, we commonly use three semaphores:
mutex: Ensures mutual exclusion when modifying the count of readers.
wrt: Controls access to the shared resource, ensuring mutual exclusion for writers.
read_count: Keeps track of the number of readers currently accessing the shared resource.

Semaphores:
mutex and wrt are binary semaphores, which can take the values 0 or 1.
read_count is an integer variable, protected by mutex, to keep track of the number of active readers.

CODE:-
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <pthread.h>

pthread_mutex_t x = PTHREAD_MUTEX_INITIALIZER;
pthread_mutex_t y = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t cond = PTHREAD_COND_INITIALIZER;

int readercount = 0;
int writercount = 0;

void *reader(void *param){
    int reader_id = *((int *)param);

    pthread_mutex_lock(&x);
    readercount++;
    if (readercount == 1)
    {
        pthread_mutex_lock(&y);
    }
    pthread_mutex_unlock(&x);

    printf("Reader %d is reading data\n", reader_id);
    usleep(100000); // Simulate reading time
    printf("Reader %d is done reading\n", reader_id);

    pthread_mutex_lock(&x);
    readercount--;
    printf("Reader %d is leaving. Remaining readers: %d\n", reader_id, readercount);
    if (readercount == 0)
    {
        pthread_mutex_unlock(&y); // Allow writers
    }
    pthread_mutex_unlock(&x);

    return NULL;
}

void *writer(void *param){
    int writer_id = *((int *)param);

    printf("Writer %d is trying to enter\n", writer_id);
    pthread_mutex_lock(&y); // Writer gets exclusive access
    printf("Writer %d is writing data\n", writer_id);
    usleep(200000); // Simulate writing time
    printf("Writer %d is done writing\n", writer_id);
    pthread_mutex_unlock(&y);

    return NULL;
}

int main(){
    int n2, i;
    printf("Enter the number of readers and writers: ");
    scanf("%d", &n2);
    printf("\n");

    pthread_t readerthreads[n2], writerthreads[n2];
    int reader_ids[n2], writer_ids[n2];

    // Create reader and writer threads
    for (i = 0; i < n2; i++){
        reader_ids[i] = i + 1;
        writer_ids[i] = i + 1;
        pthread_create(&readerthreads[i], NULL, reader, &reader_ids[i]);
        pthread_create(&writerthreads[i], NULL, writer, &writer_ids[i]);
    }

    // Wait for all threads to finish
    for (i = 0; i < n2; i++){
        pthread_join(readerthreads[i], NULL);
        pthread_join(writerthreads[i], NULL);
    }

    // Destroy mutexes
    pthread_mutex_destroy(&x);
    pthread_mutex_destroy(&y);

    return 0;
}

8. Write a program to implement Producer‐Consumer problem using semaphores. 
THEORY:-
The Producer-Consumer problem is a classic synchronization issue in operating systems. It involves two types of processes: producers, which generate data, and consumers, which process that data. Both share a common buffer. 

Problem Statement
The challenge is to ensure that the producer doesn’t add data to a full buffer and the consumer doesn’t remove data from an empty buffer while avoiding conflicts when accessing the buffer. 

Semaphore
A semaphore is a synchronization tool used in computing to manage access to shared resources. It works like a signal that allows multiple processes or threads to coordinate their actions. Semaphores use counters to keep track of how many resources are available, ensuring that no two processes can use the same resource at the same time, thus preventing conflicts and ensuring orderly execution.

In the Producer-Consumer Problem, three semaphores are typically employed:
mutex: Ensures mutual exclusion, allowing only one process to access the buffer at a time. This avoids simultaneous read/write operations by multiple producers or consumers.
empty: Counts the number of empty slots available in the buffer, ensuring that the producer only produces when there is space available.
full: Counts the number of filled slots in the buffer, ensuring that the consumer only consumes when there is data available.

Solution Using Semaphores
Each producer and consumer action is governed by conditions enforced by the semaphores. The basic idea is:
          Producer Process:
Waits on the empty semaphore to check if there’s space in the buffer.
Waits on mutex to get exclusive access to the buffer and produce an item.
Signals full after producing an item to indicate that there is now one more item for the consumer to consume.
Consumer Process:
Waits on the full semaphore to check if there’s an item to consume.
Waits on mutex to get exclusive access to the buffer and consume an item.
Signals empty after consuming an item to indicate there is now one more empty slot for the producer.

CODE:-
#include <stdio.h>
#include <stdlib.h>

int mutex = 1;
int full = 0;
int empty, x = 0; // Buffer variables

// Function to simulate producer's action
void producer()
{
    --mutex;
    ++full;
    --empty;
    x++;
    printf("Producer produces item %d\n", x);
    ++mutex;
}

// Function to simulate consumer's action
void consumer()
{
    --mutex;
    --full;
    ++empty;
    printf("Consumer consumes item %d\n", x);
    x--;
    ++mutex;
}

// Function to display current buffer status
void showBuffer()
{
    printf("\nBuffer Status:");
    printf("\nFull slots: %d", full);
    printf("\nEmpty slots: %d", empty);
    printf("\nTotal items in buffer: %d\n", x);
}

int main()
{
    int n, bufferSize;

    // Taking the buffer size as input from the user
    printf("Enter buffer size: ");
    scanf("%d", &bufferSize);

    empty = bufferSize; // Set empty slots to the buffer size

    // Display menu
    printf("\n1. Press 1 for Producer"
           "\n2. Press 2 for Consumer"
           "\n3. Press 3 for Exit"
           "\n4. Press 4 to Show Buffer Status");

    // Infinite loop to keep the system running
    while (1)
    {
        printf("\nEnter your choice: ");
        scanf("%d", &n);

        switch (n)
        {
        case 1: // Producer action
            if (mutex == 1 && empty != 0)
            {
                producer();
            }
            else
            {
                printf("Buffer is full!\n");
            }
            break;

        case 2: // Consumer action
            if (mutex == 1 && full != 0)
            {
                consumer();
            }
            else
            {
                printf("Buffer is empty!\n");
            }
            break;

        case 3: // Exit the program
            exit(0);
            break;

        case 4: // Show buffer status
            showBuffer();
            break;

        default: // Invalid choice
            printf("Invalid choice! Please try again.");
        }
    }

    return 0;
}

9. Write a program to implement Banker’s algorithm for deadlock avoidance.
THEORY:-
The Banker’s Algorithm is a deadlock avoidance algorithm introduced by Edsger Dijkstra, designed to handle resource allocation in systems where multiple processes compete for limited resources. Named after a banking system analogy, it helps determine if a system can safely allocate resources to processes without running into a deadlock.
Key Concepts in Banker's Algorithm
Safe State: A system is in a "safe state" if there is a sequence in which processes can complete their execution without running into deadlock. In this state, each process can eventually receive the resources it needs and finish, releasing those resources back to the system.
Unsafe State: If there’s no safe sequence that can ensure the successful completion of all processes, the system is in an "unsafe state." An unsafe state may lead to a deadlock, but it doesn’t necessarily mean a deadlock has occurred yet.
Resource Allocation: Banker’s Algorithm works by keeping track of allocated resources, available resources, and maximum resources needed by each process, and dynamically makes decisions about resource allocation based on these values.
Working of Banker's Algorithm
The algorithm operates under the assumption that each process must declare the maximum number of resources it might need. Based on this information, the system decides whether to grant a process's resource request by simulating the allocation and checking if the resulting state is safe.
The algorithm requires three main data structures:
Available: A vector that represents the number of resources of each type currently available in the system.
Max: A matrix where each row represents a process and each column represents the maximum number of resources of a particular type that a process may request.
Allocation: A matrix that represents the number of resources of each type currently allocated to each process.
Need: A matrix calculated by subtracting Allocation from Max for each process, representing the remaining resources each process will need to complete.

CODE:-
#include <stdio.h>
#define MAX 5

int available[MAX], max[MAX][MAX], allocation[MAX][MAX], need[MAX][MAX], n, m;

void calculateNeed()
{
    for (int i = 0; i < n; i++)
        for (int j = 0; j < m; j++)
            need[i][j] = max[i][j] - allocation[i][j];
}

int checkSafeState()
{
    int work[MAX], finish[MAX] = {0}, safeSequence[MAX], count = 0;

      for (int i = 0; i < m; i++)
        work[i] = available[i];

      while (count < n)
    {
        int found = 0;
        for (int i = 0; i < n; i++)
        {
            if (finish[i] == 0)
            {
                int j;
                               for (j = 0; j < m; j++)
                    if (need[i][j] > work[j])
                        break;

                               if (j == m)
                {
                    for (int k = 0; k < m; k++)
                        work[k] += allocation[i][k];
                    safeSequence[count++] = i;
                    finish[i] = 1;
                    found = 1;
                }
            }
        }

               if (found == 0)
        {
            printf("System is not in a safe state.\n");
            return 0;
        }
    }

   printf("System is in a safe state.\nSafe sequence is: ");
    for (int i = 0; i < n; i++)
    {
        printf("%d ", safeSequence[i]);
    }
    printf("\n");
    return 1;
}

int main()
{
   printf("Enter the number of processes: ");
    scanf("%d", &n);
    printf("Enter the number of resources: ");
    scanf("%d", &m);

   printf("Enter available resources: ");
    for (int i = 0; i < m; i++)
    {
        printf("Resource %d: ", i + 1);
        scanf("%d", &available[i]);
    }

   printf("Enter maximum resources for each process:\n");
    for (int i = 0; i < n; i++)
    {
        printf("Enter maximum resources for process %d: ", i + 1);
        for (int j = 0; j < m; j++)
        {
            scanf("%d", &max[i][j]);
        }
    }

   printf("Enter allocated resources for each process:\n");
    for (int i = 0; i < n; i++)
    {
        printf("Enter allocated resources for process %d: ", i + 1);
        for (int j = 0; j < m; j++)
        {
            scanf("%d", &allocation[i][j]);
        }
    }

   calculateNeed();

      checkSafeState();

    return 0;
}

10. Write C programs to implement the various File Organisation Techniques 
THEORY:-
File organization techniques in an operating system (OS) refer to methods used to store, manage, and access files on storage devices efficiently. Different techniques offer varying advantages depending on the requirements for quick access, minimal fragmentation, and easy maintenance. Here are the main file organization techniques:

Sequential Access
Sequential access is a file organization method where records are stored in a specific order, one after another, typically based on a key or a logical sequence. This organization is simple and commonly used for files that need to be processed in sequence, such as logs or transaction files. With sequential access, data is read or written starting from the beginning of the file, making it ideal for batch processing tasks where records are accessed in order.
Direct (or Hashed) Access
Direct, or hashed, access organizes records by computing their storage location using a hashing function, which maps a record’s key directly to a specific address in memory or on disk. This method allows for rapid retrieval, as it enables direct access to a record without reading intervening records, making it efficient for applications where records need to be accessed frequently or randomly. When a record is needed, the hashing function is applied to its key, quickly locating the exact storage location.
Indexed File Organization
Indexed file organization improves random access by using an index, similar to a book’s table of contents, which maintains pointers to the locations of records within the file. The index is typically built based on one or more key fields, and each entry in the index corresponds to a unique record in the file, containing both the key and the record’s address. This organization enables efficient access to records by allowing direct retrieval of the desired record through the index without reading the entire file. Indexed files support both sequential and random access, making them versatile for a variety of applications. They are particularly advantageous when there is a need to frequently search for, insert, or delete records.
CODE:-
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define MAX_RECORDS 100
#define NAME_LENGTH 20

typedef struct
{
    int id;                 // Unique identifier for the record
    char name[NAME_LENGTH]; // Name associated with the record
} Record;

void writeIndexed();
void readIndexed();
void writeHashed();
void readHashed();
void writeSequential();
void readSequential();

int main()
{
    int choice;
    while (1)
    {
        printf("\n1. Write Indexed File");
        printf("\n2. Read Indexed File");
        printf("\n3. Write Hashed File");
        printf("\n4. Read Hashed File");
        printf("\n5. Write Sequential File");
        printf("\n6. Read Sequential File");
        printf("\n7. Exit");
        printf("\nEnter your choice: ");
        scanf("%d", &choice);

        switch (choice)
        {
        case 1:
            writeIndexed();
            break;
        case 2:
            readIndexed();
            break;
        case 3:
            writeHashed();
            break;
        case 4:
            readHashed();
            break;
        case 5:
            writeSequential();
            break;
        case 6:
            readSequential();
            break;
        case 7:
            exit(0);
        default:
            printf("Invalid choice. Please try again.");
        }
    }
    return 0;
}

// Function to write indexed file
void writeIndexed()
{
    FILE *file = fopen("indexed.txt", "w");
    FILE *indexFile = fopen("index.txt", "w");
    if (!file || !indexFile)
    {
        printf("Error opening file!\n");
        return;
    }

    Record records[MAX_RECORDS];
    int recordCount = 0;
    char addMore;

    do
    {
        printf("Enter ID: ");
        scanf("%d", &records[recordCount].id);
        printf("Enter Name: ");
        scanf("%s", records[recordCount].name);
        recordCount++;

        printf("Do you want to add another record? (y/n): ");
        scanf(" %c", &addMore);
    } while (addMore == 'y' && recordCount < MAX_RECORDS);

    for (int i = 0; i < recordCount; i++)
    {
        fprintf(file, "%d %s\n", records[i].id, records[i].name);                    // Write the record
        fprintf(indexFile, "%d %ld\n", records[i].id, ftell(file) - sizeof(Record)); // Write ID and position to index
    }

    fclose(file);
    fclose(indexFile);
    printf("Records written to indexed.txt and index.txt.\n");
}

// Function to read indexed file
void readIndexed()
{
    FILE *file = fopen("indexed.txt", "r");
    FILE *indexFile = fopen("index.txt", "r");
    if (!file || !indexFile)
    {
        printf("Error opening file!\n");
        return;
    }

    Record record;
    printf("\nIndexed Records:\n");
    while (fscanf(file, "%d %s", &record.id, record.name) != EOF)
    {
        printf("ID: %d, Name: %s\n", record.id, record.name);
    }

    fclose(file);
    fclose(indexFile);
}

// Function to write hashed file
void writeHashed()
{
    FILE *file = fopen("hashed.txt", "a");
    if (!file)
    {
        printf("Error opening file!\n");
        return;
    }

    Record records[MAX_RECORDS];
    int recordCount = 0;
    char addMore;

    do
    {
        printf("Enter ID: ");
        scanf("%d", &records[recordCount].id);
        printf("Enter Name: ");
        scanf("%s", records[recordCount].name);
        recordCount++;

        printf("Do you want to add another record? (y/n): ");
        scanf(" %c", &addMore);
    } while (addMore == 'y' && recordCount < MAX_RECORDS);

    for (int i = 0; i < recordCount; i++)
    {
        fprintf(file, "%d %s\n", records[i].id, records[i].name); // Write the record
    }

    fclose(file);
    printf("Records written to hashed.txt.\n");
}

// Function to read hashed file
void readHashed()
{
    FILE *file = fopen("hashed.txt", "r");
    if (!file)
    {
        printf("Error opening file!\n");
        return;
    }

    Record record;
    printf("\nHashed Records:\n");
    while (fscanf(file, "%d %s", &record.id, record.name) != EOF)
    {
        printf("ID: %d, Name: %s\n", record.id, record.name);
    }

    fclose(file);
}

// Function to write sequential file
void writeSequential()
{
    FILE *file = fopen("sequential.txt", "a");
    if (!file)
    {
        printf("Error opening file!\n");
        return;
    }

    Record records[MAX_RECORDS];
    int recordCount = 0;
    char addMore;

    do
    {
        printf("Enter ID: ");
        scanf("%d", &records[recordCount].id);
        printf("Enter Name: ");
        scanf("%s", records[recordCount].name);
        recordCount++;

        printf("Do you want to add another record? (y/n): ");
        scanf(" %c", &addMore);
    } while (addMore == 'y' && recordCount < MAX_RECORDS);

    for (int i = 0; i < recordCount; i++)
    {
        fprintf(file, "%d %s\n", records[i].id, records[i].name); // Write the record
    }

    fclose(file);
    printf("Records written to sequential.txt.\n");
}

// Function to read sequential file
void readSequential()
{
    FILE *file = fopen("sequential.txt", "r");
    if (!file)
    {
        printf("Error opening file!\n");
        return;
    }

    Record record;
    printf("\nSequential Records:\n");
    while (fscanf(file, "%d %s", &record.id, record.name) != EOF)
    {
        printf("ID: %d, Name: %s\n", record.id, record.name);
    }

    fclose(file);
}
