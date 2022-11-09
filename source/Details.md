# Details on differents points
<a name="subseclatex"></a>

This section comes back on different points about FloeDyn

## On data from TOPAZ

The default mode `--fmodes 1 1` , read a file containting wind speed and currents. According to Rabatel (2015, page 132) :
The wind speeds are from ERAinterim data from February 2008 taken in the Barentz Sea at latitude 80.2 ◦ N and longitude 36.8 ◦ E. This position in time and in space may correspond to that of a marginal area of sea ice. The velocities of the surface current are obtained from the TOPAZ sea ice and ocean model (ref TOPAZ4) at the same times and positions as the wind speeds.
Some references :
* the wind come from ERAinterim : [[Dee and al., 2011]](#References)
* ocean current from TOPAZ4 : 
	+on TOPAZ : [[Rainer and al., 2002]](#References) and [[Bleck and al., 2008]](#References)
	+ on the data in FloeDyn [[Sakov and al., 2012]](#References)
## The vortex mode

The vortex mode is given by the mode 5 for the atmosphere, in this case, water speed is automatically put at 0 (see *src/floe/dynamics/physical_data.hpp* method `get_speed` near ligne 383 ). There is two possibilities for the vortex mode :
* With the mode 5, a single random vortex is created, this vortex is moved toward the floe pack during the simulation. Its attributes are : the vortex radius, the center of the eye (`m_vortex_origin`), the vortex speed, and the max norm of the wind. Default values are given by the code and it can be modified in *src/floe/dynamics/physical_data.hpp* method `init_random_vortex` (near lign 344). The random parameter (given by gen.seed(90839527656); ) allows to create a random vortex, but you can also determine the vortex data by changing attributes in this method . This is based on a formulation originally proposed for tropical cyclones [[Willoughby and Rahn, 2004]](#References), then recently adapted for wormholes in the southern region [[Jouanno and al., 2016]](#References) :
	+ the analytical vortex is a perfectly circular wind system, with only the tangential component of the wind being non-zero
tangential component of the wind
	+ It is defined by the $R_c$ radius of the heart of the vortex ("the eye of the cyclone"), the maximum wind speed $U_m$, and the speed of displacement of the centre of the vortex $V$ (for simplicity, we can consider linear trajectories of this centre)
linear trajectories of this centre).
	+ in the N hemisphere, the winds turn counter-clockwise
	+ the norm of the wind speed (tangential component) will be :
	
$$
\begin{align*}
    u_\theta&=U_m\frac{r}{R_c} \qquad \text{ si }0\leq r\leq R_c\\
    u_\theta&=U_m\frac{R_c}{r} \qquad \text{ si }r\geq R_c
\end{align*}
$$

	+ We can pass analytical vortices over our domain by randomly varying
$R_c$, $U_m$, $V$ as well as the direction of the vortex trajectory, among the following typical values [[Smirnova and al. 2015]](#References):

$$
\begin{align*}
    R_c= 300 \pm 110 \; km\\
    V= 8 \pm 4 \; m/s^{-1} \\
    15 \leq U_m \leq30  \; m/s^{-1} 
\end{align*}
$$

* with mode 6, multiple vortexes are created like in mode 5 and are moved toward differents points in the target zone. The vortices are generated and positioned at the initialization of the simulation. They are distributed in rings around the ice field. The user defines :
	+ n c , n v respectively the number of rings and the number of vortices.
	+ the number of vortices per ring.
	+ d V as the distance between the origin of the ice field and the 1st corona (no vortex
can be closer to the field than d V ).
	+ d C as the distance between two corona.
The generation of the vortexes proceeds as follows:
	+ an angle and a distance are randomly defined to place the 1st vortex in the 1st
crown.
	+ for the following wormholes, the angle is defined from the previous one by adding a random angle belonging to $[\pi/2,\pi]$, the distance varies slightly at random placing the wormhole in its associated in its associated corona.
	+ Each vortex is associated with a point randomly drawn within the Target zone. The trajectory of the vortex passes through this point.
	
Example to create 3 vortex, 1 vortex by zone (defined as rings surrounding the ice fields), thes rings are of size 100 km and the distance of the first ring to the ice field origin is 300 km: 
```
build/FLOE --fmodes 6 0 -i io/inputs/in_2800f_75p_tpCrm.h5 -v 3 1 100 300 -t 400000
```


## The OBL option

The Ocean boundary layer option activates the coupling between the floe and the ocean. This coupling model is based on the article Recent mechanical weakening of the Arctic sea ice cover as revealed from larger inertial oscillations, [Gimbert et al.]
 see *https://agupubs.onlinelibrary.wiley.com/doi/full/10.1029/2011JC007633*


Without this option, the water speed is forced (like the wind speed) and the floe speed for a given time step is determined by these two speeds (see Matthias Rabatel's thesis for more details).
Using OBL option, water speed is recalculated at each time step using wind speed and floe speed. More precisely: the given current velocity (noted $u_{w,data}$) is used as a low frequency acceleration but the velocity has higher frequency as :
$$
\begin{align*}
&    U_w(t+\Delta t) = U_w(t) + \Delta t\left( F_a (U_a,U_w(t),t) + F_f (U_f,U_w,t) + C(U_w(t),t) + a_{w,data} (t)\right)\\
&a_{w,data} (t+\delta t)= \dfrac{U_{w,data}(t+\delta_t) -U_{w,data}(t) }{\delta t}
\text{ and } a_{w,data} (T_f)=0
\end{align*}
$$
Where $F_a$ and $F_f$ are respectively the effect due to atmosphere drag and floe drag, $C$ represent the effect due to the Coriolis effect.
Thus, with the OBL option, even if there is no current at low frequency ($U_{w,data}= 0$), the current velocity ($U_w$) will still be modified by the floe speed and the wind speed and may be non-zero.

## Extract data from output files

From the parent folder (Floe_Cpp) in folder *extract data*, we created where we have created an easy-to-use 'toolkit' for first data processing. 
This 'toolkit' is separate in two python3 objects:
* `Data_extractor.py`: to extract data from on one output file.
* `Multiple_data_extractor.py`: use `Data_extractor` to extract data from a list of output file. 
Some practical applications are proposed in these objects:
* `Data_extractor` possibilities are explain in details in `trajectory_analisys.py` :
	+ convert floes trajectories, floes speed, mass center position, OBL speed in array like object
	+ sort floes by area/dimeter
	+ plot the differents trajectories/mass center position/speed etc...
	+ compute and plot FFT of the different signal, variance of distance from origine, variance of the dispersion etc...
	+ compute and plot polynomial approximations variance for the differents group sorted by area or diameter
* `Multiple_data_extractor`: do the same that `Data_extractor` for different simulation, considering the mean on all simulations and managing the different computation from `Data_extractor` for each simulations. Possibilities are explain in details in `trajectories_analisys.py`

These objects have been designed to be easy to use, adapt and modify to the user's needs

### References 

[Dee and al., 2011,] DP Dee, SM Uppala, AJ Simmons, Paul Berrisford, P Poli, S Kobayashi, U Andrae, MA Balmaseda, G Balsamo, and P Bauer. The era-interim reanalysis : Conﬁguration and performance of the data assimilation system. Quarterly Journal of the Royal Meteorological Society, 137(656) :553–597, 2011
[Bleck and al., 2002] Rainer Bleck. An oceanic general circulation model framed in hybrid isopycnic-cartesian coordinates. Ocean modelling, 4(1) :55–88, 2002
[Bertino and al.  2008] Laurent Bertino, KA Lisæter, and S Scient. The topaz monitoring and prediction system for the atlantic and arctic oceans. Journal of Operational Oceanography, 1(2) :15–18, 2008
[Sakov and al., 2012] P. Sakov, F. Counillon, L. Bertino, K. A. Lisaeter, P. R. Oke, and A. Korablev. Topaz4 : an ocean-sea ice data assimilation system for the north atlantic and arctic.
[Willoughby and Rahn, 2004] Willoughby, H., and M. Rahn (2004), Parametric representation of the primary hurricane vortex. Part
I: Observations and evaluation of the Holland (1980) model, Monthly Weather Review, 132(12),
3033-3048
[Jouanno and al., 2016]Jouanno, J., X. Capet, G. Madec, G. Roullet, P. Klein, and S. Masson (2016), Dissipation of the energy
imparted by mid-latitude storms in the Southern Ocean, Ocean Sci., 12, 7432-7769
[Smirova and al., 2015] Smirnova, J. E., P. A. Golubkin, L. P. Bobylev, E. V. Zabolotskikh, and B. Chapron (2015), Polar low
climatology over the Nordic and Barents seas based on satellite passive microwave data, Geophys.
Res. Lett., 42(13), 5603-5609.
