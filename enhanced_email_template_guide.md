# Enhanced Email Template - Approval URL, Comments, and Submitter Details

## 🎯 **Complete Email Template with All Details**

```
Hello {!Get_Approver_Details.Name}

A Case Number {!$Record.CaseNumber} has been assigned for Approval by {!Get_Submitter_Details.Name} with Comment "{!Get_Submitter_Comments.Comments}"

Case Details:
- Priority: {!$Record.Priority}
- Subject: {!$Record.Subject}
- Status: {!$Record.Status}

Submitter Comments: "{!Get_Submitter_Comments.Comments}"
Previous Approver Comments: "{!Get_Previous_Comments.Comments}"

Please click on the link to view and approve the record:
Approval URL: https://vetrotech--devgdi.sandbox.my.salesforce.com/p/process/ProcessInstanceWorkitemWizardStageManager?id={!Loop_Through_Approvers.Id}

Direct Case Link: https://vetrotech--devgdi.sandbox.my.salesforce.com/{!$Record.Id}

Thanks & Regards
CS Team
```

## 🚀 **Flow Elements to Get All Required Data**

### **Element 1: Get Current ProcessInstance**
```
Element: Get Records
Label: Get Current Approval Process
Object: ProcessInstance
Conditions:
- TargetObjectId Equals {!$Record.Id}
- Status Equals Started
Store: Only the first record
```

### **Element 2: Get Submitter Comments (Initial Submission)**
```
Element: Get Records
Label: Get Submitter Comments
Object: ProcessInstanceHistory
Conditions:
- ProcessInstanceId Equals {!Get_Current_Approval_Process.Id}
- StepStatus Equals Started
Sort: CreatedDate Ascending
Store: Only the first record
```

### **Element 3: Get All Previous Comments**
```
Element: Get Records
Label: Get All Previous Comments
Object: ProcessInstanceHistory
Conditions:
- ProcessInstanceId Equals {!Get_Current_Approval_Process.Id}
- StepStatus IN ('Approved', 'Rejected')
Sort: CreatedDate Descending
Store: All records
```

### **Element 4: Get Current Work Items (For Approval URL)**
```
Element: Get Records
Label: Get Current Work Items
Object: ProcessInstanceWorkitem
Conditions:
- ProcessInstanceId Equals {!Get_Current_Approval_Process.Id}
Store: All records
```

### **Element 5: Get Submitter Details**
```
Element: Get Records
Label: Get Submitter Details
Object: User
Conditions:
- Id Equals {!Get_Current_Approval_Process.SubmittedById}
Store: Only the first record
```

### **Element 6: Loop Through Work Items**
```
Element: Loop
Label: Loop Through Approvers
Collection: {!Get_Current_Work_Items}
```

### **Element 7: Get Approver Details (Inside Loop)**
```
Element: Get Records
Label: Get Approver Details
Object: User
Conditions:
- Id Equals {!Loop_Through_Approvers.ActorId}
Store: Only the first record
```

### **Element 8: Build Dynamic Approval URL (Inside Loop)**
```
Element: Assignment
Label: Build Approval URL
Variable: varApprovalURL
Value: https://vetrotech--devgdi.sandbox.my.salesforce.com/p/process/ProcessInstanceWorkitemWizardStageManager?id={!Loop_Through_Approvers.Id}
```

### **Element 9: Enhanced Email Template (Inside Loop)**
```
Element: Send Email
Label: Send Enhanced Approval Email
To Addresses: {!Get_Approver_Details.Email}
Subject: 🔔 Approval Required - Case {!$Record.CaseNumber}

Body:
Hello {!Get_Approver_Details.Name}

A Case Number {!$Record.CaseNumber} has been assigned for Approval by {!Get_Submitter_Details.Name} with Comment "{!Get_Submitter_Comments.Comments}"

📋 Case Details:
• Priority: {!$Record.Priority}
• Subject: {!$Record.Subject}
• Created Date: {!$Record.CreatedDate}
• Owner: {!$Record.Owner.Name}

💬 Submission Comments:
"{!Get_Submitter_Comments.Comments}"

🔗 Approval Actions:
Please click on the link to view and approve the record:
Approval URL: {!varApprovalURL}

Direct Case Link: https://vetrotech--devgdi.sandbox.my.salesforce.com/{!$Record.Id}

Thanks & Regards
CS Team
```

## 📋 **Advanced Email Template with Conditional Logic**

**Use this enhanced version to handle empty comments gracefully:**

```
Hello {!Get_Approver_Details.Name}

A Case Number {!$Record.CaseNumber} has been assigned for Approval by {!Get_Submitter_Details.Name}{!IF(LEN(Get_Submitter_Comments.Comments) > 0, " with Comment \"" & Get_Submitter_Comments.Comments & "\"", "")}

📋 Case Details:
• Priority: {!$Record.Priority}
• Subject: {!$Record.Subject}
• Created: {!$Record.CreatedDate}
• Owner: {!$Record.Owner.Name}

💬 Comments:
{!IF(LEN(Get_Submitter_Comments.Comments) > 0, "Submitter: \"" & Get_Submitter_Comments.Comments & "\"", "No submission comments")}

🔗 Approval Actions:
To approve or reject this case, click here:
{!varApprovalURL}

Or view case details:
https://vetrotech--devgdi.sandbox.my.salesforce.com/{!$Record.Id}

Thanks & Regards
CS Team
```

## 🎯 **Variables You'll Need**

**Create these variables in your flow:**

```
Variable 1:
Name: varApprovalURL
Data Type: Text
Description: Stores the complete approval URL

Variable 2:
Name: varSubmitterComments
Data Type: Text (Long)
Description: Stores submitter comments

Variable 3:
Name: varPreviousComments
Data Type: Text (Long)
Description: Stores previous approver comments
```

## 📊 **HTML Email Template (Professional)**

**For rich formatting:**

```html
<div style="font-family: Arial, sans-serif; max-width: 600px; margin: 0 auto; border: 1px solid #ddd; border-radius: 8px;">
    <!-- Header -->
    <div style="background-color: #1976d2; color: white; padding: 20px; text-align: center; border-radius: 8px 8px 0 0;">
        <h2 style="margin: 0;">🔔 Approval Required</h2>
    </div>
    
    <!-- Content -->
    <div style="padding: 20px;">
        <p style="font-size: 16px; margin-bottom: 20px;">
            Hello <strong>{!Get_Approver_Details.Name}</strong>
        </p>
        
        <p>A Case Number <strong>{!$Record.CaseNumber}</strong> has been assigned for Approval by <strong>{!Get_Submitter_Details.Name}</strong>{!IF(LEN(Get_Submitter_Comments.Comments) > 0, " with Comment \"" & Get_Submitter_Comments.Comments & "\"", "")}</p>
        
        <!-- Case Details Box -->
        <div style="background-color: #f5f5f5; padding: 15px; border-radius: 5px; margin: 20px 0;">
            <h3 style="margin-top: 0; color: #1976d2;">📋 Case Details</h3>
            <p><strong>Priority:</strong> {!$Record.Priority}</p>
            <p><strong>Subject:</strong> {!$Record.Subject}</p>
            <p><strong>Created:</strong> {!$Record.CreatedDate}</p>
            <p><strong>Owner:</strong> {!$Record.Owner.Name}</p>
        </div>
        
        <!-- Comments Section -->
        <div style="background-color: #fff3cd; padding: 15px; border-radius: 5px; margin: 20px 0; border-left: 4px solid #ffc107;">
            <h3 style="margin-top: 0; color: #856404;">💬 Submission Comments</h3>
            <p style="font-style: italic;">"{!Get_Submitter_Comments.Comments}"</p>
        </div>
        
        <!-- Action Buttons -->
        <div style="text-align: center; margin: 30px 0;">
            <a href="{!varApprovalURL}" 
               style="background-color: #28a745; color: white; padding: 12px 24px; text-decoration: none; border-radius: 5px; margin: 0 10px; display: inline-block;">
                ✅ Review & Approve
            </a>
            
            <a href="https://vetrotech--devgdi.sandbox.my.salesforce.com/{!$Record.Id}" 
               style="background-color: #17a2b8; color: white; padding: 12px 24px; text-decoration: none; border-radius: 5px; margin: 0 10px; display: inline-block;">
                📄 View Case
            </a>
        </div>
        
        <!-- Direct Links -->
        <div style="border-top: 1px solid #ddd; padding-top: 20px; margin-top: 30px;">
            <p><strong>Direct Links:</strong></p>
            <p>🔗 Approval URL: <a href="{!varApprovalURL}">{!varApprovalURL}</a></p>
            <p>📋 Case URL: <a href="https://vetrotech--devgdi.sandbox.my.salesforce.com/{!$Record.Id}">https://vetrotech--devgdi.sandbox.my.salesforce.com/{!$Record.Id}</a></p>
        </div>
    </div>
    
    <!-- Footer -->
    <div style="background-color: #f8f9fa; text-align: center; padding: 15px; border-radius: 0 0 8px 8px; border-top: 1px solid #ddd;">
        <p style="margin: 0; color: #6c757d; font-size: 14px;">
            Thanks & Regards<br/>
            <strong>CS Team</strong>
        </p>
    </div>
</div>
```

## 🔧 **URL Construction Details**

### **For Approval URL:**
```
Base URL: https://vetrotech--devgdi.sandbox.my.salesforce.com/p/process/ProcessInstanceWorkitemWizardStageManager?id=
WorkItem ID: {!Loop_Through_Approvers.Id}
Complete URL: {!varApprovalURL}
```

### **For Case URL:**
```
Base URL: https://vetrotech--devgdi.sandbox.my.salesforce.com/
Case ID: {!$Record.Id}
Complete URL: https://vetrotech--devgdi.sandbox.my.salesforce.com/{!$Record.Id}
```

## ✅ **Testing Checklist**

- [ ] Approval URL contains correct WorkItem ID
- [ ] Submitter name displays correctly
- [ ] Submitter comments appear (or "No comments" if empty)
- [ ] Case details populate correctly
- [ ] Direct case link works
- [ ] Email formatting looks professional
- [ ] All merge fields resolve properly

## 🚀 **Quick Copy-Paste Template**

**Simple version for immediate use:**

```
Hello {!Get_Approver_Details.Name}

Case {!$Record.CaseNumber} requires your approval.

Submitted by: {!Get_Submitter_Details.Name}
Comments: "{!Get_Submitter_Comments.Comments}"
Priority: {!$Record.Priority}

Approval Link: https://vetrotech--devgdi.sandbox.my.salesforce.com/p/process/ProcessInstanceWorkitemWizardStageManager?id={!Loop_Through_Approvers.Id}

Case Link: https://vetrotech--devgdi.sandbox.my.salesforce.com/{!$Record.Id}

Thanks & Regards
CS Team
```

This will give you all the details you need: approval URL, submitter comments, and professional formatting! 🚀