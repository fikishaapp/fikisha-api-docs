# Fikisha Vendor API Documentation

Documentation to guide the vendors to integrate to the vendor API.

#### Add Github
Generate a New SSH Key with a Custom Filename
   ssh-keygen -t ed25519 -C "hello@fikisha.app" -f ~/.ssh/fikisha_api_docs

Add the Custom SSH Key to the SSH Agent
   eval "$(ssh-agent -s)"
   ssh-add ~/.ssh/fikisha_api_docs

Copy the Public Key to Your Clipboard   
   pbcopy < ~/.ssh/fikisha_api_docs.pub

Add the SSH Key to Your GitHub Account
   GitHub > profile settings > SSH and GPG keys > New SSH key or Add SSH key.

Test the Connection
   ssh -T git@github.com -i ~/.ssh/fikisha_api_docs 

Create an SSH Config File (if it doesn't exist)
   nano ~/.ssh/config

Add an Alias to the SSH Config File
   Host fikisha-api-docs
   HostName github.com
   User git
   IdentityFile ~/.ssh/fikisha_api_docs
   IdentitiesOnly yes

Test Your SSH Alias   
   ssh -T fikisha-api-docs

Add Local Github Config:
    git config user.name "fikishaapp"
    git config user.email "hello@fikisha.app"

    git config user.name
    git config user.email

Add Remote Repository   
   git remote add fikisha-api-docs fikisha-api-docs:fikishaapp/fikisha-api-docs.git
   git branch -M main
   git push -u fikisha-api-docs main