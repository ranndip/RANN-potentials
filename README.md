### Potential file syntax

The RANN potential is defined by a single text file which contains all the fitting parameters for the alloy system. The potential file also defines the active fingerprints, network architecture, activation functions, etc. The potential file is divided into several sections which are identified by one of the following keywords:

- atomtypes
- mass
- fingerprintsperelement
- fingerprints
- fingerprintconstants
- screening (optional)
- networklayers
- layersize
- weight
- bias
- activationfunctions
- calibrationparameters (ignored)
\# is treated as a comment marker, similar to LAMMPS input scripts. Sections are not required to follow a rigid ordering, but do require previous definition of prerequisite information. E.g., fingerprintconstants for a particular fingerprint must follow the fingerprints definition; layersize for a particular layer must follow the declaration of network layers.

atomtypes are defined as follows using element keywords separated by spaces.
```
atomtypes:
Fe Mg Al etc.
```
_mass_ must be specified for each element keyword as follows:
```
mass:Mg:
24.305
mass:Fe:
55.847
mass:Al:
26.982
```
_fingerprintsperelement_ specifies how many fingerprints are active for computing the energy of a given atom. This number must be specified for each element keyword. Active elements for each fingerprint depend upon the type of the central atom and the neighboring atoms. Pairwise fingerprints may be defined for a Mg atom based exclusively on its Al neighbors, for example. Bond fingerprints may use two neighborlists of different element types. In computing fingerprintsperelement from all defined fingerprints, only the fingerprints defined for atoms of a particular element should be considered, regardless of the elements used in its neighbor list. In the following code, for example, some fingerprints may compute pairwise fingerprints summing contributions about Fe atoms based on a neighbor list of exclusively Al atoms, but if there are no fingerprints summing contributions of all neighbors about a central Al atom, then fingerprintsperelement of Al is zero:
```
fingerprintsperelement:Mg:
5
fingerprintsperelement:Fe:
2
fingerprintsperelement:Al:
0
```
_fingerprints_ specifies the active fingerprints for a certain element combination. Pair fingerprints are specified for two elements, while bond fingerprints are specified for three elements. Only one fingerprints header should be used for an individual combination of elements. The ordering of the fingerprints in the network input layer is determined by the order of element combinations specified by subsequent fingerprints lines, and the order of the fingerprints defined for each element combination. Multiple fingerprints of the same style or different ones may be specified. If the same style and element combination is used for multiple fingerprints, they should have different id numbers. The first element specifies the atoms for which this fingerprint is computed while the other(s) specify which atoms to use in the neighbor lists for the computation. Switching the second and third element type in bond fingerprints has no effect on the computation:
```
fingerprints:Mg_Mg:
radial_0 radialscreened_0 radial_1
fingerprints:Mg_Al_Fe:
bond_0 bondspin_0
fingerprints:Mg_Al:
radial_0 radialscreened_0
```
The following fingerprint styles are currently defined. See the :ref:`formulation section <fingerprints>` below for their definitions:

- radial
- radialscreened
- radialspin
- radialscreenedspin
- bond
- bondscreened
- bondspin
- bondscreenedspin

_fingerprintconstants_ specifies the metaparameters for a defined fingerprint. For all radial styles, re, rc, alpha, dr, o, and n must be specified. re should usually be the stable interatomic distance, rc is the cutoff radius, dr is the cutoff smoothing distance, o is the lowest radial power term (which may be negative), and n is the highest power term. The total length of the fingerprint vector is <img src="https://latex.codecogs.com/svg.latex?(n-o&plus;1)" title="(n-o+1)" />. alpha is a list of decay parameters used for exponential decay of radial contributions. It may be set proportionally to the bulk modulus similarly to MEAM potentials, but other values may provided better fitting in special cases. Bond style fingerprints require specification of <img src="https://latex.codecogs.com/svg.latex?r_{e},&space;r_{c},&space;\alpha_{k},&space;dr,&space;k," title="r_{e}, r_{c}, \alpha_{k}, dr, k," /> and <img src="https://latex.codecogs.com/svg.latex?m" title="m" />. Here <img src="https://latex.codecogs.com/svg.latex?m" title="m" /> is the power of the bond cosines and <img src="https://latex.codecogs.com/svg.latex?k" title="k" /> is the number of decay parameters. Cosine powers go from 0 to <img src="https://latex.codecogs.com/svg.latex?m-1" title="m-1" /> and are each computed for all values of <img src="https://latex.codecogs.com/svg.latex?\alpha_{k}" title="\alpha_{k}" />. Thus the total length of the fingerprint vector is <img src="https://latex.codecogs.com/svg.latex?m*k" title="m*k" />.
