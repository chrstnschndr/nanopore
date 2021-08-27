# Nanopore Basic Pre-Processing of cDNA seq
Basecalling, Mapping, Removing Duplicates, Quantification. In this case, the input is barcoded de-multiplexed inputs sorted into according folders (bc0-9).

### Content
[Basecalling](#Basecalling)  
[Mapping](#Mapping)  
[Duplicate Removal](#Duplicate Removal) 
[Quantification](#Quantification) 
<a name="headers"/>

# Basecalling
The basecalling is performed on an Ubuntu 20 device using Cuda Nvidia GPU together with the Nanopore Basecaller Guppy.
```bash
sudo ./guppy_basecaller --input_path <input> --save_path <save> --flowcell FLO-FLG001 --kit SQK-LSK109 -x cuda:all:100%
```

# Mapping
Mapping is achieved by using minimap2 against h38_from_genomic.fna file
```bash
mkdir ./aligned ./quant

for bc in {0..9}
do minimap2 -ax map-ont --secondary=no -N 0 <h38_frna_from_genomic.fna>
<in_fasta_path>/${bc}*.fasta > <out_sam_path>bc${bc}.sam
done
```

# Duplicate Removal
Duplicates and secondary alignments are removed by using pysam
```bash
import pysam
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt; plt.rcdefaults()
import os

def sam_filterer(infile, outfile):
    in_sam_file = pysam.AlignmentFile(infile, "r")
    out_path = open(outfile, "w")
    out_sam_file = pysam.AlignmentFile(out_path, "wh", template=in_sam_file)
    for read in in_sam_file:
        if read.is_supplementary == False and read.is_secondary == False:
            out_sam_file.write(read)
            
for file in os.listdir(<"in_path">):
    sam_filterer(f"<in_path/>{file}", f"<out_path/>{file}")
```

# Quantification
Read-count matrix like quantification is achieved by the salmon package, using the same h38_frna_from_genomic.fna file as for mapping.
```bash
for bc in {0..9}
do mkdir <salmon_out_path>
do salmon quant -p 6 -t  <path_fna_ref> --noErrorModel  -l SF -a <sam_in_path>.sam -o <salmon_out_path/>bc${bc}
done
```
