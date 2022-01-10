..
   This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

..
   This template should be in ReSTructured text. The filename in the git
 repository should match the launchpad URL, for example a URL of
 https://blueprints.launchpad.net/magnum/+spec/awesome-thing should be named
 awesome-thing.rst .  Please do not delete any of the sections in this
 template.  If you have nothing to say for a whole section, just write: None
 For help with syntax, see http://sphinx-doc.org/rest.html
 To test out your formatting, see http://www.tele3.cz/jbar/rest/rest.html

==================
Cluster API driver
==================

https://storyboard.openstack.org/#!/story/2009780

Problem description
===================

Generally Magnum works best when you treat your Kubernetes clusters
as a single-purpose and disposable.  If there is a problem, you
simply replace your cluster.  This can make running
production web apps quite hard (although support for sharing Octavia
load balancers between multiple clusters could help).  However, some
use cases require longer-lived clusters that are not disposable.

Upgrades in Magnum
------------------

Magnum supports rolling upgrades in some drivers [#]_, although
experience suggests this process is not always reliable.  Uncertainty
undermines confidence in the ability to upgrade production clusters.

Driving Innovation
------------------

The pace of development of the Magnum project has suffered, potentially
due to the ongoing effort for sustaining the model used in current
drivers.  A driver in which most of the core was outsourced to an
active external project should reduce maintenance burden.

Proposed change
===============

All current Magnum COE drivers are built on a common Heat-based
driver class.

This spec proposes a new driver class that implements Kubernetes
cluster orchestration using Cluster API [#]_.  The intention of this
proposal is to create a simplified Kubernetes driver that takes
advantage of a broader cross-infrastructure community using Cluster API
as a common standard.

A feature comparison shows that Cluster API has much in common with Magnum,
but introduces some innovations that would be beneficial to Magnum.

+--------------------------+----------------------+---------------------------+
| Feature                  | Magnum               | Cluster-API               |
+==========================+======================+===========================+
| Cloud Provider OpenStack | Installed by default | Installed via Helm charts |
+--------------------------+----------------------+---------------------------+
| Host OS support          | FCOS 33 supported,   | Typically Ubuntu 20.04    |
|                          | 34 and beyond WIP    | LTS, various choices      | 
|                          | (due to cgroups v2)  | supported by the image    |
|                          |                      | builder [#]_.             | 
|                          | *NOTE: no security   |                           |
|                          | updates for FCOS 33  |                           |
|                          | any more?*           |                           |
+--------------------------+----------------------+---------------------------+
| Supported CNIs           | Flannel, Calico.     | Further options available,|
|                          |                      | eg Cilium.                |
+--------------------------+----------------------+---------------------------+
| Cinder CSI               | Default from Victoria| Installed via Helm charts.|
+--------------------------+----------------------+---------------------------+
| Prometheus monitoring    | Installed by default.| Installed via Helm charts.|
+--------------------------+----------------------+---------------------------+
| Ingress controllers      | Octavia, Traefik,    | Nginx installed via Helm. |
|                          | Nginx.               |                           |
+--------------------------+----------------------+---------------------------+
| Horizon dashboard        | Supported.           | Not supported.            |
+--------------------------+----------------------+---------------------------+
| Kubernetes CRD API       | None.                | Helm, flux, argo, etc.    |
+--------------------------+----------------------+---------------------------+
| In-place upgrades        | Partial - depends on | Various supported         |
|                          | driver.              | strategies, build in      |
|                          |                      | infrastrucutre agnostic   |
|                          |                      | code. Defaults to rolling |
|                          |                      | upgrade (like Magnum).    |
+--------------------------+----------------------+---------------------------+
| Self-healing             | Partial / uncertain. | Supported with            |
|                          |                      | infrastrcuture agnostic   |
|                          |                      | code via reconciliation   |
|                          |                      | loop.                     |
+--------------------------+----------------------+---------------------------+
| Auto-scaling             | Supported for        | Supported with            |
|                          | default node group.  | infrastructure agnositc   |
|                          |                      | code.                     |
+--------------------------+----------------------+---------------------------+
| Multiple node groups     | Supported.           | Supported, with no default|
|                          |                      | group.                    |
+--------------------------+----------------------+---------------------------+
| Additional networks      | Supported (review    | Supported                 |
|                          | pending)             |                           |
+--------------------------+----------------------+---------------------------+
| New Kubernetes versions  | Test burden on Magnum| Test burden split between |
|                          | entirely.            | Cluster API for           |
|                          |                      | Kubernetes, Magnum for    |
|                          |                      | infrastructure provision. |
+--------------------------+----------------------+---------------------------+

The intention for the new driver is to make it compatible with the
Magnum REST API.

Alternatives
------------

A roadmap securing ongoing Magnum development is important for the
project.  Alternatives would include doing something similar outside
of Magnum.

Implementation
==============

Cluster API is not a drop-in replacement for a Magnum COE driver, and some
further work will be needed to ensure it has the resources it requires in
order to function.  Cluster API provisions and operates Kubernetes clusters
from a dedicated management cluster, which requires access to OpenStack
APIs and provisioned infrastructure.

Deployment of the management cluster is a significant aspect of
driver implementation.  This could be performed in two ways:

* Boostrap a Kubernetes cluster in a service tenancy on infrastructure
  hosted by OpenStack.
* Deploy Cluster API's management services to an existing "provider
  cluster".

Each approach could be valid in different configurations.  The
bootstrap cluster could feasibly be implemented outside of Magnum's
scope by OpenStack deployment frameworks.

Assignee(s)
-----------

Primary assignee:

  * *TBD* Lead at StackHPC

With support from:

  * *TBD* Lead at CERN
  * *TBD* Lead at VexxHost

Milestones
----------

*TBD*

Work Items
----------

*TBD*

A stubbed-out Cluster API driver framework was proposed here [#]_.

Dependencies
============

- The proposed Cluster API driver would integrate particularly well
  with the Helm-based approach to Kubernetes cluster deployment and
  configuration.

- Introducing a new driver with significant differences in
  implementation will expose any leaks of abstraction in the class
  hierarchy.  Additional CI tests will be required to increase
  functional coverage and catch any regressions.

Security Impact
===============

A new driver built upon Cluster API has the potential to improve
security for Magnum, due to wider scrutiny of the open source
implementation, a smaller code base for the Magnum team to maintain
and a larger community focussing on the security of Cluster API's
managed clusters.

References
==========

.. [#] https://docs.openstack.org/magnum/latest/user/#rolling-upgrade
.. [#] https://cluster-api.sigs.k8s.io
.. [#] https://github.com/kubernetes-sigs/image-builder/tree/master/images/capi/packer/qemu 
.. [#] https://review.opendev.org/c/openstack/magnum/+/815521
