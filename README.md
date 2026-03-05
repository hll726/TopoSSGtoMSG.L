# TopoSSGtoMSG
The TopoSSGtoMSG package enables the computation of topological properties of arbitrary collinear magnets using high-symmetry-point wave functions obtained from VASP under both spin space groups and magnetic space groups. It encompasses the vanishing spin–orbit coupling (SOC) regime, the negligible-SOC limit, and the realistic-SOC case for various magnetic-moment orientations. This unified framework provides a systematic means to elucidate how SOC, together with the orientation of magnetic moments once SOC is included, reshapes the topological characteristics of materials.
   
# Installation
   - unzip the TopoSSGtoMSG-V2.zip file.
   - Open a Mathematica notebook.
   - Run Import["Dir\TopoSSGtoMSG.wl"] where Dir is the path to the TopoSSGtoMSG.wl file.
   - After running, a folder selection prompt will appear. Select the folder containing TopoSSGtoMSG.wl.

# Tutorial
Using the collinear altermagnetic material 0.402_Sr<sub>4</sub>Fe<sub>4</sub>O<sub>11</sub> as an example, we introduce how to use the various modules of **TopoSSGtoMSG**. The first module, **CheckSSGinfo**, provides the basic information of the spin space group *G* of the material, including the primitive lattice vectors, the nontrivial part of the collinear spin space group operations, and the classification of magnetic-moment directions in the collinear spin space group *G*. It should be noted that any spin group operation can be written in the form {R_s || R_r | τ}, where R_s denotes the operation in spin space, while R_r and τ are the rotational and translational parts of the corresponding real-space group operation, respectively. For the nontrivial part of the spin group, for convenience, we represent the real-space group operation by its action on the fractional coordinates (x, y, z) defined with respect to the primitive basis vectors **a<sub>1</sub>**, **a<sub>2</sub>**, and **a<sub>3</sub>**. That is, an arbitrary spin group operation is written as {R_s || R_r (x, y, z) + τ}. For altermagnetic 0.402_Sr<sub>4</sub>Fe<sub>4</sub>O<sub>11</sub>, the corresponding spin space group is *G* = 65.1.2.5.L, where the notation of spin space groups follows that on https://cmpdc.iphy.ac.cn/ssg/.


```mathematica
CheckSSGinfo["65.1.2.5.L"]
(*CheckSSGinfo[SSGname], SSGname denotes the name of the collinear spin space group*)
```
<img src="https://github.com/hll726/TopoSSGtoMSG/raw/main/src/ssgoperationandmmd.png" alt="Lattice" width="50%">

Here, *E* denotes the identity operation, and *T* denotes the time-reversal operation, which reverses the spin. The quantities *n*<sub>x</sub>, *n*<sub>y</sub>, and *n*<sub>z</sub> are the components of the unit vector of the magnetic-moment direction along the *x*-, *y*-, and *z*-axes in the Cartesian coordinate system. Next, we introduce the **CheckDegenarcy** module, which displays the labels and degeneracies of the irreducible (co-)representations at high-symmetry points, high-symmetry lines, high-symmetry planes, or generic ***k***-points *(u, v, w)* under either the collinear spin space group or the magnetic space group determined by a specific class of magnetic-moment orientations. This module is useful for understanding the degeneracies of symmetry-enforced band crossings in case III, which are identified by the core module **GetTopfilling**, introduced below. First, we present the results for the spin space group *G* = 65.1.2.5.L.

```mathematica
CheckDegenarcy["65.1.2.5.L", 0]
(*CheckDegenarcy[SSGname, momclass], SSGname denotes the name of the collinear spin space group and momclass = 0 corresponds to calculations without SOC (under spin space group)*)
```
<img src="https://github.com/hll726/TopoSSGtoMSG/raw/main/src/ssg.png" alt="Lattice" width="80%">

Then, we present the results for the magnetic space group 65.486 (Belov–Neronova–Smirnova notation, magnetic space group type III), which is determined by selecting class 4 (1, 0, 0) of magnetic-moment directions in the spin space group *G* = 65.1.2.5.L.

```mathematica
CheckDegenarcy["65.1.2.5.L", 4]
(*CheckDegenarcy[SSGname, momclass], SSGname denotes the name of the collinear spin space group and momclass ≠ 0 corresponds to calculations with SOC.
The specific nonzero value of momclass labels the corresponding class of magnetic-moment orientations in the given collinear spin space group*)
```
<img src="https://github.com/hll726/TopoSSGtoMSG/raw/main/src/msg.png" alt="Lattice" width="80%">

Next, we introduce how to use **TopoSSGtoMSG** to compute the topology of collinear magnetic materials. Here, we consider three cases: without SOC (the topology is diagnosed by the spin space group), the negligible-SOC limit (the topology is diagnosed by the magnetic space group), and the realistic-SOC case (the topology is diagnosed by the magnetic space group). For the latter two cases, various magnetic-moment orientations can be considered. The computational workflow is illustrated in the flowchart below.


<img src="https://github.com/hll726/TopoSSGtoMSG/raw/main/src/topossgtomsg.png" alt="Lattice" width="80%">

Specifically, the workflow can be divided into three steps. 
- The first step is a self-consistent VASP calculation.
- The second step is a non-self-consistent calculation to obtain the wave functions at high-symmetry points, where the content of the **KPOINTS** file is generated by the **GenKpath** module.
- The third step is the topological analysis, which is performed using the **GetTopfilling** module and requires the **WAVECAR** and **OUTCAR** files generated in the second step.

Here, `GetTopfilling[SSGname, momclass, δν, n]` computes, in parallel, the band topology at relative electronic filling *δν* (electron filling minus N<sub>e</sub>, where N<sub>e</sub> is the number of valence electrons). The meaning of the parameters is as follows:
- ***momclass*** = 0 and ***n*** = 0 correspond to calculations under the collinear spin space group in the vanishing spin–orbit coupling limit. 
- ***momclass***≠ 0 and ***n*** = 0 correspond to calculations under the magnetic space group determined by a specific class of magnetic-moment orientations, with realistic SOC. 
- ***momclass*** ≠ 0 and ***n***= 1 correspond to calculations under the magnetic space group determined by a specific class of magnetic-moment orientations in the negligible SOC limit. 

The resulting topological classifications are divided into three cases:

- case **I**: possibly trivial.
- case **II**:all band crossings (nodal points or nodal lines) at generic ***k***-points under spin space group, or magnetic topological insulators/Weyl semimetals under magnetic space group.
- case **III**: symmetry-enforced band crossings at high-symmetry points, high-symmetry lines, high-symmetry planes, or generic ***k***-points (*u*, *v*, *w*).


Below, we illustrate the computational procedure using the altermagnetic material 0.402_Sr<sub>4</sub>Fe<sub>4</sub>O<sub>11</sub> (*U* = 0 eV) as an example. We first introduce how to compute the topology in the absence of SOC, which is diagnosed by the spin space group. First, perform the self-consistent calculation. After it is completed, use the **GenKpath** module to generate the **KPOINTS** file required for the non-self-consistent calculation. For 0.402_Sr<sub>4</sub>Fe<sub>4</sub>O<sub>11</sub> (*U* = 0 eV), the results are as follows.

```mathematica
GenKpath["65.1.2.5.L", 0]
(*GenKpath[SSGname, momclass], SSGname denotes the name of the collinear spin space group and momclass = 0 corresponds to calculations without SOC (under spin space group)*)
```
<img src="https://github.com/hll726/TopoSSGtoMSG/raw/main/src/kpath1.png" alt="Lattice" width="40%">

Place the **WAVECAR** and **OUTCAR** files obtained from the non-self-consistent calculation into a folder (e.g., `dir1`). Then run the following command. A folder-selection prompt will appear; select the folder `dir1` and wait for the calculation to complete.

```mathematica
GetTopfilling["65.1.2.5.L", 0, 0, 0]
```


The computed results are shown below.

<img src="https://github.com/hll726/TopoSSGtoMSG/raw/main/src/ssgtopology.png" alt="Lattice" width="100%">

In the first-level structure, the entry `0` denotes the relative electronic filling *δν*. 

In the second-level structure, **III** indicates that the topology at relative electronic filling *v* = 0 belongs to case **III**, namely symmetry-enforced band crossings at high-symmetry points, high-symmetry lines, high-symmetry planes, or generic ***k***-points (*u*, *v*, *w*). The second level further provides detailed information on the symmetry-enforced band crossings. 

In this example, the symmetry-enforced band crossings occur on the high-symmetry plane P3: (−*v*, *u*, 0) (see the result of CheckDegenarcy["65.1.2.5.L", 0]). Taking the first entry `{{-v, u, 0}, {{0, 0, 0}, {1/2, 0, 0}}, 1, {11, 12}}`  as an illustration of the data format, the information is interpreted as follows. First, the symmetry-enforced band crossing is located on the high-symmetry plane P3: (−*v*, *u*, 0). Second, the crossing lies along the line connecting the high-symmetry points {0, 0, 0} and {1/2, 0, 0}. Finally, it is formed by the crossing of the irreducible (co-)representations 11 and 12.

According to the information provided by `CheckDegenarcy["65.1.2.5.L", 0]`, both of these irreducible (co-)representations are one-dimensional. Therefore, the resulting band crossing is twofold degenerate. The subsequent entries in the parentheses describe band crossings along multiple directions, indicating the presence of Weyl nodal rings encircling the high-symmetry points {1/2, 0, 0}, {0, 1/2, 0},  {1/2, 1/2, 0},or {0, 0, 0}.

Next, we use the **WAVECAR** and **OUTCAR** files obtained from calculations without spin–orbit coupling to diagnose the band topology in the negligible-SOC limit, where the topology is determined by the magnetic space group. In the example below, we set `momclass = 4`, which corresponds to magnetic moments aligned along class 4, i.e., the (1, 0, 0) direction. The topology of the corresponding magnetic space group is then computed in the negligible-SOC limit.

```mathematica
GetTopfilling["65.1.2.5.L", 4, 0, 1]
```
The computed results are shown below.

<img src="https://github.com/hll726/TopoSSGtoMSG/raw/main/src/msgtopology3.png" alt="Lattice" width="40%">

In the second-level structure, **II** indicates that the topology at relative electronic filling *v* = 0 belongs to case **II**, namely band crossings (nodal points or nodal lines) at generic ***k***-points under the spin space group, or magnetic topological insulators / Weyl semimetals under the magnetic space group. Here we denote a generic momentum as  (*u*, *v*, *w*). The entry ``{2, 2}`` in the parentheses indicates that the symmetry-indicator group is `ℤ₂ × ℤ₂`, while ``{0, 1}`` specifies the symmetry-indicator values `(0, 1)`.


Finally, we introduce how to compute the topology with realistic SOC, which is diagnosed by the magnetic space group. First, perform a self-consistent calculation. After it is completed, use the **GenKpath** module to generate the **KPOINTS** file required for the non-self-consistent calculation. For 0.402_Sr<sub>4</sub>Fe<sub>4</sub>O<sub>11</sub> (*U* = 0 eV) with the magnetic-moment direction belonging to class 4 (1, 0, 0), the results are shown below.


```mathematica
GenKpath["65.1.2.5.L", 4]
(**GenKpath[SSGname, momclass], SSGname denotes the name of the collinear spin space group and momclass ≠ 0 corresponds to calculations with SOC.
The specific nonzero value of momclass labels the corresponding class of magnetic-moment orientations in the given collinear spin space group*)
```
<img src="https://github.com/hll726/TopoSSGtoMSG/raw/main/src/kpath2.png" alt="Lattice" width="40%">

Place the **WAVECAR** and **OUTCAR** files obtained from the non-self-consistent calculation into a folder (e.g., `dir2`). Then run the following command. A folder-selection prompt will appear; select the folder `dir2` and wait for the calculation to complete.

```mathematica
GetTopfilling["65.1.2.5.L", 4, 0, 0]
```

The computed results are shown below.


<img src="https://github.com/hll726/TopoSSGtoMSG/raw/main/src/msgtopology2.png" alt="Lattice" width="100%">

In the first-level structure, the entry `0` denotes the relative electronic filling *v*. In the second-level structure, **III** indicates that the topology at relative electronic filling *v* = 0 belongs to case **III**, namely symmetry-enforced band crossings at high-symmetry points, high-symmetry lines, high-symmetry planes, or generic ***k***-points (*u*, *v*, *w*).

The second level further provides detailed information on the symmetry-enforced band crossings. In this example, the symmetry-enforced band crossings occur on the high-symmetry lines L1: (0, 0, *w*) and L6: (-*w*, -*w*, 0), as well as on the high-symmetry plane P2: (*u*, *u*, *v*) (see the results of `CheckDegenarcy["65.1.2.5.L", 4]`). These symmetry-enforced band crossings are twofold degenerate. Combining these results, we identify a Weyl nodal line encircling the high-symmetry point {0, 0, 0}.

By comparing the results of `GetTopfilling["65.1.2.5.L", 4, 0, 1]` and `GetTopfilling["65.1.2.5.L", 4, 0, 0]`, we find that in the altermagnetic material 0.402_Sr<sub>4</sub>Fe<sub>4</sub>O<sub>11</sub>, SOC is relatively strong, and the topology cannot be inferred from the spin space group alone.

Finally, we provide data for 484 experimentally synthesized collinear magnets from the MAGNDATA database that are converged both with and without SOC for at least one value of the Hubbard $U$ (see `484materials-HSPs-symmetry-data.mx`). This dataset is intended to facilitate inspection of the high-symmetry-points symmetry data diagnosed by spin space groups and magnetic space groups (realistic SOC) at each converged Hubbard $U$ and relative electron filling *δν* (with *δν* ranging from -8 to 8).

Using the modules **ColliSSGCalTopoElectron** and **MSGCalTopoElectron**, together with the provided high-symmetry-points symmetry data, users can compute the band topology of a given material at a specified Hubbard *U* and relative electron filling *δν* under spin space groups and magnetic space groups (realistic SOC), respectively. First, we briefly describe the data structure of `484materials-HSPs-symmetry-data.mx`, which is organized as follows:

```mathematica
{{material formula and BCSID,{{U=i (eV),{{under spin space group: *δν*= -8,high-symmetry-points symmetry data for *U=i eV* and *δν=-8* under spin space group},...},{{under magnetic space group (realistic spin-orbital coupling): *δν*= -8,high-symmetry-points symmetry data for *U=i eV* and *δν=-8* under magnetic space group (realistic spin-orbital coupling)},...}},...}},...}
```
We use the altermagnetic material 0.402_Sr<sub>4</sub>Fe<sub>4</sub>O<sub>11</sub> as an example to demonstrate how to use the modules **ColliSSGCalTopoElectron** and **MSGCalTopoElectron**, together with the provided high-symmetry-points symmetry data.

```mathematica
data = Import["dir3\\484materials-HSPs-symmetry-data.mx"];
(* dir3 is the directory containing 484materials-HSPs-symmetry-data.mx *)
ColliSSGCalTopoElectron[data[[83, 2, 1, 2, 9, 2]]]
MSGCalTopoElectron[data[[83, 2, 1, 3, 9, 2]]]
```

`ColliSSGCalTopoElectron[data[[83, 2, 1, 2, 9, 2]]]` reproduces the result of `GetTopfilling["65.1.2.5.L", 0, 0, 0]`, while `MSGCalTopoElectron[data[[83, 2, 1, 3, 9, 2]]]` reproduces the result of `GetTopfilling["65.1.2.5.L", 4, 0, 0]`.





# References
L. Huang, X.Wan, and F. Tang, “Prediction of magnetic topological materials combining spin and magnetic space groups,” (2026), arXiv:2601.05142.

# Contact

L. Huang: 602025220022@smail.nju.edu.cn

# Note
**ToMSGKpoint** (https://github.com/FengTang1990/ToMSGKpoint) is a user-friendly Mathematica package designed to efficiently compute the irreducible (co-)representations of electronic bands for arbitrary crystalline materials (magnetic or nonmagnetic), both with and without spin–orbit coupling. It is particularly suitable for high-throughput studies that require determining the topological properties of crystalline materials. The output files are formatted to serve as input for subsequent topological analysis using **CalTopoEvol** (https://github.com/FengTang1990/CalTopoEvol).



