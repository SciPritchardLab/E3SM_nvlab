# E3SM MMF-NN-Emulator Version

**Contact**: Zeyuan Hu (zeyuan_hu@fas.harvard.com)


1. [Quick Start](#1-quick-start)
2. [E3SM MMF-NN-Emulator Configuration Options](#2-e3sm-mmf-nn-emulator-configuration-options)
   1. [Basic Running Modes](#21-basic-running-modes)
   2. [Partial Coupling Running Modes](#22-partical-coupling-running-modes)
3. [NN-Emulator Namelists](#3-nn-emulator-namelists)
4. [How to Test a Customized NN Emulator](#4-how-to-test-a-customized-nn-emulator)

---

## 1. Quick Start

---

## 2. E3SM MMF-NN-Emulator configuration options

### 2.1 Basic Running Modes

Currently, there are three CPP flags that can control the running modes: `MMF_NN_EMULATOR`, `MMF_ML_TRAINING`. Turning them on/off will result in different model configurations as described below:

- **MMF Mode**: Turn off all the flags. Just set `user_cpp=''` in the job script (see next section).
- **MMF Mode + Saving Training Input/Output**: Turn on the `MMF_ML_TRAINING` flag. Set `user_cpp = '-DMMF_ML_TRAINING'`. This will save the necessary data every time step which we can use to construct training data for neural nets.
- **NN Mode**: Turn on `MMF_NN_EMULATOR` by set `user_cpp = '-DMMF_NN_EMULATOR'`. `MMF_NN_EMULATOR` will use NN to replace CRM calculations.

Note currently the NN calculation requires previous two steps’ physics tendency (either the MMF tendency or the NN tendency) and the large-scale forcings (i.e., the tendencies from dynamical core plus ac physics; ac physics primarily includes vertical diffusion and convective gravity wave drag parameterizations) as input features. So at the beginning of calculation, we must first run the MMF for a few steps and then we can switch to a fully NN coupling run. The MMF spinup steps can be set by the namelist variable `cb_spinup_step` (should be at least 3 since we need to have two previous steps tendencies as input).

### 2.2 Partial Coupling Running Modes

This partial coupling option is initially developed by Sungduk Yu (sungduk.yu@intel.com).

- **NN Mode + Partial Coupling**: `user_cpp = '-DMMF_NN_EMULATOR'` and set namelist `cb_partial_coupling = '.true.'`. In this mode, the E3SM will do both NN and MMF calculation. You can choose to overwrite a customized set of MMF output variables by the NN outputs or a mixture of NN and MMF output (e.g. dT/dt = a * dT/dt_nn + (1-a) * dT/dt_mmf) to couple back to the rest of E3SM. In this mode, you need to set `cb_partial_coupling_vars` which will decide which variables will use NN outputs or a mixture of NN/MMF outputs. Variables not included in the `cb_partial_coupling_vars` will use the MMF output. `cb_do_ramp`, `cb_ramp_option`, and `cb_ramp_factor` can specify the customized mixture ratio of NN outputs and MMF outputs. If `cb_do_ramp=False` then we will use the `a=1` i.e. use the NN outputs to replace MMF output. If `cb_do_ramp=True` you can set `a<1` or let it depend on time. We have a few options of time schedules by setting `cb_ramp_option` and `cb_ramp_factor` see the namelist section for more details.

- **NN Mode + Partial Coupling + Saving Training Input/Output**: Similar to the previous mode but also add `-DMMF_ML_TRAINING` in the `user_cpp`. This will output input/output data which you can use for generating training data. In this mode, it will generate two sets of data. One is the GCM input/output for the (with filename containing ‘.mli.’ and ‘.mlo.’) the other is the GCM-MMF input/output (with filename containing ‘.mlis.’ and ‘.mlos.’). The difference is for the saved previous steps’ physics tendencies and advective tendencies and outputs. For the previous physics tendencies, GCM input is the prediction from the mixture of NN+MMF (may be either pure NN prediction, pure MMF prediction, or a mixture of NN + MMF prediction as specified by `cb_partial_coupling` and other related namelist as we described in the previous mode). The GCM-MMF input is the prediction from just the MMF. For the previous advective tendencies, the GCM input will be the tendencies from the `ac+dycore` while the GCM-MMF input will be the `ac+dycore+(GCM state - MMF state)/dt`. The last term is evaluated at the previous step after calling the NN/MMF module. It is nonzero only when the GCM states are not updated fully by the MMF outputs so that the GCM state is not synchronized with the CRM state. For the output, the GCM output contains the state updated by the partial coupling while the GCM-MMF output contains the state values assuming only use the MMF predictions to update. If you want to use this mode to generate training data likely the GCM-MMF output is what you want as the target because the GCM-MMF output is the pure MMF prediction.

Note: In this mode, you must set `cb_partial_coupling = '.true.'` otherwise the saved `SOLIN/COSZR` will be wrong.

---

## 3. NN-Emulator namelists

---

## 4. How to test a customized NN Emulator