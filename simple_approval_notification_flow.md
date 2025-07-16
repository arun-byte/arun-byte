# Simplified Approval Notification Flow - Quick Implementation

## Quick Start Solution
For immediate implementation, here's a streamlined version that sends notifications to all approvers when approval is submitted.

## Flow Configuration (Simplified)

### 1. Basic Flow Setup
```
Flow Type: Record-Triggered Flow
Object: Case (or your object)
Trigger: A record is created or updated
Entry Criteria: 
- Field: {!$Record.Approval_Status__c} 
- Operator: Equals
- Value: Pending
```

### 2. Get Active Approval Process
```
Element: Get Records
Label: Get Current Approval
Object: ProcessInstance
Conditions:
- TargetObjectId = {!$Record.Id}
- Status = Pending
Store: Only the first record
```

### 3. Get Current Approval Step
```
Element: Get Records  
Label: Get Current Step
Object: ProcessInstanceWorkitem
Conditions:
- ProcessInstanceId = {!Get_Current_Approval.Id}
Store: All records
```

### 4. Decision: Check if Approval Found
```
Element: Decision
Label: Approval Process Active?
Outcome: Approval Found
- Condition: {!Get_Current_Approval} Is Null = {!$GlobalConstant.False}
```

### 5. Loop Through Current Approvers
```
Element: Loop
Label: Loop Current Approvers
Collection: {!Get_Current_Step}
```

### 6. Get Approver Details (Inside Loop)
```
Element: Get Records
Label: Get Approver Info
Object: User
Conditions:
- Id = {!Loop_Current_Approvers.ActorId}
Store: Only the first record
```

### 7. Send Email (Inside Loop)
```
Element: Send Email
Label: Email Approver
To Addresses: {!Get_Approver_Info.Email}
Subject: 🔔 Approval Required: Case {!$Record.CaseNumber}

Body:
Hello {!Get_Approver_Info.Name},

A case requires your approval:

📋 Case Number: {!$Record.CaseNumber}
📌 Subject: {!$Record.Subject}  
⚡ Priority: {!$Record.Priority}
👤 Submitted by: {!$Record.CreatedBy.Name}
📅 Date: {!$Record.CreatedDate}

🔗 Click to Review: [View Case]({!$Record.Link})

Please review and approve as soon as possible.

Thanks,
{!$Organization.Name} Team
```

### 8. Send Custom Notification (Inside Loop)
```
Element: Send Custom Notification
Label: Notify Approver
Notification Type: [Your Custom Notification Type]
Recipients: {!Get_Approver_Info.Id}
Title: Approval Required: {!$Record.CaseNumber}
Body: Priority {!$Record.Priority} case needs approval
Target Record: {!$Record.Id}
```

## Enhanced Version with Queue Support

### Add Queue Handling (Optional)
```
Element: Decision (Inside Loop)
Label: Check User vs Queue
Outcomes:
1. Is User: {!Loop_Current_Approvers.ActorId} starts with "005"
2. Is Queue: {!Loop_Current_Approvers.ActorId} starts with "00G"
```

**For Queue Branch:**
```
Get Records: Get Queue Members
Object: GroupMember  
Conditions: GroupId = {!Loop_Current_Approvers.ActorId}

Loop: Through Queue Members
- Send Email to each member
- Send Notification to each member
```

## Pre-built Flow Template

### Copy-Paste Flow Configuration:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Flow xmlns="http://soap.sforce.com/2006/04/metadata">
    <apiVersion>59.0</apiVersion>
    <description>Send email and notifications to all approvers when approval is submitted</description>
    <label>Approval Notification Flow</label>
    <processType>AutoLaunchedFlow</processType>
    
    <!-- Start: Record Triggered -->
    <start>
        <locationX>50</locationX>
        <locationY>50</locationY>
        <connector>
            <targetReference>Get_Current_Approval</targetReference>
        </connector>
        <object>Case</object>
        <recordTriggerType>CreateAndUpdate</recordTriggerType>
        <triggerType>RecordAfterSave</triggerType>
        <filterLogic>AND</filterLogic>
        <filters>
            <field>Approval_Status__c</field>
            <operator>EqualTo</operator>
            <value>
                <stringValue>Pending</stringValue>
            </value>
        </filters>
    </start>
    
    <!-- Get Current Approval Process -->
    <recordLookups>
        <name>Get_Current_Approval</name>
        <label>Get Current Approval</label>
        <locationX>50</locationX>
        <locationY>150</locationY>
        <assignNullValuesIfNoRecordsFound>false</assignNullValuesIfNoRecordsFound>
        <connector>
            <targetReference>Check_Approval_Found</targetReference>
        </connector>
        <filterLogic>AND</filterLogic>
        <filters>
            <field>TargetObjectId</field>
            <operator>EqualTo</operator>
            <value>
                <elementReference>$Record.Id</elementReference>
            </value>
        </filters>
        <filters>
            <field>Status</field>
            <operator>EqualTo</operator>
            <value>
                <stringValue>Pending</stringValue>
            </value>
        </filters>
        <object>ProcessInstance</object>
        <outputReference>varCurrentApproval</outputReference>
        <queriedFields>Id</queriedFields>
        <queriedFields>ProcessDefinitionId</queriedFields>
        <queriedFields>TargetObjectId</queriedFields>
    </recordLookups>
    
    <!-- Additional elements would continue here... -->
</Flow>
```

## Quick Setup Checklist

### 1. Prerequisites Setup (5 minutes)
- [ ] Create Custom Notification Type
- [ ] Ensure email deliverability is configured
- [ ] Test approval process is working

### 2. Flow Creation (15 minutes)
- [ ] Create new Record-Triggered Flow
- [ ] Add all elements as described above
- [ ] Configure entry criteria
- [ ] Test in debug mode

### 3. Testing (10 minutes)
- [ ] Submit test case for approval
- [ ] Verify emails are sent
- [ ] Verify notifications appear
- [ ] Check all approvers receive notifications

### 4. Activation (2 minutes)
- [ ] Activate the flow
- [ ] Monitor initial executions
- [ ] Verify production behavior

## Email Template Examples

### Simple Text Email:
```
Subject: Approval Required: Case {!$Record.CaseNumber}

Hello {!Get_Approver_Info.Name},

You have a pending approval request for:
- Case: {!$Record.CaseNumber}
- Subject: {!$Record.Subject}
- Priority: {!$Record.Priority}

Click here to review: {!$Record.Link}

Thank you,
CS Team
```

### Rich HTML Email:
```html
<div style="font-family: Arial; max-width: 600px;">
    <div style="background: #1976d2; color: white; padding: 20px; text-align: center;">
        <h2>⚡ Approval Required</h2>
    </div>
    
    <div style="padding: 20px; background: #f5f5f5;">
        <p>Hello <strong>{!Get_Approver_Info.Name}</strong>,</p>
        
        <div style="background: white; padding: 15px; border-radius: 5px; margin: 15px 0;">
            <h3>📋 Case Details</h3>
            <p><strong>Case Number:</strong> {!$Record.CaseNumber}</p>
            <p><strong>Subject:</strong> {!$Record.Subject}</p>
            <p><strong>Priority:</strong> {!$Record.Priority}</p>
            <p><strong>Submitted By:</strong> {!$Record.CreatedBy.Name}</p>
        </div>
        
        <div style="text-align: center; margin: 30px 0;">
            <a href="{!$Record.Link}" 
               style="background: #1976d2; color: white; padding: 12px 24px; 
                      text-decoration: none; border-radius: 5px; display: inline-block;">
                🔍 Review & Approve
            </a>
        </div>
    </div>
    
    <div style="text-align: center; padding: 10px; color: #666; font-size: 12px;">
        Automated notification from {!$Organization.Name}
    </div>
</div>
```

## Custom Notification Setup

### Create Notification Type:
1. **Setup → Custom Notifications → New**
2. **Settings:**
   ```
   API Name: Approval_Required_Notification
   Master Label: Approval Required
   Description: Notifications for approval requests
   
   Channels:
   ☑ Desktop Notifications
   ☑ Mobile Push Notifications  
   ☑ Email Notifications (optional)
   ```

### Use in Flow:
```
Send Custom Notification:
- Notification Type: Approval_Required_Notification
- Title: 🔔 Approval: {!$Record.CaseNumber}
- Body: {!$Record.Priority} priority case needs approval
- Target: {!$Record.Id}
- Recipients: {!Get_Approver_Info.Id}
```

## Monitoring and Troubleshooting

### Check Flow Execution:
- **Setup → Flows → [Your Flow] → View Details**
- Monitor "Run History" tab
- Check for any failed executions

### Verify Email Delivery:
- **Setup → Email Administration → Deliverability**  
- **Setup → Email Administration → Send Email**
- Check spam folders if emails not received

### Test Notifications:
- Enable notifications in user preferences
- Check bell icon in Salesforce for notifications
- Verify mobile app notifications

This simplified approach gets you up and running quickly while still providing comprehensive approval notifications to all stakeholders!