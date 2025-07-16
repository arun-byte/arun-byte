# Troubleshooting: ProcessInstance Not Found

## 🚨 **Issue Identified**
Your flow is failing at the "Get Work Item" step because it can't find ProcessInstance records.

## 🔍 **Root Cause Analysis**

### The Problem:
```
GET RECORDS: Get Work Item
Find all ProcessInstance records where:
TargetObjectId Equals {!$Record.Id} (500FT00000DcTJGYA3)
AND Status Equals Pending
Result: Failed to find records.
```

## 🛠️ **Solution Steps**

### **Step 1: Verify Approval Process Status**

First, let's check what's actually happening with your case:

1. **Go to your Case record** (ID: 500FT00000DcTJGYA3)
2. **Look for "Approval History" related list**
3. **Check the current status** of the approval

### **Step 2: Query ProcessInstance Directly**

Let's see what ProcessInstance records exist:

1. **Open Developer Console** (Setup → Developer Console)
2. **Click "Query" tab**
3. **Run this query**:

```sql
SELECT Id, Status, TargetObjectId, ProcessDefinition.Name, CreatedDate 
FROM ProcessInstance 
WHERE TargetObjectId = '500FT00000DcTJGYA3'
ORDER BY CreatedDate DESC
```

**Expected Results:**
- If you see records with `Status = 'Pending'` → Flow should work
- If you see `Status = 'Approved'` or `'Rejected'` → Approval already completed
- If no records → Approval process never started

### **Step 3: Check ProcessInstanceWorkitem**

Also check the work items:

```sql
SELECT Id, ProcessInstanceId, ActorId, Actor.Name, ProcessInstance.Status 
FROM ProcessInstanceWorkitem 
WHERE ProcessInstance.TargetObjectId = '500FT00000DcTJGYA3'
```

## 🔧 **Fix Options**

### **Option 1: Adjust Flow Trigger Timing**

**Problem**: Flow might be running too early, before approval process creates ProcessInstance.

**Solution**: Change your flow trigger criteria:

```
Current Entry Criteria:
ApprovalStatus__c Equals "Submitted For Approval"

Change to:
ApprovalStatus__c Equals "Submitted For Approval"
AND
ISCHANGED(ApprovalStatus__c) = TRUE
```

### **Option 2: Add Multiple Status Check**

Update your Get Records element to check for multiple statuses:

**Current Condition:**
```
Status Equals Pending
```

**Change to:**
```
Status IN ('Pending', 'Started')
```

**How to do this:**
1. Edit your "Get Work Item" element
2. Change the Status condition:
   - **Operator**: Contains
   - **Value**: Pending,Started

### **Option 3: Add Wait Element**

Add a small delay to let the approval process complete:

1. **Add Wait element** before "Get Work Item"
2. **Configure Wait**:
   - **After**: 30 seconds
   - **Or add condition**: Until ProcessInstance exists

### **Option 4: Enhanced Error Handling**

Add better error handling to your flow:

#### **A. Add Decision After Get Work Item:**

```
Element: Decision
Label: Check ProcessInstance Found
Outcome: ProcessInstance Found
Condition: {!Get_Work_Item} Is Null Equals {!$GlobalConstant.False}
```

#### **B. Add Debug Email for "Not Found" Path:**

```
Element: Send Email (on "Default Outcome" path)
To: your-admin-email@company.com
Subject: DEBUG - No ProcessInstance Found
Body: 
Case ID: {!$Record.Id}
Case Number: {!$Record.CaseNumber}
Approval Status: {!$Record.ApprovalStatus__c}
Flow Run Time: {!$Flow.CurrentDateTime}
```

## 🎯 **Recommended Fix (Quick)**

Here's the fastest fix to try:

### **Update Your Get Work Item Element:**

1. **Edit the "Get Work Item" element**
2. **Change the conditions to:**

```
Find all ProcessInstance records where:
TargetObjectId Equals {!$Record.Id}
AND Status IN ('Pending', 'Started', 'Active')
```

3. **Add sort order:**
```
Sort by: CreatedDate Descending
```

4. **Save and test again**

## 🔄 **Alternative Approach: Different Object Query**

If ProcessInstance still doesn't work, try using ProcessInstanceWorkitem directly:

### **Replace "Get Work Item" with:**

```
Element: Get Records
Label: Get Current Work Items
Object: ProcessInstanceWorkitem
Conditions:
- ProcessInstance.TargetObjectId Equals {!$Record.Id}
- ProcessInstance.Status Equals Pending
Store: All records
```

## 📊 **Testing Steps**

### **Test Sequence:**
1. **Create a new test case**
2. **Submit for approval manually** (not through flow)
3. **Wait 1-2 minutes** for approval process to complete setup
4. **Check ProcessInstance exists** using the query above
5. **Then trigger your flow** by updating ApprovalStatus__c

## 🚀 **Quick Validation Query**

Run this to see exactly what's happening:

```sql
SELECT 
    c.Id,
    c.CaseNumber,
    c.ApprovalStatus__c,
    pi.Id ProcessInstanceId,
    pi.Status ProcessStatus,
    pi.CreatedDate,
    piw.Id WorkItemId,
    piw.ActorId,
    u.Name ApproverName
FROM Case c
LEFT JOIN ProcessInstance pi ON pi.TargetObjectId = c.Id
LEFT JOIN ProcessInstanceWorkitem piw ON piw.ProcessInstanceId = pi.Id
LEFT JOIN User u ON u.Id = piw.ActorId
WHERE c.Id = '500FT00000DcTJGYA3'
ORDER BY pi.CreatedDate DESC
```

This will show you exactly what approval data exists for your case.

## ✅ **Next Steps**

1. **Run the validation query** to see current state
2. **Try Option 1** (adjust status conditions) first
3. **Test with a fresh case** if needed
4. **Let me know the results** of the validation query

**Which approach would you like to try first?** I can provide more specific instructions based on what the validation query shows.