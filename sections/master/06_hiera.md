!SLIDE smbullets small
# Parameter Lookup

* Separation of configuration and data
* Automatic Lookup of parameters was introduced in Puppet 3
* Improvements in Puppet 4.9
* Default is
 * Hiera, a hierachical lookup
 * One global configuration

~~~SECTION:handouts~~~

****

Puppet 3 added an automatic lookup of parameters to allow the separation
of configuration and data. The tool used for this is Hiera, a hierachical
lookup. With Puppet 4.9 it was improved to add three layers of lookup, the
classic global hiera, environment data and module data.

~~~ENDSECTION~~~


!SLIDE smbullets small
# Hiera

* Hierarchy of lookups is configurable
 * Hierarchy level can be fix or use variables
 * Hiera 4 was never released as stable, replaced in Puppet 4.9
 * Environment and module configuration uses Hiera 5
* Different backends are avaiable
 * YAML/JSON - default
 * EYAML - YAML with encrypted fields
 * MySQL/PostgreSQL - Database lookup
 * LDAP and more

<pre>
---
version: 5
defaults:
  datadir: data
  data_hash: yaml_data
hierarchy:
  - name: "Per-node data (yaml version)"
    path: "%{::hostname}.yaml"
  - name: "Other YAML hierarchy levels"
    paths:
      - "%{::domain}.yaml"
      - "defaults.yaml"
</pre>

~~~SECTION:handouts~~~

****

Hiera 3 uses a different configuration format, has less capabilities and is
deprecated since Pupept 4.9. Hiera now is complete integrated in Puppet. Hiera 3
was a seperated project. The hierachy levels can be a fix string or use variables
from Puppet. With version 5 the file suffix i.e. .yaml is required.

All features and the complete syntax is described in the online documentation:
https://docs.puppet.com/puppet/4.9/hiera_config_yaml_5.html

Note: The old hiera_* functions are deprecated now and should be relaced with the
lookup function.

The new lookup function has an options hash to configure the search and merge behavoir.
See https://docs.puppet.com/puppet/4.9/hiera_merging.html.

~~~PAGEBREAK~~~

The automatic lookup uses the namespace to find a variable, for example parameter
manage_service of module apache has to be apache::manage_service for the automatic lookup.

Hiera can utilize different backends which are pluggable. Per default it only
provides a file based configuration in YAML or JSON format. Other common formats
are eYAML which adds encrypted fields to default YAML or Database lookups. Furthermore
specific solutions like Foreman are avaiable which use an existing datastore as backend.

~~~ENDSECTION~~~


!SLIDE smbullets small
# Lab ~~~SECTION:MAJOR~~~.~~~SECTION:MINOR~~~: Environment Data

* Objective:
 * Create a parameterized apache configuration using hiera
* Steps:
 * Use Hiera for environment data
 * Enable ssl per default
 * Add a new parameter to accept a hash of vhosts
 * Define the vhost hash for your client


!SLIDE supplemental exercises
# Lab ~~~SECTION:MAJOR~~~.~~~SECTION:MINOR~~~: Environment Data

## Objective:

****

* Create a parameterized apache configuration using hiera

## Steps:

****

* Use Hiera for environment data

Hiera can use different hierachies per every environment. Therefor you have to
put you hiera.yaml into the directory of the desired environment. The specified data
directory 'datadir' is a relative path starting from the same directory
where hiera.yaml belongs to.

* Enable ssl per default

Use a fix hierarchy level like common or default to provide default parameters.
Your your apache module enabling ssl per default will require a parameter
apache::ssl set to true.

* Add a new parameter to accept a hash of vhosts

Enhance your module to accept a hash of vhosts and iterated over it creating
apache::vhost resources.

* Define the vhost hash for your client

The variable trusted.certname can be used to define parameters on node level.


!SLIDE supplemental solutions
# Lab ~~~SECTION:MAJOR~~~.~~~SECTION:MINOR~~~: Environment Data

****

## Create a parameterized apache configuration using hiera

****

### Enable Hiera for environment data

    $ vim ~/puppet/hiera.yaml
    ---
    version: 5
    defaults:
      datadir: data
      data_hash: yaml_data
    hierarchy:
      - name: "Per-node data (yaml version)"
        path: "nodes/%{trusted.certname}.yaml"

      - name: "Other YAML hierarchy levels"
        paths:
          - "common.yaml"

    $ mkdir -p ~/puppet/data/nodes

### Enable ssl per default

    $ vim ~/puppet/data/common.yaml
    ---
    apache::ssl: true

### Add a new parameter to accept a hash of vhosts

Make also sure templates used in apache::vhost point to a relative path if not already
changed before.

    $ vim ~/puppet/modules/apache/manifests/init.pp
    class apache (
      ...
      Hash                     $vhosts      = {},
    ) inherits apache::params {
    ...
      $vhosts.each | String $name, Hash $vhost | {
        apache::vhost { $name :
          * => $vhost,
        }
      }
    }

### Define the vhost hash for your client

    $ vim data/nodes/agent-centos.localdomain.yaml
    ---
    apache::vhosts:
      foobar:
        ip: 127.0.0.5

Do not forget to commit and push these changes before testing!


!SLIDE smbullets small
# Data in modules

* Similar to environment data
* Could replace params.pp pattern
 * No inheritance
 * Better separation between code and data
 * Easier enhanced for supporting additional operatingsystems

~~~SECTION:handouts~~~

****

Data in modules could replace perhaps the params.pp pattern commonly used for
separation of parameters from code logic. A good blogpost on this can be found
on https://www.devco.net/archives/2016/01/08/native-puppet-4-data-in-modules.php

~~~ENDSECTION~~~
