### Foreword
A bit over a decade ago, I wrote my first [path tracer](https://en.wikipedia.org/wiki/Path_tracing). Ever since then, [light transport](https://en.wikipedia.org/wiki/Light_transport_theory), the mathematical theory underpinning simulation of realistic lighting with [global illumination](https://en.wikipedia.org/wiki/Global_illumination), has been an area of great interest for me. Some years later, this interest even inspired me to start working as a graphics programmer specialized in the topic!

Early in my learning, I remember wondering how the various material and shading models I stumbled upon were invented, and found that many are essentially conveniently sampleable approximations of real-world observations. At some point I learned of a type of device for making these real-world observations, called a [*gonioreflectometer*](https://en.wikipedia.org/wiki/Gonioreflectometer). This device fascinated and intrigued me, but some googling revealed that these are [unobtainium](https://en.wikipedia.org/wiki/Unobtainium) unless you have deep pockets or work at a university, so my fascination eventually waned.

Fast forwarding to the present, I've been working on [yet another path tracer](https://github.com/pema99/odin-pt), with a focus on spectral rendering and advanced material models. Hacking away at this reminded me of the device that intrigued me all those years ago. I've been meaning to get some more hands-on experience with hobby electronics, so I decided to try creating such a device for myself. In this blog post, I want to share my experience and learnings from working on it.
### What is a gonioreflectometer?
In a single sentence, a gonioreflectometer is a device that measures the [Bidirectional Reflectance Distribution Function](https://en.wikipedia.org/wiki/Bidirectional_reflectance_distribution_function) (often abbreviated BRDF) of a material sample. So, what is a BRDF?

The term is often used by graphics programmers in less-than-rigorous ways, but the classic definition is a function of 2 inputs - an *incident light direction* $\omega_i$ and a *view direction* $\omega_r$. This function answers the question the "how much light is reflected towards my eye from a point on a surface, given the directions from the point towards a light source, and towards my eye". Hence, the result of evaluating a BRDF is a value in range [0; 1]. You can think of a materials BRDF as a mathematical description of how the material interacts with light, and thus its appearance.
![[Pasted image 20261029033731.png]]

There are several different types of gonioreflectometer, but the majority of them work by systematically moving a *sensor* and a *light source* to capture measurements of the sample from different view and incident light directions. One way to think of a BRDF is as a 4 dimensional function - each of the 2 input directions can be represented with no less than 2 scalars (for example with [spherical coordinates](https://en.wikipedia.org/wiki/Spherical_coordinate_system)). Therefore, to measure an arbitrary BRDF, a gonioreflectometer needs 4 degrees of freedom - 2 for the sensor and 2 for light source.
### Building the machine
Building a mechanism for moving both a sensor and a light source, with several degrees of freedom, and within the same physical space is above my skill level, so I spent some time researching different types of gonioreflectometer. Eventually I stumbled upon Felix Schröder's masters thesis [Construction of a gonioreflectometer to capture and render BRDFs](https://f-schroeder.github.io/publication/masterthesis/) which presents a very simple and cheap design that achieves all 4 degrees of freedom. My build is largely based on their design, though with some tweaks and improvements.

The device consists of 3 main parts: A turntable, a camera attached to a movable arm, and a dome with an LED matrix attached to the inside. The turntable and the camera arm provide the 2 degrees of freedom for the sensor, which is in this case a [Raspberry PI HQ Camera](https://www.raspberrypi.com/products/raspberry-pi-high-quality-camera/). The dome is placed above the turntable, and by toggling individual LEDs in the matrix, we achieve 2 degrees of freedom for the light source, though with some quantization error due to the relatively sparse layout of LEDs.![[Pasted image 20261029041310.png]]

In the sections below, I will show how I built the machine, part by part. This will be structured as a sort of build log, with plenty of progress images along the way. If you aren't interested in the build process, you can skip to the [[#Exploring usecases]] section, in which I discuss how and what I used the machine for.
#### Testing electronics
The machine is driven by a few different electronic components. It seemed obvious to configure and test these as the first step, starting with the motor assembly. As in Schröders' thesis, I used an [Arduino Uno](https://docs.arduino.cc/hardware/uno-rev3/) with a [A4988 stepper motor driver shield](https://www.amazon.de/dp/B0D91JCWQG) to control the motors for the arm and turntable. 

The drivers require 12 volt DC. I used a cheap ATX power supply with a modified molex connector.
![[Pasted image 20261029023827.jpg|332]]![[Pasted image 20261029023758.jpg|332]]
For the turntable, I used a cheap [17HS15-1206S](https://www.omc-stepperonline.com/nema-17-unipolar-1-8deg-26ncm-36-8oz-in-1-2a-4v-42x42x39mm-6-wires-17hs15-1206s) stepper motor. I initially tried to use the same model for the camera arm, but found that it didn't have enough torque to lift the arm. Instead, I opted for a pair of [17HS19-1684S-PG5](https://www.omc-stepperonline.com/nema-17-stepper-motor-bipolar-l-48mm-w-gear-ratio-5-1-planetary-gearbox-17hs19-1684s-pg5). These are similar motors, but the latter has an integrated planetary gearbox with a 5:1 gear ratio. 

The steppers are rated for 1.2 and 1.68 amps respectively, though the drivers can get quite hot at higher currents. The [driver's datasheet](https://cdn.sparkfun.com/datasheets/Robotics/A4988-Datasheet.pdf) has a nifty formula for calculating the max current:
$$
I_{TripMAX} = \frac{V_{REF}}{(8 \cdot R_S)}
$$
Where $R_S$ is the resistance of the "sense resistor" (0.1ohm on my board), and $V_{REF}$ is the voltage measured between each driver's potentiometer and ground. I used $V_{REF}=0.8$v for the arm motors and $V_{REF}=0.32$v for the turntable, which has a much smaller load. Plugging in the numbers, we get a current of $1.0$ and $0.4$ amps respectively, well within rating for the motors.
![[Pasted image 20261029023833.jpg|653]]
Stepper motors work by switching between an array of electromagnets to move the shaft. We call the transition from one set of magnets to the next in the switching sequence a step. On my turntable motor, that corresponds to a 1.8 degree rotation. The drivers support 'microstepping', which allows for smoother and more granular rotations by powering two sets of magnets at once with varying amounts of current. This is desirable for my use case, so I configured the drivers for 1/16th microstepping by bridging the MS1, MS2, MS3 pins to 5V with solder.
![](https://electricdiylab.com/wp-content/uploads/2019/12/STEPPER-2.gif)
Image source: https://electricdiylab.com/how-stepper-motor-works/

With the drivers configured, I uploaded a quick test script to the turn my turntable stepper one full rotation.
![[Pasted image 20261029023844.jpg|644]]

To keep the assembly cool, I connected a cheap PC fan via molex to my PSU.
![[Pasted image 20261029024207.jpg|597]]

Next, it was time to test the camera. I flashed a fresh Raspbian install onto my PI, plugged in the camera to the CSI port, and took a picture using [picamera2](https://github.com/raspberrypi/picamera2). The camera module comes without a lens, I used an [official 6mm wide angle](https://www.kubii.com/en/objectives-supports/2952-official-6mm-wide-angle-lens-3272496301498.html) one.
![[Pasted image 20261029024218.jpg|337]]![[Pasted image 20261112013249.png|333]]
#### The turntable and frame
With the electronics confirmed working, I started working on the turntable assembly. To save some hassle, I bought a pack of [circular MDF cutouts](https://www.amazon.de/dp/B0DVGZYCYV?ref=ppx_yo2ov_dt_b_fed_asin_title), and stacked a few of them together using glue. To attach it to the stepper, I glued a [flange coupling](https://www.amazon.de/dp/B0993M969V?ref=ppx_yo2ov_dt_b_fed_asin_title) to the center of the underside.
![[Pasted image 20261029023751.jpg|564]]

For the main frame, I bought a [steel plate](https://www.amazon.de/dp/B088FNL29V) with a grid of pre-drilled holes, for easier mounting of components. The turntable stepper sits on the underside of the plate, though the shaft wasn't thin enough to pass through the holes, so I had to widen one a bit with a rotary tool.
![[Pasted image 20261029023849.jpg|312]]![[Pasted image 20261029023856.jpg|315]]

I made some simple feet for the frame using a cheap [pack of L-bracket](https://www.amazon.de/dp/B0D6YTK3KT) and some nuts and bolts.
![[Pasted image 20261029024036.jpg|349]]![[Pasted image 20261029024100.jpg|350]]

With these pieces prepared, I assembled the frame. I used some [Nema 17 brackets](https://www.stepperonline.co.uk/nema-17-bracket-for-stepper-motor-and-geared-stepper-motor-alloy-steel-bracket-st-m1.html) to mount the 2 arm motors to the plate, about 45cm apart, aligned with the turntable shaft.
![[Pasted image 20261029023911.jpg|337]]![[Pasted image 20261029023905.jpg|338]]
#### The camera assembly
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
#### The dome
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
The finished dome looks like this:
![[IMG20261014232819.jpg|633]]
And here's a video of me testing it with a cool animated rainbow pattern:![[VID20261010221130_cropped.mp4]]
#### Finishing touches
With the electronics, frame and dome all prepared, the machine was almost finished. All the remained was some polishing and cleanup steps. First, I painted everything black and attached the ribbon cable for the camera permanently to the arm with some glue.
![[IMG20261007200726.jpg|615]]

Next, I mounted the frame onto a large MDF plate to keep everything organized. I also made some feet for the dome to stand on, using L-brackets and bolts, similar to the frame's feet.
![[IMG20261015012845.jpg|619]]

I used some zipties to do cable management, and mounted the electronics to the MDF plate using some screws and standoffs. Additionally, I painted the area around the frame black, as light can reflect onto and into the dome.
![[Pasted image 20261112030315.png]]

To keep it in more or less the same position and orientation every time, I drew some lines on the dome and the feet that should line up.

!!!TODO: Image

Here's how the finished machine looks, with the dome resting on its feet.

!!!TODO: Final image
### Exploring usecases
- Gaussian splatting
- BRDF tabulation
	- Calibration
	- Isotropic vs anisotropic BRDF
	- MERL database
- PBR fit
- BTF vs BRDF
- Spectral capture
### Dumping ground
![[Pasted image 20261029024541.png]]











































