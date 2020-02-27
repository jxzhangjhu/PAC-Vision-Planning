# PAC-Vision-Planning

This repository contains code for the results in: [Probably Approximately Correct Vision-Based Planning using Motion Primitives](https://irom-lab.princeton.edu/wp-content/uploads/2020/02/Veer.PACBayes.pdf)

### Examples in the code:
1. Quadrotor navigating an obstacle field using a depth map from an oboard RGB-D camera
2. Quadruped (Minitaur, Ghost Robotics) traversing rough terrain using proprioceptive and exteroceptive (depth map from onbaord RGB-D camera) feedback

### Dependencies:
1. PyBullet
2. PyTorch
3. Tensorboard
4. CVXPY
4. MOSEK

### Important details before training:
1. Relevant parameters for each example are provided in a config json file located in the configs folder. 
2. Make sure that `num_cpu` and `num_gpu` parameters reflect your system before training.
3. The environments for each training example are drawn from a distribution, hence they are generated by varying the random seed. In particular, this allows us to index the environments using random seeds with ease. In the config file, `num_trials` is the number of environments to train on, `start_seed` is the starting index of the environments. We will train on environments from `start_seed` to `start_seed + num_trials`.


### The training process has three main parts:
1. Train a Prior using Evolutionary Strategies:
   - Quadrotor: ```python train_ES.py --config_file configs/config_quadrotor.json```
   - Minitaur: ```python train_ES.py --config_file configs/config_minitaur.json```

**Note:** Training with ES is computationally demanding. As an example, the quadrotor was trained on 480 environments (seeds:100-579) on an AWS g3.16xlarge instance with 60 CPU workers and 4 GPUs, while the Minitaur was trained on 10 environments (seeds:0-9) with 10 CPU workers and 1 GPU (Titan XP, 12 GB). For your convenience, we have shared the trained prior used in the paper, so this step can be skipped. Running the relevant config file will automatically load the relevant weights from the Weights folder.

2. Draw a `m` policies i.i.d. from the prior above and compute the cost for each policy on `N` new environments:
   - Quadrotor: ```python compute_policy_costs.py --config_file configs/quadrotor.json --start_seed 580 --num_envs N --num_policies m```
   - Minitaur: ```python compute_policy_costs.py --config_file configs/config_minitaur.json --start_seed 10 --num_envs N --num_policies m```

**Note:** We have shared the computed cost matrix with 4000 environments, 50 policies for the quadrotor and 2000 environments, 50 policies for the Minitaur; see `Weights/C_quadrotor.npy` and `Weights/C_minitaur.npy`

3. Perform PAC-Bayes optimization with the parametric REP in the paper on `N` environments and `m` policies using the costs computed above:
   - Quadrotor: ```python PAC_Bayes_opt.py --config_file configs/config_quadrotor.json --num_envs N --num_policies m```
   - Minitaur: ```python PAC_Bayes_opt.py --config_file configs/config_minitaur.json --num_envs N --num_policies m```

The trained posterior can be tested using the `quad_test.py` and `minitaur_test.py` scripts.



