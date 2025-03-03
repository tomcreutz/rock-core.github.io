---
title: Overview 
---

Rock offers both a __rich development environment__ and a collection of
__ready-to-use packages__. This documentation pages describe the development
environment as well as some important "core" libraries. 
{::comment}
For the package
documentation, go to the [package list](/pkg)
{:/comment}


Development Workflow
--------------------

<div class="fullwidth_video_400">
<iframe data-src="https://www.youtube.com/embed/UkZYiw7crbw" width="400" height="290" frameborder="0">Youtube Video</iframe>
</div>

First and foremost, development in Rock always starts with __[creating a
library](tutorials/100_basics_create_library.html)__. As a guideline, this
library has to be independent of Rock's component-based integration framework.
That's right, even if you don't use Rock's tooling, [feel free to use its
drivers and algorithms](packages/outside_of_rock.html) Then, this library gets
integrated in __oroGen__, Rock's component-oriented integration framework.

For runtime, network of Rock components are often setup and managed using [the
Ruby programming language](http://ruby-lang.org). Bindings to Ruby allow to start
processes, start and stop components, connect them together and bind them to
user interfaces in a very flexible way.

[Tutorials](tutorials/index.html) will guide you through getting to grips with
the process, from a library to running network of components.

Data Analysis
-------------
<div class="fullwidth_video_400">
<iframe data-src="https://www.youtube.com/embed/PhHFzCNvjlQ" width="400" height="290" frameborder="0">Youtube Video</iframe>
</div>

At this point, Rock offers extended support for runtime as well as offline data
analysis. [Logging](data_analysis/index.html) is an integral part of the
development workflow: it can be used for post-mortem analysis as well as to
test components through log replay mechanisms. Then, __Vizkit__ kicks in. is
both an oroGen-independent library of Qt-based widgets and OpenSceneGraph-based
3D visualizers, and a Ruby library that allows to seamlessly display both
logged and live data. Extending it with new widgets and visualizers is
straightforward.

Advanced System Management
--------------------------
Finally, Rock gives you rock-roby, a [model-based system management
layer](system/index.html) which will allow you to manage complex networks of
components both at design and running time. Have a look first at the
[corresponding tutorials](system_management_tutorials/).

<div class="fullwidth_video_400">
<iframe data-src="https://www.youtube.com/embed/QKl_0pGIfqI" width="400" height="300" frameborder="0">Youtube Video</iframe>
</div>

