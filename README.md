# Getting Started

## Train models

**a. Training on a single GPU**

```bash
python tools/train.py ${CONFIG_FILE} [optional arguments]
```

During training, checkpoints and other log information will be saved to the working directory, which is specified by `work_dirs` in the config file or via CLI argument `--work_dirs`.

This tool accepts several optional arguments, including:

* `--seed` : Specify a seed for numpy and pytorch.
* `--resume_from` : Resume from a previous checkpoint file.
* `--load_from` : Load a checkpoint file as pretrained parameters.
* `--cfg_options` : Override settings in the config.

e.g. Specify `work_dirs` and override the single gpu batch size with `4`:
```bash
python tools/train.py ${CONFIG_FILE} --work_dirs work_dirs/baseline --cfg_options data.samples_per_gpu=4
```

**b. Training on multiple GPUs**

```bash
bash tools/dist_train.sh ${CONFIG_FILE} ${GPU_NUM} [optional arguments]
```

Optional arguments are same as single gpu training.


## Test existing models

**a. Testing a single checkpoint on a single GPU**

```bash
python tools/test.py ${CONFIG_FILE} --ckpt ${CHECKPOINT_FILE} [optional arguments]
```

**b. Testing several checkpoints in a directory on a single GPU**

```bash
python tools/test.py ${CONFIG_FILE} --ckpt_dir ${CHECKPOINT_DIR} [optional arguments]
```

**c. Testing a single checkpoint on multiple GPUs**

```bash
bash tools/dist_test.sh ${CONFIG_FILE} ${GPU_NUM} --ckpt ${CHECKPOINT_FILE} [optional arguments]
```

**d. Testing several checkpoints in a directory on multiple GPUs**

Note: This can be run with training script simultaneously.
```bash
bash tools/dist_test.sh ${CONFIG_FILE} ${GPU_NUM} --ckpt_dir ${CHECKPOINT_DIR} [optional arguments]
```

## Model delivery

**a. Upload a checkpoint to VoyDet model zoo**

Firstly, the checkpoint should be slimmed by
```bash
python tools/model_converter/process_checkpoint.py ${IN_FILE} ${OUT_FILE}
```

Then, the model can be uploaded to the model zoo
```bash
python tools/model_converter/model_delivery.py push ${LOCAL_FILE} ${REMOTE_DIR} ${REMOTE_FILE}
```

All remote directories stored on model zoo can be listed by:
```bash
python tools/model_converter/model_delivery.py list
```

To list the files under a specific directory, run:
```bash
python tools/model_converter/model_delivery.py list -r ${REMOTE_DIR}
```

More information can be got by
```bash
python tools/model_converter/model_delivery.py -h
```

**b. Download a checkpoint from VoyDet model zoo**

```bash
python tools/model_converter/model_delivery.py pull ${REMOTE_DIR} ${REMOTE_FILE} ${LOCAL_FILE}
```

e.g.
```bash
python tools/model_converter/model_delivery.py pull demo pointpillars_demo.pth work_dirs/demo/pointpillars_demo.pth
```

## How to build custom operators for ONNX Runtime
- Download `onnxruntime-linux` from ONNX Runtime [releases](https://github.com/microsoft/onnxruntime/releases/tag/v1.7.0), extract it, expose `ONNXRUNTIME_DIR` and finally add the lib path to `LD_LIBRARY_PATH` as below:

```bash
wget https://github.com/microsoft/onnxruntime/releases/download/v1.7.0/onnxruntime-linux-x64-1.7.0.tgz

tar -zxvf onnxruntime-linux-x64-1.7.0.tgz
cd onnxruntime-linux-x64-1.7.0
export ONNXRUNTIME_DIR=$(pwd)
export LD_LIBRARY_PATH=$ONNXRUNTIME_DIR/lib:$LD_LIBRARY_PATH
```

#### Build on Linux

```bash
cd voydet ## to VOYDET root directory
VOYDET_WITH_ORT=1 python setup.py develop
```

## How to trace training performance
Luban Tracer is already integrated in voydet, follow the steps to view trace events/stats, which visualize training performance of each module (e.g. dataloading, pre_processing, forward, post_process, etc.):

a. By default, `trace stat` is captured and uploaded during training, just go to luban job list page, and click `more` and click tensorboard, you will be able to check the `trace stat` view.

b. To turn on `trace event` visualization, add the following parameter in the configuraton in your run:
```
luban_tracer: { "trace_enable": True }
```
or add the following parameter in script arguments:
```
--cfg_options luban_tracer.trace_enable=True
```
Once the job starts running, go to luban job list page, click `more` and click
tensorboard, you will be able to check `trace event` view

c. The following explains, by default, which modules are traced and how they are traced:
1. dataloader_next
```
with Tracer(op_name="dataloader_next"):
    batch = next(dataloader)
```
2. batch_to_device
```
with Tracer(op_name="batch_to_device"):
    collate_fn.to_device(batch)
```
3. pre_process
```
with Tracer(op_name="pre_process"):
    runner.call_hook('before_train_iter')
```
4. forward
```
with Tracer(op_name="forward"):
    runner.outputs = runner.model(data_batch)
```
5. post_process
```
with Tracer(op_name="post_process"):
    runner.call_hook('after_train_iter')
```

d. Add your customized trace events in your running, go to this [doc](https://cooper.didichuxing.com/docs/document/2200078176355)

## How to profile training performance

a. Add the following hook parameter in the configuration:
```
profiler_hook: {type: ProfilerHook, wait: 100, warmup: 1, active: 1, repeat: 1, profile_epochs: 1, json_trace_path: /nfs/volume-382-X/XXX}
```
or add the following hook parameter in script arguments:
```
--cfg_options profiler_hook.json_trace_path=/nfs/volume-382-X/XXX
```

note: the total size of log files to be generated = profile_epochs * active * repeat * 10MB * gpu_count * node_count, therefore we recommend keep all three parameters as 1 in the config file. e.g. if running a training job with 8 GPU * 2 machine, the total log files size will be 160MB.

b. with the parameters specified as above, profiler will wait for first 100 iterations, warmup at 101th iteration, and start capturing CPU and CUDA execution performance during the 102th iteration of 1st epoch (if epoch_based_runner is used), and repeat this cycle for 1 time. After 102th iteration done, you will find the json log files saved in the path, if json_path specified, otherwise they will be found at {work_dirs}/profiler_logs/, if work_dirs is specified. the parameters above only affect profiler behavior during training and doesn't change runner's training/validating iterations or epochs.

c. follow the wiki [Profile with pytorch profiler](http://wiki.intra.xiaojukeji.com/display/AV/%5BML+training%5D+Profile+with+pytorch+profiler) to view and understand the captured log with tensorboard.
