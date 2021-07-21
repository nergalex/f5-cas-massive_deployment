Massive deployment test using NGINX Controller App Sec
=======================================================================
.. contents:: Table of Contents
    :local:

Pre-requisites
==============
Ansible Tower
##############

Role
***************************
Clone roles from `NGINX Controller collection <https://github.com/nginxinc/ansible-collection-nginx_controller>`_ in `/etc/ansible/roles/`

- nginxinc.nginx_controller_generate_token
- nginxinc.nginx_controller_certificate
- nginxinc.nginx_controller_gateway
- nginxinc.nginx_controller_application
- nginxinc.nginx_controller_component
- nginxinc.nginx_controller_environment

Rename generated directory of cloned roles as listed above

Ansible role structure
######################
- Deployment is based on ``workflow template``. Example: ``workflow template`` = ``wf-create_create_edge_security_inbound``
- ``workflow template`` includes multiple ``job template``. Example: ``job template`` = ``poc-azure_create_hub_edge_security_inbound``
- ``job template`` have an associated ``playbook``. Example: ``playbook`` = ``playbooks/poc-azure.yaml``
- ``playbook`` launch a ``play`` in a ``role``. Example: ``role`` = ``poc-azure``

.. code:: yaml

    - hosts: localhost
      gather_facts: no
      roles:
        - role: poc-azure

- ``play`` is an ``extra variable`` named ``activity`` and set in each ``job template``. Example: ``create_hub_edge_security_inbound``
- The specified ``play`` (or ``activity``) is launched by the ``main.yaml`` task located in the role ``tasks/main.yaml``

.. code:: yaml

    - name: Run specified activity
      include_tasks: "{{ activity }}.yaml"
      when: activity is defined

- The specified ``play`` contains ``tasks`` to execute. Example: play=``create_hub_edge_security_inbound.yaml``

Infrastructure
####################################
- NGINX Controller installed
- NGINX Controller Application Security licensed
- NGINX Controller Application Security enabled using ``./helper.sh setfeature AppSec true``
- NGINX App Protect instances installed
- NGINX Controller agent installed into NGINX App Protect instances installed using a same ``Location`` for both

1.1) Create Certificate
==================================================
===============================================================   =============================================       =============================================   ===============================================   =============================================   =============================================   =============================================
Job template                                                      objective                                           playbook                                        activity                                          inventory                                       limit                                           credential
===============================================================   =============================================       =============================================   ===============================================   =============================================   =============================================   =============================================
``poc-nginx_controller-create_massive_self_signed_certificate``   Create App                                          ``playbooks/poc-nginx_controller.yaml``         ``create_massive_self_signed_certificate``        localhost
===============================================================   =============================================       =============================================   ===============================================   =============================================   =============================================   =============================================

==============================================  =============================================   ================================================================================================================================================================================================================
Extra variable                                  Description                                     Example
==============================================  =============================================   ================================================================================================================================================================================================================
``extra_nb_app``                                survey: number of Apps to deploy                ``250``
``extra_project``                               project name                                    ``cloudbuilder``
``extra_app``                                   App specifications                              dict
``extra_app.domain``                            Domain for all Apps                             ``f5app.dev``
``extra_app.name``                              Application name                                ``demo``
==============================================  =============================================   ================================================================================================================================================================================================================

.. code:: yaml

    extra_app:
      domain: f5app.dev
      name: demo
    extra_nb_app: 250
    extra_project: cloudbuilder


1.2) Create Applications
==================================================
=============================================================   =============================================       =============================================   ===============================================   =============================================   =============================================   =============================================
Job template                                                    objective                                           playbook                                        activity                                          inventory                                       limit                                           credential
=============================================================   =============================================       =============================================   ===============================================   =============================================   =============================================   =============================================
``poc-nginx_controller-create_massive``                         Create App                                          ``playbooks/poc-nginx_controller.yaml``         ``create_massive``                                localhost
=============================================================   =============================================       =============================================   ===============================================   =============================================   =============================================   =============================================

==============================================  =============================================   ================================================================================================================================================================================================================
Extra variable                                  Description                                     Example
==============================================  =============================================   ================================================================================================================================================================================================================
``extra_nb_app``                                survey: number of Apps to deploy                ``250``
``extra_nginx_controller.ip``                   NGINX Controller IP or FQDN                     ``10.0.0.4``
``extra_nginx_controller_username``             survey: NGINX Controller admin credential       ``XXXXXXXX@acme.com``
``extra_nginx_controller_password``             survey: NGINX Controller admin credential       ``XXXXXXXX``
``extra_app``                                   App specifications                              dict
``extra_app.gateways.location``                 Location of instances                           ``nginxwaf``
``extra_app.domain``                            Domain for all Apps                             ``f5app.dev``
``extra_app.environment``                       Resource Group used for RBAC                    ``massive``
``extra_app.components``                        PATH of each App                                dict
``extra_app.components.name``                   Component's logical name                        ``main`` for PATH ``/``
``extra_app.components.uri``                    Component's URI                                 ``/``
``extra_app.components.waf_policy``             attached WAF policy to component                dict
``extra_app.components.waf_policy.name``        WAF policy's name                               ``web_factory_arcadia``
``extra_app.components.waf_policy.waf_policy``  WAF policy's repository URL                     ``https://raw.githubusercontent.com/nergalex/f5-nap-policies/master/policy/arcadia_web_factory.json``
``extra_app.name``                              Application name                                ``demo``
==============================================  =============================================   ================================================================================================================================================================================================================

.. code:: yaml

    extra_app:
      components:
        - name: main
          uri: /
          waf_policy:
            name: web_factory_arcadia
            url: >-
              https://raw.githubusercontent.com/nergalex/f5-nap-policies/master/policy/arcadia_web_factory.json
          workloads:
            - 10.12.1.5
        - name: login
          uri: /trading/login.php
          waf_policy:
            name: bot_prevention
            url: >-
              https://raw.githubusercontent.com/nergalex/f5-nap-policies/master/policy/arcadia_bot_prevention.json
          workloads:
            - 10.12.1.5
        - name: acme
          uri: /.well-known/acme-challenge
          waf_policy:
            name: generic
            url: generic
          workloads:
            - 127.0.0.1
        - name: security.txt
          uri: /.well-known/security.txt
          waf_policy:
            name: generic
            url: generic
          workloads:
            - 127.0.0.1
      domain: f5app.dev
      environment: massive
      gateways:
        location: nginxwaf
      name: demo
    extra_nb_app: 250
    extra_nginx_controller:
      ip: 10.0.0.4
    extra_nginx_controller_password: $encrypted$
    extra_nginx_controller_username: nergalex@acme.com
    extra_project: cloudbuilder

2) Delete Applications
==================================================
=============================================================   =============================================       =============================================   ===============================================   =============================================   =============================================   =============================================
Job template                                                    objective                                           playbook                                        activity                                          inventory                                       limit                                           credential
=============================================================   =============================================       =============================================   ===============================================   =============================================   =============================================   =============================================
``poc-nginx_controller-delete_massive``                         Delete App                                          ``playbooks/poc-nginx_controller.yaml``         ``delete_massive``                                localhost
=============================================================   =============================================       =============================================   ===============================================   =============================================   =============================================   =============================================

==============================================  =============================================   ================================================================================================================================================================================================================
Extra variable                                  Description                                     Example
==============================================  =============================================   ================================================================================================================================================================================================================
``extra_nb_app``                                number of Apps to deploy                        ``10``
``extra_nginx_controller_ip``                   NGINX Controller IP or FQDN                     ``10.0.0.10``
``extra_nginx_controller_username``             NGINX Controller admin credential               ``XXXXXXXX@acme.com``
``extra_nginx_controller_password``             NGINX Controller admin credential               ``XXXXXXXX``
``extra_app``                                   App specifications                              dict, see below
==============================================  =============================================   ================================================================================================================================================================================================================

.. code:: yaml

    activity: delete_massive
    extra_nginx_controller:
      ip: 10.0.0.4
    extra_project: "cloudbuilder"
    extra_app:
      gateways:
        location: "nginxwaf"
      name: "demo"
      domain: "f5app.dev"
      environment: massive
      components:
        - name: main
        - name: login
        - name: acme
    extra_nb_app: 250
    extra_nginx_controller_password: $encrypted$
    extra_nginx_controller_username: nergalex@acme.com




