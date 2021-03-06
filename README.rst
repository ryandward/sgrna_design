********************************************
sgRNA Design Scripts for CRISPRi in Bacteria
********************************************

This set of instructions is meant to be used as a guide to automate the workflow for designing guides. However, the script **build_sgrna_library.py** is available for advanced users to incorporate into their own scripts.

These sgRNA design scripts are forked from initial work provided by John S. Hawkins (https://github.com/traeki/sgrna_design). Additionally, NCBI Entrez Direct Utilities commands have been given as suggestions.



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

Activate the environment to load script dependencies. This is required every time a new terminal window is opened.

::

  conda activate sgrna_design

---------------------------------------------

Obtain Genome Sequence Data from NCBI Using the Accession Number (Not RefSeq!)
=============================================

If not known, consult the NCBI Nucleotide Database (https://www.ncbi.nlm.nih.gov/nucleotide/) to locate the accession number corresponding to the chromosome of interest and adjust the environment variable “ACC_NO” accordingly. This example uses the E. coli MG1655 chromosome “U00096.3”, which number will serve as the template for the names of all other files.

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

::

  ./build_sgrna_library.py \
  --input_genbank_genome_name ${ACC_NO}.gb \
  --tsv_output_file ${ACC_NO}_sgrna.tsv && 
  echo "Output saved as ${ACC_NO}_sgrna.tsv"
---------------------------------------------



Formatting Results as Bed File (Optional)
=============================================

If you want to use the output for downstream applications, you can reformat the output as a ``bed`` file.
Consider using my tsv parser if you do a lot of genome arithmetic, converting between formats can be a 
huge pain. 

The ``sele`` command functions like an ad-hoc ``select`` command such as you might find in an 
SQL database, but much more convenient -- since the input is a tsv file, and runs in a normal bash environment. 

**More instructions here:** https://github.com/ryandward/sele

The following produces a bed file called the file **U00096.3_sgrna.bed**, 
the file will be named based on the ACC_NO variable.

To create a bed file six fields are usually required, fields 1, 2, 3, and 6 are invariable. 
The rest can be used to cram information into downstream analysis. 


+----------+----------+-----------+---------+---+------+----------------+-----------+
|Chromosome|Left Coord|Right Coord|Locus Tag|PAM|Strand|Sense/Anti-sense|Specificity|
+----------+----------+-----------+---------+---+------+----------------+-----------+


::

  # method 1:
  # using my tsv parser, columns are callable via the header line.
  # moreover, names can be matched and conditionally updated on the fly.
  # in this case, turning "rev" into "-" and "fwd" into "+".
  
  # -q for "quiet", i.e. suppress header
  # -c for "column", i.e. which headers to select 
  # -u for "update", i.e. conditionally modify output
  # -w for "where", i.e. conditionally filter output
  
  sele -q \                                                      
    -c 'chrom,start,end,gene,pam,repldir,transdir,specificity' \ 
    -u 'repldir==fwd:repldir=+,repldir==rev:repldir=-' \         
   ${ACC_NO}_sgrna.tsv > ${ACC_NO}_sgrna.bed
  

::

  # method 2:
  # using bare awk can be tough, since the fields are referenced
  # by order. every subsequent step could reorder fields 
  # For instance,  $1 does not intrinsically correspond to
  # chromosome.
  
  awk 'BEGIN  {FS = "\t" ; OFS = "\t"}
  NR==1       { next }
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
