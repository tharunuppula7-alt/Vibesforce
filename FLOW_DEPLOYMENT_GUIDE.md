# Get Available Time Slots Flow - Deployment & Agent Action Registration Guide

## Overview
This guide provides step-by-step instructions for deploying the **Get_Available_Time_Slots_Flow** and registering it as an Agent Action for the OncoGlobal Patient Scheduling Agent.

---

## Flow Summary

**Flow Name:** Get_Available_Time_Slots_Flow  
**Type:** Autolaunched Flow  
**Purpose:** Fetches all available appointment time slots for a given hospital, department or doctor, and date.

### Input Variables

| Variable Name | Type | Required | Description |
|---------------|------|----------|-------------|
| `patientProblem` | Text | No | What patient describes (e.g., "child has cancer") |
| `hospitalId` | Text | Yes | Healthcare Facility ID (Hospital__c) |
| `departmentId` | Text | No | Department__c ID (if known) |
| `doctorId` | Text | No | Doctor__c ID (if patient wants specific doctor) |
| `slotDate` | Date | Yes | Date patient wants appointment |

### Output Variables

| Variable Name | Type | Description |
|---------------|------|-------------|
| `responseMessage` | Text | Message shown to patient |
| `availableSlotsList` | Record Collection | All available Time_Slot__c records |
| `slotsFound` | Boolean | Whether slots were found |
| `slotsSummary` | Text | Formatted slot list for agent to display |

---

## Deployment Steps

### Step 1: Authenticate to Your Salesforce Org

```bash
# Login to your Salesforce org
sf org login web --alias MyOrg

# Or if already authenticated, set as default
sf config set target-org=MyOrg
```

### Step 2: Verify Flow File Exists

```bash
# Check if the flow file exists
ls force-app/main/default/flows/Get_Available_Time_Slots_Flow.flow-meta.xml
```

### Step 3: Deploy the Flow

```bash
# Deploy the flow to your org
sf project deploy start --source-dir force-app/main/default/flows/Get_Available_Time_Slots_Flow.flow-meta.xml

# Alternatively, deploy all flows
sf project deploy start --source-dir force-app/main/default/flows
```

### Step 4: Verify Deployment

```bash
# Check deployment status
sf project deploy report

# Or verify in Salesforce UI:
# Setup → Flows → Search for "Get Available Time Slots Flow"
```

---

## Agent Action Registration

### Option A: Register via Salesforce Setup UI (Recommended)

#### Step 1: Navigate to Agent Actions
1. Go to **Setup** in your Salesforce org
2. In the Quick Find box, search for **"Agent Actions"**
3. Click on **Agent Actions**

#### Step 2: Create New Agent Action
1. Click **New** button
2. Select **Flow** as the Action Type
3. Click **Next**

#### Step 3: Configure Agent Action Details

**Action Configuration:**
- **Flow:** Select `Get_Available_Time_Slots_Flow` from dropdown
- **Label:** `Get Available Time Slots`
- **API Name:** `Get_Available_Time_Slots` (auto-generated)
- **Description for AI:** 
  ```
  Fetches all available appointment time slots for a given hospital, department or doctor, and date. Call this when the patient has selected a date and wants to see available times. This action returns a formatted list of time slots with slot IDs, times, and doctor names that the patient can choose from.
  ```

**Required Input Parameters:**
- ✅ `hospitalId` - Mark as **Required**
- ✅ `slotDate` - Mark as **Required**

**Optional Input Parameters:**
- ⬜ `patientProblem` - Optional (context only)
- ⬜ `departmentId` - Optional (use when no specific doctor)
- ⬜ `doctorId` - Optional (use when patient wants specific doctor)

**Note:** Either `departmentId` OR `doctorId` should be provided, but the flow handles both cases.

#### Step 4: Configure AI Instructions (Advanced)

Add these AI-specific instructions to help the agent use this action correctly:

```
WHEN TO CALL THIS ACTION:
- Patient has selected or confirmed a hospital
- Patient has provided or confirmed a preferred date
- Patient wants to see available appointment times
- Patient asks "what times are available?"

HOW TO USE THE INPUTS:
- Always provide hospitalId and slotDate (required)
- If patient specified a doctor, provide doctorId
- If patient specified a department but no doctor, provide departmentId
- If neither doctor nor department specified, try to infer from patientProblem

WHAT TO DO WITH THE OUTPUT:
- Display the responseMessage to the patient
- Use slotsSummary to show formatted time slots
- If slotsFound is false, suggest alternative dates or hospitals
- Ask patient to select a Slot ID to proceed with booking
```

#### Step 5: Save and Activate
1. Click **Save**
2. Ensure the Agent Action is **Active**

---

### Option B: Register via Agent Builder

#### Step 1: Open Agent Builder
1. Go to **Setup** → **Agents**
2. Find and open **OncoGlobal Patient Scheduling Agent**
3. Click **Open in Agent Builder**

#### Step 2: Add Action to Agent Topics
1. In the Agent Builder, navigate to **Topics**
2. Find or create a topic for **"Book Appointment"** or **"Show Available Times"**
3. Click **Add Action**
4. Search for and select **"Get Available Time Slots"**

#### Step 3: Configure Topic Instructions
Add topic-specific instructions for when to use this action:

```
When a patient wants to book an appointment and needs to see available time slots:
1. Confirm hospital selection
2. Confirm preferred date
3. Call "Get Available Time Slots" action
4. Display the available slots to the patient
5. Ask patient to choose a slot ID
6. Proceed to booking confirmation
```

---

## Testing the Flow

### Test 1: Manual Flow Test (via Flow Builder)

1. Go to **Setup** → **Flows**
2. Open **Get Available Time Slots Flow**
3. Click **Debug** button
4. Set input values:
   - `hospitalId`: [Existing Hospital Record ID]
   - `slotDate`: [Future Date, e.g., 2026-05-01]
   - `departmentId`: [Existing Department ID] OR `doctorId`: [Existing Doctor ID]
5. Click **Run**
6. Verify:
   - `responseMessage` contains slot information
   - `slotsFound` is true (if slots exist)
   - `availableSlotsList` contains Time_Slot__c records

### Test 2: Test via Agent Conversation

1. Open the **OncoGlobal Patient Scheduling Agent**
2. Start a conversation: "I need to book an appointment"
3. Provide hospital and date information
4. Agent should automatically call the "Get Available Time Slots" action
5. Verify the response includes formatted time slots

### Test 3: Apex Test (Optional)

Create a test class to verify flow behavior:

```apex
@isTest
public class GetAvailableTimeSlotsFlowTest {
    @testSetup
    static void setup() {
        // Create test data
        Hospital__c hospital = new Hospital__c(Name = 'Test Hospital');
        insert hospital;
        
        Department__c dept = new Department__c(Name = 'Oncology');
        insert dept;
        
        Doctor__c doctor = new Doctor__c(
            Name = 'Dr. Test',
            Hospital__c = hospital.Id,
            Department__c = dept.Id,
            Specialty__c = 'Oncology'
        );
        insert doctor;
        
        Time_Slot__c slot = new Time_Slot__c(
            Healthcare_Facility__c = hospital.Id,
            Department__c = dept.Id,
            Doctor__c = doctor.Id,
            Slot_Date__c = Date.today().addDays(7),
            Start_Time__c = Time.newInstance(9, 0, 0, 0),
            End_Time__c = Time.newInstance(10, 0, 0, 0),
            Status__c = 'Available'
        );
        insert slot;
    }
    
    @isTest
    static void testGetSlotsWithDoctor() {
        Hospital__c hospital = [SELECT Id FROM Hospital__c LIMIT 1];
        Doctor__c doctor = [SELECT Id FROM Doctor__c LIMIT 1];
        Date slotDate = Date.today().addDays(7);
        
        // Prepare flow inputs
        Map<String, Object> inputs = new Map<String, Object>{
            'hospitalId' => hospital.Id,
            'doctorId' => doctor.Id,
            'slotDate' => slotDate
        };
        
        Test.startTest();
        // Invoke flow
        Flow.Interview flow = Flow.Interview.createInterview(
            'Get_Available_Time_Slots_Flow', 
            inputs
        );
        flow.start();
        
        // Get outputs
        Boolean slotsFound = (Boolean) flow.getVariableValue('slotsFound');
        String responseMessage = (String) flow.getVariableValue('responseMessage');
        Test.stopTest();
        
        // Assertions
        System.assertEquals(true, slotsFound, 'Should find available slots');
        System.assert(responseMessage.contains('available slots'), 'Response should mention available slots');
    }
}
```

---

## Troubleshooting

### Issue: Flow Not Found in Agent Actions
**Solution:** 
- Verify flow is deployed and active
- Refresh Agent Actions page
- Check flow status in Setup → Flows

### Issue: No Slots Returned
**Solution:**
- Verify Time_Slot__c records exist with Status = 'Available'
- Check that slot dates match the input date
- Verify Healthcare_Facility__c matches hospitalId
- Ensure Doctor__c or Department__c filters match

### Issue: Flow Fails on Execution
**Solution:**
- Check flow error logs in Setup → Flows → View Error Logs
- Verify all required fields on Time_Slot__c are populated
- Check object-level and field-level permissions

### Issue: Agent Doesn't Call the Action
**Solution:**
- Review Agent Action AI description - make it more specific
- Add explicit instructions in the agent topic
- Verify action is activated and assigned to the agent
- Test with more specific patient requests

---

## Next Steps

After successful deployment and registration:

1. ✅ **Create Sample Data**
   - Add Time_Slot__c records with Status='Available'
   - Ensure slots are linked to Doctors, Departments, and Hospitals

2. ✅ **Integrate with Booking Flow**
   - Create a companion flow for booking selected slots
   - Update slot status to 'Booked' after selection

3. ✅ **Monitor Performance**
   - Check Agent Action analytics
   - Review conversation logs
   - Gather patient feedback

4. ✅ **Enhance the Flow**
   - Add filtering by time of day
   - Include doctor specialties in output
   - Add slot duration in summary

---

## Related Documentation

- [Agent Actions Documentation](https://help.salesforce.com/s/articleView?id=sf.agent_actions_overview.htm)
- [Flow Builder Guide](https://help.salesforce.com/s/articleView?id=sf.flow.htm)
- [Agentforce Best Practices](https://help.salesforce.com/s/articleView?id=sf.agentforce_best_practices.htm)

---

**Version:** 1.0  
**Last Updated:** April 28, 2026  
**Maintained By:** Onco Global Development Team