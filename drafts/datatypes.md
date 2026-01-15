# How HighByte Handles Floating-Point Data when OPC UA only Supports Single Precision Floating Point

## Learn how data types are managed across explicitly defined types like from OPC-UA and implicit types like JSON over MQTT.

### What This Article Covers

This article explains how Intelligence Hub handles numeric typing when:

-   Reading plain JSON over MQTT
-   Reading typed payloads
-   Writing to SQL Outputs
-   Writing to OPC UA tags 

### Summary of Behavior

When reading a numeric value from plain JSON over MQTT, the payload does not explicitly say "float vs double" the way an OPC UA connection will. When writing that value to OPC UA, Intelligence Hub will cast the value to the OPC UA tag's data type at write time. Therefore, Intelligence Hub reads “generic number” from JSON and forces it into a smaller, less precise numeric type (Real32), so the system may truncate it to what that smaller type can store.

### How Intelligence Hub Assigns Types at Ingestion

If an input includes typed data, Intelligence Hub converts them to its own native types at ingestion and keeps those types associated with the payload in its schema.

-   Example: OPC UA branch read > Postgres create table
    -   Image 1: Reads multiple OPC UA types via a branch read
    -   Image 2: Writes the object from Postgres using "create table" 
    -   Image 3: The resulting Postgres Column types reflects OPC UA types

 ![opcua branch read on various types](https://5505680.fs1.hubspotusercontent-na1.net/hubfs/5505680/opcua%20branch%20read%20on%20various%20types.png)

![doing test write refrencing that input componene to create a new table in postghres](https://5505680.fs1.hubspotusercontent-na1.net/hubfs/5505680/doing%20test%20write%20refrencing%20that%20input%20componene%20to%20create%20a%20new%20table%20in%20postghres.png)  
  

![postgres column types reflect the OPC UA types](https://5505680.fs1.hubspotusercontent-na1.net/hubfs/5505680/postgres%20column%20types%20reflect%20the%20OPC%20UA%20types.png)

If the Input is plain JSON over MQTT, types are implicitly derived beyond JSON's single numeric concept 

-   Example: MQTT JSON > Postgres create table 
    -   Image 1: The same payload is read as above, but as a plain JSON without the underlying type schema to create the Postgres table 
    -   Image 2: The object explorer shows that the column types are mainly derived as type Int8 (8-bit Integer)

![same payload as before as a plain json to create postgres table](https://5505680.fs1.hubspotusercontent-na1.net/hubfs/5505680/same%20payload%20as%20before%20as%20a%20plain%20json%20to%20create%20postgres%20table.png)

![object explorere shows column types were mainely create as Integer](https://5505680.fs1.hubspotusercontent-na1.net/hubfs/5505680/object%20explorere%20shows%20column%20types%20were%20mainely%20create%20as%20Integer.png)

###  How Outputs Choose or Enforce Types 

When writing to a destination:

-   If the destination has defined types (e.g., existing SQL table, OPC UA tags), Intelligence Hub casts values to match those types. 
-   If the destination has no types yet (create table), Intelligence Hub assigns types based on native typing or derived values. 

If the destination schema is required to match specific types, [Modeling](https://guide.highbyte.com/configuration/model/models/) is often able to define the necessary types. Otherwise, a value like 123 may become an Integer, which can cause trouble later if the next value is 123.1 and gets casted or truncated. For OPC UA writes this is usually not a problem because tag types are predefined.

### OPC UA Write Behavior: Double Value Written to a Float Tag

Writing to a double-precision value to an OPC UA Float tag (mapped to OPC UA-native type Real32) will truncate to Float32 precision.

-   Image 1:
    -    Value written: 1.123456789
    -   Destination tage type: Float (Real32)  
        Value stored/read back: 1.1234568

  

![writing a double value](https://5505680.fs1.hubspotusercontent-na1.net/hubfs/5505680/writing%20a%20double%20value.png)

-   Image 2:
    -   Value stored/read back: 1.1234568

![reading it back out shows it got rounded](https://5505680.fs1.hubspotusercontent-na1.net/hubfs/5505680/reading%20it%20back%20out%20shows%20it%20got%20rounded.png)

#### Writing an object structure

When writing an object structure to OPC UA, each leaf tag is cast independently to its destination tag type 

### Retaining Numeric Types Inside the Pipeline

Intelligence Hub can retain datatypes inside a pipeline under certain circumstances. This depends on how the types are established and how object is transformed. Type metadata is retained when native types are established by:

-   An input schema
-   An instance
-   A Model Stage 

### How Types Can Be Lost (Transform Patterns)

A Transform does not necessarily remove type information. If the same _event.value_ object is modified, datatypes are retained. However, if the _event.value_ object is completely replaced, datat type information is lost. So a transform stage can modify the value of a property, add new properties or delete them. But when new properties are added, there isn't a way to explicitly set the type for that property directly in the transform stage.

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
