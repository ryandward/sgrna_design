Note: These sgRNA design scripts are forked from https://github.com/traeki/sgrna_design. These have been modified to yield zero-based output, with coordinates corresponding to the genome of reference. Additionally, NCBI Entrez Direct Utilities commands have been given as suggestions.

-------------------------------------------------
**Before installing, create an appropriate Conda environment by following this guide**
-------------------------------------------------

It is recommended to use the following command to obtain the prerequisites using Conda in a new environment called "sgrna_design":
::
    conda create -n sgrna_design -c bioconda 'bowtie=1.2.3' biopython pysam entrez-direct git 'python>3'

Activate the Conda environment called "sgrna_design":
::
    conda activate -n sgrna_design
    
Create a local copy of this repository and move into that directory:
::
    git clone https://github.com/ryandward/sgrna_design.git && cd sgrna_design

Since the environment contains the NCBI Entrez Direct Utilities package, it is also recommended to download the bacterial chromosomes directly from NCBI. 

First, set a bash variable equal to the chromosome of interest, in this case we have chosen "U00096.3".
::
    GUIDE_TARGET="U00096.3"

Download the chromosome genbank file directly from NCBI:
::
    efetch -db nuccore -format gb -id $GUIDE_TARGET > ${GUIDE_TARGET}.gb && file ${GUIDE_TARGET}.gb | grep -iq ascii && echo "File contains data, continue to next step." || echo "Emtpy file, try efetch step again."

Use build_sgrna_library.py to generate a list of sgRNA targets.
====================

Results will be given a name corresponding to the chromosome defined above as GUIDE_TARGET by appending _sgrna.tsv yielding: ${GUIDE_TARGET}_sgrna.tsv". 

For this example, the list is given as "U00096.3_sgrna.tsv" 
::
    ./build_sgrna_library.py --input_genbank_genome_name ${GUIDE_TARGET}.gb  --tsv_output_file ${GUIDE_TARGET}_sgrna.tsv && echo "Output stored in ${GUIDE_TARGET}_sgrna.tsv"
-------------------------------------

Notes from the orignal branch:
====================

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
