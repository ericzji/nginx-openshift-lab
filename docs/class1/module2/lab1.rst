
Deploy NGINX Plus Ingress Controller with NAP to provide security for the Arcadia application
-----------------------------------------------------------


Deployment Overview
#####################
In Module 1, we have already deployed the NGINX Ingress Controller, so we will focus on configuring NGINX App Protect in this module. For the complete installation of the NGINX Plus Ingress Controller, refer to the official documentation `Installation with Manifests`_

At a high level, we will:

  #. Configure role-based access control (RBAC)
  #. Create the common Kubernetes resources
  #. Install the Ingress Controller with NGINX App Protect WAF
  #. Configure the NGINX App Protect WAF module
  #. Attach NAP Policy to the NGINX Ingress Controller’s Virtual Server

Clone the Ingress Controller repository and navigate to the deployments folder by running the following commands:
  
   .. code-block::

      git clone https://github.com/nginxinc/kubernetes-ingress.git --branch v3.0.1
      cd kubernetes-ingress/deployments


Configure role-based access control (RBAC)
##########################################
In Module 1, we created a namespace and a service account for the Ingress Controller, as well as a cluster role and a cluster role binding for the service account. To use the App Protect WAF module, we need to create the App Protect role and role binding by running the following command:

   .. code-block::

      oc apply -f rbac/ap-rbac.yaml

Create the common Kubernetes resources
#######################################
In Module 1, we created:

- a secret with a TLS certificate and a key for the default server in NGINX:
- a config map for customizing NGINX configuration
- an IngressClass resource

No additional common resource is needed for the App Protect WAF module.
  
Create Custom Resources
########################

In Module 1, we created custom resource definitions for VirtualServer and VirtualServerRoute, TransportServer, and Policy resources. To use the App Protect WAF module, we need to create custom resource definitions for APPolicy, APLogConf, and APUserSig. Run the following commands to create these resources:
    
    .. code-block:: bash
    
       oc apply -f common/crds/appprotect.f5.com_aplogconfs.yaml
       oc apply -f common/crds/appprotect.f5.com_appolicies.yaml
       oc apply -f common/crds/appprotect.f5.com_apusersigs.yaml

Update the Ingress Controller with NGINX App Protect WAF
##########################################################

**Steps**

    #.  Enable the App Protect module in the Ingress Controller.

        Enable the App Protect module in the Ingress Controller by clicking on "Operators" and then "Installed Operators" in the OpenShift Console's left navigation column. On the page that opens, click the Nginx Ingress Controller link in the "Provided APIs" column, select "my-nginx-ingress-controller," and then click YAML to change the apppotect 'enable' field to true under spec: controller:
        
            .. code-block:: yaml

                apiVersion: charts.nginx.org/v1alpha1
                kind: NginxIngress

                spec:
                controller:
                    appprotect:
                    enable: True


        Example:
        .. image:: ./pictures/ingress-controller-nap.png


        Click Save, and Reload

        .. note::  Make sure that you have pulled the Ingress Controller image with App Protect. In this lab, we have already loaded the NGINX Plus image with App Protect to a local registry.


    #.  After reloading, wait for the KIC pod to become available by running the command:

        .. code-block:: BASH

           oc get pod -n nginx-ingress --watch

    #.  When it's ready, press ``ctrl-c`` to stop the watch.

        .. image:: ./pictures/ingress-ready.png

Configure the NGINX App Protect WAF module
###########################################
Now, it is time to configure the Ingress Controller with CRD ressources (WAF policy, Log profile, Ingress routing ...)

**Steps**

Execute the following commands to deploy the different resources. In the terminal window, copy the below text and paste+enter:

    
    .. code-block:: bash
          
       cd /home/lab-user/kubernetes-ingress/examples/custom-resources/app-protect-waf
          
       oc apply -f syslog.yaml
       oc apply -f ap-apple-uds.yaml
       oc apply -f ap-dataguard-alarm-policy.yaml
       oc apply -f ap-logconf.yaml
       oc apply -f waf.yaml


  Out of above commands, we focus on the following files: 

  1. The ``ap-dataguard-alarm-policy.yaml`` file creates the WAF policy that specifies the rules for protecting the application from layer 7 attacks. It is recommended to customize this policy according to the specific application requirements.
 
  In this lab, we will proceed by disregarding the "apple_sigs" signature set. Kindly remove the subsequent lines from ``ap-dataguard-alarm-policy.yaml``:

        .. code-block:: yaml

          signature-requirements:
          - tag: Fruits
          signature-sets:
          - name: apple_sigs
            block: true
            signatureSet:
              filter:
                tagValue: Fruits
                tagFilter: eq
     
  If preferred, you can also accomplish this using the 'sed' command as follows:

     .. code-block:: bash
     
        sed -i '/signature-requirements:/,/eq/d' ap-dataguard-alarm-policy.yaml

  Once modified, your ``ap-dataguard-alarm-policy.yaml`` should resemble this:

     .. literalinclude :: ./templates/ap-dataguard-alarm-policy.yaml
      :language: yaml
     
  
  In the terminal window, copy the below text and paste+enter, to reapply the ``ap-dataguard-alarm-policy.yaml``:
    
    .. code-block:: bash

        oc apply -f ap-dataguard-alarm-policy.yaml


  2. The ``ap-logconf.yaml`` file creates the Log Profile that specifies the format of the logs to be generated when the policy detects an attack.
 
      .. literalinclude :: ./templates/ap-logconf.yaml
       :language: yaml


  3. The ``waf.yaml`` file creates the WAF configuration that links the WAF policy and Log Profile to the NGINX Ingress Controller.
    .. literalinclude :: ./templates/waf.yaml
       :language: yaml

Attach NAP Policy to the NGINX Ingress Controller’s Virtual Server
######################################################################
It is important that the application always has a WAF protecting it.

To enable NAP for an application, a Virtual Server in NGINX Ingress Controller requires both a Policy and an APPolicy custom resource to be attached to it. You simply need to add the reference to the Virtual Server.

**Steps**

#. Examine the contents of the **VirtualServer** resource ``oc get virtualserver arcadia``.
  
      .. code-block:: bash
                    
        oc get virtualserver arcadia

#. update VirtualServer ``oc edit virtualserver arcadia``

    .. code-block:: bash
                  
       oc edit virtualserver arcadia

#. Add the following content to the lines immediately following `host: $nginx_ingress`, at the same indentation level:

          .. code-block:: yaml
            
             policies:
             - name: waf-policy

    Once modified, your ``virtualserver`` should resemble this:

          .. code-block:: yaml

            apiVersion: k8s.nginx.org/v1
            kind: VirtualServer
            metadata:
              name: arcadia
            spec:
              host: $nginx_ingress
              policies:
              - name: waf-policy
              upstreams:
              - name: arcadia-main
                service: arcadia-main
                port: 80
              - name: arcadia-app2
                service: arcadia-app2
                port: 80
              - name: arcadia-app3
                service: arcadia-app3
                port: 80

  
    The waf-policy should match the name of the WAF policy created in step 2.6.


#. Save the file and exit the editor.

.. _Installation with Manifests: https://docs.nginx.com/nginx-ingress-controller/installation/installation-with-manifests/