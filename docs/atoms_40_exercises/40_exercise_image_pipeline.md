# Exercise: Augmented Reality {#exercise-augmented-reality status=ready}

Assigned: Jonathan Michaux and Dzenan Lapandic

## Skills learned

* Understanding of all the steps in the image pipeline.
* Writing markers on images to aid in debugging.

## Introduction

During the lectures, we have explained one direction of the image pipeline:

    image -> [feature extraction] -> 2D features -> [ground projection] -> 3D world coordinates

In this exercise, we are going to look at the pipeline in the opposite direction.

It is often said that:

> "The inverse of computer vision is computer graphics."

The inverse pipeline looks like this:

    3D world coordinates -> [image projection] -> 2D features -> [rendering] -> image


## Instructions


* Do intrinsics/extrinsics camera calibration of your robot as per the instructions.
* Write the ROS node specified below in [](#exercise-augmented-reality-spec).


Then verify the results in the following 3 situations.


### Situation 1: Calibration pattern

* Put the robot in the middle of the calibration pattern.
* Run the program `dt_augmented_reality` with map file `calibration_pattern.yaml`.

(Adjust the position until you get perfect match of reality and augmented reality.)

### Situation 2: Lane

* Put the robot in the middle of a lane.
* Run the program `dt_augmented_reality` with map file `lane.yaml`.

(Adjust the position until you get a perfect match of reality and augmented reality.)

### Situation 3: Intersection

* Put the robot at a stop line at a 4-way intersection in Duckietown.
* Run the program `dt_augmented_reality` with map file `intersection_4way.yaml`.

(Adjust the position until you get a perfect match of reality and augmented reality.)

### Submission

Submit the images according to location-specific instructions.


## Specification of `dt_augmented_reality` {#exercise-augmented-reality-spec}

In this assignment you will be writing a ROS package to perform the augmented reality exercise. The program will be invoked with the following syntax:

    $ roslaunch dt_augmented_reality-![robot name] augmenter.launch map_file:=![map file] robot_name:=![robot name] local:=1

where `![map file]` is a YAML file containing the map (specified in [](#exercise-augmented-reality-map)).

If `![robot name]` is not given, it defaults to the hostname.

The program does the following:

1. It loads the intrinsic / extrinsic calibration parameters for the given robot.
2. It reads the map file.
3. It listens to the image topic `/![robot name]/camera_node/image/compressed`.
4. It reads each image, projects the map features onto the image, and then writes the resulting image to the topic

       /![robot name]/AR/![map file basename]/image/compressed

where `![map file basename]` is the basename of the file without the extension.

We provide you with ROS package template that contains the `AugmentedRealityNode`. By default, launching the `AugmentedRealityNode` should publish raw images from the camera on the new `/![robot name]/AR/![map file basename]/image/compressed` topic.

In order to complete this exercise, you will have to fill in the missing details of the `Augmenter` class by doing the following:

1. Implement a method called `process_image` that undistorts raw images.
2. Implement a method called `ground2pixel` that transforms points in ground coordinates (i.e. the robot reference frame) to pixels in the image.
3. Implement a method called `callback` that writes the augmented image to the appropriate topic.

## Specification of the map {#exercise-augmented-reality-map}

The map file contains a 3D polygon, defined as a list of points and a list of segments
that join those points.

The format is similar to any data structure for 3D computer graphics, with a few changes:

1. Points are referred to by name.
2. It is possible to specify a reference frame for each point. (This will help make this into
a general tool for debugging various types of problems).

Here is an example of the file contents, hopefully self-explanatory.

The following map file describes 3 points, and two lines.

    points:
        # define three named points: center, left, right
        center: [axle, [0, 0, 0]] # [reference frame, coordinates]
        left: [axle, [0.5, 0.1, 0]]
        right: [axle, [0.5, -0.1, 0]]
    segments:
    - points: [center, left]
      color: [rgb, [1, 0, 0]]
    - points: [center, right]
      color: [rgb, [1, 0, 0]]


### Reference frame specification

The reference frames are defined as follows:

- `axle`: center of the axle; coordinates are 3D.
- `camera`: camera frame; coordinates are 3D.
- `image01`: a reference frame in which 0,0 is top left, and 1,1 is bottom right of the image; coordinates are 2D.

(Other image frames will be introduced later, such as the `world` and `tile` reference frame, which
need the knowledge of the location of the robot.)

### Color specification

RGB colors are written as:

    [rgb, [![R], ![G], ![B]]]

where the RGB values are between 0 and 1.

Moreover, we support the following strings:

- `red` is equivalent to `[rgb, [1,0,0]]`
- `green` is equivalent to `[rgb, [0,1,0]]`
- `blue` is equivalent to `[rgb, [0,0,1]]`
- `yellow` is equivalent to `[rgb, [1,1,0]]`
- `magenta` is equivalent to `[rgb, [1,0,1]]`
- `cyan` is equivalent to `[rgb, [0,1,1]]`
- `white` is equivalent to `[rgb, [1,1,1]`
- `black` is equivalent to `[rgb, [0,0,0]]`


## "Map" files


### `hud.yaml`

This pattern serves as a simple test that we can draw lines in image coordinates:

    points:
        TL: [image01, [0, 0]]
        TR: [image01, [0, 1]]
        BR: [image01, [1, 1]]
        BL: [image01, [1, 0]]
    segments:
    - points: [TL, TR]
      color: red
    - points: [TR, BR]
      color: green
    - points: [BR, BL]
      color: blue
    - points: [BL, TL]
      color: yellow

The expected result is to put a border around the image:
red on the top, green on the right, blue on the bottom, yellow on the left.

### `calibration_pattern.yaml`

This pattern is based off the checkerboard calibration target used in estimating the intrinsic and extrinsic camera parameters:


    points:
        TL: [axle, [0.315, 0.093, 0]]
        TR: [axle, [0.315, -0.093, 0]]
        BR: [axle, [0.191, -0.093, 0]]
        BL: [axle, [0.191, 0.093, 0]]
    segments:
    - points: [TL, TR]
      color: red
    - points: [TR, BR]
      color: green
    - points: [BR, BL]
      color: blue
    - points: [BL, TL]
      color: yellow

The expected result is to put a border around the inside corners of the checkerboard: red on the top, green on the right, blue on the bottom, yellow on the left.

### `lane.yaml`

We want something like this:

                      0
     |   |          | . |             |   |
     |   |          | . |             |   |
     |   |          | . |             |   |
     |   |          | . |             |   |
     |   |          | . |             |   |
     |   |          | . |             |   |
      WW      L       WY      L         WW
     1   2          3   4             5   6

Then we have:

    points:
         p1: [axle, [0, 0.2794, 0]]
         q1: [axle, [D, 0.2794, 0]]
         p2: [axle, [0, 0.2286, 0]]
         q2: [axle, [D, 0.2286, 0]]
         p3: [axle, [0, 0.0127, 0]]
         q3: [axle, [D, 0.0127, 0]]
         p4: [axle, [0, -0.0127, 0]]
         q4: [axle, [D, -0.0127, 0]]
         p5: [axle, [0, -0.2286, 0]]
         q5: [axle, [D, -0.2286, 0]]
         p6: [axle, [0, -0.2794, 0]]
         q6: [axle, [D, -0.2794, 0]]
    segments:
     - points: [p1, q1]
       color: white
     - points: [p2, q2]
       color: white
     - points: [p3, q3]
       color: yellow
     - points: [p4, q4]
       color: yellow
     - points: [p5, q5]
       color: white
     - points: [p6, q6]
       color: white

### `intersection_4way.yaml`


TODO: to write

## Suggestions

Start by using the file `hud.yaml`. To visualize it, you do not need the
calibration data. It will be helpful to make sure that you can do the easy
parts of the exercise: loading the map, and drawing the lines.

## Useful APIs

### Loading a map file:

To load a map file, use the function `load_map` provided in `duckietown_utils`:

    from duckietown_utils import load_map

    map_data = load_map(map_filename)

(Note that `map` is a reserved symbol name in Python.)

### Reading the calibration data for a robot

To load the _intrinsic_ calibration parameters, use the function `load_camera_intrinsics` provided in `duckietown_utils`:

    from duckietown_utils import load_camera_intrinsics

    intrinsics = load_camera_intrinsics(robot_name)

To load the _extrinsic_ calibration parameters (i.e. ground projection), use the function `load_homography` provided in `duckietown_utils`:

    from duckietown_utils import load_homography

    H = load_homography(robot_name)

### Path name manipulation

From a file name like `"/path/to/map1.yaml"`, you can obtain the basename without extension `yaml` by using the function `get_base_name` provided in `duckietown_utils`:

    from duckietown_utils import get_base_name

    filename = "/path/to/map1.yaml"
    map_name = get_base_name(filename) # = "map1"

### Undistorting an image

To remove the distortion from an image, use the function `rectify` provided in `duckietown_utils`:

    from duckietown_utils import rectify

    rectified_image = rectify(image, intrinsics)

### Drawing primitives

To draw the line segments specified in a map file, use the `render_segments` method defined in the `Augmenter` class:

    class Augmenter():

        # ...

        def ground2pixel(self):
            '''Method that transforms ground points
            to pixel coordinates'''
            # Your code goes here.
            return "???"



    image_with_drawn_segments = augmenter.render_segments(image)


In order for `render_segments` to draw segments on an image, you must first implement the method `ground2pixel`.
