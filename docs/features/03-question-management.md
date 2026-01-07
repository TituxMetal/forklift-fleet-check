# Feature Shape: Question Management

## Problem

Operators need to complete safety inspections with specific questions, but there's no way to define
what those questions are. Each CACES category has different inspection requirements, and answers
need severity levels to determine equipment status.

Without questions and answers, the inspection flow cannot exist.

## Solution (Broad Strokes)

Admin-only management of inspection questions organized by CACES category:

- **Four fixed categories**: Communes (common to all), CACES 1, CACES 3, CACES 5
- **Questions** belong to a category and have a sort order
- **Answers** belong to a question with customizable text and fixed severity level (OK, Défaut,
  Critique)
- Each question has exactly 3 answers (one per severity level)

During inspection, operators see questions from:

1. "Communes" category (applies to all equipment)
2. The specific CACES category matching the equipment being inspected

## User Flow

### View Questions by Category

1. Admin navigates to Question Management
2. Admin sees four category tabs: Communes, CACES 1, CACES 3, CACES 5
3. Admin clicks on a category tab
4. List shows all questions in that category, ordered by sortOrder
5. Each question shows its text and the three answer options

### Create Question

1. Admin selects a category tab
2. Admin clicks "Add Question"
3. Form appears with:
   - Question text (e.g., "État général de la machine")
   - Answer for OK severity (e.g., "Bon état")
   - Answer for Défaut severity (e.g., "Usure visible")
   - Answer for Critique severity (e.g., "Dommage important")
4. Admin fills the form and submits
5. Question is added at the end of the category (highest sortOrder + 1)
6. Admin sees the new question in the list

### Edit Question

1. Admin clicks on a question in the list
2. Detail/edit view shows current values
3. Admin modifies question text or answer texts
4. Admin saves changes
5. Changes are persisted (severity levels cannot be changed, only answer texts)

### Delete Question

1. Admin clicks delete icon on a question
2. Confirmation dialog: "Delete this question? Existing inspection records will keep the question
   text as historical data."
3. Admin confirms
4. Question is deleted (or soft-deleted for history preservation)

### Reorder Questions

1. Admin sees drag handles or up/down arrows on each question
2. Admin drags a question to a new position (or clicks arrows)
3. sortOrder values are recalculated
4. New order is persisted

### Move Question to Different Category

1. Admin opens question detail/edit
2. Admin changes the category dropdown
3. Question moves to the new category (at the end)
4. sortOrder is recalculated for both old and new categories

## Dependencies

**Requires:**

- Auth with Roles (Feature 01) — Admin role must exist
- Role-based route protection

**Enables:**

- Daily Inspection (Feature 07) — Inspection flow needs questions to display
- Inspection Results (Feature 05) — Answers have severity for status calculation

## What Must Exist (Backend)

### Domain Layer

- **QuestionCategory entity**: Represents the four fixed categories
  - Properties: id, name, sortOrder
  - Fixed values: Communes (0), CACES 1 (1), CACES 3 (2), CACES 5 (3)
- **Question entity**: An inspection question
  - Properties: id, categoryId, text, sortOrder, isActive, createdAt, updatedAt
  - Relations: belongs to category, has many answers
- **Answer entity**: A possible answer to a question
  - Properties: id, questionId, text, severity, sortOrder
  - Severity is enum: ok, warning, critical
- **Value Objects**:
  - QuestionId, AnswerId (UUID wrappers)
  - QuestionText (validated, non-empty, max length)
  - AnswerText (validated, non-empty, max length)
  - Severity (enum: ok, warning, critical)
  - CategoryType (enum: communes, caces_1, caces_3, caces_5)
- **Repository interfaces**: QuestionRepository, AnswerRepository
- **Domain exceptions**: QuestionNotFound, CategoryNotFound, InvalidSeverity

### Application Layer

- **Create question use case**: Create question with three answers (one per severity)
- **Update question use case**: Modify question text and/or answer texts
- **Delete question use case**: Remove question (consider soft delete for history)
- **Reorder questions use case**: Update sortOrder values within a category
- **List questions by category use case**: Get all questions for a category
- **Get question detail use case**: Single question with its answers
- **DTOs**:
  - CreateQuestionDto (includes answer texts for each severity)
  - UpdateQuestionDto
  - QuestionResponseDto
  - AnswerResponseDto
  - ReorderQuestionsDto (array of id + new sortOrder)
- **Mappers**: Question entity to DTO, Answer entity to DTO

### Infrastructure Layer

- **Database schema**:
  - QuestionCategory table (seeded with 4 fixed categories)
  - Question table with categoryId foreign key
  - Answer table with questionId foreign key
- **Prisma repository implementations**
- **Questions controller**: HTTP endpoints for all operations
- **Admin role guard**: Protect all question management endpoints

### API Endpoints

- `GET /api/question-categories` — List all categories (for tabs)
- `GET /api/questions?categoryId=X` — List questions by category
- `GET /api/questions/:id` — Get question with answers
- `POST /api/questions` — Create question with answers (admin)
- `PATCH /api/questions/:id` — Update question and answers (admin)
- `DELETE /api/questions/:id` — Delete question (admin)
- `PATCH /api/questions/reorder` — Reorder questions within category (admin)

## What Must Exist (Frontend)

### Pages/Routes

- **Question management page** (admin): Category tabs, question list, add/edit

### Components

- **Category tabs**: Four tabs for the fixed categories
- **Question list**: Ordered list of questions within selected category
- **Question card**: Displays question text with its three answer options
- **Question form**: Create/edit form with question text and three answer fields
- **Answer option display**: Shows answer text with severity badge
- **Severity badge**: Color-coded (green OK, orange Défaut, red Critique)
- **Reorder controls**: Drag handles or up/down arrows
- **Delete confirmation dialog**

### State Management

- **Questions store**: Questions grouped by category, loading states
- **Selected category atom**: Currently active category tab
- **Question form state**: For create/edit operations

### Validation

- **Question schema** (Zod):
  - Question text: required, 5-500 characters
  - OK answer text: required, 2-200 characters
  - Défaut answer text: required, 2-200 characters
  - Critique answer text: required, 2-200 characters

## Open Questions

1. Should questions be soft-deleted to preserve inspection history, or can we keep the question text
   as a snapshot in the inspection record?
2. Should there be a limit to how many questions per category?
3. Can the same question text exist in multiple categories, or should it be unique?
4. Should reordering be drag-and-drop or simple up/down arrows? (Drag is nicer UX but more complex)

## Out of Scope

- Custom categories beyond the four fixed ones
- Question branching/conditional logic
- Multiple choice answers (exactly 3 answers per question, one per severity)
- Question groups/sections within a category
- Question templates or duplication
- Bulk import/export of questions
- Question versioning (track changes over time)
- Answer images or icons

## Risks / Gotchas

- **Fixed severity structure**: Every question MUST have exactly 3 answers mapping to OK, Défaut,
  Critique. The form should enforce this — not allow adding/removing answers.
- **Category seeding**: The four categories must be seeded on database setup. They are not
  user-created.
- **Inspection history**: If a question is deleted or modified, existing inspection records should
  preserve the original question/answer text. Consider storing snapshots or soft-deleting.
- **French UI**: All category names and labels in French:
  - Categories: "Communes", "CACES 1", "CACES 3", "CACES 5"
  - Severities: "OK", "Défaut", "Critique"
- **Empty categories**: A category can have zero questions initially. Operators cannot inspect
  equipment if all relevant questions are missing.
- **sortOrder integrity**: When deleting or moving questions, sortOrder values should remain
  sequential (1, 2, 3...) — consider recalculating on changes.
- **Answer order**: Within a question, answers should always display in order: OK first, then
  Défaut, then Critique (sortOrder 0, 1, 2).
