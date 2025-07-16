# Step-by-Step Implementation Guide - Approval Notification Flow

## 🎯 **What We're Building**
A Flow that automatically sends emails and notifications to all approvers when a case is submitted for approval.

---

## **STEP 1: Create Custom Notification Type**

### Navigate to Setup:
1. Click the **gear icon** (⚙️) in the top right
2. Select **Setup**
3. In Quick Find, search for **"Custom Notifications"**
4. Click **Custom Notifications**

### Create New Notification:
5. Click **New Custom Notification**
6. Fill in the details:
   ```
   Master Label: Approval Required
   API Name: Approval_Required (auto-filled)
   Description: Notification for approval requests
   ```

### Configure Channels:
7. Check these boxes:
   - ☑️ **Desktop Notifications**
   - ☑️ **Mobile Push Notifications**
   - ☑️ **Email Notifications** (optional)

8. Click **Save**

✅ **Checkpoint**: You should see "Approval Required" in your Custom Notifications list

---

## **STEP 2: Create the Record-Triggered Flow**

### Start Flow Creation:
1. In Setup, search for **"Flows"**
2. Click **Flows**
3. Click **New Flow**
4. Select **Record-Triggered Flow**
5. Click **Create**

### Configure Flow Trigger:
6. **Object**: Select **Case** (or your object)
7. **Configure Trigger**: 
   - Select **A record is created or updated**
8. **Set Entry Criteria**:
   - **Condition Requirements**: All Conditions Are Met (AND)
   - Click **Add Condition**:
     ```
     Resource: {!$Record.Approval_Status__c}
     Operator: Equals  
     Value: Pending
     ```
   - **Note**: Replace `Approval_Status__c` with your actual approval status field

9. **Optimize the Flow For**: Actions and Related Records
10. Click **Done**

✅ **Checkpoint**: Your flow canvas should show the Start element

---

## **STEP 3: Add Get Records - Current Approval Process**

### Add Element:
1. Click the **+** icon after Start
2. Select **Get Records**
3. Configure:
   ```
   Label: Get Current Approval
   API Name: Get_Current_Approval
   Object: ProcessInstance
   ```

### Set Conditions:
4. **Filter ProcessInstance Records**:
   - **Condition Requirements**: All Conditions Are Met (AND)
   - **Row 1**:
     ```
     Field: TargetObjectId
     Operator: Equals
     Value: {!$Record.Id}
     ```
   - Click **Add Condition**
   - **Row 2**:
     ```
     Field: Status  
     Operator: Equals
     Value: Pending
     ```

### Configure Storage:
5. **How Many Records to Store**: Only the first record
6. **How to Store Record Data**: Automatically store all fields
7. Click **Done**

✅ **Checkpoint**: "Get Current Approval" element should appear on canvas

---

## **STEP 4: Add Get Records - Current Approval Step**

### Add Element:
1. Click **+** after "Get Current Approval"
2. Select **Get Records**
3. Configure:
   ```
   Label: Get Current Step
   API Name: Get_Current_Step
   Object: ProcessInstanceWorkitem
   ```

### Set Conditions:
4. **Filter ProcessInstanceWorkitem Records**:
   - **Condition Requirements**: All Conditions Are Met (AND)
   - **Row 1**:
     ```
     Field: ProcessInstanceId
     Operator: Equals
     Value: {!Get_Current_Approval.Id}
     ```

### Configure Storage:
5. **How Many Records to Store**: All records
6. **How to Store Record Data**: Automatically store all fields
7. Click **Done**

✅ **Checkpoint**: "Get Current Step" element should appear on canvas

---

## **STEP 5: Add Decision - Check if Approval Found**

### Add Element:
1. Click **+** after "Get Current Step"
2. Select **Decision**
3. Configure:
   ```
   Label: Approval Process Active?
   API Name: Approval_Process_Active
   ```

### Configure Outcome:
4. **Outcome Details**:
   ```
   Label: Approval Found
   API Name: Approval_Found
   ```

5. **Condition Requirements**: All Conditions Are Met (AND)
6. **Row 1**:
   ```
   Resource: {!Get_Current_Approval}
   Operator: Is Null
   Value: {!$GlobalConstant.False}
   ```

7. Click **Done**

✅ **Checkpoint**: Decision element with "Approval Found" path should appear

---

## **STEP 6: Add Loop - Through Current Approvers**

### Add Element:
1. Click **+** on the **"Approval Found"** path
2. Select **Loop**
3. Configure:
   ```
   Label: Loop Current Approvers
   API Name: Loop_Current_Approvers
   Collection: {!Get_Current_Step}
   Direction: First item to last item
   ```

4. Click **Done**

✅ **Checkpoint**: Loop element should show "For Each" path

---

## **STEP 7: Add Get Records - Approver Details (Inside Loop)**

### Add Element:
1. Click **+** on the **"For Each"** path (inside the loop)
2. Select **Get Records**
3. Configure:
   ```
   Label: Get Approver Info
   API Name: Get_Approver_Info
   Object: User
   ```

### Set Conditions:
4. **Filter User Records**:
   - **Condition Requirements**: All Conditions Are Met (AND)
   - **Row 1**:
     ```
     Field: Id
     Operator: Equals
     Value: {!Loop_Current_Approvers.ActorId}
     ```

### Configure Storage:
5. **How Many Records to Store**: Only the first record
6. **How to Store Record Data**: Automatically store all fields
7. Click **Done**

✅ **Checkpoint**: "Get Approver Info" element inside loop

---

## **STEP 8: Add Send Email (Inside Loop)**

### Add Element:
1. Click **+** after "Get Approver Info"
2. Select **Action**
3. Search for and select **Send Email**
4. Configure:
   ```
   Label: Email Approver
   API Name: Email_Approver
   ```

### Set Input Values:
5. **To Addresses**: {!Get_Approver_Info.Email}
6. **Subject**: 
   ```
   🔔 Approval Required: Case {!$Record.CaseNumber}
   ```

7. **Body**:
   ```
   Hello {!Get_Approver_Info.Name},

   A case requires your approval:

   📋 Case Number: {!$Record.CaseNumber}
   📌 Subject: {!$Record.Subject}  
   ⚡ Priority: {!$Record.Priority}
   👤 Submitted by: {!$Record.CreatedBy.Name}
   📅 Date: {!$Record.CreatedDate}

   🔗 Click to Review: {!$Record.Link}

   Please review and approve as soon as possible.

   Thanks,
   {!$Organization.Name} Team
   ```

8. Click **Done**

✅ **Checkpoint**: "Email Approver" element inside loop

---

## **STEP 9: Add Send Custom Notification (Inside Loop)**

### Add Element:
1. Click **+** after "Email Approver"
2. Select **Action**
3. Search for and select **Send Custom Notification**
4. Configure:
   ```
   Label: Notify Approver
   API Name: Notify_Approver
   ```

### Set Input Values:
5. **Custom Notification Type Id**: 
   - Click the dropdown and select **Approval Required** (created in Step 1)

6. **Recipient Ids**: {!Get_Approver_Info.Id}

7. **Title**: 
   ```
   Approval Required: Case {!$Record.CaseNumber}
   ```

8. **Body**: 
   ```
   Priority {!$Record.Priority} case needs approval
   ```

9. **Target Record Id**: {!$Record.Id}

10. Click **Done**

✅ **Checkpoint**: "Notify Approver" element inside loop

---

## **STEP 10: Save and Activate Flow**

### Save Flow:
1. Click **Save**
2. Enter Flow Details:
   ```
   Flow Label: Approval Notification Flow
   Flow API Name: Approval_Notification_Flow (auto-filled)
   Description: Sends emails and notifications to approvers when approval is submitted
   ```

3. Click **Save**

### Test Flow (Optional but Recommended):
4. Click **Debug** 
5. Set up test data to simulate approval submission
6. Run test and verify emails/notifications

### Activate Flow:
7. Click **Activate**
8. Confirm activation

✅ **Success!** Your flow is now active and will trigger on approval submissions

---

## **STEP 11: Testing Your Implementation**

### Test Process:
1. **Create a test case** in your org
2. **Submit the case for approval** using your approval process
3. **Check for emails**: Verify approvers receive emails
4. **Check for notifications**: Look for bell icon notifications in Salesforce
5. **Verify mobile notifications** (if using Salesforce mobile app)

### Expected Results:
- ✅ Approvers receive personalized emails with case details
- ✅ Approvers receive Salesforce notifications
- ✅ Emails contain direct links to the case
- ✅ Notifications appear in the bell icon (🔔)

---

## **Troubleshooting Common Issues**

### Email Not Sending:
- Check **Setup → Email Deliverability** (should be "All Email")
- Verify user has valid email address
- Check spam/junk folders

### Notifications Not Appearing:
- Verify Custom Notification Type is created correctly
- Check user notification preferences
- Ensure user has proper permissions

### Flow Not Triggering:
- Check if approval status field name is correct
- Verify entry criteria matches your approval process
- Check Flow debug logs for errors

---

## **What's Next?**

Once this basic version is working, you can enhance it by:
- Adding queue approver support
- Creating richer HTML email templates  
- Adding error handling and logging
- Monitoring flow performance

**Congratulations! 🎉** You now have a working approval notification system that will keep all approvers informed when cases are submitted for approval.