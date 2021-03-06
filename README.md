# CarND-Controls-MPC
Self-Driving Car Engineer Nanodegree Program

![figure missed](/images/mpc-image.png)

------


## Introduction

This is the fifth project of the second term of Udacity Self-Driving Car Engineer Nanodegree Program. In this program, I used Model Predicted Control (MPC) to control the car driving along the road.

#### Model Predictive Control
The following figure shows the basic structure of MPC.

![figure missed](/images/mpc-model.png "Basic Structure of MPC")

Basic Structure of MPC [[Ref](https://www.mdpi.com/2075-1702/5/1/6/pdf)]

In the project, I used a vehicle kinetic model for prediction. Its state consisted of the following parameters: coordinates, `x`, `y`; yaw angle, `psi`, velocity, `v`; cross-track error, `cte`; psi error, `epsi`. The actuator output includes, steering angle, `delta`; acceleration, `a`. The update equations are as follows:

```
x_[t+1] = x[t] + v[t] * cos(psi[t]) * dt
y_[t+1] = y[t] + v[t] * sin(psi[t]) * dt
psi_[t+1] = psi[t] + v[t] / Lf * delta[t] * dt
v_[t+1] = v[t] + a[t] * dt
cte[t+1] = f(x[t]) - y[t] + v[t] * sin(epsi[t]) * dt
epsi[t+1] = psi[t] - psides[t] + v[t] * delta[t] / Lf * dt
```

#### Timesteps (N) and Timestep Duration (dt)

The final N and dt were set 10 and 0.1 and it performed well. I also tried other combinations by adjusting one parameter while another fixed. The value for dt I tried was from 0.05 to 0.20 with the step 0.5 while the N set 10. It was found that 0.1 was the best for dt without serious erratic driving. The adjusting range for N was from 9 to 11 since 10 was already good enough then. And I found that they performed similar to each other so 10 was kept for N.

####  Waypoints Preprocessing and Polynomial Fitting

Since the waypoits were originally in a global coordinates, they were first transformed to the vehicles perspective.

```
for (unsigned int i = 0; i < ptsx.size(); i++) {
  double xdif = ptsx[i]-px_adv;
  double ydif = ptsy[i]-py_adv;
  ptsx_vehicle[i] = xdif * cos(-psi_adv) - ydif * sin(-psi_adv);
  ptsy_vehicle[i] = xdif * sin(-psi_adv) + ydif * cos(-psi_adv);
}
```

Then I used 3rd order polynomial fitting to fit the waypoints to get the path to follow.

#### Model Predictive Control with Latency

Due to the existence of latency, the vehicle got the output of actuator later than the moment that it got the 'current state'. So I set the latency and predicted the state right after the latency using the kinetic model.

```
//Consider the latency, the calculation will start after the time lapse
double time_lapse = 0.1;

//Predict the state after the latency
double px_adv = px + (v * cos(psi) * time_lapse);
double py_adv = py + (v * sin(psi) * time_lapse);
double psi_adv = psi - ((v * delta * time_lapse)/Lf);
double v_adv = v + (acceleration * time_lapse);
```

The I used the predicted state as the initial state and fed it to MPC for vehicle control.

## Dependencies

* cmake >= 3.5
 * All OSes: [click here for installation instructions](https://cmake.org/install/)
* make >= 4.1(mac, linux), 3.81(Windows)
  * Linux: make is installed by default on most Linux distros
  * Mac: [install Xcode command line tools to get make](https://developer.apple.com/xcode/features/)
  * Windows: [Click here for installation instructions](http://gnuwin32.sourceforge.net/packages/make.htm)
* gcc/g++ >= 5.4
  * Linux: gcc / g++ is installed by default on most Linux distros
  * Mac: same deal as make - [install Xcode command line tools]((https://developer.apple.com/xcode/features/)
  * Windows: recommend using [MinGW](http://www.mingw.org/)
* [uWebSockets](https://github.com/uWebSockets/uWebSockets)
  * Run either `install-mac.sh` or `install-ubuntu.sh`.
  * If you install from source, checkout to commit `e94b6e1`, i.e.
    ```
    git clone https://github.com/uWebSockets/uWebSockets
    cd uWebSockets
    git checkout e94b6e1
    ```
    Some function signatures have changed in v0.14.x. See [this PR](https://github.com/udacity/CarND-MPC-Project/pull/3) for more details.

* **Ipopt and CppAD:** Please refer to [this document](https://github.com/udacity/CarND-MPC-Project/blob/master/install_Ipopt_CppAD.md) for installation instructions.
* [Eigen](http://eigen.tuxfamily.org/index.php?title=Main_Page). This is already part of the repo so you shouldn't have to worry about it.
* Simulator. You can download these from the [releases tab](https://github.com/udacity/self-driving-car-sim/releases).
* Not a dependency but read the [DATA.md](./DATA.md) for a description of the data sent back from the simulator.


## Basic Build Instructions

1. Clone this repo.
2. Make a build directory: `mkdir build && cd build`
3. Compile: `cmake .. && make`
4. Run it: `./mpc`.

## Tips

1. It's recommended to test the MPC on basic examples to see if your implementation behaves as desired. One possible example
is the vehicle starting offset of a straight line (reference). If the MPC implementation is correct, after some number of timesteps
(not too many) it should find and track the reference line.
2. The `lake_track_waypoints.csv` file has the waypoints of the lake track. You could use this to fit polynomials and points and see of how well your model tracks curve. NOTE: This file might be not completely in sync with the simulator so your solution should NOT depend on it.
3. For visualization this C++ [matplotlib wrapper](https://github.com/lava/matplotlib-cpp) could be helpful.)
4.  Tips for setting up your environment are available [here](https://classroom.udacity.com/nanodegrees/nd013/parts/40f38239-66b6-46ec-ae68-03afd8a601c8/modules/0949fca6-b379-42af-a919-ee50aa304e6a/lessons/f758c44c-5e40-4e01-93b5-1a82aa4e044f/concepts/23d376c7-0195-4276-bdf0-e02f1f3c665d)
5. **VM Latency:** Some students have reported differences in behavior using VM's ostensibly a result of latency.  Please let us know if issues arise as a result of a VM environment.

## Editor Settings

We've purposefully kept editor configuration files out of this repo in order to
keep it as simple and environment agnostic as possible. However, we recommend
using the following settings:

* indent using spaces
* set tab width to 2 spaces (keeps the matrices in source code aligned)

## Code Style

Please (do your best to) stick to [Google's C++ style guide](https://google.github.io/styleguide/cppguide.html).



## Call for IDE Profiles Pull Requests

Help your fellow students!

We decided to create Makefiles with cmake to keep this project as platform
agnostic as possible. Similarly, we omitted IDE profiles in order to we ensure
that students don't feel pressured to use one IDE or another.

However! I'd love to help people get up and running with their IDEs of choice.
If you've created a profile for an IDE that you think other students would
appreciate, we'd love to have you add the requisite profile files and
instructions to ide_profiles/. For example if you wanted to add a VS Code
profile, you'd add:

* /ide_profiles/vscode/.vscode
* /ide_profiles/vscode/README.md

The README should explain what the profile does, how to take advantage of it,
and how to install it.

Frankly, I've never been involved in a project with multiple IDE profiles
before. I believe the best way to handle this would be to keep them out of the
repo root to avoid clutter. My expectation is that most profiles will include
instructions to copy files to a new location to get picked up by the IDE, but
that's just a guess.

One last note here: regardless of the IDE used, every submitted project must
still be compilable with cmake and make./
