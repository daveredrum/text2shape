# Text2Shape: Generating Shapes from Natural Language by Learning Joint Embeddings

Kevin Chen, Christopher Choy, Manolis Savva, Angel Chang, Thomas Funkhouser, Silvio Savarese

* [Project Page](http://text2shape.stanford.edu/)
* [Paper](https://arxiv.org/abs/1803.08495)
* [Bibtex](http://text2shape.stanford.edu/bibtex.bib)

**Citing**

If you find this code useful in your work, please cite us:

```
@article{chen2018text2shape,
  title={Text2Shape: Generating Shapes from Natural Language by Learning Joint Embeddings},
  author={Chen, Kevin and Choy, Christopher B and Savva, Manolis and Chang, Angel X and Funkhouser, Thomas and Savarese, Silvio},
  journal={arXiv preprint arXiv:1803.08495},
  year={2018}
}
```

## Setup

### Downloads

**Data**

The following data files can be downloaded from the
[project webpage](http://text2shape.stanford.edu/) and are required for running the code:

* [ShapeNet](https://www.shapenet.org/) voxelizations
* Primitives dataset
* Additional dataset files located [here](http://text2shape.stanford.edu/dataset/text2shape-data.zip)

Due to the complicated and highly variable nature of the meshes collected in ShapeNet, a select few
of the models in the ShapeNet dataset result in unexpected voxelizations. We have noted which of
these models this phenomenon occurs in the text2shape data zip. If you find any additional such models, please report them to
kevin.chen@cs.stanford.edu. Thank you!

**Third Party**

Please download and build the [SmartScenes Toolkit](https://github.com/smartscenes/sstk).

### Configuration

Under `tools/scripts/render.sh`, edit the `$TOOLKIT_PATH` to the path to the SmartScenes Toolkit.

Additionally, set up the configuration file in `lib/config.py`. The following need to be edited:

**General**

* `__C.DIR.DATA_PATH`: Directory containing train/val/test splits and json.
* `__C.DIR.TOOLKIT_PATH`: Path to SmartScenes Toolkit.

**ShapeNet Paths**

* `__C.DIR.RGB_VOXEL_PATH`: Directory containing the RGB voxelizations.
* `__C.DIR.RAW_CAPTION_CSV`: ShapeNet captions CSV.

**Primitives Paths**

* `__C.DIR.PRIMITIVES_RGB_VOXEL_PATH`: Directory containing the RGB voxelizations and descriptions.

### Data Format: NRRD

We store our voxelizations in [NRRD](http://teem.sourceforge.net/nrrd/) format. We read the data in
Python using [pynrrd](https://github.com/mhe/pynrrd).

To visualize your voxels, you can visualize the NRRD using the `ssc/render-voxels.js` script from
SmartScenes Toolkit, or use your own method (e.g. save in a different format and visualize point
clouds, etc.).

## Usage

### Text and shape encoders

#### ShapeNet

```bash
# Train
bash run_lba_encoder.sh 0 LBA1 'shapenet/encoder_logdir' 'train encoder on shapenet' '--dataset shapenet --validation --visit_weight 0.25 --learning_rate 2e-4 --lba_mode MM --num_epochs 100 --decay_steps 2500 --lba_test_mode shape --batch_size 100 --lba_unnormalize'

# Generate text and shape embeddings in a subdirectory under the model path called train/val/test
# replace <model> with corresponding model prefix
# replace <timestamp> with correct time
# replace <split> with train/val/test to generate embeddings of corresponding split
bash tools/scripts/generate_text_embeddings.sh LBA1 outputs/shapenet/encoder_logdir/<timestamp>/<model> '--dataset shapenet --val_split <split> --visit_weight 0.25 --lba_mode MM --num_epochs 10000 --lba_test_mode text --lba_unnormalize'
```

#### Primitives

```bash
# Train
bash run_lba_encoder.sh 0 LBA1 'primitives/encoder_logdir' 'our full method on primitives' '--dataset primitives --validation --visit_weight 0.25 --learning_rate 2e-4 --lba_mode MM --num_epochs 100 --decay_steps 5000 --lba_test_mode shape --batch_size 100 --lba_unnormalize'

# Generate text embeddings
# replace <model> with corresponding model prefix
# replace <timestamp> with correct time
# replace <split> with train/val/test to generate embeddings of corresponding split
bash tools/scripts/generate_text_embeddings.sh LBA1 outputs/primitives/encoder_logdir/<timestamp>/<model> '--dataset primitives --val_split <split> --visit_weight 0.25 --lba_mode MM --num_epochs 10000 --lba_test_mode text --lba_unnormalize'

# Generate shape embeddings
# replace <model> with corresponding model prefix
# replace <timestamp> with correct time
# replace <split> with train/val/test to generate embeddings of corresponding split
bash tools/scripts/generate_text_embeddings.sh LBA1 outputs/primitives/encoder_logdir/<timestamp>/<model> '--dataset primitives --val_split <split> --visit_weight 0.25 --lba_mode MM --num_epochs 10000 --lba_test_mode shape --lba_unnormalize'
```

### Conditional Wasserstein GAN

```
# Train
./run.sh 0 shapenet/cwgan_logdir "CWGAN1 (improved WGAN) on ShapeNet" "--model CWGAN1 --dataset shapenet --cfg ./cfgs/improved_wgan.yaml --shapenet_ct_classifier --learning_rate 5e-5 --queue_capacity 20 --noise_size 8 --uniform_max 0.5 --decay_steps 10000"

# Render
./tools/scripts/render.sh CWGAN1 outputs/shapenet/cwgan_logdir
```

### Shape classifier

#### ShapeNet

```
# Train
python main.py --shapenet_ct_classifier --model Classifier128 --classifier --dataset shapenet --validation --log_path outputs/shapenet/shapenet_ct_classifier --batch_size 64
```

#### Primitives

```
# Train on all splits
./run_classifier.sh 0 primitives/classifier128 'classifier on primitives dataset (full dataset with train/val/split)' '--dataset primitives'
# or run:
python main.py --model Classifier128 --batch_size 64 --num_epochs 10000 --learning_rate 1e-3 --decay_steps 10000 --log_path ./outputs/primitives/classifier128 --dataset primitives --validation --classifier
```
