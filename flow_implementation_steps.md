# Flow Implementation Steps - Add Approval URL & Comments

## 🚀 **Exact Flow Elements to Add**

### **Step 1: Add Variable for Approval URL**

1. **In Flow Builder, click "New Resource"**
2. **Select "Variable"**
3. **Configure:**
   ```
   Resource Type: Variable
   API Name: varApprovalURL
   Data Type: Text
   Available for Input: ✅
   Available for Output: ✅
   ```

### **Step 2: Add Get Records - Submitter Details**

1. **Drag "Get Records" element to your flow**
2. **Configure:**
   ```
   Label: Get Submitter Details
   API Name: Get_Submitter_Details
   Object: User
   
   Filter Conditions:
   - Field: Id
   - Operator: Equals
   - Value: {!Get_Work_Item.SubmittedById}
   
   How Many Records: Only the first record
   Store Record Data: Automatically store all fields
   ```

### **Step 3: Add Get Records - Submitter Comments**

1. **Drag "Get Records" element to your flow**
2. **Configure:**
   ```
   Label: Get Submitter Comments
   API Name: Get_Submitter_Comments
   Object: ProcessInstanceHistory
   
   Filter Conditions:
   - Field: ProcessInstanceId
   - Operator: Equals
   - Value: {!Get_Work_Item.Id}
   
   AND
   
   - Field: StepStatus
   - Operator: Equals
   - Value: Started
   
   Sort Order: CreatedDate Ascending
   How Many Records: Only the first record
   Store Record Data: Automatically store all fields
   ```

### **Step 4: Add Assignment - Build Approval URL (Inside Loop)**

1. **Drag "Assignment" element inside your loop**
2. **Configure:**
   ```
   Label: Build Approval URL
   API Name: Build_Approval_URL
   
   Assignment:
   - Variable: {!varApprovalURL}
   - Operator: Equals
   - Value: https://vetrotech--devgdi.sandbox.my.salesforce.com/p/process/ProcessInstanceWorkitemWizardStageManager?id={!Loop_Current_Approvers.Id}
   ```

### **Step 5: Update Your Send Email Element**

1. **Edit your existing "Send Email" element**
2. **Update the Body field:**

```
Hello {!Get_Approver_Details.Name}

A Case Number {!$Record.CaseNumber} has been assigned for Approval by {!Get_Submitter_Details.Name} with Comment "{!Get_Submitter_Comments.Comments}"

📋 Case Details:
• Priority: {!$Record.Priority}
• Subject: {!$Record.Subject}
• Created: {!$Record.CreatedDate}

💬 Submitter Comments:
"{!Get_Submitter_Comments.Comments}"

🔗 Approval Link:
{!varApprovalURL}

Direct Case Link:
https://vetrotech--devgdi.sandbox.my.salesforce.com/{!$Record.Id}

Thanks & Regards
CS Team
```

## 📋 **Complete Flow Order**

Your flow should look like this:

```
1. Start (Record-Triggered, Async)
2. Get Work Item (ProcessInstance)
3. Get Submitter Details (User) ← NEW
4. Get Submitter Comments (ProcessInstanceHistory) ← NEW
5. Get Current Work Items (ProcessInstanceWorkitem)
6. Loop Through Work Items
   ├── Get Approver Details (User)
   ├── Build Approval URL (Assignment) ← NEW
   ├── Send Email (Updated body) ← UPDATED
   └── Send Notification
```

## 🛠️ **Flow Builder Configuration Steps**

### **A. Add the New Elements:**

1. **Open your existing flow**
2. **After "Get Work Item", add "Get Submitter Details"**
3. **After that, add "Get Submitter Comments"** 
4. **Keep your existing loop structure**
5. **Inside the loop, before "Send Email", add "Build Approval URL"**
6. **Update the "Send Email" element with new body**

### **B. Connect the Elements:**

```
Get Work Item → Get Submitter Details → Get Submitter Comments → Get Current Work Items → Loop...
```

### **C. Inside the Loop:**

```
Get Approver Details → Build Approval URL → Send Email → Send Notification
```

## 🎯 **Specific Field Configurations**

### **Get Submitter Details Filter:**
```
Resource: {!Get_Work_Item.SubmittedById}
Operator: Equals
Value: Field from Get_Work_Item record
```

### **Get Submitter Comments Filter:**
```
Row 1:
- Resource: {!Get_Work_Item.Id}
- Operator: Equals 
- Field: ProcessInstanceId

Row 2: 
- Resource: Started
- Operator: Equals
- Field: StepStatus
```

### **Build Approval URL Assignment:**
```
Variable: {!varApprovalURL}
Operator: Equals
Value: https://vetrotech--devgdi.sandbox.my.salesforce.com/p/process/ProcessInstanceWorkitemWizardStageManager?id={!Loop_Current_Approvers.Id}
```

## ⚡ **Quick Implementation Checklist**

- [ ] Create varApprovalURL variable
- [ ] Add "Get Submitter Details" element after Get Work Item
- [ ] Add "Get Submitter Comments" element after Submitter Details
- [ ] Add "Build Approval URL" assignment inside the loop
- [ ] Update "Send Email" body with new template
- [ ] Test flow to verify all fields populate

## 🔧 **Troubleshooting**

**If Submitter Details is blank:**
- Check Get_Work_Item.SubmittedById field is available
- Verify User object permissions

**If Comments are blank:**
- ProcessInstanceHistory might not have Started status
- Try StepStatus = "Submit" or remove StepStatus filter

**If Approval URL doesn't work:**
- Verify Loop_Current_Approvers.Id is the WorkItem ID
- Check your Salesforce instance URL

## 🚀 **Test Your Changes**

1. **Save the flow**
2. **Submit a case for approval**
3. **Check email contains:**
   - ✅ Approver name
   - ✅ Submitter name  
   - ✅ Submitter comments
   - ✅ Working approval URL
   - ✅ Case details

That's the exact Flow configuration you need! Which element would you like me to explain in more detail? 🎯