---
title: "12 Apex Triggers use cases (or) scenarios and its solutions"
seoTitle: "12 Apex Triggers use cases (or) scenarios and its solutions"
seoDescription: "12 Apex Triggers use cases (or) scenarios and its solutions"
datePublished: Wed May 10 2023 11:01:59 GMT+0000 (Coordinated Universal Time)
cuid: clhhlccua000r09l6fh0p1qgz
slug: 12-apex-triggers-use-cases-or-scenarios-and-its-solutions
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/zwsHjakE_iI/upload/65e3952d188ecab1e17f3b721ac0d1d5.jpeg
tags: salesforce, apex, salesforce-apex, triggers, sfdc

---

All the below triggers support the bulk nature of triggers while respecting the governor limits.

1. **Create a task record upon an opportunity stage change :**
    
    ```java
    trigger CreateTaskUponOppStageChange on Opportunity (after update) {
        set<Id>ProcessOpp = new set<Id>();
        for(Opportunity op : Trigger.New){
            if(op.StageName!=Trigger.oldMap.get(op.Id).StageName){
                ProcessOpp.add(op.Id);
            }
        }
        if(ProcessOpp.size()>0){
            list<Opportunity>opps = [SELECT Id, Name, OwnerId FROM Opportunity WHERE Id IN : ProcessOpp];
            list<Task>InsertTasks = new list<Task>();
            for(Opportunity op : opps){
                Task T = new Task(Subject='From Opportunity (Stage Changed) : '+'['+op.Name+']',ActivityDate=Date.today().addDays(1),WhatId=op.Id,OwnerId=op.OwnerId,Status='Not Started');
                InsertTasks.add(T);
            }
            if(InsertTasks.size()>0){
                Insert InsertTasks;
            }
        }
    }
    ```
    
2. **If the account owner changes, its contacts owner also should change :**
    
    ```java
    trigger IfAccountOwnerChangesItsContactsOwnerShouldChange on Account (after Update) {
        set<Id>ProcessAccs = new set<Id>();
        
        for(Account a : Trigger.New){
            if(a.OwnerId != Trigger.oldMap.get(a.Id).OwnerId){
                ProcessAccs.add(a.Id);
            }
        }
        
        if(ProcessAccs.size()>0){
            list<Account>accs = [SELECT Id,(SELECT Id,OwnerId,Account.OwnerId FROM Contacts) FROM Account WHERE Id IN: ProcessAccs];
            list<Contact>contacts= new list<Contact>();
            for(Account a : accs){
                if(a.Contacts.size()>0){
                    for(Contact c : a.Contacts){
                        c.OwnerId = c.Account.OwnerId;
                        contacts.add(c);
                    }
                } 
            }
            
            if(contacts.size()>0){
                update contacts;
            }
        }
    }
    ```
    
3. **Only the system administrator can delete the case record :**
    
    ```java
    trigger OnlySysAdminCanDeleteCase on Case (before delete) {
        set<Id>ProcessCase = new set<Id>();
        
        for(Case c : Trigger.Old){
            ProcessCase.add(c.Id);
        }
        
        if(ProcessCase.size()>0){
            Id UID = userinfo.getUserId();
            User U = [SELECT Id,Profile.Name FROM User WHERE Id=:UID];
            if(U.Profile.Name != 'System Administrator'){
                Trigger.Old[0].Id.addError('You Cannot delete');
            }
        }
    }
    ```
    
4. **Prevent account deletion if any case associated with it :**
    
    ```java
    trigger PreventAccountDeleteIfCasesAssociatedWithIt on Account (before delete) {
        set<Id>ProcessAccount = new set<Id>();
        for(Account a : Trigger.old){
            ProcessAccount.add(a.Id);
        }
        
        if(ProcessAccount.size()>0){
            list<Account>acc = [SELECT Id,(SELECT Id FROM Cases) FROM Account WHERE Id IN: ProcessAccount];
            for(Account a : acc){
                if(a.Cases.size()>0){
                    //Here added "Trigger.oldMap.get(a.Id)" because addError won't support if loop through list.
                    Trigger.oldMap.get(a.Id).addError('You cannot delete this account beacuse this associated with cases');
                }
            }
        }
    }
    ```
    
5. **Prevent account deletion if it is created 7+ days back :**
    
    ```java
    trigger PreventAccountDeleteIfItIsCreated7DaysBack on Account (before delete) {
        list<Account>ProcessAccs = new list<Account>();
        
        for(Account a : Trigger.old){
            ProcessAccs.add(a);
        }
        
        if(ProcessAccs.size()>0){
            for(Account a : ProcessAccs){
                Integer Diff = (a.Createddate.date()).daysBetween(system.today());
                if(Diff > 7){
                    a.addError('You cannot delete this account because it was created 7 days back');
                }
            }
        }
    }
    ```
    
6. **Update the account field(Total\_Contacts\_\_c) with a count of total contacts associated with it upon contact creation or deletion :**
    
    ```java
    trigger TotalContactsOnAccount on Contact (after insert,after delete) {
        set<Id>ProcessAccounts = new set<Id>();
        if(Trigger.New!=null && Trigger.isInsert){
            for(Contact c : Trigger.New){
                if(c.AccountId!=null){
                    ProcessAccounts.add(c.AccountId);
                }
            }
        }
        
        if(Trigger.Old!=null && Trigger.isDelete){
            for(Contact c : Trigger.Old){
                ProcessAccounts.add(c.AccountId);
            }
        }
        
        if(ProcessAccounts.size()>0){
            list<Account>UpdateAccounts = [SELECT Id,Total_Contacts__c,(SELECT Id FROM Contacts) FROM Account WHERE Id IN : ProcessAccounts];
            for(Account a : UpdateAccounts){
                a.Total_Contacts__c = a.Contacts.size();
            }
            if(UpdateAccounts.size()>0){
                update UpdateAccounts;
            }
        }
    }
    ```
    
7. **Update the account field(Latest\_Opp\_Amount\_\_c) with the latest opportunity amount :**
    
    ```java
    trigger UpdateAccFieldWithLatestOppAmount on Opportunity (after insert) {
        set<Id>ProcessOpp = new set<Id>();
        
        for(Opportunity o : Trigger.New){
            if(o.AccountId != null){
                ProcessOpp.add(o.AccountId);
            }
        }
        
        if(ProcessOpp.size()>0){
            list<Account>accs = [SELECT Id,Latest_Opp_Amount__c,(SELECT Id,Amount FROM Opportunities ORDER BY CreatedDate DESC LIMIT 1) FROM Account WHERE Id IN: ProcessOpp];
            for(Account a : accs){
                if(a.Opportunities.size()>0){
                    a.Latest_Opp_Amount__c = a.Opportunities[0].Amount;
                }
            }
            if(accs.size()>0){
                update accs;
            }
        }
    }
    ```
    
8. **Update the account's annual revenue field with the sum of its opportunities amount :**
    
    ```java
    trigger UpdateAnnualRevenueOnAccountWhenOppInserteOrDeleted on Opportunity (after insert,after delete) {
        set<Id>ProcessOpp = new set<Id>();
        if(Trigger.isInsert){
            for(Opportunity o : Trigger.New){
                if(o.AccountId!=null){
                    ProcessOpp.add(o.AccountId);
                }
            }
        }
        
        if(Trigger.isDelete){
            for(Opportunity o : Trigger.Old){
                if(o.AccountId!=null){
                    ProcessOpp.add(o.AccountId);
                }
            }
        }
        
        if(ProcessOpp.size()>0){
            list<Account>accs = [SELECT Id,AnnualRevenue,(SELECT Id,Amount FROM Opportunities) FROM Account WHERE Id IN: ProcessOpp];
            Decimal Count = 0;
            for(Account a : accs){
                for(Opportunity o : a.Opportunities){
                    Count+=o.Amount;
                }
                a.AnnualRevenue=Count;
            }
            if(accs.size()>0){
                update accs;
            }
        }
    }
    ```
    
9. **Prevent account deletion if it is active :**
    
    ```java
    trigger PreventDeleteIfAccountIsActive on Account (before delete) {
        list<Account>ProcessAccs = new list<Account>();
        
        for(Account a : Trigger.Old){
            ProcessAccs.add(a);
        }
        
        if(ProcessAccs.size()>0){
            for(Account a : ProcessAccs){
                if(a.Active__c=='Yes'){
                    Trigger.oldMap.get(a.Id).addError('Account ['+a.Name+'] is active you cannot delete');
                }
            }
        }
    }
    ```
    
10. **Update the latest account's case number on the account :**
    
    ```java
    trigger UpdateLatestCaseNumberOnAccount on Case (after insert) {
        set<Id>ProcessCase = new set<Id>();
        
        for(Case c : Trigger.New){
            ProcessCase.add(c.AccountId);
        }
        
        if(ProcessCase.size()>0){
            list<Account>accs = [SELECT Id,Description,(SELECT Id,CaseNumber,CreatedDate FROM Cases ORDER BY CreatedDate DESC LIMIT 1) FROM Account WHERE Id IN: ProcessCase];
            for(Account a : accs){
                if(a.Cases.size()>0){
                    a.Description += '\n' + 'Case Number : ' + a.Cases[0].CaseNumber + ', ' + 'Created on : ' + a.Cases[0].CreatedDate;
                }
            }
            if(accs.size()>0){
                update accs;
            }
        }
    }
    ```
    
11. **Prevent duplicate contacts based on contact email :**
    
    ```java
    trigger PreventDuplicateContactsBasedOnContactEmail on Contact (before insert) {
        set<String>ProcessContacts = new set<String>();
        
        for(Contact c : Trigger.New){
            if(c.Email!=null){
                ProcessContacts.add(c.Email);
            }
        }
        
        if(ProcessContacts.size()>0){
            list<Contact>contacts = [SELECT Id, Email FROM Contact WHERE Email IN: ProcessContacts];
            list<String>Emails = new list<String>();
            if(contacts.size()>0){
                for(Contact c : contacts){
                    Emails.add(c.Email);
                }
                for(Contact c : Trigger.New){
                    if(Emails.contains(c.Email)){
                        c.addError('Duplicate contacts are not allowed : '+ c.Email + ' already used');
                    }
                }
            }
        }
    }
    ```
    
12. **Update the opportunity stage name to 'closed lost' if the stage name is 'closed won' upon the account's active\_\_c field changes to 'No' :**
    
    ```java
    trigger UpdateOppsUponChangingAccountActiveFieldToFalse on Account (after update) {
        set<Id>ProcessAccounts = new set<Id>();
        for(Account a : Trigger.New){
            if(a.Active__c!=Trigger.oldMap.get(a.Id).Active__c && a.Active__c=='No'){
                ProcessAccounts.add(a.Id);
            }
        }
        if(ProcessAccounts.size()>0){
            list<Account>accs = [SELECT Id,(SELECT Id,StageName FROM Opportunities) FROM Account WHERE Id IN : ProcessAccounts];
            list<Opportunity>UpdateOpps=new list<Opportunity>();
            for(Account a : accs){
                for(Opportunity o : a.Opportunities){
                    if(o.StageName!='Closed Won'){
                        o.StageName='Closed Lost';
                        UpdateOpps.add(o);
                    }
                }
            }
            if(UpdateOpps.size()>0){
                Update UpdateOpps;
            }
            
        }
    }
    ```
    

You can also find these codes in below GitHub repository :  
[https://github.com/leelasairam/SF-Triggers](https://github.com/leelasairam/SF-Triggers/tree/master/force-app/main/default/triggers)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683794080757/6b38329b-0335-4d1c-87cb-7be3c2339abc.gif align="center")

Thanks for reading😊