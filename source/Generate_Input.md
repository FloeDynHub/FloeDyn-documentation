# Generate inputs files with generator

It is possible to generate an input file by giving the numbers of floes and the ice concentration desired. The generator generates floes randomly taken in a floe library of a hundred different shapes. The generator first arranges floes in spiral and made them converge in a target zone depending of the total size of floes and the concentration. When the desired concentration are reached, the final windows size is created and is fixed by the generator.  The generator must be used with and only with the mode 2 with the different option `--fmodes 2 0` or `--fmodes 0 2` or `--fmodes 2 2`.

## Generate a simple input

For example, to create an input with 1000 floes, with 60\% of ice concentration you can use command
```
build/FLOE -i generator -n 100 -c 0.6 --fmodes 2 0 --fspeeds 20 0 -t 0 --bustle 1
```
Note that you have to specify a wind or current speed because the default value is 0. As in the simulation, be careful to match mode and speed options. 
The option `-t 0` ensures simulation ends after the desired concentration is reached. If the end time option is for example `-t 1000`, the simulation will continue with mode 2 until the final time 1000s. It is good to use the bustle option with the generator, it helps to arrange floes in the target zone.

The output message for the generator indicates :
* **At the beginning** : the number of time step needed to create the desired configuration and the exact ice concentration and the exact window’s dimension, and other differents information specific to mode 2
```
Generate 100 floes...
the restitution coefficient is fixed to: 0
The exponent of the power law is: 1.5 and the floe number per size is: 1
min diameter: 11.604
max diameter: 250
The size ratio is: 21.5443
the ice/ice static friction coefficient is fixed to: 0
Floe diameter : max = 631.125, min = 27.5118
WIDTH WINDOW : 708.593
Convergent current towards the target area for initial ice pack generation
The Atmospheric current speed is fixed to: 20m/s   |   The Ocean current speed is fixed to: 0m/s
```
Warning : here by defaul max diameter is fixed to 250 m but it generate floe with max diameter 645 ???
* **At the end** : the generator creates an input file in the io/inputs folder, the name of the input contains the number of floe and the concentration plus 5 random characters. In the following example the input created is *in_200f_60p_4qHl5.h5*. It also creates an output file that contains all floes and simulation information during the convergence phase. In our exemple it is *out_i0bdG.h5*, as with all output files it is possible to create video or extract data. It also informs of the total time needed to reach the target area, the real concentration reached and the percentage of solved LCP.
```
----
Time : 4020.86 | delta_t : 3.5824 | Kinetic energy : 614.617
NB STEPS : 12503
It is time to stop floes within the target area.
Concentration : 0.609767
io/inputs/in_100f_60p_1OqHS.h5 written
OUT FILE : io/outputs/out_AmATk.h5
#TOTAL LCP solve: 20950/20950(100%) 
LCP_failed compression phase: 0, LCP_failed decompression phase: 0, LCP_solved with solution maintaining the kinetic energy: 0
Convergent current towards the target area for initial ice pack generation 
The Atmospheric current speed is fixed to: 20m/s | The Ocean current speed is fixed to: 0m/s
initial windows is: -346.101, 346.101, -346.101, 346.101
current windows is: -343.767, 340.847, -343.759, 344.906
the initial concentration is: 0.6
the current concentration is: 0.609767
The desired concentration is: 0.6
desired windows is: -346.101, 346.101, -346.101, 346.101
```
* **After creating input file** : output message informs on the simulation from time 0 (when the input file is created) until the end of the simulation (in this example the final time is 0 seconds, but with a final time at 1000 second, the simulation would have continued during 1000) with mode 2. Again, an output file containing all datas is created, in our exemple it is *out_L3hFq.h5*
```
Time : 0 | delta_t : 1 | Kinetic energy : 0
NB STEPS : 0
OUT FILE : io/outputs/out_4QEyp.h5
Chrono : Total simulation: 0h 0m 24s
```

## Adapte the stoping time to generate input file
The strategy adopted in this code to generate inpute file was describe quickly at the begining of the section : 
* floes are generated and dispose in spiral
* the mode 2 apply convergent wind/current to the center
* window size is calculated with the desired concentration
* generation stop when the desired concentration is reached and all floes are in the window

In practice, the code stop only when kinetric energy is zero ( in *generator.hpp* method *generate_floe_set*, around line 114 : `while (m_problem.get_floe_group().kinetic_energy() != 0 && end_time < 1e6 )  m_problem.get_floe_group().reset_impulses();` )
Also, a arbitrary time is fixed to stop all floes which are inside the window, this stoping time is update periodically. ( in *generator.hpp* method *generate_floe_set*, around line 100 : `time_to_stop_floe_in_target_window` is by default 1000s and then uptade every 500s).

With this strategy, depending on the number of floe generated and the wind/current speed, generator could have difficulty to stop : 
* if wind/current are too small compare to the number of generated floes and stoping time too small : it can create a "traffic jam" at the entrance of the window and the code will not be able to stop. 
* is periodic time to stop floe inside window are too large, floe can get out of the window.
For example, with default stoping time, the phenomenon of "traffic jam" is observe for wind/current speed aroud 20m/s and more than 500 floe. Solutions are :
* increase the first stoping time : this also increase the time to generate our input file.
* increase the wind/current speed : if wind/current speed are too small you have an empty and heterogene organization in your window ( big floes in the midle, small floes in the border and empty between them). At the contrary too big wind/current speed will created concentrate and homogene organization in your window ( mixed organization of big and small floes concentred in the midle of the window).
To conclude, the input file generation needs to adapt the different parameters, first of all to have a not too long generation time and also to obtain the desired structure in the window.

## Information on input file

The executable file **FLOE_INPUT_INFO** allows you to obtain a quantitative description of floe configuration in an input file. You have to build it first before to use it on a input file :
```
python3 ./waf --target FLOE_INPUT_INFO -v
build/FLOE_INPUT_INFO io/inputs/h4To7.h5
```
```
Reading "io/inputs/in_100f_60p_1OqHS.h5" ... 
100 Floes, concentration : 60%
Domain (WxH): 692.203 x 692.203
initial windows is: -346.101, 346.101, -346.101, 346.101
current windows is: -343.767, 340.847, -343.759, 344.906
the current concentration is: 0.609767
Kinetic energy before setting at rest: 0
Floe diameter : max = 613.929, min = 27.0417
MPI max grid dim : 1x1 -> average 100 floes per core
```
**Note** : MPI max grid dim is an advice on how many core can by used for the MPI version. These advice is based on floe size : a floe should not be "broken" by MPI grid. In these exemple, wa have by default one big floe, then maximal number of grid for MPI version is $1$ (non parralele). To avoid this, you can change the number of floe per size (see \ref{subsec-generator_options}).

## Resize an input file

The executable **FLOE_RESIZE_INPUT** allows you to rescale an initial floe configuration. By changing the window’s dimension, ice concentration does not change but the ice floes are resized to fit with the new target zone. 
```
python3 ./waf --target FLOE_RESIZE_INPUT -v
build/FLOE_RESIZE_INPUT io/inputs/in_100f_60p_1OqHS.h5 1200
Reading "io/inputs/in_100f_60p_1OqHS.h5" ... 
100 Floes, concentration : 60%
io/inputs/in_100f_60p_K3CPi.h5 written
```
```
build/FLOE_INPUT_INFO io/inputs/in_100f_60p_K3CPi.h5 
Reading "io/inputs/in_100f_60p_K3CPi.h5" ... 
100 Floes, concentration : 60%
Domain (WxH): 1200 x 1200
initial windows is: -600, 600, -600, 600
current windows is: -595.954, 590.892, -595.94, 597.928
the current concentration is: 0.609767
Kinetic energy before setting at rest: 0
Floe diameter : max = 1064.3, min = 46.8795
MPI max grid dim : 1x1 -> average 100 floes per core
```

## Additional generator’s options

Other options useful to precisely create input file (to be completed):
* option for the distribution law :
	+  `--nbfpersize` sets the number of floe for each considered size. By default it is 1 (one big floe then one floe foe each size). For exemple with `--nbfpersize 4` will create 4 big floes which will be smaller than the big floe with default value. 
    +  `-a`  fractal dimension for the distribution power law.
* modify the collision restitution : can be modify using `e <arg>`. 
* modify  ice/ice static friction `-mu <arg>`. 
* modify physical properties of floes : modify the random variation of the ice floe thickness `-sigma <random variation of ice floe thickness>`, and the maximum size of floes `-m <floe_max_size>`



