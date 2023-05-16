---
title: "How to retrieve specific LWC code from org"
seoTitle: "How to retrieve specific or particular LWC code from org"
seoDescription: "How to pull specific or particular LWC code from org"
datePublished: Wed Mar 01 2023 10:02:14 GMT+0000 (Coordinated Universal Time)
cuid: clepidvuv00040amm7squ2ckc
slug: how-to-retrieve-specific-lwc-code-from-org
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/szrJ3wjzOMg/upload/66223f619ae108dcdd86580b6c9f4c60.jpeg
tags: javascript, salesforce, lwc

---

Sometimes we may have to work on the LWC which was deployed by some other. In such cases, we may not have the code on our local machine.

To pull the component's code from the org, we can use the below command

```plaintext
sfdx force:source:retrieve -m LightningComponentBundle:[cmpName]
```

<mark>Note :</mark>

* Make sure **Salesforce CLI** is installed on your machine.
    
* Make sure after creating the project (`SFDX: Create Project`), authorize the Salesforce Org. To authorize, run `SFDX: Authorize an Org` command in the VSCode terminal.