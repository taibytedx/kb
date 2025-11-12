# Getting Started: REST Data API
## Configure the REST Data API
### What Does This Article Cover?

This article covers considerations for configuring the REST Data API.

### Setup

Refer to the user guide for enabling the service: https://guide.highbyte.com/configuration/servers/restdata/

Authentication types supported are Bearer token and OAuth2.0 password grant type. For more information on these, see the article: https://support.highbyte.com/kb/how-to/connections/how-to-utilize-the-rest-connector

<img width="2290" height="1567" alt="image" src="https://github.com/user-attachments/assets/bf638a38-09bc-4d88-92d2-eda17bab8dfd" />

### Predefined endpoints

The endpoints for querying the Connection Inputs, Outputs, Models, and Instances are automatically created.

### Creating endpoints

For custom endpoints, Pipelines can be exposed using the API Trigger.

#### API Trigger
##### Parameters
- When parameters are not defined, there is no restriction on the request body.
- When parameters are defined, then additional properties in the request body are ignored

Defined parameters can be queried for discovery on the following endpoints:

- /data/v1/pipelines/params
- /data/v1/pipelines/{pipelineName}/params endpoints

This may also benefit discovery by AI Agents for pipelines that are exposed on the MCP Server.

<p align="center">
  <img width="322" height="330" alt="image" src="https://github.com/user-attachments/assets/56aab578-3907-44f0-bed8-f979d96bd077" />
</p>

##### Default value
If a default value is not defined for a parameter, then it is required in the request body; otherwise, the request will error if the parameter is missing.
If a default value is defined, then the default value will be processed if the parameter is missing from the request body.
#### HTTP Headers

After the request is validated by the API Trigger, the HTTP Headers will be available as event.metadata.HTTPRequest as shown in the debug window below:

<img width="1147" height="857" alt="image" src="https://github.com/user-attachments/assets/7bd49ca8-9a80-4602-a79c-e7bfb43844e9" />

### Replay

Replays are logged as "RestDataServer" under the Trigger column, which may be useful when combined with other Trigger types.

<img width="1119" height="555" alt="image" src="https://github.com/user-attachments/assets/9a0d4dac-8af1-4297-9c47-0f6d384c03f3" />

### Additional Resources
- [User guide - REST Data API](https://guide.highbyte.com/configuration/servers/restdata/)
- [Knowledge Base - REST Connector Authentication Methods](https://support.highbyte.com/kb/how-to/connections/how-to-utilize-the-rest-connector)
- [What is OpenAPI?](https://www.openapis.org/what-is-openapi)
