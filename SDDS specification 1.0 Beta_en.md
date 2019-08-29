# Sdds Specification 1.0 beta
## Sdds Specification1.0 beta
    
Copyright： Sdds project 2018
    
License： GNU GPL
    
### Summary：
    
#### What is sdds？
    
Sdds is a short word of "stream data dynamic structure". sdds is a dsl(domain-specific language). Sdds is used to describe the structure of the binary stream data in a programmer-readable and program-understandable manner, thereby enabling reading and writing of the binary stream data with the simple program. This article describes the structure of sdds and the features that the sdds engine program needs to support.
    
#### Why sdds？
    
In data stream-based communication, or any other processing of binary data streams, we do not have an efficient way to read and write data. Usually, we have to write different programs to read and write the different binary data streams.  
    
The goal of sdds is to use a unified encoding and decoding engine program based sdds to quickly encode and decode binary data streams with minimal partial extension.
    
Sdds is dedicated to the coding and decoding of application layer network communication protocols for binary data streams, and strives to simplify communication development. 
    
The goal of sdds is: simple, easy to use, fast and efficient.
    
###  The format of Sdds document definition:
    
The definition of sdds, wihich we named sdds schema. The sdds schema is the file that encodes and decodes the data stream. These instructions parse the data through the sdds engine. That is, it is a dynamic data structure based on a communication protocol or document data format.
    
The schema definition of sdds uses the json format to facilitate syntax checking with the relevant IDE when writing the schema.
    
Keywords in the sdds schema are completely lowercase, and connections between words are underlined.
   
### Sdds data type
    
#### Basic data type:
    
Sdds currently supports the following basic data types:
    
    (1) Integer type:
       
Int8: integer type, an integer of 1 byte (8 bits) in length.
Int16: integer type, an integer of 2 bytes (16 bits) in length.
Int24: integer type, an integer of 3 bytes (24 bits) in length.
Int32: integer type, an integer of 4 bytes (32 bits) in length.
Int64: integer type, an integer of 8 bytes (64 bits) in length.
    
All of the above types support signed and unsigned. And the default is signed, if unsigned, the unsigned attribute should be specified as true.
    
    (2) Aliases of integer type:
      
Bool: Boolean type saved by int8, which is 1 byte in the byte field and 1 bit in the bit field. The values are 0 and 1.
Char: An alias of type int8.
Byte: An alias of type int8.
Short: An alias of type int16.
Word: An alias of type int16.
Dword: An alias for the int32 type.
Int: An alias for the int32 type.
Long: An alias of type int64.
    
   (3) Float type:
    
Float32: The number of points, the number of points in length of 4 bytes (32 bits).
Float64: The number of points, 8 bytes (64 bits), the number of points.

    (4) Aliases of float type :
   
Single: An alias for the float32 type.
Double: An alias for type float64.

    (5) Other basic data types:
   
Bytes: byte type, read the byte of the specified length byte.
Bit: bit type.
Bits: Reads the bits of the specified length.
String: Reads the specified length, specifying the string of the charset.
Bcd8421: bcd code based on the 8421 algorithm.
Hex: hex string type
    
Note: All data types have an endianness distinction(Big-endian, Little-endian). Can be specified through the options node in Sdds schema.
     
The sdds engine not only supports read, but also write, insert, and replace.
    
In the sdds node, the data types are specified by the following two attributes: data_type and unsinged.
    
Endianness is specified in the options node. If there are individual nodes that are inconsistent with the actual endianness, you need to implement them through a custom function.
    
#### Data type extension
    
In addition to the basic data types described above, the sdds engine supports the extending for unsupported data types. The extensions are: custom types combined by basic data types. Cf the attributes byte_fields and bit_fields.
    
It is also possible to read the bytes by the basic type bytes, and then specify the extension function implementation through attributes such as format, formula, and after_action.
   
### Node structure of Sdds schema:
   
#### Schema structure
  
Note: Any sdds schema includes the 3 branch nodes: the format is as follows:
```
{
"meta"：{},
"options"：{},
"sdds"：{}
}
```
among them:
Meta: Metadata used to describe information about this sdds schema. The sdds engine does not process this node.
Options: global program run parameters configured for the sdds engine
Sdds: sdds schema body.

#### Meta node structure
     
Since the meta node is just an information node, the program does not use it to process the data, so the fields of the meta node can be increased or decreased at will. Therefore, the following are only recommended keywords.
    
```
    "meta":{
        "comment": "comment, program comment",
        "version": "version",
        "author": "author",
        "copyright": "copyright information",
        "summary": "summary information",
        "key_words": "keywords",
        "license": "authorization information",
        "support": "Technical Support Information",
        "keywords": "keywords",
        "contact": "Contact"
    }
```
    
Of course, the above information can be optional, but if you want to open source, some of the above information is essential.
    
#### Options node structure
    
The basic content of the options node is as follows:
    
```
   "options"：{
        "endianness"：2,
        "min_length"：15,
        "debug"：true,
        "top_node"："message"
   }
```
    
The options configuration node provides global program runtime parameters for the program to encode and decode. The parameters are as follows:
    
endianness: integer, byte order, machine byte order: 0, little endian order: 1, big endian order: 2
    
min_length: integer, the minimum length of the packet. Memory allocation for writing data because node reads and writes are done in memory. This reduces the performance overhead of dynamically allocating memory.
    
debug: bool, specifies whether it is debug state. When this property is turned off, debugging information is no longer output.
    
top_node：string. Specifies from which node the program starts parsing. Usually, if not specified, the program will either be the 'document' under decode or encode, or the 'message' node. Once you find it, start with it.
    '
#### Sdds node structure
    
```
    "sdds": {
        "decode"：{
            "message"{}
        },
        "encode"：{
            "message"{}
        }
    }
```
    
Usually sdds has two branch nodes:
   
Decode:json specifies the node is a branch for decoding; But if it is pure decoding or encoding, that is, there is only a single function, you can not use this branch. 
  
Encode:json, specifies the node is a branch for encoding; but if it is pure decoding or encoding, that is, there is only a single function, you can not use this branch.
  
In the following, we have the default decode and encode branches.
  
In essence, the child nodes in the decode and encode branches are the read and write instructions of SDDS.

In the decode and encode branches, there is usually one node that is the top node. That is, this node is a node that describes the entire packet structure. In the above example, the message is the top node. By default, message or document is the first node that the program starts parsing. That is the largest node. Of course, you can specify it as document, or a node of another name, by using top_node in the options node. It is recommended to use message or document.

#### The structure of the child nodes in the encode and decode branch
    
   (1) Basic data node structure
    
The structure of the basic data node is as follows:
    
```
    "node_name"：{
        "name"："var_name",
        "id"："node_id",
        "type"："int32",
        "unsigned"：false,
        "length"：4,
        "position"：4,
        "value"："",
        "required"：false,
        "default"："",
    }
```
   
The above nodes actually define the correspondence between a certain segment of the data stream and the basic data type. The related attributes are as follows:
    
"node_name"：The name of the node。
    
"name"：The name of the variable, if you want to read, this property is indispensable when you want to write. Name could be the same in the repeating node.
    
"id"：The ID of the node must be unique within the entire schema.
    
"type"：The data type, in addition to the above basic types, can also be any name, but this name must have a corresponding node in the sdds branch, which is a custom type.
    
"unsigned"：false,Whether the variable is an unsigned type, this property is only valid for integer types, and the default is signed, ie "unsigned": false, you can not write.
    
"length"：The length of the data. The unit is byte.
    
"position"：The position where the data stream is read or written. It is the offset in bytes.
    
"value"：The value, when read, is stored in this attribute, and is read first in this lifetime when writing.
    
"required"：Whether it is required. That is, it cannot be empty, and only the transmitted data is verified.
    
"default"：The default value. Used only as the default value when the value is not specified.
    
 (2) A node of a custom type.
    
A node of a custom type is the same as a primitive data type node, except the type is the name of a node in sdds. That is, sdds uses nodes to implement custom types. E.g:
    
```    
"node_name"：{
	"name"："var_name",
	"id"："node_id",
	"type"："message_header",
	"unsigned"：false,
	"length"：4,
	"position"：4,
	"value"："",
	"required"：false,
	"default"："",
}
```
    
Then there must be a node named message_header in the decode and encode branch nodes of sdds or in the sdds.
    
(3) Node with branches:
    
A node with branches, like an SDDS node, with branches. There are only two kind of branches, a byte field branch and a bit field branch.
They are as follows:
    
```    
"node_name"：{
	"name"："var_name",
	"id"："node_id",
	"length"：4,
	"position"：4,
	"byte_fields"：{
	}
}
"node_name"：{
	"name"："var_name",
	"id"："node_id",
	"length"：4,
	"position"：4,
	"bit_fields"：{
	}
}
```
Unlike basic data type nodes, many attributes in a node are not required. At the same time, there must be branch nodes in byte_fields and bit_fields. And the byte_fields branch can be any type of node. The branch in bit_fields can only be the basic data type node.
    
In general, the top-level node must be a node with a byte_fields branch. Because the sdds engine does not actively traverse the branches of sdds in order, but the top-level node specifies the process.
    
For example, the structure of a message node might look like this:
    
```    
"message"：{
	"name"："message",
	"id"："node_id",
	"length"：4,
	"position"：4,
	"byte_fields"：{
		"header"：{},
		"body"：{},
		"footer"：{}
	}
}
```
    
Of course, the structure of the above data packet does not include the start and end delimiters of the data packet. If so, you need to remove it before calling the decode, and for the send, add it after the encoding is complete. If you don't want to write this extra code, you can also write directly in the branch. The structure is as follows:
    
```    
"message"：{
	"name"："message",
	"id"："node_id",
	"length"：4,
	"position"：4,
	"byte_fields"：{
		"stx"：{},
		"header"：{},
		"body"：{},
		"footer"：{}
		"etx"：{},
	}
}
```
(4) Single-selection node
    
A single-selection node is a structure in which a certain node is selected according to a certain condition. The structure of the single-selection node is as follows:
    
```    
"node_name"： {
        "comment"： "",
        "one_of"： {
	          "key"： {
		            "comment"： "",
		            "name"： "message_key",
		            "value"： "#message_id",
		            "format"： "0x%04X",
		            "id"： "message_key"
	          },
	          "list"： {
			    "0x0201"：{},
			    "0x0302"：{}
		  }
}
```
    
As you can see from the above example, a single-select node must have a branch named one_of. At the same time, in the one_of branch, there must be a key and a list branch, and the key indicates that the value of which node is used as the selection condition. At the same time, there is format to format the key. The result of formatting must be a node name in the list branch.
    
In the above example, value is the ID selector, that is, the value from the node value whose ID is the message_id.
    

(5) Repeating nodes
    
A repeating node means that it has multiple child nodes of the same kind. The structure of the repeating node is as follows:
    
```    
"node_name"： {
        "comment"： "",
        "position"： 0,
        "length"： -2,
        "repeat"： true,
        "count"： 4,
        "until"： "",
        "type"： "repeat_node_name",
        "id"： "node_id",
        "name"： "some_name"
} 
```
    
As you can see from the above example, there are no branches in the repeating node, but a custom type name that points to another node. Repeating nodes can tell the program by specifying the repeat attribute to true. At the same time, if you know the number of repeating, you can specify it to the count attribute. If the number is unknown, you can either specify it or you can use the until attribute to point to the custom extension function to determine the repeating condition.
    
(6) Multiple selection nodes
    
A multi-select node is a composite node. The outer layer is a repeating node and the inner layer is a single-select node. Multiple selections are achieved by repetition. Example (omitted)

### Parameter passing
    
There are 2 ways to pass parameters in the sdds schema.
    
(1) Value transfer selector
    
"#" Id selector: The id selector specifies to read the value of the node with some id. Note that the id should be unique within the same sdds file. If not unique, you should use a different selector. Example: "value": "#body_length" means that the value is read from the node with id of body_length. The current node must be the node after body_length. Therefore, there must be another node with the attribute "id": "body_length".
    
"@"： Path selector: A path selector that specifies the value of a node to be read through a path. Note: The path is searched up first, after searching for the same node, then searching down. Therefore, the path can be specified only to the fork node, not necessarily to the root node. If the path is not clear, you can specify it by debugging properties to see the actual path. Example: "value":"&body.message_body.body_length" means that the value is to find and read the value of body_length by path. The current node must be the node after body_length. The advantage of using a path is that the node you want to operate does not need to be specified. You just need to give the path selector directly where you want to operate.
However, if the node to be read is a branch in the repeat node, then the name of the node is repeat. At the same time, even if you specify the id, the id will be repeat because it is repeat. Similarly, the final generated node, the node name is named after the node's index, so the path selector is not available. At this time, you need to get this type of node through the index selector.
    
"$" Index selector The node to be operated needs to be specified as index by selector. For example: "selector": "index", and in another node, it is "value": "$item_length", in this case Specifies to operate the item_length child node in the same index child node. There must be a "selector" in this child node: "index".
    
"*" Function selector The function selector calls a custom function implemented in an extension (inherited class).In any attribute value, if the first character is the above selector character, sdds will call the selector program to read the data. Therefore, if the start of the above selector character, then use the escape. For example: "\#", "\$". Once escaping is used, it needs to be processed in the extension. Sdds does not automatically clear escaping.
    
(2) The attributes of the value passing
    
The numeric pass attribute is the selector attribute. That is, the node is assigned a selector so that it can operate on it later. The selector has two values: "id" and "index". If you use id, you must ensure that this id is unique under the decode or encode branch. For the child nodes of repeat. Since the id cannot be specified, the selector must be specified as index.


### Sdds preserved keywords

Sdds includes the following keywords, the features' description is as follows:

(1) Standard universal keywords:
    
comment: string, comment for use in document reading and program debugging. The program does not process this attribute.
    
(2) Node type keyword:
    
meta:json, the metadata node, the node that provides comments and descriptions for the document, without data parsing.
    
options:json, specifies a configuration node. Sdds reads the configuration data from it.
    
sdds:json, specifies the list of nodes for the data structure.
  
(3) node attribute keyword:

(3.1) Data structure:
    
There is two kinds of branch nodes: byte_fields and bit_fields.
    
byte_fields: json. Specifies the child nodes that the node includes, that is, fields whose data types are child nodes of the basic type that are read and written in bytes.
    
bit_fields: json. Specifies the child nodes that the node includes, that is, fields whose data types are child nodes of the basic type that are read and written by bits.
    
(3.2) keywords od data type attribute 
    
Type: string. The data type, which can be the basic data type above. If the basic data type, a custom name is used, and this name must have a corresponding defined node in sdds.
    
Position: int. The starting position of the byte or bit. If it is the current position to the last digit, fill in the negative number. This property is not required and the program will usually not process it. But it can help us proofread when writing a schema.
    
Length: int. The length of the data. I can't know the length, it can be null, if it is the current position to the last digit, fill in the negative number.
    
Unsigned: bool. Whether it is unsigned. Basic data type, defaults to false. Unsigned to be specified as true.
    
(3.3) properties of value correlation .
    
Value: The value of the variable. This keyword does not generally appear in the schema configuration. However, if the value of stx and etx is fixed, the value will be written. Value can read the values of other nodes through the selector (see the value transfer selector). This attribute is the value written by the program when decoding, and the value is read from the code.
    
Default: The default value of the variable. The value is read from the value when encoding. If the value is empty, the default value is taken.
    
Required: bool The checksum when data is written. An exception is thrown when this value is empty and default is also empty.
    
(3.4) Keyword of variable related attribute :
    
Name: string The name of the variable to be used in the program, or the field in the database. When decoding, the program will not return data without name. That is: name tells the program what we want to read. Usually, the result returned by decoding is the keyValue structure. Key uses name. In the child node whose repeat is true, note that the name is the same. The id will also be repeated.
    
(3.5) Keyword of Program Behavior Attribute:
    
repeat: bool is used to declare whether there is a continuation of multiples for this type of data.
    
one_of:json is used to select a subtype by key.
    
key:json is used for the search condition specified by one_of.
    
list:json list definition for one_of

(3.6) Pre-processing and post-processing attribute keywords:
    
before_action: json, used to indicate the user-defined function to be called before the current node data is processed. The time when the call occurs is: when the function is decoded, after the initialization, before the data is read, at the time of encoding, after the initialization, before the data is written.
    
after_action: json is used to indicate the user-defined function to be called after the current node data is processed. The time when the call occurs is: when the function is decoded, after the initialization, after the data is read, after the encoding, after the initialization, after the data is written.
    
(3.7) Value Passing Keywords:
    
selector: Selector setting, indicating the way the program looks for this node later. The values are:
    
* "id": This node is found by id. At this time, the id attribute of the node cannot be empty, and the ID must be unique in the full schema. Write "#{node_id}" when reading ({} means to be replaced with the actual name)

# "index": This node must be a child of one of the repeats. This node is found by the index of this item and the name of the child. Write "#{node_name}" when reading ({} means to be replaced with the actual name)
    
Id: The id used to obtain the value by this id. Any attribute whose value starts with # is the value of the corresponding id node. At this point: the value of the find_by attribute is: "id"
    
(3.8) Data format processing keywords:
    
format:string data format specification (only used in the definition of the search keyword key used in one_of)
    
formula: string encoding, decoding formula, in the decode branch, that is, the decoding formula, and in the encode branch, it is the decoding formula. The variable in the formula is the Value in the node, and the variable name is "A"
     
(3.9) Debugging keywords:
    
Debug: bool, when true, will interrupt at the current node and print out the current read/write data.
    
Trace: string, can be entered into the properties of the current node that need to be debugged, separated by commas, and output by the program. When the debug in the options is true, it will be output through the web or console. Otherwise, write to the log file.
    
Ignore_errors: Specifies that this node ignores errors. The default is false. When true, the exception will not be thrown. Unless you have to, make as few assignments as possible, because this will cause trouble for debugging.
    
### FAQ:
    
1. Q: Can the sdds engine perform packet validity verification?
    
A: The sdds engine does not verify the validity of the packet. This part must be done outside of the sdds engine. This can be adapted to different communication protocols. However, if the sdds engine is required to read the relevant data structure before verification, the validate node can be defined in the sdds engine for reading.
    
2. Q: The sdds engine has not read data or has not written data. But there are no errors or exceptions.
    
A: This is because the name attribute is missing. Add the name attribute to solve.
    
3. Q: How to implement multiple one_of nodes in repeat?
    
A: As long as the schema is defined as the repeat, the child node can use one_of.
    
4. Q: What is the difference between attribute and property?
  
A: Both attribute and property can be called attributes. However, attribute is all the fields that can be defined in the schema. The property is the property used in the sdds engine. We also need to understand the property, because the program needs to operate on it. This specification states that all properties begin with "_".
    
5, Q: There is no if condition in sdds, how to deal with it?
    
A: The if condition is not needed in sdds. Such a requirement can be implemented by the one_of node structure.
    
6. Q: Can the selector only search for the loaded node before the node? What to do if it is not loaded?
    
A: Yes, the selector can only search for existing nodes. To handle unloaded nodes, do not process the following nodes in the previous node, but read and write the previous nodes in the following nodes.
      
### Index of sdds attributes
   
index of attributes of sdds node
  
| Attribute Name | Data Type | Description |
|---------------|-----------|----------|   
| after_action | string | Define custom functions called after read and write operations, separated by commas |
| before_action | string | Define custom functions called before read and write operations, separated by commas |
| bit_fields | json | Define branch nodes for bit read and write |
| byte_fields | json | Define branch nodes for byte read and write |
| comment | string | A textual comment for the node or schema. |
| count | integer | Defines the number of items that are written repeatedly |
| debug | bool | The options node is used to define whether the schema is in debug state. The node is used to specify whether this node displays debugging information. |
| default | mixed | Define default values ​​to write |
| format | string | used to define the data formatting after getting the value |
| formula | string | is used to define the formula used after decoding reads or before encoding is written. |
| id | string | Define the unique ID of the node for subsequent access |
| ignore_errors | bool | Defines whether this node ignores errors. |
| key | string | Defines the key used in the selection node. |
| length | integer | Defines the length of the current node's value in Stream |
| list | json | Define a list of candidate nodes in the selection node |
| name | string | Define the variable name of the node |
| one_of | json | When there are child nodes in the node one_of attribute, the node is the selection node. |
| position | string | Defines the Offset in Stream when the value is read or written. |
| repeat | bool | Indicates whether the current node is a duplicate node. |
| required | bool | Indicates if the value to be written is required. |
| selector | string | Defines the selector for the current node. That is, it can be accessed by id or index. |
| trace | string | Defines the debug trace for the current node, which can be separated by commas. |
| type | string | Defines the data type of the current node. If it is a custom type, there must be a node with the same name under the decode or encode of sdds. |
| unsigned | string | Defines whether the data of the current node is unsigned. |
| until | string | Defines the repeat condition function for a repeating node. |
| value | mixed | The value of the current node, either read or written. Mixed is a composite type, which may be a different type. |
 
