# Deep Learning on GPU cluster in Pasteur

## GPU nodes on the new cluster
Tars is the new cluster in Pasteur, 3 GPUs nodes are available from now on on tars (in the common partition). There are 1 node with 8 teslaK80 and 2 nodes with 4 teslaM40 each for GPU based computing.

The cluster is only open to users in Institut Pasteur.

To get access, you need to pass a short online course made by DSI in Pasteur. Take the course [here](https://moocs.pasteur.fr/courses/Institut_Pasteur/DSI_01/1/about) with a good score, then send an email to informatique@pasteur.fr to ask permission to use the cluster(tars) and gpu nodes. 

## Slurm commands for GPU nodes
Slurm is used as the schedule system, you can find the detailed usage [here](http://slurm.schedmd.com/).

For using GPU nodes, you need specify these two options:
* the qos option named gpu (time limit of 7 days for a job)
* the gres option to indicate what you want.

For example:
```
srun --qos=gpu --gres=gpu:1 python ./your_script.py
```

If you want to submit on a specific type of GPU, indicate it in the gres option like this: `--gres=gpu[:type]:nb_of_gpu`

For example:
```
srun --qos=gpu --gres=gpu:teslaK80:1 python ./your_script.py
```
## Getting start
### Connect to tars.pasteur.fr
Open a terminal, run:
```bash
ssh USER_NAME@tars.pasteur.fr
```
Now, you are on a intermediate node named submission node.

Notice that, you shouldn't run any computational command on this node, it's only for sending command to computation node.

### Load modules for deep learning (with tensorflow)
```bash
module load cuda/7.5.18 cudnn/v4 test/tensorflow/0.9.0 Python/2.7.11
```
* Note: you will need to load modules(with the previous command) every time you reconnect to the cluster.

### Install Keras for Deep Learning
On the cluster, you won't have root permission, so you need to use `pip` with `--user`.
```bash
# install keras into your home folder
# do not use sudo here
pip install keras --user --upgrade
```
* Note: you only need to run this once, everything installed with pip should be automatically loaded when you have Python module loaded.

### Test installation on the submission node
As mentioned, do not run your actual code on the submission node, but you can run a shell to test your installation:
```
KERAS_BACKEND=tensorflow ipython
```
If you type:
```python
import keras
```
You should see:
```
Using TensorFlow backend.
I tensorflow/stream_executor/dso_loader.cc:108] successfully opened CUDA library libcurand.so.7.5 locally
I tensorflow/stream_executor/dso_loader.cc:108] successfully opened CUDA library libcuda.so locally
I tensorflow/stream_executor/dso_loader.cc:108] successfully opened CUDA library libcufft.so.7.5 locally
I tensorflow/stream_executor/dso_loader.cc:108] successfully opened CUDA library libcudnn.so.4 locally
I tensorflow/stream_executor/dso_loader.cc:108] successfully opened CUDA library libcublas.so.7.5 locally
```
This means, you are ready to go.

Again, within this shell, you shouldn't run any actual code, because it's on the submission node, so it doesn't even have a GPU.

In order to run it on the GPU nodes, follow the next steps.

### Run scripts
Write your code in a python file, or you can use [mnist_mlp.py](mnist_mlp.py) as an example, and run the task with `srun`(with 1 GPU).
```
# replace ./mnist_mlp.py to your own file path
srun --qos=gpu --gres=gpu:1 python ./mnist_mlp.py
```
If you want a specific GPU type, specify it:
```
srun --qos=gpu --gres=gpu:teslaK80:1 python ./mnist_mlp.py
```

If you want to launch multiple task, then you can use `sbatch`.
```bash
sbatch --qos=gpu --gres=gpu:1 ./your_script.sh
```
### Start an interactive session
Sometimes, you want to debug or run your command interactively, you can use `salloc` before `srun`:
```bash
# --pty option is used because we want to interact with ipython
salloc --pty --qos=gpu --gres=gpu:1
```
If success, you will get a shell on the requested node, you will be able to run the following command to check the gpus you have on the node you are running.

After `salloc`,  you will need prepend `srun` before your command, especially where there is GPU related command.
```bash
srun nvidia-smi
```
Then, you should be able to test with tensorflow
```
srun python -c 'import tensorflow'
```
or start a python/ipython session (with tensorflow backend for keras)
```
# in this case, you want to set a env variable for this command line, you need to put it(them) before srun
KERAS_BACKEND=tensorflow srun ipython
```

# FAQ

If you see the following error, it means your job needs more memory, you need to use '--mem' option.
```
slurmstepd: error: Step 5994415.2 exceeded memory limit (5174724 > 5120000), being killed
salloc: Exceeded job memory limit
```
For example, you can try to allocate 32GB memory:
```
salloc --mem=32G --qos=gpu --gres=gpu:1
# or, the same for srun and sbatch
srun --mem=32G --qos=gpu --gres=gpu:1 python ./your_command.py
```

If you have any other problem, you may get help in the [technical channel on slack](https://deeplearningclub.slack.com/messages/technical).

If you haven't signup, [signup here](https://deeplearningclub.slack.com/signup).


