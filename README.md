# sockeye-recipes
Training scripts and recipes for the Sockeye Neural Machine Translation (NMT) toolkit
- The original Sockeye codebase is at [AWS Labs](https://github.com/awslabs/sockeye)
- This version is based off [a fork](https://github.com/kevinduh/sockeye)

This package contains scripts that makes it easy to run NMT experiments.
All settings are specified in a user file like "hyperparams.txt", and used by:
- scripts/preprocess-bpe.sh: Preprocess bitext via BPE segmentation
- scripts/train.sh: Train the NMT model given bitext
- scripts/plot-validation-curve.sh: Compute BLEU curves by iteration on validation data

## Installation
First, clone this package: 
```bash
> git clone https://github.com/kevinduh/sockeye-recipes.git sockeye-recipes
```

We assume that Anaconda for Python virtual environments is available on the system.
Run the following to install Sockeye in two Anaconda environments, sockeye_cpu and sockeye_gpu: 

```bash
> cd path/to/sockeye-recipes
> sh ./install/install_sockeye_cpu.sh
> sh ./install/install_sockeye_gpu.sh
```

The training scripts and recipes will activate either the sockeye_cpu or sockeye_gpu environment depending on whether CPU or GPU is specified. 

## Example Run
We will train a model on some sample German-English data

(1) Download and unpack the data in any directory. Let's make our working directory "sockeye_trial" in this example:
```bash
> cd
> mkdir sockeye_trial
> cd sockeye_trial
> wget https://cs.jhu.edu/~kevinduh/j/sample-de-en.tgz
> tar -xzvf sample-de-en.tgz
```

(2) Specify the hyperparams.txt file. We can use the example in examples/hyperparams.sample-de-en.txt.

```bash
> cat examples/hyperparams.sample-de-en.txt
```

The important settings are workdir, datadir, modeldir, and the locations of the train and validation files. See the example for detailed explanation.

(3) Preprocess data with BPE segmentation. 

```bash
> sh path/to/sockeye-recipes/preprocess-bpe.sh path/to/sockeye-recipes/examples/hyperparams.sample-de-en.txt
```

This is a standard way (though not the only way) to handle large vocabulary in NMT. Currently sockeye-recipes assumes BPE segmentation before training. The preprocess-bpe.sh script takes a hyperparam.txt file as input and preprocesses accordingly. 

To get a flavor of BPE segmentation results (train.en is original, train.bpe-4000.en is BPE'ed): 
```bash
> head -3 sample-de-en/train.en sample-de-en/train.bpe-4000.en
```
Note the marks @@ indicate BPE segmentation boundaries

(4) Now, we can train the NMT model. We give the train.sh script the hyperparameters and tell it whether to train on CPU or GPU.

CPU version:
```bash
> sh path/to/sockeye-recipes/train.sh path/to/sockeye-recipes/examples/hyperparams.sample-de-en.txt cpu
```

GPU version:
```bash
> sh path/to/sockeye-recipes/train.sh path/to/sockeye-recipes/examples/hyperparams.sample-de-en.txt gpu
```
The GPU version calls scripts/get-gpu.sh to find a free GPU card on the current machine. Sockeye allows multi-GPU training but in these recipes we only use one GPU per training process. 

These commands can also be used in conjunction with Univa Grid Engine, e.g.:
```bash
> qsub -S /bin/bash -V -cwd -q gpu.q -l gpu=1,h_rt=24:00:00 -j y train.log path/to/sockeye-recipes/train.sh path/to/sockeye-recipes/examples/hyperparams.sample-de-en.txt gpu
```

