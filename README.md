AI Programming – Unreal Engine AI Guards



This project features AI guards using:



* Behavior Tree
* Blackboard
* Animation Blueprint
* Waypoint patrols
* Pawn Sensing
* Hearing detection
* Sight detection
* Chase behavior
* Team alert 



The Project includes 2 guard types:

* BP\_Guard : sight based guard
* BP\_HearingGuard: hearing based guard



BP\_HearingGuard is a child Blueprint of BP\_Guard, so both guards share the same base patrol, chase, Blackboard, Behavior Tree, animation, and team alert logic.



**Behavior Tree BT:** Both guards use the same Behavior Tree 



the BT Controls which action the guard should perform depending on the Blackboard values. it uses a Selector with 2 branches: Chase and patrol.

chase:

if the hasDetectedPlayer (Blackboard decorator) is true the guard runs the Chase branch which uses a Move To and TargetActor as the destination,if the player is no longer detected for 2s the hasDetectedPlayer = false, target actor = none, the conditions for chase are not met, the BT exits the chase sequence and returns to patrol mode. 

Patrol:

this sequence only runs when hasDetectedPlayer is false. and it uses the BTTask\_SetNextWaypoint.



**BlackBoard BB:** Both guards use the same Blackboard.



the BB stored values Control the BT decisions and are updated by the guard Blueprint and AI services.



BB Keys:

**KEY			TYPE		PURPOSE**

**HasDetectedPlayer**	Bool		Controls whether the guard should chase or patrol.

**TargetActor**		Object/Actor	Stores the player actor that the guard should chase.

**CurrentWaypoint**		Object/Actor 	Stores the current patrol waypoint.





AI service:

BTService\_UpdateGuardBlackboard



**Animation Blueprint BP\_GuardAnim: one animation for both guards** 

is controlled with 2 variables speed and IsChasing.

the speed updates every frame using the guards movement velocity:

Get Velocity	 

&#x09;-> Vector Length
	-> set speed

speed value determines if the guard is standing or moving.

speed = 0  Idle

speed > 0  movement animation



the IsChasing boolean Controls if the guard should walk or run, and it updated using the guards hasDetectedPlayer value.

hasDetectedPlayer = true -> IsChasing = true, which causes the animation state machine to transition  into running animation during chase behavior.



Animation state Machine:

it has 3 transitions

* idle
* walk
* run



idle to walk: 

speed > 10 



walk to idle

speed < 10



Walk to Run:

IsChasing = true



Run to walk:

Ischasing = false



idle to run:

speed > 10 AND IsChasing = true



run to idle:

speed < 10 And IsChasing = false





**Gaurds:**

2 types of guards using pawn sensing, BP\_Guard uses sight to detect the player (with the hearing pawn ability beeing turned of of this type of guard), whereas BP\_HearingGuard uses hearing (with the see pawn ability beeing turned of for it).



**Sight Guard:**

BP\_Guard uses sight detection with Pawn Sensing.

PawnSensing settings has:

See Pawns = true

Hear Noises = false

SightRadius = 	1000, When the player enters the guard’s sight radius, the On See Pawn event is triggered.this sets:



HasDetectedPlayer = true

TargetActor = Player



The guard then enters the Chase branch in the Behavior Tree.



If the player leaves the sight detection for 2 seconds, the guard loses interest and returns to patrol.





**Hearing Guard:**

BP\_HearingGuard uses hearing detection with Pawn Sensing and is a child Blueprint of BP\_Guard, so it reuses the same base guard logic.

PawnSensing setting has:

See Pawns = false

Hear Noises = true

Hearing Threshold = 400

LOSHearing Threshold = 500



The player makes noise while jumping,using ***Make Noise***. The player also has a ***PawnNoiseEmitter*** component so Pawn Sensing can detect the noise. When the hearing guard hears the player, the On Hear Noise event is triggered.This sets:



HasDetectedPlayer = true

TargetActor = Player



The hearing guard then chases the player.

If the player stops making noise or the guard does not continue detecting the player, the hearing guard loses interest after 2 seconds and returns to patrol.



This creates a different perception response from the sight guard:



the sight guard reacts when it sees the player

the hearing guard reacts when the player makes noise




BP Gaurd event Graph has Three events:



1. patrol:
is controlled by behavior tree and waypoint system. each guard has an Array of waypoints and an current waypoint index. the behavior tree runs a BTTask\_setNextWaypoint. 
the task:
1. gets the next waypoint from the guards Array list
2. stores  the waypoint in the blackboard
3. updates the current waypoint
4. loops back to the first waypoint when the last waypoint is reached.
after the waypoint is set the Ai uses the Move To, to move the guard. the patrol contiues unless the palyer is detected. (BT logic)

2. Chase the player:
is controlled by behavior tree and blackboard. when the player is detected with On See pawn or On Hear Noise the guard updates the blackboard values, HasDetectedPlayer = true, and TargetActor = Player. the BT has a chase sequence with blackboard detector HasDetectedPlayer is Set. the condition are true the BT switches from patrol to chase.


   1. Team alert:
allows the guard to communicate player detection information with each other.
when the sight guard detects player it alerts the other guards using: 
Get all Actors of Class BP\_Guard 
-> ForEachLoop
->AlertGuard 

The AlertGuard is a custom event that sets (updates):
HasDetectedPlayer = true and TargetActor = Player
This causes alerted guards to also enter chase behavior.



In this project, the team alert system is implemented from the sight guard. When the sight guard sees the player, all guards are alerted and start chasing. When the hearing guard detects the player, the other guard is not alerted by design.









