# Salesforce Flow Approval Process: Bell Notifications for Queue Members

## Overview

With Salesforce Spring '25, you can now create modern approval processes using Flow Builder that send both email and bell notifications (in-app notifications) to queue members when users submit records for approval. This guide will walk you through setting up a complete approval process with queue notifications.

## Key Benefits

- **Modern UI**: New Approvals Lightning App for managing submissions
- **Free to Use**: No automation credits consumed (unlike regular Flow Orchestrations)
- **Flexible Notifications**: Both email and in-app bell notifications
- **Queue Support**: Assign approvals to queues with automatic member notification
- **Email Responses**: Members can approve/reject directly via email replies

## Prerequisites

1. **Spring '25 enabled org** or later
2. **Queue setup** with appropriate members
3. **Proper permissions** for queue members
4. **Process Automation settings** configured for email responses (optional)

## Step-by-Step Implementation

### Step 1: Create and Configure the Queue

1. **Navigate to Setup** → **Users** → **Queues**
2. **Create New Queue** or use existing queue
3. **Important configurations**:
   - Add an **email address** to the queue (required for notifications)
   - Add **queue members** (users who should receive notifications)
   - Ensure the object you're approving supports queues (Cases, Leads, custom objects)

```
Queue Name: Level2_Approval_Queue
Email: level2approvals@yourcompany.com
Members: [Queue members who should receive notifications]
```

### Step 2: Create the Approval Screen Flow

1. **Navigate to Flow Builder**
2. **Create New Flow** → **Templates**
3. **Select**: "Approvals Workflow: Evaluate Approval Requests"
4. **Customize the template** (optional):
   - Modify screen elements
   - Add custom fields if needed
   - Keep required output variables: `approvalDecision` and `approvalComments`
5. **Save and Activate** the flow

### Step 3: Create Supporting Flows (Optional)

Create additional flows for post-approval actions:

#### Background Flow for Status Updates:
```
Flow Type: Autolaunched Flow
Input Variables: 
- recordId (Text)
- approvalDecision (Text)
- approvalComments (Text)

Actions:
- Update Records element to modify approval status field
- Send custom email notifications (if needed)
```

### Step 4: Build the Flow Approval Orchestration

1. **Create New Flow** → **From Scratch**
2. **Choose Flow Type**:
   - **Record-Triggered Approval Orchestration**: For automatic submission on record create/update
   - **Autolaunched Approval Orchestration**: For manual submission via buttons

#### For Record-Triggered:
```
Trigger: When record is created or updated
Object: [Your object - Case, Lead, Custom Object]
Conditions: [Define when approval should trigger]
```

#### For Autolaunched:
```
No trigger required - will be called via button or other automation
```

### Step 5: Configure the Approval Stage and Steps

#### Approval Step Configuration:
1. **Add New Stage**
2. **Add Approval Step**:
   - **Flow**: Select your approval screen flow
   - **Approver Assignment**: 
     - Type: **Queue**
     - Value: Your queue name or use **Queue Resource** for dynamic assignment
   - **Record Scope**: Select the record field (e.g., Case ID)
   - **Notification Settings**:
     - ✅ **Lock Record**: Recommended for data integrity
     - ✅ **Send Email Notifications**: Enables email notifications to queue members
     - ✅ **Enable Email Approval Response**: Allows email replies with "Approve"/"Reject"
   - **Step Completion**: "When assigned user completes the action"

#### Background Step (Optional):
1. **Add Background Step** after approval step
2. **Select autolaunched flow** for post-approval actions
3. **Map input variables** from approval step outputs

### Step 6: Add Decision Logic (If Needed)

For complex approval routing:
```
Decision Element:
- If approvalDecision equals "Approve" → Continue to next stage or end
- If approvalDecision equals "Reject" → Route to rejection actions
```

### Step 7: Configure Page Layout Components

#### Add Work Guide Component:
1. **Edit Lightning Record Page**
2. **Add Component**: "Flow Orchestration Work Guide"
3. **Placement**: Right column (recommended)
4. **Settings**: 
   - ✅ Hide component when no work items
   - Configure visibility rules if needed

#### Add Approval Trace Component:
1. **Add Component**: "Approval Trace"
2. **Purpose**: Shows approval history (replaces standard Approval History)
3. **Placement**: Separate tab or below record details

### Step 8: Enable Email Response Approvals

1. **Navigate to Setup** → **Process Automation** → **Process Automation Settings**
2. **Enable**: "Enable email approval response"
3. **Result**: Queue members can reply to emails with "Approve" or "Reject"

### Step 9: Configure Permissions

#### Required Permissions for Queue Members:
```
Object Permissions:
- Approval Submission: Read
- Approval Work Item: Read, Edit
- [Your Object]: Read, Edit (as needed)

System Permissions:
- Access to Approvals Lightning App
- Ability to view Work Guide component
```

#### Profile/Permission Set Configuration:
1. **Grant object access** to approval-related objects
2. **Enable Lightning App access** for Approvals app
3. **Verify queue membership** permissions

### Step 10: Test the Complete Process

#### Test Scenarios:
1. **Submit Record for Approval**:
   - Verify queue members receive email notifications
   - Check for in-app bell notifications
   - Confirm Work Guide appears on record page

2. **Queue Member Actions**:
   - Test approval via Work Guide component
   - Test email response approval
   - Verify only one member needs to approve

3. **Post-Approval**:
   - Confirm background actions execute
   - Verify status updates
   - Check Approval Trace component shows history

## Bell Notification Configuration

### In-App Notifications Setup:
Bell notifications are automatically enabled when:
- Queue members have access to the Approvals Lightning App
- Work items are assigned to the queue
- Users have proper permissions to view Approval Work Items

### Accessing Notifications:
Queue members will see notifications in:
1. **Bell icon** in Salesforce header
2. **Approvals Lightning App** → "Review My Approval Work Items"
3. **Work Guide component** on record pages

## Troubleshooting Common Issues

### Email Notifications Not Working:
- Verify queue has email address configured
- Check "Enable email approval response" setting
- Confirm queue members have valid email addresses
- Review email deliverability settings

### Bell Notifications Not Appearing:
- Verify user permissions on Approval objects
- Check Lightning App access
- Ensure Work Guide component is added to page layout
- Confirm user is member of the assigned queue

### Queue Assignment Issues:
- Verify object supports queues
- Check queue member permissions
- Ensure queue is active and properly configured
- For dynamic assignment, verify Queue Resource variable contains queue API name

## Best Practices

1. **Queue Management**:
   - Use descriptive queue names
   - Regularly review and update queue membership
   - Configure proper email addresses for queue notifications

2. **Notification Strategy**:
   - Enable both email and in-app notifications for redundancy
   - Consider time zones when setting up email notifications
   - Train users on both approval methods (UI and email)

3. **Process Design**:
   - Keep approval steps simple and clear
   - Use background steps for automatic actions
   - Implement proper error handling with fault paths

4. **User Experience**:
   - Add clear instructions in approval screens
   - Use the Approval Trace component for visibility
   - Consider custom help text or training materials

## Advanced Features

### Dynamic Queue Assignment:
```
Use Queue Resource variable to assign different queues based on criteria:
- Create formula or flow logic to determine appropriate queue
- Pass queue API name to Queue Resource variable
- Enable complex routing scenarios
```

### Custom Notifications:
```
Create custom email templates and flows for:
- Specialized notification content
- Additional stakeholder notifications
- Integration with external systems
```

### Reporting and Analytics:
```
Use new approval objects for reporting:
- Approval Submissions
- Approval Work Items  
- Approval Submission Details
Create custom report types for approval metrics
```

## Migration from Legacy Approval Processes

If migrating from traditional approval processes:

1. **Review current process logic** and map to Flow stages/steps
2. **Update page layouts** to include new components
3. **Train users** on new Approvals Lightning App
4. **Test thoroughly** before deactivating legacy processes
5. **Consider phased rollout** for complex processes

## Conclusion

The new Flow Approval Processes provide a modern, flexible solution for queue-based approvals with comprehensive notification capabilities. By following this guide, you'll have a robust approval process that ensures queue members are promptly notified and can efficiently handle approval requests through multiple channels.

## Additional Resources

- [Salesforce Help: Flow Approval Processes](https://help.salesforce.com/s/articleView?id=platform.automate_automated_approvals.htm)
- [Trailhead: Flow Orchestration](https://trailhead.salesforce.com/content/learn/modules/flow-orchestration)
- [Release Notes: Spring '25 Flow Features](https://help.salesforce.com/s/articleView?id=release-notes.rn_forcecom_flow.htm)