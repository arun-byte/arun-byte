# Multi-Step Approval Notification Flow Solution

## Overview
This solution creates a Flow that automatically sends emails and custom notifications to all approvers across all approval steps when a record is submitted for approval.

## Flow Architecture

### Flow Type: Record-Triggered Flow
- **Object**: Case (or your target object)
- **Trigger**: When a record is created or updated
- **Entry Criteria**: When approval status changes to "Pending"

## Implementation Steps

### Step 1: Create the Record-Triggered Flow

1. **Flow Setup**:
   - Navigate to Setup → Flows → New Flow
   - Select "Record-Triggered Flow"
   - Choose your object (Case)
   - Configure trigger: "A record is created or updated"

2. **Entry Criteria**:
   ```
   Condition Requirements: All Conditions Are Met (AND)
   Field: {!$Record.Approval_Status__c} (or equivalent field)
   Operator: Equals
   Value: Pending
   ```

### Step 2: Get Approval Process Information

**Element: Get Records (Get Approval Process)**
```
Label: Get Approval Process
Object: ProcessInstance
Conditions:
- TargetObjectId = {!$Record.Id}
- Status = Pending
Store: Only the first record
```

### Step 3: Get All Approval Steps

**Element: Get Records (Get Approval Steps)**
```
Label: Get All Approval Steps
Object: ProcessInstanceStep
Conditions:
- ProcessInstanceId = {!Get_Approval_Process.Id}
Store: All records
```

### Step 4: Get Process Definition

**Element: Get Records (Get Process Definition)**
```
Label: Get Process Definition
Object: ProcessDefinition
Conditions:
- Id = {!Get_Approval_Process.ProcessDefinitionId}
Store: Only the first record
```

### Step 5: Get Approval Process Nodes

**Element: Get Records (Get Process Nodes)**
```
Label: Get Process Nodes
Object: ProcessNode
Conditions:
- ProcessDefinitionId = {!Get_Process_Definition.Id}
Store: All records
```

### Step 6: Loop Through Approval Steps

**Element: Loop**
```
Label: Loop Through Steps
Collection: {!Get_All_Approval_Steps}
Direction: First item to last item
```

### Step 7: Get Step Approvers (Inside Loop)

**Element: Get Records (Get Step Approvers)**
```
Label: Get Current Step Approvers
Object: ProcessInstanceWorkitem
Conditions:
- ProcessInstanceId = {!Get_Approval_Process.Id}
- OriginalActorId = {!Loop_Through_Steps.ActorId}
Store: All records
```

### Step 8: Get User/Queue Information (Inside Loop)

**Element: Decision (Check Approver Type)**
```
Label: Check if User or Queue
Outcomes:
1. Is User
   - Condition: {!Get_Current_Step_Approvers.ActorId} starts with "005"
2. Is Queue  
   - Condition: {!Get_Current_Step_Approvers.ActorId} starts with "00G"
```

**For Users - Get Records:**
```
Label: Get User Details
Object: User
Conditions:
- Id = {!Get_Current_Step_Approvers.ActorId}
Store: Only the first record
```

**For Queues - Get Records:**
```
Label: Get Queue Details
Object: Group
Conditions:
- Id = {!Get_Current_Step_Approvers.ActorId}
- Type = Queue
Store: Only the first record
```

**For Queue Members - Get Records:**
```
Label: Get Queue Members
Object: GroupMember
Conditions:
- GroupId = {!Get_Queue_Details.Id}
Store: All records
```

### Step 9: Send Email Notifications

**Element: Send Email (Inside Loop)**
```
Label: Send Email to Approvers
To Addresses: 
- For Users: {!Get_User_Details.Email}
- For Queue: Use collection of queue member emails

Subject: Approval Required: {!$Record.Subject} - Case {!$Record.CaseNumber}

Body:
Hello {!IF(Get_User_Details.Name != null, Get_User_Details.Name, Get_Queue_Details.Name + " Team")},

A case has been submitted for approval and requires your attention.

Case Details:
- Case Number: {!$Record.CaseNumber}
- Subject: {!$Record.Subject}
- Priority: {!$Record.Priority}
- Submitted By: {!$Record.CreatedBy.Name}
- Submission Date: {!$Record.CreatedDate}
- Current Status: {!$Record.Status}

Comments: {!Get_Approval_Process.ProcessInstance.CompletedDate}

Please review and take action on this approval request.

Click here to view the record: {!$Record.Link}

Best regards,
{!$Organization.Name} Team
```

### Step 10: Send Custom Notifications

**Element: Send Custom Notification (Inside Loop)**
```
Label: Send Custom Notification
Notification Type: Create custom notification type or use standard
Recipients: 
- User IDs: {!Get_User_Details.Id}
- Or Queue Member IDs for queues

Title: "Approval Required: Case {!$Record.CaseNumber}"
Body: "A case requires your approval. Priority: {!$Record.Priority}"
Target Record: {!$Record.Id}
```

## Complete Flow Structure

```
Start (Record-Triggered)
│
├── Get Approval Process
│
├── Decision: Approval Found?
│   │
│   ├── YES Branch:
│   │   ├── Get All Approval Steps
│   │   ├── Get Process Definition  
│   │   ├── Get Process Nodes
│   │   │
│   │   ├── Loop: Through All Steps
│   │   │   ├── Get Current Step Approvers
│   │   │   ├── Decision: User or Queue?
│   │   │   │   │
│   │   │   │   ├── User Branch:
│   │   │   │   │   ├── Get User Details
│   │   │   │   │   ├── Send Email to User
│   │   │   │   │   └── Send Notification to User
│   │   │   │   │
│   │   │   │   └── Queue Branch:
│   │   │   │       ├── Get Queue Details
│   │   │   │       ├── Get Queue Members
│   │   │   │       ├── Loop: Through Queue Members
│   │   │   │       │   ├── Send Email to Member
│   │   │   │       │   └── Send Notification to Member
│   │   │   │       └── End Queue Member Loop
│   │   │   └── End Step Loop
│   │   │
│   │   └── Success Actions
│   │
│   └── NO Branch:
│       └── Error Handling
│
└── End
```

## Advanced Email Template

Create a more comprehensive email template:

```html
<!DOCTYPE html>
<html>
<head>
    <style>
        .container { font-family: Arial, sans-serif; max-width: 600px; margin: 0 auto; }
        .header { background-color: #1976d2; color: white; padding: 20px; text-align: center; }
        .content { padding: 20px; background-color: #f5f5f5; }
        .case-details { background-color: white; padding: 15px; border-radius: 5px; margin: 10px 0; }
        .button { background-color: #1976d2; color: white; padding: 10px 20px; text-decoration: none; border-radius: 5px; display: inline-block; }
        .footer { text-align: center; padding: 10px; color: #666; }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <h2>Approval Required</h2>
        </div>
        
        <div class="content">
            <p>Hello {!IF(Get_User_Details.Name != null, Get_User_Details.Name, Get_Queue_Details.Name + " Team")},</p>
            
            <p>A case has been submitted for approval and requires your immediate attention.</p>
            
            <div class="case-details">
                <h3>Case Information</h3>
                <p><strong>Case Number:</strong> {!$Record.CaseNumber}</p>
                <p><strong>Subject:</strong> {!$Record.Subject}</p>
                <p><strong>Priority:</strong> {!$Record.Priority}</p>
                <p><strong>Status:</strong> {!$Record.Status}</p>
                <p><strong>Submitted By:</strong> {!$Record.CreatedBy.Name}</p>
                <p><strong>Submission Date:</strong> {!$Record.CreatedDate}</p>
                <p><strong>Description:</strong> {!$Record.Description}</p>
            </div>
            
            <div class="case-details">
                <h3>Approval Information</h3>
                <p><strong>Approval Step:</strong> Step {!Loop_Through_Steps.StepNumber}</p>
                <p><strong>Process Name:</strong> {!Get_Process_Definition.Name}</p>
                <p><strong>Comments:</strong> {!Get_Approval_Process.ProcessInstance.CompletedDate}</p>
            </div>
            
            <p style="text-align: center; margin: 30px 0;">
                <a href="{!$Record.Link}" class="button">Review and Approve</a>
            </p>
            
            <p><strong>Next Steps:</strong></p>
            <ul>
                <li>Click the button above to review the case details</li>
                <li>Approve or reject the request</li>
                <li>Add any necessary comments</li>
            </ul>
        </div>
        
        <div class="footer">
            <p>This is an automated notification from {!$Organization.Name}</p>
            <p>Please do not reply to this email</p>
        </div>
    </div>
</body>
</html>
```

## Custom Notification Setup

### Step 1: Create Custom Notification Type

1. **Setup → Custom Notifications**
2. **New Custom Notification**:
   - API Name: `Approval_Required`
   - Master Label: `Approval Required`
   - Description: `Notification for approval requests`

3. **Configure Channels**:
   - ☑ Desktop
   - ☑ Mobile
   - ☑ Email (optional)

### Step 2: Configure Notification in Flow

```
Send Custom Notification Element:
- Notification Type: Approval_Required
- Title: Approval Required: Case {!$Record.CaseNumber}
- Body: Priority {!$Record.Priority} case requires approval
- Target Record: {!$Record.Id}
- Recipients: {!Get_User_Details.Id} or queue members
```

## Error Handling and Best Practices

### Add Error Handling Elements

1. **Fault Connector**: Connect fault paths to error handling
2. **Error Notification**: Send admin notification if flow fails
3. **Logging**: Create records for audit trail

### Performance Optimization

1. **Limit Records**: Use "Only first record" where appropriate
2. **Bulk Operations**: Process collections efficiently
3. **Async Processing**: Use async path for heavy operations

### Testing Strategy

1. **Test with different approval processes**
2. **Test with user and queue approvers**
3. **Test with multi-step approvals**
4. **Verify email delivery**
5. **Check notification delivery**

## Variables and Resources

### Text Variables
```
varApprovalProcessId (Text)
varCurrentStepNumber (Number)
varApprovalComments (Text)
varEmailBody (Text, Long Text Area)
```

### Collection Variables
```
colStepApprovers (Text Collection)
colQueueMembers (Text Collection)
colEmailAddresses (Text Collection)
```

## Activation and Monitoring

### Before Activation
- [ ] Test in sandbox environment
- [ ] Verify all email templates render correctly
- [ ] Test notification delivery
- [ ] Validate with different user types

### After Activation
- [ ] Monitor flow execution in Debug Logs
- [ ] Check email deliverability
- [ ] Verify notification receipt
- [ ] Monitor for any errors

### Monitoring Dashboard
Create reports to track:
- Flow execution success rate
- Email delivery status
- Notification delivery status
- Approval response times

## Troubleshooting

### Common Issues
1. **Emails not sending**: Check email deliverability settings
2. **Notifications not appearing**: Verify notification type setup
3. **Flow not triggering**: Check entry criteria
4. **Missing approvers**: Verify approval process configuration

### Debug Tips
1. Enable debug mode for flow
2. Check System Debug Logs
3. Use Email Log monitoring
4. Test with Debug mode enabled

This comprehensive solution will ensure all approvers across all steps receive both email notifications and custom notifications when an approval is submitted.