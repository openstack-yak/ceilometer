Multi-node Configuration Setup for Development
==============================================

(e.g. Triage & Load Testing)
============================

Getting started
---------------

The overview we’re looking at is
`*overview-hostlayout.html* <http://docs.openstack.org/developer/openstack-ansible/liberty/install-guide/overview-hostlayout.html>`__
(now an obsolete version) and (what used to be the following) page,
`*overview-requirements.html* <http://docs.openstack.org/developer/openstack-ansible/install-guide/overview-requirements.html>`__.

We will engage in an exercise to demonstrate how we create a private
tenant network inside of openstack and demonstrate ICMP traffic between
two instances on that network.

First, let's start with a test network to verify we understand the steps
in configuration of a network (that we have traffic pathways):

1. Go to
       `*https://cloud1.osic.org/project/networks/* <https://cloud1.osic.org/project/networks/>`__
       and “Create Network”;

   -  You want to create a private (i.e. no external access) tenant
          network

      -  Ref:
             `*https://www.arin.net/knowledge/address\_filters.html* <https://www.arin.net/knowledge/address_filters.html>`__

      -  Submask ref:
             `*https://en.wikipedia.org/wiki/Classless\_Inter-Domain\_Routing* <https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing>`__

   -  That means “disable gateway” in the subnet creation;

2. Next, go to
       `*https://cloud1.osic.org/project/instances/* <https://cloud1.osic.org/project/instances/>`__
       and create some instances, one at a time, so we can test the
       private network

   -  “Launch Instance” (upper right) will begin the process of creating
          an instance

   -  In the “Details” tab:

      -  Create instances (m1.tiny is too small to boot current ubuntu
             images; use m2.tiny) to test connectivity; Recommended
             image is 14.04

      -  In the “Access & Security” tab:

      -  set “default” security group;

      -  When you create your instance, be sure to use a keypair on your
             “jump-box” (or deploy machine, from which you access your
             network);

         -  If you’re using an existing jump-box, use the root’s
                ~.ssh/id\_rsa.pub key (add it if you don’t already have
                it), or one which has a private key pair also

   -  In the “Networking” tab:

      -  You’ll want to your instance connected to both the your private
             network (created above), and the GATEWAY\_NET.

   -  After you create a VM and attach it to 2 networks (GATEWAY\_NET
          and Private Net) login to your VMs using GATEWAY\_NET assigned
          ip.

      -  Check machines ethernet config using “ ip a “ command to ensure
             the expected inet address for the private network is
             assigned and the appropriate interface is UP (not DOWN).
             That is, you should see an interface showing each of:

         -  The GATEWAY\_NET address (as shown in the Instance IP
                addresses);

         -  The private net address (e.g. a 192.\*.\*.\* address)

      -  If you find there is misconfiguration for eth1 (or whatever
             interface it appears as, this seems to be a problem with
             the linux 14.04 image on cloud1). Follow these steps for
             configuring your inet config for eth1 and then RESTART each
             vm.

         -  Create the file /etc/network/interfaces.d/eth1.cfg with the
                following format, as root, replacing addresses,
                netmasks, and interfaces as needed:
                `*https://gist.github.com/stevelle/03bec81f20c19ca5dca96ab846a2f4d7* <https://gist.github.com/stevelle/03bec81f20c19ca5dca96ab846a2f4d7>`__

            -  Adjust the filename (here, .../interfaces/eth2.cfg) to
                   match the device needing configuration on our
                   installation;

      -  As an alternative to restarting the following commands can be
             used (substitute the correct ip, netmask in cidr format,
             and interface for that in the example). Note: this means
             you still need to write the interfaces.d/ file!

         -  $ DEVICE=”eth1” # replace as appropriate for your
                installation

         -  $ ip addr add 10.10.20.5/24 dev ${DEVICE}

         -  $ ip link set ${DEVICE} up

    What’s the correct interface? See the output of \`ip a\` for the
    interface missing an IP address, and which is not up.

    Your installation may have one file with many interfaces setup in
    it; create your own file.

-  If, when using sudo or logging in as root, you get message indicating
       the (hostname) could not be resolved, add the hostname to
       /etc/hosts at the end of the loopback line (e.g. 127.0.0.1
       localhost somehostname)

1. Test connectivity:

   -  From the jump-box (the deploy machine), as root, ssh to your
          *first test machine’s gateway IP*;

      -  If you can’t access to your jump-box, be sure your SSH public
             key has been added to the jump-box, and that the matching
             private key is in $HOME/.ssh directory of your account on
             the machine you are trying to connect from.

         -  if someone else configured the jump-box your are using, you
                should have given them your public key for them to add
                to the appropriate account on jump box (if you are
                giving someone access, add their public key to the
                authorized\_keys file in the login account’s $HOME/.ssh/
                directory);

         -  If you don’t know how to configure a jump box, see the
                [First Box setup - TBD, if we need it] doc.

   -  From the first test box, ping your second machine’s private net
          address

      -  If this works, you should be able to ping from any machine on
             the private local network to any other;

The above process outlines the detail of establishing a network under
OpenStack (Liberty). It will be used during host and network preparation
before deploying OSA for load testing.

Create a management network, storage network, and tunnel network 
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

For overview of OSA Network design, see
`*openstack-ansible/install-guide/targethosts-network.html* <http://docs.openstack.org/developer/openstack-ansible/install-guide/targethosts-network.html>`__

definitions:

-  As above (step 1), no gateway, no router.

   -  See the `*private address filter
          reference* <https://www.arin.net/knowledge/address_filters.html>`__
          (above);

   -  Assign an easily identifiable unique address space for each of the
          three networks, e.g:

      -  Management Network: 192.168.1.0/24

      -  Storage Network: 192.168.2.0/24

      -  Tunnel Network: 192.168.3.0/24

Create infrastructure controller hosts:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  Use an SSD equipped availability\_zone

-  Use m2.xlarge flavor

-  Boot from Image, using Ubuntu 14.04 server

-  Use the shared deploy keypair

-  Specify hosts to be in the default Security Group, (or another
       security group that allows all TCP ingress from any other member
       of the Security Group).

-  Add both your Gateway network and your management network to the
       instance.

Ensure your deploy host and all controller hosts are on the management
network

-  As above.

Create other hosts, as appropriate for your deployment, adding the
appropriate network(s).

Follow instructions from the install guide for preparing the deployment
host and the target hosts (see:

`*http://docs.openstack.org/developer/openstack-ansible/install-guide/deploymenthost.html* <http://docs.openstack.org/developer/openstack-ansible/install-guide/deploymenthost.html>`__
and

`*http://docs.openstack.org/developer/openstack-ansible/install-guide/targethosts-prepare.html* <http://docs.openstack.org/developer/openstack-ansible/install-guide/targethosts-prepare.html>`__
etc…)

E.g. Install and configure basic packages

$ sudo apt-get update && sudo apt-get install ntp ntpdate bridge-utils
debootstrap ifenslave ifenslave-2.6 lsof lvm2 ntp ntpdate openssh-server
sudo tcpdump vlan

Etc…

When you get to
`*http://docs.openstack.org/developer/openstack-ansible/install-guide/targethosts-network.html* <http://docs.openstack.org/developer/openstack-ansible/install-guide/targethosts-network.html>`__

Panic, because you don’t know what to do!

(^^^ THIS is a note that says we need to develop the solution from here
forward, probably based on
`*http://docs.openstack.org/developer/openstack-ansible/install-guide/targethosts-networkexample.html* <http://docs.openstack.org/developer/openstack-ansible/install-guide/targethosts-networkexample.html>`__)
