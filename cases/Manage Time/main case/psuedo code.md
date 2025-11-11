# pseudo code

## Workflow Configuration

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

## Workflow Helper Functions

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

## employee: -

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

==============================================

## approver (manager, HR, finance, etc.): -

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
