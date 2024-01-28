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

When creating a new Kubernetes cluster on the command line you can use the label
``admission_control_list`` to supplement or override the default labels.  Don't forget to use ``--merge-labels`` when 
adding or overriding specific labels.

.. code-block:: bash

  openstack coe cluster create k8s-1 \
  --merge-labels --labels admission_control_list=PodSecurity,ValidatingAdmissionPolicy,ValidatingAdmissionWebhook \
  --cluster-template kubernetes-v1.28.2-prod-20230630

Dashboard
~~~~~~~~~

When using dashboard to create Kubernetes cluster, on the last Advanced tag,
you can set additional labels as below:

.. image:: _containers_assets/k8s_admission_controller.png

*********
Sandboxed Containers
*********

Containers typically share kernel resources of the host VM with other 
containers. While this is generally considered to be one of the key benefits
of containers as it makes them more lightweight, it also makes them less secure
than traditional VMs.

For additional security in a Kubernetes cluster it can be useful to run certain
containers in a restricted runtime environment known as a sandbox. One option for this
is to use [gVisor](https://gvisor.dev/docs/) which provides a layer
of separation between a running container and the host kernel.

All of our cluster nodes come with the gVisor executable, ``runsc``, installed and configured for ``containerd`` to use.
The only thing you need to in order to begin running sandboxed containers is create a ``RuntimeClass`` object
in your cluster as follows:

.. code-block:: bash

  cat <<EOF | kubectl apply -f -
  ---
  apiVersion: node.k8s.io/v1
  kind: RuntimeClass
  metadata:
    # The name the RuntimeClass will be referenced by.
    # RuntimeClass is a non-namespaced resource.
    name: gvisor
  handler: gvisor
  EOF

Now, to run a pod in the sandboxed environment you just need to specify the name of the RuntimeClass
using ``runtimeClassName`` in the Pod spec:

.. code-block:: yaml

  ---
  apiVersion: v1
  kind: Pod
  metadata:
    name: test-sandboxed-pod
  spec:
    runtimeClassName: gvisor
    containers:
      - name: sandboxed-container
        image: nginx

Once the pod is up and running, you can verify by using ``kubectl exec`` to start a shell on the
pod and run ``dmesg``. If the container sandbox is running correctly you should see output similar
to the following:

.. code-block:: bash
   kubectl exec test-sandboxed-pod -- dmesg

  [    0.000000] Starting gVisor...
  [    0.511752] Digging up root...
  [    0.910192] Recruiting cron-ies...
  [    1.075793] Rewriting operating system in Javascript...
  [    1.351495] Mounting deweydecimalfs...
  [    1.648946] Searching for socket adapter...
  [    2.115789] Checking naughty and nice process list...
  [    2.351749] Granting licence to kill(2)...
  [    2.627640] Creating bureaucratic processes...
  [    2.954404] Constructing home...
  [    3.396065] Segmenting fault lines...
  [    3.812981] Setting up VFS...
  [    4.164302] Setting up FUSE...
  [    4.224418] Ready!

You are running a sandboxed container.

Resources:
`Container Sandboxing | gVisor`_

.. _`Container Sandboxing | gVisor`: https://medium.com/geekculture/container-sandboxing-gvisor-b191dafdc8a2
