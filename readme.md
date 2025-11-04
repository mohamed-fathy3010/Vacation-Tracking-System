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

- An employee can’t take more than X consecutive days of leave for Y type of grant.
- Vacation time of type X cannot be taken when directly adjacent to a company or location-specific holiday.
- Vacation time of type X is limited to Y hours per week or month.
- Vacation time may not be granted when there are only X number of employees scheduled to work from the list Y of employees.
- Vacation time may not be granted on these dates: X.
- Vacation time of this type is limited to certain days of the week: {M, T, W, Th, F, Sat, Sun}

## Assumptions

- employees and managers should use the same home screen

## Domain

Human Resource Managament

## Analysis Artifacts

- [Flow Diagram](analysis/flow%20diagram.drawio.svg)
- [ERD (Entity Relationship Diagram)](analysis/ERD.drawio.svg)
- [Sequential Diagram](analysis/sequential%20diagram.drawio.svg)
- [Pseudo Code](analysis/psuedo%20code.md)
