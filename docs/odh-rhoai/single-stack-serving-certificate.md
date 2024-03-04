# Use Existing OpenShift Certificate for Single Stack Serving

By default, the Single Stack Serving in Openshift AI uses a self-signed certificate generated at installation for the endpoints that are created when deploying a server.
This can be counter-intuitive because if you already have certificates configured on your OpenShift cluster, they will be used by default for other types of endpoints like Routes.

The installation procedure for the Single Stack Serving available [here](https://access.redhat.com/documentation/en-us/red_hat_openshift_ai_self-managed/2-latest/html/serving_models/serving-large-models_serving-large-models#configuring-automated-installation-of-kserve_serving-large-models){:target="_blank"} (section 4.vi) states that you can provide your own certificate instead of using a self-signed one.

This following procedure explains how to use the same certificate that you already have for your OpenShift cluster.

## Procedure

- Configure OpenShift to use a valid certificate for accessing the Console and in general for any created Route (normally, this is something that was already done).
- From the **openshift-ingress** namespace, copy the content of a Secret whose name includes **"certs"**. For example `rhods-internal-primary-cert-bundle-secret` or `ingress-certs-....`. The content of the Secret (data) should contain two items, tls.cert and tls.key. They are the certificate and key that are used for all the OpenShift Routes.
- Cleanup the YAML to just keep the relevant content. It should look like this (the name of the secret will be different, it's normally tied to your cluster name):

    ```yaml
    kind: Secret
    apiVersion: v1
    metadata:
    name: rhods-internal-primary-cert-bundle-secret
    data:
    tls.crt: >-
        LS0tLS1CRUd...
    tls.key: >-
        LS0tLS1CRUd...
    type: kubernetes.io/tls
    ```

- Apply this YAML into the istio-system namespace (basically, copy the Secret from one namespace to the other).
- Following the Single Stack Serving installation procedure, in your DSC Configuration, refer to this secret for the kserve configuration (don’t forget to change the secretName for the one you just created):

    ```yaml
    kserve:
    devFlags: {}
    managementState: Managed
    serving:
        ingressGateway:
        certificate:
            secretName: rhods-internal-primary-cert-bundle-secret
            type: Provided
        managementState: Managed
        name: knative-serving
    ```

Your Model Servers will now be deployed with the same certificate as you are using for OpenShift Routes. If this is a trusted certificate, your Endpoints will be accessible using SSL without having to ignore error messages or create special configurations.
