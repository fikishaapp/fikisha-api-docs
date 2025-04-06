# Fikisha Vendor API Documentation

Welcome to the official documentation for the Fikisha Vendor API. This documentation provides guides and API references to help you integrate your systems with Fikisha to programmatically sell airtime, data bundles, and manage your vendor account.

## Getting Started

-   **[Introduction](/introduction)**: Overview of the API capabilities.
-   **[Quickstart Guide](/quickstart)**: Make your first API call quickly.
-   **[Authentication](/authentication)**: Learn how to authenticate your requests.

## API Reference

Explore the detailed documentation for each API endpoint:

-   **[Airtime](/api-reference/airtime/request-airtime)**
-   **[Data Bundles](/api-reference/data/list-bundles)**
-   **[Wallet](/api-reference/wallet/get-balance)**
-   **[Sale Requests](/api-reference/sales/initiate-sale)**

## Guides

-   **[Error Handling](/guides/error-handling)**: Understand error codes and responses.
-   **[Webhooks](/guides/webhooks)**: Receive real-time updates on asynchronous operations.
-   **[Testing](/guides/testing)**: Information on testing your integration.

## Need Help?

-   Visit the [Fikisha Vendor Dashboard](https://vendor.fikisha.com) to manage your account and API keys.
-   Contact support at [support@fikisha.app](mailto:support@fikisha.app).

---

### Local Development (Using Mintlify CLI)

1.  **Install Mintlify CLI:**
    ```bash
    npm install -g mintlify
    ```
2.  **Run the development server:**
    ```bash
    mintlify dev
    ```
    This will start a local server, typically at `http://localhost:3000`.

### Deployment

Mintlify handles deployment automatically when connected to your Git repository (GitHub, GitLab, Bitbucket). Push changes to your main branch to deploy.

---
*SSH Key Setup instructions (as previously provided - keep this if needed for repo access)*

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

Add Remote Repository
   git remote add fikisha-api-docs fikisha-api-docs:fikishaapp/fikisha-api-docs.git
   git branch -M main
   git push -u fikisha-api-docs main