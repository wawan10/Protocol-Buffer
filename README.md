Protocol buffers are Google's language-neutral, platform-neutral, extensible mechanism for serializing structured data – think XML, but smaller, faster, and simpler. You define how you want your data to be structured once, then you can use special generated source code to easily write and read your structured data to and from a variety of data streams and using a variety of languages – Java, C++, or Python.

The Protocol Buffers serialised format is a binary encoded format that is not easily human readable. As Protocol Buffers messages are commonly exchanged over HTTP we have added full support for viewing and editing Protocol Buffers messages in a human readable way.

Charles currently supports version 2.4.1 of Protocol Buffers, which is largely backwards compatible with earlier versions.

OVERVIEW

Charles identifies that an HTTP request or response contains a Protocol Buffers message when the Content-Type header has a MIME type ofapplication/x-protobuf or application/x-google-protobuf. Two new HTTP body content viewers become available when viewing the content, the Protobuf Text Viewer and the Protobuf Structured Viewer.

In order for these viewers to be able to display the message content they need access to the protocol buffers descriptor for the message(s) that are contained in the HTTP body content. Charles looks for desc and messageType parameters in the content-type to discover the location of the FileDescriptorSet (*.desc file) and fully qualified message type name, it then uses these to retrieve and load the appropriate descriptor for the message(s). A protocol buffers FileDescriptorSet can be generated from a *.proto file by the protocol buffers compiler (protoc) by using the -o or --descriptor_set_out option e.g. protoc -oModel.desc Model.proto.

Finally the HTTP body content may contain a single message or a list of messages which have been serialised using the standard protocol buffers length delimited format. To determine whether the HTTP body contains a single message or a delimited list of messages an optionaldelimited parameter in the content-type is looked for. This must be present and have the value true to indicate a delimited list of messages has been sent. When a delimited list of messages has been sent all messages must be of the same message type.

This means a complete Content-Type header will look like:Content-Type: application/x-protobuf; desc="http://localhost/Model.desc"; messageType="com.xk72.sample.PurchaseOrder"; delimited=true

LOADING THE FILEDESCRIPTORSET

Charles expects the desc parameter of the Content-Type header to contain a valid URL pointing to the location of the FileDescriptorSet for the protocol buffer message. This can be any valid URL but is usually a file or http URL. Charles attempts to retrieve the FileDescriptorSet every time one one of the Protobuf viewers is selected to display the HTTP body content, this means that the desc URLs are not resolved until needed which is critical when large numbers of protocol buffer transactions are being recorded by Charles.

FILEDESCRIPTORSET CACHING

Typically you still get many transactions using the same protocol buffer message types, so Charles does implement caching of the FileDescriptorSets based upon standard HTTP 1.1 caching for http URLs and the last modified timestamp for file URLs.

This means you can ensure the performance of the FileDescriptorSet loading by ensuring the web server which is serving up the files is setting appropriate HTTP 1.1 caching and/or validation (Last-Modified and ETag) headers. Caching can also be effectively disabled by setting the appropriate HTTP 1.1 no-cache headers.

When no HTTP caching or validation headers are present on the FileDescriptorSet responses then Charles calculates its own heuristic expiration time for cached FileDescriptorSets. The length of the time Charles will cache these resources for is configurable in the Preferences->Viewersdialog in Charles, it defaults to 5 minutes. This can be set to 0 to disable the heuristic caching.

Please note this caching is implemented for the resolving of the desc URLs only, it does not in any way change the behaviour of Charles with respect to the HTTP transactions being recorded.

PROTOBUF TEXT VIEWER

The Protobuf Text body content viewer displays the default textual representation of the protocol buffer message. This is the same format implemented by the com.google.protobuf.TextFormat class in the Java API, the google.protobuf.text_format.h C++ API or thegoogle.protobuf.text_format Python module that come with the Protocol Buffers distribution.

When displaying a delimited list of messages each message is separated by the textual delimiter:>--------------------------------next message--------------------------------<

PROTOBUF VIEWER

Basic Message Display
The Protobuf body content viewer shows a tree-structured view of the protocol buffer message, showing the full hierarchical structure of the message with all fields and sub-messages.

In the structured viewer the first column displays the defined field name for each field in the tree-structure defined by the Protocol Buffer message definition.

The second column displays the type of the field:

Scalar fields are given their .proto type name e.g. string, uint32, bool, sfixed64 etc
Enumerations are labeled with the fully qualified type name of the enumeration prefixed with Enum
Message typed fields are given the fully qualified type name of the message
Any repeating field is prefixed with Repeating e.g. Repeating string
The third column displays information about the value of the field:

Scalar fields have a textual representation of their value displayed
Enumerations have the defined name for their value displayed
Repeating fields display the number of repeats that exist for the field
Message typed and repeating fields can be expanded to display the individual fields or repeats they encapsulate. Note that for string fields what is being displayed is not the literal string value but an encoding of it to support representing non-printable characters. The encoding used is the format used for string literals in C code, so that newlines become \n, tabs become \t and so on.

Any field may be double-clicked to open a pop-up dialog containing a string representation of the value of that field. This is useful when the field content is too large to be easily displayed in the table. Also when a message typed field is double-clicked the entire content of the message contained in that field including all its child fields is displayed.

Handling of Missing Optional Fields
With protocol buffer messages fields that are defined in the message definition as optional do not have to be included in the message. In this situation where the field does not explicitly exist in the message it still has an implied value. This will be the default value if one is defined for the field in the message definition or else it gets the normal default value for its type e.g. 0 for the number types, false for bool types and the empty string for string types.

Usually these unspecified fields are hidden as it is presumed they are of little interest. This includes hiding repeating fields that have 0 repeats. You can configure the viewer to display these unspecified fields in the Preferences->Viewers dialog in Charles.

When unspecified fields are being displayed their field name, type information and implied or default value show up in italics in the viewer.

Unknown Fields
With protocol buffers messages it is possible to get unknown fields in the message content, this typically happens when the message definition has changed and an older message that contains fields that no longer exist in the message definition is being parsed.

When this happens the structured viewer displays an Unknown Fields child node in the message hierarchy which can be expanded to see the available information pertaining to the unknown fields. Because the meta-data that is normally found for these fields in the message definition is not available all you can see is the field's numeric tag, serialised wire type and serialised value. The possible wire types are:

Varint
32-bit
64-bit
Length-delimited
see the protocol buffers encoding documentation for information on the mapping from .proto scalar types to these wire types and how to decode the serialised values.

UNKNOWN MESSAGES

There are some situations where the protocol buffers descriptor for the message is not available. For example this could happen when thedesc or messageType parameters were not specified in the Content-Type header or there was an error resolving the URL provided in thedesc parameter.

When this happens the protocol buffers message is parsed as an Unknown message, in effect this means that all the fields in the message become unknown fields and are displayed with just the minimal information we can discover about them from the wire format. The error that caused the message to be parsed as an Unknown message is also displayed in the viewer, hopefully allowing you to fix whatever caused the problem.

CONFIGURATION OPTIONS

There are some aspects of the Protocol Buffers functionality that is user configurable. These options are available in the Viewers section of thePreferences dialog. The Preferences dialog is activated from the Edit menu (or Charles menu on Mac OS X).

Hide Unspecified Fields - when selected optional fields that have not been specified in a protocol buffer message are hidden when that message is displayed in the structured viewer, when not selected the unspecified fields are displayed in italics along with the default or implied value that they take
Cache Protobuf Descriptors - enables the caching of protocol buffers descriptors as described in the caching section, when not selected descriptors will not be cached
Heuristic Cache TTL - when caching is enabled this is the cache TTL or expiry time used for descriptors retrieved from an HTTP URL where no HTTP caching headers are specified, set to 0 to never cache when there are no explicit HTTP caching headers
Clear Cached Resources - clicking this button immediately clears all cached protocol buffers descriptors
COMPARING MESSAGES

When you have exactly two transactions selected in either the structure or sequence views you can compare their content by using the Comparecommand from the right-click context menu.

When both of the transactions contain protocol buffer messages as well as the standard comparison viewers normally available you have the option of using the Protobuf Text Viewer or Protobuf Structured Viewer to view the comparison between the transactions.

EDITING MESSAGES

The Protobuf Text Viewer and Protobuf Structured Viewer may also be used when editing the content of an HTTP request. When you choose to edit an HTTP request if it is identified as a protocol buffers request (by having a content type of application/x-protobuf orapplication/x-google-protobuf) then the two protobuf specific viewers will be available as options for editing the request content.

Protobuf Text Viewer - Edit mode
The Protobuf Text viewer in edit mode simply allows you to edit the human readable textual representation of the protocol buffers message. This text is both generated and then reparsed (after editing) into a serialised protocol buffers message using the standard Protocol Buffers library API, that is the com.google.protobuf.TextFormat class in the Java API, the google.protobuf.text_format.h C++ API or thegoogle.protobuf.text_format Python module.

One quirk of the implementation you may need to be aware of is that if the original message contained unknown fields these are represented in the text generated by the protocol buffers text formatter but they cause an error (*Message type XXX has no field named YYY*) when the text is attempted to be reparsed. This means they need to be manually deleted from the text when you are editing.

When editing a delimited list of messages the editor expects the textual delimiter>--------------------------------next message--------------------------------< between each message. You can add or delete instances of this delimiter to add or remove messages from the list.

Protobuf Viewer - Edit mode
The Protobuf viewer in edit mode shows the current message content in the same structured view described in the Protobuf Structured Viewersection. The one key difference is that all the defined fields for a message are always shown, regardless of whether they are currently populated. The fields which are defined for a message but are not currently populated are shown in italics. This makes it very easy to see what fields may exist for a message and to set values for those fields.

The value of any scalar field may be set by simply double-clicking the value column (double-clicking elsewhere on the row will instead open a dialog containing the field value, a feature useful for viewing very large values). Scalar fields can be cleared (turned into a field that is not populated in the message) by right-clicking on the field and selecting Clear value.

Message valued fields can similarly be cleared by right-clicking on the field and selecting Clear entire message. Message valued fields that are not populated are treated very similarly to scalar fields, they are shown in italics and populated with their default or implied values. As soon as any child field of an unpopulated message is explicitly set then that message will become a concrete, explicitly-defined value.

Repeating fields may have new repeats added to them by right-clicking on the field and selecting Add repeat or by using the Insert before orInsert after commands when you right-click on an existing repeat. There is also Delete repeat option.