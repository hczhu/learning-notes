file-created-at:: 2026-09-25

- **Source**: Grady Hillhouse, [“How Do Traffic Signals Work?”](https://practical.engineering/blog/2019/5/11/how-do-traffic-signals-work), *Practical Engineering*, May 14, 2019.
- **One-liner**: Traffic signals increase intersection throughput by allocating right-of-way among conflicting movements; increasingly sophisticated control uses live and network-wide data, but every design still trades efficiency against safety, cost, predictability, and induced demand.

- ## Why Intersections Matter
	- Arterial roads have **interrupted flow** because only compatible traffic streams can cross an at-grade intersection at once.
	- The intersection, rather than lane count or speed limit, is often the road’s capacity bottleneck. Adding lanes upstream may accomplish little if the intersection cannot discharge more vehicles.
	- Intersections also concentrate conflict points and crashes, so maximizing throughput cannot override safety and human factors.
	- Traffic signals are popular because they balance relatively low cost and space requirements with the ability to handle substantial volumes.

- ## Movements, Phases, and Barriers
	- A **movement** is a permitted path through the intersection: typically right, through, or left from each approach, plus pedestrian crossings.
	- A **phase** gives right-of-way to one or more movements that can operate together without conflict—for example, opposing protected left turns.
	- Engineers use **ring-and-barrier diagrams** to define allowable phase sequences:
		- Rings represent sequences of phases that can run concurrently.
		- Barriers force all active phases to finish before conflicting movements begin.
	- A typical sequence might serve major-street left turns, major-street through traffic and pedestrians, clear the intersection, then repeat for the minor street.
	- Protected versus permissive left turns illustrate the core tradeoff: protection improves safety but consumes exclusive signal time and may reduce capacity.

- ## Signal Timing
	- **Green interval**: ideally long enough to discharge the queue accumulated during red. At saturated intersections, longer greens can reduce the fraction of time lost to vehicle startup and phase changes.
	- **Yellow interval**: must allow drivers to perceive the change and stop comfortably; approach speed, slope, and local conditions affect the required duration.
	- **All-red clearance interval**: briefly holds every movement on red so vehicles that entered during yellow can leave the conflict area before another phase begins.
	- Timing is therefore not just about dividing seconds among approaches; it also manages reaction time, stopping distance, queue clearance, and otherwise unused transition time.

- ## Levels of Signal Control
	- **Fixed-time control**
		- Repeats a preset phase sequence and timing plan.
		- Simple and predictable, but cannot respond to short-term changes in demand.
	- **Actuated control**
		- Uses detectors—commonly inductive loops, cameras, or radar—to identify waiting traffic and adjust phase timing or sequence.
		- Can reduce unnecessary waiting, respond to detours and events, and prioritize transit or emergency vehicles.
		- Detection quality matters: small vehicles and bicycles may fail to trigger some inductive loops.
	- **Coordinated control**
		- Times adjacent signals so a group of vehicles, or **platoon**, receives successive green lights along a corridor.
		- Prevents downstream queues from blocking upstream intersections and can increase corridor throughput.
		- Benefits weaken when driveways, businesses, or other interruptions break up the platoon.
	- **Adaptive signal control**
		- Combines detector data from many intersections and continually adjusts network-wide timing using optimization algorithms.
		- Can respond to changing traffic patterns better than isolated controllers, but centralization and connectivity increase system complexity and cybersecurity risk.

- ## Limits and Broader Lessons
	- Intersection control is a network problem: locally optimizing one signal can create queues that reduce the capacity of neighboring signals.
	- Greater road capacity does not guarantee lasting congestion relief. **Latent demand** can fill newly available capacity as travelers shift routes or departure times.
	- The engineering objective is not “eliminate waiting”; it is to allocate unavoidable delay safely and efficiently among vehicles, pedestrians, cyclists, transit, and emergency services.
	- Better sensing and algorithms improve control, but physical geometry, human behavior, conflicting goals, and demand ultimately constrain performance.
