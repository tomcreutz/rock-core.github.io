---
secondsection: system
section: System Management
title: Data Services
sort_info: 140
---

In Syskit, data services are used as placeholders for "actual" components. They
allow to create models that e.g. refer to "an orientation provider" without
actually saying _which_ orientation provider.

Data services represent both an _interface_ (i.e. a set of inputs and outputs)
and a _functionality_ (i.e. what happens to the data). This page will describe
how to define them, and how to represent that, sometimes, certain data services
can used in place of other ones. For instance, that a pose provider can be used
when an orientation provider is needed.

The relationship with other component models (task contexts and compositions)
is described later on these model's respective pages: [compositions](compositions.html) and
[task contexts](task_contexts.html).

A data service is defined with

~~~ ruby
data_service_type "Orientation" do
    output_port "orientation_samples",
        "base/samples/RigidBodyState"
end
~~~

This defines a Ruby module whose name is given as a string. It is defined in the
local namespace, so one can (and should) organize the data service definitions
using modules:

~~~ ruby
module Base
  data_service_type 'Orientation' do
    ...
  end
end
~~~

which is then referred to as Base::Orientation. The syntax for the input and
output ports is the same [than in oroGen files](../orogen/task_interface.html#ports).

<div class="block" markdown="1">
__Generation__ Template files for a new data service can be generated with

~~~
syskit gen srv subnamespace/name_of_service
~~~

for instance

~~~
syskit gen srv localization/pose
~~~

would create the Rock::Services::Localization::Pose service in
models/services/localization/pose.rb file, and update
models/services/localization.rb to require the pose file.
</div>

__Naming__ Services should be placed inside ${BundleName}::Services module (e.g.
Rock::Services for the rock bundle)
{: .block}

__Filesystem__ Data services are defined in models/services/. Whenever a data
service is used in a syskit file, the file that defines it should be required
first. Types must be loaded using import_types_from "project", where "project"
is the oroGen project that defines the type. Use [syskit
browse](general_concept.html#browsing) to find out what is defined where.
{: .block}


Model Hierarchy
---------------
The fact that a data service Y can be used instead of another one X (for instance, a Pose
data service that also provides an Orientation), is reflected by telling Syskit
that "Y provides X"

~~~ ruby
data_service_type "Pose" do
  output_port "pose_samples", "base/samples/RigidBodyState"
  provides Orientation
end
~~~

The resulting data service will have __both__ the ports of Y (here,
pose_samples) and the ports of X (orientation_samples) This is clearly not what
is needed here: whenever a Pose service is used in place of an Orientation
service, one would want the pose_samples port to __replace__
orientation_samples. This information must be given explicitly through _port
mappings_:

~~~ ruby
data_service_type "Pose" do
    output_port "pose_samples", "base/samples/RigidBodyState"
    provides Orientation, 'orientation_samples' => 'pose_samples'
end
~~~

In this definition, the name mapping on the "provides" stanza specifies that the
"orientation_samples" port of the provided service (Orientation) should be
replaced by the "pose_samples" port of the new service.

