---
section: Basics
title: Creating libraries
sort_info: 100
---

Abstract
--------------

This tutorial will give you some hands-on experience on:

 * How to create libraries in Rock and
 * how to embed them into the build system of Rock.

If you don't want to execute the following steps yourself, the result can
also be found in the package 'tutorials/' after you [installed the tutorial
package set](index.html#installing).

For this tutorial, it is assumed that your autoproj installation can be found in
~/dev.

Creating libraries
-----------
Before you start developing components, you will need to think about the
functionality that is required for your component. This tutorial teaches you
how to write a message producer and a message consumer component, which will
pass timestamped messages between each other.  For the message producer and
the consumer, we will create a message library to provide message creation and
message printing functionality. This library will make use of the existing
package base/types, which simplifies creation of timestamps.

Now, that it is clear what functionality is needed, you can start writing the
library.  Note, that Rock strongly suggests to encapsulate your main
functionality in libraries. Thus, your library code remains independent of the
actual framework in use, and can be easily reused and maintained separately
from any framework that wraps this functionality.

Rock allows you to create a C++ library from an existing template. Calling the
command 'rock-create-lib' starts a command line dialog to create a library
called 'tutorials/message_driver'. Remember: ~/dev is the path to the Rock
installation. It might differ on your machine.

~~~ text
~/dev$ rock-create-lib tutorials/message_driver
~~~

Once the the create script is called, a command line dialog is started which will
request basic information to configure the template for you. The following
output is the output of the script AND the expected answers. Do not copy/paste
the whole block at once !

~~~ text
Initialized empty Git repository
in ~/dev/tutorials/message_driver/.git/
Do you want to start the configuration of
the cmake project: message_driver
Proceed [y|n]
y
------------------------------------------
We require some information to update the manifest.xml
------------------------------------------
Brief package description (Press ENTER when finished):
A message_driver for the basic Rock tutorial
Long description:
This is a library that allows message production
and message handling for the the basic Rock tutorial
Author:
New user
Author email:
new-user@rock-robotics.org
Url (optional):

Enter your dependencies as a comma separated list.
Press ENTER when finished:
base/types
Initialized empty shared Git repository
in ~/dev/tutorials/message_driver/.git/
[master (root-commit) 37aa552] Initial commit
8 files changed, 108 insertions(+), 0 deletions(-)
create mode 100644 CMakeLists.txt
create mode 100644 INSTALL
create mode 100644 LICENSE
create mode 100644 README
create mode 100644 manifest.xml
create mode 100644 src/CMakeLists.txt
create mode 100644 src/Dummy.cpp
create mode 100644 src/Dummy.hpp
create mode 100644 src/Main.cpp
create mode 100644 src/dummyproject.pc.in
Done.
~~~

The newly created package comes in some kind of ready-to-run: You can build and install it right away using the build tool [autoproj](../autoproj/basic_usage.html).

~~~ text
amake tutorials/message_driver
~~~

***NOTE***: The template project will generate several files for you in the src/ and
test/ directories. This is meant to give you an example, but these files are
usually deleted as soon as you start developing the library. Don't forget to
remove the corresponding references from src/CMakeLists.txt and
test/CMakeLists.txt as well.
{: .warning}

### Adding the required functionality
The library does not contain message handling capabilities yet. So we create
three new files, i.e. a header file Messages.hpp which will contain the message
type that is used to transport message between components, and a header and
source file for the library functionality, the MessageDriver.
Put all those files into the src/ folder of the
newly created package (~/dev/tutorials/message_driver/src).

***NOTE***: Rock recommends to stick to use CamelCase for new structures, and to
name the files after the class it defines (i.e. Message.hpp defines the Message
structure, MessageDriver.hpp declares the MessageDriver class and
MessageDriver.cpp defines it). However, due to historic reasons, not all packages
of Rock conform to this style. See [these guidelines](http://rock.opendfki.de/wiki/WikiStart/Standards/RG4)
for more information
{: .warning}

In Rock, common C++ types are defined [in the base/types package]({rock_pkg:
base/types}). If some type you need is defined here, it is __highly
recommended__ to use the common type. In our case, we want to timestamp the
message that our library will manipulate.  If we have a look at the
[base/types](../../api/base/types/structbase_1_1Time.html)) API documentation
we can see that there is a base::Time type that suits our needs.

First things first, we need to declare the dependency on the base/types package.
Edit manifest.xml and make sure that the following line is there (it should already be there):

~~~ xml
<depend package="base/types" />
~~~

The message class definition will be contained in a header called
src/Message.hpp, to follow Rock naming guidelines (a Message class should be
declared in the src/Message.hpp header). It should contain the following code:

~~~ cpp
#ifndef _MESSAGE_DRIVER_MESSAGE_HPP_
#define _MESSAGE_DRIVER_MESSAGE_HPP_

#include <string>
#include <base/Time.hpp>

namespace message_driver
{
    struct Message
    {   
        // The message content
        std::string content;

        // The timestamp when the message was created
        base::Time time;

	// Default Constructor -- required
        Message()
                : content()
                , time(base::Time::now())
        {   
        }   

        Message(const std::string& msg)
                : content(msg)
                , time(base::Time::now())
        {   
        }   
    };  

}
#endif // _MESSAGE_DRIVER_MESSAGE_HPP_
~~~

To follow the Rock guidelines, the MessageDriver class should be declared in
src/MessageDriver.hpp and defined in src/MessageDriver.cpp.
src/MessageDriver.hpp should therefore contain:

~~~ cpp
#ifndef _MESSAGE_DRIVER_HPP_
#define _MESSAGE_DRIVER_HPP_

#include <message_driver/Message.hpp>

namespace message_driver
{

class MessageDriver
{

public:
    /**
     * Create a timestamped message
     * \return A timestamped message
     */
    Message createMessage();

    /**
     * Print a message to stdout
     * \param msg Message to be printed
     */
    void printMessage(const Message& msg);
};

}

#endif // _MESSAGE_DRIVER_HPP_
~~~

And, finally, the driver implementation creates the timestamped message, and is
in src/MessageDriver.cpp:

~~~ cpp
#include "MessageDriver.hpp"
#include <iostream>

namespace message_driver
{

Message MessageDriver::createMessage()
{
        Message msg("Message from MessageDriver");
        return msg;
}

void MessageDriver::printMessage(const Message& msg)
{
        std::cout << "[" << msg.time.toString()
                  << "] " << msg.content
                  << std::endl;
}

}
~~~


### Build

Once you created a library in Rock, integrate it into the build system autoproj.
The first step towards integration into the build system is adding the newly
created files to the src/CMakeLists.txt file, since Rock C++ libraries
use CMake for the build process by default. You can use standard CMake directives, but Rock
also comes with some CMake macros that facilitate setting up libraries, and
resolving any required dependencies (have a look [here](../packages/cmake_macros.html) for details).

~~~ text
rock_library(message_driver
    SOURCES MessageDriver.cpp
    HEADERS MessageDriver.hpp Message.hpp
    DEPS_PKGCONFIG base-types)

# Do not forget to remove the rock_executable line that
# compiles Main.cpp, as it is not used anymore
~~~

After adapting the CMakeLists.txt, add the package to the build configuration, so
that eventually you can embed the library into oroGen components. The easiest
way to adapt the build configuration is by adding the package to the manifest's
layout section. Thus, edit ~/dev/autoproj/manifest and add the package to the
layout section. Package management in detail is discussed in [Adding
packages](../autoproj/adding_packages.html).
{: #add-to-manifest}

***NOTE***: When adding the package, make sure you use the same indentation as
the previous line, here '- rock.toolchain'. The manifest file is parsed as
.yaml, and thus relies on proper indentation.
{: .warning}

~~~ text
package_sets:
  - github: rock-core/package_set

# Layout. Note that the rock.base, rock.toolchain
# and orocos.toolchain sets are imported
# by other rock sets.
layout:
   - rock.core
   - tutorials/message_driver
~~~

### Build it
Just verify that your component builds.

~~~ text
~/dev/tutorials/message_driver$ amake
~~~

### Test it
Optionally, you can modify the Boost test_suite created by default and test our MessageDriver library.
So we create a new test file, i.e.: test/test_MessageDriver.cpp which will create and print a Message:

~~~ cpp
#include <boost/test/unit_test.hpp>
#include <message_driver/MessageDriver.hpp>

using namespace message_driver;

BOOST_AUTO_TEST_CASE(it_should_not_crash_when_message_is_created)
{
    message_driver::MessageDriver messageDriver;

    Message message = messageDriver.createMessage();

    messageDriver.printMessage(message);
}
~~~

In the same manner as for building the library, the step towards integration into the build system is adding the newly
created file to the test/CMakeLists.txt file

~~~ text
rock_testsuite(test_suite suite.cpp
    test_MessageDriver.cpp
    DEPS message_driver)
~~~

Also enable test building for the library via command line-

~~~ text
~/dev/tutorials/message_driver$ autoproj test enable tutorials/message_driver
~~~

After adapting the CMakeLists.txt for the test_suite and enabling test building we can just build the library again and run the test_suite.

~~~ text
~/dev/tutorials/message_driver$ amake
~/dev/tutorials/message_driver$ ./build/test/test_suite
Running 1 test case...
[20140123-16:20:14:704568] Message from MessageDriver

*** No errors detected
~~~

You already finished your first Rock tutorial.

Summary
----------------------------
In this tutorial you have learned to:

 * Create a C++ library from the Rock template and
 * embed new packages into the build system.

In the next tutorial you will learn how to create an oroGen component and embed your library into it.

Progress to the [next tutorial](110_basics_create_component.html).
