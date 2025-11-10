# Migration Plan: GitHub Advanced Search API

## Overview

This document outlines the migration plan from the deprecated GitHub Issues Search API to the new Advanced Search API. The old endpoint is scheduled for removal on **September 4, 2025**.

## Background

### Deprecation Notice

```
[octokit/request] "GET https://api.github.com/search/issues?q=..." is deprecated.
It is scheduled to be removed on Thu, 04 Sep 2025 00:00:00 GMT.
See https://github.blog/changelog/2025-10-06-github-issues-projects-api-support-for-issues-advanced-search-and-more/
```

### What's Changing

- GitHub is transitioning to Advanced Search as the default for issue/PR searches
- The old search endpoint will be removed entirely on September 4, 2025
- The new API requires the `advanced_search: true` parameter until it becomes the default
- Advanced Search provides more powerful querying with AND/OR operators and parentheses

### Impact

- **Current Code**: `src/extension/notebookProvider.ts` line 116 uses the deprecated API
- **Functionality**: All existing query functionality will be preserved
- **Breaking Changes**: None expected - the API endpoint remains the same, only a parameter is added

## Migration Steps

### Phase 1: Research and Planning
- [x] Research GitHub Advanced Search API documentation
- [x] Identify affected code locations
- [x] Understand the new API parameters and behavior
- [x] Create migration plan document
- [x] Run baseline tests to ensure current state

### Phase 2: Code Changes
- [ ] Update `src/extension/notebookProvider.ts` to add `advanced_search: true` parameter
- [ ] Review any other potential locations using the search API
- [ ] Update type definitions if needed

### Phase 3: Testing
- [ ] Create unit tests for the search functionality with advanced_search parameter
- [ ] Run existing unit tests to ensure no regressions
- [ ] Manually test various query types:
  - [ ] Simple queries (e.g., `repo:owner/name is:issue`)
  - [ ] Complex queries with OR operators
  - [ ] Queries with variables
  - [ ] Queries with sorting
  - [ ] Queries with @me placeholder
- [ ] Test error handling scenarios
- [ ] Test rate limiting scenarios

### Phase 4: Validation
- [ ] Run complete test suite
- [ ] Run linter to ensure code quality
- [ ] Build the extension
- [ ] Verify no new deprecation warnings

### Phase 5: Documentation
- [ ] Update README if necessary
- [ ] Add comments explaining the advanced_search parameter
- [ ] Document any behavior changes (if any)

## Technical Details

### Current Implementation

```typescript
const response = await octokit.rest.search.issuesAndPullRequests({
    q: queryData.q,
    sort: (<any>queryData.sort),
    order: queryData.order,
    per_page: 100,
    page,
    request: { signal: abortCtl.signal }
});
```

### Updated Implementation

```typescript
const response = await octokit.rest.search.issuesAndPullRequests({
    q: queryData.q,
    sort: (<any>queryData.sort),
    order: queryData.order,
    per_page: 100,
    page,
    advanced_search: true,  // Enable advanced search
    request: { signal: abortCtl.signal }
});
```

## Key Considerations

### Advanced Search Syntax Changes

1. **AND Operator**: Multiple filters separated by spaces are now treated as AND (previously OR)
   - Old: `repo:a repo:b` meant "a OR b"
   - New: `repo:a repo:b` means "a AND b"
   - To get OR: `repo:a OR repo:b`

2. **Nested Queries**: Can use parentheses for complex queries
   - Example: `is:issue AND (label:bug OR label:enhancement)`

3. **Backward Compatibility**: The current query syntax in this extension should remain compatible
   - The extension already uses OR explicitly in queries
   - Existing queries should work without modification

### Testing Strategy

1. **Unit Tests**: Test the API call with the new parameter
2. **Integration Tests**: Verify queries execute correctly
3. **Manual Testing**: Test various real-world scenarios
4. **Error Handling**: Ensure error messages remain clear

### Rollback Plan

If issues are discovered:
1. The change is minimal (one parameter addition)
2. Can be easily reverted by removing the `advanced_search: true` parameter
3. Old API continues to work until September 4, 2025

## Timeline

- **Immediate**: Complete migration to new API
- **Before Sept 4, 2025**: Required completion date
- **After Sept 4, 2025**: `advanced_search: true` becomes default and parameter becomes redundant

## Success Criteria

- [x] All existing unit tests pass
- [ ] No deprecation warnings in console
- [ ] All query types work as expected
- [ ] Code passes linting
- [ ] Extension builds successfully
- [ ] Documentation is updated

## References

- [GitHub Changelog: Advanced Search API](https://github.blog/changelog/2025-03-06-github-issues-projects-api-support-for-issues-advanced-search-and-more/)
- [Octokit Issue #2832](https://github.com/octokit/octokit.js/issues/2832)
- [GitHub Search Syntax Documentation](https://docs.github.com/en/search-github/searching-on-github/searching-issues-and-pull-requests)

## Notes

- The deprecation warning is currently unavoidable even with `advanced_search: true` until GitHub fully rolls out the change
- After September 4, 2025, the `advanced_search` parameter will no longer be needed
- This is a forward-compatible change that prepares for the future default behavior
