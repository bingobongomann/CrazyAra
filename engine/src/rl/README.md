## Reinforcement learning

This directory contains the main files which are needed to run CrazyAra in selfplay mode.

<img align="right" src="https://www.docker.com/sites/default/files/d8/2019-07/horizontal-logo-monochromatic-white.png" width="128">

### Docker Image

For a convenient setup installation, we provide a
[Dockerfile](https://github.com/QueensGambit/CrazyAra/blob/master/engine/src/rl/Dockerfile).
The dockerfile is based on the [official NVIDIA 
MXNet Docker container](https://docs.nvidia.com/deeplearning/frameworks/mxnet-release-notes/overview.html#overview) and
installs all additional libraries for reinforcement learning.
Lastly, it compiles the CrazyAra executable from the C++ source code using the current repository state.
:warning: NVIDIA Docker [does not work on Windows](https://github.com/NVIDIA/nvidia-docker/wiki/Frequently-Asked-Questions#is-microsoft-windows-supported).

In order to build the docker container, use the following command:
 
```shell script
 docker build -t crazyara_docker .
```

Afterwards you can start the container using a specified list of GPUs:
```shell script
docker run -gpus all -it \
 --rm -v local_dir:/data/RL crazyara_docker:latest
```
If you want to launch the docker using only a subset of availabe you can specify them by e.g. `--gpus '"device=10,11,12"'` instead.

The parameter `-v` describes the mount directory, where the selfplay data will be stored.

For older docker version you can use the `nvidia-docker run` command instead:
```shell script
nvidia-docker run -it --rm \
 -v <local_dir>:/data/RL crazyara_docker:latest
```

---


#### CrazyAra binary

The Dockerfile builds the _CrazyAra_ binary from source with reinforcement learning support at `root/CrazyAra/engine/build/`.
Now, you can move the binary to the main reinforcement learning directory where the selfplay games are generated:
```shell script
mv /root/CrazyAra/engine/build/CrazyAra /data/RL
```

#### Network file
You can download a network which was trained via
 supervised learning as a starting point:

```shell script
cd /data/RL
wget https://github.com/QueensGambit/CrazyAra/releases/download/0.6.0/RISEv2-mobile.zip
unzip RISEv2-mobile.zip
```

Alternatively, if a model file is already available on the host machine, you can move the model directory into the mounted docker directory.

#### Selfplay

After all premilirary action have been done, you can finally start selfplay from a given checkpoint file, which is stored in the directory `/data/RL/model/`.
If you want to start learning from zero knowledge, you may use a set of weights which have initialized randomly.

If the program cannot find a model inside `/data/RL/model/` it will look at `/data/RL/model/<variant>/`, where `<variant>` is the selected chess variant.

The python script [**rl_loop.py**](https://github.com/QueensGambit/CrazyAra/blob/master/engine/src/rl/rl_loop.py) is the main script for managing the reinforcement learning loop.
It can be started in two different modes: a generator mode, and a generator+training mode.

```
cd /root/CrazyAra/engine/src/rl
```

##### Trainer
You need to specify at least one gpu to also update the current neural network weights.
The gpu trainer will stop generating games and update the network as soon as enough training samples have been acquired.

:warning: There can only be one trainer and it must be started before starting any generators to ensure correct indexing.

```shell script
python rl_loop.py --device-id 0 --trainer &
```

##### Generator
The other gpu's can be used to generate games.
```shell script
python rl_loop.py --device-id 1 &
```

#### Configuration
The main configuration files for reinforcement learning can be found at `/root/CrazyAra/DeepCrazyhouse/configs/`:
*   https://github.com/QueensGambit/CrazyAra/tree/master/DeepCrazyhouse/configs

---

#### Useful commands

*   `nvidia-smi`: Shows GPU utilization
*   `docker images`: Lists all availabe docker images
*   `docker ps`: List all running docker containers
*   `Ctrl-p + Ctrl-q`: To detach the tty without exiting the shell. Processes will continue running in daemon mode.
*   `docker attach [container-id]`: Attach to a running docker container session
*   `docker exec -it [container-id] bash`: Enter a running docker container in shell mode and create a new session
*   `docker kill [OPTIONS] CONTAINER [CONTAINER...]`: Kill one or more running containers
*   `docker image rm [OPTIONS] IMAGE [IMAGE...]`: Remove one or more images

* `tmux`: Start a new tmux session
* `tmux detach`: Detach from a tmux session
* `tmux attach -t <id>`: Attach to a tmux session
