# `UPECfinder` testing

## Results from completed Uropathogenic _E. coli_ genomes

Between 2018 and 2019, _E. coli_ was collected from patients with urinary tract infections (UTIs) in Peru. The isolates underwent whole genome sequencing (WGS), and the resulting genomes were identified as UPEC using `UPECfinder`.

### Commands used

To run `UPECfinder` in a directory named "genomes":

```{bash}
for fasta_file in "genomes"/*.fasta; do
    base_name=$(basename "$fasta_file" .fasta)
    camlhmp-blast-regions -i "$fasta_file" -y upec.yaml -t targets.fasta -p "$base_name" -o "results/$base_name.fasta"
done
```

To concatenate the output of all genomes into a single TSV file:

```{bash}
for dir in results/*/; do
    for tsv_file in "$dir"/*.tsv; do
        if [[ -f "$tsv_file" && ! "$tsv_file" =~ (blastn|details) ]]; then
            cat "$tsv_file" >> results.tsv
        fi
    done
done

awk '!seen[$0]++' results.tsv > temp && mv temp completed-genomes.tsv
```


