---
secondsection: Advanced
section: Build System
title: Cross-Set Dependencies
sort_info: 275
---

A given package set can tell autoproj to import another one BEFORE it gets
loaded. This is done in the 'imports' section of the source.yml file, using the
same syntax that in the 'package_sets' section of the manifest

For instance, the rock.core package set has

~~~ yaml
imports:
  - github: orocos-toolchain/autoproj
~~~

which means that autoproj will automatically import the orocos toolchain build
configuration when the main manifest lists rock.core. It tells you so at
checkout with

<pre>
autoproj: updating remote definitions of package sets
  checking out git:git://github.com/rock-core/package_set branch=master
  rock.core: auto-importing git:git://github.com/orocos-toolchain/autoproj
  checking out git:git://github.com/orocos-toolchain/autoproj
</pre>

and at update time with

<pre>
autoproj: updating remote definitions of package sets
  updating rock.core
  rock.core: auto-importing orocos.toolchain
  updating orocos.toolchain
</pre>

If some imports are not needed (or harmful) to your complete build
configuration, they can be disabled package-set-wide either by modifying the
manifest or programmatically

Imports will be disabled if you add the "auto_imports: false" flag in the manifest
to the package set's definition:

~~~ yaml
package_sets:
  - type: git
    url: git://github.com/rock-core/package_set
    auto_imports: false
~~~

Or by adding the following statement to autoproj/init.rb:

~~~ ruby
disable_imports_from 'rock.core'
~~~

