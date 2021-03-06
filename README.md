**Note: This is a fork of a [fork](https://bitbucket.org/clemi/trac_ik/src/devel/) of the [original](https://bitbucket.org/traclabs/trac_ik/src/master/) trac_ik repo.**

**If you're using Python 2.7 on Ubuntu 18.04, skip the instructions below and install using ```sudo apt-get install ros-melodic-trac-ik```**

# How does this branch/repository differ from the trac_ik one?

* `trac_ik_lib` cannot load a URDF from the ROS parameter server (but in turn does not instantiate a node handle)
* `trac_ik_python` cannot load a URDF from the ROS parameter server (but in turn does not depend on `rospy`)
* `trac_ik_python` does not depend on `tf_conversions` (which would create an unresolvable dependency)
* there's an installation instruction for Python 3 (see below)

For a detailed diff summary look [here](https://bitbucket.org/clemi/trac_ik/branches/compare/devel..#diff).

# How to install trac_ik_python for Python3 (and with minimal ROS dependencies)?

Given a fresh Ubuntu 18.04 minimal installation do the following steps.

### (0) General requirements:
###
```
sudo apt-get install git build-essential cmake python3-pip checkinstall
```

### (1) Install catkin (and catkin_tools):
###    
```
sudo apt-get install python3-empy python3-nose libgtest-dev 
pip3 install catkin_pkg

git clone https://github.com/ros/catkin.git
cd catkin
git checkout <version>-devel # e.g. kinetic-devel for ubuntu 16.04, melodic-devel for 18.04, noetic-devel for 20.04
mkdir build && cd build
cmake -DPYTHON_VERSION=3 -DPYTHON_EXECUTABLE=/usr/bin/python3  -DCMAKE_INSTALL_PREFIX=/usr ..
make
sudo checkinstall --pkgname=catkin-github
# Reply [y] and set description, e.g.: "catkin from github"
# Answer the questions ("Do you want me to list them?") with [n]/yes/[n]/[y]
# (it's the default answer in 3 out of the 4 questions)

pip3 install catkin_tools
# NOTE: if you use the above, you may run into a syntax error when running catkin init below because the version at the time of writing (0.6.x) relies on trollius
# which is written in python2. Version 0.7 will fix this, but in the mean time, you can install from source using:
# cd ~ # or wherever you want to store it
# git clone https://github.com/catkin/catkin_tools.git
# cd catkin_tools
# pip3 install .
# cd <directory where you cloned catkin>/catkin/build
```

### (2) Install trac_ik:
###
```
sudo apt-get install ros-cmake-modules libkdl-parser-dev libeigen3-dev libnlopt-dev liborocos-kdl-dev liburdfdom-dev swig

mkdir catkin_ws/src -p && cd catkin_ws/src
git clone -b devel https://clemi@bitbucket.org/clemi/trac_ik.git
cd ..
catkin init
catkin config -DPYTHON_EXECUTABLE=/usr/bin/python3 -DPYTHON_VERSION=3 -DCMAKE_BUILD_TYPE=Release --merge-devel --blacklist trac_ik trac_ik_examples
# for conda environment replace /usr/bin/python3 with $(which python)
# if you have a conventional ROS installation, you can ignore the '--extend  /usr' step
trac_ik_kinematics_plugin --extend /usr
catkin build
```

### (3) Test installation:
###
```
source devel/setup.bash

# Get an example URDF file
wget https://raw.githubusercontent.com/ros-planning/moveit_resources/master/panda_description/urdf/panda.urdf

# Test inverse kinematics
python3 -c "from trac_ik_python.trac_ik import IK; urdfstring = ''.join(open('panda.urdf', 'r').readlines()); ik = IK('panda_link0', 'panda_hand', urdf_string=urdfstring); print(ik.get_ik([0.0]*7, 0.5, 0.5, 0.5, 0, 0, 0, 1))"

# The std output will most likely be one of the following 7-tuple:
# [Panda has a redundant DoF so there can be multiple joint configs for a single Cartesian pose]
# (0.14720490730995048, 0.8472134373227671, 0.8598701236671977, -1.4895870318659121, 2.4598493739297553, 1.1226250704200282, -0.35609815106501336)
# (1.5642459790162786, 1.033826419580488, -1.137034628893683, -1.4641733616752757, -2.1826248503279584, 1.2612257933132356, 0.4657470590149234)
```


# ============================================================================
#   
#   
# trac_ik
The ROS packages in this repository were created to provide an alternative
Inverse Kinematics solver to the popular inverse Jacobian methods in KDL.
Specifically, KDL's convergence algorithms are based on Newton's method, which
does not work well in the presence of joint limits --- common for many robotic
platforms.  TRAC-IK concurrently runs two IK implementations.  One is a simple
extension to KDL's Newton-based convergence algorithm that detects and
mitigates local minima due to joint limits by random jumps.  The second is an
SQP (Sequential Quadratic Programming) nonlinear optimization approach which
uses quasi-Newton methods that better handle joint limits.  By default, the IK
search returns immediately when either of these algorithms converges to an
answer.  Secondary constraints of distance and manipulability are also provided 
in order to receive back the "best" IK solution.

###This repo contains 5 ROS packages:###

- trac\_ik is a metapackage with build and complete [Changelog](https://bitbucket.org/traclabs/trac_ik/src/HEAD/trac_ik/CHANGELOG.rst) info.

- trac\_ik\_examples contains examples on how to use the standalone TRAC-IK library.

- [trac\_ik\_lib](https://bitbucket.org/traclabs/trac_ik/src/HEAD/trac_ik_lib), the TRAC-IK kinematics code,
builds a .so library that can be used as a drop in replacement for KDL's IK
functions for KDL chains. Details for use are in trac\_ik\_lib/README.md.

- [trac\_ik\_kinematics\_plugin](https://bitbucket.org/traclabs/trac_ik/src/HEAD/trac_ik_kinematics_plugin) builds a [MoveIt! plugin](http://moveit.ros.org/documentation/concepts/#kinematics) that can
replace the default KDL plugin for MoveIt! with TRAC-IK for use in planning.
Details for use are in trac\_ik\_kinematics\_plugin/README.md. (Note prior to v1.1.2, the plugin was not thread safe.)

- [trac\_ik\_python](https://bitbucket.org/traclabs/trac_ik/src/HEAD/trac_ik_python), SWIG based python wrapper to use TRAC-IK. Details for use are in trac\_ik\_python/README.md.


###As of v1.4.5, this package is part of the ROS Kinetic binaries: `sudo apt-get install ros-kinetic-trac-ik` (or indigo or jade).  Starting with v1.4.8, this has been released for ROS Lunar as well. Melodic packages have been released with 1.5.0.


###A detailed writeup on TRAC-IK can be found here:###

[Humanoids-2015](https://personal.traclabs.com/~pbeeson/publications/b2hd-Beeson-humanoids-15.html) (reported results are from v1.0.0 of TRAC-IK, see below for newer results).

###Some sample results are below: 

_Orocos' **KDL**_ (inverse Jacobian w/ joint limits), _**KDL-RR**_ (our fixes to KDL joint limit handling), and _**TRAC-IK**_ (our concurrent inverse Jacobian and non-linear optimization solver; Speed mode) are compared below.

IK success and average speed as of TRAC-IK tag v1.5.1.  All results are from 10,000 randomly generated, reachable joint configurations.  Full 3D pose IK was requested at 1e-5 Cartesian error for x,y,z,roll,pitch,yaw with a maximum solve time of 5 ms.  All IK queries are seeded from the chain's "nominal" pose midway between joint limits.

**Note on success**: Neither KDL nor TRAC-IK uses any mesh information to determine if _valid_ IK solutions result in self-collisions.  IK solutions deal with link distances and joint ranges, and remain agnostic about self-collisions due to volumes.  Expected future enhancements to TRAC-IK that search for multiple solutions may also include the ability to throw out solutions that result in self collisions (provided the URDF has valid geometry information); however, this is currently not the behaviour of any generic IK solver examined to date.

**Note on timings**: The timings provided include both successful and unsuccessful runs.  When an IK solution is not found, the numerical IK solver implementations will run for the full timeout requested, searching for an answer; thus for robot chains where KDL fails much of the time (e.g., Jaco-2), the KDL times are skewed towards the user requested timeout value (here 5 ms).  

Chain | DOFs | Orocos' _KDL_ solve rate | Orocos' _KDL_ Avg Time | _KDL-RR_ solve rate | _KDL-RR_ Avg Time | _TRAC-IK_ solve rate | _TRAC-IK_ Avg Time
- | - | - | - | - | - | - | -
ABB IRB120 | 6 | **39.41%** | 3.08ms | **98.51%** | 0.33ms | **99.96%** | 0.24ms
ABB Yumi 'single arm' | 7 | **77.35%** | 1.43ms | **91.31%** | 0.87ms | **99.70%** | 0.42ms
Atlas 2013 arm | 6 | **75.54%** | 1.32ms | **97.24%** | 0.34ms | **99.99%** | 0.20ms
Atlas 2015 arm | 7 | **76.22%** | 1.44ms | **94.12%** | 0.71ms | **99.80%** | 0.32ms
Baxter arm | 7 | **61.43%** | 2.15ms | **90.78%** | 0.91ms | **99.83%** | 0.37ms
Denso VS-068 | 6 | **27.95%** | 3.67ms | **98.32%** | 0.35ms | **99.96%** | 0.26ms
Fanuc M-430iA/2F | 5 | **21.08%** | 3.98ms | **88.69%** | 0.84ms | **99.93%** | 0.36ms
Fetch arm | 7 | **93.28%** | 0.65ms | **94.72%** | 0.63ms | **99.98%** | 0.24ms
Franka Emika Panda | 7 | **62.02%** | 2.11ms | **93.21%** | 0.79ms | **99.88%** | 0.37ms
Jaco2 | 6 | **26.25%** | 3.77ms | **97.85%** | 0.47ms | **99.92%** | 0.35ms
KUKA LBR iiwa 14 R820 | 7 | **38.09%** | 3.31ms | **95.15%** | 0.64ms | **99.92%** | 0.28ms
KUKA LWR 4+ | 7 | **68.22%** | 1.82ms | **96.26%** | 0.53ms | **99.98%** | 0.23ms
Motoman CSDA10F 'torso/1-arm' | 8 | **53.58%** | 2.73ms | **96.08%** | 0.60ms | **99.96%** | 0.32ms
Motoman MH180 | 6 | **68.46%** | 1.65ms | **99.39%** | 0.22ms | **99.99%** | 0.18ms
NASA Robonaut2 arm | 7 | **86.89%** | 0.96ms | **95.23%** | 0.64ms | **99.85%** | 0.30ms
NASA Robonaut2 'grasping leg' | 7 | **61.70%** | 2.21ms | **88.77%** | 1.00ms | **99.91%** | 0.40ms
NASA Robonaut2 'leg' + waist + arm | 15 | **98.42%** | 0.67ms | **98.58%** | 0.66ms | **99.83%** | 0.57ms
NASA Robosimian arm | 7 | **62.10%** | 2.33ms | **99.88%** | 0.28ms | **99.97%** | 0.30ms
NASA Valkyrie arm | 7 | **45.78%** | 2.95ms | **92.34%** | 1.11ms | **99.90%** | 0.37ms
PR2 arm | 7 | **84.70%** | 1.26ms | **88.82%** | 1.15ms | **99.96%** | 0.31ms
Schunk LWA4D | 7 | **68.57%** | 1.79ms | **97.26%** | 0.43ms | **100.00%** | 0.23ms
TRACLabs modular arm | 7 | **79.59%** | 1.28ms | **96.11%** | 0.54ms | **99.99%** | 0.27ms
Universal UR3 | 6 | **17.11%** | 4.18ms | **89.08%** | 0.77ms | **98.60%** | 0.45ms
UR5 | 6 | **16.52%** | 4.21ms | **88.58%** | 0.74ms | **99.17%** | 0.37ms
UR10 | 6 | **14.90%** | 4.29ms | **88.63%** | 0.74ms | **99.33%** | 0.36ms

Feel free to [email Patrick](mailto:pbeeson@traclabs.com) if there is a robot chain that you would like to see added above.
