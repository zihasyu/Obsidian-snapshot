# 简易说明文档

## 一些环境要求

首先如果想在星际上做一些实验，需要通过`bash install_sc2.sh`安装SC2环境，并且设置`SC2PATH`变量：

```
export SC2PATH=[Your SC2 folder like /abc/xyz/3rdparty/StarCraftII]
```

需要安装的python环境，之前总结需要的一些依赖如下：

```
conda create -n pymarl python=3.7 -y
conda activate pymarl
conda install pytorch torchvision torchaudio cudatoolkit=10.2 -c pytorch -y
pip install sacred numpy scipy matplotlib seaborn pyyaml pygame pytest probscale imageio snakeviz tensorboard-logger tensorboard tensorboardx sympy
```

## 星际新地图

要跑星际上的实验的话，可能会涉及到一些星际默认配置没有到地图，在目录底下的`new_maps.zip`提供了一些新地图文件供使用。使用的时候需要把这些文件放到`SC2PATH`目录下专门存放地图文件的指定位置，并且需要在`smac`的`smac_map.py`文件中添加关于这些地图的配置参数（否则会报错）。我们提供了一个文件夹`qplex_smac`，其中`qplex_smac/smac/env/starcraft2/maps/smac_maps.py`文件里的`map_param_registry`变量记录了这些新地图的参数设置，可以作为参考，或者也可以直接在`qplex_smac`下`pip install -e .`用这个作为`smac`环境。

## 怎么跑实验

如果想用MATTAR跑单任务上的实验，和一般pymarl框架下的命令一致，执行如下指令即可：

```
python3 src/main.py --config=sota_beta --env-config=[Env name] with env_args.map_name=[Map name]
```

如果是进行多任务的迁移实验，需要分两个步骤进行。首先多个源任务上的训练阶段，执行如下指令：

```
python3 src/main.py --config=xtrans_train_beta --task-config=[Task name] --remark=[Remark str]
```

其中`Task name`可以设置为`./src/config/tasks`下给出的任务名称，目前这份文件里预设置了`marine_battle, stalkers_and_zealots`和`MMM`.

- 等源任务训练阶段完成之后，训练得到的模型文件会保存到目录`results`下的某个位置，具体来说会保存到`results/meta_train/[Task name]/[Src tasks information]/xtrans_train/models/[Item name]/`(`checkpoint_path`)下面，脚本里设置的`remark`参数会影响最后保存目录的`Item name`；

- 另外，源任务的表征向量会保存到目录`outputs`下的某个位置，具体来说会保存到`outputs/meta_train/[Task name]/[Src tasks information]/xtrans_train/[Item name]/task_repre/`(`load_repre_dir`)下面。

迁移测试的时候，我们执行下面的命令:

```
python3 src/main.py --config=xmeta_test_beta --task-config=[Task name] --map_name=[Map name] --transfer_training=True/False --few_shot_adaptation=True --checkpoint_path=[checkpoint_path] --load_repre_dir=[load_repre_dir] --use_tensorboard=True/False --remark=[Remark str]
```

如果是做迁移学习，`transfer_training`就设置成`True`，如果是做few_shot test，则将`transfer_training`设置为`False`；建议做迁移学习的时候把`use_tensorboard`设置成`True`，few_shot test时可设置成`False`，后者的测试结果程序将存储到`results/meta_test`目录下对应位置的`.json`文件中。

另外请注意，如果想要跑`mpe`相关的实验，请使用`xtrans_train_mpe_beta`和`xmeta_test_mpe_beta`配置，原因是`mpe`环境和`SMAC`有一些地方存在差异，`mpe`中的动作维度是确定的，与场景中智能体数无关，不需要做额外处理。

## 接入新实验

如果想要在`SMAC`和`mpe`的环境基础上进行更多的实验，需要在`./src/config/tasks`下设置新的任务配置，并在执行程序时指定该任务配置；如果希望在其他的环境上进行实验，需要在`./modules/decomposers`下针对环境观测空间的特性设置相应的观测分解器，例如现在目录下有两种分解器结构。


![[Pasted image 20240509142905.png]]