# Get Users




# POWERAPPS-- Connector: Office365Groups
Connector: Office365Groups
App: PowerApps

Set(
    varUsers,
    Office365Groups.HttpRequest(
        "https://graph.microsoft.com/v1.0/users?$filter=userType eq 'Member'",
        "GET",
        ""
    )
);

Transform JSON
    ClearCollect(
    colUsers,

        
            ForAll(Table(varGroups_b1.value),  
            {
                displayName: Text(ThisRecord.Value.displayName),
                id: Text(ThisRecord.Value.id)
            }
            ))



    Loads the first 100



# POWER AUTOMATE-- Connector: Search for users (V2)

Leave the all parameters blank
The returns all users
