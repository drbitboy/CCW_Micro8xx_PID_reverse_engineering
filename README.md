# CCW_Micro8xx_PID_reverse_engineering

Reverse-engineering the PID instruction used by Allen-Bradley Micro8xx PLCs and Connected Components Workbench (CCW)

## Summary

The [PID] instruction in CCW for Micro8xx PIC uses the Dependent Gains form of a PID algorithm.  However, the [PID] instruction does not work as advertised.  Specifically, the tuning parameter members of the PID_GAINS object used by [PID] are not all consistent the with the canonical/conventional meanings of their names.  If pidgains is a tag in CCW of data type PID_GAINS, then its members have the following meanings:

* pidgains.Kc - Controller gain, units of dCV/dError
* pidgains.Ti - Integral time, units of seconds (s)
* pidgains.Td - Factor for calculating a first-order filter time constant, units of seconds-squared (s**2)
* pidgains.FC - Derivative time, units of seconds (s)
* the product (pidgains.Td * pidgains.FC) is the CV output filter time constant

Note specifically that

* the .FC member of PID_GAINS is not the filter constant that the CCW documentation claims it to be;
* the .Td member of PID_GAINS is not the derivative time that the CCW documentation claims it to be.

## Details

The program under the CCWcode/ subdirectory of this repository contains the Connected Components Workbench program used to perform this work.  In summary, that program runs a PID instruction via a STI (Selectable Timed Interrupt).  The user of CCW runs experiments as follows:

* assign the tuning parameters in the PID_GAINS object that is the [Gains] input to the PID instructions,
* manually change the Process Variable (PV) and/or SetPoint (SP) inputs to the PID instruction using the CCW interface,
* observe and/or trend the Controlled Variable (CV) output of the PID instruction.

There is no feedback from CV to PV; PV only changes when the user changes it manually; it could be said that the user making those changes is the process.  The most typical user change is a step change in PV.

The images under the images subdirectory of this repository summarize the experiments used to reverse engineer the CCW PID instruction and determine its behavior.  Each image comprises

* a time trend plot of data from an experiment,
* a display of the tuning parameters in tag pidgains
* annotation of CV changes during the experiment

### Reverse engineering the proportional action of the CCW PID

These two images show trends of PV and CV for proportional-only experiments on the PID.  The integral action was disabled by assigning a large value toe .Ti; the derivative action was disabled by assigning 0 to the values of .Td and .FC.

The two cases demonstrate that the .Kc member of the PID_GAINS object is controller gain, in units of dCV/dPV..

![](https://github.com/drbitboy/CCW_Micro8xx_PID_reverse_engineering/raw/main/images/ccw_pid_ProportionalOnly_Kc0.10.png)

![](https://github.com/drbitboy/CCW_Micro8xx_PID_reverse_engineering/raw/main/images/ccw_pid_ProportionalOnly_Kc0.15.png)

### Reverse engineering the integral action of the CCW PID

These two images show trends of PV and CV for proportional-integral experiments on the PID.  The derivative action was disabled by assigning 0 to the values of .Td and .FC.

The two cases demonstrate that the .Ti member of the PID_GAINS object is integral time, and that the CCW PID uses the "Dependent Gains" form of the PID formula cf. https://literature.rockwellautomation.com/idc/groups/literature/documents/wp/logix-wp008_-en-p.pdf.

![](https://github.com/drbitboy/CCW_Micro8xx_PID_reverse_engineering/raw/main/images/ccw_pid_ProportionalIntegral_Kc0.10_Ti05.0.png)

![](https://github.com/drbitboy/CCW_Micro8xx_PID_reverse_engineering/raw/main/images/ccw_pid_ProportionalIntegral_Kc0.10_Ti10.0.png)

### Reverse engineering the derivative action of the CCW PID

These images show trends of PV and CV for proportional-derivative experiments on the PID.  The integral action was disabled by assigning a large value toe .Ti.

The first two images demonstrate that .FC is derivative time; .Td is set to 0.001 which effectively disables the CV output filter.  Both .Kc and .FC are varied, and the results further confirm that the CCW PID uses the Dependent Gains formula.

![](https://github.com/drbitboy/CCW_Micro8xx_PID_reverse_engineering/raw/main/images/ccw_pid_ProportionalDerivative_Kc0.10_Td0.001_FC1.0.png)

![](https://github.com/drbitboy/CCW_Micro8xx_PID_reverse_engineering/raw/main/images/ccw_pid_ProportionalDerivative_Kc0.15_Td0.001_FC2.0.png)

The last four images demonstrat that the output filter time constant is

* directly proportional to .Td
* inversely proportional to .FC

![](https://github.com/drbitboy/CCW_Micro8xx_PID_reverse_engineering/raw/main/images/ccw_pid_ProportionalDerivative_Kc0.10_Td1.0_FC1.0.png)

![](https://github.com/drbitboy/CCW_Micro8xx_PID_reverse_engineering/raw/main/images/ccw_pid_ProportionalDerivative_Kc0.10_Td2.0_FC1.0.png)

![](https://github.com/drbitboy/CCW_Micro8xx_PID_reverse_engineering/raw/main/images/ccw_pid_ProportionalDerivative_Kc0.05_Td2.0_FC2.0.png)

![](https://github.com/drbitboy/CCW_Micro8xx_PID_reverse_engineering/raw/main/images/ccw_pid_ProportionalDerivative_Kc0.025_Td2.67_FC4.0.png)
