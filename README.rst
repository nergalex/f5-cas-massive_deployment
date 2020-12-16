Test deployment perf on CAS
=======================================================================
.. contents:: Table of Contents

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

Rename generated directory of these roles as listed above

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

CAS
##############
- Instances deployed and attached to a location

0) Create Applications
==================================================
Workflow
###############################
=============================================================   =============================================       =============================================   ===============================================   =============================================   =============================================   =============================================
Job template                                                    objective                                           playbook                                        activity                                          inventory                                       limit                                           credential
=============================================================   =============================================       =============================================   ===============================================   =============================================   =============================================   =============================================
``poc-nginx_controller-create_massive``                         Create App                                          ``playbooks/poc-nginx_controller.yaml``         ``create_massive_gw_app_component_vmss_north``    localhost
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

    extra_app:
      components:
        - name: Arcadia_main
          type: adc
          uri: /
          workloads:
            - 'https://arcadia.acme.dev'
        - name: Arcadia_app2
          type: adc
          uri: /api/
          workloads:
            - 'https://arcadia.acme.dev'
        - name: Arcadia_app3
          type: adc
          uri: /app3/
          workloads:
            - 'https://arcadia.acme.dev'
        - name: Arcadia_db
          type: adc
          uri: /files/
          workloads:
            - 'https://arcadia.acme.dev'
      domain: acme.dev
      environment: prod
      gateways:
        location: nginxwaf
      monitor_uri: /
      name: arcadia-test
      preserveHostHeader: ENABLED
      tls:
        crt: "-----BEGIN CERTIFICATE-----\r\nXXXXXXXXXXXXXXXXXXX\r\n-----END CERTIFICATE-----"
        key: "-----BEGIN RSA PRIVATE KEY-----\r\nXXXXXXXXXXXXXXX\r\n-----END RSA PRIVATE KEY-----"

0) Delete Applications
==================================================
Workflow
###############################
=============================================================   =============================================       =============================================   ===============================================   =============================================   =============================================   =============================================
Job template                                                    objective                                           playbook                                        activity                                          inventory                                       limit                                           credential
=============================================================   =============================================       =============================================   ===============================================   =============================================   =============================================   =============================================
``poc-nginx_controller-delete_massive``                         Delete App                                          ``playbooks/poc-nginx_controller.yaml``         ``delete_massive_gw_app_component_vmss_north``    localhost
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

    extra_app:
      components:
        - name: Arcadia_main
        - name: Arcadia_app2
        - name: Arcadia_app3
        - name: Arcadia_db
      domain: acme.dev
      environment: prod
      name: arcadia-test
