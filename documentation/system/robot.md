---
secondsection: system
section: System Management
title: Robot Configurations
sort_info: 130
---

It is very common for a bunch of robots to share the same, but not _exactly the
same_ hardware and general functionality. In some cases, you will want to factor
the common parts out into [a separate bundle](file_layout.html), while in other
cases you would prefer working inside the same general file layout.

In Syskit, having different setups within the same bundle is achieved by
defining different robots.

A new robot is created with

~~~
syskit gen robot ROBOT_NAME
~~~

it is for instance very common to have a live robot and a simulated one:

~~~
syskit gen robot flatfish-live
syskit gen robot flatfish-gazebo
~~~

When one starts to use robots, the robot configuration file in config/robots/
(e.g. config/robots/flatfish-live.rb) becomes the entry point for Syskit's
loading mechanisms.

Most (if not all) of Syskit command line accepts a -r option as e.g.

~~~
syskit run -rflatfish-live -c
~~~

or

~~~
syskit browse -rflatfish-live
~~~

In the robot configuration templates, you can see a bunch of calls to methods on
Robot, each given a block. The rest of this page will introduce you to each and
every one of them.

Robot.init:: there, one should add code that sets up things required for the
  loading itself, such as plugin activation or enabling/disabling of backward
  compatibility options. The config/init.rb file can be used for the same
  purpose if you want code that would be common to all robots.
Robot.requires:: loads the model files for this system. Tailoring the requires
  to what this system needs has two positive effects: it reduces loading time, and
  reduces the amount of dependencies on oroGen tasks the robot has (as
  any using_task_library statement effectively requires the task library to be
  present for the file to load). The template calls Roby.app.load_default_models
  which loads the toplevel files in models/ (e.g. models/compositions.rb, ...)
Robot.config:: sets up configuration parameters
Robot.controller:: block of code executed just after the Syskit application
  finished starting up. This is usually where one starts permanent actions.
Robot.actions:: set up of the main action interface. It is highly recommended to
  __NOT__ define actions there, but to only import action interfaces with the use
  and use_profile statements.

__Converting "old-style" robot configurations__ Any robot-specific content in
config/init.rb should go in Robot.init. The content of config/ROBOT.rb should be
split between requires and config. The content of scripts/controllers/ROBOT.rb
obviously goes in the controller blocks, and the Main action classes are now
setup through the actions block
{: .block}


