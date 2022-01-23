# Tutorial 3) Creating re-usable models

## Prerequisites

- <https://github.com/tue-robotics/ed.git>
- <https://github.com/tue-robotics/ed_gui_server.git>
- <https://github.com/tue-robotics/ed_rviz_plugins.git>

## Tutorial

Imagine you just took some time describing the shape of a certain table, but the robots' environment contains multiple instances of that table. It would be nice to create a model, and simply re-use that model multiple times within your environment description, or maybe even in other environment descriptions you have.

Well, you can! Before we start, let's create a directory which will hold these models:

```bash
mkdir my-model-dir
```

You have to let ED know where it will be able to find the models. You can do this by setting the ED_MODEL_PATH variable.

```bash
export ED_MODEL_PATH=`pwd`/my-model-dir
```

You can specify multiple paths by separating them with colons (':').

Let's say we want to create an object model called 'table'. Create a directory with this name inside the model directory

```bash
cd my-model-dir
mkdir table
```

A model specification should always contain at least a file called 'model.yaml'. This will contain the description of the model. Of course you are free to add any other files that have something to do with the model, ''e.g.'', heightmap images, mesh binaries, etc. Inside the 'model.yaml' you can specify the shape of the model and some other properties such as color. So, inside the table directory create a file called 'model.yaml' with the following content:

```yaml
shape:
  group:
  - box:
      pose: {x: 0,    y: 0,      z: 0.75}
      size: {x: 1.20, y: 0.80,   z: 0.02}
  - box:
      pose: {x: -0.55, y: -0.35, z: 0.37}
      size: {x: 0.05,  y: 0.05,  z: 0.74}
  - box:
      pose: {x: 0.55,  y: -0.35, z: 0.37}
      size: {x: 0.05,  y: 0.05,  z: 0.74}
  - box:
      pose: {x: -0.55, y: 0.35,  z: 0.37}
      size: {x: 0.05,  y: 0.05,  z: 0.74}
  - box:
      pose: {x: 0.55,  y: 0.35,  z: 0.37}
      size: {x: 0.05,  y: 0.05,  z: 0.74}
```

So, what does this say? Note that this format looks *a lot* like the world description we specified above in the configuration file. And you know what? It has the exact same syntax! This simply states that the shape of the table is a collection of boxes (5 to be precise: 1 table top and 4 legs). In itself, this model is pretty useless. But we can instantiate it (multiple times if we want!) in you world specification. For example, try the following ED configuration:

```yaml
world:
- id: table1
  type: table
  pose: { x: 2, y: 0, z: 0 }
- id: table2
  type: table
  pose: { x: 3, y: 2, z: 0, Z: 1.54 }

plugins:
  - name: gui_server
    type: ed_gui_server/gui_server
```

**Make sure the ED_MODEL_PATH environment variable is set!**

The 'type' field specifies we should instantiate an entity of the given model, 'table' in our case. What will happen under the hood is that ED starts looking for a directory called 'table' inside the model directories, and if it finds it, inspects the 'model.yaml' file. Then the entities 'table1' and 'table2' *inherit* all properties that are specified in the 'model.yaml'.

When you run visualization (you know how) you should see two tables. Neat!

![2tables](img/2tables.png)
