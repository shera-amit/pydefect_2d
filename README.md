pydefect_2d
===========

First-principles calculations are increasingly used to study dominant point defects in 2D materials.
However, the number of studies on these defects is still limited compared to those in 3D materials.
A major reason for this limitation is the difficulty in finite size corrections,
which are crucial for calculating charged defects under periodic boundary conditions (PBC).
Several groups have reported methods to correct defect formation energies,
yet routinely applying these techniques in practical applications remains challenging.
This is because each defect-charge combination requires a unique correction process.
Considering typical native defects in 2D materials, such as vacancies, antisites, interstitials,
and adsorbed atoms, requires calculations of dozens of defect-charge combinations.
Therefore, automating the correction process and minimizing computational costs is vital
for advancing defect calculations in 2D materials.
To this end, we have developed a code to automate these corrections.
We have also introduced an interpolation technique to lessen computational costs and simplify the processes.

* In this code, we employ the correction method proposed by Noh et al. 
[Phys. Rev. B, 89, 205417 (2014)](https://journals.aps.org/prb/abstract/10.1103/PhysRevB.89.205417)
and Komsa [Phys. Rev. X 4, 031044 (2014)](https://journals.aps.org/prx/abstract/10.1103/PhysRevX.4.031044) 
(NK method).

* In the current implementation, we can treat tetragonal systems only
For convention, the directions perpendicular to the surfaces are aligned with the $z$-direction.

* In the current implementation, only VASP is supported.

* This code creates the `correction.json` and `eigenvalue_shift.yaml` files, 
which can be used in the pydefect code.

Requirements
------------
- Python 3.10 or higher
- [vise](https://github.com/kumagai-group/vise)
- [pydefect](https://github.com/kumagai-group/pydefect)
- see requirements.txt for others

License
-----------------------
Python code is licensed under the MIT License.


Explanation of physical parameters
---------------------------------
The command line options correspond to several physical quantities.
Lengths are given in angstroms unless noted otherwise and fractional
coordinates are measured along the supercell $z$ axis from 0 to 1.

* **--center**: fractional $z$ position(s) of the 2D layer(s) used when
  constructing dielectric profiles.
* **--std_dev**: standard deviation $\sigma$ of the Gaussian smearing
  applied either to the dielectric profile or to the defect charge.
* **--step_width** / **--step_width_z**: transition widths of the
  step-like dielectric function in the in-plane and out-of-plane
  directions.
* **--range**: lower and upper bounds of candidate charge positions in
  fractional coordinates.
* **--mesh_distance**: spacing between successive charge positions used
  when scanning the above range.
* **--z_pos**: explicit $z$ positions for which long-range energies are
  calculated.
* **--k_max** and **--k_mesh_dist**: parameters of the reciprocal-space
  integration for computing isolated Gaussian energies.
* **--slab_center**: reference center of the slab for estimating the
  eigenvalue shift.

The ``gauss_model_from_z`` command parallelizes over the supplied
``--z_pos`` values unless ``--no_multiprocess`` is given.


Workflow for 2D Point Defect Calculations
-----------------------------------------
<img src="https://github.com/kumagai-group/pydefect_2d/assets/4986887/99bd7211-588b-4292-9453-906b457f2650" width="600" alt="">

Our goal is to evaluate the corrections of defect formation energies (E<sub>f</sub>).
The workflow is depicted above.

1. (Step 5) Generate `unitcell.yaml` that is generated from the pristine slab model calculations using pydefect. 
Only dielectric constants are used in this code.

2. (Step 6) The NK correction method necessitates knowledge of the dielectric profiles.
   Use either ``gauss_diele_dist`` (``gdd``) or ``step_diele_dist`` (``sdd``)
   to build these profiles. The ``-c`` option gives the layer center in
   fractional coordinates, ``-w`` (and ``-wz``) specify the interface width in
   Å, and ``-s`` sets the Gaussian smearing of the transition.
   Example:
```pydefect_2d sdd -c 0.5 -s 0.5 -w 7.15 -wz 7.15 -u unitcell.yaml -pl ../../defects/6_30A/perfect/LOCPOT --denominator 2```

3. Create `correction/` in the defect calculation directory.

4. (Step 7) To locate the charge center, ``1d_gauss_models`` (``1gm``) creates
   ``1d_gauss/`` and outputs 1D potentials for a series of ``z_0`` values. The
   two numbers supplied to ``-r`` give the fractional coordinate range, while
   ``-m`` sets the spacing between samples.
   Example:
```pydefect_2d 1gm -s ../../supercell_info.json -r 0.3 0.5 -dd ../dielectric_const_dist.json```

5. (Step 8)
Compute the one-dimensional potential from first-principles calculations and determine the Gaussian charge center.
An example is:
   ```pydefect_2d 1fp -d . -pl ../../perfect/LOCPOT -od ../1d_gauss```

6. (Steps 9 and 10)
   ``gauss_model_from_z`` (``gmz``) evaluates the long-range correction for
   selected ``-z`` positions (fractional coordinates). Multiple positions are
   processed in parallel; use ``--no_multiprocess`` to disable this feature.
   Example:
```pydefect_2d gmz -z 0.3{0,2,4,6,8} 0.4{0,2,4,6,8} 0.5 -s ../../supercell_info.json -cd . -dd ../dielectric_const_dist.json```

7. (Step 11)
Then, we interpolate long-range correction term as a function of z<sub>0</sub>, and  
generate **gauss_energies.json** inside defects/correction/.
An example is:
```pydefect_2d ge```

8. (Step 12) Generate slab_model.json and correction.json (at this point, it converges with pydefect).
If slab_center is given, `eigenvalue_shift.yaml` is also generated, which can be used for shifting the eigenvalues in the pydefect processes.
An example is:
```pydefect_2d 1sm -d Va_MoS6_-2 -dd dielectric_const_dist.json -od 1d_gauss -g correction/gauss_energies.json -s 0.5```

Development notes
-------------------
### Bugs, requests and questions
Please use the [Issue Tracker](https://github.com/kumagai-group/pydefect_2d/issues) to report bugs, request features.

### Code contributions
Although pydefect_2d is free to use, we sincerely appreciate if you help us to improve this code.
The simplest but most valuable contribution is to send the feature requests and bug reports.

Please report any bugs and issues at PyDefect's [GitHub Issues page](https://github.com/kumagai-group/pydefect_2d).
Please use the ["Fork and Pull"](https://guides.github.com/activities/forking/) workflow to make contributions and stick as closely as possible to the following:

- Code style follows [PEP8](http://www.python.org/dev/peps/pep-0008) and [Google's writing style](https://google.github.io/styleguide/pyguide.html).
- Add unittests wherever possible including scripts for command line interfaces.

### Tests
Run the tests using `pytest pydefect_2d`.

Citing pydefect_2d
---------------
If pydefect_2d has been used in your research, please cite the following paper.

[Corrections on Formation Energies and Eigenvalues of Point Defect Calculations in Two-Dimensional Materials]<br>
Yu Kumagai<br>


Contact info
--------------
Yu Kumagai<br>
yukumagai@tohoku.ac.jp<br>

Tohoku University (Japan)
