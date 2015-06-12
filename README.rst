=========
zookeeper
=========

Formula to set up and configure a single-node zookeeper server.

.. note::

    See the full `Salt Formulas installation and usage instructions
    <http://docs.saltstack.com/en/latest/topics/development/conventions/formulas.html>`_.

Formula Dependencies
====================

* sun-java

Available states
================

.. contents::
    :local:

``zookeeper``
-------------

Downloads the zookeeper tarball from zookeeper:source_url (either pillar or grain), installs the package.

``zookeeper.server``
--------------------

Installs the server configuration and enables and starts the zookeeper service.
Only works if 'zookeeper' is one of the roles (grains) of the node. This separation
allows for nodes to have the zookeeper libs and environment available without running the service.

Zookeeper role and Salt Minion Configuration
============================================

The implementation by default relies on the existence of the _roles_ grain in your minion configuration - at least
one minion in your network has to have the *zookeeper* role which means that it is a zookeeper server. 

For this to work it is necessary to setup salt mine like below in /etc/salt/minion.d/mine_functions.conf:

::

    mine_functions:
      network.ip_addrs: []
      grains.items: []


This will allow you to use the zookeeper.settings state in other states to configure clients - the result of calling

::

    {%- from 'zookeeper/settings.sls' import zk with context %}

    /etc/somefile.conf:
      file.managed:
        - source: salt://some-formula/files/something.xml
        - user: root
        - group: root
        - mode: 644
        - template: jinja
        - context:
          zookeepers: {{ zk.connection_string }}

is a string that reflects the names and ports of the hosts with the zookeeper role in the cluster, like

::

    host1.mycluster.net:2181,host2.mycluster.net:2181,host3.mycluster.net:2181

and this will also work for single-node configurations. Whenever you have more than 2 hosts with the zookeeper role the formula will setup
a zookeeper cluster, whenever there is an even number it will be (number - 1). Additionally you can limit the maximum number of active server instances
by setting max_node_num in the zookeeper config pillar:

::

    zookeeper:
      config:
        max_node_num: 5

Alternative targeting methods
=============================

If you don't want to use custom grains to configure where your server instances are started there is a way to pick the hosts by other means.

::

    zookeeper:
      targeting_method: compound
      server_target: 'E@zknode'

This is an example where minion IDs (which have to start with zknode) are used to populate the list of servers. Most other targeting methods should work as well.
Before you use a non-default target in your pillar it is a good idea to test the target to see what list it will yield:

::

    # salt-run mine.get 'E@zknode' network.ip_addrs compound
      zknode1.zktest.local:
      - 192.168.111.100
      zknode2.zktest.local:
      - 192.168.111.101

would be the test for the example above.
Since the alternative targeting modes still require the salt mine (thus - a Salt master process) there is now also a manual override which allows to explicitly specify
a dictionary of minion_id and IP addresses.

::

    zookeeper:
      config:
        server_list_override:
          minion_id1: 10.0.0.11
          minion_id2: 10.0.0.12
          minion_id3: 10.0.0.13

Keep in mind that this pillar configuration emulates what normally comes out of the mine - you cannot make up keys or values here, instead you'll have to use the real minion IDs with the corresponding IP addresses.

