# Creating-Folder-Doc-and-External-Share-Link-ZohoWorkDrive
A guide on how to create a folder within another folder on Zoho WorkDrive, and to create/merge Writer Documents and store in a WorkDrive folder.

## Core Idea
If you've ever searched for the Zoho WorkDrive API documentation online, you'd realize that there isn't an official one yet. The closest thing to one is this unpublished document (https://writer.zohopublic.com/writer/published/ankqibdb934759c6446bca8899b2f426428fc). While useful, it's still not comprehensive and the explanations are not thorough.

In this document, we will cover the following:
* Creating a folder within another folder on WorkDrive
* Creating/Merging a Writer Doc and storing in a WorkDrive folder
* Generating an External Share Link for the Folder/Document

## Configuration
* Zoho WorkDrive OAuth Connections Scopes Needed:
  * WorkDrive.Files.ALL
* Zoho Writer OAuth Connections Scopes Needed:
  * ZohoWriter.documentEditor.ALL
  * ZohoWriter.merge.ALL 

## Tutorial

### Create Folder/Doc in WorkDrive

#### 1. Create a Folder Within Another Folder
Zoho has made this part easy with a `createFolder` function. Just input the subfolder name, main folder ID (you can get this at the end of the URL on Workdrive), and your WorkDrive Connection name.

```javascript
 response = zoho.workdrive.createFolder("INSERT SUBFOLDER NAME", "INSERT MAIN FOLDER ID", "INSERT_WORKDRIVE_OAUTH_CONNECTION");
```

#### 2. Create a New Document in a Folder
This script creates a new blank Zoho Writer doc within a WorkDrive folder.

```javascript
attributes = Map();
attributes.put("filename","INSERT FILE NAME HERE");
attributes.put("folder_id","INSERT MAIN FOLDER ID");
newDoc = invokeurl
[
	url :"https://zohoapis.com/writer/api/v1/documents"
	type :POST
	parameters:attributes
	connection:"INSERT_WRITER_OAUTH_CONNECTION"
];
```

#### 3. Merge a Writer Template and Store the Created Document in a Folder
This enables you to create a document by merging fields into a Writer Doc template and store in a Workdrive folder.
```javascript
values_map = Map();
values_map.put("INSERT MERGE FIELD NAME","INSERT MERGE FIELD VALUE"); //Add more accordingly
output_settings = Map();
output_settings.put("doc_name","INSERT FILE NAME HERE");
output_settings.put("folder_id","INSERT FOLDER ID");
createDoc = zoho.writer.mergeAndStore("INSERT WRITER DOC TEMPLATE ID",values_map,output_settings,"INSERT_WORKDRIVE_OAUTH_CONNECTION");
info createDoc;
```

### Create External Share Link
To create an external share link, an API call with a header and specific parameters are needed. 
* Header
  * All WorkDrive endpoints are JsonAPI compliant, thus, the framework expects a header. If it is not present, an Error Code '415' will be thrown. To know more, check out https://jsonapi.org/
* Parameters
  * Below are the attribute details.
    * resource_id: file or folder id to which the custom share link has to be created.
    * allow_download: if set as `true`, this setting allows external users who access share link to download the file
    * request_user_data: if set as `true`, external users are required to enter their details (Name, Email and Phone) before accessing the share link. It can be useful to track guest users.
    * link_name: name used to identify the share link.
    * expiration_date: date in which the link will expire
    * role_id: permission of the share link
      * 5 - External share for folder with *EDIT* access
      * 6 - External share for folder with *VIEW* access
      * 7 - External share for folder with *UPLOAD* access
    * password_text: the text you set here will enable password protection for the document
    * input_fields: these are the fields mentioned in the request_user_data attribute 
  * Note:
    * No need to pass attributes like *input_fields, password_text* and *expiration_date* if the share link does not require user data, password protection and expiry date respectively.
    * Mandatory attributes like *resource_id, link_name, request_user_data*, and *allow_download* are required to create a custom share link.

Below is the example script to generate an external share link. Configure the parameters attributes based on your requirements accordingly.

```javascript

//Header
mp = Map();
mp.put("Accept","application/vnd.api+json");

//Parameters Map
externalLinkMap = Map();
externalLinkMap.put("resource_id","INSERT FOLDER OR WRITER DOC ID");
externalLinkMap.put("link_name","INSERT A LINK NAME");
externalLinkMap.put("request_user_data",false);
externalLinkMap.put("allow_download",true);
externalLinkMap.put("role_id","7");
attributesMap = {"attributes":externalLinkMap,"type":"links"};
dataMap = Map();
dataMap.put("data",attributesMap);

//API Call
createExternalLink = invokeurl
[
	url :"https://workdrive.zoho.com/api/v1/links"
	type :POST
	parameters:dataMap.toString()
	headers:mp
	connection:"INSERT_WORKDRIVE_OAUTH_CONNECTION"
];

//Get External Share Link
shareLink = createExternalLink.get("data").get("attributes").get("link");
info shareLink;
```

### DISCLAIMER!
If you are generating an external share link for a new blank doc that you have created via Deluge, chances are you might run into this error:
* {"errors":[{"id":"R012","title":"Resource is not in active status to process this action"}]}

For some reason, when a new blank Writer Doc is created via custom function, it is by default in a **DRAFT** status. Because of that, the generate external share link function cannot execute successfully. In order to overcome this, you would first need to change the document status to **ACTIVE** before generating the share link. Here's how you can do it.

#### Mark Document as Ready
The script below marks the newly created Writer Doc as ready, converting it from **DRAFT** status to **ACTIVE**.
* Note:
	* Make sure that this script is placed above the "Generate External Share Link" script
	* You can get the Writer document ID by getting info on the new Writer document creation function response.
	

```javascript
header = Map();
header.put("Accept","application/vnd.api+json");
data = {"data":{"attributes":{"status":"1"},"type":"files"}};
markAsReady = invokeurl
[
	url :"https://workdrive.zoho.com/api/v1/files/" + INSERT_WRITER_DOC_ID
	type :PATCH
	parameters:data.toString()
	headers:header
	connection:"INSERT_WRITER_OAUTH_CONNECTION"
];
info markAsReady;
```
