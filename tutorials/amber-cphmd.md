---
title: Amber CPHMD
description: 
published: true
date: 2020-11-13T16:31:37.293Z
tags: 
editor: markdown
dateCreated: 2020-08-25T16:27:37.300Z
---

# Amber (all-atom) CpHMD
## PME CpHMD (In progress)

Step 1:  use generate.inp to create protein, generate_wat.inp to create waterbox, add_ions.inp to add titratable water and counterions, and finally assembly.inp to create combined psf file.
>Note:  All of these files will need modification to run on a different system.  
Be especially careful with add_ions.inp.  In the end, you should have a number of TIPU = number of ASP + number of GLU and number of TIPP = number of HSP.  
For adding counterions, you will need to determine by hand what charge the system will have at pH 7 and put the appropriate numbers of ions as nneg and npos in add_ions.inp

Step 2: Use chamber in parmed to create amber parm7 and rst7 files.  Check the parmed.log file for commands.
>Note: you will need the modified file toppar_phmd_c22_foramber.str, find it here: skylab /home/rharris/toppar
Also, make sure to check waterbox size, which will be in *waterbox.prm

Step 3:  Next, do minimization. using min.scr, then heating of the system using haeting.scr.
>Note: the link_res_list variable in phmdin. This says which titratable waters are linked to which titratable residues. Make sure that you link HSP with TIPP and ASP/GLU with TIPU. A good check that you have done everything up to this point correctly is that Amber should report the charge of the system in heating.mdout to be ~0.

Step 4: Perform equilibraton with equil.sh. 

Step 5: Finally, you can run the replica exchange.  Check the docstrings in the python file. 
>Note: set AMBERHOME to point to your amber, and CUDA_VISIBLE_DEVICES to point to your GPUs.  Ie., if you want to use GPUS 0 and 3, do *export CUDA_VISIBLE_DEVICES="0,3"*

**Some other important TIPS:**
1.  Make sure the box is CUBIC not OCTA, as chamber cannot take octahedral boxes.
2. Use correct and updated version of executable, toppar and parm file.

Latest executable (July/2020): software/gitlab_amber
Latest toppar (July/2020): skylab:/home/rharris/toppar 
Latest parm (July/2020): skylab:/home/rotation/charrm_pme.parm
(Anyone who change or update these files please make relevant changes here too)

3. For handling charges: As for a general procedure: We usually want the system to have a net charge of zero at pH=7 (or whatever critical pH you are interested in), and we add sufficient ions to achieve that. For each Asp and Glu, if they are linked to a titrating TIPU water, the charge of the Asp/Glu + TIPU will be -1. For His + TIPP, the system will be neutral.
4. For a ligand, if it is titrating, you will need to link it to either a TIPU (if charge is neutral->negative when deprotonating) or TIPP (if charge is positive->neutral). Then the charge of the ligand + titrating water should behave as for the corresponding residue. If your ligand is not titrating, just count its fixed charge when determining how many ions to add.
