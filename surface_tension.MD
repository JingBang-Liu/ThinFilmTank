# 3d Lennard-Jones melt*******************************

# ****************************************************
# basic simulation properties
# ****************************************************
variable        temperature equal 1.1
units		lj
atom_style	atomic

# ****************************************************
# atomic properties
# ****************************************************
#read_restart 	restart.save.mpiio
read_data	atomdata

pair_style	lj/cut 5.5
pair_coeff	1 1 1.0 1.0

neighbor	0.5 bin
neigh_modify	delay 0 every 10 check no

mass		1 1.0

# ****************************************************
# initialise atoms
# ****************************************************
velocity	all create ${temperature} 87287 loop geom

# ****************************************************
# thermostat/ensemble
# ****************************************************
fix             1 all temp/berendsen ${temperature} ${temperature} 0.05
fix             2 all nve

# ****************************************************
# Output settings
# ****************************************************
compute 	myTemp all temp
compute 	myke all ke
compute		mype all pe
compute		myke_atom all ke/atom
compute 	mystr all stress/atom NULL
thermo_style 	custom step temp etotal ke pe
fix 		extra all print 6000 "$(step) $(temp) $(etotal) $(ke) $(pe)"& 
		append iteration_info.dat screen yes
fix		myave all ave/atom 1 10000 10000 x y z c_mystr[*]
dump		1 all custom 6000 dump.lj id type x y z vx vy vz c_mystr[1] c_mystr[2] c_mystr[3]
dump		2 all custom 10000 dump.ave id type f_myave[1] f_myave[2] f_myave[3] f_myave[4] &
		f_myave[5] f_myave[6]

# ****************************************************
# Run settings
# ****************************************************
timestep	0.001
run		600000
#run 		100 every 20 &
#		"print 'test' append iteration_info.dat screen no"

# ****************************************************
# restart
# ****************************************************
write_restart 	restart.save.mpiio
#write_restart 	restart.save
