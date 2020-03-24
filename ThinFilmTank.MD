############################################################
#####	 	    Argon in tank	   	   #########
############################################################
variable		temperature equal 85.0

units			real	
atom_style		atomic
dimension 		3	
boundary		p p p	

read_data		atomdata
#read_restart 		water.restart.mpiio

############################################################	
#############INTERACTION PARAMETERS########################
group			drop    type 1
group	   	 	wall    type 2

mass           		1 	18.05
mass                    2       195.084

neighbor        	2.0 bin
neigh_modify		every 1 delay 1 check yes

pair_style 		hybrid/overlay lj/cut 18.7 sw

#pair_coeff           	 * * mW.sw Si NULL
pair_coeff              * * sw mW.sw mW NULL
#pair_coeff              * *  lj/cut 6.189 2.3925 1.80
pair_coeff	    	2 2  lj/cut 15.999 2.47 7.0
pair_coeff		1 2  lj/cut 0.9 2.8140 7.0

fix	    		f1 wall setforce 0.0 0.0 0.0
thermo			5
min_style 		fire
minimize    		1.0e-5 1.0e-7 200 10000	
	

############################################################	
#############EQUILIBRATION PARAMETERS######################
velocity 		drop create ${temperature} 12345
compute 		myTemp drop temp
fix			1 drop nvt temp ${temperature} ${temperature} 100.0
fix_modify   		1 temp myTemp

fix	    		f2 wall setforce 0.0 0.0 0.0
velocity		wall set 0.0 0.0 0.0

###########################################################
############### Post parameters############################
thermo_style    	custom step temp pe etotal 
thermo_modify	        temp myTemp
thermo			100
fix            		extra drop print 2000 "$(step) $(temp) $(pe)" append iteration_info.dat &
			screen no

timestep		5.0
reset_timestep 		0
restart     		100000 water.restart.mpiio
dump			1 all custom 500 thinfilmtank.dat id type x y z
run			50000


