<%*
// ============================================================================
// DYNAMIC TEMPLATE - Auto-discovers documents and sections
// ============================================================================
// This template uses Templater to dynamically discover all .md files in the
// project and extract their section headers, eliminating the need for manual
// updates when documents or sections change.
// ============================================================================

// Helper function to extract section headers from markdown content
function extractSections(content) {
  const sections = ["General"];
  const headerRegex = /^##\s+(.+)$/gm;
  let match;
  
  while ((match = headerRegex.exec(content)) !== null) {
    sections.push(match[1].trim());
  }
  
  return sections;
}

// Get all markdown files in the project
const allFiles = app.vault.getMarkdownFiles();

// Filter files to exclude (check filename only, not full path)
const excludePatterns = ['Template', 'Feedback-'];
const projectFiles = allFiles
  .filter(file => {
    const filename = file.basename;
    // Exclude if filename contains any exclude pattern
    if (excludePatterns.some(pattern => filename.includes(pattern))) {
      return false;
    }
    // Exclude if in Archive folder
    if (file.path.includes('Archive/')) {
      return false;
    }
    return true;
  })
  .map(file => file.basename + '.md')
  .sort();

// Add "General" option at the end
projectFiles.push("General");

// Let user select document (SINGLE SELECT - reliable)
const selectedDoc = await tp.system.suggester(projectFiles, projectFiles);

// If "General" selected, use simple section list
if (selectedDoc === "General") {
  const selectedSection = "General";
-%>
Document: <% selectedDoc %>
Section: <% selectedSection %>
Type: <% await tp.system.suggester([
  "Review - Section needs assessment for completeness, coherence, or quality",
  "Question - Need clarification or decision on something unclear",
  "Addition - New content/detail that should be added to existing section",
  "Concern - Potential issue, inconsistency, or problem that needs attention"
], ["Review", "Question", "Addition", "Concern"]) %>
Action: <% await tp.system.suggester([
  "Flag issues - List specific problems with line/section references, no fixes",
  "Review - Assess completeness and coherence, identify gaps",
  "Review as skeptic - Challenge assumptions and try to break the design",
  "Implement - Make the specific change described in feedback",
  "Discuss - Collaborative exploration before making documentation changes",
  "Add to parking lot - Record in 05-future-considerations.md",
  "(skip - optional field)"
], ["Flag issues", "Review", "Review as skeptic", "Implement", "Discuss", "Add to parking lot", ""]) %>

<% tp.file.cursor(1) %>

---
<%* } else {
  // Find the actual file
  const targetFile = allFiles.find(f => (f.basename + '.md') === selectedDoc);
  
  if (targetFile) {
    // Read the file content
    const content = await app.vault.read(targetFile);
    
    // Extract sections
    const sections = extractSections(content);
    
    // Let user select section (SINGLE SELECT - reliable)
    const selectedSection = await tp.system.suggester(sections, sections);
-%>
Document: <% selectedDoc %>
Section: <% selectedSection %>
Type: <% await tp.system.suggester([
  "Review - Section needs assessment for completeness, coherence, or quality",
  "Question - Need clarification or decision on something unclear",
  "Addition - New content/detail that should be added to existing section",
  "Concern - Potential issue, inconsistency, or problem that needs attention"
], ["Review", "Question", "Addition", "Concern"]) %>
Action: <% await tp.system.suggester([
  "Flag issues - List specific problems with line/section references, no fixes",
  "Review - Assess completeness and coherence, identify gaps",
  "Review as skeptic - Challenge assumptions and try to break the design",
  "Implement - Make the specific change described in feedback",
  "Discuss - Collaborative exploration before making documentation changes",
  "Add to parking lot - Record in 05-future-considerations.md",
  "(skip - optional field)"
], ["Flag issues", "Review", "Review as skeptic", "Implement", "Discuss", "Add to parking lot", ""]) %>

<% tp.file.cursor(1) %>

---
<%* } else {
    // Fallback if file not found
-%>
Document: <% selectedDoc %>
Section: General
Type: <% await tp.system.suggester([
  "Review - Section needs assessment for completeness, coherence, or quality",
  "Question - Need clarification or decision on something unclear",
  "Addition - New content/detail that should be added to existing section",
  "Concern - Potential issue, inconsistency, or problem that needs attention"
], ["Review", "Question", "Addition", "Concern"]) %>
Action: <% await tp.system.suggester([
  "Flag issues - List specific problems with line/section references, no fixes",
  "Review - Assess completeness and coherence, identify gaps",
  "Review as skeptic - Challenge assumptions and try to break the design",
  "Implement - Make the specific change described in feedback",
  "Discuss - Collaborative exploration before making documentation changes",
  "Add to parking lot - Record in 05-future-considerations.md",
  "(skip - optional field)"
], ["Flag issues", "Review", "Review as skeptic", "Implement", "Discuss", "Add to parking lot", ""]) %>

<% tp.file.cursor(1) %>

---
<%* }
} %>
