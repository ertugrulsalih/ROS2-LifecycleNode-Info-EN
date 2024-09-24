# ROS2 Lifecycle Node

## What is a Lifecycle Node?
In addition to a standard Node, using a **Lifecycle Node** allows us to observe and control the stages of a Node. This feature enables developers to manage Node state transitions, creating more robust, reliable, and maintainable software. The lifecycle model in ROS 2 provides an interface that controls the creation, starting, running, stopping, resetting, and destruction of nodes in a defined sequence.

A **Lifecycle Node** is in the `Unconfigured` state when first launched. At this stage, the Node is initialized but does not perform any active tasks. The diagram below explains each state and the task of transitions (Figure 1.1). This scenario allows for Node management without going beyond the defined lifecycle stages.

![Lifecycle Node Diagram](https://github.com/ertugrulsalih/ROS2-LifecycleNode-Info-TR/blob/main/img/life_cycle_sm.png?raw=true)

## Example Scenario: Camera Configuration
Imagine a robot connected to a camera for object detection and obstacle avoidance. Configuring the camera is critical for ensuring reliable data. At this point, by using a **Lifecycle Node** instead of a standard Node, we can configure camera parameters such as resolution, frame rate, and exposure. We can also reliably publish camera data to detection and obstacle avoidance components.

This approach ensures that the camera Node is correctly set up and that its data is reliable, allowing for more effective lifecycle management.

## Example Scenario: ROS Navigation Stack

![Lifecycle Node Diagram](https://github.com/ertugrulsalih/ROS2-LifecycleNode-Info-TR/blob/main/img/nav_stack.png?raw=true)

The **ROS Navigation Stack** is a widely used software package for robotic navigation that includes various components like the map server, cost map generator, local planner, and global planner. These components need to be started and stopped in a specific order, and coordinating their lifecycles is critical for the overall system to work correctly. For example, the map server and sensor topics need to be loaded before the cost map and planner nodes. With the ROS 2 Lifecycle Node API, you can define a lifecycle state machine that includes the necessary steps for launching, shutting down, and transitioning states for each component. 

Thus, you can ensure that the Navigation Stack components are started and stopped in the correct order, guaranteeing stable and reliable system operation.

In addition to starting and stopping components, the ROS 2 Lifecycle Node can also manage runtime events like reconfiguring the navigation stack or reacting to environmental changes. For instance, if the robot encounters a new obstacle during navigation, the cost map generator may need to be reconfigured to account for it. By using the ROS 2 Lifecycle Node API, you can define a state transition to perform this reconfiguration while ensuring other components continue operating.

## Lifecycle Node States
There are four types of Primary States for Lifecycle Nodes:

- **Unconfigured**
- **Inactive**
- **Active**
- **Finalized**

Exiting a Primary State requires an external controller process to act, except for the error occurring in the Active state. There are also six Transition States:

- Configuring
- CleaningUp
- ShuttingDown
- Activating
- Deactivating
- ErrorProcessing

In the transition states, a logical operation is performed to determine whether the transition was successful. The success or failure of the transition is communicated to the lifecycle management software via the Lifecycle Management Interface.

## Lifecycle Node States

**Unconfigured State:** This is the state immediately after the Node is started. It is also the state the Node can be reset to if an error occurs. At this stage, the Node has no persistent state.

**Inactive State:** The Inactive state represents a Node that is not performing any operations. The purpose of this state is to allow the Node to be reconfigured (e.g., changing configuration parameters, adding/removing topic publications or subscriptions) without affecting its behavior. In this state, the Node does not use any execution time; i.e., it does not read topics, process data, or respond to functional service requests. A Node in the *Inactive* state also does not read or process data from managed topics and does not respond to managed service requests (which are immediately marked as failed for the caller).

**Active State:** The Active state is the primary stage of the Node’s lifecycle. In this state, the Node performs any operations, responds to service requests, reads and processes data, produces output, etc. If an unhandled error occurs while the Node is in this state, the Node transitions to the *ErrorProcessing* state.

**Finalized State:** The Finalized state is the stage where the Node is terminated before destruction. The only transition from this state is the destruction process. This state exists to support debugging and introspection. A failed Node remains visible for introspection instead of being immediately destroyed. If a Node is being respawned, or there is a known reason for being in a respawn loop, the controller process is expected to have a policy for automatically destroying and recreating the Node.

## Transitions

In a Lifecycle Node, each transition is managed by specific callbacks. For example:
- **on_configure:** Called when the Node transitions from Unconfigured to Inactive. Ideal for initializing motors or calibrating sensors.
- **on_activate:** Called when transitioning from Inactive to Active. Motor control signals can be activated during this phase.
- **on_deactivate:** Called when transitioning from Active to Inactive. Motors can be safely stopped during this phase.
- **on_cleanup:** Called when transitioning from Inactive to Unconfigured. This is the ideal phase for releasing unused resources.
- **on_shutdown:** Callback invoked when the Node is shutting down. Remaining processes are cleaned up during system shutdown.

These callbacks enable a robot to transition smoothly between different modes of operation.

## Lifecycle Node Management

A managed Node is incorporated into the ROS ecosystem via the Lifecycle interface as seen by the managing tools. This interface should not be subject to communication constraints over the lifecycle states when visible to management tools.

The most common patterns involve a container class loading a “managed node implementation” from a library and automatically exposing the required management interface through methods via a plugin architecture.

Containers themselves are not subject to lifecycle management. However, any implementation that provides this interface and adheres to lifecycle principles is considered a valid managed node. Otherwise, objects that provide these services but fail to follow the lifecycle state machine structure are considered misconfigured.

Management services can be provided both through local management (Attributes and Methods) and remote management (ROS messages and Topics). When providing a ROS interface, specific Topics should be used, and an appropriate namespace should be designated.

A managed Node may request arguments for “configure” and “activate” automatically when running in an unmanaged system. There are several ways a managed Node can transition between states. Most transitions are expected to be coordinated by an external manager (for instance, this could be a lifecycle_manager_node). The external management tool is also expected to monitor the lifecycle Node and perform recovery actions in case of failure. A local management tool can also be used, utilizing method-level interfaces. It is also possible for a Node to manage itself, though this is not recommended, as it could conflict with external logic attempting to manage the Node through the interface.

Support for this lifecycle toolchain is required at all stages; *therefore, it is not intended for this design to be extended with additional states.* More complex, application-specific state machines are expected to exist within any lifecycle state or at a macro level. These lifecycle states are expected to be useful building blocks within a controller system.

## Useful Links
- [Managed Nodes](https://design.ros2.org/articles/node_lifecycle.html)
- [ROS2 from the Ground Up: Part 8- Simplify Robotic Software Components Management with ROS2 Lifecycle Nodes](https://medium.com/@nullbyte.in/ros2-from-the-ground-up-part-8-simplify-robotic-software-components-management-with-ros2-5fafa2738700)
- [How to Use ROS 2 Lifecycle Nodes](https://docs.ros.org/en/ros2_packages/rolling/api/lifecycle_msgs/index.html)
- [ROS2 - Lifecycle Node Tutorial](https://www.youtube.com/watch?v=_GXHBP5sA70)
- [Managed (Lifecycle) Nodes in ROS 2 (Concept + Cpp Code Demo)](https://www.youtube.com/watch?v=axraRVgFRec)
- [ROS Package: lifecycle](https://index.ros.org/p/lifecycle/)
- [ROS2 -Rolling Demo - Lifecucle](https://github.com/ros2/demos/tree/rolling/lifecycle)
- [Nav2 - Repo - Lifecycle](https://github.com/ros-navigation/navigation2/blob/main/nav2_lifecycle_manager/README.md)
- [rclcpp_lifecycle::LifecycleNode Class Reference](https://docs.ros2.org/latest/api/rclcpp_lifecycle/classrclcpp__lifecycle_1_1LifecycleNode.html)
