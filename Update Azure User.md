# UPDATE A USER IN AZURE


            MicrosoftEntraID.UpdateUser(varSelContact.'A unique identifer for Microsoft Entra ID',
            {
                
                    displayName:txtDisplayName.Value,
                    givenName:txtFirstName.Value,
                    surname:txtLastName.Value,
                    mobilePhone:txtPhone.Value,
                    jobTitle:txtJobTitle.Value,
                    department:txtDepartment.Value
            });
