# GET USERS
Connector: Office 365 Users
Action: Send an HTTP request

Loads first 100 batch


#Get All Members Only


# Connector: Variables
{
  "type": "InitializeVariable",
  "inputs": {
    "variables": [
      {
        "name": "GroupArray",
        "type": "array",
        "value": []
      }
    ]
  },
  "runAfter": {},
  "metadata": {
    "operationMetadataId": "7bb91dc8-7588-4a17-a63b-8d14aa9249e1"
  }
}

# Connector: Office 365 Users - Action: Send an HTTP request
{
  "type": "OpenApiConnection",
  "inputs": {
    "parameters": {
      "Uri": "https://graph.microsoft.com/v1.0/users?$filter=userType eq 'Member'",
      "Method": "GET",
      "ContentType": "application/json"
    },
    "host": {
      "apiId": "/providers/Microsoft.PowerApps/apis/shared_office365users",
      "connection": "shared_office365users",
      "operationId": "HttpRequest"
    }
  },
  "runAfter": {
    "Initialize_variable-ContactsArray": [
      "Succeeded"
    ]
  }
}

# Appends value to array variable.


{
  "type": "AppendToArrayVariable",
  "inputs": {
    "name": "GroupArray",
    "value": "@join(body('GetUsers')?['value'],',')"
  },
  "runAfter": {
    "GetUsers": [
      "Succeeded"
    ]
  },
  "metadata": {
    "operationMetadataId": "1c105c96-a761-4e5f-8c44-b4b60951373a"
  }
}

# Appends value to array variable - Get next link
{
  "type": "InitializeVariable",
  "inputs": {
    "variables": [
      {
        "name": "varNextLink",
        "type": "string",
        "value": "@body('GetUsers')?['@odata.nextLink']"
      }
    ]
  },
  "runAfter": {
    "Append_to_array_variable-1": [
      "Succeeded"
    ]
  },
  "metadata": {
    "operationMetadataId": "c84f637e-2666-4a6a-9254-2122f4843015"
  }
}


# Do Until loop

{
  "type": "Until",
  "expression": "@equals(variables('varNextLink'),'')",
  "limit": {
    "timeout": "PT1H"
  },
  "actions": {
    "Append_to_array_variable-2": {
      "type": "AppendToArrayVariable",
      "inputs": {
        "name": "GroupArray",
        "value": "@join(body('GetUsersLoop')?['value'],',')"
      },
      "runAfter": {
        "GetUsersLoop": [
          "Succeeded"
        ]
      },
      "metadata": {
        "operationMetadataId": "6899eef5-b499-4f41-99c1-4a06c75cf6cd"
      }
    },
    "Set_variable": {
      "type": "SetVariable",
      "inputs": {
        "name": "varNextLink",
        "value": "@body('GetUsersLoop')?['@odata.nextLink']"
      },
      "runAfter": {
        "Append_to_array_variable-2": [
          "Succeeded"
        ]
      },
      "metadata": {
        "operationMetadataId": "f13b5224-45aa-4688-98cf-b3253d1d7e6d"
      }
    },
    "GetUsersLoop": {
      "type": "OpenApiConnection",
      "inputs": {
        "parameters": {
          "Uri": "@variables('varNextLink')",
          "Method": "GET",
          "ContentType": "application/json"
        },
        "host": {
          "apiId": "/providers/Microsoft.PowerApps/apis/shared_office365users",
          "connection": "shared_office365users",
          "operationId": "HttpRequest"
        }
      }
    }
  },
  "runAfter": {
    "Initialize_variable-NextLinkAvailable": [
      "Succeeded"
    ]
  },
  "metadata": {
    "operationMetadataId": "a2d01241-2372-45bf-8172-36ff80f3d5ec"
  }
}


# Compose-Array to Object

{
  "type": "Compose",
  "description": "Do this to parse the JSON Response of the 'Append to variable' GroupArray",
  "inputs": "@join(variables('GroupArray'),',')",
  "runAfter": {
    "Do_until": [
      "Succeeded"
    ]
  }
}

# Compose-Object to Array

{
  "type": "Compose",
  "description": "Do this to parse convert the objects into an array by surrounding with []",
  "inputs": "@json(concat('[', outputs('Compose-Array_to_Object'), ']'))\r\n",
  "runAfter": {
    "Compose-Array_to_Object": [
      "Succeeded"
    ]
  }
}

# Select - Choose Columns

{
  "type": "Select",
  "inputs": {
    "from": "@json(concat('[', outputs('Compose-Array_to_Object'), ']'))\n",
    "select": {
      "id": "@item()?['id']",
      "displayName": "@item()?['displayName']",
      "createdDateTime": "@item()?['createdDateTime']",
      "description": "@item()?['description']",
      "mail": "@item()?['mail']",
      "mailEnabled": "@item()?['mailEnabled']",
      "securityEnabled": "@item()?['securityEnabled']",
      "mailNickname": "@item()?['mailNickname']",
      "groupTypes": "@item()?['groupTypes']"
    }
  },
  "runAfter": {
    "Compose-Object_to_Array": [
      "Succeeded"
    ]
  }
}

# Compose - Count Rows

{
  "type": "Compose",
  "inputs": "@length(body('Select'))",
  "runAfter": {
    "Select": [
      "Succeeded"
    ]
  }
}

# Respond to a Power App or flow

{
  "type": "Response",
  "kind": "PowerApp",
  "inputs": {
    "schema": {
      "type": "object",
      "properties": {
        "resp": {
          "title": "resp",
          "x-ms-dynamically-added": true,
          "type": "string"
        }
      },
      "additionalProperties": {}
    },
    "statusCode": 200,
    "body": {
      "resp": "@{body('Select')}"
    }
  },
  "runAfter": {
    "Compose_1": [
      "Succeeded"
    ]
  },
  "metadata": {
    "operationMetadataId": "f3da1a38-262a-413f-bb7b-3537a56124b9"
  }
}

![image](https://github.com/user-attachments/assets/5454457f-e147-48fc-8380-d7063fad30df)

