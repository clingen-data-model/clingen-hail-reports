# gnomad-frequency-report
Performs filtering against gnomAD and ClinVar datasets. Uses Hail to report records with a population FAF above certain thresholds by gene.

## Configuring Hail

To configure Hail, which this project depends on, please see `hail-instructions.md`.

## Running

This can either be utilized via the Jupyter notebook `Clingen-Reports.ipynb` which can be opened in the Jupyter notebook environment launched in the Hail connect step, or via a command line utility called `clingen-gnomad-reporting.py`.
