# Tutorial 11) Classification

## Prerequisites

- Tutorial 09
- The perception models must be trained using the [ed_perception tutorial](https://github.com/tue-robotics/ed_perception).

## Tutorial

We now have entities with associated RGBD measurement in our world model (output of tutorial 10). With use of the perception plugin within the world model, classifiers can use the associated measurements to add a symbolic label (type) to the specific entity.

We need to add the perception plugin to our world model configuration:

```yaml
- name: perception
  type: ed_perception/image_recognition
```

Now we launch the world model again and segment some objects (tutorial 09). This will result in something like this:

![segment](img/segment.png)

We have created the entities with the following ID's (can be queries with use of the query interface explained in tutorial 09):

- id: 486c8a807eeb4ba5b27ed53e5af9ad7e
- id: 94a87f4c8c6e45ce1f813ae510713fed
- id: d6cce35c487a58feab29032c2621f38d

Now we would like to classify the newly created entities based on the measurement that we took when we did the segmentation. The new perception plugin exposed a ROS service interface:

```bash
/ed/classify
```

We can classify the entities as follows:

```bash
rosservice call /ed/classify "{perception_models_path: '/home/amigo/ros/indigo/system/src/ed_perception_models/models/robotics_testlabs', ids: ["486c8a807eeb4ba5b27ed53e5af9ad7e","486c8a807eeb4ba5b27ed53e5af9ad7e","486c8a807eeb4ba5b27ed53e5af9ad7e"], property: 'type', prior: {values: [],  probabilities: [], unknown_probability: 0.1}}
```

Result:

```yaml
    ids: ['486c8a807eeb4ba5b27ed53e5af9ad7e', '486c8a807eeb4ba5b27ed53e5af9ad7e', '486c8a807eeb4ba5b27ed53e5af9ad7e']
posteriors:
  - values: ['beer', 'bifrutas', 'coffee_pads', 'coke', 'deodorant', 'fanta', 'ice_tea', 'mentos', 'sprite', 'tea', 'teddy_bear', 'water', 'xylit24_spearmint', 'xylit24_white']
    probabilities: [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0]
    unknown_probability: 1.0
  - values: ['beer', 'bifrutas', 'coffee_pads', 'coke', 'deodorant', 'fanta', 'ice_tea', 'mentos', 'sprite', 'tea', 'teddy_bear', 'water', 'xylit24_spearmint', 'xylit24_white']
    probabilities: [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0]
    unknown_probability: 1.0
  - values: ['beer', 'bifrutas', 'coffee_pads', 'coke', 'deodorant', 'fanta', 'ice_tea', 'mentos', 'sprite', 'tea', 'teddy_bear', 'water', 'xylit24_spearmint', 'xylit24_white']
    probabilities: [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0]
    unknown_probability: 1.0
expected_values: ['', '', '']
expected_value_probabilities: [1.0, 1.0, 1.0]
error_msg: ''
```
