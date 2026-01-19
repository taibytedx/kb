# How Intelligence Hub handles data types

## Learn how data types are managed across explicitly defined types like from OPC-UA and implicit types like JSON over MQTT.

### What This Article Covers

This article explains how Intelligence Hub handles data types when:

-   Reading typed payloads (OPC-UA)
-   Reading plain JSON (MQTT)
-   Writing to SQL Outputs
-   Writing to OPC UA
-   Handling in Pipeline

### Summary of Behavior

When reading a value from JSON over MQTT, the payload does not specify its type like float, double, etc. like the way an OPC UA connection can, so its type will be derived from the value. When writing that value to OPC UA, Intelligence Hub will cast the value to the OPC UA tag's cached data type. A double value in a JSON payload will maintain precision when writing to a double in OPC UA, but will lose precision if written to a less precise type like float. JSON values are implicitly handled unless specified in a Model definition and if the data is being instantiated at the target then it will be typecasted according to the native Intelligence Hub mapping table (i.e. creating a column in a table and setting its type. An example of this type mapping for writing to SQL is found [**here**](https://guide.highbyte.com/configuration/connect/connections/sql/microsoft_sql_server/#create-table)).

### How Intelligence Hub Assigns Types at Input and Output
If an input includes typed data, Intelligence Hub converts them to its own native types at input and keeps those types associated with the payload object and its underlying schema.

### Example 1: OPC UA branch read > Postgres create table

-   Image 1: Reads multiple OPC UA types via a branch read
-   Image 2: Writes the Component object reference to Postgres using "create table"
-   Image 3: The resulting Postgres column types are respected by OPC UA types

![opcua branch read on various types](https://5505680.fs1.hubspotusercontent-na1.net/hubfs/5505680/opcua%20branch%20read%20on%20various%20types.png)

![doing test write refrencing that input componene to create a new table in postghres](https://5505680.fs1.hubspotusercontent-na1.net/hubfs/5505680/doing%20test%20write%20refrencing%20that%20input%20componene%20to%20create%20a%20new%20table%20in%20postghres.png)  
  
![postgres column types reflect the OPC UA types](https://5505680.fs1.hubspotusercontent-na1.net/hubfs/5505680/postgres%20column%20types%20reflect%20the%20OPC%20UA%20types.png)

### Example 2: MQTT JSON > Postgres create table
When the Input is JSON from a connection like MQTT, types are implicitly derived: integers map to Int64 and decimals map to Real64
-   Image 1: The object expression is published into MQTT using UNS Client
-   Image 2: Captured using MQTT Input
-   Image 3: The JSON data types are implicitly derived and mapped to the native types and published out to create a Postgres table
-   Image 4: The object explorer shows that the column types are mainly Int8 (64-bit signed integer) or Float8 (64-bit float)

**UNS Client Publish**
```
{
  "boolean": true,
  "int": 1,
  "float1p0": 1.0,
  "float1p1": 1.1,
  "string": "1"
}
```

<img width="945" height="749" alt="image" src="https://github.com/user-attachments/assets/961a2a2e-5643-4b1c-b243-01f556023a5d" />
<img width="1162" height="719" alt="image" src="https://github.com/user-attachments/assets/258b6a70-0c18-4fe3-9436-c10437c6b72b" />
<img width="1700" height="730" alt="image" src="https://github.com/user-attachments/assets/57dc9c57-dacd-4c9e-8ca3-bf97c8b01d00" />
<img width="1153" height="971" alt="image" src="https://github.com/user-attachments/assets/3c7f1b31-de90-4cda-8bca-6b3c161fdebb" />

### Example 3: Expression JSON > Postgres create table
When the JSON is defined with a JS object Expression, the type handling is generic and can deviate from how it gets mapped from a Connection Input. More info on this further below.
-   Image 1: The same payload is written out as a Test Write Expression to create the Postgres table 
-   Image 2: The object explorer shows how 1.0 was truncated to 1 as Int8 (64-bit signed integer), but 1.1 was mapped as float8 (64-bit float)

**Test Write Expression**
```
const payload = {
  "boolean": true,
  "int": 1,
  "float1p0": 1.0,
  "float1p1": 1.1,
  "string": "1"
}; 
payload;
```

<img width="1171" height="639" alt="image" src="https://github.com/user-attachments/assets/bf7a7dc7-13a9-46b9-8948-c6f2198adab8" />
<img width="1160" height="969" alt="image" src="https://github.com/user-attachments/assets/dd6eff06-86a5-416c-b2a3-72f343670ede" />

### Example 4: Expression JSON > **Model Stage** > Postgres create table
Modeling using Instance or Model Stage can be used to specify the types.
-   Image 1: The same payload is written as a Debug Write 
-   Image 2: The implicitly derived types are Modeled as follows - *the event.value object is mapped to the .root property so the child properties are automapped for same names*:
    -    boolean - boolean (true) is casted to real32 (1)
    -    int - int64 (1) is casted to int8 (1)
    -    float1p0 - int64 (1) is casted to real64 (1)
    -    float1p1 - float64 (1.1) is casted to real32 (1)
    -    string - string ("1") is casted to int8 (1)
-   Image 3: Points out the specified types for each attribute
-   Image 4: A Transform Stage elevates the child properties and remove the root property before writing out to postgres and creating a table
-   Image 5: The object explorer shows the table columns respecting the specified types
    -    boolean was cast as float4 (32-bit float)
    -    int as int2 (16-bit signed int)
    -    float1p0 as float8 (64-bit float)
    -    float1p1 as float4 (32-bit float)
    -    string as int2 (16-bit sign int)

<img width="1637" height="820" alt="image" src="https://github.com/user-attachments/assets/db72dcac-e990-48ec-b7b3-e22e043113b1" />
<img width="1710" height="931" alt="image" src="https://github.com/user-attachments/assets/58a4bc77-c84c-434d-b0bb-cd8c7a8b1bf8" />
<img width="827" height="766" alt="image" src="https://github.com/user-attachments/assets/dd7a3e1a-dc08-487c-899f-f5908d9787c2" />
<img width="1569" height="613" alt="image" src="https://github.com/user-attachments/assets/670824f8-2996-4b0d-acee-d6115a5457de" />
<img width="1149" height="966" alt="image" src="https://github.com/user-attachments/assets/2d95d917-6536-41a8-979b-1547db8bc039" />

### How Outputs Choose or Enforce Types 

When writing to a destination:

-   If the destination has defined types (e.g., existing SQL table, OPC UA tags), Intelligence Hub casts values to match those types. 
-   If the destination has no types yet (create table), Intelligence Hub assigns types based on native typing or derived values. 

If the destination schema is required to match specific types, [Modeling](https://guide.highbyte.com/configuration/model/models/) is often able to define the necessary types. Otherwise, a value like 123 and 123.0 may become an Integer, which can cause trouble later if the next value is 123.1 and gets miscasted or truncated to 123. For writes to systems like OPC UA this is usually not a problem since tag types that are predefined.

### OPC UA Write Behavior: Double Value Written to a Float Tag

Writing to a double-precision value to an OPC UA float tag (which maps to the native type Real32) will truncate to 32-bit precision.

-   Image 1:
    -   Value written: 1.123456789
    -   Destination tage type: Float (Real32)  
        Value stored/read back: 1.1234568

![writing a double value](https://5505680.fs1.hubspotusercontent-na1.net/hubfs/5505680/writing%20a%20double%20value.png)

-   Image 2:
    -   Value stored/read back: 1.1234568

![reading it back out shows it got rounded](https://5505680.fs1.hubspotusercontent-na1.net/hubfs/5505680/reading%20it%20back%20out%20shows%20it%20got%20rounded.png)

#### Writing an object structure

When writing an object structure to OPC UA, each leaf tag is cast independently to its destination tag type 

### Retaining Data Types Inside the Pipeline

Intelligence Hub can retain data types inside a pipeline under certain circumstances. This depends on how the types are established and how object is transformed. Type metadata is retained when native types are established by:

-   An Input schema
-   An Instance
-   A Model Stage 

**We saw in Example 4 above how the .root object was elevated but the child attributes retained their type info.**

### How Types Can Be Lost (Transform Patterns)

A Transform does not necessarily remove type information. If the same _event.value_ object is modified, data types are retained. However, if the _event.value_ object is completely replaced, data type information is lost. So a transform stage can modify the value of a property, add new properties or delete them. But when new properties are added, there isn't a way to explicitly set the type for that property directly in the Transform stage.

[The Model Stage](https://guide.highbyte.com/configuration/pipeline/stages/common/model/) may be used to explicitly set attribute properties. Then, _event.metadata_ may be adjusted, but this will not affect the object that _event.value_ is referencing.  
  

// event.value = {float:1, double:1, int:1}  
  
const newObj1 = {...event.value}; // this creates a new object that copies the values of event.value so type info is lost.  
  
const newObj2 = Object.assign( {}, event.value); // this creates a new {} and copies the values of event.value into it, so type info is lost  
  
const newObj3 = {  
  
    "att1":event.value.float  
  
}; //  
  
This creates a new object where we copy in the value of event.value.float so type info is lost

Example of merging the properties of .Source1 to the root level.   
  
const. newObj4 = Object.assign(event.value, {"newAtt":123}); 

Instead of building or reshaping the payload by manually creating  _key: value_ pairs in a Transform, it is often advantageous to create payloads directly in a [Model Stage](https://guide.highbyte.com/configuration/pipeline/stages/common/model/) (or by parameterizing the output of a Read Stage). That way, the fields you create can have clear, consistent types.

Most pipeline stages preserve type information when they move typed objects between higher-level attributes, such as [Merge Read](https://guide.highbyte.com/configuration/pipeline/stages/io/merge_read/) or [Flatten](https://guide.highbyte.com/configuration/pipeline/stages/common/flatten/). But creating new objects will lose type data as discussed above. Therefore, a common pattern is to finish with a Model Stage (to set the final shape and types) and/or a Model Validation Stage (to confirm the payload matches the expected types).

![model stage at end to package up payload](https://5505680.fs1.hubspotusercontent-na1.net/hubfs/5505680/model%20stage%20at%20end%20to%20package%20up%20payload.png)

### Other Related Material:

-   [OPC UA User Guide](https://guide.highbyte.com/configuration/connect/connections/opc_ua_tcp/#data-type:~:text=NodeId%20and%20Path.-,Data%20Type,the%20write%20will%20fail%20and%20output%20an%20error%20to%20the%20logs.,-Max%20Branch%20Depth)
-   [Microsoft SQL Server User Guide](https://guide.highbyte.com/configuration/connect/connections/sql/microsoft_sql_server/)
