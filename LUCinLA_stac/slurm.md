(SlurmCommands)=
# Slurm
================================================================================================================================

Slurm is a workload manager for high-capacity computers. When you submit your process, it is either run immeadiatley or places in the queue depending on how many other jobs are being processed.

| Getting job status       |            |
|:------------------------ | :--------- |
| To check your position in line:   | <span style='color:green'> squeue -u [username] </span> |
| to see if there are a lot of jobs before you: | <span style='color:green'> top </span> |
| To estimate when process will start:  | <span style='color:green'> squeue --user=[username] --start </span> |
| To get info on a job after it has run (e.g. how much time it took):| <span style='color:green'> sacct -j <jobid> --format=JobID,JobName,MaxRSS,Elapsed </span> |
    
| Cancelling jobs          |            |
|:------------------------ | :--------- |
| To cancel a single job:   | <span style='color:green'> scancel [jobID] </span> |
| To cancel all your jobs:  | <span style='color:green'> scancel -u [username] </span> |
| To cancel all pending jobs:  | <span style='color:green'> scancel -t PENDING -u [username] </span> |                                                        

