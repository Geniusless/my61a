# DEMO

### pointpillars

We prepared a tiny pointpillars as a demo. To run the demo of pointpillars, the checkpoint file should be pulled from the model zoo firstly:
```shell
mkdir -p work_dirs/demo
python tools/model_converter/model_delivery.py pull pointpillars pp_pandarsmall_2d_20e_v020.pth work_dirs/demo/pointpillars_demo.pth
```

And the testing and evaluation process can be conducted:
```shell
python tools/test.py configs/demo/pointpillars.yaml --ckpt work_dirs/demo/pointpillars_demo.pth
```

The evaluation results shown below can be got:
```shell
+----------+--------+---------+------------+--------+
| metrics  |  Car   | Bicycle | Pedestrian |  mean  |
+----------+--------+---------+------------+--------+
| iou_0.30 | 1.0000 |  0.7486 |   0.7439   | 0.8308 |
| iou_0.50 | 1.0000 |  0.7486 |   0.7439   | 0.8308 |
| iou_0.80 | 0.7834 |  0.0366 |   0.0000   | 0.2733 |
+----------+--------+---------+------------+--------+
```

To save 3d detection visualization results into `work_dirs/demo/vis`, just add `--show` in CLI:
```shell
python tools/test.py configs/demo/pointpillars.yaml --ckpt work_dirs/demo/pointpillars_demo.pth --show
```
