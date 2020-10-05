## README B0AT1 Simulation Set-up. 

#Gromacs/2018.3 was used throughout the development of this system 
#posres .itp files should be made prior to running this script. 

#conversion of protein to coarse-grain (CG)
1.	 6M1D structure of B0AT1 was downloaded as pdb and altered to show only chain A. Chain A is B0AT1 protein and will remove ACE2 and the SARS-nCoV2 S-glycoprotein. 
2.	 This requires the martinize.py script 
3.	 This will procide a topology (.top) file for B0AT1. 

```
./martinize.py –f 6M1D_B0AT1_MONOMER.pdb –o 6M1D_B0AT1_MONOMER.top –n B0AT1-CG.ndx –nmap 6M1D_B0AT1_MONOMER_mapping.ndx –dssp mkdssp –name 6M1D_B0AT1_MONOMER-CG –ff martini22 –p backbone –elastic –ef 500 –el 0.5 –eu 0.9 –ea 0 –ep 0
```
This was visualised in VMD to make sure there was no issues. 

#protein embedding
```
./python2 insane+SF.py -u PBSM:0.007 -u PXSM:0.004 -u DPSM:0.017 -u PNSM:0.008 -u DPMC:0.007 -u POMC:0.043 -u DOMC:0.009 -u PIMC:0.009 -u OIMC:0.004 -u OEMC:0.005 -u PAMC:0.012 -u PUMC:0.029 -u POME:0.001 -u DOME:0.006 -u OIME:0.001 -u OQME:0.002 -u OAME:0.014 -u OUME:0.006 -u IAME:0.007 -u IQME:0.001 -u LPPC:0.008 -u DPPC:0.003 -u POPC:0.147 -u DOPC:0.011 -u PIPC:0.088 -u OIPC:0.05 -u PAPC:0.052 -u PUPC:0.05 -u POPE:0.007 -u OPPE:0.002 -u DOPE:0.007 -u PAPE:0.006 -u PUPE:0.002 -u OIPE:0.002 -u PQPE:0.002 -u OQPE:0.001 -u OAPE:0.002 -u OUPE:0.001 -u POPG:0.004 -u DOPG:0.003 -u DODG:0.004 -u PODG:0.007 -u CHOL:0.35 
-l OGPS:0.004 -l DPPS:0.001 -l POPS:0.094 -l DOPS:0.006 -l PAPI:0.028 -l PAP1:0.014 -l PAP2:0.014 -l PQPI:0.019 -l PQP1:0.01 -l PQP2:0.01 -l POPI:0.013 -l POP1:0.006 -l POP2:0.006 -l PUPI:0.008 -l DOPI:0.008 -l PIPI:0.006 -l PEPI:0.004 -l PBSM:0.003 -l PXSM:0.002 -l DPSM:0.007 -l PNSM:0.003 -l DPMC:0.003 -l POMC:0.016 -l DOMC:0.003 -l PIMC:0.003 -l OIMC:0.002 -l OEMC:0.002 -l PAMC:0.004 -l PUMC:0.011 -l POME:0.005 -l DOME:0.02 -l OIME:0.003 -l OQME:0.006 -l OAME:0.052 -l OUME:0.023 -l IAME:0.024 -l IQME:0.003 -l LPPC:0.003 -l DPPC:0.001 -l POPC:0.056 -l DOPC:0.004 -l PIPC:0.034 -l OIPC:0.019 -l PAPC:0.02 -l PUPC:0.019 -l POPA:0.01 -l DOPA:0.005 -l POPE:0.025 -l OPPE:0.008 -l DOPE:0.026 -l PAPE:0.02 -l PUPE:0.007 -l OIPE:0.008 -l PQPE:0.006 -l OQPE:0.004 -l OAPE:0.009 -l OUPE:0.005 -l POPG:0.003 -l DOPG:0.002 -l DODG:0.002 -l PODG:0.004 -l CHOL:0.253 -x 31 -y 31 -z 16 -f  6M1D_B0AT1_MONOMER-CG.pdb -center -o  6M1D_B0AT1_MONOMER-CG.gro -d 0 -p  6M1D_B0AT1_MONOMER-CG.top -sol PW -salt 0.15
```

#add .itp files to .top
```
sed -i '8 a #include "martini.ff/6M1D_B0AT1_MONOMER-CG_A.itp"' 6M1D_B0AT1_MONOMER-CG.top
sed -i 's/Protein/6M1D_B0AT1_MONOMER-CG_A/g' 6M1D_B0AT1_MONOMER-CG.top
```

#minimisation
```
gmx grompp -f 1-EM.mdp -c 6M1D_B0AT1_MONOMER-CG.gro -p 6M1D_B0AT1_MONOMER-CG.top -o EM_6M1D_B0AT1_MONOMER-CG.tpr
gmx mdrun -v -deffnm EM_6M1D_B0AT1_MONOMER-CG
```
```
gmx grompp -f 1-EM.mdp -c EM_6M1D_B0AT1_MONOMER-CG.gro -p 6M1D_B0AT1_MONOMER-CG.top -o EM2_6M1D_B0AT1_MONOMER-CG.tpr
gmx mdrun -v -deffnm EM2_6M1D_B0AT1_MONOMER-CG
```
Energy minimisation requires define = -DFLEXIBLE which removes contraints on water molecules and allowed for flexible bonds between them. This allows for a much better energy minimisation and removes many LINCS errors which would otherwise occur. Flexible water molecules should not be used in the equilibrations or later simulations. 

```
gmx grompp -f emin.mdp -c EM2_6M1D_B0AT1_MONOMER-CG.gro -p 6M1D_B0AT1_MONOMER-CG.top -o EM3_6M1D_B0AT1_MONOMER-CG.tpr
gmx mdrun -v -deffnm EM3_6M1D_B0AT1_MONOMER-CG
```

#Neutralise and Ionise with [0.15] NaCl
```
gmx grompp -f emin.mdp -c EM3_6M1D_B0AT1_MONOMER-CG.gro -p 6M1D_B0AT1_MONOMER-CG.top -o GENION_6M1D_B0AT1_MONOMER-CG.tpr
(sleep 5; echo PW) | gmx genion -s GENION_6M1D_B0AT1_MONOMER-CG.tpr -p 6M1D_B0AT1_MONOMER-CG.top -conc 0.15 -neutral -o ION_6M1D_B0AT1_MONOMER-CG.gro
```

#alligning .gro and .top files with the martini representation of Na and Cl.
```
sed -i 's?NA      NA?ION    NA+?' ION_6M1D_B0AT1_MONOMER-CG.gro
sed -i 's?CL      CL?ION    CL-?' ION_6M1D_B0AT1_MONOMER-CG.gro
sed -i 's/CL/CL-/g' 6M1D_B0AT1_MONOMER-CG.top
sed -i 's/NA/NA+/g' 6M1D_B0AT1_MONOMER-CG.top
sed -i 's/NA++/NA+/g' 6M1D_B0AT1_MONOMER-CG.top
sed -i 's/CL--/CL-/g' 6M1D_B0AT1_MONOMER-CG.top
```
#minimise the system
``` 
gmx grompp -f emin.mdp -c ION_6M1D_B0AT1_MONOMER-CG.gro -p 6M1D_B0AT1_MONOMER-CG.top -o ION_6M1D_B0AT1_MONOMER-CG.tpr
gmx mdrun -v -deffnm ION_6M1D_B0AT1_MONOMER-CG
```

#Duplicates of .itp for different posres have already been made. This now duplicates the .top file to be representative of the .itp files. 
```
cp 6M1D_B0AT1_MONOMER-CG.top 6M1D_B0AT1_MONOMER-CG_1000.top
sed -i '8 a #include "martini.ff/6M1D_B0AT1_MONOMER-CG_A_1000.itp"' 6M1D_B0AT1_MONOMER-CG_1000.top
sed -i '9d' 6M1D_B0AT1_MONOMER-CG_1000.top
```
```
cp 6M1D_B0AT1_MONOMER-CG.top 6M1D_B0AT1_MONOMER-CG_100.top
sed -i '8 a #include "martini.ff/6M1D_B0AT1_MONOMER-CG_A_100.itp"' 6M1D_B0AT1_MONOMER-CG_100.top
sed -i '9d' 6M1D_B0AT1_MONOMER-CG_100.top
```
```
cp 6M1D_B0AT1_MONOMER-CG.top 6M1D_B0AT1_MONOMER-CG_10.top
sed -i '8 a #include "martini.ff/6M1D_B0AT1_MONOMER-CG_A_10.itp"' 6M1D_B0AT1_MONOMER-CG_10.top
sed -i '9d' 6M1D_B0AT1_MONOMER-CG_10.top
```
```
cp 6M1D_B0AT1_MONOMER-CG.top 6M1D_B0AT1_MONOMER-CG_50.top
sed -i '8 a #include "martini.ff/6M1D_B0AT1_MONOMER-CG_A_50.itp"' 6M1D_B0AT1_MONOMER-CG_50.top
sed -i '9d' 6M1D_B0AT1_MONOMER-CG_50.top
```
```
cp 6M1D_B0AT1_MONOMER-CG.top 6M1D_B0AT1_MONOMER-CG_500.top
sed -i '8 a #include "martini.ff/6M1D_B0AT1_MONOMER-CG_A_500.itp"' 6M1D_B0AT1_MONOMER-CG_500.top
sed -i '9d' 6M1D_B0AT1_MONOMER-CG_500.top
```
#make index file.
Once this is done for all Posres values, we can begin to group our molecules. In the .mdp file it is important to group lipids, protein, and the water and ions into separate groups.  Open the .mdp file to view the groups which have been made. If protein is not there, add it as another group. 
 
```
echo "13|14|15|16|17|18|19|20|21|22|23|24|25|26|27|28|29|30|31|32|33|34|35|36|37|38|39|40|41|42|43|44|45|46|47|48|49|50|51|52|53|54|55|56|57|58|59|60|61|62|63|64|65|66|67|68|69|70|71|72|73
name 76 lipids 
74|75 
name 77  Water_and_ions
q" | gmx make_ndx -f ION_6M1D_B0AT1_MONOMER-CG.gro -o 6M1D_MONOMER.ndx
```
#posres  equilibration
```
gmx grompp -f equilibrate_10fs.mdp -c ION_6M1D_B0AT1_MONOMER-CG.gro -r ION_6M1D_B0AT1_MONOMER-CG.gro -n 6M1D_MONOMER.ndx -p 6M1D_B0AT1_MONOMER-CG_1000.top -o EQ10_1000_6M1D_B0AT1_MONOMER-CG.tpr
gmx mdrun -v -deffnm EQ10_1000_6M1D_B0AT1_MONOMER-CG
```
```
gmx grompp -f equilibrate_10fs.mdp -c EQ10_1000_6M1D_B0AT1_MONOMER-CG.gro -r EQ10_1000_6M1D_B0AT1_MONOMER-CG.gro -n 6M1D_MONOMER.ndx -p 6M1D_B0AT1_MONOMER-CG_500.top -o EQ10_500_6M1D_B0AT1_MONOMER-CG.tpr
gmx mdrun -v -deffnm EQ10_500_6M1D_B0AT1_MONOMER-CG
```
```
gmx grompp -f equilibrate_10fs.mdp -c EQ10_500_6M1D_B0AT1_MONOMER-CG.gro -r EQ10_500_6M1D_B0AT1_MONOMER-CG.gro -n 6M1D_MONOMER.ndx -p 6M1D_B0AT1_MONOMER-CG_100.top -o EQ10_100_6M1D_B0AT1_MONOMER-CG.tpr
gmx mdrun -v -deffnm EQ10_100_6M1D_B0AT1_MONOMER-CG
```
```
gmx grompp -f equilibrate_10fs.mdp -c EQ10_100_6M1D_B0AT1_MONOMER-CG.gro -r EQ10_100_6M1D_B0AT1_MONOMER-CG.gro -n 6M1D_MONOMER.ndx -p 6M1D_B0AT1_MONOMER-CG_50.top -o EQ10_50_6M1D_B0AT1_MONOMER-CG.tpr
gmx mdrun -v -deffnm EQ10_50_6M1D_B0AT1_MONOMER-CG
```
```
gmx grompp -f equilibrate_10fs.mdp -c EQ10_50_6M1D_B0AT1_MONOMER-CG.gro -r EQ10_50_6M1D_B0AT1_MONOMER-CG.gro -n 6M1D_MONOMER.ndx -p 6M1D_B0AT1_MONOMER-CG_10.top -o EQ10_10_6M1D_B0AT1_MONOMER-CG.tpr
gmx mdrun -v -deffnm EQ10_10_6M1D_B0AT1_MONOMER-CG
```

#timestep equilibration
```
gmx grompp -f equilibrate_20fs.mdp -c EQ10_10_6M1D_B0AT1_MONOMER-CG.gro -r EQ10_10_6M1D_B0AT1_MONOMER-CG.gro -n 6M1D_MONOMER.ndx -p 6M1D_B0AT1_MONOMER-CG.top -o EQ20_6M1D_B0AT1_MONOMER-CG.tpr
gmx mdrun -v -deffnm EQ20_6M1D_B0AT1_MONOMER-CG
```
```
gmx grompp -f equilibrate_25fs.mdp -c EQ20_6M1D_B0AT1_MONOMER-CG.gro -r EQ20_6M1D_B0AT1_MONOMER-CG.gro -n 6M1D_MONOMER.ndx -p 6M1D_B0AT1_MONOMER-CG.top -o EQ25_6M1D_B0AT1_MONOMER-CG.tpr
gmx mdrun -v -deffnm EQ25_6M1D_B0AT1_MONOMER-CG
```
#Make a new dirctory for running the simulation
```
mkdir production
```

Within this directory should contain the martini forcefields,EQ25_6M1D_B0AT1_MONOMER-CG.gro,  molecular dynamics parameters file (.mdp), bash script for running simulation jobs. This was provided by Dr Katie Wilson (O'Mara group, RSC) and can also be found in the supplementary files. 

All files should be renamed the same such that it becomes the JOBNAME. For this system the job name is 6M1D_B0AT1_PR. EQ25_6M1D_B0AT1_MONOMER-CG.gro should be renamed 6M1D_B0AT1_PR_start.gro.

Using the qsub command for batch client script submitting, we can schedule jobs to be performed. In which case, is the simulation of B0AT1 in the epithelial membrane. 
```
qsub 6M1D_B0AT1_PR_start.gro
```

Status of submission can be viewed using;
```
qstat
```


