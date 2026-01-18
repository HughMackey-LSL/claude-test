# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a web-based syllabus builder application for NYU Stern School of Business. It's a single-page application (SPA) built with vanilla JavaScript that helps instructors create, edit, and export course syllabi in multiple formats (JSON, Word, PDF).

**Technology Stack:**
- Pure HTML/CSS/JavaScript (no build system or framework)
- ES6 modules with CDN imports
- External libraries loaded via CDN:
  - `docx` (v8.5.0) - Word document generation
  - `jspdf` (v2.5.1) - PDF generation
  - `jspdf-autotable` (v3.5.31) - PDF table formatting
  - `html2canvas` (v1.4.1) - HTML to canvas conversion
  - `FileSaver.js` (v2.0.5) - Client-side file saving

## Running the Application

**Local Development:**
```bash
# Simply open index.html in a browser, or use a local server:
python3 -m http.server 8000
# Then navigate to http://localhost:8000/index.html

# Or use Python 2:
python -m SimpleHTTPServer 8000
```

**Testing:**
- No automated test suite exists
- Manual testing using the "Load Sample" button loads sample syllabus data
- Test files: `test-buttons.html`, `test-export.html`

## Architecture

### Core Structure

The application uses a single JavaScript class `SyllabusBuilder` that manages:
1. **Form Management** - 18 sections with required/optional fields
2. **Dynamic Course Outline Builder** - Nested modules containing class days
3. **Table of Contents (TOC)** - Real-time completion tracking with visual indicators
4. **Data Persistence** - JSON save/load functionality
5. **Export System** - Word (DOCX) and PDF generation

### Key Components

**Form Sections (script.js:6-25):**
The syllabus is organized into 18 sections stored in `this.sections` array:
- Required sections: Basic info, description, learning outcomes, requirements, outline, integrity policies, grading, accessibility
- Optional sections: Communication strategy, technical requirements, wellness, religious observances, devices policy, AI guidance
- Pre-filled fixed sections: Code of Conduct, General Conduct, Student Accessibility, Name & Pronouns

**Course Outline Builder (script.js:48-189):**
- Hierarchical structure: Modules → Class Days
- Dynamic add/remove functionality using event delegation (script.js:106-143)
- Each module contains: title, description, and multiple class days
- Each class day contains: title and content (readings/assignments/activities)
- Data structure: `{ modules: [{ title, description, classDays: [{ title, content }] }] }`

**TOC Status Tracking (script.js:306-364):**
- Real-time visual feedback with CSS classes: `.complete`, `.incomplete`, `.optional`
- Intersection Observer for scroll-based active section highlighting
- Updates on any form input/change event

**Export Functions:**
- **Word Export** (script.js:644-890): Uses `docx` library to generate structured DOCX with tables, styling, and NYU branding
- **PDF Export** (script.js:1113-1423): Uses `jspdf` and `jspdf-autotable` with custom formatting and page breaks
- **JSON Save/Load** (script.js:608-641): Preserves full form state for editing

### Data Flow

1. User fills form → `collectFormData()` gathers all data
2. Course outline → `collectCourseOutlineData()` builds nested module structure
3. Export triggers → `exportToWord()` or `exportToPDF()` validates and generates file
4. Load triggers → `handleFileLoad()` → `populateForm()` → `populateCourseOutline()` restores state

### Styling

**NYU Stern Branding:**
- Primary color: `#57068C` (NYU Purple)
- Fixed action buttons at top
- Sidebar TOC with completion indicators
- Responsive design with ~1200px max-width content area

## Key Implementation Details

### Course Outline Table Generation

**For Word (script.js:432-531):**
- Creates `docx` Table objects with purple headers (#57068C)
- Column widths: 35% for class day titles, 65% for content
- Handles multi-line content by splitting on `\n`

**For PDF (script.js:1252-1339):**
- Uses `autoTable` plugin if available
- Same column width ratios (35/65)
- Automatic page breaks with custom `addText()` helper

### Event Delegation Pattern

The course outline uses event delegation (script.js:106-143) to handle dynamically added modules and class days:
- Single event listener on `#moduleContainer`
- Handles `.add-class-day-btn`, `.remove-module-btn`, `.remove-class-btn` clicks
- Prevents removing last module or last class day

### Form Validation

Export functions validate:
1. All required fields are filled (script.js:664-671, 1134-1141)
2. At least one module exists (script.js:674-677, 1144-1147)
3. At least one class day has content (script.js:679-686, 1149-1156)
4. Select dropdowns convert value→display text before export (script.js:648-661, 1117-1131)

## Common Modifications

**Adding a new form section:**
1. Add HTML structure to `index.html` with unique section ID
2. Add TOC entry to sidebar navigation
3. Add section metadata to `this.sections` array (script.js:6-25)
4. Add export logic to `generateRequiredSections()` or similar

**Modifying export styling:**
- Word: Update `generateCourseOutlineTables()` and paragraph spacing
- PDF: Modify color constants, font sizes in `exportToPDF()` (script.js:1190-1417)

**Changing course outline structure:**
- Update data collection: `collectCourseOutlineData()` (script.js:158-189)
- Update HTML generation: `generateCourseOutlineHTML()` (script.js:191-232)
- Update Word tables: `generateCourseOutlineTables()` (script.js:432-531)
- Update PDF tables: autoTable config in `exportToPDF()` (script.js:1286-1335)

## File Structure

```
.
├── index.html           # Main application page
├── script.js            # Core application logic (1554 lines)
├── styles.css           # Styling with NYU branding
├── test-buttons.html    # Testing interface
├── test-export.html     # Export testing
└── .claude/
    └── settings.local.json  # Claude Code permissions
```

## Git Workflow

Recent commits show feature development pattern:
```
e02690f Course outline export implemented
f92a1a3 Course Outline Interface adjusted, export pending
98cde8e Version 1
```

Current branch: `main` (clean working tree)

## Notes

- No package.json or build system - this is intentional for simplicity
- All dependencies loaded via CDN - no npm install needed
- Browser compatibility: Modern browsers only (ES6 modules, Intersection Observer)
- Data privacy: All processing happens client-side, no server backend
