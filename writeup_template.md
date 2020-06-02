## Project: 3D Motion Planning
![Quad Image](./misc/enroute.png)

---


# Required Steps for a Passing Submission:
1. Load the 2.5D map in the colliders.csv file describing the environment.
2. Discretize the environment into a grid or graph representation.
3. Define the start and goal locations.
4. Perform a search using A* or other search algorithm.
5. Use a collinearity test or ray tracing method (like Bresenham) to remove unnecessary waypoints.
6. Return waypoints in local ECEF coordinates (format for `self.all_waypoints` is [N, E, altitude, heading], where the drone’s start location corresponds to [0, 0, 0, 0].
7. Write it up.
8. Congratulations!  Your Done!

## [Rubric](https://review.udacity.com/#!/rubrics/1534/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  

You're reading it! Below I describe how I addressed each rubric point and where in my code each point is handled.

### Explain the Starter Code

#### 1. Explain the functionality of what's provided in `motion_planning.py` and `planning_utils.py`
`backyard_flyer_solution.py` script contain a basic planning implementation that move the drone in square box and reach again at same location. Where square box is calculated based on the defined waypoints. This is not going to work in real world scenario where we mention the start and goal location. This script working on 6 states which are defined as  
    MANUAL = 0
    ARMING = 1
    TAKEOFF = 2
    WAYPOINT = 3
    LANDING = 4
    DISARMING = 5

`motion_planning.py` script contain advance version of `backyard_flyer_solution.py` script where start location is drone current location and goal is 10 m ahead. This script working on 7 states which are defined as
    MANUAL = auto()
    ARMING = auto()
    TAKEOFF = auto()
    WAYPOINT = auto()
    LANDING = auto()
    DISARMING = auto()
    PLANNING = auto()
    
which is taking one extra PLANNING state. This state is executing after the drone is armed. Basically it is planning the path based on the algorithm used in `planning_utils.py`. And once the path is planned the waypoint is send to the simulator and further action is taken care by this script.

`planning_utils.py` script contain the algorithm for path planning. Currently it is used to generate the map grid on that it is using the A* search algorithm to find the path between start to goal node.

### Implementing Your Path Planning Algorithm

#### 1. Set your global home position

The below modification in `motion_planning.py` file

Using a csv package i have read the `colliders.csv` file. Because the lat0 and lon0 stored at first line so i have created read only the first line of the file and split the data and copnverted the data in float values. For that i have created `getLatLonFromColliders(fileName)` member function of class which is taking the filename and return the lat0 and lon0 float value.

Then i have used `self.set_home_position()` method to set global home. I have written below code for this implementation

`self.set_home_position(lon0, lat0,0.0)`

#### 2. Set your current local position

The below modification in `motion_planning.py` file

Because my current global position is in geodectic coordinate (latitute, longitude, altitude) system. But to work by the drone i need to convert them in local North, East and Down coordinate system. To do that i have calculated my local position relative to global home using `global_to_local()` function from `udacidrone.frame_utils`. 

#### 3. Set grid start position from local position

The below modification in `motion_planning.py` file

Here i have converted start position to current position rather than map center as below
Where `drone_local_position` is drone local position

`grid_start = (int(drone_local_position[0]-north_offset),int(drone_local_position[1]-east_offset))`

#### 4. Set grid goal position from geodetic coords

The below modification in `motion_planning.py` file

For goal position i have taken one point in lat, lon coordinates as defined below
`target_global_position=np.array([-122.398362,37.793677,0.0])`

But to work with drone i have converted that in local North, East and Down coordinate system relative to global home using `global_to_local()` function. And the calculated the grid goal position as below

```
        #arbitrary lat, lon point on grid
        target_global_position=np.array([-122.398362,37.793677,0.0])
        #Conversion of geodetic to local coordinate 
        drone_goal_local_position=global_to_local(target_global_position, self.global_home)
        grid_goal = (int(drone_goal_local_position[0]-north_offset), int(drone_goal_local_position[1]-east_offset))
```
#### 5. Modify A* to include diagonal motion (or replace A* altogether)

Minimal requirement here is to modify the code in planning_utils() to update the A* implementation to include diagonal motions on the grid that have a cost of sqrt(2), but more creative solutions are welcome.

The below modification in `planning_utils.py` file

To Implement the above requirement i have done below steps

1. Added four new action for diagonal motions with cost of sqrt(2)
    NORTH_EAST = (-1, 1, sqrt(2))
    SOUTH_EAST = (1, 1, sqrt(2))
    NORTH_WEST = (-1, -1, sqrt(2))
    SOUTH_WEST = (1, -1, sqrt(2))

2. Added four new if condition in valid_actions method to check that digonal actions do not make the drone fly off the grid and into obstacles. Invalid actions get removed from the list of valid actions.

#### 6. Cull waypoints 

For this step you can use a collinearity test or ray tracing method like Bresenham. The idea is simply to prune your path of unnecessary waypoints.

The below modification in `planning_utils.py` file

I have used collinearity test with an epsilon = 5 to pruned the path and remove the unnecessary waypoints. pruned path is returned from `prune_path` function.


### Execute the flight
#### 1. Does it work?
It works fine!
I have given the goal position as lat, lon value and it is successfully moved to that location. Below os the result image

![Result Image](./misc/result.png)


### Double check that you've met specifications for each of the [rubric](https://review.udacity.com/#!/rubrics/1534/view) points.


