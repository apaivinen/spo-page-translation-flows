# SharePoint Online Page Translation Flows

Automatic SharePoint online page translation feature which allows users to create page translations from Finnish/English/Swedish to Finnish/English/Swedish depending on site language settings.

Automation is built by using Power Automate Flow and [Microsoft Translator](https://learn.microsoft.com/en-us/connectors/translatorv2).  (Translation limit with free is 55 000 characters per day).

##  Usage

1. Create a new SharePoint Page.
2. Add your content to the page.
3. Save the page as draft (or publish the page).
4. From the page top suite link bar select "Translation"
5. Select Create on the language you want to translate to. You can also select "Create for all languages".

![Picture 1. Usage](./img/usage.png)

6. Automation will start and notify you via teams message when translation is done (3000 characters takes around 1 minute to process)
7. Go to translated page, review it and publish it.

## Installation

Requirements:
- App registration for power automate flow
- Power automate account with Power Automate Premium license
- Sites with additional languages enabled (Currently supports finnish, english, swedish)

## Entra ID, App registration

To enable the flow to read and write SharePoint pages, authorization to the Microsoft Graph API is required.
**Required permission:** `Sites.ReadWrite.All` or if you want to be more restrictive select `Sites.Selected`

Navigagte to https://entra.microsoft.com/ -> Applications -> App Registration
1. Create a new app registration
2. Name the app, for example "**SharePoint online page translation flow**"
	-   Select: **Accounts in this organizational directory only (Single tenant)**
	- No need to set Redirect URI
3. Open the app you just created
4. Go to **API Permissions**
5. Remove existing permissions
6. Add a permission
	1. Choose **Microsoft Graph**
	2. Choose **Application permission**
	3. Search for and select `Sites.ReadWrite.All`(or `Sites.Selected`=
	4. Click **Add permissions**
7. Click **Grant admin consent** 
8. Go to Certificates & Secrets
9. Create a client secret
	1. Give it a name, for example **Power Automate Translation Flow**
	2. Select expires date, for example **720 days**
10. Copy the **secret value** and save it for later use
11. Go to **Overview**
12. Copy **Application (client) ID** and save it for later use
13. Copy **Directory (tenant) ID** and save it for later use

You now have an application with the necessary permissions, along with the **Client ID**, **Client Secret**, and **Tenant ID**. This information will be used when importing the Power Automate solution.

## SharePoint Online

### Power Automate Account settings
1. Go to the site where you want to install page translation automation
2. Add your power automate account to site permissions as member
3. Make sure you have added site languages to your site(Site settings --> Site Languages) 
4. Navigate to Site Pages library
5. Go to library settings 
6. Get the library GUID and save it for later use
	- GUID can be found from URL, for example `/_layouts/15/listedit.aspx?List=%7B796ce85d-28cc-4fb0-b9ef-7183ecbc08b7%7D` The guid is in between of `%7B` and `%7D` so in this example it would be `796ce85d-28cc-4fb0-b9ef-7183ecbc08b7`
7. Get the site address for later use

Now you should have sharepoint configurations done for one site and you should have following information saved:
- Site Address
- Site Pages GUID

Repeat the steps for each site you want to enable the page translation feature.

### Application permissions to site if using sites.selected permissions
Get site ID

1. In browsers access URL (Replace YOURTENANT and YOURSITE with your SPO site) `https://YOURTENANT.sharepoint.com/sites/YOURSITE/_api/site/id`
2. Use ID value to create Post request to set site permissions for App (replace YOURSITEID with the id):

Use for example Graph explorer https://developer.microsoft.com/en-us/graph/graph-explorer  

Set read permissions:  
**POST** request to **URI**: `https://graph.microsoft.com/v1.0/sites/YOURSITEID/permissions`  

Body:  
```json
{ 
 "roles": ["read"],
 "grantedToIdentities": [{
    "application": {
      "id": "YOUR APPLICATION ID",  //Target Application’s Client Id
      "displayName": "YOUR APPLICATION DISPLAY NAME"       //Target Application’s Display name
    }
  }]
}
```

Set write permissions:  
**POST** request to **URI**: `https://graph.microsoft.com/v1.0/sites/YOURSITEID/permissions`  

Body:  
```json
{ 
 "roles": ["write"],
 "grantedToIdentities": [{
    "application": {
      "id": "YOUR APPLICATION ID",  //Target Application’s Client Id
      "displayName": "YOUR APPLICATION DISPLAY NAME"       //Target Application’s Display name
    }
  }]
}
```

> [!TIP]
> Repeat the steps if you if you have more than one site.


## Power Automate Solution
1. Navigate to https://make.powerautomate.com/ with your power automate account
2. Select target environment
3. Go to Solutions
4. Import a solution
5. During import authorize connections:
	- **Microsoft Teams**
	- **SharePoint**
	- **Microsoft Translator**
6. During import configure following environment variables
	- **AutomationAccountEmail**
	- **ClientID**
	- **ClientSecret**
	- **TenantID**
    - **InitialSiteID**
    - **InitialSitePagesID**

7. After the import go to **Parent Flow - Start translation** flow and rename flow from **Parent flow - Start translation** to **Parent flow - Start translation - siteName**
8. Modify Child flow - Automatic page translations section
	- Modify **Compose AdaptiveCard Message** from bottom of the flow if needed. This is the message teams card will show
	- Modify **Post card in chat or channel** from bottom of the flow if needed. This contains teams message title: "`SharePoint sivu on käännetty!`"

> [!IMPORTANT] 
> If you have multiple sites
> For each site create a new flow (copy of Parent flow - Start translation) and modify Site Address & GUID to match the site in question.


## Solution overview

### Environmental variables
- **ClientID, ClientSecret, TenantID, Audience**: Used for authenticating with Microsoft Graph API.
- **AI-Information-Text**: The notice text added to each translated page.
- **AutomationAccountEmail**: Email address of the automation which is calling child flow
- **InitialSiteUrl**: The initial site URL where you want connect page translations
- **InitialSitePagesID**: The initial site Pages ID where you want to connect page translations
### Connections
- **SharePoint Online**: For retrieving and updating page content.
- **Microsoft Translator**: For translating text.
- **Microsoft Teams**: For sending Teams adaptive card notifications.