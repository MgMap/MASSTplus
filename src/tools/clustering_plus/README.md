# CLUSTERING+


CLUSTERING+ is a part of NETWORKING+ , an improvement on the GNPS spectrometry networking tool. CLUSTERING+ removes redundant spectras of the same peptide by replacing them with a single representative spectrum. CLUSTERING+ reduces the clustering time of MS-Cluster by over two orders of magnitude. It's capable of performing clustering on the whole GNPS dataset and remove redundant spectras of the same peptide.


#Using CLUSTERING+ Source Code
The clustering_plus binary integrates the tools needed for reading and clustering mass spectrum database. The binary is only for x84-64 linux systems. You can always consult `clustering_plus -h` for a detailed description of the available commands and flags

### Compile source code with O3 flag:
```
    g++ -O3 clustering_plus.cpp -o clustering_plus
```

### Usage
 To perform clustering on a single spectra file, run the following
```sh
  ./clustering_plus [OPTIONS] -i <PATH> -o <PATH>...
```
 To perform clustering on a file containing paths to a list of spectra files, run the following

 ```sh
  ./clustering_plus [OPTIONS] -l <PATH> -o <PATH>...
```
The `-o <PATH>` should take in the path of the output file containing the cluster information. The threshold can be modified through the argument `-t <THRESHOLD>` , which is set to be 0.7 by default.

When unsure about parameters, it is sufficient to provide only the  `-i <PATH>` or `-l <PATH>` and `-o <PATHS>` arguments. CLUSTERING+ will set `[OPTIONS]` to default.

Note that you should only provide either `-i <PATH>` and `-l <PATH>`, when using `-l <PATH>` flag, the argument should be a file in which each line corresponds to the path of a spectra file. 


<details>
<summary>Available options</summary>

```
OPTIONS:
    -i <PATH>,
        This field is a path to a single input spectra file.
    -l <PATH>
        This field is a path to a list file containing the paths to input spectra files
    -o <output_path> 
        output path for cluster results 
    
    -h, --help
            Print help information
    -t <THRESHOLD>
        The minimum similarity threshold with a cluster center for joining the cluster 
        [default: 0.7]
    
    -c <CLUSTER_CENTER>
        Output path containing the representative spectra of each cluster
        
        [default: centers.mgf]
    
    -s <CLUSTER_INFORMATION>
        Output CSV file containing the content information of each cluster
        
        [default: cluster_info.csv]
    
    -f <PEAK_FILTER_WINDOW> <TOP_PEAKS>
        Window size for peak filtering (Da) and number of preserved peaks in each window
        This value is used in the filtering pre-processing step of the spectrum. This removes
        low intensity peaks by keeping only the <TOP_PEAKS> most intense peaks in non-overlapping m/z windows of size <PEAK_FILTER_WINDOW>. 
        
        [default: 50.0, 5]
    
    --pepmass_resolution
        Absolute pepmass tolerance (in Daltons)
        
        This error-tolerance is used during the indexing process to divide the spectra into bins
        of approximately similar pepmass. While scoring a query spectrum we only look at the
        spectra in the same bin (and a couple neighboring bins) as the query spectrum's pepmass.
        The indexing strategy relies on this - a smaller tolerance means we divide spectra into
        more bins and this implies a more fine-grained comparison.
        
        [default: 1.0]
        
    --peak_resolution  <PEAK_RESOLUTION>
        Absolute peak m/z tolerance 
        
        The error tolerance used to match query spectra peak m/zs to cluster center spectra m/zs.
        Peaks from cluster center spectras that have a difference in m/z bigger than this are
        considered to have distinct m/zs, otherwise they are considered to be (approximately)
        equal. The indexing strategy relies on this - a smaller tolerance means we divide 
        spectras into more bins and this implies a more fine-grained comparison.
        
        [default: 0.01]
    
    --topfour_threshold <TOPFOUR_THRESHOLD>
        The minimum number of spectra within within the same --pepmass_resolution range to apply top-four peaks filtering
        
        Top-four peaks filtering only considers the possibility of a query belonging to a cluster 
        
        if there are matched peaks within --topfour_resolution between the top four peaks of the query and top four peaks of the cluster center.
        
        can be skipped with --no-topfour.
        
        [default: 5000]
        
    --topfour_resolution <TOPFOUR_RESOLUTION>
        Absolute m/z tolerance of topfour peaks.
        
        The error tolerance used to match query spectra peak m/zs to cluster center spectra m/zs when conducting topfour filtering.
        
        can be skipped with --no-topfour.
        
        [default: 0.1]
         
    --no-topfour
        Disable topfour filtering
    
    --cluster_minsize <CLUSTER_MINSIZE>: 
        the minimum cluster size for writing the cluster center into the output file
        
        Usually for each peptide we observe multiple spectras in the database. Small clusters or singletons are often resulted from noise when generating mass spectrometry file.
        
        Clusters smaller than this amount will be considered as noise data and filtered out before writing the output file

        [default: 2]
    
    --connected_components_out <CONNECTED_COMPONENTS_OUT>
        The output file containing the connected components information of spectral networking
        
        [default:  cluster_info_with_cc.tsv]
        
    --bruteforce 
        Use bruteforce clustering technique rather than indexing 
        
        The clustering will be much slower if applied
```
</details>

### Examples 
```sh
./clustering_plus -i path/to/input.mgf  -o path/to/out.tsv 
```
This reads in the spectras from input.mgf and outputs the clustering results to out.tsv with the default options

```sh
./pairing_plus -l path/to/list.txt  -o path/to/out.tsv 
```
This reads in the spectras from spectra file path contained in list.txt and outputs the clustering results to out.tsv with the default options

```sh
./clustering_plus -i path/to/list.txt  -o path/to/out.tsv -t 0.9 --no_topfour
```
This reads in the spectras from input.mgf and outputs the clustering results to out.tsv. The exact similarity score has to be at least 0.9 between a query spectra and cluster center for joining the cluster. The clustering performs without top four filtering.

```sh
./clustering_plus -l path/to/list.txt  -o path/to/out.tsv -t 0.9 --peak_resolution 0.02 --pepmass_resolution 2.0
```
This reads in the spectras from spectra file path contained in list.txt and outputs the networking dot product results to out.tsv. The query spectra cluster center need to have a pepmass difference within 2.0 Dalton range and a similarity score of at least 0.9 for the spectra to be added to the cluster. When calculating the similarity scores between query spectra and clsuter center, their peaks needs to be within 0.02 Dalton range to be considered in the same location. 


###Result Format
The output will be in `tsv` format. Each row of the `tsv` output represents the cluster information of a spectra:
- `cluster_idx` is a unique ID assigned to the cluster containing this spectra
- `scan` is a unique ID assigned to the spectra
- `mz` is the precursor mass of the spectra
- `RTINSECONDS` is the 
- `dot_product` is the similarity score between the two spectras
- `dot_product_shared` is the shared peak product component in the similarity score
- `dot_product_shifted` is shifted peak product component in the similarity score

An example output is as follows:

|scan_1 |  mz_1 |   scan_2 | mz_2 | dot_product | dot_product_shared | dot_product_shifted
| --- | --- | --- | --- | --- | --- | --- |
| 8453819 | 2223.57 | 8453821 | 2224.57 | 0.948327 | 0.948327 | 0 |
| 8453818 | 2222.58 | 8453820 | 2223.57 | 0.970785 | 0.970785 | 0 |
| 8453817 | 2221.58 | 8453818 | 2222.58 | 0.981046 | 0.981046 | 0 |
| 8453817 | 2221.58 | 8453820 | 2223.57 | 0.967376 | 0.967376 | 0 |
| 8453815 | 2211.6  | 8453819 | 2223.57 | 0.949034 | 0.949034 | 0 |
| 8453815 | 2211.6  | 8453821 | 2224.57 | 0.945403 | 0.945403 | 0 |
