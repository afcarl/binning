Requirements
==========
The current pipeline has been developed and used only on Linux. It should run on all nix platforms.
The current pipeline mostly uses a Condor cluster for running jobs on distributed systems in parallel.
However, changing the pipeline to work with other clusters should be relatively straight-forward. 

The following are required.
 - python 2.7
 - Dendropy 3.12 or later
 - java 


Installation:
=========
After installing the requirements, unzip the code directory to a location of choice and define `BIN_HOME`
environmental variable to the location where the files are extracted. 

USAGE:
========
The assumption of the pipeline is that you have all your initial gene trees with support values drawn on them
in a certain directory (we will call it `genes_dir`). Inside `genes_dir`, you need to have one folder per gene. 
Inside that folder, you need to have the alignments and the gene tree with support values. The name of each 
gene (`gene_name`) is just the name of its directory. The name of the alignment file should be `gene_name.fasta`.
So, if your gene is called `300`, your alignment should be called `300.fasta`. The tree file can be named anything, 
*However*, all the trees should have the same exact name. So, for example, the tree for gene 300 could be under
directory `genes_dir/300/RAxML_bootstrap.final`, and if so, the tree for gene 301 should be under
`genes_dir/300/RAxML_bootstrap.final`. We will refer to the name of your tree file as `tree_file_name`.

Once you have your initial gene trees in this structure, to get the bins, you need to:

1- If you have condor, run:
``` 
   $BIN_HOME/makecondor.compatibility.sh [genes_dir] [support_threshold (e.g. 50)] [pairwise_output_dir] [tree_file_name]
```
   This step generates a condor file (`condor.compat.[support]`) that has all the commands required for calculating all pairwise compatibilities.

   If you don't have condor, instead run:
``` 
   $BIN_HOME/makecommands.compatibility.sh [genes_dir] [support_threshold (e.g. 50)] [pairwise_output_dir] [tree_file_name]
```
   This creates a bash file that includes all the commands that need to be run.
 
2- run the condor file using `condor_submit condor.compat.[threshold]`.
   Alternatively, if you used makecommands in the previous step, use your cluster system to run all the commands in the commands file. 
   If you don't have a cluster, just run `sh commands.compat.[threshold]` 

3- Once the condor jobs have finished (i.e. your runs from previous step have finished), it is time to build the bin definitions.
   run (be sure to replace 50 with support threshold you used in step 1):
``` 
   cd [pairwise_output_dir]; 
   ls| grep -v ge|sed -e "s/.50$//g" > genes   
   python $BIN_HOME/cluster_genetrees.py genes 50
```
  once this finished, you have all your bins defined in text files called bin.0.txt, bin.1.txt, etc inside [pairwise_output_dir]. 
  You can look at these and examine bin sizes if you want. 

4- Now is time to actually concatenate all the gene alignments for each bin and to create the supergene alignments.
   Go to the place where you want to have your supergene alignments saved:
```
   $BIN_HOME/build.supergene.alignments.sh [pairwise_output_dir] [genes_dir]
``` 
   This will create a directory called supergenes and will put all the supergene alignments in there. 


Now you can now use your favorite tree estimation software to estimate gene trees for each of these supergenes. 


Acknowledgment
========
In our pipeline, we use few scripts developed by others:

1. Our graph coloring code is a modification of [code](http://shah.freeshell.org/graphcoloring/) by Shalin Shah
2. Our compatibility code is based on the older versions of [phylonet](http://bioinfo.cs.rice.edu/phylonet) with some reverse-engineering and code modifications
3. Our code to concatenate alignments is based on a perl scripts that has been lying around in our lab and we need to figure out who develped it (maybe [Kevin Liu](http://www.cse.msu.edu/~kjl/) or [Li-Sang Wang](http://tesla.pcbi.upenn.edu/~lswang/)).
