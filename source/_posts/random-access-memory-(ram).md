title: Random Access Memory (RAM)
date: 2013-03-04 11:02:40
tags: hardware
---

RAM has address lines on it, which are connected to a series of transistors and capacitors. The purpose of the address line is to turn its transistors on or off. When the transistor is turned on, it is connected to its capacitor and dateline. 

Data lines are connected to the CPU and are used to to read or write data. 

To write data, the CPU sends an electrical charge to an address line to open all the transistors connected to it. It then sends a pulse of electricity through the relevant data lines to charge the capacitors. In this way, though all the transistors are closed, only certain capacitors are charged. In this example, each address line has eight transistors and capacitors (8 bits) which store a byte of data. 

To read data, the CPU sends out a charge to an address line to open all the transistors connect to it. The capacitors then discharge into the data lines through the transistors. In this way, the CPU is able to read the values of the bits; it will interpret a discharged charge as the binary value of 1 and no discharge as 0. 

