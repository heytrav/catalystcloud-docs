***********
Sandboxed Workloads
***********

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
