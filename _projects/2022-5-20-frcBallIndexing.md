---
title: FRC Ball Indexing
date: 2022-5-20
categories: [FRC/VEX Robotics]
tags: [wiring, automation]
author: Jacob
image:
  path: /assets/frc/robot_side_left.jpg
  lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: 
---

A complete overview of the ball sorting/indexing system I designed. Starting with an intro to the sensors and finishing with the code, this article is a full deep dive into how I made the ball sorting/indexing system for the 2022 FRC robot.

## Introduction 

While this small project might not be spectacularly interesting to most, I am quite fond of this small mix of hardware and software working together to automate and perform a task.
The picture above depicts our 2021-2022 First Robotics Competition (FRC) robot. In the middle of the robot are two uprights that contain our ball indexing subsystem. The design goal of this robot was to be able to intake balls on either side of the robot and store them within the indexing subsystem until they were ready to be shot out into the low goal. At worlds, the indexing subsystem was slightly modified to be able to score in the high goal. These changes will be discussed in depth later.

After initial testing of the physical design, it became rather apparent that the system would have to be automated to function correctly. Strict timing combined with added stress on the driver was not something our team wanted, and as the person driving at the time, neither did I. Therefore, the decision to automate the system was made.

## The Sensors

The first step in automating the system was finding suitable sensors for the application. Ideally, whatever sensors we chose would be contactless to reduce the risk getting knocked off.
Searching around within what the team already had, I began playing around with some red-light photoelectric switches.

![Allen Bradley Sensor](/assets/frc/allenbradley_photoelectric_sensor.png)

Upon initial testing (with red balls), these switches appeared to work great, although they were not without their problems. The sensors struggled to detect blue balls. When they did,  they usually stopped in different places than the red balls. This created the problem of having to adjust the sensors depending on what color we were playing for. Another major problem with these sensors is although they did have a digital output, the “LOW” signal was too high for the RoboRio’s (main microcontroller aboard the robot) threshold resulting in a constant “HIGH” read. This was fixed by tying a 10.2kΩ resistor between the signal and ground to drop the voltage down to a readable level.

<img src="/assets/frc/20220309_171003.jpg" style="width: 60%; height: auto; margin-left: 15%; margin-right: auto;"/>

We ran these sensors at the Victoria Regional with minimal issues but decided to keep exploring other options to minimize faults. Eventually, we settled on using the REV Robotics V2 Color Sensors.

We already had one on the robot for color sorting and their IR proximity sensor made detecting the balls a lot more consistent, regardless of color.

<image src="/assets/frc/revv2_on_robot.jpg" style="width: 60%; height: auto; margin-left: 15%; margin-right: auto; border-radius: 5px;"></image>

These sensors were not without their issues either though. Because of their fixed I2C addresses, there would be address conflicts when putting them on the same bus. While the RoboRio does come with two I2C buses, one of the buses caused frequent system lockups preventing it from being used. Therefore, a Raspberry Pi was used to add a second I2C bus to the system. The Raspberry Pi used Network Tables to communicate with the RoboRio.

<image src="/assets/frc/pi_wiring.jpg" style="width: 60%; height: auto; margin-left: 15%; margin-right: auto; border-radius: 5px;"></image>

After extensive testing both in preparation for and during the world championship tournament, the REV V2 Color sensors held up extremely well and proved to be a reliable option for detecting balls in the indexing system. It would be my recommendation to use these sensors in the future when trying to detect objects of varying color. Otherwise, the red light photoelectric sensors are much easier to use with the most complicated part being to not forget the resistor tying signal into ground.

## The Software

The software for this project can easily be split into two parts: The software running on the Pi, and the algorithm itself. The python script running on the Raspberry Pi was not written by me and is used by a lot of other teams for the same purpose. The only thing that I changed was to expose the IR proximity reading of the sensor, as well as to restructure some of the Network Tables directories.

```python
sensor1 = ColorSensorV3(1)
rawColorEntry1 = ntinst.getEntry("/rpi/rawcolor1")
proxEntry1 = ntinst.getEntry("/rpi/proximity1")
smartDashboardProximEntry = ntinst.getEntry("/SmartDashboard/Bot Proxim")
colorEntry1 = ntinst.getEntry("/rpi/color1")

# loop forever
while True:
    # read sensor and send to NT
    rawcolor = sensor1.getRawColor()
    color = [sensor1.getColor().red, sensor1.getColor().green, sensor1.getColor().blue]
    prox = sensor1.getProximity()
    rawColorEntry1.setDoubleArray(
        [rawcolor.red, rawcolor.green, rawcolor.blue, rawcolor.ir])
    proxEntry1.setDouble(prox)
    smartDashboardProximEntry.setDouble(prox)
    colorEntry1.setDoubleArray(color)
            </code></pre>
```

While there was a lot of trial and error that went into building this algorithm, I will simply examine the final state of it for the purpose of this writeup.

The Indexing system uses a command-based robot structure. This is essentially the FRC equivalent of a scheduler. The subsystem is assigned a default task. This default task will run until it is interrupted by another task, then resume when the interrupt task is finished. This indexing subsystem has 6 commands. Below is a flowchart that demonstrates the tasking works. When each task reaches its finished condition, the subsystem returns to its default task: Index.

<div class="mxgraph" style="width:100%; border:1px solid transparent; margin: 0px;" data-mxgraph="{&quot;highlight&quot;:&quot;#0000ff&quot;,&quot;nav&quot;:true,&quot;resize&quot;:true,&quot;toolbar&quot;:&quot;zoom layers tags lightbox&quot;,&quot;edit&quot;:&quot;_blank&quot;,&quot;xml&quot;:&quot;&lt;mxfile host=\&quot;app.diagrams.net\&quot; modified=\&quot;2023-05-14T15:56:37.750Z\&quot; agent=\&quot;Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/113.0.0.0 Safari/537.36\&quot; etag=\&quot;jKzgJ8BnQPrVriXo2dU7\&quot; version=\&quot;21.2.9\&quot; type=\&quot;google\&quot;&gt;\n  &lt;diagram id=\&quot;C5RBs43oDa-KdzZeNtuy\&quot; name=\&quot;Page-1\&quot;&gt;\n    &lt;mxGraphModel dx=\&quot;830\&quot; dy=\&quot;424\&quot; grid=\&quot;0\&quot; gridSize=\&quot;10\&quot; guides=\&quot;1\&quot; tooltips=\&quot;1\&quot; connect=\&quot;1\&quot; arrows=\&quot;1\&quot; fold=\&quot;1\&quot; page=\&quot;1\&quot; pageScale=\&quot;1\&quot; pageWidth=\&quot;413\&quot; pageHeight=\&quot;291\&quot; math=\&quot;0\&quot; shadow=\&quot;0\&quot;&gt;\n      &lt;root&gt;\n        &lt;mxCell id=\&quot;WIyWlLk6GJQsqaUBKTNV-0\&quot; /&gt;\n        &lt;mxCell id=\&quot;WIyWlLk6GJQsqaUBKTNV-1\&quot; parent=\&quot;WIyWlLk6GJQsqaUBKTNV-0\&quot; /&gt;\n        &lt;mxCell id=\&quot;Uespunp7UwCYpG6FZ7zO-3\&quot; value=\&quot;Button Press\&quot; style=\&quot;edgeStyle=orthogonalEdgeStyle;rounded=0;orthogonalLoop=1;jettySize=auto;html=1;exitX=1;exitY=0.5;exitDx=0;exitDy=0;entryX=0;entryY=0.5;entryDx=0;entryDy=0;\&quot; parent=\&quot;WIyWlLk6GJQsqaUBKTNV-1\&quot; source=\&quot;WIyWlLk6GJQsqaUBKTNV-3\&quot; target=\&quot;WIyWlLk6GJQsqaUBKTNV-7\&quot; edge=\&quot;1\&quot;&gt;\n          &lt;mxGeometry relative=\&quot;1\&quot; as=\&quot;geometry\&quot; /&gt;\n        &lt;/mxCell&gt;\n        &lt;mxCell id=\&quot;Uespunp7UwCYpG6FZ7zO-4\&quot; value=\&quot;Button Press\&quot; style=\&quot;edgeStyle=orthogonalEdgeStyle;rounded=0;orthogonalLoop=1;jettySize=auto;html=1;exitX=0;exitY=0.5;exitDx=0;exitDy=0;\&quot; parent=\&quot;WIyWlLk6GJQsqaUBKTNV-1\&quot; source=\&quot;WIyWlLk6GJQsqaUBKTNV-3\&quot; target=\&quot;WIyWlLk6GJQsqaUBKTNV-12\&quot; edge=\&quot;1\&quot;&gt;\n          &lt;mxGeometry relative=\&quot;1\&quot; as=\&quot;geometry\&quot; /&gt;\n        &lt;/mxCell&gt;\n        &lt;mxCell id=\&quot;Uespunp7UwCYpG6FZ7zO-5\&quot; value=\&quot;Button Press\&quot; style=\&quot;edgeStyle=orthogonalEdgeStyle;rounded=0;orthogonalLoop=1;jettySize=auto;html=1;exitX=0.25;exitY=1;exitDx=0;exitDy=0;entryX=0.75;entryY=0;entryDx=0;entryDy=0;\&quot; parent=\&quot;WIyWlLk6GJQsqaUBKTNV-1\&quot; source=\&quot;WIyWlLk6GJQsqaUBKTNV-3\&quot; target=\&quot;WIyWlLk6GJQsqaUBKTNV-11\&quot; edge=\&quot;1\&quot;&gt;\n          &lt;mxGeometry relative=\&quot;1\&quot; as=\&quot;geometry\&quot; /&gt;\n        &lt;/mxCell&gt;\n        &lt;mxCell id=\&quot;Uespunp7UwCYpG6FZ7zO-11\&quot; value=\&quot;Button Press\&quot; style=\&quot;edgeStyle=orthogonalEdgeStyle;rounded=0;orthogonalLoop=1;jettySize=auto;html=1;exitX=0.75;exitY=1;exitDx=0;exitDy=0;entryX=0.25;entryY=0;entryDx=0;entryDy=0;\&quot; parent=\&quot;WIyWlLk6GJQsqaUBKTNV-1\&quot; source=\&quot;WIyWlLk6GJQsqaUBKTNV-3\&quot; target=\&quot;Uespunp7UwCYpG6FZ7zO-9\&quot; edge=\&quot;1\&quot;&gt;\n          &lt;mxGeometry relative=\&quot;1\&quot; as=\&quot;geometry\&quot; /&gt;\n        &lt;/mxCell&gt;\n        &lt;mxCell id=\&quot;Uespunp7UwCYpG6FZ7zO-16\&quot; value=\&quot;Button Press\&quot; style=\&quot;edgeStyle=orthogonalEdgeStyle;rounded=0;orthogonalLoop=1;jettySize=auto;html=1;exitX=0.5;exitY=1;exitDx=0;exitDy=0;entryX=0.5;entryY=0;entryDx=0;entryDy=0;startArrow=classic;startFill=1;\&quot; parent=\&quot;WIyWlLk6GJQsqaUBKTNV-1\&quot; source=\&quot;WIyWlLk6GJQsqaUBKTNV-3\&quot; target=\&quot;Uespunp7UwCYpG6FZ7zO-13\&quot; edge=\&quot;1\&quot;&gt;\n          &lt;mxGeometry relative=\&quot;1\&quot; as=\&quot;geometry\&quot; /&gt;\n        &lt;/mxCell&gt;\n        &lt;mxCell id=\&quot;WIyWlLk6GJQsqaUBKTNV-3\&quot; value=\&quot;Index\&quot; style=\&quot;rounded=1;whiteSpace=wrap;html=1;fontSize=12;glass=0;strokeWidth=1;shadow=0;\&quot; parent=\&quot;WIyWlLk6GJQsqaUBKTNV-1\&quot; vertex=\&quot;1\&quot;&gt;\n          &lt;mxGeometry x=\&quot;352\&quot; y=\&quot;255\&quot; width=\&quot;120\&quot; height=\&quot;40\&quot; as=\&quot;geometry\&quot; /&gt;\n        &lt;/mxCell&gt;\n        &lt;mxCell id=\&quot;Uespunp7UwCYpG6FZ7zO-8\&quot; value=\&quot;Button Press\&quot; style=\&quot;edgeStyle=orthogonalEdgeStyle;rounded=0;orthogonalLoop=1;jettySize=auto;html=1;exitX=0.5;exitY=0;exitDx=0;exitDy=0;entryX=1;entryY=0;entryDx=0;entryDy=0;\&quot; parent=\&quot;WIyWlLk6GJQsqaUBKTNV-1\&quot; source=\&quot;WIyWlLk6GJQsqaUBKTNV-7\&quot; target=\&quot;WIyWlLk6GJQsqaUBKTNV-3\&quot; edge=\&quot;1\&quot;&gt;\n          &lt;mxGeometry relative=\&quot;1\&quot; as=\&quot;geometry\&quot; /&gt;\n        &lt;/mxCell&gt;\n        &lt;mxCell id=\&quot;WIyWlLk6GJQsqaUBKTNV-7\&quot; value=\&quot;Stop\&quot; style=\&quot;rounded=1;whiteSpace=wrap;html=1;fontSize=12;glass=0;strokeWidth=1;shadow=0;\&quot; parent=\&quot;WIyWlLk6GJQsqaUBKTNV-1\&quot; vertex=\&quot;1\&quot;&gt;\n          &lt;mxGeometry x=\&quot;572\&quot; y=\&quot;255\&quot; width=\&quot;120\&quot; height=\&quot;40\&quot; as=\&quot;geometry\&quot; /&gt;\n        &lt;/mxCell&gt;\n        &lt;mxCell id=\&quot;Uespunp7UwCYpG6FZ7zO-6\&quot; value=\&quot;Timer\&quot; style=\&quot;edgeStyle=orthogonalEdgeStyle;rounded=0;orthogonalLoop=1;jettySize=auto;html=1;exitX=0;exitY=0.5;exitDx=0;exitDy=0;entryX=0;entryY=1;entryDx=0;entryDy=0;\&quot; parent=\&quot;WIyWlLk6GJQsqaUBKTNV-1\&quot; source=\&quot;WIyWlLk6GJQsqaUBKTNV-11\&quot; target=\&quot;WIyWlLk6GJQsqaUBKTNV-3\&quot; edge=\&quot;1\&quot;&gt;\n          &lt;mxGeometry relative=\&quot;1\&quot; as=\&quot;geometry\&quot;&gt;\n            &lt;Array as=\&quot;points\&quot;&gt;\n              &lt;mxPoint x=\&quot;192\&quot; y=\&quot;375\&quot; /&gt;\n              &lt;mxPoint x=\&quot;192\&quot; y=\&quot;315\&quot; /&gt;\n              &lt;mxPoint x=\&quot;352\&quot; y=\&quot;315\&quot; /&gt;\n            &lt;/Array&gt;\n          &lt;/mxGeometry&gt;\n        &lt;/mxCell&gt;\n        &lt;mxCell id=\&quot;WIyWlLk6GJQsqaUBKTNV-11\&quot; value=\&quot;Shoot High\&quot; style=\&quot;rounded=1;whiteSpace=wrap;html=1;fontSize=12;glass=0;strokeWidth=1;shadow=0;\&quot; parent=\&quot;WIyWlLk6GJQsqaUBKTNV-1\&quot; vertex=\&quot;1\&quot;&gt;\n          &lt;mxGeometry x=\&quot;212\&quot; y=\&quot;355\&quot; width=\&quot;120\&quot; height=\&quot;40\&quot; as=\&quot;geometry\&quot; /&gt;\n        &lt;/mxCell&gt;\n        &lt;mxCell id=\&quot;Uespunp7UwCYpG6FZ7zO-7\&quot; value=\&quot;Timer\&quot; style=\&quot;edgeStyle=orthogonalEdgeStyle;rounded=0;orthogonalLoop=1;jettySize=auto;html=1;exitX=0.5;exitY=0;exitDx=0;exitDy=0;entryX=0;entryY=0;entryDx=0;entryDy=0;\&quot; parent=\&quot;WIyWlLk6GJQsqaUBKTNV-1\&quot; source=\&quot;WIyWlLk6GJQsqaUBKTNV-12\&quot; target=\&quot;WIyWlLk6GJQsqaUBKTNV-3\&quot; edge=\&quot;1\&quot;&gt;\n          &lt;mxGeometry relative=\&quot;1\&quot; as=\&quot;geometry\&quot; /&gt;\n        &lt;/mxCell&gt;\n        &lt;mxCell id=\&quot;WIyWlLk6GJQsqaUBKTNV-12\&quot; value=\&quot;Shoot Low\&quot; style=\&quot;rounded=1;whiteSpace=wrap;html=1;fontSize=12;glass=0;strokeWidth=1;shadow=0;\&quot; parent=\&quot;WIyWlLk6GJQsqaUBKTNV-1\&quot; vertex=\&quot;1\&quot;&gt;\n          &lt;mxGeometry x=\&quot;132\&quot; y=\&quot;255\&quot; width=\&quot;120\&quot; height=\&quot;40\&quot; as=\&quot;geometry\&quot; /&gt;\n        &lt;/mxCell&gt;\n        &lt;mxCell id=\&quot;Uespunp7UwCYpG6FZ7zO-2\&quot; style=\&quot;edgeStyle=orthogonalEdgeStyle;rounded=0;orthogonalLoop=1;jettySize=auto;html=1;exitX=0.5;exitY=1;exitDx=0;exitDy=0;entryX=0.5;entryY=0;entryDx=0;entryDy=0;\&quot; parent=\&quot;WIyWlLk6GJQsqaUBKTNV-1\&quot; source=\&quot;Uespunp7UwCYpG6FZ7zO-1\&quot; target=\&quot;WIyWlLk6GJQsqaUBKTNV-3\&quot; edge=\&quot;1\&quot;&gt;\n          &lt;mxGeometry relative=\&quot;1\&quot; as=\&quot;geometry\&quot; /&gt;\n        &lt;/mxCell&gt;\n        &lt;mxCell id=\&quot;Uespunp7UwCYpG6FZ7zO-1\&quot; value=\&quot;Program Start\&quot; style=\&quot;rounded=1;whiteSpace=wrap;html=1;fontSize=12;glass=0;strokeWidth=1;shadow=0;\&quot; parent=\&quot;WIyWlLk6GJQsqaUBKTNV-1\&quot; vertex=\&quot;1\&quot;&gt;\n          &lt;mxGeometry x=\&quot;352\&quot; y=\&quot;155\&quot; width=\&quot;120\&quot; height=\&quot;40\&quot; as=\&quot;geometry\&quot; /&gt;\n        &lt;/mxCell&gt;\n        &lt;mxCell id=\&quot;Uespunp7UwCYpG6FZ7zO-12\&quot; value=\&quot;Button Press\&quot; style=\&quot;edgeStyle=orthogonalEdgeStyle;rounded=0;orthogonalLoop=1;jettySize=auto;html=1;exitX=1;exitY=0.5;exitDx=0;exitDy=0;entryX=1;entryY=1;entryDx=0;entryDy=0;\&quot; parent=\&quot;WIyWlLk6GJQsqaUBKTNV-1\&quot; source=\&quot;Uespunp7UwCYpG6FZ7zO-9\&quot; target=\&quot;WIyWlLk6GJQsqaUBKTNV-3\&quot; edge=\&quot;1\&quot;&gt;\n          &lt;mxGeometry relative=\&quot;1\&quot; as=\&quot;geometry\&quot;&gt;\n            &lt;Array as=\&quot;points\&quot;&gt;\n              &lt;mxPoint x=\&quot;622\&quot; y=\&quot;375\&quot; /&gt;\n              &lt;mxPoint x=\&quot;622\&quot; y=\&quot;315\&quot; /&gt;\n              &lt;mxPoint x=\&quot;472\&quot; y=\&quot;315\&quot; /&gt;\n            &lt;/Array&gt;\n          &lt;/mxGeometry&gt;\n        &lt;/mxCell&gt;\n        &lt;mxCell id=\&quot;Uespunp7UwCYpG6FZ7zO-9\&quot; value=\&quot;Manual Override\&quot; style=\&quot;rounded=1;whiteSpace=wrap;html=1;fontSize=12;glass=0;strokeWidth=1;shadow=0;\&quot; parent=\&quot;WIyWlLk6GJQsqaUBKTNV-1\&quot; vertex=\&quot;1\&quot;&gt;\n          &lt;mxGeometry x=\&quot;482\&quot; y=\&quot;355\&quot; width=\&quot;120\&quot; height=\&quot;40\&quot; as=\&quot;geometry\&quot; /&gt;\n        &lt;/mxCell&gt;\n        &lt;mxCell id=\&quot;Uespunp7UwCYpG6FZ7zO-13\&quot; value=\&quot;High Index\&quot; style=\&quot;rounded=1;whiteSpace=wrap;html=1;fontSize=12;glass=0;strokeWidth=1;shadow=0;\&quot; parent=\&quot;WIyWlLk6GJQsqaUBKTNV-1\&quot; vertex=\&quot;1\&quot;&gt;\n          &lt;mxGeometry x=\&quot;352\&quot; y=\&quot;415\&quot; width=\&quot;120\&quot; height=\&quot;40\&quot; as=\&quot;geometry\&quot; /&gt;\n        &lt;/mxCell&gt;\n      &lt;/root&gt;\n    &lt;/mxGraphModel&gt;\n  &lt;/diagram&gt;\n&lt;/mxfile&gt;\n&quot;}"></div>
<script type="text/javascript" src="https://viewer.diagrams.net/js/viewer-static.min.js"></script>


The indexing task itself is a finite state machine that uses a variable called “codex” to track the state of the system. Depending on the state of the system, the top, bottom, and feed rollers are set to move at certain intake speeds or stop. Below is the code that controls these conditions.

```cpp
if ( codex == 2 ){
subsystem->setTop(0);
subsystem->setBottom(0);
subsystem->setFeed(0);
}
else if ( codex == 1 ){
subsystem->setTop(ControlMode::Velocity, 0);
subsystem->setBottom(iPow);
subsystem->setFeed(1);
}
else if ( codex == 0 ){
subsystem->setTopVel(vPow);
subsystem->setBottom(iPow);
subsystem->setFeed(1);
}
else{
printf("Codex OverFlow");
codex = 0;
}  
```

While the above code controls the state of the motors using the codex as an input, the code below controls the state of the codex depending on its current state, and if any of the sensors have been triggered. Also within this code is some color sorting logic to prevent indexing balls of the wrong color.

```cpp
if(teamColor.GetSelected() != IndexSubsystem::TeamColors::null){
    // if BallColor == TeamColor && codex == 0 -> Index ball
    if (teamColor.GetSelected() == subsystem->getTopBallColor() && codex == 0){
        codex = 1;
    }

    // If a ball has been registered by the Proximity Sensor, But the Color is undeterminable, set top indexer drive power to zero
    // Fixes Balls getting caught too high in the top indexer
    if(subsystem->getTopIR() && subsystem->getTopBallColor() == IndexSubsystem::TeamColors::null){
        vPow = 0;
    }
    else{
        vPow = 6000;
    }
}
else{
if ( subsystem->getTopIR() && codex == 0){ codex = 1; };
}

if ( subsystem->getBotIR() && codex == 1 ){ codex = 2; };
```

Once the driver is ready to score the balls indexed by the system, they can press a trigger button that will interrupt the “Index” task and schedule a different task depending on which button was pressed.

```cpp
// Shoot Balls Stored within the index when Right Bumper is Pressed
frc2::JoystickButton(&master, frc::XboxController::Button::kRightBumper)
.WhenPressed(new IndexCommands::ShootLow(&subsystem_index));

// Shoot High Goal -> Speeds up top indexer and shoots once it reaches the correct Velocity
frc2::JoystickButton(&master, frc::XboxController::Button::kLeftBumper)
.WhenPressed(new IndexCommands::ShootHigh(&subsystem_index)
);

// Master A -> Runs Auton Indexing sequence for high goal shooting
frc2::JoystickButton(&master, frc::XboxController::Button::kA)
.ToggleWhenPressed(IndexCommands::Index(&subsystem_index)); 
```

The driver can also choose to stop all motor movement or engage manual control of the indexing system.

```cpp
/**
* @brief Index Toggle Command
* Note: Shoot Commands will override this command,
* Upon Exiting a shoot command, the Indexing command will resume
* To continue manual operation, the partner controller will have to re-enable the override
* 
* Allow the partner controller to toggle between indexing modes
* 
* Default Mode: Automatic Indexing though Photoelectric Sensors
*               IndexCommands/Index
* 
* Override Mode: Manual Indexing through the Partner Controller
*                IndexCommands/Manual
*/
frc2::JoystickButton(&partner, frc::XboxController::Button::kLeftBumper)
    .ToggleWhenPressed(IndexCommands::Manual(&subsystem_index));
```

The system features two different shooting tasks that can be run. The first is the “Low Shot” task that we made for the Victoria Regional tournament and simply spins all the indexing motors upwards for a given period.

```cpp
// Copyright (c) FIRST and other WPILib contributors.
// Open Source Software; you can modify and/or share it under the terms of
// the WPILib BSD license file in the root directory of this project.

#include "commands/IndexCommands/ShootLow.hpp"
#include "commands/IndexCommands/Index.hpp"
#include <frc/smartdashboard/SmartDashboard.h>
#include "RobotContainer.h"
#include <frc/Timer.h>
#include <tools/Tools.hh>

using namespace IndexCommands;

ShootLow::ShootLow(IndexSubsystem* indexSubsystem) : subsystem{indexSubsystem}, isFinished{false}, startTime{frcTools::Time::Millis()}{
    // Use addRequirements() here to declare subsystem dependencies.
    AddRequirements(indexSubsystem);
    SetName("Low Shot");
}

// Called when the command is initially scheduled.
void ShootLow::Initialize() {

    frc::SmartDashboard::PutString("Indexing", "Low Shot");

    startTime = frcTools::Time::Millis();
    isFinished = false;

    subsystem->setFeed(0);
    //subsystem->setTopVel(TopIndexConverter(fxMaxRPM*0.5));
    subsystem->setTop(0.55);
    subsystem->setBottom(1);
    IndexCommands::codex = 0;

    compressor.Disable();
}

// Called repeatedly when this Command is scheduled to run
void ShootLow::Execute() {
    if(frcTools::Time::Millis() > startTime + 1500){
    isFinished = true;
    }
}

// Called once the command ends or is interrupted.
void ShootLow::End(bool interrupted) {
    frc::SmartDashboard::PutString("Indexing", "Disabled");
    IndexCommands::codex = 0;
    compressor.EnableDigital();
}

// Returns true when the command should end.
bool ShootLow::IsFinished() {
    return isFinished;
}
```

The “Shoot High” task functions similarly to the “Shoot Low” task but also includes a delay before firing the balls to allow the flywheel to spin up to speed before firing.

```cpp
/**
* @file ShootHigh.hpp
* @author Jacob Chisholm
* @brief Index High Shot Command
* @date 2022-03-21
* 
* @copyright Copyright (c) 2022
* 
*/

#include "commands/IndexCommands/ShootHigh.hpp"
#include "Constants.h"
#include "tools/Tools.hh"
#include "commands/IndexCommands/Index.hpp"
#include "frc/smartdashboard/SmartDashboard.h"

IndexCommands::ShootHigh::ShootHigh(IndexSubsystem* sys_index) : index{sys_index}, isFinished{false}, reachedRPM{false} {
    AddRequirements(index);
    SetName("High Shot");
}

// Called when the command is initially scheduled.
void IndexCommands::ShootHigh::Initialize() {

    frc::SmartDashboard::PutString("Indexing", "High Shot");

    index->setFeed(0);
    index->setBottom(0);
    index->setTopVel(c_TalonUPR(6300));
    //index->setTop(1);

    startTime = frcTools::Time::Millis();
    spinUpTime = frcTools::Time::Millis();
    isFinished = false;
    reachedRPM = false;
    IndexCommands::codex = 0;
}

// Called repeatedly when this Command is scheduled to run
void IndexCommands::ShootHigh::Execute() {
    
    if(index->getTopVelocity() > 17000 && reachedRPM == false){
    index->setBottom(1);
    reachedRPM = true;
    spinUpTime = frcTools::Time::Millis();
    }

    if(reachedRPM && (spinUpTime + 666 < frcTools::Time::Millis())){
    index->setFeed(1);
    }

    if(frcTools::Time::Millis() > startTime + 3000){
    isFinished = true;
    }
}

// Called once the command ends or is interrupted.
void IndexCommands::ShootHigh::End(bool interrupted) {
    frc::SmartDashboard::PutString("Indexing", "Disabled");
}

// Returns true when the command should end.
bool IndexCommands::ShootHigh::IsFinished() {
    return isFinished;
}  
```

Again, an if statement is used as a delay to avoid system lockups.

All code for the Indexing system can be found <a href="https://github.com/Jchisholm204/Ten-Ton-FRC-2022/tree/main/6364%20Worlds/src">here</a>, on the 6364 GitHub Repository.