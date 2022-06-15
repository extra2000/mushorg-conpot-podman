Production Deployment
=====================

Example how to deploy Conpot for single-instance production environment.

Clone repositories
------------------

.. code-block:: bash

    mkdir ~/extra2000
    cd ~/extra2000
    git clone https://github.com/extra2000/mushorg-conpot-podman.git
    git clone https://github.com/extra2000/mushorg-conpot.git mushorg-conpot-podman/src/conpot

Then, ``cd`` into project root directory:

.. code-block:: bash

    cd mushorg-conpot-podman

Build conpot Image
------------------

From the project root directory, ``cd`` into ``src/conpot/`` and then build:

.. code-block:: bash

    cd src/conpot/
    podman build -t extra2000/mushorg/conpot .

Deploy conpot
-------------

From the project root directory, ``cd`` into ``deployment/production/conpot``:

.. code-block:: bash

    cd deployment/production/conpot

Create config files and fix permissions:

.. code-block:: bash

    cp -v configmaps/conpot.yaml{.example,}
    cp -v configs/config.cfg{.example,}
    cp -rv configs/templates{.example,}
    chmod og+r ./configs/*

.. note::

    For security purpose, all modules are listening to ``127.0.0.1`` by default. To change to ``0.0.0.0``, modify modules in ``configs/templates/default/``.

Create pod file:

.. code-block:: bash

    cp -v conpot-pod.yaml{.example,}

For SELinux platform, label the following files to allow to be mounted into container:

.. code-block:: bash

    chcon -R -v -t container_file_t ./configs

Create SELinux security policy:

.. code-block:: bash

    cp -v selinux/conpot_podman.cil{.example,}

Load SELinux security policy:

.. code-block:: bash

    sudo semodule -i selinux/conpot_podman.cil /usr/share/udica/templates/base_container.cil

Verify that the SELinux module exists:

.. code-block:: bash

    sudo semodule --list | grep -e "conpot_podman"

Deploy conpot:

.. code-block:: bash

    podman play kube --configmap configmaps/conpot.yaml --seccomp-profile-root ./seccomp conpot-pod.yaml

Create systemd files to run at startup:

.. code-block:: bash

    mkdir -pv ~/.config/systemd/user
    cd ~/.config/systemd/user
    podman generate systemd --files --name conpot-pod-srv01
    systemctl --user enable container-conpot-pod-srv01.service

Testing
-------

Try ``ftp`` access:

.. code-block:: bash

    ftp 127.0.0.1
