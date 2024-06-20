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

## Other Workarounds

If the above method does not work or you don't want or can't do any modification, you can try to bypass the certificate verification in your application or code. Depending on the library used, there are different solutions.

### Using Langchain with OpenAI API compatible runtimes

The underlying library used for communication by the base OpenAI module of Langchain is `httpx`. You can directly specify `httpx` settings when you instantiate the llm object in Langchain. With the following settings on the last two lines of this example, any certificate error will be ignored:

```python
import httpx
# LLM definition
llm = VLLMOpenAI(
    openai_api_key="EMPTY",
    openai_api_base= f"{inference_server_url}/v1",
    model_name="/mnt/models/",
    top_p=0.92,
    temperature=0.01,
    max_tokens=512,
    presence_penalty=1.03,
    streaming=True,
    callbacks=[StreamingStdOutCallbackHandler()]
    async_client=httpx.AsyncClient(verify=False),
    http_client=httpx.Client(verify=False)
)
```
