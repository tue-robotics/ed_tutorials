# Tutorial 1) Running ED with a configuration file

## Prerequisites

- <https://github.com/tue-robotics/ed.git>
- <https://github.com/tue-robotics/ed_gui_server.git>
- <https://github.com/tue-robotics/ed_rviz_plugins.git>

## Tutorial

ED strongly relies on plugins to integrate sensor data, estimate object positions, recognize objects, etc. The plugins are discovered by the exports from the `package.xml` files. To get an overview of all defined plugins, use the `list_plugin` tool:

```bash
rosrun ed list_plugins
```

You can start ED by running:

```bash
rosrun ed ed
```

This will start the ED server which holds the world data structure. However, no world data is loaded, so the ED instance is quite useless at the moment. The world data that should be loaded and the plugins that should be started can be specified in a *configuration file*. Then, it's simply a matter of running ED like this:

```bash
rosrun ed ed CONFIG-FILE
```

The configuration file should be written in a [YAML-format](www.yaml.org) and mainly consists of two parts: a world specification, and a list of plugins. The first determines what the world looks like, the second what we want to do with the world representation. Create a file called 'my-ed-config.yaml' and put this in:

```yaml
world:
- id: block1
  pose: { x: 2, y: 0, z: 0.5 }
  shape:
    box:
      size: { x: 0.5, y: 0.5, z: 1 }
- id: block2
  pose: { x: 3, y: 2, z: 0.75 }
  shape:
    box:
      size: { x: 0.5, y: 0.5, z: 1.5 }

plugins:
  - name: gui_server
    type: ed_gui_server/gui_server
```

Take some time to try to understand what the config file states. A world is specified consisting of two objects - we'll call these entities. Both entities have an ID, a position in the world, and a shape. The shape is described as being a box with a certain size. Furthermore, we specify that we want to run the gui_server plugin. This will allow us to visualize the world in RViz.

Now, run ED and point it to the configuration file created above, like:

```bash
rosrun ed ed <path/to/my-ed-config.yaml>
```

For example, if we are still in the same folder as the configuration file:

```bash
rosrun ed ed my-ed-config.yaml
```

You probably won't see a lot happening. But ED *is* running. To visualize the world we have described, we need to run another node. This is some sort of proxy which communicates over a low-bandwidth channel with the ED GUI server plugin and generates visualization markers, which are typically high-bandwidth. This is useful: when running ED on the robot, make sure you run the proxy on a local machine. The local machine will communicate with the robot over a low-bandwidth topic, while RViz will 'listen' to the high-bandwidth visualization markers which are produced on the same local machine. To run the proxy:

```bash
rosrun ed_gui_server ed_rviz_publisher
```

Now start RViz, and listen to the Marker topic `/ed/rviz`. You should see two blocks appearing: the blocks you specified in the configuration file. Or use the rviz_plugin from the `ed_rviz_plugins` package to visualize.

![publisher](img/rviz_publisher.png)

You can also use the rviz plugin: Start rviz and add the `ed_rviz_plugins/WorldModel` display. Configure the service for querying the meshes and the ED entities topic, e.g. `/ed/gui/entities` and `/ed/gui/query_meshes`.

![plugin](img/plugin.png)
