![GoodVibes](https://github.com/patonlab/GoodVibes/blob/master/GoodVibes.png)
===
GoodVibes2 is a Python program to compute thermochemical data from one or a series of electronic structure calculations in Gaussian or ORCA.
In GoodVibes2 contributions from low frequencies are virtually omitted via the use of Truhlar's approximation with cutoff of 175 cm<sup>-1</sup>, which we have found to be the most robust and accurate in [Velmiskina et al. JPC, 2025](https://doi.org/10.1063/5.0255622).

It is a fork of [GoodVibes](https://github.com/patonlab/GoodVibes) developed by [Paton Research group](http://patonlab.com/), which has been used since 2015 by several groups, primarily to correct the poor description of low frequency vibrations by the rigid-rotor harmonic oscillator treatment.

The following description is mostly identical to the original GoodVibes **with differences highlighted in bold**:

This version includes thermochemistry at variable temperature/concentration, various quasi-harmonic entropy and enthalpy schemes, automated detection of frequency scaling factors, D3-dispersion corrections calculations, Boltzmann averaging, duplicate conformer detection, automated tabulation and plotting of energy profiles, and error checking.

All (electronic, translational, rotational and vibrational) partition functions are recomputed and will be adjusted to any temperature or concentration. These default to 298.15 Kelvin and 1 atmosphere.

The program will attempt to parse the level of theory and basis set used in the calculations and then try to apply the appropriate vibrational (zpe) scaling factor. Scaling factors are taken from the [Truhlar group database](https://t1.chem.umn.edu/freqscale/index.html).

#### Quasi-Harmonic Approximation
Two types of quasi-harmonic approximation are readily applied. The first is vibrational entropy: below a given cut-off value vibrational normal modes are not well described by the rigid-rotor-harmonic-oscillator (RRHO) approximation and an alternative expression is instead used to compute the associated entropy. The quasi-harmonic vibrational entropy is always less than or equal to the standard (RRHO) value obtained using Gaussian. Two literature approaches have been implemented. In the simplest approach, from [Cramer and Truhlar](http://pubs.acs.org/doi/abs/10.1021/jp205508z),<sup>1</sup> all frequencies below the cut-off are uniformly shifted up to the cut-off value before entropy calculation in the RRHO approximation. Alternatively, as proposed by [Grimme](http://onlinelibrary.wiley.com/doi/10.1002/chem.201200497/full),<sup>2</sup> entropic terms for frequencies below the cut-off are obtained from the free-rotor approximation; for those above the RRHO expression is retained. A damping function is used to interpolate between these two expressions close to the cut-off frequency.

The second type of quasi-harmonic approximation available is applied to the vibrational energy used in enthalpy calculations. Similar to the entropy corrections, the enthalpy correction implements a quasi-harmonic correction to the RRHO vibrational energy computed in DFT methods. The quasi-harmonic enthalpy value as specified by [Head-Gordon](https://pubs.acs.org/doi/10.1021/jp509921r)<sup>3</sup> will be less than or equal to the uncorrected value using the RRHO approach, as the quasi-RRHO value of the vibrational energy used to compute the enthalpy is damped to approach a value of 0.5RT, opposed to the RRHO value of RT. Because of this, the quasi-harmonic enthalpy correction is appropriate for use in systems and reactions resulting in a loss of a rotational or translational degree of freedom.

#### Installation
*  Manually Cloning the repository **https://github.com/TheorChemGroup/GoodVibes2.git** and then adding the location of the GoodVibes directory to the PYTHONPATH environment variable.
*  Run the script with your Gaussian **or ORCA** output files (the program expects .log or .out extensions)

#### Citing GoodVibes2
**J. Velmiskina, TheorChemGroup/GoodVibes2, 2025, https://github.com/TheorChemGroup/GoodVibes2**

#### Using GoodVibes2

<pre>
python -m goodvibes [-q] [--qs grimme/truhlar] [--qh] [-f cutoff_freq] [--fs S_cutoff_freq] [--fh H_cutoff_freq]
[--check] [-t temperature] [-c concentration] [--ti 't_initial, t_final, step'] [--ee] [--bav "global" or "conf"]
<b>[--symmbyhand] [--random 'cutoff_value,spread_range,number_of_tries'] [--norot]</b>
[--cosmo cosmo_filename] [--cosmoint cosmo_filename,initial_temp,final_temp] [-v frequency_scale_factor]
[--vmm mm_freq_scale_factor][--ssymm] [--spc link/filename] [--boltz] [--dup][--pes pes_yaml] [--nogconf]
[--graph graph_yaml] [--cpu] [--imag] [--invertifreq invert_cutoff <b> or "auto"</b>] [--freespace solvent_name] [--output output_name]
[--media solvent_name] [--xyz] [--csv] [--custom_ext file_extension] <output_file(s)>
<code>
*	The `-h` option gives help by listing all available options, default values and units, and proper usage.
*	The `-q` option turns on quasi-harmonic corrections to entropy, defaulting to the <b>Truhlar's</b> method for entropy correction with cutoff of 175 cm<sup>-1</sup>.
*	The `--qs` option selects the approximation for the quasi-harmonic entropic correction: `--qs truhlar` or `--qs grimme` request the options explained above. Both avoid the tendency of RRHO vibrational entropies towards infinite values for low frequencies.
*	The `--qh` option selects the approximation for the quasi-harmonic enthalpy correction. Calling this argument requests the enthalpy correction option explained above. This replaces harmonic energy contributions with a quasi-RRHO vibrational energy term. If not specified then the <b>regular harmonic approximation</b> is used.
*	The `-f` option specifies the frequency cut-off for both entropy and enthalpy calculations (in wavenumbers) i.e. `-f 10` would use 10 cm<sup>-1</sup> when calculating thermochemical values. The default value is <b>175 cm<sup>-1</sup></b>. N.B. when set to zero all thermochemical values match standard (i.e. harmonic) Gaussian quantities.
*	The `--fs` option specifies the frequency cut-off for only entropy calculations(in wavenumbers). `--fs 40` would use 40 cm<sup>-1</sup> when calculating entropies. The default value is <b>175 cm<sup>-1</sup></b>.
*	The `--fh` option specifies the frequency cut-off for only enthalpy calculations (in wavenumbers).`--fh 200` would use 200 cm<sup>-1</sup> when calculating enthalpies. The default value is <b>175 cm<sup>-1</sup></b>.
*	The `--check` option applies the checks specified above to the calculation output files and displays a pass or fail message to the user.
*	The `-t` option specifies temperature (in Kelvin). N.B. This does not have to correspond to the temperature used in the Gaussian calculation since all thermal quantities are reevalulated by GoodVibes at the requested temperature. The default value is 298.15 K.
*	The `-c` option specifies concentration (in mol/l).  It is important to notice that the ideal gas approximation is used to relate the concentration with the pressure, so this option is the same as the Gaussian Pressure route line specification. The correction is applied to the Sackur-Tetrode equation of the translational entropy e.g. `-c 1` corrects to a solution-phase standard state of 1 mol/l. The default is 1 atmosphere.
*	The `--ti` option specifies a temperature interval (for example to see how a free energy barrier changes with the temperature). Usage is `--ti 'initial_temperature, final_temperature, step_size'`. The step_size is optional, the default is set by the relationship (final_temp-initial_temp) / 10
*	<b>The `--symmbyhand` option takes a file naming pattern `"*_symm*.log"` (e.g. structure_symmC1.log, structure_symmD4h.out), and will use symmetry number corresponding to the symmetry from the name of the file for the system.
*	The `--random` option simulates the spread of values of frequencies below provided cut-off according to a uniform distribution of a given range. Usage is `'--random cutoff_value,spread_range,number_of_tries'`.
*	The `--norot` option turns off rotational contributions to the Gibbs free energy.</b>
*	The `--ee` option takes a file naming pattern (such as `"*_R*:*_S*"`) with files named as structure_R.log, structure_S.log, and will calculate and display values for stereoisomer excess (in %), ratio, major isomer present, and ddG.
*	The `--cosmo` option can be used to read Gibbs Free Energy of Solvation data from a COSMO-RS .out formatted file. GSOLV should be used as a COSMO-RS input with no argument. `-c 1` should be used in conjunction with this argument.
*	The `--cosmo_int` option allows for Gibbs Free Energy of Solvation calculated using COSMO-RS with a temperature interval to be applied at a range of temperatures. Since temperature gaps may not be consistent, the interval is automatically detected. Usage is `--cosmo_int cosmo_gsolv.out,initial_temp,final_temp`. GoodVibes will detect temperatures within the range provided.
*	The `-v` option is a scaling factor for vibrational frequencies. DFT-computed harmonic frequencies tend to overestimate experimentally measured IR and Raman absorptions. Empirical scaling factors have been determined for several functional/basis set combinations, and these are applied automatically using values from the Truhlar group<sup>4</sup> based on detection of the level of theory and basis set in the output files. This correction scales the ZPE by the same factor, and also affects vibrational entropies. The default value when no scaling factor is available is 1 (no scale factor). The automated scaling can also be suppressed by `-v 1.0`
*	The `--vmm` option is a second scaling factor for vibrational frequencies when performing QM/MM calculations with ONIOM. The correction is applied using the additional information in the output file, %ModelSys and %RealSys. This correction is only applied when requested with ONIOM calculation files. The option is activated with the command `-vmm scale_factor`
*	The `--ssymm` option will apply a symmetry correction to the entropy by detecting a molecule's internal and external symmetry. Symmetry correction is calculated separately (GoodVibes does not read the symmetry number from QM packages when this option is on to avoid duplicate corrections).
*	The `--spc` option can be used to obtain single point energy corrected values. For multi-step jobs in which a frequency calculation is followed by an additional (e.g. single point energy) calculation, the energy is taken from the final job and all thermal corrections are taken from the frequency calculation. Alternatively, the energy can be taken from an additional file.
*	The `--boltz` option will display the Boltzmann weighted factors based on free energy of each specified output file.
*	The `--dup` option will check multiple output files for duplicate structures based on energy, rotational constants and calculated frequencies. Cutoffs are currently specified as: energy cutoff = 0.1 mHartree; RMS Rotational Constant cutoff = 0.1 GHz; RMS Freq cutoff = 10 wavenumbers, Max Freq difference = 10 wavenumbers.
*	The `--pes` option takes a .yaml file input (see template below) along with calculation output files to allow for the construction of a potential energy surface from relative computed Gibbs free-energy values.
*	The `--nogconf` option will turn off a correction to the Gibbs free-energy due to multiple conformations when constructing a potential energy surface (use only with --pes option). Default is to calculate Gconf correction.
*	The `--graph` option takes a .yaml file input (see template below) along with calculation output files and will compute and graph relative Gibbs free-energy values along a reaction path (requires matplotlib library to be installed)
*	The `--cpu` option will add up all of the CPU time across all files (including single point calculations if requested).
*	The `--imag` option will print any imaginary frequencies (in wavenumbers) for each structure. Presently, all are reported. The hard-coded variable im_freq_cutoff can be edited to change this. To generate new input files (i.e. if this is an undesirable imaginary frequency) see [pyQRC](https://github.com/bobbypaton/pyQRC)
*	The `--invertifreq` option converts imaginary frequencies into real ones. If supplied by a number it real-izes frequencies with absolute values above this number. <b>If supplied by "auto" it inverts all imaginary frequencies if output file corresponds to a minimum search, or all imaginary frequencies but the lowest one if output file corresponds to a transition state search.</b>
*	The `--freespace` option specifies the solvent. The amount of free space accessible to the solute is computed based on the solvent's molecular and bulk densities. This is then used to correct the volume available to each molecule from the ideal gas approximation used in the Sackur-Tetrode calculation of translational entropy, as proposed by [Shakhnovich and Whitesides](http://pubs.acs.org/doi/abs/10.1021/jo970944f).<sup>5</sup> The keywords H2O, toluene, DMF (N,N-dimethylformamide), AcOH (acetic acid) and chloroform are recognized.
*	The `--output` option is used to change the default output file name to a specified name instead. Use as  `--output NAME` to change the name of the output file of thermochemical data from "GoodVibes.dat" to "GoodVibes_NAME.dat"
*	The `--media` option applies an entropy correction to calculations done on solvent molecules calculated from their standard concentration. `-c 1` should be used in conjunction with this argument.
*	The `--xyz` option will write all molecular Cartesian coordinates to a .xyz output file.
*	The `--csv` option will write GoodVibes calculated thermochemical data to a .csv output file.
*	The `--custom_ext` option allows for custom file extensions to be used. Current default calculation output files accepted are `.log` or `.out` file extensions. New extensions can be detected by using GoodVibes with the option `--custom_ext file_extension`.
*	The `--bav` option allows the user to choose how the average moment of inertia is computed, used in computing the free-rotor entropy. Options are `--bav global` to have all molecules computed with the same moment of inertia=10*10-44 kg m2 or `--bav conf` to use the averaged rotational constants parsed from Gaussian output files to compute the average moment of inertia
*	The `--g4` option allows the user to analyze G4 calculations from Gaussian. This option might be combined with a scaling factor of 0.9854 for vibrational frequencies (`-v 0.9854`) as suggested previously (doi: 10.1021/jp508422u).

<b>Note: the examples below corresponds to behavior of the original GoodVibes. The behavior of the GoodVibes2 should be essentially identical, but for exact values due to changes to the defaults. </b>

#### Example 1: Grimme-type quasi-harmonic correction with a (Grimme type) cut-off of 150 cm<sup>-1</sup>
```python
python -m goodvibes examples/methylaniline.out -f 150

   Structure                    E        ZPE             H        T.S     T.qh-S          G(T)       qh-G(T)
   *********************************************************************************************************
o  methylaniline      -326.664901   0.142118   -326.514489   0.039668   0.039465   -326.554157   -326.553954
   *********************************************************************************************************

```

The output shows both standard harmonic and quasi-harmonic corrected thermochemical data (in Hartree). The corrected enthalpy and entropy values are always less than or equal to the harmonic value.

#### Example 2: Quasi-harmonic thermochemistry with a larger basis set single point energy correction link job
```python
python -m goodvibes examples/ethane_spc.out --spc link

   Structure                E_SPC             E        ZPE         H_SPC        T.S     T.qh-S      G(T)_SPC   qh-G(T)_SPC
   ***********************************************************************************************************************
o  ethane_spc          -79.858399    -79.830421   0.073508    -79.780448   0.027569   0.027570    -79.808017    -79.808019
   ***********************************************************************************************************************
```

This calculation contains a multi-step job: an optimization and frequency calculation with a small basis set followed by (--Link1--) a larger basis set single point energy. Note the use of the `--spc link` option. The standard harmonic and quasi-harmonic corrected thermochemical data are obtained from the small basis set partition function combined with the larger basis set single point electronic energy. In this example, GoodVibes automatically recognizes the level of theory used in the frequency calculation, B3LYP/6-31G(d), and applies the appropriate scaling factor of 0.977 (this can be suppressed to apply no scaling with -v 1.0)

Alternatively, if a single point energy calculation has been performed separately, provided both file names share a common root e.g. `ethane.out` and `ethane_TZ.out` then use of the `--spc TZ` option is appropriate. This will give identical results as above.

```python
python -m goodvibes examples/ethane.out --spc TZ

   Structure                E_SPC             E        ZPE         H_SPC        T.S     T.qh-S      G(T)_SPC   qh-G(T)_SPC
   ***********************************************************************************************************************
o  ethane              -79.858399    -79.830421   0.073508    -79.780448   0.027569   0.027570    -79.808017    -79.808019
   ***********************************************************************************************************************
```


#### Example 3: Changing the temperature (from standard 298.15 K to 1000 K) and concentration (from standard state in gas phase, 1 atm, to standard state in solution, 1 mol/l)
```python
python -m goodvibes examples/methylaniline.out –t 1000 –c 1.0

   Structure                    E        ZPE             H        T.S     T.qh-S          G(T)       qh-G(T)
   *********************************************************************************************************
o  methylaniline      -326.664901   0.142118   -326.452307   0.218212   0.216559   -326.670519   -326.668866
   *********************************************************************************************************
```

This correction from 1 atm to 1 mol/l is responsible for the addition 1.89 kcal/mol to the Gibbs energy of each species (at 298K). It affects the translational entropy, which is the only component of the molecular partition function to show concentration dependence. In the example above the correction is larger due to the increase in temperature.

#### Example 4: Analyzing the Gibbs energy across an interval of temperatures 300-1000 K with a stepsize of 100 K, applying a (Truhlar type) cut-off of 100 cm<sup>-1</sup>
```python
python -m goodvibes examples/methylaniline.out --ti '300,1000,100' --qs truhlar -f 120

   Structure               Temp/K                        H        T.S     T.qh-S          G(T)       qh-G(T)
   ******************************************************************************************************
o  methylaniline            300.0              -326.514399   0.040005   0.039842   -326.554404   -326.554241
o  methylaniline            400.0              -326.508735   0.059816   0.059596   -326.568551   -326.568331
o  methylaniline            500.0              -326.501670   0.082625   0.082349   -326.584296   -326.584020
o  methylaniline            600.0              -326.493429   0.108148   0.107816   -326.601577   -326.601245
o  methylaniline            700.0              -326.484222   0.136095   0.135707   -326.620317   -326.619930
o  methylaniline            800.0              -326.474218   0.166216   0.165772   -326.640434   -326.639990
o  methylaniline            900.0              -326.463545   0.198300   0.197800   -326.661845   -326.661346
o  methylaniline           1000.0              -326.452307   0.232169   0.231614   -326.684476   -326.683921
   ******************************************************************************************************
```

Note that the energy and ZPE are not printed in this instance since they are temperature-independent. The Truhlar-type quasi-harmonic correction sets all frequencies below than 120 cm<sup>-1</sup> to a value of 100. Constant pressure is assumed, so that the concentration is recomputed at each temperature.

#### Example 5: Analyzing the Gibbs Energy using scaled vibrational frequencies
```python
python -m goodvibes examples/methylaniline.out -v 0.95

   Structure                    E        ZPE             H        T.S     T.qh-S          G(T)       qh-G(T)
   *********************************************************************************************************
o  methylaniline      -326.664901   0.135012   -326.521265   0.040238   0.040091   -326.561503   -326.561356
   *********************************************************************************************************
```

The frequencies are scaled by a factor of 0.95 before they are used in the computation of the vibrational energies (including ZPE) and entropies.

#### Example 6: Writing Cartesian coordinates
```python
python -m goodvibes examples/HCN*.out --xyz
```

Optimized cartesian-coordinates found in files HCN_singlet.out and HCN_triplet.out are written to Goodvibes_output.xyz

#### Example 7: Analyzing multiple files at once
```python
python -m goodvibes examples/*.out --cpu

   Structure                    E        ZPE             H        T.S     T.qh-S          G(T)       qh-G(T)
   *********************************************************************************************************
o  Al_298K            -242.328708   0.000000   -242.326347   0.017670   0.017670   -242.344018   -242.344018
o  Al_400K            -242.328708   0.000000   -242.326347   0.017670   0.017670   -242.344018   -242.344018
o  H2O                 -76.368128   0.020772    -76.343577   0.021458   0.021458    -76.365035    -76.365035
o  HCN_singlet         -93.358851   0.015978    -93.339373   0.022896   0.022896    -93.362269    -93.362269
o  HCN_triplet         -93.153787   0.012567    -93.137780   0.024070   0.024070    -93.161850    -93.161850
o  allene             -116.569605   0.053913   -116.510916   0.027618   0.027621   -116.538534   -116.538537
o  benzene            -232.227201   0.101377   -232.120521   0.032742   0.032745   -232.153263   -232.153265
o  ethane              -79.830421   0.075238    -79.750770   0.027523   0.027525    -79.778293    -79.778295
o  isobutane          -158.458811   0.132380   -158.319804   0.034241   0.034252   -158.354046   -158.354056
o  methylaniline      -326.664901   0.142118   -326.514489   0.039668   0.039535   -326.554157   -326.554024
o  neopentane         -197.772980   0.160311   -197.604824   0.036952   0.036966   -197.641776   -197.641791
   *********************************************************************************************************
TOTAL CPU      0 days  2 hrs 29 mins 28 secs

```
The program will detect several different levels of theory and give a warning that any vibrational scaling factor other than 1 would be inappropriate in this case. Wildcard characters (`*`) can be used to represent any character or string of characters.

#### Example 8: Entropic Symmetry Correction
```python
python -m goodvibes examples/allene.out examples/benzene.out examples/ethane.out examples/isobutane.out examples/neopentane.out --ssymm

   Structure                    E        ZPE             H        T.S     T.qh-S          G(T)       qh-G(T)  Point Group
   **********************************************************************************************************************
o  allene             -116.569605   0.053913   -116.510916   0.026309   0.026312   -116.537225   -116.537228          D2d
o  benzene            -232.227201   0.101377   -232.120521   0.030396   0.030399   -232.150917   -232.150919          D6h
o  ethane              -79.830421   0.075238    -79.750770   0.025831   0.025833    -79.776601    -79.776603          D3d
o  isobutane          -158.458811   0.132380   -158.319804   0.033204   0.033214   -158.353008   -158.353019          C3v
o  neopentane         -197.772980   0.160311   -197.604824   0.034606   0.034620   -197.639430   -197.639444           Td
   *********************************************************************************************************************************************
```
GoodVibes will apply a symmetry correction described above to the entropy term of each molecule after determining the symmetry number. It is always a good idea to double-check that the point group GoodVibes returns is the correct group.

#### Example 9: Potential Energy Surface (PES) Comparison with Accessible Conformer Correction
```python
python -m goodvibes examples/gconf_ee_boltz/*.log --pes examples/gconf_ee_boltz/gconf_aminox_cat.yaml

   Structure                       E        ZPE             H        T.S     T.qh-S          G(T)       qh-G(T)
   ************************************************************************************************************
o  Aminoxylation_TS1_R   -879.405138   0.295352   -879.091374   0.063746   0.061481   -879.155120   -879.152855
o  Aminoxylation_TS2_S   -879.404445   0.295301   -879.090562   0.064366   0.061891   -879.154928   -879.152453
o  aminox_cat_conf212_S  -517.875165   0.200338   -517.662195   0.051817   0.049814   -517.714012   -517.712009
o  aminox_cat_conf280_R  -517.877308   0.200869   -517.664171   0.049996   0.048777   -517.714167   -517.712948
o  aminox_cat_conf65_S   -517.877161   0.200789   -517.664159   0.049790   0.048656   -517.713949   -517.712815
o  aminox_subs_conf713   -361.535757   0.095336   -361.433167   0.037824   0.037696   -361.470991   -361.470863
   ************************************************************************************************************

   Gconf correction requested to be applied to below relative values using quasi-harmonic Boltzmann factors

   RXN: Reaction (kcal/mol)       DE       DZPE            DH       T.DS    T.qh-DS         DG(T)      qh-DG(T)
   ************************************************************************************************************
o  Cat+Subs                     0.00       0.00          0.00       0.00       0.00          0.00          0.00
o  TS                           4.72      -0.46          3.53     -15.85     -16.37         19.39         19.90
   ************************************************************************************************************
```
A `.yaml` file is given to the `--pes` argument which specifies the reaction: `Catalyst + Substrate -> TS`. Because multiple conformers for the catalysts and transition states have been provided, GoodVibes will calculate a correction to the free energy based on the number of accessible conformations based on the Boltzmann-weighted energies of the conformers. To turn this correction off, `--nogconf` should be specified. An example `.yaml` file is shown below to show how these files should be formatted.

#### Example 10: Stereoselectivity and Boltzmann populations

```python
python -m goodvibes examples/gconf_ee_boltz/Aminoxylation_TS1_R.log examples/gconf_ee_boltz/Aminoxylation_TS2_S.log --boltz --ee "*_R*:*_S*"

   Structure                       E        ZPE             H        T.S     T.qh-S          G(T)       qh-G(T)  Boltz
   *******************************************************************************************************************
o  Aminoxylation_TS1_R   -879.405138   0.295352   -879.091374   0.063746   0.061481   -879.155120   -879.152855  0.605
o  Aminoxylation_TS2_S   -879.404445   0.295301   -879.090562   0.064366   0.061891   -879.154928   -879.152453  0.395
   *******************************************************************************************************************

   Selectivity            Excess (%)     Ratio (%)         Ratio     Major Iso           ddG
   *****************************************************************************************
o                              20.98         60:40         1.5:1             R          0.25
   *****************************************************************************************
```

The `--boltz` option will provide Boltzmann probabilities to the right of energy results under the `boltz` tab. With the `--ee` option, %ee, er and a reduced ratio are shown along with the dominant isomer and a calculated transition state energy value, ddG or ΔG‡.

#### Checks
A computational workflow can become less effective without consistency throughout the process. By using the `--check` option, GoodVibes will enforce a number of pass/fail checks on the input files given to make sure uniform options were used. Checks employed are:

###### Gaussian Output Checks
*   Same version of Gaussian used across all output files
*   Same solvation state/gas phase used across all output files
*   Same level of theory and basis set used
*   Same charge and multiplicity used
*   Check if standard concenctration of 1 atm was used in calculation
*   Check for duplicate structures or enantiomeric conformers based on E, H, qh_T.S and qh_G with a cutoff of 0.1 kcal/mol
*   Check for potential calculation error in linear molecules by Gaussian
*   Check for transition states (one imaginary frequency in output file)
*   Check if empirical dispersion is used and consistent across all output files

###### Single Point Calculation Checks
*   Same version and program used for all single point calculations
*   Same solvation model used across output files
*   Same level of theory used across all output files
*   Same charge and multiplicity used
*   Same geometry coordinates for SPC and associated geometry optimized and frequency calculation output file
*   Check if empirical dispersion is used and consistent across all output files

#### Symmetry
GoodVibes is able to detect a probable symmetry point group for each species and apply a symmetry correction to the entropy (S<sub>sym</sub>) by finding a molecule's internal symmetry number using atom connectivity, and external symmetry with the help of the external open source C program, "Brute Force Symmetry Analyzer" developed by S. Patchkovskii. These numbers are combined to give a symmetry number, n, and S<sub>sym</sub> is then defined as -Rln(n), which is applied to the GoodVibes calculated entropy.
*Note: this option may not function properly on some versions of Windows.*

#### File Naming Conventions
Some options (--pes, --graph, --spc, --ee, --media) require the calculation output files to be named in a certain way for GoodVibes to recognize them and perform extra calculations properly.

* **PES & Graph**

    PES and graphing file names need correlate with the file names specified in the `# SPECIES` block of the .yaml file (see below for .yaml formatting).

* **SPC**

    To link a frequency output file to a separately performed single point energy calculation file, the single point calculation file should have the same common root as the frequency file, with an additional underscore and descriptor at the end, such as `ethane.out` and `ethane_TZ.out` shown above, where `ethane_TZ.out` is the separate single point calculation file. When running GoodVibes in this case, the descriptor TZ should be passed as an argument, as `--spc TZ`.

* **Selectivity**

    To calculate enantiomeric excess, enantiomeric ratio, or diastereomeric ratios,
    file names should begin or end with a pattern identifier, such as `_R` and `_S`. The argument then passed to GoodVibes should be `--ee "*_R*:*_S*"`.

* **Media**

    To apply an entropic media correction to calculations performed on solvent molecules, the calculation output file should match the name passed in the media argument, for example, if performing the correction on water, the output file should be named `H2O.log` and the command line option should be `--media H2O`.

    GoodVibes will recognize the following solvent molecule names:

        meco2h / aceticacid, acetone, mecn / acetonitrile, benzene, 1buoh / 1butanol, 2buoh / 2butanol, 2butanone, tbuoh / tbutylalcohol, ccl4 / carbontetrachloride,
        phcl / chlorobenzene, chcl3 / chloroform, cyclohexane, 12dce  / 12dichloroethane, diethyleneglycol, et2o / diethylether, diglyme, dme / 12dimethoxyethane,
        dmf / dimethylformamide, dmso / dimethylsulfoxide, 14dioxane, etoh / ethanol, etoac / acoet / ethylacetate, ethyleneglycol, glycerin, hmpa / hexamethylphosphoramide,
        hmpt / hexamethylphosphoroustriamide, hexane, meoh / methanol, mtbe / methyltbutylether, ch2cl2 / methylenechloride, dcm / dichloromethane, nmp / nmethyl2pyrrolidinone,
        meno2 / nitromethane, pentane, 1propanol, 2propanol, pyridine, thf / tetrahydrofuran, toluene, et3n / triethylamine, h2o / water, oxylene, mxylene, pxylene

#### .yaml File Formatting
When using the --pes or --graph options in GoodVibes, a .yaml file must be provided to the program to specify qualities like reaction pathways, provided conformers, and other formatting options. The same .yaml may be used for both --pes and --graphing options. An example .yaml file from an external [Zenodo repository](https://doi.org/10.5281/zenodo.3662846) is shown below:

    --- # PES
        Ph: [Ph-Int1 + EtOH, Ph-TS1 + EtOH, Ph-Int2 + EtOH, Ph-TS2 + EtOH, Ph-Int3 + EtOH]
        Py: [Py-Int1 + EtOH, Py-TS1 + EtOH, Py-Int2 + EtOH, Py-TS2 + EtOH, Py-Int3 + EtOH]

    --- # SPECIES
        Ph-Int1     : Int_I_Ph*
        Ph-TS1      : TS_1_Ph*
        Ph-Int2     : Int_II_Ph*
        Ph-TS2      : TS_II_Ph*
        Ph-Int3     : Int_III_Ph*
        Py-Int1     : Int_I_Py*
        Py-TS1      : TS_1_Py*
        Py-Int2     : Int_II_Py*
        Py-TS2      : TS_II_Py*
        Py-Int3     : Int_III_Py*
        EtOH        : ethanol*
        
    --- # FORMAT
        dec :  2 
        legend : False
        color : black,#26a6a4
        pointlabel : False 
        gridlines: True
        show_conformers: True
        show_gconf: False
        dpi : 400
        title: Potential Energy Surface

options in the # FORMAT block are optional, but allow for stylistic choices to be employed, especially when graphing. All current options that can be specified for either --pes or --graph options are:

        dec : 1 or 2 (decimal points in output)
        legend : True or False (puts legend on graph)
        ylim : y_min,y_max (y axis limits on graph)
        color : Color (color of line for a reaction pathway, multiple pathways can have different colors i.e. color1,color2,color3 etc., this follows rules for matplotlib standard colors)
        pointlabel : True or False (labels relative energy on graph at point)
        xlabel : True or False (displays structure labels at pathway points on x axis)
        title : Title (title displayed on graph)
        gridlines: True or False (displays gridlines on graph)
        dpi : number (specify dpi (dots per inch) of an image, will automatically save output image at specified dpi)
        show_conformers : True or False (displays a point for each conformer of a certain compound at its relative energy on graph)
        show_gconf : True or False (displays the effect of multiple accessible conformers correction if applied)

#### Tips and Troubleshooting
*	The python file doesn’t need to be in the same folder as the Gaussian files. Just set the location of GoodVibes.py in the `$PATH` variable of your system (this is not necessary if installed with pip or conda)
*	It is possible to run on any number of files at once using wildcards to specify all of the Gaussian files in a directory (specify `*.out` or `*.log`)
*   File names not in the form of filename.log or filename.out are not read, however more file extensions can be added with the option `--custom_ext`
*	The script will not work if terse output was requested in the Gaussian job
*  Problems may occur with Restart Gaussian jobs due to missing information in the output file.
*  HF, DFT, MP2, semi-empirical, time dependent (TD) DFT and HF, ONIOM, and G4 calculations from Gaussian are also supported.


#### References for the underlying theory
1. Ribeiro, R. F.; Marenich, A. V.; Cramer, C. J.; Truhlar, D. G. *J. Phys. Chem. B* **2011**, *115*, 14556-14562 [**DOI:** 10.1021/jp205508z](http://dx.doi.org/10.1021/jp205508z)
2. Grimme, S. *Chem. Eur. J.* **2012**, *18*, 9955–9964 [**DOI:** 10.1021/jp509921r](http://dx.doi.org/10.1002/chem.201200497)
3. Li, Y.; Gomes, J.; Sharada, S. M.; Bell, A. T.; Head-Gordon, M. *J. Phys. Chem. C* **2015**, *119*, 1840-1850 [**DOI:** 10.1002/chem.201200497](http://dx.doi.org/10.1021/jp509921r)
4. Alecu, I. M.; Zheng, J.; Zhao, Y.; Truhlar, D. G.; *J. Chem. Theory Comput.* **2010**, *6*, 2872-2887 [**DOI:** 10.1021/ct100326h](http://dx.doi.org/10.1021/ct100326h)
5. Mammen, M.; Shakhnovich, E. I.; Deutch, J. M.; Whitesides, G. M. *J. Org. Chem.* **1998**, *63*, 3821-3830 [**DOI:** 10.1021/jo970944f](http://dx.doi.org/10.1021/jo970944f)

---
#### License: 

GoodVibes2 is freely available under an [MIT](https://opensource.org/licenses/MIT) License
