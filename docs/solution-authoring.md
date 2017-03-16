---
layout: default
title: Solution authoring
navigation_weight: 3
---
# Solution authoring
## Solution manifest
### LocationToExclude
### Parameters
#### Parameter
#### Credential
## Provisioning steps
### ArmDeployment
### Manual
Manual steps are most typically used to display post-deployment instructions. For example:
```xml
<Manual title="Done">
  <Instructions src="Instructions.md" format="markdown" />
</Manual>
```

CIQS has no restrictions as to where *Manual* steps can appear inside the __&lt;ProvisioningSteps&gt;__ section of the *Manifest*, nor is there any limit on their number.

The platform guarantees that no automated activities will be injected prior to the *Manual* steps that appear in the very beginning. This allows collecting all input parameters immediately after a deployment is created and performing the rest of the process without interruptions.

The two examples below demonstrate slightly different ways of collecting input parameters with *Manual* steps. The first one uses an ARM template as a *parameter source* and produces a configuration page exactly like __&lt;ArmDeployment&gt;__, except no deployment occurs afterwards. Instead, all parameters enter the **{Inputs.*}** variable pool.

```xml
<Manual parameterSource="armTemplate.json" title="Setup SQL server account">
  <Parameters>
    <Credential type="sql" username="sqlServerUsername" password="sqlServerPassword" />
  </Parameters>
</Manual>
```
If no **parameterSource** attribute is provided, all parameters need to be defined explicitly. This allows collecting arbitrary parameters and use them in various contexts like, for instance, as inputs of the  **Functions**.
```xml
<Manual title="Configure Twitter listener">
  <Parameters>
    <Parameter name="twitterKeywords" description="Twitter topics" type="string"
      defaultValue="@MicrosoftR,@OpenAtMicrosoft,@Azure,#CortanaIntelligence">

      <ExtraDescription>
        Comma-separated list of words, phrases, #hashtags and @mentions
      </ExtraDescription>
    </Parameter>
    <Parameter name="oauthConsumerKey" description="Consumer key (API key)" type="string" />
    <Parameter name="oauthConsumerSecret" description="Consumer secret (API secret)" type="string" />
    <Parameter name="oauthToken" description="Access token" type="string" />
    <Parameter name="oauthTokenSecret" description="Access token secret" type="string" />
  </Parameters>
</Manual>
```
Finally, if all parameters required to perform an ARM deployment are already present in the variable pool, _{Inputs.*}_ or _{Outputs.*}_, it is not necessary to explicitly define them as part of the __&lt;ArmDeployment&gt;__. Instead, use **autoResolveParameters**. For example, assuming that a *Manual* step has been used to collect parameters for *armTemplate.json*, the subsequent __&lt;ArmDeployment&gt;__ provisioning step can be defined as follows:

```xml
<ArmDeployment source="armTemplate.json" autoResolveParameters="true" title="Deploying..." />
```

A demonstration of this technique can be found in the [twitterstreaming](https://github.com/Azure/Azure-CortanaIntelligence-SolutionAuthoringWorkspace/tree/master/Samples/009-twitterstreaming) SAW sample.

### Function
### WebJob
### AzureMLWebService
* **Summary**

  AzureMlWebService is a first-party provisioning step that empowers solution authors provisioning an Azure Machine Learning experiment from gallery and then deploying as a web service easily. This feature empowers pattern authors to:
  1) Provision an Azure ML experiment from [Cortana Intelligence Gallery](https://gallery.cortanaintelligence.com/experiments) by only providing the `GalleryUrl`;
  2) Modify the **Experiment Graph** of the provisioned Azure ML experiment with a customized funciton plug-in;
  3) Provisioning multiple Azure ML web services;
  4) Create high-throughput Azure ML web service.

* **Documentation**
  * __&lt;AzureMlWebService/&gt;__
    
    To use this feature, please specify `<AzureMlWebService/>` within `<ProvisioningSteps/>` in your solution **Manifest.xml**.
    
    * **Attributes**
    
      | Name    | Description |
      | ------------ | -------------|
      | *title*: `string` | The title to be displayed in CIQS deployment page |
      | *hiddenParameters*: `boolean` | Set to `true` if this step is supposed to be automated; otherwise `false` | 

	* **Properties**
	
      | Name | Description |
      | ------------ | ------------- |
      | *GalleryUrl*: `string` | The url of the experiment to be provisioned on Gallery, such as: https://gallery.cortanaintelligence.com/Details/nyc-taxi-binary-classification-scoring-exp-2 |
      | *Outputs*: `object` | Allows users to overwrite the outputs; This is useful when provisioning more than one Azure ML experiments so that the outputs of each other will not collide |
      | *HighThroughputEndpoint*: `object` | Allows users to create a high-throughput Web service endpoint |
      | *ModifyExperimentGraph*: `object` | Allows users to plug-in a custom function to modify the experiment graph of the provisioned experiment |
      
      * __&lt;Outputs/&gt;__ (*Optional*)

        This tag is used to **overwrite** the default output names, so that outputs from multiple *AzureMlWebService* provisionings will not overwrite each other.

        **Attributes**

        All properties below can be used as attributes in **Camel Case**. Please see a Github sample [here](https://github.com/Azure/Azure-CortanaIntelligence-SolutionAuthoringWorkspace/blob/master/Samples/012-mlwebsvc/core/Manifest.xml).

        **Properties**
        | Name | Description |
        | ------------ | ------------- |
        | *ExperimentUrl*: `string` | Overwriting the output name of the Azure ML experiment url |
        | *WebServiceApiUrl*: `string` | Overwriting the output name of the Azure ML web service api url |
        | *WebServiceApiKey*: `string` | Overwriting the output name of the Azure ML web service api key |
        | *WebServiceHelpUrl*: `string` | Overwriting the output name of the Azure ML web service help url |

      * __&lt;HighThroughputEndpoint/&gt;__ (*Optional*)

        This tag is used to create a high-throughput Web service endpoint as a result of this provisioning step.

        **Attributes**

        All properties below can be used as attributes in **Camel Case**. Please see a Github sample [here](https://github.com/Azure/Azure-CortanaIntelligence-SolutionAuthoringWorkspace/blob/master/Samples/012-mlwebsvc/core/Manifest.xml).

        **Properties**
        | Name | Description |
        | ------------ | ------------- |
        | *EndpointName*: `string` | Endpoint names must be 24 character or less in length, and must be made up of lower-case letters or numbers; If not specified, the default value is "*secondep*" |
        | *ThrottleLevel*: `string` | Allowed values: `High` or `Low`; If not specified, the default value is `Low` |
        | *MaxConcurrentCalls*: `string` | The maximum concurrent calls for Azure ML Web service is between 1 and 200. See [here](https://github.com/hning86/azuremlps#new-amlwebservice) for details; If not specified, the default value is `4` |

      * __&lt;ModifyExperimentGraph/&gt;__ (*Optional*)

        This tag is used to plug-in a customized function to modify the experiment graph of the provisioned experiment.

        **Attributes**
        | Name | Description |
        | ------------ | ------------- |
        | *name*: `string` | **The name of** the customized function to modify the ML experiment graph |

        **Properties**
        | Name | Description |
        | ------------ | ------------- |
        | *Parameters*: `array` | Array of __&lt;Parameter/&gt;__ that is used in the customized function |

        **Examples**

        The sample code in **Manifest.xml**:
        ```xml
        <AzureMlWebService title="Creating Energy Forecasting ML Web service" hiddenParameter="true">
          <GalleryUrl>https://gallery.cortanaintelligence.com/Details/975ed028d71b490b9268d35094138358</GalleryUrl>
          <ModifyExperimentGraph name="UpdateDatabaseInfoInExperiment">
            <Parameters>
              <Parameter type="string" name="sqlServer" defaultValue="{Outputs.sqlServerName}" description="SQL Server Name"/>
              <Parameter type="string" name="sqlUser" defaultValue="{Outputs.sqlServerUserName}" description="SQL Server User Name"/>
              <Parameter type="string" name="sqlPassword" defaultValue="{Outputs.sqlServerPassword}" description="SQL Server Password"/>
              <Parameter type="string" name="sqlDatabase" defaultValue="{Outputs.databaseName}" description="SQL Server Database Name"/>
            </Parameters>
          </ModifyExperimentGraph>
        </AzureMlWebService>
        ```

        A custom function named "**UpdateDatabaseInfoInExperiment**" need to be added into the solution alongside with the above code snippet. In this custom function, all you need to do is to get the "**graphJsonObject**" parameter and then return the modified value as the function output.
        ```c#
        #load "..\CiqsHelpers\All.csx"

        public static async Task<object> Run(HttpRequestMessage req, TraceWriter log)
        {
            var parametersReader = await CiqsInputParametersReader.FromHttpRequestMessage(req);
            dynamic graphJsonObject = serializer.Deserialize<object>(parametersReader.GetParameter<string>("graphJsonObject"));

            /*do whatever is needed on the graph JsonObject and return the graph JsonObject*/

            return graphJsonObject;
        }
        ```

	* **Default Outputs**
	
      | Name | Description
      | ------------ | ------------- |
      | *experimentUrl*: `string` | The Azure ML experiment url |
      | *webServiceApiUrl*: `string` | The Azure ML web service api url |
      | *webServiceApiKey*: `string` | The Azure ML web service api key |
      | *webServiceHelpUrl*: `string` | The Azure ML web service help url |
      | *mlListExperimentsUrl*: `string` | The Azure ML experiments list url |

* **Examples**
  
  Please see a Github sample [here](https://github.com/Azure/Azure-CortanaIntelligence-SolutionAuthoringWorkspace/tree/master/Samples/012-mlwebsvc).
  
  The simplest use case would be provisioning a single Azure ML experiment and deploying a default Web service endpoint:
    ```xml
    <AzureMlWebService title="Copying and experiment from Gallery and deploy as Web Service" hiddenParameters ="true">
      <GalleryUrl>https://gallery.cortanaintelligence.com/Details/nyc-taxi-binary-classification-scoring-exp-2</GalleryUrl>
    </AzureMlWebService>
    ```
  A slightly more complex use case would be creating a high-throughput Web service endpoint for the experiment:
    ```xml
    <AzureMlWebService title="Copying and experiment from Gallery and deploy as Web Service" hiddenParameters ="true">
      <GalleryUrl>https://gallery.cortanaintelligence.com/Details/nyc-taxi-binary-classification-scoring-exp-2</GalleryUrl>
      <HighThroughputEndpoint>
        <EndpointName>HighThroughputEndpoint</EndpointName>
        <ThrottleLevel>High</ThrottleLevel>
        <MaxConcurrentCalls>100</MaxConcurrentCalls>
      </HighThroughputEndpoint>
    </AzureMlWebService>
    ```
  Another common use case would be provisioning multiple Azure ML experiments, with output overwritten to avoid conflicts:
  ```xml
  <AzureMlWebService title="Copying NYC taxi xperiment from Gallery and deploy as Web Service" hiddenParameters ="true">
      <GalleryUrl>https://gallery.cortanaintelligence.com/Details/nyc-taxi-binary-classification-scoring-exp-2</GalleryUrl>
      <Outputs experimentUrl="mlFunctionEndpoint1" webServiceApiUrl="mlFunctionApiUrl1" webServiceApiKey="mlFunctionApiKey1" />
    </AzureMlWebService>
    <AzureMlWebService title="Copying connected car Experiment from Gallery and deploy as Web Service" hiddenParameters ="true">
      <GalleryUrl>https://gallery.cortanaintelligence.com/Details/connected-cars-aml-v2-noreader-scoring-exp-2</GalleryUrl>
      <Outputs>
        <ExperimentUrl>mlFunctionEndpoint2</ExperimentUrl>
        <WebServiceApiUrl>mlFunctionApiUrl2</WebServiceApiUrl>
        <WebServiceApiKey>mlFunctionApiKey2</WebServiceApiKey>
        <WebServiceHelpUrl>mlFunctionHelpUrl2</WebServiceHelpUrl>
      </Outputs>
    </AzureMlWebService>
    <AzureMlWebService title="Copying predictive maintenance Experiment from Gallery and deploy as Web Service" hiddenParameters ="true">
      <GalleryUrl>https://gallery.cortanaintelligence.com/Details/bcae226bc74a4cbbb0ff700ac97448bf</GalleryUrl>
    </AzureMlWebService>
  ```

### SolutionDashboard
### WebJobDeployment