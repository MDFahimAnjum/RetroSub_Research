# RetroSub-Revamped

## Single-step Retrosynthesis Prediction by Leveraging Commonly Preserved Substructures
- This is a fork of the original [repository](https://github.com/fangleigit/RetroSub)
- Original Nat Comm Paper [Here](https://doi.org/10.1038/s41467-023-37969-w) 


## What's New in This Fork  

This fork introduces several enhancements to improve compatibility, usability, and functionality:  

- **Compatibility Fixes**: Resolved package installation issues for seamless setup in Windows.  
- **Version Management**: Fixed package version conflicts found during installation and setup.  
- **Enhanced Documentation**: Added clear, step-by-step instructions for easy onboarding.  
- **Windows Support**: Verified compatibility of the demo run with Windows 11 using Anaconda/Conda.  
- **Improved Demo Workflow**:  
  - Organized the original demo run into three logical parts for better readability and understanding.  
  - Added essential scripts and functionalities tailored for Windows environments.  
- **Submodule Issues Resolved**: Fixed issues in submodules that were causing errors during the demo run.  
- **Integrated Submodules into Main Repository**: Consolidated submodules into the primary repository for simpler management.  

## Notable External Libraries Utilized

- [Molecule Transformer](https://github.com/pschwllr/MolecularTransformer) for substructure-level sequence-to-sequence learning _[Now directly added to the repository]_
- [Faiss](https://github.com/facebookresearch/faiss) by Facebook research for efficient similarity search and clustering of dense vectors for all the reactants
- [RDKit](https://github.com/rdkit/rdkit/tree/Release_2022_03_1) to extract common substructures
- [RetrievalModel](https://github.com/jcyk/copyisallyouneed) to produce a list of candidates similar to the given query from a large collection of data _[Now directly added to the repository]_


## Join the Discussion  
Have questions, suggestions, or issues? Head over to the [Discussions](https://github.com/MDFahimAnjum/RetroSub-Revamped/discussions) section to share your thoughts!  
- Ask questions or clarify doubts.  
- Propose new features or improvements.  
- Collaborate with the community to resolve issues.  

Your input is invaluable in making this repository even better. Let's build together!  

## Overview of the Methodology

The work consists of the following modules: 

- **Reaction retrieval**

  This module retrieves similar reactions, given a product molecule as a query. It uses a learnable cross-lingual memory retriever to align reactants and products in high-dimensional vector space. For this a Dual encoder ([RetrievalModel](https://github.com/jcyk/copyisallyouneed)) and [Faiss](https://github.com/facebookresearch/faiss) are utilized. The submodule `RetrievalModel`  implements the dual encoder introduced in the paper.

- **Substructure extraction**

  Extract the common substructures from the product molecule and the top cross-aligned candidates, based on molecular fingerprints. These substructures provide a reaction-level, fragment-to-fragment mapping between reactants and products. For this, [RDKit](https://github.com/rdkit/rdkit/tree/Release_2022_03_1) is utilized. The `sub*.py` implement the extraction process.

- **Substructure-level sequence-to-sequence Learning**

  We convert the original token-level sequence to a substructure-level sequence. The new input sequence includes the SMILES strings of the substructures followed by the SMILES strings of other fragments with virtual number labels. The output sequences are the fragments with virtual numbers. The virtual numbers are used to indicate the bond breaking/connecting site. For this a transformer architecture ([Molecule Transformer](https://github.com/pschwllr/MolecularTransformer)) is utilized. We use the submodule `MolecularTransformer` for sequence to sequence learning.


## Setup 
- clone repo. If you are in windows, best if you clone in the system drive (C:)
```bash
    git clone https://github.com/MDFahimAnjum/RetroSub_Research
```

- fix typo, and change some codes to run the submodules with recent pytorch version. In windows, open git bash and run following. If you try the following in cmd or powershell, you will get errors due to path notations
```bash
    bash scripts/step0_fix_submodule.sh 
```
- conda environment for reaction retrieval. In windows, open anaconda prompt in the root directory
```bash
    conda create -n retrieval python=3.6
    conda activate retrieval
    #conda run -n retrieval pip install -r RetrievalModel/requirements.txt -f https://download.pytorch.org/whl/torch_stable.html
    pip install torch sacrebleu transformers==2.11.0 jsonlines regex scikit-learn scipy
    conda install -c pytorch faiss-cpu
```
- conda environment for substructure extraction, seq2seq model inference, ranking model training. for windows, open anaconda prompt and cd to the root folder
```bash
    conda create -n retrosub python=3.7 -y
    conda activate retrosub
    conda install -c pytorch pytorch torchvision -y #Alternatively: pip install torch torchvision
    pip install rdkit-pypi tqdm func-timeout future six pandas gputil notebook
    cd MolecularTransformer
    pip install torchtext==0.3.1 
    conda run -n retrosub pip install -e .
```
- conda environment for model training (requires python 3.5)
```bash
    cd MolecularTransformer
    conda create -n mol_transformer python=3.5 -y
    conda activate mol_transformer
    conda install -c pytorch pytorch torchvision -y
    pip install future six tqdm pandas
    pip install torchtext==0.3.1
    conda run -n mol_transformer pip install -e . 
```

## Download models and data 
- Download processed data, models and results [here](https://figshare.com/ndownloader/files/41144306).
- Extract in root folder

## Prepare Codebase for Demo run

- Change code of submodule (reaction retrieval) to run on CPU in the code folder.
```bash
    bash scripts/step0_fix_demo.sh  
```    
## Demo Run 
1. Run [demo_part1.ipynb](demo_part1.ipynb).
2. Run [./RetrievalModel/demo_part2.ipynb](RetrievalModel/demo_part2.ipynb).
3. Run [demo_part3.ipynb](demo_part3.ipynb).


## Retrosynthesis on USPTO_full
We provide our [processed data, trained models, and predictions on the test data](https://figshare.com/ndownloader/files/41144306) as references. Reproducing the paper results with this would be quite easy (the following steps 0-6 can be skipped).
    
```bash
# in the root folder of this repo
wget https://bdmstorage.blob.core.windows.net/shared/release_data.tar.gz 
tar xzvf release_data.tar.gz --strip-components=2
rm release_data.tar.gz 

# the directory layout should be:
.
├── ...
├── ckpts
│   └── uspto_full
│            └── dual_encoder  # checkpoint of the dual encoder model
├── data  
│   └── uspto_full    
│            └── retrieval     # the *.json files are used for subextraction.
│            └── subextraction # the training and valid data used for substructure-level seq2seq training.
│            └── vanilla_AT    # AugmentedTransformers predictions for test data with no extracted substructures.
├── models                     # all the models to reproduce the paper results. 
└── ...

# Go to Step 7 to reproduce the results.
```
### Step 0: Donwload and preprocess the data

* Download the USPTO_full raw data released by  [GLN](https://github.com/Hanjun-Dai/GLN) ([dropbox link](https://www.dropbox.com/sh/6ideflxcakrak10/AADTbFBC0F8ax55-z-EDgrIza?dl=0)). 

```bash
curl -L https://www.dropbox.com/sh/6ideflxcakrak10/AAB6bLHH32CvtGTjRsXTCL02a/uspto_multi?dl=1 > download.zip
    
mkdir -p data
unzip download.zip -d data/uspto_full
rm download.zip

conda run -n retrosub --no-capture-output python data_utils/preprocess_raw_data.py  
# on test, the valid reaction ratio in the above script should be 95.616%    

# The directory layout should be:
.
├── ...
├── data  
│   └── uspto_full       
└── ...
```
    

### Step 1: Reaction retrieval
```bash
conda activate retrieval

# train dual encoder, the dev acc shall be around 0.79, we train the model on one V100 32G GPU
bash scripts/uspto_full/step1.1_reaction_retrieval_train_dual_encoder.sh

# build and search the index, please change the dual encoder checkpoint if the model is re-trained.
bash scripts/uspto_full/step1.2_reaction_retrieval_build_and_search_index.sh epoch116_batch349999_acc0.79

conda deactivate
```
### Step 2: Substructure extraction
```bash
conda activate retrosub

# build reaction dictionary from reactant to products, which will be used during extraction.
python data_utils/collect_reaction.py --dir ./data/uspto_full

# Do substructure extraction on uspto_full, and generate the training data. 
# This step was done on a CPU cluster, the data was split into 200 chunks. 
# Following the reviewers' suggestion, we find that pre-computing the fingerprints could
# significantly reduce the extraction time. However, we leave the code as it was in order 
# to reproduce the paper results.
for chunk_id in {0..199}
do
    bash scripts/uspto_full/step2_substructure_extraction.sh $chunk_id 200 subextraction
done
conda deactivate

# build the training data 

# Collect substructures on train set only in order to obtain predictions on valid data.
# This is used to collect data to train the ranker. 
python data_utils/merge_training_data.py --total_chunks 200 \
            --out_dir ./data/uspto_full/subextraction/

# train model (on train/val set) to obtain predictions on test data, and report results in the paper.
python data_utils/merge_training_data2.py --total_chunks 200 \
            --out_dir ./data/uspto_full/subextraction/

# collect the statistics over the substructures (reproduce numbers in the paper)
python data_utils/merge_stat.py --total_chunks 200 --out_dir ./data/uspto_full/subextraction/
```

### Step 3: Substructure-level seq2seq
```bash
# train the model with src-train.txt/tgt-train.txt and src-val.txt/tgt-val.txt
# we trained the model on 8xV100 32G GPU for about 1.5 days.
conda activate mol_transformer    
bash scripts/uspto_full/step3_substructure_seq2seq.sh subextraction 10
conda deactivate
# get the averged parameters of the last 5 checkpoints when the ppl on training data 
# stops decreasing, and place the model to ./models/uspto_full_retrosub.pt
```
### Step 4: Collect predictions
```bash
conda activate retrosub
# predict and merge the predicted fragments with substructures.
# it takes about 5 hours on 8xV100 32G GPU
python -u data_utils/dispatch_infer.py --model uspto_full_retrosub --dir subextraction

# merge predictions of all chunks. 
python data_utils/merge_prediction.py --total_chunks 200 \
                --dir data/result_uspto_full_retrosub_subextraction/    

# the output file dump_res_False_analysis.json and dump_res_True_analysis.json
# in the folder data/result_uspto_full_retrosub_subextraction are the predictions
# using all the extracted substructures and the correct substructures, respectively.
```
### Step 5: Reproduce the vanilla AT model used in our paper
```bash
# generate training data
conda activate retrosub
python data_utils/prepare_vanilla_AT_data.py --input_dir data/uspto_full \
                --output_dir data/uspto_full/vanilla_AT
conda deactivate

# train the model, we train the model on 8xV100 32G GPU for about two days.
conda activate mol_transformer
bash scripts/uspto_full/step3_substructure_seq2seq.sh vanilla_AT 8    

# get the averged parameters of the last 5 checkpoints when the ppl on training data stop decreasing,
# and place the model to ./models/uspto_full_vanilla_AT.pt
# we share the data with no extracted substructures in vanilla_AT folder, src-no_sub.txt and tgt-no_sub.txt
python MolecularTransformer/translate.py -model ./models/uspto_full_vanilla_AT.pt \
        -src ./data/uspto_full/vanilla_AT/src-no_sub.txt \
        -output ./data/uspto_full/vanilla_AT/predictions-no_sub.txt \
        -batch_size 32 -replace_unk -max_length 200 -fast -n_best 10 -beam_size 10 -gpu 0
conda deactivate
```

### Step 6: Train the ranker.
```bash
# The ranker should be trained on the predictions of the valid data, i.e.,
#   in Step 2, obtain the training data with data_utils/merge_training_data.py. 
#   in Step 3, train the the substructure-level seq2seqmodel
#   in Step 4, obtain predictions on the valid data.

conda activate retrosub
# build the training data for ranker
python ranker.py --val_preds VAL_DATA_RESULT_DIR/dump_res_False_analysis.json \
            --data_save_path data/rank_training_data.pkl

# train the ranking model, stop training after several epochs, the final results should be comparable.
python ranker.py --do_training --data_save_path data/rank_training_data.pkl \
        --model_save_dir models/ranker
```
### Step 7: Re-produce paper results with [1-topk_acc.ipynb](notebooks/1-topk_acc.ipynb) and [2-amidation.ipynb](notebooks/2-amidation.ipynb).
