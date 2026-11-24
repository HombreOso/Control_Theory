# Modern Robotics Ch. 9: why straight-line paths, and are they always possible?

Source: books/Modern_Robotics_book_2017.pdf, Chapter 9 (book pages 325–329). The relevant section is 9.2.1 “Straight-Line Paths”; page 328 is its last page.


## 1. Short answers

- Is the decoupled straight-line motion always possible? As a mathematical path: almost always. Eq. 9.7 (a straight line for the origin) is defined for any two positions, and Eq. 9.8 is defined for any two orientations (the only subtlety is a relative rotation of exactly 180°, where the axis of the logarithm is not unique). As a motion the robot can actually execute: no. The book lists the reasons on p. 327 (see section 4).
- Why is it in the book? Because Chapter 9 builds a trajectory from a path plus a time scaling, and the simplest path between two points is a straight line. Section 9.2.1 asks what “straight line” should mean in joint space, in task-space coordinates, and finally in SE(3). Page 328 is the last step of that chain, the SE(3) case.

## 2. The logical chain leading to page 328 (what the book actually says)

- p. 325, intro and 9.1: a trajectory = a path (pure geometry, θ(s), s from 0 to 1) + a time scaling s(t) (when each point is reached). Separating them lets you design the geometry first and the timing second.
- p. 326, 9.2: “The simplest type of motion is from rest at one configuration to rest at another. We call this a point-to-point motion. The simplest type of path for point-to-point motion is a straight line.” This sentence is the reason the straight line appears.
- p. 326, joint space (Eq. 9.3–9.5): θ(s) = θ_start + s(θ_end − θ_start). Advantage: joint limits form a convex set, so the line between two feasible configurations is always feasible. Drawback: “Straight lines in joint space generally do not yield straight-line motion of the end-effector in task space.”
- p. 326–327, task space with minimum coordinates: if a straight line of the end-effector is wanted, give X_start and X_end in task space and use X(s) = X_start + s(X_end − X_start). New problems: (i) singularities, (ii) the reachable set is not convex (Fig. 9.1).
- p. 327–328, the bridge to the page you read: “if X_start and X_end are represented as elements of SE(3) … there is the question of how to define a straight line in SE(3). A configuration of the form X_start + s(X_end − X_start) does not generally lie in SE(3).” A sum of rotation matrices is not a rotation matrix (SE(3) is a curved manifold, not a vector space), so the simple formula breaks.
- p. 328: two ways to repair it. Option 1: the screw motion, Eq. 9.6, constant screw axis (uses the exp/log machinery from Chapter 3). Option 2: decouple, Eq. 9.7–9.8, a straight line for p and a constant-axis rotation for R.
- p. 328, 9.2.2 right after: the time scaling s(t) is applied to whichever path was chosen (cubic, quintic, trapezoidal on pp. 329–333). This is the payoff: a line in s makes velocity = (ds/dt) × constant, so joint or Cartesian limits become simple bounds on ds/dt and d²s/dt².
The transition you missed is therefore the jump from “straight line in a vector space” (Eq. 9.3 and the X coordinates) to “straight line in SE(3)”, which is not a vector space. The book spends only one paragraph on this, which is why it feels abrupt.


## 3. Where the straight-line idea is used later in the book

- Appendix A summary (p. 573) and the code interface list (p. 347): the function that generates a task-space trajectory has the origin following a straight line “decoupled from the rotation”.
- Chapter 10 (motion planning): straight-line segments are used as the simple local planner in roadmaps and RRT trees (pp. 366–384).
- Chapter 11 (p. 420): the task-space motion control law decouples linear and angular control, visible as a straight-line motion of the end-effector origin (Fig. 11.10).

## 4. When the decoupled straight line is NOT achievable (from p. 327 and the chapter)

- Kinematic singularities near the path: the joint velocities needed to follow it can become unreasonably large for almost any time scaling (p. 327).
- Non-convex reachable workspace: two reachable endpoints can be joined by a line that leaves the workspace (Fig. 9.1, bottom): the dashed task-space line would violate the joint limits (p. 327).
- Joint limits (Fig. 9.1): a straight line in joint space always respects them, but a straight line in task space may not.
- Additions of mine, not stated on these pages: obstacles and self-collision along the line (the book leaves obstacle avoidance to Chapter 10), and the choice among multiple inverse-kinematics solutions along the path. Also, a path that is feasible geometrically may still violate velocity, acceleration or torque limits unless the time scaling is slow enough.
- Practical rule: use the joint-space line when you only need a safe, simple move. Use the task-space (decoupled or screw) path when the tool must follow a line or when the Cartesian geometry matters, and check it for singularities and limits.

## 5. Correction to my previous answer

In the saved note on p. 328 I called the section “9.1” and referred to time scaling as “Section 9.2”. The correct numbers are 9.2.1 (Straight-Line Paths, which contains Eq. 9.6–9.8) and 9.2.2 (Time Scaling a Straight-Line Path). I have corrected that file.

