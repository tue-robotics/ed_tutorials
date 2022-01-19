# Tutorial 6) Localization

## Prerequisites

- <https://github.com/tue-robotics/ed.git>
- <https://github.com/tue-robotics/ed_gui_server.git>
- <https://github.com/tue-robotics/ed_rviz_plugins.git>
- <https://github.com/tue-robotics/ed_localization.git>
- 2D Range Finder (<http://wiki.ros.org/Sensors>) on the robot which scans in a plane parallel to the floor
- [TF tree](wiki.ros.org/tf) containing transforms from the robots' odometry frame to the 2D Range Finder frame
- Knowledge about TF and robot localization. Have a look at the [AMCL wiki page](http://wiki.ros.org/amcl) if not.

## Tutorial

So far we've created a world model using heightmaps, primitives and other models, and we were able to visualize it. However, a world model on its own is not very useful. Let's start using it to make a robot behave autonomously! The package [ed_localization](https://github.com/tue-robotics/ed_localization) allows your robot to localize itself in the ED world model, using the robot's odometry and 2D Range Finder.

The ED localization is largely based on ROS' [AMCL-module](http://wiki.ros.org/amcl), which in turn uses several algorithms from the book 'Probabilistic Robotics' by Thrun, Burgard, and Fox. It is a particle filter implementation which uses the robot's odometry for the *prediction* step, and the 2D Range Finder scan for the *update*. The main difference with AMCL is that ED localization does not operate on an occupancy grid given by the user, but on the 3D world model contained in ED. And by using a technique based on GPU rasterization methods instead of ray casting the sensor data for each particle, the ED localization module is quite a bit more efficient than AMCL! But let's not get too technical. How do we use it?!

As stated above, we assume that you have a 2D Range Finder (we will sometimes use the word **laser**) available and that a transform from the robot odometry frame to the Range Finder frame can be calculated. The latter means that the odometry of your wheels is expressed in a TF frame from a virtual 'odometry frame' to your robot. If the wheels of your robot turn, the odometry frame should be updated. This is used by the localization algorithm to calculate how much the robot has moved since the last update.

The ED localization module is in fact just an ED plugin. So, as you may have guessed, we need to specify it in our ED configuration file. Add the localization plugin and it's parameters to the list of plugins in your configuration file. It will then look like this:

```yaml
world:
- type: robot-lab
  pose: { x: 0, y: 0, z: 0 }

plugins:
  - name: gui_server
    type: ed_gui_server/gui_server
  - name: localization
    type: ed_localization/localization
    parameters:
      robot_name: robot    # the robot will also be in the world model. This is the
                           # id the robot entity will get in ED
      initial_pose_topic: [INITIAL_POSE_TOPIC]
      particle_filter:
        min_particles: 100
        max_particles: 750
        kld_err: 0.02
        kld_z: 0.98
        recovery_alpha_slow: 0 #0.001
        recovery_alpha_fast: 0 #0.1
        cell_size_x: 0.5
        cell_size_y: 0.5
        cell_size_theta: 0.1745
      initial_pose:        # where does the robot start (in map frame)?
        x: 0
        y: 0
        rz: 0              # rotation
      laser_model:
        topic: [LASER_TOPIC]   # Laser topic
        num_beams: 100         # Max number of beams used per particle (evenly spread)
        z_hit: 0.95            # \
        sigma_hit: 0.2         # |
        z_short: 0.1           # |-- These are all parameters of the probabilistic laser
        z_max: 0.05            # |    model. See 'Probabilistic Robotics' for more info.
        z_rand: 0.05           # |
        lambda_short: 0.1      # /
        range_max: 10
        min_particle_distance: 0.01            # Particles that are too close together will
        min_particle_rotation_distance: 0.02   # be combined (resulting in less particles)
      odom_model:
        map_frame: map
        odom_frame: [ODOM_TF_FRAME]
        base_link_frame: [BASE_TF_FRAME]
        alpha1: 0.2   # rot -> trans + strafe    # \
        alpha2: 0.2    # trans -> rot            # |-- These are all parameters of the
        alpha3: 0.2    # trans -> trans          # |   probabilistic odom model. See
        alpha4: 0.2    # rot -> rot              # |   'Probabilistic Robotics' for more info
        alpha5: 0.2    # trans -> strafe         # /
```

Woah, that's a lot of parameters! Fortunately, it will probably work quite well with the parameters above. The most important thing is to fill in the values represented as [...], i.e., the initial pose topic, laser topic, odometry tf frame, and base tf frame. If you want to know more about the model parameters, have a look at the [AMCL wiki page](http://wiki.ros.org/amcl).

Now run ED with this configuration file, and visualize the whole thing in RViz. You should be able to select the 'map' frame as global frame, and see a transform from the map frame to your robot. You might notice that the robot is badly localized at first. That is because you need to tell the localization plugin the **initial pose** of the robot. You can either specify this in the configuration file, or your can use the initial pose topic to set it (you can do this in RViz if you correctly set the initial pose topic). Now drive around your robot. You should notice how the robot keeps itself localized with respect to the environment.

If the above doesn't work, make sure that:

- The Range Finder is indeed broadcasting scan messages (e.g., use 'rostopic echo' to find out)
- The laser topic is correctly specified in the configuration file
- The frame_id specified in the Range Finder messages are correct (i.e., there is a valid transformation from the odometry frame to this frame)
- The TF frames in the configuration file are correct

![localization](img/localization.png)
