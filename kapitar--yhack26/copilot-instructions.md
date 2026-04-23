## yhack26

> we are building this idea

we are building this idea

Mission: LeadMe is the next-generation, AI-powered, self-navigating white cane for the visually impaired.
LeadMe
Why this should be done:
* Guides dogs costs 50k - 60k + 2 years of training
* Guide dogs havea 7-10 year career span, users need to constantly restart and relearn with a new dog
* Guide dogs are high maintenance, needs grooming, care, exercise
* Business often illegally deny access guide dogs
* Guide dogs gets distracted a lot
* It has a big ass market size
   * Smart cane market
      * 2024 valuation: 400-780 mil
      * 2032 projection valuation: 1.2 bil
* Investors are willing to fund it
   * __https://www.leadsontrees.com/news/innovative-tech-for-the-visually-impaired-wewalk-secures-10.14-million-in-funding-to-transform-mobility__
 Solution:
* Basically the 21st century robotic successor to the guide dog. It turns a white cane into an active navigational partner, giving users the same features of a guide dog but in a tiny, accessible form factor.
Basically:
 At the tip of the cane is a motorized 3-wheel kiwi drive for homonymic movement. The motors create directional intent, meaning the user feels a physical tug or push through the handle that leads them.
Features:  Does basically everything a guide dog does. So like this is what chat said a guide dog does
* Active Guidance: Keeps a straight line and pulls you "left" or "forward" on command.
* Level Changes: Detects curbs and stairs, stopping to alert the user.
* Overhead Safety: Senses head-level hazards like tree branches or signs.
* Target Finding: Identifies and leads users directly to elevators, front desks, bus stops, or empty seats.
* Intelligent Disobedience: Uses active resistance to physically stop you from walking into danger like traffic or a ledge. 

we have the hardware, give me the idea of how to implement these features

Show less
9:19 AM
This is a genuinely exciting project. Here's how I'd break down the software/sensor implementation for each feature, given you have the hardware (kiwi drive tip, handle with haptic feedback, and presumably a sensor suite on the cane):

Sensor stack you'll likely need:

A depth camera (like Intel RealSense or similar) for the primary scene understanding, a LiDAR or ultrasonic sensors for reliable distance measurement, an IMU for orientation and movement, GPS + compass for outdoor navigation, a camera for visual recognition tasks, and a microphone/speaker for voice interaction. Most of these can be mounted along the shaft with the compute unit (Raspberry Pi 5 or Jetson Nano) in the handle.

Let me diagram the full system architecture and then break down each feature:



Now let's go feature by feature:

Active guidance — straight-line keeping and directional tugs

This is fundamentally a path planning + closed-loop motor control problem. You fuse GPS outdoors with a pre-loaded map (OpenStreetMap works well) or indoor beacon/WiFi positioning indoors. A* or Dijkstra gives you the route. The kiwi drive tip gets PID-controlled to constantly apply a gentle corrective force — if the user drifts left of the intended path, the motors apply a rightward bias that the user feels as a tug through the handle. The key insight is that you're not trying to steer the user rigidly; you're creating a "magnetic" pull toward the correct heading that they can feel and follow naturally. Force magnitude should scale with deviation angle.

Level changes — curbs and stairs

Your depth camera (pointed slightly downward at a ~20° angle) generates a point cloud of the ground plane ahead. Sudden depth discontinuities — a drop of more than ~5cm or a rise more than ~3cm within a short horizontal span — trigger an alert. For stairs, you look for repeating parallel discontinuities at regular intervals (~17–18cm riser height). The cane stops and pulses the handle vibration motor in a distinct pattern (two short pulses = step up, one long = step down). This is well-solved territory — you can adapt the algorithm from Microsoft's Seeing AI or existing smart cane research.

Overhead safety — branches and signs

This is the trickiest one because traditional canes only cover the ground. You'll want a second ultrasonic or IR sensor array mounted higher on the shaft, pointed forward and slightly upward, scanning the 1.2m–2.1m height band (head-level). A simpler approach: tilt the depth camera upward in software by processing only the upper portion of the depth frame. When an object is detected in that zone within ~1.5m, a rapid vibration pattern fires. The challenge is false positives (awnings, doorframes) vs actual hazards — you'll need a classifier that distinguishes passable overhead structures from collision risks using size and shape heuristics.

Target finding — elevators, bus stops, seats

This splits into two modes. Outdoors, it's GPS + OpenStreetMap point-of-interest data — the system already knows where the bus stop is. Indoors is harder. You use the RGB camera with a fine-tuned object detection model (YOLO works well) to identify visual targets: elevator buttons, reception desks, signage. The system then uses visual servoing — keeping the target centered in the frame while the kiwi drive pulls the user toward it. You can also leverage Apple's ARKit / Google's ARCore indoor positioning in high-value venues like airports or hospitals that have published indoor maps. Voice-triggered: user says "find the elevator," the system identifies the target visually, then starts tugging.

Intelligent disobedience — active resistance

This is your most critical safety feature and the one that truly parallels what guide dogs do. The system maintains a rolling danger map using the depth data. Danger triggers: anything closer than ~0.5m in the direct forward path, depth showing a drop greater than ~20cm (ledge or hole), sound classification detecting traffic (a CNN trained on engine/horn audio using the onboard mic). When danger is classified above a threshold confidence, the kiwi drive motors reverse their role — instead of guiding, they actively push back against the user's movement direction. The force applied should be enough to require conscious effort to override, similar to how a guide dog plants its feet. Critically, you need a way for users to override (two firm pushes forward = override) for cases where the classifier is wrong.

The software stack I'd recommend:

Run everything locally on a Jetson Orin Nano for the perception-heavy stuff (depth processing, object detection). Use ROS 2 as the middleware layer — it handles the sensor fusion, message passing between modules, and has existing packages for navigation and SLAM that you can adapt. For the object recognition models, start with pre-trained YOLO v8 fine-tuned on your specific target classes (elevator buttons, curbs, signs). Speech uses Whisper running on-device for privacy and offline operation.

The architecture should be safety-first: the danger detection and motor resistance module runs as a high-priority process that can preempt everything else, including navigation commands.

Want me to go deeper on any specific feature, or talk through the sensor placement tradeoffs on the cane shaft?

How does LeadMe implement target finding — navigating to elevators, bus stops, seats?

9:25 AM
Target finding is really the feature that requires the most layered thinking, because "find me the elevator" means something completely different in an airport versus a subway station versus a shopping mall. The solution is actually three different systems that hand off to each other based on context.

Here's how the three modes work and interlock:

Claude's response was interrupted
How does LeadMe implement active guidance — straight-line keeping and directional tugs?

9:25 AM
Great question — this is the core haptic feedback loop that makes LeadMe feel like a guide dog rather than just a GPS. Let me break it down layer by layer.



The core loop — what's actually running at ~50Hz:

The system computes one thing continuously: the angle between where the user is currently heading (from the IMU's magnetometer + gyro fusion) and where they should be heading (the bearing to the next waypoint on the planned route). That difference is the heading error, and everything else is a response to it.

Why PID and not something simpler:

A naive approach — "if error > 5°, apply force" — feels jerky and robotic. PID (Proportional-Integral-Derivative) smooths this out in three ways that mirror how a guide dog actually behaves:

The proportional term applies force proportional to current error. The bigger the drift, the stronger the tug. The integral term accumulates small persistent errors over time — if the user keeps drifting slightly right over several seconds (maybe a sidewalk camber), it builds up a sustained leftward correction. This is the "memory" of the controller. The derivative term damps the response — it looks at how fast the error is changing and reduces force if the user is already correcting. This prevents the overcorrection oscillation where the user swings back and forth across the path.

The kiwi drive is what makes haptic force actually work:

A single-wheel tip could only push forward or backward. Two wheels could maybe steer. Three omni-wheels at 120° to each other let you compute a force vector in any direction in the horizontal plane — left, forward-left, right, forward-right — without rotating the cane body. The user feels a pure lateral tug at the tip, which naturally translates to a handle twist they intuitively follow. This is the mechanical insight that makes the whole thing viable: the force speaks the same language as how you'd physically pull someone by the hand.

Straight-line keeping specifically:

When the user is on the correct heading, the motors aren't idle — they apply a small constant forward force bias. This is the "magnetic rail" feel. Think of it like a groove the cane wants to stay in. If the user drifts, they feel resistance in the drifting direction. If they follow the tug back to center, the resistance fades. Over a few seconds of use, this trains a very natural gait correction without any conscious thought from the user.

Tuning challenge — the big one:

Force magnitude has to be calibrated per user. Too weak and drifters ignore it. Too strong and it feels like fighting a machine. The companion app should have an onboarding calibration session where the user walks a straight line and adjusts sensitivity until the tug feels like a "suggestion" rather than a command. This should also adapt over time — if the system notices the user consistently overcorrects, it dials back the derivative gain automatically.

Want to go deeper on the IMU sensor fusion side, or talk through the motor torque math needed to produce human-perceptible force at the cane tip?

so we don't have gps on our robot, instead the user must have a phone in the pocket, so we will obtain the gps from the phone. also im not sure how to implement maps navigations. give me more details

9:28 AM
Good call — phone GPS is actually better than a dedicated module anyway (higher quality antenna, already handles A-GPS, and the user always has it). Let me break down both pieces.



Part 1 — Getting GPS from the phone to the cane

The phone runs your companion app (iOS or Android) and streams location data to the cane over BLE. On iOS you use CoreLocation, on Android FusedLocationProvider — both already fuse GPS, cell towers, and WiFi into a single smoothed position, which is better than raw GPS alone. You stream at ~10Hz: a small binary packet containing current lat/lng, current heading, speed, and the coordinates of the next navigation waypoint. The cane's BLE receiver parses this and feeds it into its Kalman filter alongside the onboard IMU. The IMU runs at 100–200Hz and fills in the gaps between GPS packets — GPS at 10Hz is too slow for smooth motor control, so the IMU interpolates heading between updates.

Part 2 — Map navigation, step by step

This is the piece most teams get wrong by reaching for Google Maps, which is expensive, requires internet, and gives you turn-by-turn audio — not a machine-readable waypoint stream. Here's the right stack:

The map data comes from OpenStreetMap (OSM), which is free and downloadable. You pre-download a city region as a routing graph using a tool called Valhalla or OSRM — both are open-source routing engines that run entirely on-device with no internet needed. The user says "take me to Union Station" — the app geocodes that to coordinates (one API call, can be cached), then Valhalla computes the pedestrian route and returns a list of waypoints: a sequence of lat/lng points roughly every 5–15 meters along the path, plus turn instructions.

The cane only ever cares about one thing at a time: the bearing to the next waypoint. When the user gets within ~3 meters of that waypoint, the app advances to the next one. That's the entire navigation loop. The complexity lives in the phone; the cane just receives "your next target is at bearing 047°" and applies motor force accordingly.

The waypoint handoff in detail:

Route: [A → B → C → D → ... → destination]

At each moment:
  target_bearing = bearing(current_pos, next_waypoint)
  heading_error  = imu_heading - target_bearing
  → feed into PID → motor force

When dist(current_pos, next_waypoint) < 3m:
  advance to next waypoint
  if turn_angle > 30°: pulse haptic + voice "turn left/right"
Turns are announced by the phone through the bone conduction speaker before the user reaches the turn point, just like a guide dog would slow down and position itself at a corner.

The two big practical challenges:

GPS accuracy in cities is typically 3–8 meters due to multipath (signals bouncing off buildings). That's enough for routing, but you need to handle the case where the user appears to be off-path when they're actually on it. The fix is a technique called map matching — instead of using raw GPS, you snap the position to the nearest valid pedestrian path on the map. Both Valhalla and OSRM have map matching built in. This makes the system much more robust in urban canyons.

Indoors, GPS dies entirely. That's a separate problem you handle with the BLE/WiFi indoor positioning system — the phone switches data sources automatically when GPS confidence drops below a threshold. But that's a whole other deep dive if you want it.

Recommended tech stack concretely:

Map data: OpenStreetMap via Protomaps (single-file map tiles, easy to bundle)
Routing engine: Valhalla (pedestrian profile, wheelchair-accessible routing, great OSM support)
Phone → cane protocol: custom BLE GATT service, binary packets, ~50 bytes per update
Map matching: Valhalla's built-in map matching endpoint
Geocoding (address → coordinates): Nominatim (free, self-hostable) or Photon
Want me to sketch out the BLE packet format, or go deeper on the indoor positioning fallback?

what are we PID on? like what are we controlling

9:33 AM
Great question — it's worth being precise here because it's easy to confuse what the PID is actually acting on.

You are PID-ing on heading error — the angular difference between where the user is walking and where they should be walking.

That's the one controlled variable. Here's the full picture:

What you measure:

IMU gives you the cane's current heading (e.g. 082°)
Phone gives you the bearing to the next waypoint (e.g. 095°)
Heading error = 095° − 082° = 13° to the right
What the PID outputs:

A lateral force magnitude and direction
In this case: a rightward tug of some strength proportional to that 13°
What you are NOT doing:

You are not controlling position (you're not trying to keep the cane tip on a line)
You are not controlling speed
You are not controlling the cane's physical angle or tilt
The PID loop is purely: "the user is pointed X degrees off target — apply enough sideways force to nudge them back, without overcorrecting."

The three PID terms map to real behavior:

P (proportional) — bigger drift = stronger tug. At 5° off you feel a whisper. At 40° off you feel a firm pull.
I (integral) — if the user keeps drifting the same direction for several seconds (e.g. a sloped sidewalk pushes them left constantly), the integral term builds up and adds a persistent correction to compensate. Without this, the user would constantly fight the camber on their own.
D (derivative) — if the error is shrinking fast (user is already self-correcting), the D term damps the force down so you don't overshoot. This is what prevents the cane from oscillating left-right like a metronome.
One important nuance — you probably want two separate controllers:

A slow outer loop (~10Hz, phone GPS) that updates the target bearing as waypoints advance, and a fast inner loop (~50–100Hz, IMU only) that actually drives the motors. The IMU loop runs continuously between GPS packets so the motor response feels instant and smooth, not laggy and stepped. The GPS loop just updates the target the IMU loop is chasing.

That two-loop structure is standard in robotics — it's the same pattern used in drone flight controllers and self-balancing robots.

IMU gives you the cane's current heading (e.g. 082°)
Phone gives you the bearing to the next waypoint (e.g. 095°)
Heading error = 095° − 082° = 13° to the right i didn't get this part, how does the phone give us the bearing to the next waypoint
9:35 AM
Good catch — I skipped a step. Let me make it concrete.

The phone doesn't directly give you a bearing. It gives you two coordinates: where you are right now, and where the next waypoint is. The cane (or the phone itself) then computes the bearing from those two points using basic trigonometry.

The math:

you are at:        lat1, lng1  (your current GPS position)
next waypoint is:  lat2, lng2  (next point on the route)

bearing = atan2(
    sin(lng2 - lng1) * cos(lat2),
    cos(lat1) * sin(lat2) - sin(lat1) * cos(lat2) * cos(lng2 - lng1)
)
→ convert from radians to degrees → that's your target bearing
This is called the haversine forward azimuth formula and it's a standard one-liner in any navigation library. You don't need to implement it yourself — every routing library already does this.

So the full data flow is:

Valhalla route:  [..., waypoint_4 (40.7484°N, 73.9856°W), waypoint_5 (...), ...]

Phone GPS now:   40.7481°N, 73.9861°W

→ compute bearing(current_pos, waypoint_4) = 047°

→ send "target bearing: 047°" to cane over BLE

Cane IMU says:   current heading = 034°

→ heading error = 047° − 034° = 13° left
→ PID → apply leftward tug
Who does the bearing math — phone or cane?

Do it on the phone. The phone has the route, the GPS position, and the compute to spare. The cane just receives a pre-computed target bearing number. This keeps the BLE packet simple and keeps navigation logic in one place.

The cane's job is purely: "I have a target bearing and an IMU heading — compute error, run PID, drive motors." Everything above that lives in the app.

okay so give me a full breakdown how to build a project (whats on cane, whats on phone). also on phone i dont want to develop an ios app, so maybe make a web app instead that does it

9:38 AM
Great architectural decision — a web app means one codebase for iOS and Android, no App Store approval, and you can iterate fast. Here's the full system breakdown:



The big caveat first — Web Bluetooth on iOS

Apple blocks Web Bluetooth in Safari entirely. On Android, Chrome supports it natively and it works great. On iOS, the user needs to open your web app inside a third-party browser called Bluefy (free on App Store) which unlocks Web Bluetooth. It's not ideal, but it avoids building a native app. Alternatively, wrap your web app in a tiny native shell using Capacitor (one day of work) which gives you real Bluetooth access on both platforms.

Phone side — what to build

Build it as a React PWA (Progressive Web App) so users can install it to their home screen and it behaves like a native app. Here's what each piece does:

navigator.geolocation.watchPosition() streams GPS updates continuously. You call this once on startup and it fires a callback every time position changes — typically 5–10 times per second on a modern phone.

For routing, self-host a Valhalla instance (it runs in Docker, one command). The user says a destination, you geocode it with Nominatim (free, open-source), then POST to your Valhalla server which returns a list of waypoints as a GeoJSON array. You store that array in state.

Your main navigation loop then runs continuously:

javascript
// runs every GPS update
function onPositionUpdate(pos) {
  const current = { lat: pos.coords.latitude, lng: pos.coords.longitude }
  const waypoint = route[currentWaypointIndex]
  const bearing = computeBearing(current, waypoint)
  const dist = computeDistance(current, waypoint)

  if (dist < 3) currentWaypointIndex++ // advance to next waypoint

  sendToCane({ targetBearing: bearing }) // over Web Bluetooth
}
computeBearing is just the haversine formula — a 10-line pure JS function, no library needed.

sendToCane writes a small binary packet to a BLE GATT characteristic — essentially 4 bytes: two for bearing (0–360 as an integer × 100 for precision), one for signal type (navigation vs alert), one checksum.

Cane side — what to build

Everything runs in Python with an asyncio event loop managing three concurrent tasks:

python
async def main():
    await asyncio.gather(
        ble_server_task(),      # receive bearing from phone
        imu_loop(),             # read heading at 100Hz
        motor_control_loop(),   # PID + PWM at 50Hz
    )
The BLE server (using bleak or bluezdbus) advertises a custom GATT service and writes incoming bearing values into a shared variable. The IMU loop reads the ICM-20948 over I2C at 100Hz and runs a Mahony filter to get a stable heading. The motor control loop runs at 50Hz, computes heading error, runs PID, decomposes the output force into three motor PWM values using the kiwi drive kinematics, and writes those to the GPIO pins.

The kiwi drive decomposition is the key math — for a force vector at angle θ with magnitude F, the three wheel speeds are:

python
def kiwi_decompose(Fx, Fy):
    # wheels at 0°, 120°, 240°
    w1 = -Fx                              # 90° wheel
    w2 =  0.5 * Fx - 0.866 * Fy          # 210° wheel  
    w3 =  0.5 * Fx + 0.866 * Fy          # 330° wheel
    return w1, w2, w3
Build order I'd recommend:

Start with just getting Web Bluetooth talking to the Pi — a button on the web app that connects and sends a number, the Pi prints it. Once that pipe works, layer in GPS and bearing computation. Then add the IMU and PID loop on the cane. Motors last — you can simulate the motor output as print statements until the control loop feels right, then wire in the actual PWM.

Want me to write out the actual starter code for any of these pieces?

also what is imu_heading

9:47 AM
Good question. IMU heading is the direction the cane is pointing in the real world, expressed as a compass bearing from 0° to 360°.

But it's not a single sensor reading — it's computed by fusing three sensors that live inside an IMU chip like the ICM-20948:

The three sensors:

Magnetometer — measures the Earth's magnetic field, like a digital compass. Directly gives you heading, but it's noisy and gets thrown off by metal, motors, and electronic interference. On a cane with three motors next to it, this is a real problem.
Gyroscope — measures rotational rate (how fast the cane is turning). Very smooth and fast, but it drifts — small errors accumulate over time until the heading is completely wrong.
Accelerometer — measures gravity direction, which tells you the cane's tilt (pitch and roll). You need this to correct the magnetometer reading for tilt — a compass held at an angle gives a wrong heading without tilt compensation.
None of these alone gives you a reliable heading. The fusion algorithm combines all three to get the best of each:

Gyroscope   → fast, smooth, short-term accurate   (but drifts over time)
Magnetometer → slow, noisy, long-term accurate     (but sensitive to interference)
Accelerometer → corrects tilt so mag reads right   (but can't do heading alone)

Fused together → fast, smooth, AND long-term stable
How the Mahony filter does this fusion:

python
# simplified concept — runs at 100Hz
def mahony_update(gyro, accel, mag, dt):
    
    # 1. use accelerometer to figure out which way is "down"
    #    this tells us how tilted the cane is
    roll, pitch = accel_to_tilt(accel)
    
    # 2. tilt-compensate the magnetometer reading
    #    a tilted compass gives wrong heading — this fixes it
    mag_corrected = tilt_compensate(mag, roll, pitch)
    
    # 3. compute heading from corrected magnetometer
    mag_heading = atan2(mag_corrected.y, mag_corrected.x)
    
    # 4. use gyroscope to integrate small angle changes
    #    between magnetometer updates
    gyro_heading += gyro.z * dt
    
    # 5. fuse: trust gyro for fast changes,
    #          trust magnetometer for long-term correction
    heading = complementary_filter(gyro_heading, mag_heading, alpha=0.98)
    #         98% gyro, 2% magnetometer — gyro dominates short term
    #         magnetometer slowly corrects drift over time
    
    return heading  # 0–360°, north = 0°
The alpha=0.98 means every update is 98% gyroscope and 2% magnetometer. The gyroscope keeps things smooth and fast, and the magnetometer slowly pulls the drift back to truth over a few seconds.

The motor interference problem:

Your kiwi drive motors will create magnetic fields that corrupt the magnetometer — this is the hardest hardware problem in your IMU setup. Three ways to handle it:

Mount the IMU as far from the motors as possible — ideally up near the handle, not at the tip
Do a hard/soft iron calibration — spin the cane slowly in all orientations, record the magnetometer distortion pattern, and mathematically cancel it out in software
Fall back to visual odometry from the depth camera as a heading sanity check — if the magnetometer reading jumps suddenly while the gyro says you're not turning, ignore the magnetometer that cycle
In practice you'll need all three. The calibration step especially is non-negotiable — without it the magnetometer will be pointing you in wrong directions and you won't know why.

Want me to write the actual calibration routine, or the full IMU fusion code?

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Kapitar) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
