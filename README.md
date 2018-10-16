# ZrH-ZrH2
evaluation of the diffusion  coefficient of ZrH
Responding to some of questions I got about how to use COMB3 potential for calculating the ZrH mean square displacement to be used for the diffusion coefficient calculation. I would put it here at once for all of you, below is an full input script you just probably copy it and run after you download the potential model from the above link I previously mentioned, it is up to you how to modify and refine for your HCP structure or other aspects:
####################################################
units metal
boundary p p p
atom_style charge
pair_style comb3 polar_off #for comb3 potential
variable X equal "16"
variable Y equal "10"
variable Z equal "10"
variable T equal "500"
lattice hcp 3.232
region box block 0 $X 0 $Y 0 $Z
create_box 2 box
create_atoms 1 box
create_atoms 2 random 100 878567 box
group zr type 1
group hydrogen type 2
pair_coeff * * ffield.comb3 Zr H #H
neighbor 2 bin
neigh_modify delay 10 check yes
mass 1 91.224
mass 2 1.0079
thermo 500
# Energy minimization and pressure minimization
fix 1 all nve/limit 0.1
thermo_style custom step pe lx ly lz press pxx pyy pzz vol etotal
minimize 1.00e-30 1.00e-30 100000 1000000
unfix 1
fix 1 all box/relax aniso 0.0 vmax 0.1
thermo_style custom step pe lx ly lz press pxx pyy pzz vol etotal
minimize 1.0e-30 1.0e-30 100000 1000000
unfix 1
write_data data_after_E_P_Min_step_1
# Equilibration at desired temperature
timestep 0.0002
velocity all create $T 32145 mom yes rot no
fix 1 all npt temp $T $T 1 iso 0 0 1 drag 1
# Set thermo output
thermo 500
thermo_style custom step lx ly lz press pxx pyy pzz pe temp
run 300000
write_data data_after_T_Min_step_2
unfix 1
reset_timestep 0
timestep 0.0002
fix 1 all nvt temp $T $T 100.0
thermo 100#00
compute 5 hydrogen msd com yes
compute 6 zr msd
thermo_style custom step temp pe press vol etotal
fix 3 hydrogen ave/time 10 5 1000 c_5[1] c_5[2] c_5[3] c_5[4] c_6[4] file $T_MSD_zr_h
run 10000000
unfix 1
unfix 3
write_data data_after_MSD_step_3
