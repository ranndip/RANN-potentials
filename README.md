[LAMMPS implementation of rapid artificial neural network derived interatomic potentials](https://www.sciencedirect.com/science/article/pii/S0927025621002068)

###  ![#f03c15](https://via.placeholder.com/15/f03c15/000000?text=+)  Potential file syntax

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

_**atomtypes**_ are defined as follows using element keywords separated by spaces.
```
atomtypes:
Fe Mg Al etc.
```
_**mass**_ must be specified for each element keyword as follows:
```
mass:Mg:
24.305
mass:Fe:
55.847
mass:Al:
26.982
```
_**fingerprintsperelement**_ specifies how many fingerprints are active for computing the energy of a given atom. This number must be specified for each element keyword. Active elements for each fingerprint depend upon the type of the central atom and the neighboring atoms. Pairwise fingerprints may be defined for a Mg atom based exclusively on its Al neighbors, for example. Bond fingerprints may use two neighborlists of different element types. In computing fingerprintsperelement from all defined fingerprints, only the fingerprints defined for atoms of a particular element should be considered, regardless of the elements used in its neighbor list. In the following code, for example, some fingerprints may compute pairwise fingerprints summing contributions about Fe atoms based on a neighbor list of exclusively Al atoms, but if there are no fingerprints summing contributions of all neighbors about a central Al atom, then fingerprintsperelement of Al is zero:
```
fingerprintsperelement:Mg:
5
fingerprintsperelement:Fe:
2
fingerprintsperelement:Al:
0
```
_**fingerprints**_ specifies the active fingerprints for a certain element combination. Pair fingerprints are specified for two elements, while bond fingerprints are specified for three elements. Only one fingerprints header should be used for an individual combination of elements. The ordering of the fingerprints in the network input layer is determined by the order of element combinations specified by subsequent fingerprints lines, and the order of the fingerprints defined for each element combination. Multiple fingerprints of the same style or different ones may be specified. If the same style and element combination is used for multiple fingerprints, they should have different id numbers. The first element specifies the atoms for which this fingerprint is computed while the other(s) specify which atoms to use in the neighbor lists for the computation. Switching the second and third element type in bond fingerprints has no effect on the computation:
```
fingerprints:Mg_Mg:
radial_0 radialscreened_0 radial_1
fingerprints:Mg_Al_Fe:
bond_0 bondspin_0
fingerprints:Mg_Al:
radial_0 radialscreened_0
```
The following fingerprint styles are currently defined. See the [formulation section](#fingerprints) below for their definitions:

- radial
- radialscreened
- radialspin
- radialscreenedspin
- bond
- bondscreened
- bondspin
- bondscreenedspin

_**fingerprintconstants**_ specifies the metaparameters for a defined fingerprint. For all radial styles,<img src="https://latex.codecogs.com/svg.latex?r_{e},&space;r_{c},&space;\alpha,&space;dr,&space;o," title="r_{e}, r_{c}, \alpha, dr, o," /> and <img src="https://latex.codecogs.com/svg.latex?n" title="n" /> must be specified. <img src="https://latex.codecogs.com/svg.latex?r_{e}" title="r_{e}" /> should usually be the stable interatomic distance, <img src="https://latex.codecogs.com/svg.latex?r_{c}" title="r_{c}" /> is the cutoff radius, <img src="https://latex.codecogs.com/svg.latex?dr" title="dr" /> is the cutoff smoothing distance, <img src="https://latex.codecogs.com/svg.latex?o" title="o" /> is the lowest radial power term (which may be negative), and <img src="https://latex.codecogs.com/svg.latex?n" title="n" /> is the highest power term. The total length of the fingerprint vector is <img src="https://latex.codecogs.com/svg.latex?(n-o&plus;1)" title="(n-o+1)" />. alpha is a list of decay parameters used for exponential decay of radial contributions. It may be set proportionally to the bulk modulus similarly to MEAM potentials, but other values may provided better fitting in special cases. Bond style fingerprints require specification of <img src="https://latex.codecogs.com/svg.latex?r_{e},&space;r_{c},&space;\alpha_{k},&space;dr,&space;k," title="r_{e}, r_{c}, \alpha_{k}, dr, k," /> and <img src="https://latex.codecogs.com/svg.latex?m" title="m" />. Here <img src="https://latex.codecogs.com/svg.latex?m" title="m" /> is the power of the bond cosines and <img src="https://latex.codecogs.com/svg.latex?k" title="k" /> is the number of decay parameters. Cosine powers go from 0 to <img src="https://latex.codecogs.com/svg.latex?m-1" title="m-1" /> and are each computed for all values of <img src="https://latex.codecogs.com/svg.latex?\alpha_{k}" title="\alpha_{k}" />. Thus the total length of the fingerprint vector is <img src="https://latex.codecogs.com/svg.latex?m*k" title="m*k" />.

```
fingerprintconstants:Mg_Mg:radialscreened_0:re:
3.193592
fingerprintconstants:Mg_Mg:radialscreened_0:rc:
6.000000
fingerprintconstants:Mg_Mg:radialscreened_0:alpha:
5.520000 5.520000 5.520000 5.520000 5.520000
fingerprintconstants:Mg_Mg:radialscreened_0:dr:
2.806408
fingerprintconstants:Mg_Mg:radialscreened_0:o:
-1
fingerprintconstants:Mg_Mg:radialscreened_0:n:
3
```
_**screening**_ specifies the Cmax and Cmin values used in the screening fingerprints. Neighbors' contribution to the fingerprint are ommitted if they are blocked by a closer neighbor, and reduced if they are partially blocked. Larger values of <img src="https://latex.codecogs.com/svg.latex?C_{min}" title="C_{min}" /> correspond to neighbors being blocked more easily. <img src="https://latex.codecogs.com/svg.latex?C_{max}" title="C_{max}" /> cannot be greater than 3, and <img src="https://latex.codecogs.com/svg.latex?C_{min}" title="C_{min}" /> cannot be greater than  <img src="https://latex.codecogs.com/svg.latex?C_{max}" title="C_{max}" /> or less than zero. Screening may be ommitted in which case the default values <img src="https://latex.codecogs.com/svg.latex?C_{max}=2.8" title="C_{max}=2.8" />, <img src="https://latex.codecogs.com/svg.latex?C_{min}=0.8" title="C_{min}=0.8" /> are used. Since screening is a bond computation, it is specified separately for each combination of three elements in which the latter two may be interchanged with no effect.
```
screening:Mg_Mg_Mg:Cmax:
2.800000
screening:Mg_Mg_Mg:Cmin:
0.400000
```
_**networklayers**_ species the size of the neural network for each atom. It counts both the input and output layer and so is 2+hiddenlayers.
```
networklayers:Mg:
3
```
_**layersize**_ specifies the length of each layer, including the input layer and output layer. The input layer is layer 0. The size of the input layer size must match the summed length of all the fingerprints for that element, and the output layer size must be 1:
```
layersize:Mg:0:
14
layersize:Mg:1:
20
layersize:Mg:2:
1
```
**_weight_** specifies the weight for a given element and layer. Weight cannot be specified for the output layer. The weight of layer <img src="https://latex.codecogs.com/svg.latex?i" title="i" /> is a mxn matrix where m is the layer size of <img src="https://latex.codecogs.com/svg.latex?i" title="i" /> and <img src="https://latex.codecogs.com/svg.latex?n" title="n" /> is the layer size of <img src="https://latex.codecogs.com/svg.latex?i&plus;1" title="i+1" />:
```
weight:Mg:0:
w11 w12 w13 ...
w21 w22 w23 ...
...
```
_**bias**_ specifies the bias for a given element and layer. Bias cannot be specified for the output layer. The bias of layer <img src="https://latex.codecogs.com/svg.latex?i" title="i" />  is a <img src="https://latex.codecogs.com/svg.latex?n\times&space;1" title="n\times 1" /> vector where <img src="https://latex.codecogs.com/svg.latex?n" title="n" />  is the layer size of  <img src="https://latex.codecogs.com/svg.latex?i&plus;1" title="i+1" />:
```
bias:Mg:0:
b1
b2
b3
...
```
**_activationfunctions_** specifies the activation function for a given element and layer. Activation functions cannot be specified for the output layer:
```
activationfunctions:Mg:0:
sigI
activationfunctions:Mg:1:
linear
```
The following activation styles are currently specified. See the [formulation section](#activations) below for their definitions.
- sigI
- linear

**_calibrationparameters_** specifies a number of parameters used to calibrate the potential. These are ignored by LAMMPS.

### ![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+)  Formulation
In the RANN formulation, the total energy of a system of atoms is given by:

<img src="https://latex.codecogs.com/svg.latex?E&space;=&space;\sum_{\alpha}&space;E^{\alpha}\\\\&space;E^{\alpha}&space;=&space;{}^{N}\!A^{\alpha}\\\\&space;{}^{n&plus;1}\!A_i^{\alpha}&space;=&space;{}^{n}\!F\left({}^{n}\!W_{ij}{\;}^{n}\!A_j^{\alpha}&plus;{}^{n}\!B_i\right)\\\\&space;{}^{0}\!A_i^{\alpha}&space;=&space;\left[\begin{array}{c}&space;{}^1\!S\!f^\alpha\\&space;{}^2\!S\!f^\alpha&space;\\...\\\end{array}\right]" title="E = \sum_{\alpha} E^{\alpha}\\\\ E^{\alpha} = {}^{N}\!A^{\alpha}\\\\ {}^{n+1}\!A_i^{\alpha} = {}^{n}\!F\left({}^{n}\!W_{ij}{\;}^{n}\!A_j^{\alpha}+{}^{n}\!B_i\right)\\\\ {}^{0}\!A_i^{\alpha} = \left[\begin{array}{c} {}^1\!S\!f^\alpha\\ {}^2\!S\!f^\alpha \\...\\\end{array}\right]" />

Here <img src="https://latex.codecogs.com/svg.latex?E^\alpha" title="E^\alpha" /> is the energy of atom <img src="https://latex.codecogs.com/svg.latex?\alpha,&space;{}^n\!F(),&space;{}^n\!W_{ij}" title="\alpha, {}^n\!F(), {}^n\!W_{ij}" /> and <img src="https://latex.codecogs.com/svg.latex?{}^n\!B_i" title="{}^n\!B_i" /> are the activation function, weight matrix and bias vector of the <img src="https://latex.codecogs.com/svg.latex?n-th" title="n-th" /> layer respectively. The inputs to the first layer are a collection of structural fingerprints which are collected and reshaped into a single long vector. The individual fingerprints may be defined in any order and have various shapes and sizes. Multiple fingerprints of the same type and varying parameters may also be defined in the input layer.

Eight types of structural fingerprints are currently defined. In the following, <img src="https://latex.codecogs.com/svg.latex?\beta" title="\beta" /> and <img src="https://latex.codecogs.com/svg.latex?\gamma" title="\gamma" /> span the full neighborlist of atom <img src="https://latex.codecogs.com/svg.latex?\alpha" title="\alpha" />. <img src="https://latex.codecogs.com/svg.latex?\delta_i" title="\delta_i" /> are decay metaparameters, and <img src="https://latex.codecogs.com/svg.latex?r_e" title="r_e" /> is a metaparameter roughly proportional to the first neighbor distance. <img src="https://latex.codecogs.com/svg.latex?r_c" title="r_c" /> and <img src="https://latex.codecogs.com/svg.latex?\Delta&space;r" title="\Delta r" /> are the neighbor cutoff distance and cutoff smoothing distance respectively. <img src="https://latex.codecogs.com/svg.latex?S^{\alpha\beta}" title="S^{\alpha\beta}" /> is the MEAM screening function [(Baskes)](https://www.sciencedirect.com/science/article/pii/S0254058497802520), <img src="https://latex.codecogs.com/svg.latex?s_i^\alpha" title="s_i^\alpha" /> and <img src="https://latex.codecogs.com/svg.latex?s_i^\beta" title="s_i^\beta" /> are the atom spin vectors [(Tranchida)](https://www.sciencedirect.com/science/article/pii/S0021999118304200). <img src="https://latex.codecogs.com/svg.latex?r^{\alpha\beta}" title="r^{\alpha\beta}" /> is the distance from atom <img src="https://latex.codecogs.com/svg.latex?\alpha" title="\alpha" /> to atom <img src="https://latex.codecogs.com/svg.latex?\beta" title="\beta" />, and <img src="https://latex.codecogs.com/svg.latex?\theta^{\alpha\beta\gamma}" title="\theta^{\alpha\beta\gamma}" /> is the bond angle:

<img src="https://latex.codecogs.com/svg.latex?cos\left(\theta^{\alpha\beta\gamma}\right)=\frac{\mathbf{r}^{\alpha\beta}&space;\cdot&space;\mathbf{r}^{\alpha\gamma}}{r^{\alpha\beta}r^{\alpha\gamma}}" title="cos\left(\theta^{\alpha\beta\gamma}\right)=\frac{\mathbf{r}^{\alpha\beta} \cdot \mathbf{r}^{\alpha\gamma}}{r^{\alpha\beta}r^{\alpha\gamma}}" />

<img src="https://latex.codecogs.com/svg.latex?S^{\alpha\beta}" title="S^{\alpha\beta}" /> is defined as [(Baskes)](https://www.sciencedirect.com/science/article/pii/S0254058497802520):

<img src="https://latex.codecogs.com/svg.latex?X^{\gamma\beta}&space;=&space;\left(\frac{r^{\gamma\beta}}{r^{\alpha\beta}}\right)^2\\&space;\\&space;X^{\alpha\gamma}&space;=&space;\left(\frac{r^{\alpha\gamma}}{r^{\alpha\beta}}\right)^2\\&space;\\&space;C&space;=&space;\frac{2\left(X^{\alpha\gamma}&plus;X^{\gamma\beta}\right)-\left(X^{\alpha\gamma}-X^{\gamma\beta}\right)^2-1}{1-\left(X^{\alpha\gamma}-X^{\gamma\beta}\right)^2}\\&space;\\&space;f_c(x)&space;=&space;\left[\begin{array}{l}&space;1&space;\;&space;\:&space;x&space;\geq&space;1\\&space;\left(1-\left(1-x\right)^4\right)^2&space;\;&space;\:&space;0<x<1\\0\;&space;\;&space;x\leq0\end{array}\right.\\&space;\\&space;S^{\alpha\beta\gamma}&space;=&space;f_c\left(\frac{C-C_{min}}{C_{max}-C_{min}}\right)\\&space;\\&space;S^{\alpha\beta}&space;=&space;\prod_\gamma&space;S^{\alpha\beta\gamma}\\" title="X^{\gamma\beta} = \left(\frac{r^{\gamma\beta}}{r^{\alpha\beta}}\right)^2\\ \\ X^{\alpha\gamma} = \left(\frac{r^{\alpha\gamma}}{r^{\alpha\beta}}\right)^2\\ \\ C = \frac{2\left(X^{\alpha\gamma}+X^{\gamma\beta}\right)-\left(X^{\alpha\gamma}-X^{\gamma\beta}\right)^2-1}{1-\left(X^{\alpha\gamma}-X^{\gamma\beta}\right)^2}\\ \\ f_c(x) = \left[\begin{array}{l} 1 \; \: x \geq 1\\ \left(1-\left(1-x\right)^4\right)^2 \; \: 0<x<1\\0\; \; x\leq0\end{array}\right.\\ \\ S^{\alpha\beta\gamma} = f_c\left(\frac{C-C_{min}}{C_{max}-C_{min}}\right)\\ \\ S^{\alpha\beta} = \prod_\gamma S^{\alpha\beta\gamma}\\" />

_**The structural <a name="fingerprints"></a>  fingerprints are computed as follows:**_

**- radial**

<img src="https://latex.codecogs.com/svg.latex?\!S\!f_i^\alpha&space;=&space;\sum_{\beta}&space;\left(\frac{r^{\alpha\beta}}{r_e}\right)^ie^{-\delta_i&space;\frac{r^{\alpha\beta}}{r_e}}f_c\left(\frac{r_c-r^{\alpha\beta}}{dr}\right)" title="\!S\!f_i^\alpha = \sum_{\beta} \left(\frac{r^{\alpha\beta}}{r_e}\right)^ie^{-\delta_i \frac{r^{\alpha\beta}}{r_e}}f_c\left(\frac{r_c-r^{\alpha\beta}}{dr}\right)" />

**- bond**

<img src="https://latex.codecogs.com/svg.latex?\!S\!f_{ij}^\alpha&space;=&space;\sum_{\beta}\sum_{\gamma}&space;\left(cos(\theta_{\alpha\beta\gamma})\right)^ie^{-\delta_j&space;\frac{r^{\alpha\beta}}{r_e}}e^{-\delta_j&space;\frac{r^{\alpha\gamma}}{r_e}}f_c\left(\frac{r_c-r^{\alpha\beta}}{dr}\right)f_c\left(\frac{r_c-r^{\alpha\gamma}}{dr}\right)" title="\!S\!f_{ij}^\alpha = \sum_{\beta}\sum_{\gamma} \left(cos(\theta_{\alpha\beta\gamma})\right)^ie^{-\delta_j \frac{r^{\alpha\beta}}{r_e}}e^{-\delta_j \frac{r^{\alpha\gamma}}{r_e}}f_c\left(\frac{r_c-r^{\alpha\beta}}{dr}\right)f_c\left(\frac{r_c-r^{\alpha\gamma}}{dr}\right)" />

**- radialscreened**

<img src="https://latex.codecogs.com/svg.latex?\!S\!f_i^\alpha&space;=&space;\sum_{\beta}&space;\left(\frac{r^{\alpha\beta}}{r_e}\right)^ie^{-\delta_i&space;\frac{r^{\alpha\beta}}{r_e}}S^{\alpha\beta}f_c\left(\frac{r_c-r^{\alpha\beta}}{dr}\right)" title="\!S\!f_i^\alpha = \sum_{\beta} \left(\frac{r^{\alpha\beta}}{r_e}\right)^ie^{-\delta_i \frac{r^{\alpha\beta}}{r_e}}S^{\alpha\beta}f_c\left(\frac{r_c-r^{\alpha\beta}}{dr}\right)" />

**- bondscreened**

<img src="https://latex.codecogs.com/svg.latex?\!S\!f_{ij}^\alpha&space;=&space;\sum_{\beta}\sum_{\gamma}&space;\left(cos(\theta_{\alpha\beta\gamma})\right)^ie^{-\delta_j&space;\frac{r^{\alpha\beta}}{r_e}}e^{-\delta_j&space;\frac{r^{\alpha\gamma}}{r_e}}S^{\alpha\beta}S^{\alpha\gamma}f_c\left(\frac{r_c-r^{\alpha\beta}}{dr}\right)f_c\left(\frac{r_c-r^{\alpha\gamma}}{dr}\right)" title="\!S\!f_{ij}^\alpha = \sum_{\beta}\sum_{\gamma} \left(cos(\theta_{\alpha\beta\gamma})\right)^ie^{-\delta_j \frac{r^{\alpha\beta}}{r_e}}e^{-\delta_j \frac{r^{\alpha\gamma}}{r_e}}S^{\alpha\beta}S^{\alpha\gamma}f_c\left(\frac{r_c-r^{\alpha\beta}}{dr}\right)f_c\left(\frac{r_c-r^{\alpha\gamma}}{dr}\right)" />

**- radialspin**

<img src="https://latex.codecogs.com/svg.latex?\!S\!f_i^\alpha&space;=&space;\sum_{\beta}&space;\left(\frac{r^{\alpha\beta}}{r_e}\right)^ie^{-\delta_i&space;\frac{r^{\alpha\beta}}{r_e}}\left(\mathbf{s^\alpha&space;\cdot&space;s^\beta}\right)f_c\left(\frac{r_c-r^{\alpha\beta}}{dr}\right)" title="\!S\!f_i^\alpha = \sum_{\beta} \left(\frac{r^{\alpha\beta}}{r_e}\right)^ie^{-\delta_i \frac{r^{\alpha\beta}}{r_e}}\left(\mathbf{s^\alpha \cdot s^\beta}\right)f_c\left(\frac{r_c-r^{\alpha\beta}}{dr}\right)" />

**- bondspin**

<img src="https://latex.codecogs.com/svg.latex?\!S\!f_{ij}^\alpha&space;=&space;\sum_{\beta}\sum_{\gamma}&space;\left(cos(\theta_{\alpha\beta\gamma})\right)^ie^{-\delta_j&space;\frac{r^{\alpha\beta}}{r_e}}e^{-\delta_j&space;\frac{r^{\alpha\gamma}}{r_e}}\left(\mathbf{s^\alpha&space;\cdot&space;s^\beta}\right)\left(\mathbf{s^\alpha&space;\cdot&space;s^\gamma}\right)f_c\left(\frac{r_c-r^{\alpha\beta}}{dr}\right)f_c\left(\frac{r_c-r^{\alpha\gamma}}{dr}\right)" title="\!S\!f_{ij}^\alpha = \sum_{\beta}\sum_{\gamma} \left(cos(\theta_{\alpha\beta\gamma})\right)^ie^{-\delta_j \frac{r^{\alpha\beta}}{r_e}}e^{-\delta_j \frac{r^{\alpha\gamma}}{r_e}}\left(\mathbf{s^\alpha \cdot s^\beta}\right)\left(\mathbf{s^\alpha \cdot s^\gamma}\right)f_c\left(\frac{r_c-r^{\alpha\beta}}{dr}\right)f_c\left(\frac{r_c-r^{\alpha\gamma}}{dr}\right)" />

**- radialscreenedspin**

<img src="https://latex.codecogs.com/svg.latex?\!S\!f_i^\alpha&space;=&space;\sum_{\beta}&space;\left(\frac{r^{\alpha\beta}}{r_e}\right)^ie^{-\delta_i&space;\frac{r^{\alpha\beta}}{r_e}}S^{\alpha\beta}\left(\mathbf{s^\alpha&space;\cdot&space;s^\beta}\right)f_c\left(\frac{r_c-r^{\alpha\beta}}{dr}\right)" title="\!S\!f_i^\alpha = \sum_{\beta} \left(\frac{r^{\alpha\beta}}{r_e}\right)^ie^{-\delta_i \frac{r^{\alpha\beta}}{r_e}}S^{\alpha\beta}\left(\mathbf{s^\alpha \cdot s^\beta}\right)f_c\left(\frac{r_c-r^{\alpha\beta}}{dr}\right)" />

**- bondscreenedspin**

<img src="https://latex.codecogs.com/svg.latex?\!S\!f_{ij}^\alpha&space;=&space;\sum_{\beta}\sum_{\gamma}&space;\left(cos(\theta_{\alpha\beta\gamma})\right)^ie^{-\delta_j&space;\frac{r^{\alpha\beta}}{r_e}}e^{-\delta_j&space;\frac{r^{\alpha\gamma}}{r_e}}S^{\alpha\beta}S^{\alpha\gamma}\left(\mathbf{s^\alpha&space;\cdot&space;s^\beta}\right)\left(\mathbf{s^\alpha&space;\cdot&space;s^\gamma}\right)f_c\left(\frac{r_c-r^{\alpha\beta}}{dr}\right)f_c\left(\frac{r_c-r^{\alpha\gamma}}{dr}\right)" title="\!S\!f_{ij}^\alpha = \sum_{\beta}\sum_{\gamma} \left(cos(\theta_{\alpha\beta\gamma})\right)^ie^{-\delta_j \frac{r^{\alpha\beta}}{r_e}}e^{-\delta_j \frac{r^{\alpha\gamma}}{r_e}}S^{\alpha\beta}S^{\alpha\gamma}\left(\mathbf{s^\alpha \cdot s^\beta}\right)\left(\mathbf{s^\alpha \cdot s^\gamma}\right)f_c\left(\frac{r_c-r^{\alpha\beta}}{dr}\right)f_c\left(\frac{r_c-r^{\alpha\gamma}}{dr}\right)" />

_**The <a name="activations"></a> activation functions are computed as follows:**_

- sigI
<img src="https://latex.codecogs.com/svg.latex?F^{sigI}(x)&space;=&space;0.1x&plus;0.9ln\left(e^x&plus;1\right)" title="F^{sigI}(x) = 0.1x+0.9ln\left(e^x+1\right)" />

- linear
<img src="https://latex.codecogs.com/svg.latex?F^{linear}(x)&space;=&space;x" title="F^{linear}(x) = x" />
