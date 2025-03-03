---
secondsection: system
section: System Management
title: Profiles
sort_info: 510
---

Profiles
--------
At some point, one has to bind everything together. We have so far a bunch of
models, and a way to specify our actual needs. What we are now looking for is a
way to provide a consistent set of predefined component networks that will be
the actual robot function.

This consistent set of definitions is called a profile. Profiles are defined
inside a module with:

~~~ ruby
module Profiles
  profile "ProfileName" do
  end
end
~~~

This creates a ProfileName object in the enclosing module.

<div class="block" markdown="1">
__Generation__ Template for a profile, following Syskit's naming and filesystem
conventions, can be created with

~~~
syskit gen profile name/of_profile
~~~

for instance

~~~
syskit gen profile rovers/localization
~~~

would create a Rock::Profiles::Rovers::Localization profile in
models/profiles/rovers/localization.rb, and the associated test file
test/profiles/rovers/test_localization.rb. Files in the hierarchy
(models/profiles/rovers.rb and test/profiles/suite_rovers.rb) are updated to
require the newly created files.
</div>

__Naming__ Profiles are by convention defined under the BundleName::Profiles
namespace (e.g. Rock::Profiles for the rock bundle). 

__Filesystem__ Profiles are defined in models/profiles/. The file
names should be the snake_case version of the profile name (e.g.
models/profiles/my_project.rb for the profile defined above). Any task context,
composition or data service used in the profile file should be required first
with either the using_task_library statement (for task contexts) or a simple
require of the relevant file. Use [syskit browse](general_concept.html#browsing)
to find out what is defined where.
{: .block}

The first thing one can do in a profile is to give names to [instance
requirements](subsystem_design.html). For instance, following the example above, one would probably
create a pose estimation definition:

~~~ ruby
profile "Base" do
  define "pose_estimation", Compositions::PoseEstimation.
    use('orientation' => OroGen::XsensImu::Task.with_conf('default', 'high_sampling_rate'))
end
~~~

Once this definition is created, it can be referred to by using the _def suffix.
For instance, one would add the corridor servoing definition from the previous
examples with:

~~~ ruby
profile "Base" do
  define "pose_estimation", Compositions::PoseEstimation.
    use('orientation' => OroGen::XsensImu::Task.with_conf('default', 'high_sampling_rate'))
  define "servoing", Compositions::CorridorNavigation::CorridorServoing.
    use('pose' => pose_estimation_def)
end
~~~

Profile-wide selections
-----------------------
The "use" statement allows to provide selections at the level of a profile, e.g.
for all the definitions in that profile. Each definition can, of course,
override that profile-wide setting. For instance:

~~~ ruby
profile "Base" do
  define "pose_estimation", Compositions::PoseEstimation.
    use('orientation' => OroGen::XsensImu::Task.with_conf('default', 'high_sampling_rate'))
  use Services::Pose => pose_estimation_def
end
~~~

A common usage for this statement is to give some default configuration for a
class of type contexts. For instance:

~~~ ruby
profile "Base" do
  use OroGen::PoseEstimator::Task =>
    OroGen::PoseEstimator::Task.use_conf('default', 'low_latency')
end
~~~

Reusing Profiles
----------------
Profiles are often an extension of existing "generic" profiles. For instance,
Rock's rock_ugv_nav bundle defines the RockUGVNav::Profiles::Skid4 profile that gives
common definitions for any system that is a four-wheeled, skid-steered system.
This is a great way to provide pre-integrated functionality for some classes of
systems, leaving only the "binding to the specific platform" to the system's
integrator.

Reusing such a profile is done in two steps. First, one must require the file of
the parent profile

~~~
require 'rock_ugv_nav/models/profiles/skid4.rb'
~~~

Second, one has to declare that the "child" profile must reuse the definitions
from the "parent" profile.

~~~ ruby
profile "Robot do
  use_profile RockUGVNav::Profiles::Skid4
end
~~~

However, the whole purpose of reusing profiles in this case is the ability to
leave some "inflexion points" in profiles, that are then selected for the
particular robot it gets applied to. These inflexion points are defined in the
parent profile with the 'tag' statement:

~~~
profile "Skid4" do
  # This profile has a tag of the provided type
  tag "wheel_control", Services::JointController
  # The tag can be used as a service would
  define "manual_control", Compositions::ManualControl.
    use('controller' => wheel_control_tag)
end
~~~

then, when reused, the child profile must say what to use for which tag:

~~~
profile "Robot" do
  use_profile RockUGVNav::Profiles::Skid4,
    'wheel_control' => OroGen::Robot::ControllerTask
end
~~~

Ensuring that all the inflexion points of a given profile are properly using
tags is not done every time the profile gets loaded (it would be too costly).
Instead, this is done at testing time with the it_should_be_self_contained
statement (which is enabled by default if you use syskit gen).

