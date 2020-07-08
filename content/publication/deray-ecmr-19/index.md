---
title: "Joint on-manifold self-calibration of odometry model and sensor extrinsics using pre-integration"
date: 2019-09-01
publishDate: 2020-05-09T22:12:35.024059Z
authors: ["J. Deray", "J. Solà", "J. Andrade-Cetto"]
publication_types: ["1"]
abstract: "This paper describes a self-calibration procedure that jointly estimates the extrinsic parameters of an exteroceptive sensor able to observe ego-motion, and the intrinsic parameters of an odometry motion model, consisting of wheel radii and wheel separation. We use iterative nonlinear on-manifold optimization with a graphical representation of the state, and resort to an adaptation of the pre-integration theory, initially developed for the IMU motion sensor, to be applied to the differential drive motion model. For this, we describe the construction of a pre-integrated factor for the differential drive motion model, which includes the motion increment, its covariance, and a first-order approximation of its dependence with the calibration parameters. As the calibration parameters change at each solver iteration, this allows a posteriori factor correction without the need of re-integrating the motion data. We validate our proposal in simulations and on a real robot and show the convergence of the calibration towards the true values of the parameters. It is then tested online in simulation and is shown to accommodate to variations in the calibration parameters when the vehicle is subject to physical changes such as loading and unloading a freight."
featured: false
publication: "*Proceedings European Conference on Mobile Robots*"
tags: ["Calibration;Robot sensing systems;Wheels;Kinematics;Jacobian matrices;Mobile robots"]
doi: "10.1109/ECMR.2019.8870942"
url_pdf: https://upcommons.upc.edu/bitstream/handle/2117/180773/2210-Joint-on-manifold-self-calibration-of-odometry-model-and-sensor-extrinsics-using-pre-integration.pdf
---
