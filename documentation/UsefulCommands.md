# Main Controls
These controls are the basic and advanced controls for the polyscope UI. But they can also be implemented via .script files allowing for more  
## Movement
### MoveJ 
optimal routing between waypoints -> path might curve and differ in speed, joints will start and stop at the same time.
Sidenote: If we would ever need to change robots, using the joint angle mode might help, but would still need adjsuting the base/features/etc.

### MoveL
linear movement between points, straight lines but joints need more complex mocements (would that be a problem?)

### MoveP
movement between points with constant speed -> needs to add a circular blend (same for all waypoints on the path) for points on the trajectory and might "miss" them.
We probably dont need it that much unless we would be using like a conveyor belt or something with constant speed that needs matching.

### using the TCP
we can change the TCP in the moves to ensure the correct reference point to move to the specified waypoints. Seems to also affect orientation then (not sure for now)

### using with features
Setting a feature enables the move to be on a differnt point/line/plane allowing for reuse of predefined movesets like picking up a breaker or setting it in the cabinet at different locations by transposing the location with new points. For more see [Feature](#Feature)

### optimove
With optimove the speed and acceleration are more smoothly rendered. Using this with [moveJ](#movej) or [moveL](#moveL) should result in the same path (at least thats what I understood).

## Feature
A feature is a reference point/line or plane for the robot to move relativ to. This enables us to reuse functions that are implemented in a way, that we define a move between different points and only adjust the reference point for each new approach. For our setting this might be picking up or releasing the breaker.
> [!NOTE]
> The use of the feature for the explained purpose is only possible with variable Waypionts. Using Fixed Waypoints i.e. The move function has the waypiont with fixed values the shifting of the plane is not possible.
