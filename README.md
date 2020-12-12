# How to use ray with slurm cluster?

PENG Zhenghao

December 5, 2020

### Quick Start

```
git clone https://github.com/pengzhenghao/use-ray-with-slurm.git
cd use-ray-with-slurm

# Please make sure you have install ray first!
python launch.py --exp-name test --command "echo 1"

# or a RLLib task:
python launch.py --exp-name test --command "rllib train --run PPO --env CartPole-v0"
```

The above command will launch a ray cluster within slurm cluster with 1 computing node. 
Concretely, the `launch.py` does the following things:

1. It automatically writes your requirements, e.g. number of CPUs, GPUs per node, the number of nodes and so on, 
to a sbatch script name `{exp-name}_{date}-{time}.sh`. In the above example, it is `test_1205-1132.sh`. 
Your command (`--command`) to launch your own job is also written into the sbatch script.
2. Then it will submit the sbatch script to slurm manager via a new process.
3. Finally, the python process will terminate itself and leaves a log file named `{exp-name}_{date}-{time}.log` 
to record the progress of your submitted command.


### Specify number of computing nodes

If you want to utilize multiple computing node in slurm and let ray recognizes them, please use:

```
python launch.py --exp-name test --command "python your_file.py" --num-nodes 3
```

### Specify computing nodes

If you want to specify the computing nodes, just use the same node name in `sinfo` command:

```
python launch.py --exp-name test --command "python your_file.py" --num-nodes 3 --node chpc-cn[003-005]
```

### The list of all options

* `--exp-name`: The experiment name. Will generate `{exp-name}_{date}-{time}.sh` and  `{exp-name}_{date}-{time}.log`.
* `--command`: The command you wish to run. For example: `rllib train XXX` or `python XXX.py`. 
* `--num-gpus`: The number of GPUs you wish to use in each computing node. Default: 0.
* `--node` (`-w`): The specify nodes you wish to use, in the same form of the return of `sinfo`. Automatically assign if not specify.
* `--num-nodes` (`-n`): The number of nodes you wish to use. Default: 1.
* `--partition` (`-p`): The partition you wish to use. Default: "chpc" (CUHK cluster partition name, change to yours!)
* `--load-env`: The command to setup your environment. For example: `module load cuda/10.1`. Default: "".


### The procedure

The sbatch script does the following things:

1. It fetches the list of computing nodes and their IP addresses.
2. It launches a head ray process in one of the node, and get the address of the head node.
3. It launches ray processes in (n-1) worker nodes and connects them to the head node by providing the head node address.
4. It submit the user specified task to ray.

Since all n nodes have launched their own ray processes, and they are all connected to the head node's ray process, 
ray cluster will perform resources allocation as in other cluster.

### Misc

1. It works well with ray 1.0.0, feel free to open issue if you find it doesn't work.
2. Feel free to copy the script to your own projects.
3. This script is compatible with both IPV4 and IPV6 ip address of the computing nodes.
4. This project is inspired by [Yet Another Slurm Python Interface](https://github.com/albanie/yaspi) and [Ray sbatch submission](https://github.com/NERSC/slurm-ray-cluster) scripts used at NERSC.
