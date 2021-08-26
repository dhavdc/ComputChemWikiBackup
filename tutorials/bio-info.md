---
title: Bioinformatics / Protein Modeling
description: 
published: true
date: 2020-11-13T16:35:42.547Z
tags: 
editor: markdown
dateCreated: 2020-08-25T16:31:26.384Z
---

# Bioinformatics / Protein Modeling
## Repair protein structure:
Step 1 - Download PDB from rcsb website (preferable biological unit of the protein)
Step 2 - Check for missing residues in pymol (switch on sequence in pymol, look for residues in grey colour)
Step 3 - You can also download complete sequence from UNIPROT (see below for TIPS)
Step 4 - Copy paste the sequence to the modelling webserver (see below for webservers and tips)
TIP: If you want to repair your structure, use "User Template" base modelling and give your pdb structure as Template.


## Homology modelling:
Step 1- Download the complete sequence from the UNIPROT
Step 2- Run BLAST for finding templates (Tip: you can run withing the modeling webserver or separately)
Step 3- Perform sequence alignment (See below for webservers)
Step 4- built homology model
TIP: Choose templates to use before modelling on the basis of literature or sequence identity score (it will increase the speed of the process)


**Download sequence from UNIPROT**
TIP1: Find UNIPROT ID for the protein from rcsb 
TIP2: Always download "fasta" file 
TIP3: You can put multiple sequences in the basket on uniprot and run sequence alignment on uniprot

## Webservers for Homology modelling:
1. SWISS-Model
2. CPHModel
Webservers for ab initio folding and threading method
1. ROBETTA
2. I-TASSER
3. Phyre2
4. RaptorX
Webserver for De novo protein structure prediction
1. trRosetta
2. EVfold
Webservers to perform sequence alignment for multiple proteins
1. Uniprot
2. Clustal W
3. Clustal Omega
4. NCBI BLAST



