# Fixed Approval Flow - ProcessInstance Issues Resolved

## 🚨 **Issue Found:** Wrong Field in ProcessInstanceStep Query

You're using `Id Equals {!Get_Work_Item.Id}` but should use `ProcessInstanceId Equals {!Get_Work_Item.Id}`

## 🛠️ **Immediate Fix**

### **Fix Your ProcessInstanceStep Query:**

**Current (Wrong):**
```
Find all ProcessInstanceStep records where:
Id Equals {!Get_Work_Item.Id}
```

**Change to (Correct):**
```
Find all ProcessInstanceStep records where:
ProcessInstanceId Equals {!Get_Work_Item.Id}
```

### **Complete Working Flow Configuration:**

## 📋 **Step-by-Step Working Flow**

### **Element 1: Get Work Item (Keep as is - it's working!)**
```
Element: Get Records
Label: Get Work Item  
Object: ProcessInstance
Conditions:
- TargetObjectId Equals {!$Record.Id}
- Status Equals Started
Store: Only the first record
```

### **Element 2: Get Approval Steps (Fixed)**
```
Element: Get Records
Label: Get Approval Steps
Object: ProcessInstanceStep
Conditions:
- ProcessInstanceId Equals {!Get_Work_Item.Id}
Store: All records
```

### **Element 3: Loop Through Steps**
```
Element: Loop
Label: Loop Through Steps
Collection: {!Get_Approval_Steps}
```

### **Element 4: Get Current Workitems (Inside Loop)**
```
Element: Get Records
Label: Get Current Workitems
Object: ProcessInstanceWorkitem
Conditions:
- ProcessInstanceId Equals {!Get_Work_Item.Id}
- ProcessInstanceStepId Equals {!Loop_Through_Steps.Id}
Store: All records
```

### **Element 5: Loop Through Workitems (Nested Loop)**
```
Element: Loop
Label: Loop Through Workitems  
Collection: {!Get_Current_Workitems}
```

### **Element 6: Get Approver Details (Inside Nested Loop)**
```
Element: Get Records
Label: Get Approver Details
Object: User
Conditions:
- Id Equals {!Loop_Through_Workitems.ActorId}
Store: Only the first record
```

### **Element 7: Send Email (Inside Nested Loop)**
```
Element: Send Email
To: {!Get_Approver_Details.Email}
Subject: Approval Required: {!$Record.CaseNumber}
Body: [Your email template]
```

### **Element 8: Send Notification (Inside Nested Loop)**
```
Element: Send Custom Notification
Recipients: {!Get_Approver_Details.Id}
Title: Approval Required: {!$Record.CaseNumber}
Body: Case needs approval
```

## 🚀 **Simplified Alternative (Recommended)**

If the above is still complex, here's a simpler working approach:

### **Option A: Query ProcessInstanceHistory**
```
Element: Get Records
Label: Get Pending Approvals
Object: ProcessInstanceHistory
Conditions:
- ProcessInstanceId Equals {!Get_Work_Item.Id}
- StepStatus Equals Pending
Store: All records
```

### **Option B: Direct Query Approach**

**Replace everything after "Get Work Item" with:**

```
Element: Get Records
Label: Get All Pending Work
Object: ProcessInstanceWorkitem
Conditions:
- ProcessInstance.TargetObjectId Equals {!$Record.Id}
- ProcessInstance.Status Equals Started
Store: All records
```

Then loop through this collection directly.

## 🔍 **Debug Queries to Run**

Run these in Developer Console to see what's available:

### **Query 1: Check ProcessInstanceStep**
```sql
SELECT Id, ProcessInstanceId, StepStatus, Comments, ActorId, Actor.Name
FROM ProcessInstanceStep 
WHERE ProcessInstanceId = '04gFT000001M7n3YAC'
```

### **Query 2: Check ProcessInstanceWorkitem**
```sql
SELECT Id, ProcessInstanceId, ProcessInstanceStepId, ActorId, Actor.Name, OriginalActorId
FROM ProcessInstanceWorkitem 
WHERE ProcessInstanceId = '04gFT000001M7n3YAC'
```

### **Query 3: Check ProcessInstanceHistory**
```sql
SELECT Id, ProcessInstanceId, StepStatus, Actor.Name, ActorId, Comments, CreatedDate
FROM ProcessInstanceHistory 
WHERE ProcessInstanceId = '04gFT000001M7n3YAC'
ORDER BY CreatedDate DESC
```

### **Query 4: Complete Approval Picture**
```sql
SELECT 
    pi.Id ProcessInstanceId,
    pi.Status,
    pi.TargetObjectId,
    pis.Id StepId,
    pis.StepStatus,
    piw.Id WorkitemId,
    piw.ActorId,
    u.Name ApproverName
FROM ProcessInstance pi
LEFT JOIN ProcessInstanceStep pis ON pis.ProcessInstanceId = pi.Id
LEFT JOIN ProcessInstanceWorkitem piw ON piw.ProcessInstanceId = pi.Id
LEFT JOIN User u ON u.Id = piw.ActorId
WHERE pi.Id = '04gFT000001M7n3YAC'
```

## ⚡ **Quick Working Solution (Copy-Paste)**

**Here's a simplified flow that should work immediately:**

### **After your "Get Work Item" element, replace everything with:**

1. **Get Records:**
   ```
   Label: Get All Current Approvers
   Object: ProcessInstanceWorkitem
   Conditions:
   - ProcessInstance.TargetObjectId Equals {!$Record.Id}
   Store: All records
   ```

2. **Loop:**
   ```
   Collection: {!Get_All_Current_Approvers}
   ```

3. **Decision (Inside Loop):**
   ```
   Check if ActorId starts with "005" (User) or "00G" (Queue)
   ```

4. **Get User Details (User path):**
   ```
   Object: User
   Conditions: Id Equals {!Loop_Approvers.ActorId}
   ```

5. **Send Email and Notification**

## 🎯 **Action Plan:**

1. **Run Query 4 above** - tell me what it returns
2. **Try the Quick Working Solution** 
3. **If that fails, try Option A (ProcessInstanceHistory)**

**Which approach would you like to try first?** The "Quick Working Solution" should bypass all the step complexity and get approvers directly.

Let me know what Query 4 returns and I'll give you the exact working configuration! 🚀