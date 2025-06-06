# Automating Google Workspace OU Assignment via Entra ID Provisioning

This guide outlines how to automate the assignment of Organizational Units (OUs) for users in Google Workspace by provisioning them from Microsoft Entra ID (formerly Azure AD) using the SCIM protocol.

## Core Mechanism: SCIM Provisioning from Entra ID to Google Workspace

Microsoft Entra ID's provisioning service uses the SCIM (System for Cross-domain Identity Management) protocol to synchronize user attributes, including their Google Workspace OU path, from Entra ID to Google Workspace. This is typically done through the "Google Cloud / G Suite Connector by Microsoft" application available in the Entra ID Enterprise Applications gallery.

## Steps to Automate OU Assignment

### 1. Prerequisites

* **Google Workspace Super Administrator Account:** Required to authorize the connection from Entra ID.
* **Microsoft Entra ID Administrator Account:** With sufficient permissions to manage Enterprise Applications and Provisioning.
* **Existing OUs in Google Workspace:** Your desired OU structure **must already be set up** in the Google Admin Console. Entra ID cannot create OUs during provisioning; it can only place users into existing ones.

### 2. Configure Google Workspace for SCIM Provisioning

1.  Sign in to the Google Admin console: [admin.google.com](https://admin.google.com/).
2.  Navigate to **Security > Access and data control > API Controls**.
3.  Under "App access control," ensure "Trust internal, domain-owned apps" is checked or that the provisioning service has the necessary access.
4.  **Important:** You **must** use a Google Workspace Super Admin account to authorize the SCIM connection in the next step.

### 3. Add and Configure the "Google Cloud / G Suite Connector by Microsoft" Application in Entra ID

1.  Sign in to the Microsoft Entra admin center: [entra.microsoft.com](https://entra.microsoft.com/).
2.  Browse to **Identity > Applications > Enterprise applications**.
3.  Click **New application**.
4.  In the gallery, search for and select "Google Cloud / G Suite Connector by Microsoft." Click **Add**.
5.  Once added, navigate to the **Provisioning** tab for this application.
6.  Set **Provisioning Mode** to **Automatic**.
7.  Under **Admin Credentials**, click **Authorize**.
    * This will redirect you to Google. Sign in with a Google Workspace Super Administrator account and grant the necessary permissions for Entra ID to manage users and groups.
8.  Click **Test Connection** to ensure the connection is successful.

### 4. Map Attributes (The Key to OU Assignment)

This is the most crucial step for assigning users to specific OUs.

1.  In the **Provisioning** tab of the Google Cloud / G Suite Connector app in Entra ID, go to the **Mappings** section.
2.  Select **Provision Microsoft Entra users**.
3.  You'll see a list of default attribute mappings.
4.  **To map the Organizational Unit:**
    * **Identify an attribute in Entra ID** that stores the desired Google Workspace OU path for your users. Common choices:
        * **`extensionAttribute` (e.g., `extensionAttribute1`, `extensionAttribute2`, etc.):** These are flexible, customizable attributes in Entra ID (which can be synced from on-premises Active Directory) that you can populate with the Google Workspace OU path (e.g., `/Sales/EMEA`, `/HR/Full-time`).
        * **`department`:** If your Entra ID department names directly match your Google Workspace OU names (e.g., "Sales" in Entra maps to a `/Sales` OU in Google). Be cautious if your OU structure is nested or doesn't directly correspond.
        * **Custom extension attributes:** If you have specific structured data requirements.
    * Click **Add New Mapping**.
    * Set **Source attribute** to the Entra ID attribute you've chosen (e.g., `extensionAttribute1`).
    * Set **Target attribute** to **`organizationalUnits[displayName eq "Primary"].path`**. (The exact target attribute name might vary slightly, but it will be related to `organizationalUnits.path`).
        * **Crucial:** The value you map from Entra ID **must exactly match the full OU path** in Google Workspace, including the leading slash (e.g., `/Sales/North America`). If the specified OU path does not exist in Google Workspace, the provisioning will fail for that user.

### 5. Scope Users for Provisioning

1.  In the **Provisioning** tab, under **Settings**, define the **Scope**.
2.  Choose one of the following:
    * **"Sync only assigned users and groups" (Recommended for testing and controlled rollout).** If you select this, you must then go to the **Users and groups** tab of the Enterprise Application and explicitly assign the relevant users or Entra ID groups that you want to provision.
    * **"Sync all users and groups."**

### 6. Enable Provisioning

1.  In the **Provisioning** tab, change **Provisioning Status** to **On**.
2.  Click **Save**.

## Important Considerations and Best Practices

* **OU Path Format:** The OU path in Google Workspace is **case-sensitive** and **must start with a `/`**. Examples:
    * `/Sales`
    * `/HR/Full-time`
    * `/Students/Grade 5`
* **OU Creation:** Entra ID provisioning **cannot create OUs** in Google Workspace. All necessary OUs must be manually created in the Google Admin Console *before* Entra ID attempts to provision users to them. If an OU path doesn't exist, provisioning for that user will fail.
* **Existing Users:** For users that already exist in Google Workspace before provisioning is enabled, the initial sync will attempt to match them based on their primary email. If the OU attribute is mapped, it will update their OU in Google Workspace. Exercise caution and test with a small group first.
* **Deprovisioning:** When a user is removed from scope in Entra ID or disabled, the SCIM provisioning will typically (depending on your deprovisioning settings) suspend or delete the user in Google Workspace. This usually moves them to the top-level OU or deletes them entirely.
* **Attribute Consistency:** Maintain strict consistency between the OU path values in your chosen Entra ID attribute and your actual Google Workspace OU structure. Any discrepancies will lead to provisioning errors.
* **Testing is Critical:** Always perform thorough testing with a small group of users in a non-production environment (if available) or a dedicated test OU before rolling out to your entire organization.
* **Monitoring:** Regularly check the provisioning logs in Entra ID (under the Provisioning tab for the application) for any errors, failures, or warnings. These logs are invaluable for troubleshooting.

By following these steps and considering the best practices, you can effectively automate the assignment of users to their correct Organizational Units in Google Workspace, streamlining user management and ensuring proper policy application based on their departmental or functional grouping.
