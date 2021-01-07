
# HIPPS-DIMES
Maximum Entropy Based HI-C/Distance Map - Polymer Physics - Structures Method

# Description

This python script can be used to generate ensemble of genome structures from Hi-C contact map or mean spatial distance map. The method is based on Maximum Entropy principle and the relation between the contact probability and the mean spatial distance from polymer theory. The application of the method can be found in this work https://www.biorxiv.org/content/10.1101/2020.05.21.109421v1.

In general, this script accepts the input file of a Hi-C contact or a mean spatial distance map (can be measured in Multiplex FISH experiment), and generate an ensemble of individual conformations consistent with the inputted contact map or distance map. The output conformations are in `.xyz` format, which users can use to calculate quantities of interest and can be viewed using `VMD` or other compatible softwares.

# Documentation

## Install

First, download this repository using,

```bash
git clone https://github.com/anyuzx/HIPPS-DIMES
```

Next, install required packages using the command below,

```bash
pip install --editable .
```

This command will install the required packages, and install the script as a python module. Once installed, you can call `HippsDimes` directly in the terminal to run the script. The packages installed are:

* `Click`
* `Numpy`
* `Scipy`
* `Pandas`
* `Tqdm`
* `Cooler`

We recommend run this script using Python 3.8+

## How to use

### To get started

To get started, please go through the jupyter notebook `walkthrough.ipynb`.

In addition, it is helpful to view the help information for each arguments and options. To display help information, use

```bash
HippsDimes --help
```

### Input files

This script accept input files in two formats. If the input file is a Hi-C contact map, it can be in `cooler` format or pure text format. If the input file is a mean spatial distance map, the script only accepts a pure text formatted file. The text format for a matrix is the following: each row of the file corresponds to the row of the matrix. Each value is separated by commas. An example of such file is,

``` text
1, 2, 3
2, 1, 2
3, 2, 1
```

### Output files

This script will generate several files:

- A text file for the final simulated mean distance map
- A text file for the connectivity matrix
- A `.xyz` formatted file for the ensemble of genome structures generated (can be turned off)
- A csv formatted file for cost versus iteration data (can be turned off)

### Examples

First, download a cooler format Hi-C contact map from [here](https://drive.google.com/file/d/1eIxGv1JbIrEAVoUSQK_O_ebIjWo6toTJ/view?usp=sharing). This Hi-C contact map is for Chicken cell mitotic chromosome, originally retrieved from [GEO repository](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE102740). Rename it to `hic_example.cool`. Then execute the following command,

```bash
HippsDimes hic_example.cool test --input-type cmap --input-format cooler -s chr7:10M-15M -i 10 -e 10
```

This command tells the script to load the Hi-C contact map `hic_example.cool` and perform the iterative scaling algorithm. The argument `test` instructs the files names of output files start with `test_`. Option `--input-type cmap` specifies that the input file is a contact map. Option `--input-format cooler` specifies that the input file is a `cooler` file. Option `-s chr7:10M-15M` specifies that the algorithm is performed on the region 10 Mbps - 15 Mbps on Chromosome 7. Note that these three options are required and cannot be neglected. **Some option arguments are optional, some are required. Please refer to the section below for details**

When the program finishes, the script will generate several output files: `test.xyz`, `test_connectivity_matrix.txt`, and `test_dmap_final.txt`. `test.xyz` contains 10 sets of individual conformations of x, y, z coordinates and can be viewed using `VMD` or other compatible visualization softwares.

### Explanantion of the arguments and options

#### Argument

- `INPUT`: File path for the input file. The input file can be a Hi-C contact map or a mean spatial distance map as measured in Multiplexed FISH experiment.
- `OUTPUT_PREFIX`: Prefix for outputfiles. For instance, if one specify it to be `TEST`, then all the output files will start with `TEST_`.

#### Options

- `-e` or `--ensemble`: Number of individual conformations to be generated. This script will generate an ensemble of structures consistent with the input Hi-C contact map or the mean spatial distance map. Each individual conformations are different from each other. You can specify how many such individual conformations you want to generate.
- `-a` or `--alpha`: Value of the contact map to distance map conversion exponent. If the input file is Hi-C contact map, the method first convert the contact map to a mean spatial distance map. The equation of the conversion is d_{ij} ~ c_{ij}^{1/\alpha}. The default value of \alpha is 4.0, estimated in this work 10.1126/science.aaf8084
- `-s` or `--selection`: Specify chromosome or region. This option only works when the input file has [`cooler`](https://github.com/open2c/cooler) format. The value of this option is passed to the `cooler.Cooler.matrix().fetch()` method. For details, please refer their [documentation](https://cooler.readthedocs.io/en/latest/concepts.html#matrix-selector)
- `-i` or `--iteration`: The method relies on iterative scaling to find the optimal parameters. This option specifies the number of iterations. Generally, the more iterations the model runs, the better results are. However, the convergence of the model slow down when iteration increases. For larger size of contact map and the mean distance map, the number of iterations needed to good convergence is larger.
- `-r` or `--learning-rate`: The learning rate for iterative scaling. Higher learning rate achieves faster convergence. However, the model can crash if learning rate is too large. The default value is 10. One should play around this option to see what works best.
- `--input-type`: The type of the input file. To use the script, the type must be specified. The method can work on both the contact map (`cmap`) or distance map (`dmap`).
- `--input-format`: The format of the input file. If the type of input file is Hi-C contact map, then the script support `cooler` format Hi-C contact map file or a pure text based file. In the text based file, each line corresponds to the row of the contact map. If the type of input file is mean distance map, then the script only support the text based file in which each line represents the row of the mean distance map.
- `--log`: A log file will be written if this option is specified. The log file contains the data of cost versus iteration
- `--no-xyzs`: Turn off writing x,y,z coordinates of genome structures to files.
