# Helm Values Configuration Options

## Available options

| Field | Default | Description |
| - | - | - |
| `verbosityLevel`| 3 | Sets the verbosity level of the AGIC logging infrastructure. See [Logging Levels](troubleshooting.md#logging-levels) for possible values. |
| `reconcilePeriodSeconds` | | Enable periodic reconciliation to checks if the latest gateway configuration is different from what it cached. Range: 30 - 300 seconds. Disabled by default. |
| `appgw.applicationGatewayID` | | Resource Id of the Application Gateway. Example: `applicationgatewayd0f0` |
| `appgw.subscriptionId` | Default is agent node pool's subscriptionId derived from CloudProvider config  | The Azure Subscription ID in which App Gateway resides. Example: `a123b234-a3b4-557d-b2df-a0bc12de1234` |
| `appgw.resourceGroup` | Default is agent node pool's resource group derived from CloudProvider config | Name of the Azure Resource Group in which App Gateway was created. Example: `app-gw-resource-group` |
| `appgw.name` | | Name of the Application Gateway. Example: `applicationgatewayd0f0` |
| `appgw.environment`| `AZUREPUBLICCLOUD` | Specify which cloud environment. Possbile values: `AZURECHINACLOUD`, `AZUREGERMANCLOUD`, `AZUREPUBLICCLOUD`, `AZUREUSGOVERNMENTCLOUD` |
| `appgw.shared` | false | This boolean flag should be defaulted to `false`. Set to `true` should you need a [Shared App Gateway](setup/install-existing.md#multi-cluster--shared-app-gateway). |
| `appgw.subResourceNamePrefix` | No prefix if empty | Prefix that should be used in the naming of the Application Gateway's sub-resources|
| `kubernetes.watchNamespace` | Watches all if empty | Specify the name space, which AGIC should watch. This could be a single string value, or a comma-separated list of namespaces. |
| `kubernetes.securityContext` | `runAsUser: 0` | Specify security context to use with AGIC deployment. By default, AGIC will assume `root` permission. Jump to [Security Context](#security-context) for more information. |
| `kubernetes.podAnnotations` | `{}` | Specify custom annotations for AGIC pod |
| `kubernetes.nodeSelector` | `{}` | Scheduling node selector |
| `kubernetes.tolerations` | `[]` | Scheduling tolerations |
| `kubernetes.affinity` | `{}` | Scheduling affinity |
| `kubernetes.ingressClass` | `azure/application-gateway` | Specify a [custom ingress class](features\custom-ingress-class.md) which will be used to match `kubernetes.io/ingress.class` in ingress manifest |
| `rbac.enabled` | false | Specify true if kubernetes cluster is rbac enabled |
| `armAuth.type` | | could be `aadPodIdentity` or `servicePrincipal` |
| `armAuth.identityResourceID` | | Resource ID of the Azure Managed Identity |
| `armAuth.identityClientId` | | The Client ID of the Identity. See below for more information on Identity |
| `armAuth.secretJSON` | | Only needed when Service Principal Secret type is chosen (when `armAuth.type` has been set to `servicePrincipal`) |
| `nodeSelector` | `{}` | (Legacy: use `kubernetes.nodeSelector` instead) Scheduling node selector |

## Example

```yaml
appgw:
    applicationGatewayID: <application-gateway-resource-id>
    environment: "AZUREUSGOVERNMENTCLOUD" # default: AZUREPUBLICCLOUD

armAuth:
    type: aadPodIdentity
    identityResourceID: <identityResourceId>
    identityClientID:  <identityClientId>

kubernetes:
  nodeSelector: {}
  tolerations: []
  affinity: {}

rbac:
    enabled: false
```

---
### Security Context
By default, AGIC will assume `root` permission which allows it to read `cloud-provider` config and get meta-data information about the cluster.
If you want AGIC to run without `root` access, then make sure that AGIC is installed with atleast the following information to run successfully:
* `appgw.subscriptionId`, `appgw.resourceGroup` and `appgw.name`  
or
* `appgw.applicationGatewayID`

AGIC also uses `cloud-provider` config to get Node's Virtual Network Name / Subscription and Route table name. If AGIC is not able to reach this information,  It will skip assigning the Node's route table to Application Gateway's subnet which is required when using `kubenet` network plugin. To workaround, this assignment can be performed manually.

