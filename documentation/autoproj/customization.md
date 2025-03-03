---
section: Build System
title: Customizing the installation
sort_info: 48
---

Changing the installation's layout
----------------------------------

The <tt>layout</tt> section of <tt>autoproj/manifest</tt> offers two things:

 * select which packages/package sets to build and
 * build packages in specific subdirectories

This section lists packages or package sets that ought to be built by autoproj.
For instance, in the following example, only the <tt>orogen</tt> and
<tt>tools/orocos.rb</tt> packages of the rock.core package set and their
dependencies will be built. The other will be ignored.

~~~ yaml
package_sets:
  - github: rock-core/package_set

layout:
  - orogen
  - tools/orocos.rb
~~~

Instead of giving a package name, the name of a package set can be provided,
which translates into "all the packages of that set".

As we mentionned before, the layout mechanism also allows you to place packages
in subdirectories of the main installation. For instance, the following snippet
will build the <tt>typelib</tt>, <tt>utilmm</tt> and <tt>utilrb</tt> libraries
of orocos-toolchain into the <tt>lib/</tt> subdirectory and all the <tt>orocos/</tt> packages in the root.

~~~ yaml
layout:
  - lib:
    - typelib
    - utilmm
    - utilrb
  - orocos/
~~~

Configuration files like <tt>autoproj/manifest</tt> are YAML file. As such, they
are sensible to indentation. The snippet above should be read as: in the layout,
there is first a "lib" part and second the packages whose names start with
"orocos/" _(first level of indentation)_. In the "lib" part, the packages are
"typelib", "utilmm" and "utilrb" _(second level of indentation)_. 
{: .warning}

Alternatively, the example above could be written:

~~~ yaml
layout:
  - lib: [typelib, utilmm, utilrb]
  - orocos/
~~~

Finally, names of sublayouts can be used as arguments in the autoproj command
line, instead of package names:

autoproj build lib
{: .commandline}

Removing packages from the build (<tt>exclude_packages</tt>) {#exclude_packages}
--------------------------------
Instead of having to list all packages that you do want to build, it is possible
to list the packages that you don't want to build. Simply list them in the
<tt>exclude_packages</tt> section of the manifest file. In the following example, all
of the rock-core package set will be built *but* the pocolog package.

~~~ yaml
package_sets:
  - github: rock-core/package_set
layout:
  - rock.core
exclude_packages:
  - tools/pocolog
~~~

If another required package (i.e. a package listed in the layout) depends on an
excluded package, the default behaviour is to fail with
[an error](error_messages.html#exclusions).

For some package sets, such as the rock package set, this behaviour can be
relaxed by setting the package set's weak_dependencies flag to true in e.g. the
package set's init.rb or autoproj/init.rb:

``` ruby
metapackage('rock').weak_dependencies = true
```

In this case, autoproj will only issue a warning and simply automatically
exclude the offending packages from the build.

Using packages that are already installed (<tt>ignore_packages</tt>) 
-----------------------------------------

If some packages are already installed elsewhere, and you want to use that
version instead of the one listed in the package sets, list them in the
<tt>ignore_packages</tt> section of the manifest. In the following example, the
<tt>rtt</tt> package will *not* be built by autoproj, but autoproj will assume
that it is present.

~~~ yaml
package_sets:
  - github: rock-core/package_set
layout:
  - rock.core
exclude_packages:
  - tools/pocolog
ignore_packages:
  - rtt
~~~

Unlike with <tt>exclude_packages</tt>, no error is generated for ignored
packages that are depended-upon by other packages in the build. This is because
ignore_packages is meant to list packages that are already installed outside of
autoproj, while exclude_packages lists what you do **not** want to have at all.

Local overrides of version control information
----------------------------------------------

The <tt>autoproj/overrides.yml</tt> allows you to override version control information
for specific packages. It has the same format than the source.yml file of
package sets, so [check that page out](advanced/importers.html) for more information.

This file can in particular be used to avoid further updates to a given software
package. Simply do:

~~~ yaml
overrides:
  - rtt:
    type: none
~~~

To track a different branch that the default branch, you will have to do:

~~~ yaml
overrides:
  - rtt:
    branch: experimental
~~~

Tuning what files autoproj looks at to determine if a package should be updated
-------------------------------------------------------------------------------
When a package A depends on a package B, autoproj checks if some of the files in
B's directory are newer than the latest build of A. If it is the case, it will
trigger the build of B and then the one of A.

Autoproj has a default list of files that it should ignore. Unfortunately, it
may be that you are generating some files in the source directory that autoproj
interprets as new files and trigger builds.

If you have the impression that autoproj does too many rebuilds, run the build
once with the <tt>\-\-list-newest-files</tt> option. For instance,

~~~
autoproj --list-newest-files fast-build
~~~

If you find some files that should be ignored, add them either to the package
sets (i.e. autoproj/remotes/blablab/init.rb) or in autoproj/init.rb with

~~~ ruby
ignore /a_regular_expression/
~~~

where /a_regular_expression/ is a regular expression matching your files. For
instance, to eliminate all files that have an extension in ".swp", one would do

~~~ ruby
ignore /\.swp$/
~~~

Building local packages
-----------------------

You can list local packages that are not in an imported package set by placing
the definitions in autoproj/, in a file ending with <tt>.autobuild</tt>. See [this
page](advanced/autobuild.html) for information on how to write autobuild files.

Setting up the path to specific commands (make, parallel builds)
----------------------------------------------------------------

The autobuild API allows to specify what specific installed command to use for
each tool needed during the build. These commands can be used in
<tt>autoproj/init.rb</tt> to tune the build system. Example:

~~~ ruby
Autobuild.programs['make'] = '/path/to/ccmake'
Autobuild.parallel_build_level = 10 # build with -j10
~~~

More complex customization
--------------------------

More complex customization can be achieved by accessing the [Autoproj
API](http://rubydoc.info/gems/autoproj/frames) and
the [Autobuild API](http://rubydoc.info/gems/autobuild/frames) directly in the <tt>autoproj/init.rb</tt> and
<tt>autoproj/overrides.rb</tt>
files. The former is loaded before all source files and the latter is loaded
after all source files.

Some examples:

 *  fixing dependencies: if a dependency is not listed in a package's manifest,
    then you can fix it by adding the following in <tt>autoproj/overrides.rb</tt>

    ~~~ ruby
    a = Autobuild::Package['a_package']
    a.depends_on "other_package"
    ~~~

 *  changing the configuration of some cmake package:

    ~~~ ruby
    a = Autobuild::Package['a_package']
    a.define "CONFIG_VAR", "CONFIG_VALUE"
    ~~~

Building packages selectively on the command line
-------------------------------------------------

The autoproj command line accepts subdirectories defined in the layout as well
as package names.

For instance, with the following layout:

~~~ yaml
layout:
  - typelib
  - asguard:
    - modules/base
    - modules/logger
~~~

If the command line is

autoproj build modules/logger
{: .commandline}

then modules/logger and its dependencies will be built. Idem, if the command line is:

autoproj build asguard
{: .commandline}

then all packages or asguard/ are built, including their dependencies

