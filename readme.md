# Vacation Tracking System

## Vision

A Vacation Tracking System (VTS) will provide individual employees with the capability to manage their own vacation time, sick leave, and personal time off, without having to be an expert in company policy or the local facility’s leave policies

## Functional Requirements

- Implements a flexible rules-based system for validating and verifying leave time requests
- Enables manager approval (optional)
- Provides access to requests for the previous calendar year, and allows requests to be made up to a year and a half in the future
- Uses e-mail notification to request manager approval and notify employees of request status changes
- Uses existing hardware and middleware
- Is implemented as an extension to the existing intranet portal system, and
- uses the portal’s single-sign-on mechanisms for all authentication
- Keeps activity logs for all transactions
- Enables the HR and system administration personnel to override all actions restricted by rules, with logging of those overrides
- Allows managers to directly award personal leave time (with system-set limits)
- Provides a Web service interface for other internal systems to query any given employee’s vacation request summary
- Interfaces with the HR department legacy systems to retrieve required employee information and changes

## Non Functional Requirements

- Streamlines the functions of the human resources (HR) department
- Minimizes noncore, business-related activities of management
- Gives a senseof empowerment to the employees
- The system must be easy to use.

## Constraints

- Integrations
- Legacy Hardware
- Single-sign-on
- User experience

## Assumptions

- employees and managers should use the same home screen

## Domain

Human Resource Managament

## Analysis Artifacts

The following analysis artifacts are for the use case: **Manage Time**

**Use Case Name:** Manage Time  
**Actor:** Employee  
**Goal:** The employee wishes to submit a new request for vacation time.  
**Preconditions:** The employee is authenticated by the portal framework and identified as an employee of the company with privileges to manage his or her own vacation time.  
**Main flow:**

### Flow Diagram

![Flow Diagram](cases/Manage%20Time/main%20case/flow%20diagram.drawio.svg)

### ERD (Entity Relationship Diagram)

![ERD](cases/Manage%20Time/ERD.drawio.svg)

### Sequential Diagram

![Sequential Diagram](cases/Manage%20Time/main%20case/sequential%20diagram.drawio.svg)

### Page Design

![Page Design](cases/Manage%20Time/page%20design.drawio.svg)

### State Diagram

![State Diagram](cases/Manage%20Time/State%20Diagram.drawio.svg)

### Pseudo Code

#### Workflow Configuration

```
// Request Status Enum
enum RequestStatus {
    DRAFT, PENDING_MANAGER_APPROVAL, PENDING_HR_APPROVAL,
    PENDING_FINANCE_APPROVAL, APPROVED, REJECTED, WITHDRAWN, CANCELED
}

// Workflow Structure: Each workflow has stages with approval roles
structure WorkflowDefinition {
    workflowId: string
    stages: [{stageOrder, roleName, pendingStatus, approvedNextStage}]
    conditions: {employeeLevels, leaveTypes, minDuration, maxDuration}
}

// Example: Standard workflow (Manager only), HR workflow (Manager→HR), Finance workflow (Manager→HR→Finance)
// Workflows are selected based on employee level, leave type, and duration
```

#### Workflow Helper Functions

```
function determineWorkflow(employeeId, leaveType, durationHours)
    // Select workflow based on employee level, leave type, and duration
    // Returns workflow definition with approval stages
end function

function processApprovalStage(requestObject, approverId, approverRole, approved, explanation)
    // Save approval record
    // Determine next stage using workflow definition
    // If next stage exists: move to next stage and notify approvers
    // If final stage: complete workflow and notify employee
    // Return final status
end function
```

#### Employee Functions

```
function accessVts()
    link = selectLinkFromIntranet("VTS")
    navigateTo(link)
end function

function displayEmployeeStatus(employeeId)
    employeeData = lookupCredentials(employeeId)
    vacationStatus = getVacationStatus(employeeId)
    displayRequests(vacationStatus, "Last 6 Months", "Next 18 Months")
    displayBalances(vacationStatus)
end function

function createNewRequest(employeeId)
    availableCategories = filterPositiveBalances(getBalances(employeeId))

    if availableCategories is empty then
        displayError("No positive vacation time balances available.")
        return
    end if

    selectedCategory = promptEmployeeChoice(availableCategories)
    selectedDatesHours = promptDatesAndHours(displayVisualCalendar())
    title = promptTextInput("Short Title")
    description = promptTextInput("Description")

    newRequest = createRequestObject(employeeId, selectedCategory, selectedDatesHours, title, description)

    if validateRequest(newRequest) is false then
        displayVtsPage(newRequest, errors: true)
        action = promptEmployeeChoice("Change Information", "Cancel Request")
        if action is "Change Information" then
            createNewRequest(employeeId)
        end if
        return
    end if

    // Determine workflow and initiate first approval stage
    totalHours = calculateTotalHours(selectedDatesHours)
    workflow = determineWorkflow(employeeId, selectedCategory, totalHours)
    firstStage = workflow.stages[0]

    newRequest.workflowId = workflow.workflowId
    newRequest.currentStageOrder = firstStage.stageOrder
    saveRequest(newRequest, status: firstStage.pendingStatus)

    // Notify first approvers
    approverList = getApproversForRole(firstStage.roleName, employeeId)
    sendEmailNotification(approverList, "New Vacation Request Pending Your Approval", newRequest)

    navigateTo("VTS Main Home Page")
end function
```

#### Approver Functions (Manager, HR, Finance, etc.)

```
function approverAccessVts(approverId, approverRole)
    authenticateUser(approverId)
    navigateTo("VTS Main Home Page")
end function

function displayApproverHome(approverId, approverRole)
    displayEmployeeStatus(approverId)

    pendingRequests = getPendingRequestsForApprover(approverId, approverRole)
    displaySection("Requests Pending My Approval (" + approverRole + ")", pendingRequests)

    selectedRequest = promptApproverSelection(pendingRequests)
    if selectedRequest is not null then
        reviewRequest(selectedRequest, approverId, approverRole)
    end if
end function

function reviewRequest(requestObject, approverId, approverRole)
    displayRequestDetails(requestObject)
    displayApprovalHistory(requestObject.id)
    displayWorkflowProgress(requestObject)

    action = promptApproverChoice("Approve", "Disapprove")
    approved = (action is "Approve")
    explanation = promptTextInput(approved ? "Comments (Optional)" : "Reason for Disapproval", required: !approved)

    // Process approval through workflow engine
    finalStatus = processApprovalStage(requestObject, approverId, approverRole, approved, explanation)

    displayMessage("Request " + (approved ? "approved" : "rejected") + ". New status: " + finalStatus)
    navigateTo("VTS Main Home Page")
end function
```

## Use Case: Withdraw Request (Alternate Flow)

**Goal:** The employee wants to withdraw an outstanding request for vacation time.

**Preconditions:**

- An employee has made a vacation time request, and that request has yet to be approved or denied by an authorized manager.
- See also main flow preconditions.

### Flow Diagram

![Withdraw Request Flow Diagram](cases/Manage%20Time/Withdrawal%20case/Flow%20Diagram.drawio.svg)

### Sequence Diagram

![Withdraw Request Sequence Diagram](cases/Manage%20Time/Withdrawal%20case/Sequence%20Diagram.drawio.svg)

**Pseudo Code:**

1. Employee navigates to the VTS home page through the intranet portal application (employee is authenticated and authorized).
2. Display summary of vacation time requests, outstanding balances per category, and status of all active vacation time requests (previous 6 months to 18 months in the future).
3. Employee selects a vacation time request to withdraw (must be pending approval).
4. Prompt employee to confirm withdrawal of the selected request.
5. If employee confirms:
   - Remove the request from the manager’s list of pending approvals.
   - Send notification e-mail to the manager.
   - Update the request state to 'withdrawn'.

## Use Case: Cancel Approved Request (Alternate Flow)

**Goal:** The employee wants to cancel an approved vacation time request.

**Preconditions:**

- The employee has a vacation time request that has been approved and is scheduled for some time in the future or the recent past (previous 5 business days).
- See also main flow preconditions.

### Cancel Request Flowchart

![Cancel Request Flowchart](cases/Manage%20Time/cancel%20approved%20case/Cancel%20request%20flowchart.drawio.svg)

### Sequential Diagram

![Cancel Request Sequential Diagram](cases/Manage%20Time/cancel%20approved%20case/sequential%20diagram.drawio.svg)

**Pseudo Code:**

1. Employee navigates to the VTS home page through the intranet portal application (employee is authenticated and authorized).
2. Display summary of vacation time requests, outstanding balance per category, and status of all active vacation time requests (previous 6 months to 18 months in the future).
3. Employee selects a vacation time request to cancel (must be approved and in the future or recent past).
4. If the request is in the future:
   - Prompt employee to confirm cancellation.
   - If confirmed, send notification e-mail to the manager, change request state to 'canceled', and return time allowances to the employee.
   - If aborted, make no changes.
     If the request is in the recent past:
   - Prompt employee to confirm cancellation and provide a short explanation.
   - If confirmed and explanation provided, send notification e-mail to the manager, change request state to 'canceled', and return time allowances to the employee.
   - If aborted, make no changes.
5. Return employee to the main VTS home page and update summaries to reflect any changes made to outstanding vacation time requests.

## Use Case: Edit Pending Request (Alternate Flow)

**Goal:** The employee wants to edit the description or title of a pending request.

**Preconditions:**

- An employee has made a vacation time request, and that request has yet to be approved or denied by an authorized manager.
- See also main flow preconditions.

### Edit Case Flow Diagram

![Edit Case Flow Diagram](cases/Manage%20Time/Edit%20case/Edit%20case%20flow%20diagram.drawio.svg)

### Edit Case Sequential Diagram

![Edit Case Sequential Diagram](cases/Manage%20Time/Edit%20case/edit%20case%20squential%20diagram.drawio.svg)

**Pseudo Code:**

1. The employee navigates to the VTS home page through the intranet portal application, which identifies and authenticates the employee with the privileges necessary for using the VTS.
2. The VTS home page contains a summary of vacation time requests, outstanding balances per category of time, and the current status of all active vacation time requests for the previous 6 months and up to 18 months in the future.
3. The employee selects a request to edit, one that is pending approval.
4. The VTS displays an editable view of the request. The employee is allowed to change the title, comments, or dates. The employee can also choose to delete or withdraw this request.
5. The employee changes request information and submits the changes to the system.
6. If the employee withdraws the request, the VTS prompts for confirmation before withdrawing the request. If changes are made only to the information, the changes are accepted, and the screen returns to the main VTS home page. If there are errors or problems with the information changes, the VTS redisplays the editing page and highlights and explains all problems.
