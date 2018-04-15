# PBHYB
Source files for a particle-based hybrid code for planet formation 
Instruction for particle-based hybrid code (PBHYB) for planetary accretion 

Ryuji Morishima (fastspin@hotmail.co.jp),
Feb. 1, 2018

1. Configuration

There are five directories in the pbhyb package.

a)source:   the source files of PBHYB
b)mdl2:     a part of the source file which handles 
            the machine dependent layer (mdl)
c)analysis: analysis programs, including 
            an initial condition generator
d)pebble_run: an example run with pebble supply in a gaseous disk; Run1 of Morishima (2018) (w. collisional destruction)
e)planetesimal_run : another example run starting with planetesimals only w/o a gaseous disk (perfect merging)

Units in PBHYB are: solar mass = 1, G = 1, 1 AU = 1.
Thus, 1 yr is 2*pi.


2 Source program (source)

2.1. Compilation

The directry "mdl2" must be in the same level of "source"  
(as it is in default). 

In "source", configure as follows:
 
>./configure --enable-soft-linear --enable-planets --enable-symba --enable-sm2d
  
Clean if there are any executable and object (.o) files: 

> make clean
  
Then compile  
  
> make null   
  
This creates the exacutable file "pbhyb_null".
We have only a serial version for now.

2.2 If you want to modify the source code

Most of routines are in master.c or pkd.c.
If you add a function in master.c,
you usually need to add another function in pst.c

The current code works only in a serial enviroment.
Originally, I attempted to parallelize the code, but its efficiency
was turned out to be low for N ~ 10,000. 

3. Programs for analysis (analysis)

3.1. Compilation

Clean if there are any executable and object (.o) files
in the directory "analysis"
 
> make clean 
  
Then make

> make 

3.2. Executable files

ssa:      reads a position file and creates some analyzed output files
ssic:     creates a position file (i.e. an initial condition) reading 
	  a parameter file ssic.par
collread: reads a binary collision file "ss.coll.bin" for the data 
	  stored in the N-body routine and outputs a text style data (imp.dat).  
collread_PBHYB: reads a binary collision file "ss.coll_PBHYB.bin" for the data 
	  stored in the statistical routine and outputs a text style data (imp.dat).  
	  (in defaults it outputs a collision log for a certain particle with the id = idm)


We have other programs origianlly used for PKDGRAV.
They probably need modification for use of PBHYB.
addjs:    adds Jovian planets to a position file (named such as ss.00100000)
ene:      reads a position file and calculates the total energy
me2pkd:   converts a position file in the style of Mercury code to
	  that of PBHYB 
pkd2me:   converts a position file in the style of PBHYB
	  to that of Mercury  
pkd2sw:   converts a position file in the style of PBHYB
	  to that of the Swift package 

4. Start jobs  

4.1. Necessary files

Goto the directry "run1" or make a new directry 
with ss.par and ssic.par 

ss.par: a parameter file for a job 
ssic.par: a parameter file for creating an initial condition

4.2. Creation of an initial position file 

Modify ssic.par for your purpose and execute 
 
> ../analysis/ssic
  
This creates a position file named ssic.ss.

4.3. Prepare your parameter file

Modify the parameter file, ss.par.

You generally need to modify the following lines:
dDelta		= 1.0	        # time step in units of yr/(2*pi) 
nSteps		= 6500000000     # number of steps in intervals of dDelta
N0		= 10000		# Initial number of particles
iOutInterval	= 10000  	# output file interval in timesteps (logarithmic intervals for negative; -105 gives ratio of 1.05)
iGasModel       = 1            # 0: no gas, 1: uniform dissipation 2: inside-out 3: Stepinski model 4: User design
fMt0            = 6.00694E-08  # mt0 in solar mass
fff             = 100.0        # factor for full embyro mass
ffdr            = 0.25         # factor for radial size of 2D fixed grid (fff^-1/3< ffdr <=1)
fdtheta         = 0.25         # azimuthal size of the moving grid in pi (<=1)
fDt             = 30.0         # timestep for the statistical routine divided by dDelta

Parameters for gas forces are not written in a user-friendly way in
the current code.  There are additional parameters in pkd.h 
(near the end of the file). Check msrGasTable in master.c and 
pkdGasAccel in pkd.c to see how these parameters are used. 

4.4. Excecute the job 

> ../source/pbhyb_null ss.par

4.5 Output files

After the job starts, the program produces following output files:
a) binary position files: named such as ss.0010000000 
b) collision data files for the N-body routine: ss.coll.bin (binary) and ss.coll.txt (text)
c) collision data files for the statistical routine: ss.collPBHYB.bin (binary) 
d) log file: ss.log  
e) other 

5. Restart

For a restart using a certain output position file,
1) Modify following two lines in ss.par.
   iStartstep: starting step (number after ss. in the position file)
   achInFile: input position file
2) Move ss.coll.bin to ss.coll.bin_old and ss.collPBHYB.bin to ss.collPBHYB.bin_old

   (Let us assume the time of the restart to be t_re. In the beginning of restart, 
   the code reads ss.coll.bin_old and copy collision data to a new ss.coll.bin.
   Only data with < t_re are copied in order to avoid double countsafter the restart.)
   If you have ss.coll.bin at the restart, you will get an error. 
3) Delete a lock file named .lockfile
4) Delete other output files such as ss.log and .out (not necessary)

Then execute the job again. Check the size of a new ss.coll.bin which should be 
equal to or less than ss.coll.bin_old. 
(we do not avoid double counts of collision data in ss.coll.txt in a restart)

6. Analysis of output data

6.1 ssa

ssa reads position files and creates some analyzed outputs in text style. 
You can specify as many position files as possible.

> ../analyse/ssa ss.0010000000 ss.0020000000
  
This creates following files (see ssa.c for details):
a) ssa.sss: particle list in text style for the last input file
line 1: particle id
line 2: particle class (3: full embryo, 5: sub-embryo, 6: tracer)
line 3: number of planetesimals in the tracer
line 4: mass of the tracer (planetesimal mass = line4/line3)
line 5: tracer radius (planetesimal radius = (line5^3/line3)^(1/3))
line 6-8: a, e, i
line 9-11: spin vector 
line 10,11: w (longitude of perihelion), W (argument of ascending node) 
line 12: gas mass (>0 only for giant planets)
line 13: time at which a pebble tracer is converted into a planetesimal tracer
    
b) ssa.meff: statistics for big and small particles
c) ssa.planets: orbital elements of specified planets 
                      (you need to specify id's of big particles')

Other outputs (not checked yet)
d) ssa.abin: number distribution in radial bins 
e) ssa.mbin: number distribution in mass bins 
f) ssa.out: summary1

6.2 ene
"ene" calculates the total energies and angular momentum. 

> ../analysis/ene ss.0010000000 ss.0020000000

Output are displayed on the terminal and also tabulated in ene.out.

6.3 collread 
"collread" reads ss.coll.bin and outputs collision data in text style (imp.dat).

> ../analysis/collread ss.coll.bin
 
Columns in imp.dat are as follows:
1)time in yr 
2) id 3) mass 4)x 5)y 6)z 7)vx 8)vy 9)vz of partlce 1
10)-17) the same as  2)-9) but for particle 2 

Rules of id's:
If two particles merge, the id of a new particle is one 
from the larger impactor. If two colliding particles had 
the same mass, the id of the new particle is smaller one 
of the colliding particles id's. 
  
6.4 collreadPBHYB
see the source file and modify it for your purpose.

ss.collPBHYB.bin is usually huge. It is not a good idea 
to put all collision log in the text file.
 



 


