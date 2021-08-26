---
title: GBNeck2 CpHMD
description: 
published: true
date: 2021-02-24T16:07:11.117Z
tags: 
editor: markdown
dateCreated: 2020-10-06T17:30:08.098Z
---

# GBNeck2 implicit-solvent CpHMD 
## Introduction
GBNeck2-CpHMD is a continuous constant pH MD method using a generalized Born (GB) based implicit-solvent model for both conformational and protonation state sampling. It is implemented for both CPU (_pmemd.MPI_, see Huang, Harris, Shen, JCIM 2018) and GPU computing (_pmemd.cuda_, see Harris and Shen, JCIM 2019). GBNeck2-CpHMD is invoked by setting _iphmd=1_ and _igb=8_ in the input file _mdin_. The GPU accelerated GBNeck2-CpHMD is very fast (mins to hours) for predicting protein pKa's. A caveat is that conformational details may be as accurate as those obtained by PME all-atom CpHMD. 
## Tutorial
__Step1. Prepare a CpHMD compatible PDB file for the system__
In this step, we make sure CpHMD recognizes titratable residues. To do so, we add two dummy hydrogens to Asp (AS2), Glu (GL2), and make sure we use protonated His (HIP) and Cys (CYS). Currently, Asp, Glu, His, Cys, and Lys sidechains can be set to titratable in the CpHMD input file _phmdin_. In the future, Tyr, N- and C-terminus can be made titratatable (see how to obtain CpHMD model parameters). Note, the N-terminus has a solution pKa of 7.5 and C-terminus has a solution pKa of 3.8. Thus, if one is to simulate at pH 7, the N-terminus is likely titratable (i.e., changes its protonation state). To deal with this situation, we add capping group, for example, ACE for N terminus and NHE for C terminus. AS2 and GL2 are not in the standard _leap_ libraries but can be loaded from the _phmd.lib_ file (see _tleap_ example file below). 

__Step2. Generate topological file _rst7_ and parameter file _parm7_ using _tleap___
After we confirm the system PDB file is correct (by checking the PDB file and viewing the structure in Pymol/VMD...), we can use tleap to generate all input files for Amber simulations. Here is an example `build.in` file.
```build.in
# Reads in the FF
source $amberdir/dat/leap/cmd/leaprc.protein.ff14SB                      	# Loads Amber ff14SB force field where 'amberdir' is the Amber installation directory
set default PBradii mbondi3                                      					# Modifies GB radii to mbondi3 set, needed for IGB=8 this is the version of the GB
loadOff phmd.lib                                        		 							# Loads definitions for AS2 GL2 (ASPP AND GLUPP)   
phmdparm = loadamberparams frcmod.phmd           													# load modifications to bond length and dihedrals.
$protein = loadPDB $protein.pdb                                  					# Loads PDB file
saveamberparm $protein $protein.parm7 $protein.rst7              					# Generates the parameter and coordinate file
savepdb $protein $protein.pdb_tmp                    											# Saves the pdb file with the same atom numbering scheme wrt the parm7 file
quit
```
Note, if the PDB file doesn't contain correct disulfide information (make sure those residue names are CYX instead of CYS), we need to build them mannually. Here is a way to build them in terminal. If we define two arrays, one for one ends of the disulfide bonds, and one for the other corresponding ends of them:
```
disulfides_a=(159 221 273)
disulfides_b=(363 386 323)
# Here sulfur atom 159 bonds to sulfur atom 363, and so on.
```
We then add disulfide information into the `build.in` file:
```
for i in `seq 1 ${#disulfides_a[@]}`
do
 echo "bond ${protein}.$((${disulfides_a[$(($i-1))]}+1)).SG ${protein}.$((${disulfides_b[$(($i-1))]}+1)).SG" >> build.in
done
```
And finally we can feed it to _tleap_ as
```
tleap -f build.in > tleap.log
```
A further step is that we need to exclude the interactions between dummy hygrogens of GL2 and AS2 types, and modify GB radii for HIP, LYS, and CYS. The `processParm7.py` file in https://gitlab.com/rocharri/cphmd_tools fixes all those issues.
```
processParm7.py -f $protein.pdb_tmp -p $protein.parm7
mv $protein.parm7 orig_$protein.parm7  # To get a record of the original parm7 file
mv cphmd_$protein.parm7 $protein.parm7
```
__Step3. Minimization__
Now we have the necessary files `$protein.rst7` and `$protein.parm7`, we need to minimize the initial structure a little bit. Here is an example Amber _mdin_ input file `$protein_mini.mdin`:
```
Minimization
        &cntrl                                         	! cntrl is a name list, it stores all conrol parameters for amber
        imin = 1, maxcyc = 1000, ncyc = 500, 						! Do minimization, max number of steps (Run both SD and Conjugate Gradient), here first ncyc=500 steps are SD and the remaining maxcyc-ncyc=500 steps are CG.
        ntx = 1,                                     		! Initial coordinates
        ntwe = 0, ntwr = 50, ntpr = 50,        					! Print frq for energy and temp to mden file, write frq for restart trj, print frq for energy to mdout 
        ntc = 1, ntf = 1, ntb = 0, ntp = 0,          		! Shake (1 = No Shake), Force Eval. (1 = complete interaction is calced), Use PBC (1 = const. vol.), Use Const. Press. (0 = no press. scaling)
        cut = 999.0,                               			! Nonbond cutoff (Ang.); for implicit solvent we use a extreme number so that all nonbonded interactions are included.
        ntr = 1, restraintmask = ':1-$nrestr&!@H=',    	! restraint atoms (1 = yes), which atoms are restrained; $nrestr is number of residues in the system; we restrain all heavy atoms
        restraint_wt = 100.0,                   					! Harmonic force to be applied as the restraint
        ioutfm = 1, ntxo = 2,                        		! Fomrat of coor. and vel. trj files, write NetCDF restrt fuile for final coor., vel., and box size
        igb = 8,					 															! For GBNeck2 implicit solvent, igb=8
        /
```
We restrain all heavy atoms and minimize the built hydrogen atoms.
```
mpirun -n 4 pmemd.MPI -O -i ${protein}_mini.mdin -c ${protein}.rst7 -p ${protein}.parm7 \
        -ref ${protein}.rst7 -r ${protein}_mini.rst7 -o ${protein}_mini.out &>> minilogfile
```
__Step4. Equilibration__
Heating is not really necessary for implicit solvent and we can skip it. However, we need to equilibrate the system at a single pH for a while. The the best pH to use is the pH used to crystalize the protein. Here we show one example of an equilibration stage, but it's best practice to equilibrate in 4 stages with reducing force constant, see below for details. An example of an `equil.mdin` file:
``` 
                    &cntrl
                    imin = 0, nstlim = 2000, dt = 0.002,
                    irest = 0, ntx = 1,ig = -1,
                    temp0 = 300,
                    ntc = 2, ntf = 2, tol = 0.00001,
                    ntwx = 5000, ntwe = 5000, ntwr = 500000, ntpr = 500000,
                    cut = 999.0, iwrap = 0, igb = 8,           
                    ntt = 3, gamma_ln = 1.0, ntb = 0,                               ! ntp (1 = isotropic position scaling)
                    iphmd = 1, solvph = $crysph, saltcon = $conc,
                    ntr = 1, restraintmask = ':1-$nrestr&!@H=',
                    restraint_wt = 5.0,
                    ioutfm = 1, ntxo = 2,                                           ! Fomrat of coor. and vel. trj files, write NetCDF restrt fuile for final coor., vel., and box size
                    /
```
Note, the _iphmd_ flag is set to 1 for GBNeck2-CpHMD and thus we need to have an input file for CpHMD simulations. Here is an example:
```
				&phmdin
        QMass_PHMD = 10,                                ! Mass of the Lambda Particle
        Temp_PHMD = 300,                                ! Temp of the lambda particle
        phbeta = 5,                                     ! Friction Coefficient (1/ps) for titration integrator
        iphfrq = 1,                                     ! Frequency to update the lambda forces
        NPrint_PHMD = 1000,                             ! How often to print the lambda
        PrLam = .true.,                                 ! Should lambda be printed
        PrDeriv = .false.,                              ! Do you want to print the derivatives?
        PRNLEV = 7,                                     ! Sets what is printed out, (7) normal print level
        PHTest = 0,                                     ! Are lambda and theta fixed, (0) = not fixed, (1) = fixed
        MaskTitrRes(:) = 'AS2','GL2','HIP','CYS','LYS', ! Residues to include as titratable
        MaskTitrResTypes = 5,             							! number of titratable types
        QPHMDStart = .true.,                            ! Initialize velocities of titration varibles with a Boltzmann distribution
        /" > ${protein}_phmdin
```
And then,
```
pmemd.cuda -O -i equil.mdin -c ${protein}_mini.rst7 -p ${protein}.parm7 -ref ${protein}.rst7 -r ${protein}_equil.rst7 -o ${protein}_equil.out -x ${protein}_equil.nc -phmdin ${protein}_phmdin -phmdparm $inputparm -phmdout ${protein}_equil.lambda -phmdrestrt ${protein}_equil.phmdrst &>> equillogfile
```
To conduct a multiple-step equilibrations, we need to loosen the restraint strength from `restraint_wt = 5.0` to `restraint_wt = 2.0`, `restraint_wt = 1.0`, and `restraint_wt = 0.0` sequentially in the `equil.mdin` file. Besides changing the restaint force a few other settings need to be changed after the initial equilibration. `irest = 0, ntx = 1` should be changed to `irest = 1, ntx = 5` in the `equil.mdin` file and `QPHMDStart` should be set to `.false.` in the `${protein}_phmdin` file. In addition, we need to apply `sed -i 's/PHMDRST/PHMDSTRT/g' ${protein}_equil.phmdrst` to the initial/previous phmdrst file for input into the next equilibration.
Here is an example to restart an equilibration,
```
pmemd.cuda -O -i equil_new.mdin (or updated mdin) -c ${protein}_equil.rst7 -p ${protein}.parm7 -ref ${protein}.rst7 -r ${protein}_equil_new.rst7 -o ${protein}_equil_new.out -x ${protein}_equil_new.nc -phmdin ${protein}_phmdin -phmdparm $inputparm -phmdstrt ${protein}_equil.phmdrst -phmdout ${protein}_equil_new.lambda -phmdrestrt ${protein}_equil_new.phmdrst &>> equillogfilenew
```
__Step5. Production__
For a 10 ns production run at pH=7.0, we can use an example _mdin_ input file `${protein}_pH7.0.mdin` like:
```
                    &cntrl
                    imin = 0, nstlim = 5000000, dt = 0.002, 
                    irest = 1, ntx = 5, ig = -1, 
                    temp0 = 300, 
                    ntc = 2, ntf = 2, tol = 0.00001,
                    ntwx = 5000, ntwe = 0, ntwr = 500000, ntpr = 500000, 
                    cut = 999.0, iwrap = 0, igb = 8,
                    ntt = 3, gamma_ln = 1.0, ntb = 0,
                    iphmd = 1, solvph= 7.0, saltcon = 0.15,
                    ioutfm = 1, ntxo = 2,                        
                    /
```
And then,
```
pmemd.cuda -O -i ${protein}_pH7.0.mdin -c ${protein}_equil.rst7 -p ${protein}.parm7 -r ${protein}_pH7.0_prod1.rst7 -o ${protein}_pH7.0_prod1.out -x ${protein}_pH7.0_prod1.nc -phmdin ${protein}_phmdin -phmdparm $inputparm -phmdstrt ${protein}_equil.phmdrst -phmdout ${protein}_pH7.0_prod1.lambda -phmdrestrt ${protein}_pH7.0_prod1.phmdrst -inf pH7.0_mdinfo &>> prodlogfile &
```
For other pH conditions, we generate similar _mdin_ files and use similar commands as above. For Rex-Exchange CpHMD simulations, please follow Amber mannual to generate a groupfile and run properly on your cluster architecture.
__Step6. Restart__
If we want to extend a simulation, say, at pH=7.0, for another 5ns, we can simply change the above `${protein}_pH7.0.mdin` file by setting `nstlim = 2500000`. And then,
```
sed -i 's/PHMDRST/PHMDSTRT/g' ${protein}_prod1.phmdrst
pmemd.cuda -O -i ${protein}_pH7.0.mdin -c ${protein}_pH7.0_prod1.rst7 -p ${protein}.parm7 -r ${protein}_pH7.0_prod2.rst7 -o ${protein}_pH7.0_prod2.out -x ${protein}_pH7.0_prod2.nc -phmdin ${protein}_phmdin -phmdparm $inputparm -phmdstrt ${protein}_prod1.phmdrst -phmdout ${protein}_pH7.0_prod2.lambda -phmdrestrt ${protein}_pH7.0_prod2.phmdrst -inf pH7.0_mdinfo &>> restartlogfile &
```

For a quick start, we can use [cphmd_prep](https://gitlab.com/shenlab-amber-cphmd/cphmd-prep) tools to easily setup GBNeck2-CpHMD simulations on a workstation with Nvidia GPU(s). And after the CpHMD simulations are done, we can use [cphmd_analysis](https://gitlab.com/shenlab-amber-cphmd/cphmd-analysis) to perform p*K*a calculations and other analysis.