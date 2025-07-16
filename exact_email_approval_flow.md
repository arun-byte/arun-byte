# Exact Email Format Approval Flow - Personalized Approver Notifications

## 🎯 **Email Template You Want:**
```
Hello "Approver Name"

A Case Number 00001221 has been assigned for Approval by Meenakshi Meenu with Comment "approveddd"

Please click on the link to view the record : Approval url Link https://urldefense.com/v3/__https://vetrotech--devgdi.sandbox.my.salesforce.com/p/process/ProcessInstanceWorkitemWizardStageManager?id=04iFT000000qTAn__;!!Mj5yV84exmbMMB4!pNr6e4se5TYKmOaR9om5jk2d-8wNuyhp2xYLfZ1hEhV5hz5JiOt5HE856gWUsKhGTHm9j_Ky_4cujw5VaV6zLypp-aVD39ZxEY0$

Thanks & Regards
CS Team
```

## 🚀 **Complete Working Flow**

### **Flow Configuration:**
```
Flow Type: Record-Triggered Flow
Object: Case
Trigger: A record is created or updated
Entry Criteria: ApprovalStatus__c = "Submitted For Approval"
Run Asynchronously: ✅ (This fixes the timing issue!)
```

### **Flow Elements:**

### **Element 1: Get Approval Process**
```
Element: Get Records
Label: Get Current Approval Process
Object: ProcessInstance
Conditions:
- TargetObjectId Equals {!$Record.Id}
- Status Equals Started
Sort: CreatedDate Descending
Store: Only the first record
```

### **Element 2: Get Approval Comments**
```
Element: Get Records
Label: Get Approval Comments
Object: ProcessInstanceHistory
Conditions:
- ProcessInstanceId Equals {!Get_Current_Approval_Process.Id}
- StepStatus Equals Started
Sort: CreatedDate Descending
Store: Only the first record
```

### **Element 3: Get Current Work Items**
```
Element: Get Records
Label: Get Current Work Items
Object: ProcessInstanceWorkitem
Conditions:
- ProcessInstanceId Equals {!Get_Current_Approval_Process.Id}
Store: All records
```

### **Element 4: Loop Through Work Items**
```
Element: Loop
Label: Loop Through Approvers
Collection: {!Get_Current_Work_Items}
```

### **Element 5: Get Approver Details (Inside Loop)**
```
Element: Get Records
Label: Get Approver Details
Object: User
Conditions:
- Id Equals {!Loop_Through_Approvers.ActorId}
Store: Only the first record
```

### **Element 6: Send Personalized Email (Inside Loop)**
```
Element: Send Email
Label: Send Approval Email
To Addresses: {!Get_Approver_Details.Email}
Subject: Approval Required - Case {!$Record.CaseNumber}

Body:
Hello {!Get_Approver_Details.Name}

A Case Number {!$Record.CaseNumber} has been assigned for Approval by {!Get_Current_Approval_Process.SubmittedBy.Name} with Comment "{!Get_Approval_Comments.Comments}"

Please click on the link to view the record : Approval url Link https://vetrotech--devgdi.sandbox.my.salesforce.com/p/process/ProcessInstanceWorkitemWizardStageManager?id={!Loop_Through_Approvers.Id}

Thanks & Regards
CS Team
```

## 📋 **Step-by-Step Implementation**

### **Step 1: Configure Flow for Async Execution**
```
1. Edit your existing flow
2. Start element → Configure Trigger
3. ✅ Check "Run Asynchronously"
4. Optimize for: Actions and Related Records
```

### **Step 2: Update Your Email Element**

**Replace your email body with this exact format:**

```
Hello {!Get_Approver_Details.Name}

A Case Number {!$Record.CaseNumber} has been assigned for Approval by {!$Record.CreatedBy.Name} with Comment "{!Get_Approval_Comments.Comments}"

Please click on the link to view the record : Approval url Link https://vetrotech--devgdi.sandbox.my.salesforce.com/p/process/ProcessInstanceWorkitemWizardStageManager?id={!Loop_Through_Approvers.Id}

Thanks & Regards
CS Team
```

### **Step 3: Handle Missing Comments**

**If comments might be empty, use this enhanced version:**

```
Hello {!Get_Approver_Details.Name}

A Case Number {!$Record.CaseNumber} has been assigned for Approval by {!$Record.CreatedBy.Name}{!IF(LEN(Get_Approval_Comments.Comments) > 0, " with Comment \"" & Get_Approval_Comments.Comments & "\"", "")}

Please click on the link to view the record : Approval url Link https://vetrotech--devgdi.sandbox.my.salesforce.com/p/process/ProcessInstanceWorkitemWizardStageManager?id={!Loop_Through_Approvers.Id}

Thanks & Regards
CS Team
```

## 🔧 **Advanced Email Template (HTML Version)**

**For better formatting:**

```html
<div style="font-family: Arial, sans-serif; font-size: 14px;">
    <p>Hello {!Get_Approver_Details.Name}</p>
    
    <p>A Case Number <strong>{!$Record.CaseNumber}</strong> has been assigned for Approval by <strong>{!$Record.CreatedBy.Name}</strong> with Comment "<em>{!Get_Approval_Comments.Comments}</em>"</p>
    
    <p>Please click on the link to view the record: 
    <a href="https://vetrotech--devgdi.sandbox.my.salesforce.com/p/process/ProcessInstanceWorkitemWizardStageManager?id={!Loop_Through_Approvers.Id}">Approval url Link</a></p>
    
    <p>Thanks &amp; Regards<br/>
    CS Team</p>
</div>
```

## 🎯 **Variables You'll Need**

**Create these variables if needed:**

```
Variable Name: varApprovalComments
Data Type: Text (Long)
Description: Stores approval comments

Variable Name: varApprovalURL
Data Type: Text
Description: Stores the approval URL
```

## ⚡ **Complete Flow Structure**

```
1. Start (Record-Triggered, Async) ✅
2. Get Current Approval Process
3. Get Approval Comments
4. Get Current Work Items
5. Loop Through Work Items
   ├── Get Approver Details
   ├── Send Personalized Email
   └── Send Custom Notification (optional)
```

## 🚀 **Quick Copy-Paste Email Body**

**Use this exact email body in your Send Email element:**

```
Hello {!Get_Approver_Details.Name}

A Case Number {!$Record.CaseNumber} has been assigned for Approval by {!$Record.CreatedBy.Name} with Comment "{!Get_Approval_Comments.Comments}"

Please click on the link to view the record : Approval url Link https://vetrotech--devgdi.sandbox.my.salesforce.com/p/process/ProcessInstanceWorkitemWizardStageManager?id={!Loop_Through_Approvers.Id}

Thanks & Regards
CS Team
```

## ✅ **Testing Checklist**

- [ ] Flow set to Run Asynchronously
- [ ] Email contains approver's actual name
- [ ] Case number displays correctly
- [ ] Submitter name shows correctly
- [ ] Comments appear (even if empty)
- [ ] Approval URL contains correct WorkItem ID
- [ ] Email sent to all approvers

## 🔧 **Troubleshooting**

**If approver name shows as blank:**
- Check User object permissions
- Verify ActorId is correct
- Add fallback: `{!IF(LEN(Get_Approver_Details.Name) > 0, Get_Approver_Details.Name, "Team")}`

**If comments are blank:**
- Comments might not exist for initial submission
- Use conditional logic to handle empty comments

**If approval URL doesn't work:**
- Verify WorkItem ID format
- Check your Salesforce instance URL

This will give you exactly the email format you specified with personalized approver names! 🚀