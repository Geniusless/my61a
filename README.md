# Dataset Preparation

## Before Preparation

For a specific task, it is highly recommended to symlink the dataset into `$VOYDET/data`.


```
voydet
├── voydet
├── tools
├── configs
├── data
│   ├── camera_datasets
│   │   ├──11007_20200408_094909_1586310655209_0.tfrecords
│   │   ├──11007_20200408_094909_1586310707110_0.tfrecords
│   │   ├──......
│   ├── pc_datasets
│   │   ├── mini_pandar_0
│   │   │   ├──11007_20200408_094909_1586310655209_0.tfrecords
│   │   │   ├──11007_20200408_094909_1586310707110_0.tfrecords
│   │   │   ├──......
│   │   ├── mini_pandar_1
│   │   │   ├──11007_20200409_094142_1586397018534_0.tfrecords
│   │   │   ├──11007_20200409_102049_1586399204135_0.tfrecords
│   │   │   ├──......
│   ├── waymo
│   |   ├── training
│   |   │   ├── segment-10017090168044687777_6380_000_6400_000_with_camera_labels.tfrecord
│   |   │   ├── segment-10023947602400723454_1120_000_1140_000_with_camera_labels.tfrecord
│   |   ├── validation
│   |   │   ├── segment-10203656353524179475_7625_000_7645_000_with_camera_labels.tfrecord
│   |   │   ├── segment-1024360143612057520_3580_000_3600_000_with_camera_labels.tfrecord
│   |   ├── testing
│   |   │   ├── segment-10084636266401282188_1120_000_1140_000_with_camera_labels.tfrecord
│   |   │   ├── segment-10149575340910243572_2720_000_2740_000_with_camera_labels.tfrecord

```

## Data preparation

### TFrecord dataset preparation

TFrecord dataset is a folder consisted of several tfrecords and dumps a pickle file
that contains meta infos (e.g. bbox labels, classes and other attributes) after preparation.
It can be built in 2 ways:
1. A folder (e.g. `camera_datasets`) with several `xxx.tfrecords` files.
2. A folder (e.g. `pc_datasets`) with several sub-folders, and each
sub-folder (`mini_pandar_0`, `mini_pandar_1`) contains several `xxx.tfrecords` files.

For example, to prepare `camera_datasets` in 8 threads, run:
```bash
python tools/data_converter/prepare_data.py tfrecord --sample_type det3d --root_path data/camera_datasets --save_tag .cam_infos_pkl --workers 8
```
and there will be a hidden file at `data/camera_datasets/.cam_infos_pkl`.


To prepare dataset that is managed as `pc_datasets`, add `--sub_dirs` in command line:
```bash
python tools/data_converter/prepare_data.py tfrecord --sample_type det3d --root_path data/pc_datasets --sub_dirs --save_tag .pc_info_pkl --workers 8
```
which is equal to:
```bash
python tools/data_converter/prepare_data.py tfrecord --sample_type det3d --root_path data/pc_datasets/mini_pandar_0 data/pc_datasets/mini_pandar_1 --save_tag .pc_info_pkl --workers 8
```
and there will be a hidden file in each sub-folders (e.g. `data/pc_datasets/mini_pandar_0/.pc_info_pkl`).

To apply computational resources on luban to prepare data, use the script:

``tools/run_command.sh tools/data_converter/prepare_data.py tfrecord --sample_type det3d --root_path data/pc_datasets --sub_dirs --save_tag .pc_info_pkl --workers 8``

### Waymo dataset preparation

Waymo dataset has three folders(training, validation and testing), each folder consists several tfrecord files.

To mount Waymo dataset and ln it to data, follow the following command
```bash
mkdir ./data/waymo/training/ && mkdir ./data/waymo/validation/ && mkdir ./data/waymo/testing/;
sudo bash /mnt/common/jianshu/liquidio/common-dataset-mount/common_dateset_mount.sh;
mount | grep s3_common_dataset;
ln -s /nfs/s3_common_dataset/waymo_perception_v1.2/training/* ./data/waymo/training/
ln -s /nfs/s3_common_dataset/waymo_perception_v1.2/validation/* ./data/waymo/validation/
ln -s /nfs/s3_common_dataset/waymo_perception_v1.2/testing* ./data/waymo/testing/
```

To prepare dataset that is managed as `waymo`, add `--sub_dirs` in command line:
```bash
python tools/data_converter/prepare_data.py tfrecord --sample_type waymo --root_path data/waymo --sub_dirs --save_tag .info_pkl --workers 8
```

and there will be a hidden file in each sub-folders (e.g. `data/waymo/training/.info_pkl`).
```
voydet
├── data
│   ├── waymo
│   |   ├── training
│   |   |   ├── .pc_info_pkl
│   |   |   ├── range_image
│   |   |   |   ├── TOP
│   |   |   |   |   ├── segment-10017090168044687777_6380_000_6400_000_with_camera_labels_0.pkl
│   |   |   |   |   ├── segment-10017090168044687777_6380_000_6400_000_with_camera_labels_1.pkl
│   |   |   |   ├── FRONT
│   |   |   |   |   ├── segment-10017090168044687777_6380_000_6400_000_with_camera_labels_0.pkl
│   |   |   |   |   ├── segment-10017090168044687777_6380_000_6400_000_with_camera_labels_1.pkl
│   |   |   |   ├── SIDE_LEFT
│   |   |   |   |   ├── segment-10017090168044687777_6380_000_6400_000_with_camera_labels_0.pkl
│   |   |   |   |   ├── segment-10017090168044687777_6380_000_6400_000_with_camera_labels_1.pkl
│   |   |   |   ├── SIDE_RIGHT
│   |   |   |   |   ├── segment-10017090168044687777_6380_000_6400_000_with_camera_labels_0.pkl
│   |   |   |   |   ├── segment-10017090168044687777_6380_000_6400_000_with_camera_labels_1.pkl
│   |   |   |   ├── REAR
│   |   |   |   |   ├── segment-10017090168044687777_6380_000_6400_000_with_camera_labels_0.pkl
│   |   |   |   |   ├── segment-10017090168044687777_6380_000_6400_000_with_camera_labels_1.pkl
│   |   │   ├── segment-10017090168044687777_6380_000_6400_000_with_camera_labels.tfrecord
│   |   │   ├── segment-10023947602400723454_1120_000_1140_000_with_camera_labels.tfrecord

```

To apply computational resources on luban to prepare data, use the script:

``
tools/run_command.sh tools/data_converter/prepare_data.py tfrecord --sample_type waymo --root_path data/waymo --sub_dirs --save_tag .info_pkl --workers 8``

If you use the generated data, you should change the data path in the config files.


### Custom dataset preparation

You can customize you own dataset parser in `tools/data_converter/prepare_data.py`.


## GT database creation

GT database is a folder that contains several object instances (represented by points, boxes and other information) captured from specific dataset. It's usually used for some data augmentations (e.g. `DBSampler`).

To create a gt database from a TFRecord dataset:
```bash
python tools/data_converter/create_gt_database.py tfrecord --root_path data/pc_datasets/mini_pandar_0 data/pc_datasets/mini_pandar_1 --save_path data/pc_datasets/gt_database --classes Car Bicycle Pedestrian --anno_tag .info_pkl --workers 4
```

To create the gt database for Waymo Open Dataset
```bash
python tools/data_converter/create_gt_database.py waymo --root_path $WAYMO_PATH/training --save_path $WAYMO_PATH/gt_database --classes Vehicle Pedestrian Cyclist --anno_tag .pc_info_pkl --workers 4
```

## Multi-frame GT database creation

Now we support multi-frame gt database. Both multi-frame and single-frame model can use it.

To create a multi-frame gt database from Tripsegment dataset:
```bash
python tools/data_converter/create_gt_database.py tripsegment_track --root_path path/to/data1 path/to/data2 --save_path path/to/gt_database --classes Car Bicycle Pedestrian --anno_tag .info_pkl --workers 4
```

To perform copy-paste during training. Please follow the example in configs/copy_paste.
