saphana-deploy
==============

# THIS ROLE IS DEPRECATED AND WILL BE DELETED IN 12.2022<BR>use _community.sap_install_ collection instead

This role deploys SAP HANA 1.0 SPS 12 and above (HANA2) on a properly configured  RHEL 6.7 or 7.x system.

Requirements
------------

This role requires the system to be prepared with saphana-preconfigure role. Please make sure that the provided storage is sufficient with SAP requirements.


Role Variables
--------------

### Configuration for Instance deployment

This role uses the same variables as the saphana-preconfigure role.  It expects the installation  directory to be available at `hana_installdir`.

The following variables need to be set in the host_vars file. See [Best Practises](http://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html?highlight=host_var#group-and-host-variables) for more details on how to use host_vars and group_vars.

The dictionary instances describe the parameters of the instances. You can define more than one instance for multiple hana instances on one system.
The variables you define here are used for the unattended installation with hdblcm. The variables are the same as in the config file and prefixed with `hana_`.
You can also set `hdblcm_params` to pass additional parameters to the hdblcm command, such as:

      hdblcm_params: "--ignore=<check1>[,<check2>]..."

ignoring failing prerequisite checks can sometimes be important in testinstallations with low memory or officially unsupported CPUs. The following checks can be ignored:  check_component_dependencies, check_diskspace,
                                                 check_min_mem, check_platform, check_signature_file, verify_signature)


hostname is used for the instance, saphostagent uses for communication

set deployment_instance to true if you want to execute the actual installation. In scale-out scenarios only one host should have set this to true

### Example host_vars file for server "node1"

     ---
     hostname: "{{ ansible_hostname }}"
     deployment_instance: true

     instances:
        l01:
           id_user_sidadm: "30210"
           pw_user_sidadm: "Adm12356"
           hana_pw_system_user_clear: "System123"
           hana_components: "client,server"
           hana_system_type: "Master"
           id_group_shm: "30220"
           hana_instance_hostname: node1
           hana_addhosts:
           hana_sid: L01
           hana_instance_number: 10
           hana_system_usage: custom
           hdblcm_params: "--ignore=check_platform"


Example Playbook
----------------

Here is an example playbook that installs a complete server

    ---
    - hosts: hana
      remote_user: root

      vars:
              # subscribe-rhn role variables
              reg_activation_key: myregistration
              reg_organization_id: 123456

              repositories:
                      - rhel-7-server-rpms
                      - rhel-sap-hana-for-rhel-7-server-rpms

              # If you want to use 4 years update services, use:
              #       - rhel-7-server-e4s-rpms
              #       - rhel-sap-hana-for-rhel-7-server-e4s-rpms

              # disk-init role variables
              disks:
                      /dev/vdc: vg00
                      /dev/vdb: vg00
              logvols:
                      hana_shared:
                              size: 24G
                              vol: vg00
                              mountpoint: /hana/shared
                      hana_data:
                              size: 24G
                              vol: vg00
                              mountpoint: /hana/data
                      hana_logs:
                              size: 12G
                              vol: vg00
                              mountpoint: /hana/logs
                      usr_sap:
                              size: 49G
                              vol: vg00
                              mountpoint: /usr/sap


              # rhel-system-roles.timesync variables
              ntp_servers:
                      - hostname: 0.rhel.pool.ntp.org
                        iburst: yes
                      - hostname: 1.rhel.pool.ntp.org
                        iburst: yes
                      - hostname: 2.rhel.pool.ntp.org
                        iburst: yes
                      - hostname: 3.rhel.pool.ntp.org
                        iburst: yes


              # SAP Precoonfigure role

              # SAP-Media Check
              install_nfs: "mynfsserver:/installi-export"
              installroot: /install
              hana_installdir: "{{ installroot + '/HANA2SPS02' }}"

              hana_pw_hostagent_ssl: "MyS3cret!"
              id_user_sapadm: "30200"
              id_group_shm: "30220"
              id_group_sapsys: "30200"
              pw_user_sapadm_clear: "MyS3cret!"

      roles:
              - { role: mk-ansible-roles.subscribe-rhn }
              - { role: mk-ansible-roles.disk-init }
              - { role: linux-system-roles.timesync }
              - { role: mk-ansible-roles.saphana-preconfigure }
              - { role: mk-ansible-roles.saphana-deploy }
              - { role: mk-ansible-roles.saphana-hsr }

License
-------

Apache License
Version 2.0, January 2004

Author Information
------------------

Markus Koch

Please leave comments in the github repo issue list
