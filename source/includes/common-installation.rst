.. start-install-minio-binary-desc

The following tabs provide examples of installing MinIO onto 64-bit Linux
operating systems using RPM, DEB, or binary. The RPM and DEB packages
automatically install MinIO to the necessary system paths and create a
``systemd`` service file for running MinIO automatically. MinIO strongly
recommends using RPM or DEB installation routes.

.. tab-set::

   .. tab-item:: RPM (RHEL)
      :sync: rpm

      Use the following commands to download the latest stable MinIO RPM and
      install it.

      .. code-block:: shell
         :class: copyable

         wget https://dl.min.io/server/minio/release/linux-amd64/minio.rpm
         sudo dnf install minio.rpm

   .. tab-item:: DEB (Debian/Ubuntu)
      :sync: deb

      Use the following commands to download the latest stable MinIO DEB and
      install it:

      .. code-block:: shell
         :class: copyable

         wget https://dl.min.io/server/minio/release/linux-amd64/minio.deb
         sudo dpkg -i minio.deb

   .. tab-item:: Binary
      :sync: binary

      Use the following commands to download the latest stable MinIO binary and
      install it to the system ``$PATH``:

      .. code-block:: shell
         :class: copyable

         wget https://dl.min.io/server/minio/release/linux-amd64/minio
         chmod +x minio
         sudo mv minio /usr/local/bin/

.. end-install-minio-binary-desc

.. start-install-minio-tls-desc

MinIO enables :ref:`Transport Layer Security (TLS) <minio-tls>` 1.2+ 
automatically upon detecting a valid x.509 certificate (``.crt``) and
private key (``.key``) in the MinIO ``${HOME}/.minio/certs`` directory.

If *any* MinIO server or client uses certificates signed by an unknown
Certificate Authority (self-signed or internal CA), you *must* place the CA
certs in the ``${HOME}/.minio/certs/CAs`` on all MinIO hosts in the deployment.
MinIO rejects invalid certificates (untrusted, expired, or malformed).

You can override the certificate directory using the 
:mc-cmd-option:`minio server certs-dir` commandline argument.

For more specific guidance on configuring MinIO for TLS, including multi-domain
support via Server Name Indication (SNI), see :ref:`minio-tls`. You can
optionally skip this step to deploy without TLS enabled. MinIO strongly
recommends *against* non-TLS deployments outside of early development.

.. end-install-minio-tls-desc

.. start-install-minio-systemd-desc

The ``.deb`` or ``.rpm`` packages install the following 
`systemd <https://www.freedesktop.org/wiki/Software/systemd/>`__ service file to 
``/etc/systemd/system/minio.service``. For binary installations, create this
file manually on all MinIO hosts:

.. code-block:: shell
   :class: copyable

   [Unit]
   Description=MinIO
   Documentation=https://docs.min.io
   Wants=network-online.target
   After=network-online.target
   AssertFileIsExecutable=/usr/local/bin/minio

   [Service]
   WorkingDirectory=/usr/local

   User=minio-user
   Group=minio-user
   ProtectProc=invisible

   EnvironmentFile=-/etc/default/minio
   ExecStartPre=/bin/bash -c "if [ -z \"${MINIO_VOLUMES}\" ]; then echo \"Variable MINIO_VOLUMES not set in /etc/default/minio\"; exit 1; fi"
   ExecStart=/usr/local/bin/minio server $MINIO_OPTS $MINIO_VOLUMES

   # Let systemd restart this service always
   Restart=always

   # Specifies the maximum file descriptor number that can be opened by this process
   LimitNOFILE=65536

   # Specifies the maximum number of threads this process can create
   TasksMax=infinity

   # Disable timeout logic and wait until process is stopped
   TimeoutStopSec=infinity
   SendSIGKILL=no

   [Install]
   WantedBy=multi-user.target

   # Built for ${project.name}-${project.version} (${project.name})

The ``minio.service`` file runs as the ``minio-user`` User and Group by default.
You can create the user and group using the ``groupadd`` and ``useradd``
commands. The following example creates the user, group, and sets permissions
to access the folder paths intended for use by MinIO. These commands typically
require root (``sudo``) permissions.

.. code-block:: shell
   :class: copyable

   groupadd -r minio-user
   useradd -M -r -g minio-user miniouser
   chown minio-user:minio-user /mnt/disk1 /mnt/disk2 /mnt/disk3 /mnt/disk4

The specified disk paths are provided as an example. Change them to match
the path to those disks intended for use by MinIO.

Alternatively, change the ``User`` and ``Group`` values to another user and
group on the system host with the necessary access and permissions.

MinIO publishes additional startup script examples on 
:minio-git:`github.com/minio/minio-service <minio-service>`.

.. end-install-minio-systemd-desc

.. start-install-minio-start-service-desc

.. code-block:: shell
   :class: copyable

   sudo systemctl start minio.service

Use the following commands to confirm the service is online and functional:

.. code-block:: shell
   :class: copyable

   sudo systemctl status minio.service
   journalctl -f -u minio.service

MinIO may log an increased number of non-critical warnings while the 
server processes connect and synchronize. These warnings are typically 
transient and should resolve as the deployment comes online.

.. end-install-minio-start-service-desc

.. start-install-minio-console-desc

Open your browser and access any of the MinIO hostnames at port ``:9001`` to
open the :ref:`MinIO Console <minio-console>` login page. For example,
``https://minio1.example.com:9001``.

Log in with the :guilabel:`MINIO_ROOT_USER` and :guilabel:`MINIO_ROOT_PASSWORD`
from the previous step.

.. image:: /images/minio-console-dashboard.png
   :width: 600px
   :alt: MinIO Console Dashboard displaying Monitoring Data
   :align: center

You can use the MinIO Console for general administration tasks like
Identity and Access Management, Metrics and Log Monitoring, or 
Server Configuration. Each MinIO server includes its own embedded MinIO
Console.

.. end-install-minio-console-desc