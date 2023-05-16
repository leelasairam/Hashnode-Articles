---
title: "How to open Salesforce org with one click?"
seoTitle: "How to open Salesforce org with one click"
seoDescription: "How to open Salesforce org with single click"
datePublished: Thu Mar 30 2023 15:14:48 GMT+0000 (Coordinated Universal Time)
cuid: clfv9bkjs000109mj6u6uggpg
slug: how-to-open-salesforce-org-with-one-click
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/n8Qb1ZAkK88/upload/0c1f74ebfc024761623367716539acd2.jpeg
tags: salesforce, salesforce-development, salesforce-crm, sfdx

---

Opening a Salesforce org requires multiple clicks but there is a way to open and login into the org with a single click.

It requires Salesforce CLI installed on the local machine. If you don't have one, can install it from this [link](https://developer.salesforce.com/tools/sfdxcli).

Once you had Salesforce CLI on your local machine, Follow below steps :

1. Open Command Prompt and run the below code:
    
    `sfdx auth:web:login -r` [`https://login.salesforce.com`](https://login.salesforce.com) `-a <alias>`
    
    Here **&lt;alias&gt;** is a nickname or simple name of your org. It can be anything. For example,
    
    `sfdx auth:web:login -r`[`https://login.salesforce.com`](https://login.salesforce.com) `-a creativefox`
    
    If you want to open sandbox or scratch org, then replace the command as below :
    
    `sfdx auth:web:login -r` [`https://test.salesforce.com`](https://login.salesforce.com) `-a <alias>`
    
2. Once you run the command, it will open the login page there you need to login with your org credentials.
    
3. Now open the **Notepad** and enter the below command:
    
    `sfdx force:org:open -u <alias>`
    
    replace the **&lt;alias&gt;** with the name you have given previously. It may look like this `sfdx force:org:open -u creativefox` .
    
4. Save the file with any name but replace the `.txt` with `.bat` (batch file). Then create a shortcut and place it on the Desktop. That's it.
    
5. Now, you need to just double-click the shortcut. It will automatically log you in and opens the org. It works even after you shut down your machine.
    

Thanks for reading.