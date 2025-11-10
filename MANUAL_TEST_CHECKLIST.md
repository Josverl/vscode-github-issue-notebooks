# Manual Test Checklist for Advanced Search API Migration

This document provides a comprehensive checklist for manually testing the advanced search API migration. These tests require a live GitHub API connection and should be performed in a VS Code environment with the extension installed.

## Prerequisites

- [ ] VS Code with the GitHub Issue Notebooks extension installed
- [ ] GitHub authentication configured
- [ ] Access to public and private repositories for testing

## Test Cases

### 1. Simple Queries

#### Test 1.1: Basic repository search
```
repo:microsoft/vscode is:issue
```
**Expected**: Returns open issues from the microsoft/vscode repository
**Verify**: 
- [ ] Query executes without errors
- [ ] Results are displayed correctly
- [ ] No deprecation warnings in the debug console

#### Test 1.2: Issue with label filter
```
repo:microsoft/vscode is:issue label:bug
```
**Expected**: Returns issues with the "bug" label
**Verify**: 
- [ ] Only issues with bug label are returned
- [ ] No deprecation warnings

#### Test 1.3: Pull request search
```
repo:microsoft/vscode is:pr is:open
```
**Expected**: Returns open pull requests
**Verify**: 
- [ ] Only pull requests are returned
- [ ] No deprecation warnings

### 2. Complex Queries with OR Operators

#### Test 2.1: Multiple repositories with OR
```
repo:microsoft/vscode OR repo:microsoft/TypeScript is:issue is:open
```
**Expected**: Returns open issues from either repository
**Verify**: 
- [ ] Issues from both repositories are returned
- [ ] OR operator works correctly
- [ ] No deprecation warnings

#### Test 2.2: Multiple labels with OR
```
repo:microsoft/vscode is:issue (label:bug OR label:feature-request)
```
**Expected**: Returns issues with either bug or feature-request label
**Verify**: 
- [ ] Issues match the label criteria
- [ ] Parentheses grouping works correctly
- [ ] No deprecation warnings

### 3. Queries with Variables

#### Test 3.1: Variable definition and usage
```
$myrepo=repo:microsoft/vscode is:issue
$myrepo is:open
```
**Expected**: Variable is defined and used correctly
**Verify**: 
- [ ] Variable definition cell executes
- [ ] Variable usage cell executes with correct query
- [ ] Results match the expected query
- [ ] No deprecation warnings

#### Test 3.2: Multiple variables
```
$repo=repo:microsoft/vscode
$state=is:open
$repo $state is:issue
```
**Expected**: Multiple variables are combined correctly
**Verify**: 
- [ ] All variables are resolved
- [ ] Final query executes correctly
- [ ] No deprecation warnings

### 4. Queries with Sorting

#### Test 4.1: Sort by comments ascending
```
repo:microsoft/vscode is:issue sort:comments-asc
```
**Expected**: Issues sorted by comment count (low to high)
**Verify**: 
- [ ] Results are sorted correctly
- [ ] Sort parameter is applied
- [ ] No deprecation warnings

#### Test 4.2: Sort by updated descending
```
repo:microsoft/vscode is:issue sort:updated-desc
```
**Expected**: Issues sorted by update date (newest first)
**Verify**: 
- [ ] Results are sorted correctly
- [ ] No deprecation warnings

#### Test 4.3: Sort by created
```
repo:microsoft/vscode is:issue sort:created-asc
```
**Expected**: Issues sorted by creation date (oldest first)
**Verify**: 
- [ ] Results are sorted correctly
- [ ] No deprecation warnings

### 5. Queries with @me Placeholder

#### Test 5.1: Assigned to me
```
is:issue assignee:@me is:open
```
**Expected**: Returns issues assigned to the authenticated user
**Verify**: 
- [ ] Only issues assigned to current user are returned
- [ ] @me placeholder is resolved correctly
- [ ] No deprecation warnings

#### Test 5.2: Created by me
```
is:issue author:@me
```
**Expected**: Returns issues created by the authenticated user
**Verify**: 
- [ ] Only issues authored by current user are returned
- [ ] No deprecation warnings

### 6. Error Handling

#### Test 6.1: Invalid query syntax
```
repo: is:issue
```
**Expected**: Appropriate error message
**Verify**: 
- [ ] Error is displayed clearly
- [ ] No crash or hang
- [ ] Error message is helpful

#### Test 6.2: Non-existent repository
```
repo:nonexistent/repository12345 is:issue
```
**Expected**: Empty results or appropriate message
**Verify**: 
- [ ] Handles gracefully
- [ ] No crash
- [ ] Clear feedback to user

#### Test 6.3: Rate limiting
**Note**: This test may require making many requests
**Expected**: Appropriate rate limit message if triggered
**Verify**: 
- [ ] Rate limit message is clear
- [ ] Suggests authentication if anonymous
- [ ] No crash

### 7. Authentication Scenarios

#### Test 7.1: Unauthenticated with @me
```
is:issue assignee:@me
```
**Expected**: Error message prompting to log in
**Verify**: 
- [ ] Clear message about needing authentication
- [ ] Link/button to authenticate is provided
- [ ] No crash

#### Test 7.2: After authentication
**Action**: Log in with GitHub authentication, then re-run queries
**Verify**: 
- [ ] Higher rate limits apply
- [ ] @me queries work
- [ ] Private repository access works (if applicable)

### 8. Pagination

#### Test 8.1: Large result set
```
repo:microsoft/vscode is:issue
```
**Expected**: Handles pagination automatically (up to 1000 results)
**Verify**: 
- [ ] All results are fetched
- [ ] No duplicate results
- [ ] Performance is acceptable
- [ ] No deprecation warnings

### 9. Advanced Search Specific Tests

#### Test 9.1: Complex AND/OR query
```
repo:microsoft/vscode is:issue (label:bug OR label:feature-request) state:open
```
**Expected**: Advanced search handles complex query
**Verify**: 
- [ ] Query executes correctly
- [ ] Results match expected criteria
- [ ] No deprecation warnings

#### Test 9.2: Nested parentheses
```
repo:microsoft/vscode is:issue (label:bug OR (label:feature-request AND assignee:@me))
```
**Expected**: Nested logic is evaluated correctly
**Verify**: 
- [ ] Query executes
- [ ] Results are correct
- [ ] No errors

### 10. Deprecation Warning Check

#### Test 10.1: Open VS Code Developer Tools
**Action**: 
1. Open Command Palette (Ctrl/Cmd+Shift+P)
2. Run "Developer: Toggle Developer Tools"
3. Go to Console tab
4. Run any search query
5. Check for deprecation warnings

**Verify**: 
- [ ] No deprecation warnings about "/search/issues" endpoint
- [ ] No warnings about "issuesAndPullRequests" being deprecated
- [ ] Only normal API logs (if any)

## Performance Benchmarks

### Baseline Measurements
Record query execution times for comparison:

| Query Type | Result Count | Time (ms) | Notes |
|------------|--------------|-----------|-------|
| Simple query | | | |
| OR query | | | |
| Large result set | | | |
| With sorting | | | |

**Expected**: Performance should be similar to previous implementation

## Post-Test Validation

- [ ] All test cases executed
- [ ] No deprecation warnings observed
- [ ] All functionality working as expected
- [ ] Performance is acceptable
- [ ] Error handling is appropriate
- [ ] Documentation is accurate

## Issues Found

If any issues are discovered during testing, document them here:

1. **Issue**: 
   - **Test Case**: 
   - **Description**: 
   - **Expected**: 
   - **Actual**: 
   - **Severity**: 

## Sign-off

- **Tester**: _______________
- **Date**: _______________
- **Result**: [ ] Pass [ ] Fail (with issues documented)
- **Notes**: 

## References

- [GitHub Search Syntax](https://docs.github.com/en/search-github/searching-on-github/searching-issues-and-pull-requests)
- [Advanced Search Documentation](https://github.blog/changelog/2025-03-06-github-issues-projects-api-support-for-issues-advanced-search-and-more/)
- Migration Plan: `MIGRATION_ADVANCED_SEARCH.md`
