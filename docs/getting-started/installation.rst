Installation Guide
##################

Robusta is installed with Helm. You can handwrite the values.yaml, but it is easier to autogenerate it.

The standard installation uses Helm and the robusta-cli, but :ref:`other alternative methods are described below.
<Alternative Installation Methods>`

Standard Installation
------------------------------

1. Download the Helm chart and install Robusta-CLI:

.. code-block:: bash

   helm repo add robusta https://robusta-charts.storage.googleapis.com && helm repo update
   pip install -U robusta-cli --no-cache
   

.. warning:: If you are using a system such as macOS that includes both Python 2 and Python 3, run pip3 instead of pip.

2. Generate a Robusta configuration. This will setup Slack and other integrations.

.. code-block:: bash

    robusta gen-config

3. Save ``generated_values.yaml``, somewhere safe. This is your Helm ``values.yaml`` file.

4. Install Robusta using Helm:

.. code-block:: bash

    helm install robusta robusta/robusta -f ./generated_values.yaml

5. Verify that Robusta installed two deployments in the current namespace:

.. code-block:: bash

    kubectl get pods

Seeing Robusta in action
------------------------------

By default, Robusta sends Slack notifications when Kubernetes pods crash.

1. Create a crashing pod:

.. code-block:: python

   kubectl apply -f https://gist.githubusercontent.com/robusta-lab/283609047306dc1f05cf59806ade30b6/raw


2. Verify that the pod is actually crashing:

.. code-block:: bash

   $ kubectl get pods -A
   NAME                            READY   STATUS             RESTARTS   AGE
   crashpod-64d8fbfd-s2dvn         0/1     CrashLoopBackOff   1          7s

3. Once the pod has reached two restarts, check your Slack channel for a message about the crashing pod.

.. admonition:: Example Slack Message

    .. image:: /images/crash-report.png


4. Clean up the crashing pod:

.. code-block:: python

   kubectl delete deployment crashpod

Forwarding Prometheus Alerts to Robusta
----------------------------------------

Robusta can improve your existing Prometheus alerts by adding extra context and "fix-it" buttons.

For this to work, :ref:`you must configure an AlertManager webhook. <Sending Alerts to Robusta>` This is **not**
necessary if you installed Robusta's bundled Prometheus Stack. In that case, the webhook is preconfigured.

Next Steps
---------------------------------

1. Explore the `Robusta UI <https://home.robusta.dev/ui/>`_ (use the URL you received during installation)
2. :ref:`Learn how to write your own Robusta automations. <Example Playbook>`
3. :ref:`Learn about Robusta's features for manual troubleshooting <Manual Triggers>`

Additional Installation Methods
---------------------------------

.. dropdown:: Installing with GitOps
    :color: light

    Follow the instructions above to generate ``generated_values.yaml``. Commit it to git and use ArgoCD or
    your favorite tool to install.

.. dropdown:: Installing without the Robusta CLI
    :color: light

    Using the cli is totally optional. If you prefer, you can skip the CLI and fetch the default ``values.yaml``:

    .. code-block:: yaml

        helm repo add robusta https://robusta-charts.storage.googleapis.com && helm repo update
        helm show values robusta/robusta


    Most values are documented in the :ref:`Configuration Guide`

    Do not use the ``values.yaml`` file in the GitHub repo. It has some empty placeholders which are replaced during
    our release process.

.. dropdown:: Installing in a different namespace
    :color: light

    Create a namespace ``robusta`` and install robusta in the new namespace using:

    .. code-block:: bash

        helm install robusta robusta/robusta -f ./generated_values.yaml -n robusta --create-namespace

    Verify that Robusta installed two deployments in the ``robusta`` namespace:

    .. code-block:: bash

        kubectl get pods -n robusta

.. dropdown:: Installing a second cluster
    :color: light

    When installing a second cluster on the same account, there is no need to run ``robusta gen-config`` again.

    Just change ``clusterName`` in values.yaml. It can have any value as long as it is unique between clusters.
