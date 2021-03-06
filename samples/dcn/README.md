# DCN CTR SAMPLE #
A sample of building and training Deep & Cross Network with HugeCTR [(link)](https://arxiv.org/pdf/1708.05123.pdf).

## Dataset and preprocess ##
The data is provided by CriteoLabs (http://labs.criteo.com/2014/02/kaggle-display-advertising-challenge-dataset/).
The original training set contains 45,840,617 examples.
Each example contains a label (1 if the ad was clicked, otherwise 0) and 39 features (13 integer features and 26 categorical features).
The dataset also has the significant amounts of missing values across the feature columns, which should be preprocessed acordingly.
The original test set doesn't contain labels, so it's not used.

### Requirements ###
* Python >= 3.6.9
* Pandas 1.0.1
* Sklearn 0.22.1

1. Download the dataset and preprocess

Go to [(link)](http://labs.criteo.com/2014/02/kaggle-display-advertising-challenge-dataset/)
and download kaggle-display dataset into the folder "${project_home}/tools/criteo_script/".
The script `preprocess.sh` fills the missing values by mapping them to the unused unique integer or category.
It also replaces unique values which appear less than six times across the entire dataset with the unique value for missing values.
Its purpose is to redcue the vocabulary size of each columm while not losing too much information.
In addition, it normalizes the integer feature values to the range [0, 1],
but it doesn't create any feature crosses.

```shell
# The preprocessing can take 40 minutes to 1 hour based on the system configuration.
$ cd ../../tools/criteo_script/
$ bash preprocess.sh dcn 1 0
$ cd ../../samples/dcn/
```

2. Build HugeCTR with the instructions on README.md under home directory.

3. Convert the dataset to HugeCTR format
```shell
$ cp ../../build/bin/criteo2hugectr ./
$ ./criteo2hugectr ../../tools/criteo_script/dcn_data/train criteo/sparse_embedding file_list.txt
$ ./criteo2hugectr ../../tools/criteo_script/dcn_data/val criteo_test/sparse_embedding file_list_test.txt
```

## Training with HugeCTR ##

1. Copy huge_ctr to samples/dcn
```shell
$ cp ../../build/bin/huge_ctr ./
```

2. Run huge_ctr
```shell
$ ./huge_ctr --train ./dcn.json
```

## Training with localized slot embedding ##

1. Plan file generation

If gossip communication library is used, a plan file is needed to be generated first as below. If NCCL communication library is used, there is no need to generate a plan file, just go to step 2. 
```shell
$ export CUDA_DEVICE_ORDER=PCI_BUS_ID
$ python3 ../../tools/plan_generation_no_mpi/plan_generator_no_mpi.py dcn_localized_embedding.json
```

2. Run huge_ctr
```shell
$ ./huge_ctr --train dcn_localized_embedding.json
```

