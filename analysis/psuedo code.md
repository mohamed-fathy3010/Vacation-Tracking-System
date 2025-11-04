# pseudo code

## employee: -

```
function accessVts()
link = selectLinkFromIntranet("VTS")
navigateTo(link)

function displayEmployeeStatus(employeeId)
employeeData = lookupCredentials(employeeId)
vacationStatus = getVacationStatus(employeeId)
displayRequests(vacationStatus, "Last 6 Months", "Next 18 Months")
displayBalances(vacationStatus)

function createNewRequest(employeeId)
availableCategories = filterPositiveBalances(getBalances(employeeId))

    if availableCategories is not empty then
        selectedCategory = promptEmployeeChoice(availableCategories)
    else
        displayError("No positive vacation time balances available.")
        return
    end if

    dateSelection = displayVisualCalendar()
    selectedDatesHours = promptDatesAndHours(dateSelection)
    title = promptTextInput("Short Title")
    description = promptTextInput("Description")

    newRequest = createRequestObject(employeeId, selectedCategory, selectedDatesHours, title, description)

    if validateRequest(newRequest) is false then
        displayVtsPage(newRequest, errors: true)

        action = promptEmployeeChoice("Change Information", "Cancel Request")
        if action is "Change Information" then
            createNewRequest(employeeId)
        else if action is "Cancel Request" then
            return
        end if
    else
        saveRequest(newRequest, status: "Pending Approval")

        if managerApprovalRequired(employeeId) is true then
            managerList = getAuthorizedManagers(employeeId)
            sendEmailNotification(managerList, subject: "New Vacation Request Pending")
        end if

        updateRequestState(newRequest, state: "Pending Approval")
        navigateTo("VTS Main Home Page")
    end if
```

==============================================

## manager: -

```
function managerAccessVts(managerId)
if accessMethod is "Email Link" or "Intranet Login" then
if authenticationRequired() then
authenticateUser(managerId)
end if
navigateTo("VTS Main Home Page")
end if

function displayManagerHome(managerId)
displayEmployeeStatus(managerId)

    pendingRequests = getSubordinateRequests(managerId, status: "Pending Approval")
    displaySection("Requests Pending My Approval", pendingRequests)

    selectedRequest = promptManagerSelection(pendingRequests)
    if selectedRequest is not null then
        reviewRequest(selectedRequest, managerId)
    end if

function reviewRequest(requestObject, managerId)
displayRequestDetails(requestObject)

    action = promptManagerChoice("Approve", "Disapprove")

    if action is "Approve" then
        finalState = "Approved"
        explanation = null
    else if action is "Disapprove" then
        finalState = "Rejected"
        explanation = promptTextInput("Reason for Disapproval", required: true)
    end if

    submitApprovalDecision(requestObject, managerId, action, explanation)
    updateRequestState(requestObject, state: finalState)

    employeeEmail = getEmployeeEmail(requestObject.employeeId)
    sendEmailNotification(employeeEmail, subject: "Vacation Request Status Update", status: finalState)

    navigateTo("VTS Main Home Page")
```
