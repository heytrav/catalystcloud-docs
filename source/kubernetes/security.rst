########
Security
########

*****************
Admission Control
*****************

After Kubernetes API server received API request from client side, and just
before persisting the resource in etcd, there are a list of admission controller
which will intercepts the API request to validate and/or mutate the request.
Please refer `Kubernetes official document`_ for the full list of admission controller.

.. _`Kubernetes official document`: https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/


Catalyst Cloud Kubernetes Service supports setting admission controllers
when creating a new cluster by specifying labels. The following 
admission controllers are enabled by default:

* CertificateApproval
* CertificateSigning
* CertificateSubjectRestriction
* DefaultIngressClass
* DefaultStorageClass
* DefaultTolerationSeconds
* LimitRanger
* MutatingAdmissionWebhook
* NamespaceLifecycle
* PersistentVolumeClaimResize
* PodSecurity
* Priority
* ResourceQuota
* RuntimeClass
* ServiceAccount
* StorageObjectInUseProtection
* TaintNodesByCondition
* ValidatingAdmissionPolicy
* ValidatingAdmissionWebhook

.. Note:: 
   The exact set of admission controllers may vary depending on which version
   of Kubernetes you are using.

How to turn on an admission controller
======================================

There are some other useful admission controller can be enabled to enhance
the security of Kubernetes cluster. To turn on an admission controllers, user
can do it by either command line or dashboard with label ``admission_control_list``.

Command Line
~~~~~~~~~~~~

With command line, when creating a new Kubernetes cluster, please use label
``admission_control_list`` and make sure --merge-labels used as well.

.. code-block:: bash

  openstack coe cluster create k8s-1 --merge-labels --labels admission_control_list=PodSecurity,ValidatingAdmissionPolicy,ValidatingAdmissionWebhook --cluster-template kubernetes-v1.28.2-prod-20230630

Dashboard
~~~~~~~~~

When using dashboard to create Kubernetes cluster, on the last Advanced tag,
you can set additional labels as below:

.. image:: _containers_assets/k8s_admission_controller.png
