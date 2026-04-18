# Article Lifecycle Test Cases

## Scope

This document defines black-box test cases derived only from repository specification files for the article lifecycle feature family.

### Source specifications
- `specification/features/article-lifecycle.md`
- `specification/features/authorization-ownership.md`

## Assumptions and notation

- The system supports authenticated and unauthenticated user interactions.
- Article retrieval is available after creation and update.
- If a specification states an exact status code, it is included below.
- If a specification only states that an operation is rejected or should fail, the expected result is described without inventing a concrete status code.
- Symbolic test data is used to avoid implementation-specific assumptions.

## Shared test data

- `USER_A`: authenticated article owner
- `USER_B`: authenticated non-owner
- `TITLE_A`: valid article title
- `TITLE_DUP`: duplicate article title used twice
- `DESC_A`: valid article description
- `BODY_A`: initial article body
- `BODY_V2`: updated article body
- `TAGS_ORIG`: initial tag list, for example `["backend", "testing"]`
- `TAGS_EMPTY`: empty tag list `[]`
- `TAGS_NULL`: null tag list

---

## Create

### TC-AL-001 Authenticated user creates an article successfully
**Test objective**  
Verify that an authenticated user can create an article with valid fields.

**Preconditions**  
- `USER_A` is authenticated.

**Test data**  
- title: `TITLE_A`
- description: `DESC_A`
- body: `BODY_A`
- tagList: `TAGS_ORIG`

**Test steps**  
1. As `USER_A`, submit a create-article request with valid title, description, body, and tag list.
2. Observe the create response.

**Expected results**  
- The request succeeds with status `201`.
- The returned article includes `title`, `slug`, `description`, `body`, `tagList`, timestamps, favorite state, favorite count, and author username.
- The returned values match the submitted article content except for system-generated fields such as slug and timestamps.

**Specification source**  
- `article-lifecycle.md` Creation
- `article-lifecycle.md` Acceptance criteria

### TC-AL-002 Unauthenticated article creation is rejected
**Test objective**  
Verify that authentication is required for article creation.

**Preconditions**  
- No authenticated user context is present.

**Test data**  
- title: `TITLE_A`
- description: `DESC_A`
- body: `BODY_A`
- tagList: `TAGS_ORIG`

**Test steps**  
1. Submit a create-article request without authentication.
2. Observe the response.

**Expected results**  
- The request is rejected.
- No article is created.

**Specification source**  
- `article-lifecycle.md` Validation
- `article-lifecycle.md` Acceptance criteria

### TC-AL-003 Empty title is rejected during article creation
**Test objective**  
Verify that article creation rejects an empty title.

**Preconditions**  
- `USER_A` is authenticated.

**Test data**  
- title: empty string
- description: `DESC_A`
- body: `BODY_A`
- tagList: `TAGS_ORIG`

**Test steps**  
1. As `USER_A`, submit a create-article request with an empty title.
2. Observe the response.

**Expected results**  
- The request is rejected.
- No article is created.

**Specification source**  
- `article-lifecycle.md` Validation
- `article-lifecycle.md` Acceptance criteria

### TC-AL-004 Empty description is rejected during article creation
**Test objective**  
Verify that article creation rejects an empty description.

**Preconditions**  
- `USER_A` is authenticated.

**Test data**  
- title: `TITLE_A`
- description: empty string
- body: `BODY_A`
- tagList: `TAGS_ORIG`

**Test steps**  
1. As `USER_A`, submit a create-article request with an empty description.
2. Observe the response.

**Expected results**  
- The request is rejected.
- No article is created.

**Specification source**  
- `article-lifecycle.md` Validation
- `article-lifecycle.md` Acceptance criteria

### TC-AL-005 Empty body is rejected during article creation
**Test objective**  
Verify that article creation rejects an empty body.

**Preconditions**  
- `USER_A` is authenticated.

**Test data**  
- title: `TITLE_A`
- description: `DESC_A`
- body: empty string
- tagList: `TAGS_ORIG`

**Test steps**  
1. As `USER_A`, submit a create-article request with an empty body.
2. Observe the response.

**Expected results**  
- The request is rejected.
- No article is created.

**Specification source**  
- `article-lifecycle.md` Validation
- `article-lifecycle.md` Acceptance criteria

### TC-AL-006 Duplicate titles are allowed and produce unique slugs
**Test objective**  
Verify that duplicate titles are accepted while generated slugs remain unique.

**Preconditions**  
- `USER_A` is authenticated.

**Test data**  
- first article title: `TITLE_DUP`
- second article title: `TITLE_DUP`
- description: `DESC_A`
- body: `BODY_A`

**Test steps**  
1. As `USER_A`, create the first article using `TITLE_DUP`.
2. As `USER_A`, create a second article using the same title `TITLE_DUP`.
3. Compare the returned slugs.

**Expected results**  
- Both create requests succeed.
- The two created articles have the same title.
- The two created articles have different slugs.

**Specification source**  
- `article-lifecycle.md` Creation
- `article-lifecycle.md` Acceptance criteria

---

## Update semantics

### TC-AL-007 Owner updates only the article body and other fields remain unchanged
**Test objective**  
Verify partial update behavior when only the body is modified.

**Preconditions**  
- `USER_A` is authenticated.
- `USER_A` has an existing article with title `TITLE_A`, description `DESC_A`, body `BODY_A`, and tag list `TAGS_ORIG`.

**Test data**  
- updated body: `BODY_V2`

**Test steps**  
1. As `USER_A`, update the existing article by changing only the body to `BODY_V2`.
2. Retrieve the article after the update.

**Expected results**  
- The update succeeds for the owner.
- The article body is updated to `BODY_V2`.
- The title remains `TITLE_A`.
- The description remains `DESC_A`.
- The tag list remains `TAGS_ORIG`.
- The persisted article state is observable on retrieval.

**Specification source**  
- `article-lifecycle.md` Update semantics
- `article-lifecycle.md` Acceptance criteria

### TC-AL-008 Omitting tagList during update preserves existing tags
**Test objective**  
Verify that tags are preserved when `tagList` is omitted from an update.

**Preconditions**  
- `USER_A` is authenticated.
- `USER_A` has an existing article with tag list `TAGS_ORIG`.

**Test data**  
- updated body: `BODY_V2`
- tagList: omitted

**Test steps**  
1. As `USER_A`, update the article without providing `tagList`.
2. Retrieve the article after the update.

**Expected results**  
- The update succeeds.
- The existing tag list remains unchanged as `TAGS_ORIG`.
- The preserved tag state is observable on retrieval.

**Specification source**  
- `article-lifecycle.md` Update semantics
- `article-lifecycle.md` Acceptance criteria

### TC-AL-009 Empty tagList removes all tags during update
**Test objective**  
Verify that an empty tag list clears all tags from the article.

**Preconditions**  
- `USER_A` is authenticated.
- `USER_A` has an existing article with tag list `TAGS_ORIG`.

**Test data**  
- tagList: `TAGS_EMPTY`

**Test steps**  
1. As `USER_A`, update the article with `tagList=[]`.
2. Retrieve the article after the update.

**Expected results**  
- The update succeeds.
- The article has no tags after the update.
- The cleared tag state is observable on retrieval.

**Specification source**  
- `article-lifecycle.md` Update semantics
- `article-lifecycle.md` Acceptance criteria

### TC-AL-010 Null tagList is rejected during update
**Test objective**  
Verify that `tagList=null` is not accepted during update.

**Preconditions**  
- `USER_A` is authenticated.
- `USER_A` has an existing article with tag list `TAGS_ORIG`.

**Test data**  
- tagList: `TAGS_NULL`

**Test steps**  
1. As `USER_A`, update the article with `tagList=null`.
2. Observe the update response.
3. Retrieve the article after the failed update attempt.

**Expected results**  
- The request is rejected.
- The article retains its previous tag list and other original data.

**Specification source**  
- `article-lifecycle.md` Update semantics
- `article-lifecycle.md` Acceptance criteria

---

## Delete

### TC-AL-011 Unauthenticated article update is rejected
**Test objective**  
Verify that authentication is required for article update.

**Preconditions**  
- An existing article is present.
- No authenticated user context is present.

**Test data**  
- attempted body update: `BODY_V2`

**Test steps**  
1. Attempt to update an existing article without authentication.
2. Observe the update response.
3. Retrieve the article afterwards.

**Expected results**  
- The request is rejected.
- The original article remains unchanged.

**Specification source**  
- `article-lifecycle.md` Black-box test dimensions

### TC-AL-012 Unauthenticated article deletion is rejected
**Test objective**  
Verify that authentication is required for article deletion.

**Preconditions**  
- An existing article is present.
- No authenticated user context is present.

**Test data**  
- existing article

**Test steps**  
1. Attempt to delete an existing article without authentication.
2. Observe the delete response.
3. Retrieve the article afterwards.

**Expected results**  
- The request is rejected.
- The original article still exists.

**Specification source**  
- `article-lifecycle.md` Black-box test dimensions

### TC-AL-013 Owner deletes an article successfully
**Test objective**  
Verify that an article owner can delete an existing article.

**Preconditions**  
- `USER_A` is authenticated.
- `USER_A` has an existing article.

**Test data**  
- existing article created by `USER_A`

**Test steps**  
1. As `USER_A`, delete the existing article.
2. Observe the delete response.

**Expected results**  
- The delete operation succeeds.

**Specification source**  
- `article-lifecycle.md` Deletion
- `article-lifecycle.md` Acceptance criteria

### TC-AL-014 Deleted article cannot be retrieved afterwards
**Test objective**  
Verify that deletion is persistent and visible in later retrieval.

**Preconditions**  
- `USER_A` has successfully deleted a previously existing article.

**Test data**  
- slug of deleted article

**Test steps**  
1. Attempt to retrieve the deleted article.
2. Observe the retrieval response.

**Expected results**  
- Retrieval fails or clearly indicates that the article no longer exists.

**Specification source**  
- `article-lifecycle.md` Deletion
- `article-lifecycle.md` Acceptance criteria

---

## Ownership authorization

### TC-AL-015 Non-owner cannot update another user's article
**Test objective**  
Verify that an authenticated non-owner cannot update another user's article.

**Preconditions**  
- `USER_A` is authenticated and has created an article.
- `USER_B` is authenticated.

**Test data**  
- attempted body update by `USER_B`: `BODY_V2`

**Test steps**  
1. As `USER_B`, attempt to update the article created by `USER_A`.
2. Observe the update response.
3. Retrieve the article afterwards.

**Expected results**  
- The update is rejected with status `403`.
- The original article remains unchanged after the failed update attempt.

**Specification source**  
- `authorization-ownership.md` Article ownership
- `authorization-ownership.md` Acceptance criteria

### TC-AL-016 Non-owner cannot delete another user's article
**Test objective**  
Verify that an authenticated non-owner cannot delete another user's article.

**Preconditions**  
- `USER_A` is authenticated and has created an article.
- `USER_B` is authenticated.

**Test data**  
- article created by `USER_A`

**Test steps**  
1. As `USER_B`, attempt to delete the article created by `USER_A`.
2. Observe the delete response.
3. Retrieve the article afterwards.

**Expected results**  
- The delete is rejected with status `403`.
- The original article still exists after the failed delete attempt.

**Specification source**  
- `authorization-ownership.md` Article ownership
- `authorization-ownership.md` Acceptance criteria

---

## Coverage summary

This document covers the following specification dimensions:
- valid create flow
- field-level validation failures
- authentication required for create
- duplicate title handling with unique slug generation
- partial update behavior
- tag preservation when omitted
- tag removal with empty list
- rejection of null tag list
- persistence after update
- deletion persistence
- non-owner update/delete rejection with resource preservation

## Open ambiguities intentionally left unspecified

To stay faithful to the repository specifications only, this document does not assume:
- exact API paths
- exact request or response envelope structure
- exact error-body field names
- exact status codes for all rejection cases unless explicitly stated in the specifications
