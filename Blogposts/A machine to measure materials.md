# Foreword
A bit over a decade ago, I wrote my first [path tracer](https://en.wikipedia.org/wiki/Path_tracing). Ever since then, [light transport](https://en.wikipedia.org/wiki/Light_transport_theory), the mathematical theory underpinning simulation of realistic lighting with [global illumination](https://en.wikipedia.org/wiki/Global_illumination), has been an area of great interest for me. Some years later, this interest even inspired me to start working as a graphics programmer specialized in the topic!

Early in my learning, I remember wondering how the various material and shading models I stumbled upon were invented, and found that many are essentially conveniently sampleable approximations of real-world observations. At some point I learned of a type of device for making these real-world observations, called a [*gonioreflectometer*](https://en.wikipedia.org/wiki/Gonioreflectometer). This device fascinated and intrigued me, but some googling revealed that these are [unobtainium](https://en.wikipedia.org/wiki/Unobtainium) unless you have deep pockets or work at a university, so my fascination eventually waned.

Fast forwarding to the present, I've been working on [yet another path tracer](https://github.com/pema99/odin-pt), with a focus on spectral rendering and advanced material models. Hacking away at this reminded me of the device that intrigued me all those years ago. I've been meaning to get some more hands-on experience with hobby electronics, so I decided to try creating such a device for myself. In this blog post, I want to share my experience and learnings from working on it.
# What is a gonioreflectometer?
In a single sentence, a gonioreflectometer is a device that measures the [Bidirectional Reflectance Distribution Function](https://en.wikipedia.org/wiki/Bidirectional_reflectance_distribution_function) (often abbreviated BRDF) of a material sample. So, what is a BRDF?

The term is often used by graphics programmers in less-than-rigorous ways, but the classic definition is a function of 2 inputs - an *incident light direction* $\omega_i$ and a *view direction* $\omega_r$. This function answers the question the "how much light is reflected towards my eye from a point on a surface, given the directions from the point towards a light source, and towards my eye". Hence, the result of evaluating a BRDF is a value in range [0; 1]. You can think of a materials BRDF as a mathematical description of how the material interacts with light, and thus its appearance.
![[Pasted image 20261029033731.png]]

There are several different types of gonioreflectometer, but the majority of them work by systematically moving a *sensor* and a *light source* to capture measurements of the sample from different view and incident light directions. One way to think of a BRDF is as a 4 dimensional function - each of the 2 input directions can be represented with no less than 2 scalars (for example with [spherical coordinates](https://en.wikipedia.org/wiki/Spherical_coordinate_system)). Therefore, to measure an arbitrary BRDF, a gonioreflectometer needs 4 degrees of freedom - 2 for the sensor and 2 for light source.
# Building the machine
Building a mechanism for moving both a sensor and a light source, with several degrees of freedom, and within the same physical space is above my skill level, so I spent some time researching different types of gonioreflectometer. Eventually I stumbled upon Felix Schröder's masters thesis [Construction of a gonioreflectometer to capture and render BRDFs](https://f-schroeder.github.io/publication/masterthesis/) which presents a very simple and cheap design that achieves all 4 degrees of freedom. My build is largely based on their design, though with some tweaks and improvements.

The device consists of 3 main parts: A turntable, a camera attached to a movable arm, and a dome with an LED matrix attached to the inside. The turntable and the camera arm provide the 2 degrees of freedom for the sensor, which is in this case a [Raspberry PI HQ Camera](https://www.raspberrypi.com/products/raspberry-pi-high-quality-camera/). The dome is placed above the turntable, and by toggling individual LEDs in the matrix, I achieve 2 degrees of freedom for the light source, though with some quantization error due to the relatively sparse layout of LEDs.![[Pasted image 20261029041310.png]]

In the sections below, I will show how I built the machine, part by part. This will be structured as a sort of build log, with plenty of progress images along the way. If you aren't interested in the build process, you can skip to the [[#Exploring usecases]] section, in which I discuss how and what I used the machine for.
## Testing electronics
The machine is driven by a few different electronic components. It seemed obvious to configure and test these as the first step, starting with the motor assembly. As in Schröders' thesis, I used an [Arduino Uno](https://docs.arduino.cc/hardware/uno-rev3/) with a [A4988 stepper motor driver shield](https://www.amazon.de/dp/B0D91JCWQG) to control the motors for the arm and turntable. 

The drivers require 12 volt DC. I used a cheap ATX power supply with a modified molex connector.
![[Pasted image 20261029023827.jpg|332]]![[Pasted image 20261029023758.jpg|332]]
For the turntable, I used a cheap [17HS15-1206S](https://www.omc-stepperonline.com/nema-17-unipolar-1-8deg-26ncm-36-8oz-in-1-2a-4v-42x42x39mm-6-wires-17hs15-1206s) stepper motor. I initially tried to use the same model for the camera arm, but found that it didn't have enough torque to lift the arm. Instead, I opted for a pair of [17HS19-1684S-PG5](https://www.omc-stepperonline.com/nema-17-stepper-motor-bipolar-l-48mm-w-gear-ratio-5-1-planetary-gearbox-17hs19-1684s-pg5). These are similar motors, but the latter has an integrated planetary gearbox with a 5:1 gear ratio. 

The steppers are rated for 1.2 and 1.68 amps respectively, though the drivers can get quite hot at higher currents. The [driver's datasheet](https://cdn.sparkfun.com/datasheets/Robotics/A4988-Datasheet.pdf) has a nifty formula for calculating the max current:
$$
I_{TripMAX} = \frac{V_{REF}}{(8 \cdot R_S)}
$$
Where $R_S$ is the resistance of the "sense resistor" (0.1ohm on my board), and $V_{REF}$ is the voltage measured between each driver's potentiometer and ground. I used $V_{REF}=0.8$v for the arm motors and $V_{REF}=0.32$v for the turntable, which has a much smaller load. Plugging in the numbers, I get a current of $1.0$ and $0.4$ amps respectively, well within rating for the motors.
![[Pasted image 20261029023833.jpg|653]]
Stepper motors work by switching between an array of electromagnets to move the shaft. We call the transition from one set of magnets to the next in the switching sequence a step. On my turntable motor, that corresponds to a 1.8 degree rotation. The drivers support _microstepping_, which allows for smoother and more granular rotations by powering two sets of magnets at once with varying amounts of current. This is desirable for my use case, so I configured the drivers for 1/16th microstepping by bridging the MS1, MS2, MS3 pins to 5V with solder.
![](https://electricdiylab.com/wp-content/uploads/2019/12/STEPPER-2.gif)
Image source: https://electricdiylab.com/how-stepper-motor-works/

With the drivers configured, I uploaded a quick test script to the turn my turntable stepper one full rotation.
![[Pasted image 20261029023844.jpg|644]]

To keep the assembly cool, I connected a cheap PC fan via molex to my PSU.
![[Pasted image 20261029024207.jpg|597]]

Next, it was time to test the camera. I flashed a fresh Raspbian install onto my PI, plugged in the camera to the CSI port, and took a picture using [picamera2](https://github.com/raspberrypi/picamera2). The camera module comes without a lens, I used an [official 6mm wide angle](https://www.kubii.com/en/objectives-supports/2952-official-6mm-wide-angle-lens-3272496301498.html) one.
![[Pasted image 20261029024218.jpg|337]]![[Pasted image 20261112013249.png|333]]
## The turntable and frame
With the electronics confirmed working, I started working on the turntable assembly. To save some hassle, I bought a pack of [circular MDF cutouts](https://www.amazon.de/dp/B0DVGZYCYV?ref=ppx_yo2ov_dt_b_fed_asin_title), and stacked a few of them together using glue. To attach it to the stepper, I glued a [flange coupling](https://www.amazon.de/dp/B0993M969V?ref=ppx_yo2ov_dt_b_fed_asin_title) to the center of the underside.
![[Pasted image 20261029023751.jpg|564]]

For the main frame, I bought a [steel plate](https://www.amazon.de/dp/B088FNL29V) with a grid of pre-drilled holes, for easier mounting of components. The turntable stepper sits on the underside of the plate, though the shaft wasn't thin enough to pass through the holes, so I had to widen one a bit with a rotary tool.
![[Pasted image 20261029023849.jpg|312]]![[Pasted image 20261029023856.jpg|315]]

I made some simple feet for the frame using a cheap [pack of L-bracket](https://www.amazon.de/dp/B0D6YTK3KT) and some nuts and bolts.
![[Pasted image 20261029024036.jpg|349]]![[Pasted image 20261029024100.jpg|350]]

With these pieces prepared, I assembled the frame. I used some [Nema 17 brackets](https://www.stepperonline.co.uk/nema-17-bracket-for-stepper-motor-and-geared-stepper-motor-alloy-steel-bracket-st-m1.html) to mount the 2 arm motors to the plate, about 45cm apart, aligned with the turntable shaft.
![[Pasted image 20261029023911.jpg|337]]![[Pasted image 20261029023905.jpg|338]]
## The camera assembly
The next missing piece was the arm which attaches the camera to the steppers. Clearance was pretty important here, as the entire assembly will be covered with a dome. I planned for the domes diameter to be 60cm, and used some cardboard to make a rough prototype of the arm shape that I was confident would fit inside the dome. 
![[Pasted image 20261029023942.jpg|639]]

I cut a 20x20mm hollow [aluminium rod](https://www.amazon.de/dp/B008GZNF62) to the appropriate length, then cut grooves at each of the arms bends, using a rotary tool. Aluminium is soft, so the rod was easy to bend into shape by hand. I mostly eyeballed this, though I used a paper cutout to check that all of my angles were matching expectations.
![[Pasted image 20261029023946.jpg|330]]![[Pasted image 20261029023957.jpg|331]]

To make the arm a bit more sturdy, I cut a flat [aluminum rod](https://www.amazon.de/dp/B0054M2DRM) into short pieces, drilled 2 holes in each piece, and corresponding holes through the arm. I then used some nuts and bolts to fasten each arm segment together. I attached some more [flange couplings](https://www.amazon.de/dp/B0833PM19X) to the base of the arm, to and used them to attach the arm to the steppers. This was again mostly eyeballed.
![[Pasted image 20261029024011.jpg|649]]
> Note: You may notice these steppers do, in fact, *not* have planetary gearboxes attached. This image was taken moments before I realized I hadn't properly calculated the torque required by my motors, and had to buy a new set. 😅 

I needed a mechanism to attach the camera to the top of the arm, so I improvised a bit and used a cutout of one of the MDF circles. I drilled 2 holes through it for mounting, and 4 more for screw standoffs for the camera PCB.
![[Pasted image 20261112020229.png|311]]![[Pasted image 20261029024030.jpg|332]]
I used a few [nylon M2.5 screws](https://www.amazon.de/dp/B0DCS5C7SN) to attach the bracket to the arm, and additionally attached some nylon standoffs from the same pack.
![[Pasted image 20261112020952.png]]

Finally, to avoid damaging the arm, and for calibration purposes, I needed a way to detect when the arm was at the maximum possible rotation in either direction. For this, I attached some [end-stop microswitches](https://www.amazon.de/dp/B07VWRS24K) to the plate on either side of the arm, about where the arm would touch the plate. I used some cotton pads to avoid shorting these against the metal plate.
![[Pasted image 20261029024223.jpg|324]]![[Pasted image 20261029024229.jpg|326]]
These end stops have 3 wires - VCC, ground and signal. Each of these plug directly into the appropriate end-stop pins on the stepper driver shield. They even have a nice little LED that is on when the switch is pressed. VCC and ground connect to the 5V rail and ground on the Arduino, and the signal wire  connects to the appropriate signal pin on the stepper driver shield. They even have a nice little LED that lights up when triggered.

With this done, I was ready for a first arm movement test. I was a bit worried I would get the rotation direction wrong for one of the steppers and damage the arm, so I made to triple-check the wiring and code. To my slight surprise, it worked great on first attempt.
![[VID20261001001537 (2).mp4]]
Next, I mounted the camera and hooked up the Arduino the Raspberry PI. 
![[Pasted image 20261029024151.jpg|639]]
With that done, I had a working 2-degree-of-freedom camera setup! Here are some videos from my first tests with it.
![[VID20261002211142.mp4]]
![[2026-10-02 21-04-11 - Trim (2) - Trim - Muted.mp4]]
## The LED dome
With the camera portion finished, and 2 of the 4 required degrees of freedom covered, I started working on the dome, which will cover the whole assembly, and provide the remaining 2 degrees of freedom via a matrix of LEDs. 

Suitable 60cm plastic domes are surprisingly hard to come by, so I borrowed the method Schröder used in their thesis to make my own. I bought a 60cm inflatable yoga ball, and a 60cm paper-lantern-lampshape. I inflated the yoga ball inside the lamp to hold it in place, and placed the ball on a little bowl. I then started layering strips of wetted [gypsum gauze](https://www.amazon.de/dp/B07GZTJQ5F), similar to papier-mâché. To make the the dome as robust as possible, I applied several overlapping layers of gauze once the first layer dried.
![[IMG20261003183312.jpg|309]]![[IMG20261003193602.jpg|309]]

Once fully dried, I used my rotary tool to cut them dome into 2 halves.
![[Pasted image 20261112023223.png|373]]![[Pasted image 20261112023244.png|325]]

I then painted the inside of the dome black with some acrylic paint.
![[IMG20261004193305.jpg|608]]

At this point I noticed the dome was a bit too small, and was actually less than a half sphere. I had mistakenly trimmed too much of the gypsum off while cutting it into shape - woops. I improvised a bit here with some more gypsum gauze, and applied another thing band attaching to the rim of the dome, in order to recover some of the trimmed surface. This seems to have worked well enough.
![[Pasted image 20261112023641.png]]

After painting the added gypsum layer black, the next step was to attach the LED matrix. I decided to use the same layout of LEDs as Schröder, with 809 LEDs in total. I bought 3 [WS2812B based addressable LED strips](https://www.amazon.de/dp/B088BPGMXB) and cut them into shorter strips, with 32 segments in total. Getting the lengths, layout and orientation of each strips right was a bit tricky, so I made a diagram help me visualize it.![[Pasted image 20261112024505.png]]
All of the strips are daisy chained together into what is essentially 1 long 809-LED strip. The arrows show which direction the data flows in. Notice that adjacent strips have opposite data directions. These allows for easy wiring when daisy-chaining, like so:
![[Pasted image 20261112025041.png]]

Here's how the dome looked with the strips attached:
![[IMG20261008220214.jpg|681]]
The positive and ground terminal of each strips needs to be connected together. I did this by soldering a bunch of small wires near the rim of the dome (red and black). Additionally, every adjacent pair of strips has their data terminals chained near the rim (yellow).![[Pasted image 20261112025152.png]]
The data terminals near the apex of the dome also need to be connected, as per the diagram. I did this with what I can only described as a clusterfuck of dense wiring. I did my best to bend the wires such that no LEDs were occluded by wiring, then used some hot glue to hold the wires in place.
![[Pasted image 20261112025354.png|636]]
The finished dome looks like this. The whole thing is connected to the Arduino with a single 3 pin terminal (VCC, ground, data).
![[IMG20261014232819.jpg|633]]
And here's a video of me testing it with a cool animated rainbow pattern:![[VID20261010221130_cropped.mp4]]
## Finishing touches
With the electronics, frame and dome all prepared, the machine was almost finished. All the remained was some polishing and cleanup steps. First, I painted everything black and attached the ribbon cable for the camera permanently to the arm with some glue.
![[IMG20261007200726.jpg|615]]

Next, I mounted the frame onto a large MDF plate to keep everything organized. I also made some feet for the dome to stand on, using L-brackets and bolts, similar to the frame's feet.
![[IMG20261015012845.jpg|619]]

I used some zipties to do cable management, and mounted the electronics to the MDF plate using some screws and standoffs. Additionally, I painted the area around the frame black, as light can reflect onto and into the dome.
![[Pasted image 20261112030315.png]]

To keep it in more or less the same position and orientation every time, I drew some lines on the dome and the feet that should line up. Here's how the finished machine looks, with the dome resting on its feet.
![[Pasted image 20261114184057.png]]
# Exploring usecases
With the build out of the way, I want to discuss how I used the machine, and the various use cases I found for it. Before we get the fun stuff, though, I'll talk a bit about the software the powers the machine.
## Plumbing
To make the motors and LEDs do anything, I first had to make some (TODO link) firmware for Arduino to run. The Raspberry PI, which will be the main 'brains' of the machine, must be able to send commands for the Arduino to execute. I used a simple little protocol for communication over serial, with various useful commands, mostly taken from Schröder's thesis. Each command consists of a 2 character identifier and optionally an operand, followed by a semicolon. The commands are listed below.

| Command                     | Description                                                                                                                                                          |
| --------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ar;                         | Move the arm to the "home position", i.e. until it hits one of the 2 endstops.                                                                                       |
| aa\<degrees\>;              | Move the arm to an absolute angle specified in degrees.                                                                                                              |
| ap;                         | Print the current arm angle in degrees.                                                                                                                              |
| es;                         | Print the current state of both end stops, whether they are pressed or not.                                                                                          |
| da\<degrees\>;              | Turn the turntable to an absolute angle specified in degrees.                                                                                                        |
| dm\<degrees\>;              | Turn the turntable by a relative angle specified in degrees.                                                                                                         |
| dp;                         | Print the current turntable angle in degrees.                                                                                                                        |
| li\<index\>\<rgb triplet\>; | Set the color of the LED at the given index to an RGB value, and turn every other LED off. For example `li192232126089;` set LED `922` to `r=232`, `g=126`, `b=089`. |
| la\<rgb triplet\>;          | Set the color of every LED to the given RGB value. For example `la255255255;` set every LED to pure white i.e. `r=g=b=255`.                                          |
| !                           | Special command that stops whatever is currently executing. Like a cancel button.                                                                                    |
To be able to talk about arm positions in degrees, I needed to know the angle of the arm at the 2 endstops, since the whole range of the arm is actually greater than 180 degrees. To do that, I moved the arm to one endstop, which I designated as the "home" position. I then rotated the arm until it hit the other end stop, and counted how many (micro)steps the motor took, which turned out to be 8530. One full rotation of the stepper is 200 full steps, so accounting for the 5:1 gear ratio, it is $200*16*5=16000$ microsteps, or $16000/360\approx 44.44$ microsteps per degree. With 8530 total steps, the arms range is $8530/44.44\approx 192$ degrees, which puts the endstop positions at $\pm192/2=\pm96$ degrees. Plugging in those values, and moving the arm to 0 degrees absolute, resulted in the arm standing perfectly vertical 😃.

Driving the LEDs introduced some additional complexity. The timing required to drive the WS2812B LED strips is quite precise, precise enough that Arduino's built-in `digitalWrite` is too slow. I found and yoinked some code from Josh Levine's [SimpleNeoPixelDemo](https://github.com/bigjosh/SimpleNeoPixelDemo/blob/master/SimpleNeopixelDemo/SimpleNeopixelDemo.ino#L58-L149) that solves this exact issue by sending bits to a specific pin directly using inline assembly, counting CPU cycles to get the timing just right.

With the Arduino firmware in place, it was time to set up the Raspberry PI. I started by configuring WIFI and SSH on it, so I could control it remotely. After connecting the Arduino via USB, I used [pyserial](https://pypi.org/project/pyserial/) with some Python scripts to test out the various functions of the firmware.
## Calibration
The main use case I had in mind for the machine was capturing BRDFs. This involves moving the camera arm and switching the LEDs to explore all 4 aforementioned degrees of freedom, while capturing images of a sample at each state. For this to be tenable, it is necessary to account for several sources of error, which would otherwise make the captured data unreliable.

The sample will typically be a flat square piece of material. You can think of each visible pixel in the region of an image which covers the sample as a 'measurement' of the BRDF. Likewise, each image can be thought of us an array of measurements. To make use of each measurement, I needed to know 3 things:
- Which point on the sample the measurement's pixel sees, the 'measurement position'.
- The direction from that point to the light, the 'light direction'.
- The direction from that point to the camera, the 'view direction'.
![[Pasted image 20261114190409.png]]
### Camera calibration
In order to map a pixel position, I needed to know some properties of the camera. In particular, the focal length and optical center.
![[Pasted image 20261114192730.png]]
In the pinhole camera model, the focal length is the distance from the sensor plane to the pinhole. It determines the field of view. The optical center is the point on the sensor plane directly below the pinhole, which may or may not be the center of the sensor plane.

In addition to these camera properties, I needed to account for distortion caused by the lens. My lens exhibits fairly strong barrel distortion (which you may know as fisheye effect). This causes straight lines to appear curved.
![[Pasted image 20261114193404.png]]

To do this, I made use of [OpenCV](https://opencv.org/) via the [cv2 python library](https://pypi.org/project/opencv-python/), which has some [functionality built in](https://docs.opencv.org/4.13.0/dc/dbb/tutorial_py_calibration.html) for this exact purpose. I used a [ChArUco](https://docs.opencv.org/4.13.0/df/d4a/tutorial_charuco_detection.html) board, which I displayed on my phone, and captured from various different camera and turntable positions. This is a chessboard pattern with icons on each white square, designed for easy identification of points on the board.
![[Pasted image 20261114194033.png]]

The setup I used to calibrate the aforementioned parameters (TODO link) works like so:
1. For each image captured of the ChArUco board:
	1. Locate the inner corners of each chessboard cell using `CharucoDetector.detectBoard()`
	2. Map the corners from pixel positions to positions on the ChArUco board using `CharucoBoard.matchImagePoints()`
	3. Gather all the pixel position and board position pairs into lists.
2. Feed the pixel positions and board positions to `cv2.calibrateCamera()`, write the result to a file for later use.
The `cv2.calibrateCamera()` function returns (among some other thing I don't use) a camera matrix, which describes the focal length and optical center, as well as 3 coefficients describing the radial distortion. 

The camera matrix and distortion coefficients can be fed to `cv2.undistortPoints()` to obtain undistorted image coordinates in units of focal length. Here's an example of undistorting one of the ChArUco image captures:
![[Pasted image 20261114190949.png]]
From the undistorted image coordinates $(x, y)$ for a pixel, one can obtain the corresponding direction from camera to measurement position, in the camera's local coordinate frame, by normalizing the vector $(x, y, 1)$. To convert this direction a view direction in a global coordinate frame, some additional data was needed.
### Arm calibration
With a way to correct for error introduced by properties of the camera in place, I needed a way to determine the _pose_ of the camera (the position and rotation in a global coordinate frame) for each arm angle. 

First, I'd like to define to global coordinate frame I used: The origin is on the surface of the turntable, at its axis of rotation. +Z points up along the turntables axis of rotation, and +X points along the camera's image plane +X while the arm is at 0 degrees (perfectly vertical). Everything is measured in units of millimeters. All of the math for our BRDF captures will be done in this coordinate frame.
![[Pasted image 20261114212128.png]]

To determine a mapping from arm angle to camera pose. I split the problem into 2 steps:
- Determine the camera pose at an arm angle of 0 degrees
- Determine how the camera pose changes as a function of arm angle

First, I captured some more images of the ChArUco board from the previous section, at an arm angle of 0 degrees, and with varying turntable angle. I used 6 evenly spaced turntable rotations, so 60 degrees between each. By using `CharucoDetector.detectBoard()` and `cv2.solvePnP()` (short for [Perspective-n-Point](https://docs.opencv.org/4.13.0/d5/d1f/calib3d_solvePnP.html)), I obtained the boards rotation and position relative to the camera in each image. This gave me everything I needed to determine the transformation from camera space (at arm angle 0) to global space:
- The +Z axis of the global frame is given by the average of the normal vectors of the board in each image, normalized.
- To determine the +X axis, the +X direction in the camera's image plane is projected onto the turntable plane by subtracting its component along +Z and normalizing, i.e. $normalize(X_{cam} - (X_{cam} \cdot Z) \ Z)$, where $X_{cam}$ is the camera's image plane +X, and Z is the global +Z direction. With OpenCV's camera coordinate convention, $X_{cam}$ is just $(1,0,0)$.
- The +Y is simply a unit vector perpendicular to +X and +Z, obtained by the cross product of the 2.
- The origin of the global frame is given by the average of the positions of the center of the board in each image, minus the offset between the turntable surface and the board along +Z, i.e. the boards height.
The transformation is conveniently described by:
- A matrix $M_0$ whose rows are +X, +Y, +Z, which represents rotation.
- A 3D vector $p_0$, which represents translation. I have the vector from the camera to the global origin, which I will call $o$, from averaging the board centers in the previous step. By negating it and transforming it to global space, I get the camera position in global space, i.e. $p_0 = -M_0 \cdot o$

Next, I needed to determine the arm-angle to camera pose mapping. By applying the camera-to-global transformation from the previous step to the board pose at arm angle 0, I get the board rotation and position in global space:
$$
\begin{align}
B &= M_0 \cdot R_0\\
b &= p_0 + M_0 \cdot t_0
\end{align}
$$
Where $R_0$ and $t_0$ are the board rotation matrix and position in camera space (the result of `cv2.solvePnP()` at arm angle 0). I then captured some more images of the ChArUco board, but this time from varying arm angles, and with constant turntable angle. For each angle, I again used `CharucoDetector.detectBoard()` and `cv2.solvePnP()` to get the board pose in camera space. By inverting this pose, I obtained the camera's rotation and position relative to the board:
$$
\begin{align}
C_a &= R_a^\top\\
c_a &= -R_a^\top \cdot t_a
\end{align}
$$
Where $R_a$ and $t_a$ are the board rotation matrix and position in camera space (the result of `cv2.solvePnP()` at arm angle $a$). Finally, the camera pose in board-relative space can be transformed to global space using the rotation and position of the board in global space:
$$
\begin{align}
M_a &= B \cdot C_a\\
p_a &= b + B \cdot c_a
\end{align}
$$
These $M_a$ and $p_a$ rotation and position pairs describe the camera pose in global space at each measured arm angle $a$. I stored these in a lookup table for later use. For angles that weren't measured, one can simply interpolate entries of the lookup.
### LED calibration
With camera and arm calibration done, I had a clearly defined global coordinate space, and a way to map each arm-angle to a camera pose. The next problem is determining the position of each LED in this global space. 

The idea behind my method for determining LED positions is fairly simple: I bought a [40mm diameter steel ball](https://www.amazon.de/dp/B0B8G7FP75), which I placed on top of a bottlecap on the turntable, a bit off-center. By capturing images of the ball at a few different turntable angles, and with one LED enabled at a time, one can use the position of the reflection of the LED on the ball at each angle, alongside the balls size and the camera calibration, to determine where the position of the LED must be. That said, the devil is in the details, and this turned out to be the most complicated calibration step overall. For that reason, I will gloss over some of the less interesting details. 
![[Pasted image 20261114234524.png]]

The steps involved in this process can be roughly summarized as:
1. Take an image of the ball with on LED lit at a time, at a few different turntable positions.
2. Find where the ball is on each image by searching for its outline. Use this to determine its position in global space.
3. Find where the LED's reflection is on each image by searching for the brightest pixel.
4. Reflect the camera's view direction off the ball at the reflection's position, which gives a ray direction from the reflection position to the LED.
5. Intersect the reflection-to-LED rays at each turntable angle with each other. The position where they intersect is the LED position.

For
### Image preparation
- Exposure, white balance, gain
- HDR exposure bracketing
- Debayering
- Cropping
- White card sweep and shadowmasks
## BRDF tabulation
#### BTF vs BRDF
#### Isotropic vs anisotropic BRDF
#### MERL parameterization and isoptropic vs anisotropic BRDF
#### Making the data usable
- White card capture
- Gallery of captured BRDFs
## Fitting PBR maps
- Optimization problem
## Anisotropic capture
## Spectral capture
## Bonus: Gaussian splatting

## Dumping ground
TODO:
- Fix grammar tense past/present etc
- Add references
- Less detail in some of calibration sections?
- Link to each python script
![[Pasted image 20261029024541.png]]











































