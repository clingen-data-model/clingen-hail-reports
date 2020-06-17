# gnomad-frequency-report
Performs filtering against gnomAD and ClinVar datasets. Uses Hail to report records with a population FAF above certain thresholds by gene.

## Configuring Hail

To configure Hail, which this project depends on, please see `hail-instructions.md`.

## Running

This can either be utilized via the Jupyter notebook `Clingen-Gnomad-FAF-Report.ipynb` which can be opened in the Jupyter notebook environment launched in the Hail connect step.

After running the whole notebook, a file, by default called `report.tsv`, will be output into the same cloud storage directory that the notebook is running in.
