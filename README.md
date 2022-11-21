# OpenFASOC
Fully Open-Source Autonomous SoC Synthesis using Customizable Cell-Based Synthesizable Analog Circuits
OpenFASoC is a project focused on automated analog generation from user specification to GDSII with fully open-sourced tools. It is led by a team of researchers at the University of Michigan and is inspired from FASoC which sits on proprietary software.
The tool is comprised of analog and mixed-signal circuit generators, which automatically create a physical design based on user specifications.
See more about this at this [Site](https://fasoc.engin.umich.edu/)

# FREQUENCY TO DIGITAL CONVERTER (COUNTER)
## Inputs
1) Reference clock
2) Clock generated from SLC(Split Level Controller)
3) Reset for Resettng the counter to zero

## Outputs
1) Dout
2) 24 bit output

## Working
<p align="center">
  <img src="/images/temp_sens_io.png">
</p><br>

# Prerequisites
****************

Install all the prerequisites using `dependencies.sh` script provided in the home location of this project (where this readme.rst file is found). Supports CentOS7 and Ubuntu20.


(Or) Please install the following tools by building the tools manually from their code base with the recommended commit ids for a stable functioning of the flow:

  1. `Magic <https://github.com/RTimothyEdwards/magic>`_ (version:8.3.334)

  2. `Netgen <https://github.com/RTimothyEdwards/netgen>`_ (version:1.5.240)

  3. `Klayout <https://github.com/KLayout/klayout>`_ (version:0.27.10-1)

      - Please use this command to build preferably: `./build.sh -option '-j8' -noruby -without-qt-multimedia -without-qt-xml -without-qt-svg`


  4. `Yosys <https://github.com/The-OpenROAD-Project/yosys>`_ (version:0.22+70)

  5. `OpenROAD <https://github.com/The-OpenROAD-Project/OpenROAD>`_ (version:2.0_5525)

  6. `Open_pdks <https://github.com/RTimothyEdwards/open_pdks>`_ (version:1.0.353)

   - open_pdks is required to run drc/lvs check and the simulations
   - After open_pdks is installed, please update the **open_pdks** key in `common/platform_config.json` with the installed path, down to the sky130A folder


# Installation
## 1. OpenFASOC:
The command used to install OpenFASOC are 
```
git clone https://github.com/idea-fasoc/openfasoc

cd openfasoc

pip install -r requirements.txt
```
For the complete steps of installing OpenFASOC, refer Manual Installation from [here](https://github.com/idea-fasoc/OpenFASOC/blob/main/docs/source/getting-started.rst). 

## 2. OpenROAD: 
OpenROAD is an integrated chip physical design tool that takes a design from synthesized Verilog to routed layout. OpenROAD uses the OpenDB database and OpenSTA for static timing analysis. Documentation is also available [here](https://openroad.readthedocs.io/en/latest/main/README.html).


The commands to install OpenROAD are,
```
git clone --recursive https://github.com/The-OpenROAD-Project/OpenROAD.git

cd OpenROAD

./etc/DependencyInstaller.sh

./etc/DependencyInstaller.sh -run

./etc/DependencyInstaller.sh -dev

mkdir build

cd build

cmake ..

make

```

## 3. Klayout
Downlaod the latest version of the Klayout from [here](https://www.klayout.de/build.html). Install the following dependencies: qt5-default, qttools5-dev, libqt5xmlpatterns5-dev, qtmultimedia5-dev, libqt5multimediawidgets5 and libqt5svg5-dev.
```
sudo apt-get install -y libqt5widgets5

sudo dpkg -i klayout_0.27.11-1_amd64.deb

```

## 4. Netgen
To install Netgen, 
```
sudo add-apt-repository ppa:ngsolve/ngsolve

sudo apt-get update

sudo apt-get install ngsolve

```

## 5. Yosys
The software used to run gate level synthesis is Yosys. Yosys is a framework for Verilog RTL synthesis. It currently has extensive Verilog-2005 support and provides a basic set of synthesis algorithms for various application domains. Yosys can be adapted to perform any synthesis job by combining the existing passes (algorithms) using synthesis scripts and adding additional passes as needed by extending the Yosys C++ code base.


To install yosys, install the prerequisites using the following command 
 ```
 sudo apt-get install build-essential clang bison flex \
	libreadline-dev gawk tcl-dev libffi-dev git \
	graphviz xdot pkg-config python3 libboost-system-dev \
	libboost-python-dev libboost-filesystem-dev zlib1g-dev
```
To install latest Version of Yosys, 
```
git clone https://github.com/YosysHQ/yosys.git

make

sudo make install 

make test

```

## 6. Magic
Run following commands one by one to fulfill the system requirement.
#### Prerequisites for magic
```
sudo apt-get install m4

sudo apt-get install tcsh

sudo apt-get install csh

sudo apt-get install libx11-dev

sudo apt-get install tcl-dev tk-dev

sudo apt-get install libcairo2-dev

sudo apt-get install mesa-common-dev libglu1-mesa-dev

sudo apt-get install libncurses-dev
```
#### Install magic
```
git clone https://github.com/RTimothyEdwards/magic

cd magic/

./configure

sudo make

sudo make install
```
type `magic` terminal to check whether it installed succesfully or not. Type `exit` to exit magic.

# Temperature Sensor Generator

An all-digital temperature sensor, that relies on a new subthreshold oscillator (achieved using the auxiliary cell “Header Cell“) for realizing synthesizable thermal sensors.

The way that works is we have a subthreshold current that has an exponential dependency on the temperature, the frequency generated from the subthreshold ring oscillator is also dependent on temperature. So we can sense the temperature by comparing the difference between the clock frequency generated from a reference oscillator and the clock frequency from the proposed frequency generator.

## Temperature Sensor Description

**User Specs**
* Temperature sensing range: -20⁰C – 125⁰C
* Frequency range of operation: 100Hz – 10MHz

**Block Architecture**
<p align="center">
  <img src="/images/temp_sens_io.png">
</p><br>

 _Inputs_
 *  CLK_REF: System clock taken from input.
 *  RESET_COUNTERn: Input signal to reset the module initial state.

 * SEL_CONV_TIME: Four bit input used to select how many times the 1 bit of the output DOUT is fractionated (0-16).

_Outputs_
 *  DOUT:  The output voltage whose frequency is dependent on temperature.
 *  DONE: Validity signal for DOUT
 
Circuit
-------
This generator creates a compact mixed-signal temperature sensor based on the topology from this [paper](https://ieeexplore.ieee.org/document/9816083).

It consists of a ring oscillator whose frequency is controlled by the voltage drop over a MOSFET operating in subthreshold regime, where its dependency on temperature is exponential.

![tempsense_ckt](https://user-images.githubusercontent.com/110079631/199317479-67f157c5-6934-470b-8552-5451b1361b9c.png)

  Block diagram of the temperature sensor’s circuit

The physical implementation of the analog blocks in the circuit is done using two manually designed standard cells:
1. HEADER cell, containing the transistors in subthreshold operation;
2. SLC cell, containing the Split-Control Level Converter.

The gds and lef files of HEADER and SLC cells are pre-created before the start of the Generator flow.

# OpenFASOC Flow

The generator must first parse the user’s requirements into a high-level circuit description or verilog. User input parsing is implemented by reading from a JSON spec file directly in the temp-sense-gen repository. The JSON allows for specifying power, area, maximum error (temperature result accuracy),
an optimization option (to choose which option to prioritize), and an operating temperature range (minimum and maximum operating temperature values).
The operating temperature range and optimization must be specified, but other items can be left blank. 


The generator uses this model file to automatically determine the number of headers and inverters, among other necessary modifications that can be made to meet spec. The generator references the model file in an iterative process until either meeting spec or failing. A verilog description is then
produced by substituting specifics into several template verilog files.

## Case Study: Temp_Sensor

### 1. Verilog Files geneartion and dir? User specs, iterative approach and generated verilog files.

In the test.json file, only the temperature can be modified and the range of temperature should always be between –20C to 100C. Based on the operating temperature range, generator calculates the number of header and inverters to minimize the error. 

To run the verilog generation, run the following command:
```
make sky130hd_temp_verilog
```

The generator references the model file in an iterative process until either meeting specifications or failing.


Here, using the generic template, extra blocks of counter, TEMP_ANALOG_hv.nl.v, TEMP_ANALOG_lv.nl.v
are created in the src folder.

## Contributors

- **Pankaj Agrawal**
- **Kunal Ghosh**
- **Tejas B N**
- **Ajay**
- **Aditya Singh**


## Acknowledgments


- Kunal Ghosh, Director, VSD Corp. Pvt. Ltd.
- Madhav Rao, Associate Professor, IIIT Bangalore
- V N Muralidhara, Associate Professor, IIIT Bangalore


## Contact Information

- Pankaj Agrawal, Postgraduate Student, International Institute of Information Technology, Bangalore  1811pankajagrawal@gmail.com
- Kunal Ghosh, Director, VSD Corp. Pvt. Ltd. kunalghosh@gmail.com
- Tejas B N, Postgraduate Student, International Institute of Information Technology, Bangalore  bntejas@gmail.com
- Aditya Singh, Postgraduate Student, International Institute of Information Technology, Bangalore
- Ajay, Postgraduate Student, International Institute of Information Technology, Bangalore
