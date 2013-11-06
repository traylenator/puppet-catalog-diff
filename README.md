What?
=====
A tool to compare two Puppet catalogs.

While upgrading versions of Puppet or refactoring Puppet code you want to
ensure that no unexpected changes will be made prior to doing the upgrade.

This tool will allow you to diff catalogs created by different versions of
Puppet.  This will let you guage the impact of a Puppet upgrade before actually
touching any of your nodes.

This tool is delivered as a Puppet Face. It thus requires a Puppet installation
to properly run.

The diff tool recognizes catalogs in yaml, marshall, or pson format.

Validation Process:

 - Grab a catalog from your existing machine running the old version
 - Configure your new Puppet master, copy the facts from your old master
   to the new one
 - Compile the catalog for this host on the new master:

      puppet master --compile fqdn > fqdn.pson

 - Puppet puts a header in some catalogs compiled in this way, remove it if present
 - At this point you should have two different catalogs. To compare them run:

      puppet catalog diff <catalog1> <catalog2>

 - Alternatively you can process a directory containing matching files
 - i.e. path/to/old/node_name.yaml and path/to/new/node_name.yaml
      puppet catalog diff <path/to/old> <path/to/new>

Example Output:

During the transition of 0.24.x to 0.25.x there was a serialization bug that
resulted in unexpected file content changes, I've recreated this bug in a tiny
2 resource catalog, the output below shows how this tool would have highlighted
this bug prior to upgrading any nodes.

    Resource counts:
      Old: 2
      New: 2

    Catalogs contain the same resources by resource title


    Individual Resource differences:
    Old Resource:
      file{"/tmp/foo":
        content => d3b07384d113edec49eaa6238ad5ff00
      }

    New Resource:
      file{"/tmp/foo":
        content => dbb53f3699703c028483658773628452
      }

Had resources simply gone missing - not the case here - you would have seen a
list of that.

This code only validates the catalogs, it cannot tell you if the behavior of
the providers that interpret the catalog has changed so testing is still
recommended, this is just one tool to take away some of the uncertainty.

You can get some inline help with:

    puppet man catalog

Installation?
-------------

This version works best with Puppet 3.0.0 or newer, simply install it with the
module tool into your module path:

    # puppet module install ripienaar-catalog_diff
    # puppet man catalog diff

Example Usage
----------
```shell
#!/bin/bash
test -f /tmp/2.7.23_catalogs && rm -rf /tmp/2.7.23_catalogs
test -f /tmp/3.3.1_catalogs && rm -rf /tmp/3.3.1_catalogs
export RUBYLIB=/etc/puppet/modules/catalog_diff/lib/
time puppet catalog pull /tmp/2.7.23_catalogs /tmp/3.3.1_catalogs kernel=Linux \
--old_server puppet27.example.com \
--new_server puppet30.example.com \
--debug \
--trace \
--yamldir /var/lib/puppet/yaml \
--ssldir /var/lib/puppet/ssl
time puppet catalog diff /tmp/2.7.23_catalogs /tmp/3.3.1_catalogs \
--debug \
--show_resource_diff \
--content_diff \
--trace
```

Changelog?
----------

 - 2012/10/20 - Make it a puppet face

 - 2010/10/19 - Change options and logic so it's easier to use comparing catalogs
                from the same version of puppet


Who?
----
R.I.Pienaar <rip@devco.net> / www.devco.net / @ripienaar
