QUALYS CONTAINERIZED SCANNER APPLIANCE (QCSA) HELM CHART
=========================================================

A Helm chart for deploying the Qualys Containerized Scanner Appliance
on Kubernetes. QCSA runs vulnerability scans against targets from
within your cluster, reporting results back to the Qualys platform.


PREREQUISITES
-------------

- Kubernetes cluster with EC2/VM-backed nodes (not Fargate)
- Helm 3.x
- A Qualys subscription with VM module enabled
- A personalization code (single-use, one per scanner)
- Your Qualys platform URL


GENERATING A PERSONALIZATION CODE
----------------------------------

1. Log in to the Qualys platform
2. Navigate to: Scans > Appliances > New > Containerized Scanner
3. Copy the personalization code

Note: Each code is single-use. If a deployment fails and you need to
redeploy, you must generate a new code.


FINDING YOUR QUALYS URL
------------------------

1. Log in to the Qualys platform
2. Go to: Help > About
3. Note the QualysGuard endpoint listed under Security Operations Center (SOC)

Or use the table below to match your platform:

  Platform   URL
  --------   ---
  US1        https://qualysguard.qualys.com
  US2        https://qualysguard.qg2.apps.qualys.com
  US3        https://qualysguard.qg3.apps.qualys.com
  US4        https://qualysguard.qg4.apps.qualys.com
  EU1        https://qualysguard.qualys.eu
  EU2        https://qualysguard.qg2.apps.qualys.eu
  EU3        https://qualysguard.qg3.apps.qualys.it
  IN1        https://qualysguard.qg1.apps.qualys.in
  CA1        https://qualysguard.qg1.apps.qualys.ca
  AE1        https://qualysguard.qg1.apps.qualys.ae
  UK1        https://qualysguard.qg1.apps.qualys.co.uk
  AU1        https://qualysguard.qg1.apps.qualys.com.au
  KSA1       https://qualysguard.qg1.apps.qualysksa.com


INSTALL
-------

  helm install qcsa ./qcsa_helm \
    --set-string personalizationCode=XXXXXXXXXXXX \
    --set qualysUrl=https://qualysguard.qg2.apps.qualys.com \
    -n qualys --create-namespace

Example with a UK1 subscription:

  helm install qcsa ./qcsa_helm \
    --set-string personalizationCode=21808056913298 \
    --set qualysUrl=https://qualysguard.qg1.apps.qualys.co.uk \
    -n qualys --create-namespace


UPGRADE
-------

  helm upgrade qcsa ./qcsa_helm \
    --set-string personalizationCode=XXXXXXXXXXXX \
    --set qualysUrl=https://qualysguard.qg2.apps.qualys.com \
    -n qualys


UNINSTALL
---------

  helm uninstall qcsa -n qualys


VERIFY
------

Check the pod is running:

  kubectl get pods -n qualys

View scanner logs:

  kubectl logs -l app.kubernetes.io/name=qcsa -n qualys -f

The scanner should appear in the Qualys UI under Scans > Appliances
within a few minutes of starting.


CONFIGURATION
-------------

All values can be set via --set flags or a custom values file
(helm install ... -f myvalues.yaml).

  Parameter              Default              Description
  ---------              -------              -----------
  personalizationCode    ""                   REQUIRED. Single-use code from Qualys UI.
  qualysUrl              ""                   REQUIRED. Qualys platform URL (see table above).
  scannerName            ""                   Optional friendly name for the scanner.
  existingSecret         ""                   Use a pre-existing K8s secret instead of creating one.
  existingSecretKey      "PERSONALIZATION_CODE"  Key within the existing secret.
  image.repository       qualys/qcsa          Container image repository.
  image.tag              "latest"             Container image tag.
  image.pullPolicy       IfNotPresent         Image pull policy.
  imagePullSecrets       []                   Registry pull secrets.
  securityContext.privileged  true             Scanner requires privileged mode.
  resources.limits.cpu       "2"              CPU limit.
  resources.limits.memory    4Gi              Memory limit.
  resources.requests.cpu     "500m"           CPU request.
  resources.requests.memory  2Gi              Memory request.
  proxy.enabled          false                Enable proxy for Qualys Cloud connectivity.
  proxy.proxyvalue       ""                   Proxy address as "host:port".
  proxy.proxycertpath    ""                   Path to proxy CA certificate.
  extraEnv               []                   Additional environment variables.
  tolerations            []                   Pod tolerations.
  nodeSelector           {}                   Node selector constraints.
  affinity               {}                   Pod affinity rules.
  replicaCount           1                    Number of scanner replicas.


PROXY EXAMPLE
-------------

  helm install qcsa ./qcsa_helm \
    --set-string personalizationCode=XXXXXXXXXXXX \
    --set qualysUrl=https://qualysguard.qg2.apps.qualys.com \
    --set proxy.enabled=true \
    --set proxy.proxyvalue="proxy.corp.com:3128" \
    -n qualys --create-namespace


PLAIN YAML MANIFEST (NO HELM)
------------------------------

A standalone Kubernetes manifest is also provided for environments
where Helm is not available:

  qcsa-manifest.yaml

Edit the file to fill in your base64-encoded personalization code
and Qualys URL, then apply directly:

  kubectl apply -f qcsa-manifest.yaml -n qualys


TROUBLESHOOTING
---------------

"Shared store is not mounted"
  The container needs a volume at /usr/local/qualys. This chart
  handles it automatically with emptyDir volumes.

"Private store is not mounted"
  The container also needs a volume at /usr/local/qualys/admin/etc.
  This chart provides both mounts.

"perscode has already been used"
  Personalization codes are single-use. Generate a new one in the
  Qualys UI and redeploy.

"required variable is not defined, use -e QUALYS_URL"
  You must set qualysUrl. See the platform URL table above.

Pod shows CrashLoopBackOff
  Check logs with: kubectl logs <pod-name> -n qualys
  The log output usually indicates the specific issue.
