---
title: Computer Info
description: 
published: true
date: 2021-07-15T18:51:17.857Z
tags: 
editor: markdown
dateCreated: 2020-08-20T18:25:25.303Z
---

# Clusters
`134.192.69.200` apollo.umaryland.edu
`134.192.71.44` jbay.umaryland.edu (test cluster)
`134.192.?.?`	  spirit.umaryland.edu (new GPU cluster)

apollo.umaryland.edu
2496 CPUs as of Nov 2020
- apollo1: 64 CPUs
- ...
- apollo34: 64 CPUs -- slow node!!
- ...
- apollo39: 64 CPUs

jbay.umaryland.edu
6 GPUs as of June 2020 

- n029: GeForce GTX 980, 2 CPUs 
- n030: GeForce GTX 980, 2 CPUs
- n039: GeForce GTX 980, 2 CPUs
- n081: GeForce RTX 2080, 4 CPUs
- n082: GeForce RTX 2080, 4 CPUs
- n082: GeForce RTX 2080, 4 CPUs

You can view the apollo cluster status via the following link:
http://apollo.umaryland.edu/ganglia

For all CADD clusters see
http://structure.umaryland.edu/ganglia

# Workstations (as of July 2021)
`10.229.17.50` aura.umaryland.edu # Jack (Chris, Yoann)
2020 workstation -- `GeForce RTX 2080Ti/2080`, RAM 32 GB, HD 20 TB

`10.229.17.51` altair.umaryland.edu  # Paween (Wei)
2012 DELL workstation with 1 GPU -- `GeForce RTX 2080`, RAM 8 GB, HD 14 TB 

`10.229.17.52` atlantis.umaryland.edu # Aarion (Ruibin, Shane)
2020 workstation -- `2 X GeForce RTX 2080Ti`, RAM 32 GB, HD 12 TB

`10.229.17.53` explorer.umaryland.edu  # Vinicius (rotation, Kevin)
2020 workstation -- `2 X GeForce RTX 2080Ti`, RAM 32 GB, HD 20 TB

`10.229.17.54` skylab.umaryland.edu  # Erik (Robert, Brian) 
2020 workstation -- `GeForce RTX 2080Ti/2080`, RAM 64 GB, HD 20 TB

`10.229.17.55` voyager.umaryland.edu  # Shaoqi (Neha, Yandong) 
2020 workstation -- `2 X GeForce RTX 2080Ti`, RAM 32 GB, HD 12 TB

`10.229.17.56` odyssey.umaryland.edu  # Rotation
(has old git, nfs mount to frontside) 
2012 DELL workstation, RAM 8 GB, HD 5 TB

`10.229.17.57` pathfinder.umaryland.edu  # Jana 
2012 DELL workstation -- `NVS 300`, HD 2 TB

`10.229.17.58` enterprise.umaryland.edu # Q 
2020 workstation -- `GeForce RTX 2080/3080`, RAM 32 GB, HD 22 TB

`10.229.17.59` sputnik.umaryland.edu # Rotation 
2012 DELL workstation -- `NVS 300`, RAM 8 GB, HD 4 TB

#  Printers
`10.227.186.101`   endeavour 	# HSFII 555; Jana's office  Dell 1355cn Color MFP
`10.227.186.102`   hubble    	# HSFII 612; HP LaserJet P3015
`10.227.186.103`   kepler    	# HSFII 612; Dell 2155cdn Color MFP
<hr>

# Job Submission on Apollo - Basic
```
#!/usr/bin/tcsh
#$ -S /bin/tcsh
#$ -cwd
#$ -V
#$ -N Job_name 
#$ -o $JOB_ID.o
#$ -e $JOB_ID.e
#$ -j y
#$ -l h_data=500M,h_rt=999:00:00
#$ -pe ppn64 64
#$ -R y

if ($#argv < 1) then
    USAGE:
    echo "Usage: qsub run conf"
    exit 1
endif

set input = $argv[1]

limit coredumpsize 4
limit cputime      unlimited
limit filesize     unlimited
limit datasize     unlimite
limit stacksize    unlimited
limit memoryuse    unlimited
limit vmemoryuse   unlimited

set namd     = "/state/partition1/home/mpaween/software/NAMD_Git-2019-04-19_Linux-x86_64-multicore/namd2"

set mpirun = "mpirun --leave-session-attached"

$namd +p64 $input.conf > $input.log  
exit
```

Once you have made edits to the above script submit the job with the following command 

```
qsub name.qsub
```

#### Qstat monitoring

To see all job information, queues, and more you can use

```
 qstat -f 
```

which returns something like this:


```

queuename                      qtype resv/used/tot. load_avg arch          states
---------------------------------------------------------------------------------
all.q@apollo36                 BIP   0/64/64        44.62    lx26-amd64
 209009 0.30000 RMF_E_17_2 quynhv       r     11/18/2020 15:13:35    32
 209062 0.17647 f00        mpaween      r     11/25/2020 12:23:08    32
---------------------------------------------------------------------------------
all.q@apollo35                 BIP   0/0/64         0.01     lx26-amd64
---------------------------------------------------------------------------------
all.q@apollo34                 BIP   0/64/64        64.05    lx26-amd64
 209037 0.17647 run.4      mpaween      r     11/21/2020 11:51:37    64
---------------------------------------------------------------------------------
all.q@apollo33                 BIP   0/64/64        52.05    lx26-amd64
 209073 0.30000 RMF_E_20_2 quynhv       r     11/28/2020 17:06:40    32
 209074 0.17647 f08        mpaween      r     11/28/2020 18:04:31    32
---------------------------------------------------------------------------------
all.q@apollo32                 BIP   0/64/64        40.44    lx26-amd64
 209047 0.17647 run        mpaween      r     11/23/2020 15:26:45    32
 209109 0.23077 salt-10-X- sulstice     r     12/03/2020 13:03:35    32
---------------------------------------------------------------------------------

```

To check jobs of a specific user you can do:

```
qstat -u sulstice 
```

which returns something like this:

```

job-ID  prior   name       user         state submit/start at     queue                          slots ja-task-ID
-----------------------------------------------------------------------------------------------------------------
 209028 0.23077 T3.4V.Z.10 sulstice     r     11/20/2020 09:36:01 all.q@apollo18                    32
 209029 0.23077 T2.04V.Y.1 sulstice     r     11/20/2020 09:38:00 all.q@apollo11                    32
 209031 0.23077 T2.004V.Y. sulstice     r     11/20/2020 09:41:11 all.q@apollo23                    32

```

If you would like to check the scheduling info on a job in queue:

```
qstat -j <job-id>
```

which returns:

```
job_number:                 209097
exec_file:                  job_scripts/209097
submission_time:            Mon Nov 30 12:49:26 2020
owner:                      jackh
uid:                        2916
group:                      Shen
gid:                        2001
sge_o_home:                 /home/jackh
sge_o_log_name:             jackh
sge_o_path:                 /home/jana/software/amber_dev_hybrid/bin:/usr/local/bin:/usr/bin:/bin:/usr/games:/usr/lib
sge_o_shell:                /bin/tcsh
sge_o_workdir:              /state/partition1/home/jackh/simulations/apo_plasmepsin
sge_o_host:                 apollo
account:                    sge
cwd:                        /home/jackh/simulations/apo_plasmepsin
stderr_path_list:           NONE:NONE:$JOB_ID.e
reserve:                    y
merge:                      y
hard resource_list:         h_data=500M,h_rt=3585600
mail_list:                  jackh@apollo.umaryland.edu
notify:                     FALSE
job_name:                   A.PLASME
stdout_path_list:           NONE:NONE:$JOB_ID.o
jobshare:                   0
shell_list:                 NONE:/bin/tcsh
env_list:                   CHARMMDATA=/home/jackh/simulations/toppar,MAIL=/var/mail/jackh,USER=jackh,SSH_CLIENT=10.229.17.50 59546 22,MACHTYPE=x86_64,VENDOR=unknown,SHLVL=1,LD_LIBRARY_PATH=/home/jana/software/amber_dev_hybrid/lib:/usr/local/lib,HOME=/home/jackh,SSH_TTY=/dev/pts/0,LC_TERMINAL_VERSION=3.3.12,GROUP=Shen,LOGNAME=jackh,TERM=xterm-256color,XDG_SESSION_ID=53911,AMBERHOME=/home/jana/software/amber_dev_hybrid,SGE_ROOT=/var/lib/gridengine,PATH=/home/jana/software/amber_dev_hybrid/bin:/usr/local/bin:/usr/bin:/bin:/usr/games:/usr/lib,XDG_RUNTIME_DIR=/run/user/2916,REMOTEHOST=aura.umaryland.edu,SGE_CELL=default,LANG=en_US.UTF-8,SHELL=/bin/tcsh,HOST=apollo,LC_TERMINAL=iTerm2,OSTYPE=linux,PWD=/home/jackh/simulations/apo_plasmepsin,SSH_CONNECTION=10.229.17.50 59546 134.192.69.200 22,PYTHONPATH=/home/jana/software/amber_dev_hybrid/lib/python2.7/site-packages,HOSTTYPE=x86_64-linux
script_file:                qsub_hphrex.qsub
parallel environment:  ppn64 range: 192
scheduling info:            queue instance "all.q@apollo10" dropped because it is full
                            queue instance "all.q@apollo11" dropped because it is full

```

To check job status on apollo type:
`qstat -f -u "*"`

<hr>
