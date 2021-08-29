# CCW_Micro8xx_PID_reverse_engineering

Reverse-engineering the PID instruction used by Allen-Bradley Micro8xx PLCs and Connected Components Workbench (CCW)

## Summary

The [PID] instruction in CCW for Micro8xx PIC uses the Dependent Gains form of a PID algorithm.  However, the [PID] instruction does not work as advertised.  Specifically, the tuning parameter members of the PID_GAINS object used by [PID] are not all consistent the with the canonical/conventional meanings of their names.  If pidgains is a tag in CCW of data type PID_GAINS, then its members have the following meanings:

    pidgains.Kc - Controller gain, units of dCV/dError
    pidgains.Ti - Integral time, units of seconds (s)
    pidgains.Td - Factor for calculating a first-order filter time constant, units of seconds-squared (s**2)
    pidgains.FC - Derivative time, units of seconds (s)

Note specifically that

* the .FC member of PID_GAINS is not the filter constant that the CCW documentaiont claims it to be;
* the .Td member of PID_GAINS is not the derivative time that the CCW documentaiont claims it to be.

## Details

The program under the CCWcode/ subdirectory of this repository contains the Connected Components Workbench program used to perform this work.  In summary, that program runs a PID instruction via a STI (Selectable Timed Interrupt).  The user running CCW assigns the tuning parameters in the PID_GAINS object that is the [Gains] input to the PID instructions, then manually changes the Process Variable (PV) and SetPoint (SP) inputs to the PID instruction using the CCW interface, and finally observes and/or trends the Controlled Variable (CV) output of the PID instruction.  There is no feedback from CV to PV; PV only changes when the user does so manually.
