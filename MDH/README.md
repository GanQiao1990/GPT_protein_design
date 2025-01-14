
## MDH (malate dehydrogenase) generation & screening

### Prerequisites
1) Please follow environment setup instructions in upper directory
2) Set the current directory as working directory
3) install [cd-hit](https://www.bioinformatics.org/cd-hit/cd-hit-user-guide#:~:text=Installing%20%EE%80%80CD-HIT%EE%80%81%20package%20is%20very%20simple%3A%201.%20download,5.%20you%20will%20have%20all%20%EE%80%80cd-hit%EE%80%81%20programs%20compiled)
4) install [BLAST+](https://blast.ncbi.nlm.nih.gov/Blast.cgi?CMD=Web&PAGE_TYPE=BlastDocs&DOC_TYPE=Download)

### STEP 1: data preparation

**Inputs** 
1) `train_sequences.fasta`, MDH sequences from [Repecka et al. 2021](https://www.nature.com/articles/s42256-021-00310-5)
2) `val_sequences.fasta`, MDH sequences from Repecka et al. 2021 
3) `uniprot-reviewed_yes.tab`, reviewed UniProt enzyme sequences, not included in this repository due to limitation of file size, but can be easily obtained from [this UniProt link](https://www.uniprot.org/uniprotkb?facets=reviewed%3Atrue&query=enzyme)

**Outputs**
1) `X_train`, sequences of training set
2) `y_train`, labels of training set (0 is non-MDH, 1 is MDH)
3) `X_test`, sequences of testing set
4) `y_test`, labels of testing set (0 is non-MDH, 1 is MDH)
5) `mdh_close_ec_test.csv`, contains sequences of EC1.1.1.X, EC1.1.X.X, and sequences from `val_sequences.fasta`
6) `cdhit/all_MDHs_for_cdhit.fa`, `train_sequences.fasta` with modified headers for cd-hit clustering
7) `cdhit/cluster765.txt`, finetuning input file for cluster765; `cdhit/cluster765.fa`, the fasta format
8) `cdhit/cluster2029.txt`, finetuning input file for cluster2029; `cdhit/cluster2029.fa`, the fasta format
9) `cdhit/cluster5987.txt`, finetuning input file for cluster5987; `cdhit/cluster5987.fa`, the fasta format
10) `cdhit/cluster7477.txt`, finetuning input file for cluster7477; `cdhit/cluster7477.fa`, the fasta format
11) `cluster_summary` (pickle file), summarizes the clustering result (seq id and correpsonding representative seq)

**Script**  
`data_preparation.py`  

### STEP 2: data embedding and discriminator training
**Inputs**
1) `X_train`
2) `y_train`
3) `X_test`
4) `y_test`

**Outputs**  
`model` (the discriminator, a directory with Keras model object). This object is not included in the repository due to file size limitation, but it can be easily reproduced following the python script. Please make sure to specify the path to the model.

**Script**  
`discriminator_training.py`  
**NOTE:** embedding all (33k) sequences will take a long time, we recommend submitting the job to a server.

### STEP 3: finetuning ProtGPT2
**Inputs**  
1) `cdhit/cluster765.txt`
2) `cdhit/cluster2029.txt`
3) `cdhit/cluster5987.txt`
4) `cdhit/cluster7477.txt`

**Outputs**  
Four directories containing the finetuned ProtGPT2 models finetuned with cluster765, cluster2029, cluster5987, and cluster7477, respectively.  

**Script**  
`finetune.sh`  
**NOTE**: Please make sure to specify the paths to input files and to the output.

### STEP 4: sequence generation
**Inputs**
1) `cluster_summary`
2) `train_sequences.fasta`

**Outputs**  
`all_gen.csv`, protein sequences generated by the four finetuned ProtGPT2 models

**Script**  
`generation.py`  
**NOTE:** Please make sure to specify the paths to the finetuned ProtGPT2 models.

### STEP 5: evaluation (predictions) on the generated sequences
**Inputs**
1) `model` (the discriminator, a directory with Keras model object). This object is not included in the repository due to file size limitation, but it can be easily reproduced following the python script. Please make sure to specify the path to the model.
2) `all_gen.csv`

**Outputs**  
`all_gen.csv`, containing sequences and predictions for random sequences, non-finetuned-generated sequences, 
and finetuned-generated sequences. Overiding the original `all_gen.csv`.

**Script**  
`evaluation.py`  
**NOTE:** Please make sure to specify the path to `model` (the discrminator). Embedding the sequences will take a long time, we recommend
 submitting the job to a server.

### STEP 6: analyze the results (best hit identity, Pfam MDH domain, redundancy)
Follow `setup_and_run_hmmscan.sh` to set-up and run Pfam domain scan on all sequences.

**Inputs**  
`all_gen.csv`

**Outputs**  
`all_gen.csv`, with annotations, overiding the original `all_gen.csv`.

**Script**  
`analysis.py`









