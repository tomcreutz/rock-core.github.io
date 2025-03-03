---
section: Basics
title: Configure components
sort_info: 120
---

Abstract 
---------------------------------

This tutorial will give you some hands-on experience on:
 
 * How to add configuration support for your component and
 * how to embed that configuration into the Ruby script.

Components and underlying libraries often need to be configured before starting them. This tutorial will teach you
how to use the oroGen component's properties to map to your library configuration option, and how to add the functionality
to the oroGen component. 

At the end of this tutorial, the component will look like:

![Component Interface](120_producer_config.png)
{: .align-center}

Let's go back into the MessageDriver library:

acd message_driver
{: .cmdline}

Configure a component 
---------------------------------
Some libraries will require configuration before you can use them, e.g. here we want to add a property to the MessageDriver library to
configure if the produced message should be presented in uppercase or not.
Therefore, "cd" into the message driver library (_~/dev/tutorials/message_driver_).

Add the configuration object code to _src/Config.hpp_ in the message
driver library:

~~~ cpp
#ifndef _MESSAGE_DRIVER_CONFIG_HPP_
#define _MESSAGE_DRIVER_CONFIG_HPP_

namespace message_driver
{

/**
* This configuration struct is a simple example of what you
* can do in order to wrap multiple configuration properties
* into a single object
* 
* This way you can manage configuration properties by grouping
* them into struct, and you don't have to change the oroGen 
* components interface when your configuration object changes
*/
struct Config
{
        bool uppercase;

        Config()
            : uppercase(false)
        {   
        }   

};

}
#endif // _MESSAGE_DRIVER_CONFIG_HPP_
~~~

Then adapt the message driver class (in _src/MessageDriver.cpp_ and
_src/MessageDriver.hpp_):

_MessageDriver.hpp:_

~~~ cpp
#include <message_driver/Message.hpp>
#include <message_driver/Config.hpp>
	...

        /**
        * MessageDriver configuration
        * \param config Configuration object
        */
        MessageDriver(const Config& config = Config());

	...
private:
        Config mConfig;

~~~

_MessageDriver.cpp:_

~~~ cpp
...
#include <algorithm>
...

Message MessageDriver::createMessage()
{
        Message msg("Message from MessageDriver");

        if(mConfig.uppercase)
            std::transform(msg.content.begin()
	    	 , msg.content.end()
	         , msg.content.begin()
	         , toupper);

        return msg;
}

...

MessageDriver::MessageDriver(const Config& config)
        : mConfig(config)
{
}

...
~~~

### Update the build configuration in src/CMakeLists.txt
This makes sure that the configuration header will also be installed.

Remember: The build configuration can be found here:

 _~/dev/tutorials/message_driver/src/CMakeLists.txt_

Update the file as follows:

~~~ text
rock_library(message_driver
    SOURCES MessageDriver.cpp
    HEADERS MessageDriver.hpp Message.hpp Config.hpp
    DEPS_PKGCONFIG base-types)
~~~


### Embed the configuration property into the oroGen component
 
Since we want a specific configuration step, the new
task should not be started without configuration, which should already have been
set by adding the statement 'needs_configuration' in the task description in the orogen file.

In order to embed the configuration property into the oroGen component, we will
have to do the following steps:

 1. __declare__ the property in the messages.orogen specification file (in _~/dev/tutorials/orogen/messages_).
 2. __move__ the construction/destruction of the driver from the task's
    constructor and destructor into configureHook / cleanupHook.

The final version of the component's code is in branch 'with_config' of _basic_tutorials/orogen/message_producer_ (_git checkout with_config_). 

__(1)__ We modify the task description in the messages.orogen file.
We add a property of the configuration type as follows (see also [Task
Interface](../orogen/task_interface.html)).

__(2)__ Since we want a specific configuration step, the new
task should not be started without configuration, i.e. this is why we add the
statement 'needs_configuration'.

~~~ ruby
import_types_from "message_driver/Config.hpp"

task_context "Producer" do
  needs_configuration

  property "config", "message_driver/Config"
  ...

task_context "Consumer" do
  needs_configuration

  property "config", "message_driver/Config"
  ...
end
~~~

__(2)__ Finally, we remove the allocation and deallocation from the constructor and destructor, since it will be moved into the configureHook and the cleanupHook:

In _~/dev/tutorials/orogen/messages/tasks/Producer.cpp_:

~~~ cpp
bool Producer::configureHook()
{
    if (! ProducerBase::configureHook())
        return false;

    message_driver::Config configuration = _config.get();
    mpMessageDriver = new message_driver::MessageDriver(configuration);

    return true;
}
~~~

~~~ cpp
void Producer::cleanupHook()
{
    ProducerBase::cleanupHook();

    delete mpMessageDriver;
}
~~~

### Build the task

Now, build the task: Assuming that you are in the message_producer folder or in one of its
subfolders call:

~~~ cpp
amake
~~~

### Embedding configuration into the ruby script

Now, "cd" into your folder _~/dev/tutorials/orogen/messages/scripts_, 
and copy 'start.rb' to a new file 'configure.rb' -
you will reuse 'start.rb' at a later stage. 
Modify 'configure.rb' according to the following 
code block:

~~~ ruby
require 'orocos'

include Orocos
Orocos.initialize

Orocos.run 'messages::Producer' => 'messages' do  

    messages = Orocos.name_service.get 'messages'

    # 'config' is the name of the property
    messages.config do |p|
        p.uppercase = true
    end

    # Call to configure is required for this component
    # since it has been generated with 'needs_configuration'
    messages.configure
    messages.start

    reader = messages.messages.reader

    while true
        if msg = reader.read_new
            puts "#{msg.time} #{msg.content}"
        end 

        sleep 0.5 
    end 
end
~~~

### Run it

Now you can run the script. 

~~~ cpp
ruby configure.rb
~~~

Again, you should see something similar to the following. You can switch
between uppercase and mixed case printing by using your newly defined configuration options. With the script above, you should see something like the following:

~~~ text
Wed Aug 03 09:40:28 +0200 2011 MESSAGE FROM MESSAGEDRIVER
Wed Aug 03 09:40:29 +0200 2011 MESSAGE FROM MESSAGEDRIVER
~~~

### Use a configuration file
Property values can be read from a file, what is especially handy in case of more properties to set. The configuration file is a yaml-file which organises the properties in a structured way. For the present example, create a file, e.g. _messages_config.yml_ with the
following content:

~~~ yaml
\--- name:default
config:
    uppercase: True
~~~

Note that yaml uses indentation for organizing the data. The script can now be changed to use this file. Instead of

~~~ ruby
    # 'config' is the name of the property
    messages.config do |p|
        p.uppercase = true
    end
~~~

put

~~~ ruby
    # load property from configuration file
    messages.apply_conf_file("messages_config.yml", ["default"])
~~~

The result when running the script should be the same as above. It is possible to
have one configuration file per task context and then load the properties for each task
from the corresponding file. Of course the properties in the file have to match the
properties given in the orogen definition. A good way to create a proper configuration
file is to generate such a file from the task model definition. To do so, run the 
following command from the shell:

~~~ sh
oroconf extract messages::Producer --save messages.yml
~~~

_oroconf_ is the command to access the configuration of tasks. It can do more than 
extracting configuration files (_\-\-help_), but we will stick to the command _extract_
for now. The first argument after _extract_ gives the task model which is the orogen
name plus double colon plus task name. Behind _\-\-save_, the file is given to which the
data should be written. You might note that there are some variations to the hand 
written file:

~~~ yaml
\--- name:default
# no documentation available for this property
config:
  uppercase: false
~~~

In this file, __uppercase__ is _false_, which is the default
setting for the property (see configuration constructor). A comment before the declaring
the property is automatically picked up as documentation line.

~~~ ruby
    # Configuration property for the message driver
    property("config","message_driver/Config")
~~~

The first line is the name for the configuration. It is possible to have multiple 
configurations in one file which can be distinguished by their name. Think of a
file called 'messages_multi.yml':

~~~ yaml
\--- name:default
# Configuration property for the message driver
config: 
  uppercase: 0

\--- name:uppercase
# Configuration property for the message driver
config: 
  uppercase: 1
~~~

Then one can apply the uppercase configuration with
    

~~~ ruby
    # load property from configuration file
    messages.apply_conf_file("messages_multi.yml", 
        ["uppercase"])
~~~

For more information on configuration files, have a look [here](../runtime/configuration.html).

### Dynamic properties
"Normal" properties - such as the ones dealt with in the preceding tutorial -
should only be used for the starting configuration of the component and should
not be changed at runtime. If a component allows dynamic changes of properties,
those have to be declared explicitly as explained hereafter. 

The framework automatically creates a callback function which is called
within the task's thread. This function makes it possible that changes of
properties can be accepted or rejected if they are invalid.

In the messages.orogen file, a dynamic property must be declared as
follows:

~~~ ruby
property("some_dynamic_property","double",0.1).dynamic
~~~

Inside the task, the declared function will look as follows:

~~~ cpp
bool Task::setSome_Dynamic_Property(double value)
{
 //Implement the needed steps here
 return(orogen_name::TaskBase::setSome_Dynamic_Property(value));
}
~~~

Note that if you modify a task which already exists, the code must be copied out
of the _templates/tasks_ folder!

The base class is called to make sure that superordinated tasks are called and
properties can be changed by those tasks. If a superordinated task rejects the
changes, the return value should be _false_.

If the task has not yet been configured, consequently is not in a _running_
or _stopped_ state, the callback will NOT be triggered in case of an external
change of the property. What will happen, in that case, is that the property
__some_dynamic_property_ will be updated.

As a simplification for the user, if a task has dynamic properties, a callable
function is generated  which calls all callback functions with the new 
parameters. By calling _updateDynamicProperties()_ inside the configure hook
after the component is set up, the changes of the dynamic properties can be 
applied, avoiding code duplication.

#### Special case inheriting tasks
If a non-dynamic property shall become dynamic inside the inheriting task, it 
must be declared as follows in the messages.orogen file:

~~~ ruby
make_property_dynamic("some_non-dynamic_property")
~~~

Summary
----------------
In this tutorial, you have learned: 

 * How to embed configuration into an oroGen component.
 * How to use the _templates/_ subfolder of an oroGen component.
 * How to set configuration properties for your component in a ruby script.
 * How to generate and use configuration files.

In the next tutorial, you will learn how to create a data driven component and how to connect it
to an existing component.

Progress to the [next tutorial](130_basics_connect_components.html).

