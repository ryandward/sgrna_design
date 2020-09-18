********************************************
sgRNA Design Scripts for CRISPRi in Bacteria
********************************************

This set of instructions is meant to be used as a guide to automate the workflow for designing guides. However, the script **build_sgrna_library.py** is available for advanced users to incorporate into their own scripts.

These sgRNA design scripts are forked from https://github.com/traeki/sgrna_design. These have been modified to yield zero-based output, with coordinates corresponding to the genome of reference. Additionally, NCBI Entrez Direct Utilities commands have been given as suggestions.



---------------------------------------------


********************************************
Quick-start Guide
********************************************


Using conda, create an environment to acquire the dependencies and their appropriate versions.
=============================================
Throughout this guide, we call the environment **sgrna_design**.

``conda create -n sgrna_design -c bioconda 'bowtie=1.2.3' biopython pysam entrez-direct git 'python>3'``

---------------------------------------------

Activate the environment.
=============================================

``conda activate sgrna_design``

---------------------------------------------

Create a local copy of this repository and move into sgrna_design directory.
=============================================

This step automates downloading the current iteration of this **sgrna_design** project into your present directory on your commandline. Navigate somewhere you'd like to install before continuing. After successfully acquiring this project, you change into the sgrna_design directory.

``git clone https://github.com/ryandward/sgrna_design.git && cd sgrna_design``

---------------------------------------------


Set a envionment variable equal to the accession number of the chromosome of interest.
=============================================

In any POSIX shell -- including bash, zsh, and fish -- the syntax is the same to define a variable.

In this case we have chosen **U00096.3**, the circular chromosome from *Escherichia coli* MG1655. The accession number is listed underneath the chromosome title -- here as **GenBank: U00096.3** from https://www.ncbi.nlm.nih.gov/nuccore/U00096. If you're not sure, check out the link and search for your organism.

``GUIDE_TARGET="U00096.3"``

---------------------------------------------


Fetch the chromosome feature file, in genbank format.
=============================================

Since the environment contains the NCBI Entrez Direct Utilities package, it is also **highly** recommended to download the bacterial chromosomes directly from NCBI. This file is used as sole input to extract suitable sgRNA targets.


``efetch -db nuccore -format gb -id $GUIDE_TARGET > ${GUIDE_TARGET}.gb && file ${GUIDE_TARGET}.gb | grep -iq ascii && echo "File contains data, continue to next step." || echo "Emtpy file, try efetch step again."``

NCBI sometimes fails without warning, a failsafe has been built into this step. If you see the warning *Emtpy file, try efetch step again.*, just hit the up arrow, and re-run this command.

---------------------------------------------

Use build_sgrna_library.py to generate a list of sgRNA targets.
=============================================

It is recommended to use the following parameters to run the script, and will work **as is**, considering previous steps have been followed.

``./build_sgrna_library.py --input_genbank_genome_name ${GUIDE_TARGET}.gb  --tsv_output_file ${GUIDE_TARGET}_sgrna.tsv && echo "Output stored in ${GUIDE_TARGET}_sgrna.tsv"``

---------------------------------------------

Accessing Results
=============================================

Results will be listed in a tab-separated variable (tsv) formatted file corresponding to the chromosome defined above as GUIDE_TARGET by appending **_sgrna.tsv**.

In this example, view the file **U00096.3_sgrna.tsv**. This file is fully compatible with both LibreOffice and Excel. Ensure that when importing, you choose variable length columns with **tab** as the delimiter.

Briefly check that the results are available before moving on.

``less ${GUIDE_TARGET}_sgrna.tsv``

---------------------------------------------

Formatting Results as Bed File (Optional)
=============================================

If you plan to use the output for downstream processing, you can reformat the output as a bed file; the coordinates are standard zero-width compatible.

+----------+----------+-----------+---------+---+------+----------------+-----------+
|Chromosome|Left Coord|Right Coord|Locus Tag|PAM|Strand|Sense/Anti-sense|Specificity|
+----------+----------+-----------+---------+---+------+----------------+-----------+

The following produces a bed file called the file **U00096.3_sgrna.bed**, but the prefix will change based on the GUIDE_TARGET variable.

``awk 'NR==1{next;}$8=="rev"{$8="-"} $8=="fwd"{$8="+"} {print $5, $6, $7, $1, $4, $8, $9, $10}' ${GUIDE_TARGET}_sgrna.tsv > ${GUIDE_TARGET}_sgrna.bed``

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
