# First use of FloeDyn

A first help for use FloeDyn where different options are explained :
```
build/FLOE --help
```
We here explain in more detail the different possibilities.


## Run simulation from an existing input file

There exist, There exist, in “io” directory, input files ready to be used as initial conditions. The minimum requirement is to give a final time with the option '-t   <final time in second>'

***Example with one single big floe***
```
build/FLOE -i io/intputs/in_single_big_floe.h5 -t 1000
```
Here by default, the simulation is made with the mode  '--fmodes 1 1' which affect wind and currents from prescribed data (see paragraph below on TOPAZ data). We propose to begin with the mode '--fmodes 0 0' meaning no wind and no ocean currents. In this configuration, the floe will not move and previous command is equivalent to :
```
build/FLOE -i io/intputs/in_single_big_floe.h5 --fmodes 0 0 -t 1000
```
The output messages :
* **At the beginning**: It recalls the input file readed, the ice/water concentration in the considered window, and the mode for wind speed and water speed. 
```
Reading "io/inputs/in_single_big_floe.h5" ... 
1 Floes, concentration : 1%
NO Atmospheric and Ocean currents!
```
* **At the end** of the outputs message gives the total number of time step in the simulation, the output file containing all the information of the simulation (see following section for visualization and extract data) and the total time of the simulation (here it is 0 second because simulation is very simple and so very fast !)
```
----
NB STEPS : 100
OUT FILE : io/outputs/out_3QaBv.h5
Chrono : Total simulation: 0h 0m 0s
```
* **Between them, after SOLVE , for each time step**: it indicates the state of the time, the used time step for solving contact and state, the total kinetic energy. Chrono give the detail of the time needed to solve the different step (collision, calculate time step and move the floe)
```
----
Time : 40 | delta_t : 10 | Kinetic energy : 0
Chrono : collisions 0.00605 ms + time_step 0.000102 ms + move 0.03549 ms = 0.041642 ms
----
```
   
Here the simulations contain only one floe and no wind or oceanic current are considered so nothing special happened during the simulation. For more complex simulations, some collisions can appear then output messages inform on the number of solved LCP for a given time where there is a contact problem to solve. The algorithm adapts the time step used to compute floes state (called delta_t) . The number of solved LCP given at each time step corresponds to the number of resolved contact problems during this time step.

***Example with additional option***
An example with another mode which is explain in the paragraph on vortex. Try
```
build/FLOE -i io/inputs/in_200f_60p_ZcgXf.h5 --fmodes 5 0 -t 10000
```
We have more information given at each time step , the norm of vortex speed and when the number of solved LCP at the time step. 
```
----
Time : 9990.47 | delta_t : 1.86098 | Kinetic energy : 31509.4
#LCP solve: 3 / 3
the vortex wind speed is: 1.32236
Chrono : collisions 0.423399 ms + time_step 0.471882 ms + move 3.94897 ms = 4.84425 ms
----
```
Total LCP solved is recall at the end of the simulation in the output message :
```
----
Time : 10001.6 | delta_t : 1.86311 | Kinetic energy : 31453.9
NB STEPS : 15457
OUT FILE : io/outputs/out_cuh7K.h5
#TOTAL LCP solve: 1798/1798(100%) 
LCP_failed compression phase: 0, LCP_failed decompression phase: 0, LCP_solved with solution maintaining the kinetic energy: 1
Chrono : Total simulation: 0h 1m 3s
```

## The different modes

To give external forces (i.e.  wind and ocean current) to FloeDyn’s simulation, you have to choose a predetermined mode for the atmosphere and the ocean, mode are given by the option '--fmodes <atmosphere mode> <ocean mode>'.At this time, there exist three different mode for simulations from inputs file:
* Wind and water speeds are determined by real observations given in the file TOPAZ file. Mode 1 can only be use in these configuration (you can’t mix  `--fmodes 1 0` or `--fmodes 0 1` )
```
build/FLOE -i io/intputs/in_single_big_floe.h5 --fmodes 1 1 -t 1000
```
* Create convergent wind and oceanic current to a target zone depending of the ice concentration disired with the mode `--fmode 2 0` or  `--fmode 0 2`. See more details in section \ref{sec-generator}.
* Mode used to throw floe on obstacle or in a corridor . Mode '--fmodes 4 0' (or '--fmodes 0 4') apply first converged wind (resp. ocean current) to point (0,0) then after 500 second, appy a wind (resp. ocean current) which goes in a unique direction. The point of convergence, the time convergence and the direction of the wind can be modified in *physical_data.hpp* method *x_convergent_then_constant*. The wind’s speed and ocean current’s speed after the convergence phase can be given by the command line `--fspeed <wind’s speed> <ocean’s speed>` . For example
```
build/FLOE -i io/intputs/in_single_big_floe.h5 --fmodes 4 0 --fspeed 20 0 -t 1000
```
    Note that if if mode 0 is given (in our example for ocean current) then give a speed will have no effect (for example `--fmodes 4 0 --fspeed 20 20` will have no effect on ocean currents and `--fmodes 4 0 --fspeed 0 20` have no sense but `--fmodes 4 0 --fspeed 20 0` have.)
* The vortex mode : there actually exist two vortex modes, both give wind’s currents and 0 mode for the ocean. A simple one that is a single vortex with mode `--fmodes 5 0` and a mode with different vortex `--fmodes 6 0`. These modes do not need to specify any speed. There is a special section on vortex mode below.
```
build/FLOE -i io/intputs/in_single_big_floe.h5 --fmodes 5 0 -t 1000
```
Because wind speed is given by code in the file \TODO \textit{check this}, add an option `--fspeed` will not be taken into account. 


## Visualize the output
FloeDyn creates an output file in the io/outputs folder that contains the different state of the simulation at each *output time step*, that is 60 second by default. These time steps can be modified using the option `-o <output time step>` . Details on these output files are explained later in paragraph .

FlowDyn also allow to create images and videos of the simulations :
* Create image at a given time step :
```
python3 plot img -f io/outputs/myoutput.h5
```
* Create video of the simulation :
```
python3 plot anim -f io/outputs/myoutput.h5
```
	or
```
python3 plot mkvid -f io/outputs/myoutput.h5
```
In these cases, floes are colored, the color corresponds to the kinetic energy of the floe (red is high kinetic energy).


***plotting options***
For image or video, following options can be added :
* `--hd` create hd image/video
* `-w` draw the window where floe are concentrated
* `--follow` centered the image/video on the center of the floe pack.
* `--nocolor` hide floe’s color
* `--step=10` for video, accelerate the video by taking 1 image every 10 time step.
* `--ghosts` draw also the ghosts cells, option for the PBC simulation.


## Other options for simulation
In the help, all options are detailed, some options can be used only by the generator (see section \ref{sec-generator}). Here we detail the different option that can be added for simulations:
* **bustle option:** To improve the behavior of each floe, it is possible to add a random flow velocity at each floe with option `--bustle 1`  These options produce more contact between flow. The option `--nbustle <norm of the random speed>` give the max norm of these random speed, by default it is in the order of magnitude of 1e-7 m/sec
* **obl option:** coupling between ocean and floe speed. See paragraph OBL
* **obstacles** Give an infitine mass for given floe. For exemple `--obstacle 0 5 10` will transform floe number 0 5 and 10 as obstacle.
* **simulation from an outfile.h5:** to resume a previously stopped simulation at the breaktime 
```
build/FLOE -i io/inputs/inputfile -r breaktime -f io/outputs/outputfile.h5 -t <finaltime>
```

## Two other executable to run simulations
Simulations can be runned using two other strategy :
* **with periodic boundary conditions:** This strategy considers 9 identical cells arranged in 3 by 3 grid. The central cells is the referent cell and the other are ghosts cells, and physical data are applied on the 9 cells. With this approach, the ice concentration is maintained during all the simulation and in each cell.
Example : 
```
build/FLOE_PBC -i io/inputs/floes_config.h5 -t <final time>
```
All options from the classical version of FloeDyn are available to the PBC version.
When plotting the result from a PBC simulation, you can add option `--ghosts` to visualize ghosts cells.
```
python3 plot img -f io/outputs/out_files.h5 --ghosts
```
* **parallel version:** There exists version of FloeDyn using MPI. To run simulation write :
parallel version
mpirun -np 10 ./build/FLOE_MPI -i io/inputs/floes_config.h5 -t <final time>
parallel version
Where `-np` is the number of processor. From the given number of processor, FloeDyn calculates how many zones it will cut the window. Here with 10 processor, FloeDyn will cut the window in 4 zones (1 processor by zone, plus 1 prossesor for the 5 zones between each zone and 1 "global" processor).


