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

The images under the images subdirectory of this repository summarize the experiments used to 
