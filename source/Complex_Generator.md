# Generate more complex floes assembly

To generate more complex floe assembly, different executable file can be used :
* **FLOE_EXTRACT_INPUT** : create an input configuration from the output of a previous scenario (can select time)
* **FLOE_MAKE_NXP_INPUT** : create an initial floe configuration using multiple copies of the configuration in an input file
* **FLOE_MAKE_X4_INPUT** : create an initial floe configuration using 4 copies of the configuration in an input file
* **FLOE_PATCHWORK_INPUT** : create an initial floe configuration using configurations multiple different input file (i.e., stitching them together)
* **FLOE_SET_AT_REST** : zero out the kinetic energy a floe configuration
We describe some possibilities of use, already explained in doxygen file page  4 (but probably it is an non updated version).

*** Example to create huge floe assembly***

The generator is able to handle less than 5 000 floes. A good optimum is about 2 000 floes. Then to create a huge floe assembly, one should use the following processus:
* Knowing the total area of the floe assembly, split it in N*P cells (equal or not) containing about at most 2000 floes. For example, to create a 8000 assembly we need 4 cells of 2000 floes. Generate each cells using the generator :
```
build/FLOE -i generator -n 2000 --fmodes 2 0 --fspeeds 10 0 -c 0.6 -t 0 --bustle 1
```
Running 4 times that create 4 different window with 2000 floes. In our example input files created are *in_2000f_06p_1.h5*, *in_2000f_06p_2.h5*, *in_2000f_06p_3.h5*, *in_2000f_06p_4.h5*
* To obtain a 8000 floes assembly, one can use **FLOE_MAKE_X4_INPUT**, this loads four times the same input file (i.e.: same floe assembly repeated to create a 2X2 square)
```
build/FLOE_MAKE_X4_INPUT io/inputs/in_2000f_06p_1.h5
```
* Or use **FLOE_MAKE_NXP_INPUT**, this loads N * P times the same input file (i.e.: same floe assembly N*P repeated)
```
build/FLOE_MAKE_NXP_INPUT io/inputs/in_2000f_06p_1.h5 <nb_row> <nb_col>
```
*  Or, use the executable **FLOE_PATCHWORK_INPUT** to reproduce the whole area from different inputs (and resize for N_m x P_m).
* **WARNING** :  **FLOE_MAKE_X4_INPUT**, **FLOE_MAKE_NXP_INPUT** and **FLOE_PATCHWORK_INPUT** generate miniature packs on an window 1meter x 1 meter. You have to resize your input with **FLOE_RESIZE_INPUT**
*  In order to erase the artificial border between cells, one should run simulation without mode 2 as to change ice concentration. The given water speed is low to avoid too strong collision or displacement which can make a problem too hard to solve. The bustle, even weak, helps to rearrange the floe assembly and set the ice/ice friction mu to 0 helps to rearrange as well. Also, adding a higher concentration (in our example at 0.7)  reduces the target zone, so convergent currents, applied to the floes outside the reduced target zone, act to rearrange the configuration.
```
build/FLOE -i <initial_floe_config.h5> --fmodes 0 2 --fspeeds 0 5e-2 -c 0.7 --mu 0 --bustle 1 --nbustle 1e-7 -t 40000
```
* Then to extract an input file from the output file use **FLOE_EXTRACT_INPUT** :
```
build/FLOE_EXTRACT_INPUT io/outputs/out_JKDfE.h5 40000
Reading "io/outputs/out_JKDfE.h5" ... 
100 Floes, concentration : 59%
RECOVER : 9961.29
io/inputs/in_8000f_60p_Pj32N.h5 written
```
* Reset the kinetic energy to zero using 
```
build/FLOE_SET_AT_RESET io/inputs/in_8000f_60p_Pj32N.h5
```
