********************************************
sgRNA Design Scripts for CRISPRi in Bacteria
********************************************

This set of instructions is meant to be used as a guide to automate the workflow for designing guides. However, the script **build_sgrna_library.py** is available for advanced users to incorporate into their own scripts.

These sgRNA design scripts are forked from https://github.com/traeki/sgrna_design. These have been modified to yield zero-based output, with coordinates corresponding to the genome of reference. Additionally, NCBI Entrez Direct Utilities commands have been given as suggestions.



---------------------------------------------


********************************************
Quick-start Guide
********************************************


Download Dependencies for sgRNA Design Script
=============================================

Prepare a conda environment: “sgrna_design” to host the sgRNA design scripts. It is only necessary to perform environment creation once.

::
  
  conda create -n sgrna_design -c bioconda 'bowtie=1.2.3' biopython pysam entrez-direct git 'python>3'

Retrieve the “sgrna_design” project from GitHub, then move into the newly created directory. It is only necessary to perform project download once.

::

  git clone https://github.com/ryandward/sgrna_design.git && 
  cd sgrna_design

---------------------------------------------

Activate Conda Environment
=============================================

Activate the environment to load script dependencies This is required every time a new terminal window is opened.

::

  conda activate sgrna_design

---------------------------------------------

Obtain Genome Sequence Data from NCBI Using the Accession Number
=============================================

If not known, consult the NCBI Nucleotide Database (see Internet Resources) to locate the accession number corresponding to the chromosome of interest and adjust the environment variable “ACC_NO” accordingly. This example uses the E. coli MG1655 chromosome “U00096.3”, which number will serve as the template for the names of all other files.

::

  ACC_NO="U00096.3"

Retrieve and save the genbank chromosome – here automatically named “U00096.3.gb”. This command also issues a warning if NCBI returns an empty response and may be run multiple times as needed.

::

  efetch -db nuccore \
  -format gb \
  -id $ACC_NO > ${ACC_NO}.gb && 
  file ${ACC_NO}.gb | 
  grep -iq ascii && 
  echo "File contains data, continue." || 
  echo "Empty file, retry this step."


---------------------------------------------


Run sgRNA Design Script
=============================================

Run the main python script, producing a tab-separated variable file corresponding to the accession number appended appended by “_sgrna”. Upon successful completion, this command also confirms the name of the results file – here “U00096.3_sgrna.tsv”.

In this case we have chosen **U00096.3**, the circular chromosome from *Escherichia coli* MG1655. The accession number is listed underneath the chromosome title -- here as **GenBank: U00096.3** from https://www.ncbi.nlm.nih.gov/nuccore/U00096. If you're not sure, check out the link and search for your organism.

::

  ./build_sgrna_library.py \
  --input_genbank_genome_name ${ACC_NO}.gb \
  --tsv_output_file ${ACC_NO}_sgrna.tsv && 
  echo "Output saved as ${ACC_NO}_sgrna.tsv"
---------------------------------------------



Formatting Results as Bed File (Optional)
=============================================

If you plan to use the output for downstream processing, you can reformat the output as a bed file; the coordinates are standard zero-width compatible.

+----------+----------+-----------+---------+---+------+----------------+-----------+
|Chromosome|Left Coord|Right Coord|Locus Tag|PAM|Strand|Sense/Anti-sense|Specificity|
+----------+----------+-----------+---------+---+------+----------------+-----------+

The following produces a bed file called the file **U00096.3_sgrna.bed**, but the prefix will change based on the ACC_NO variable.

::

  awk 'BEGIN {FS = "\t" ; OFS = "\t"}
  NR==1{ next }
  $8 == "rev" { $8 = "-" } 
  $8 == "fwd" { $8 = "+" } 
  { print $5, $6, $7, $1, $4, $8, $9, $10 }' \
  ${ACC_NO}_sgrna.tsv > ${ACC_NO}_sgrna.bed

---------------------------------------------


Notes from the orignal branch:
=============================================

Author: John S. Hawkins [really@gmail.com]

For bacteria we suggest using guides that

*   have a small, positive offset

*   are on the antisense strand ('anti' in the 'transdir' column)

*   have a SPECIFICITY score of 39

If a guide meeting these criteria is not available, lower specificity can be
used, but you should check for near-matches elsewhere in the genome to see if
they are likely to cause issues.  Guides on the 'sense' strand are not
recommended.  They generally have a greatly reduced, and hard to predict, level
of effect.  If reduced effect is desired, we suggest the use of
http://www.github.com/traeki/mismatch_crispri to achieve more reliable
outcomes.
