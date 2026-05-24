# Learning Guide: AI Technical Documentation Generator

## What You Need to Learn

This project requires understanding how to read and parse code and APIs, generate structured text with LLMs, and integrate with developer workflows. The unique skill here is writing prompts that reliably produce documentation that is technically accurate and formatted to a standard — not just verbose text.

---

## Phase 1: Code Parsing and AST Basics (Week 1-2)

### Core Concepts
- Python Abstract Syntax Tree (AST) module
- Extracting function signatures, docstrings, and class hierarchies from code
- Parsing OpenAPI/Swagger JSON and YAML specs
- Reading Markdown and converting to structured data

### Resources
- **YouTube**: "Python AST Tutorial" by mCoding — https://www.youtube.com/watch?v=OjPT15y2EpE
- **Docs**: Python `ast` module — https://docs.python.org/3/library/ast.html
- **Docs**: OpenAPI Specification — https://swagger.io/specification/
- **Library**: `pydoc` for Python documentation extraction — https://docs.python.org/3/library/pydoc.html

---

## Phase 2: Prompt Engineering for Technical Writing (Week 2-3)

### Core Concepts
- Writing prompts for consistent output structure
- Using few-shot examples for documentation style
- Chain-of-thought for complex code explanations
- Template-based documentation: API docs, runbooks, architecture guides

### Resources
- **YouTube**: "Advanced Prompt Engineering Techniques" by Andrew Ng — https://www.youtube.com/watch?v=dOxUroR57xs
- **Course**: DeepLearning.AI Prompt Engineering course — https://learn.deeplearning.ai/chatgpt-prompt-eng
- **Article**: "Few-Shot Prompting Guide" — https://www.promptingguide.ai/techniques/fewshot
- **Docs**: OpenAI system prompts — https://platform.openai.com/docs/guides/prompt-engineering

---

## Phase 3: Template Systems and Structured Output (Week 3-4)

### Core Concepts
- Jinja2 templating for documentation generation
- Pydantic models for validating LLM output structure
- OpenAI function calling / JSON mode for consistent output
- Mustache and other template formats

### Resources
- **YouTube**: "Jinja2 Templating Tutorial" by Corey Schafer — https://www.youtube.com/watch?v=bxhXQG1qJPM
- **Docs**: Jinja2 documentation — https://jinja.palletsprojects.com/en/3.1.x/
- **Docs**: Pydantic documentation — https://docs.pydantic.dev/latest/
- **Course**: DeepLearning.AI "Functions, Tools and Agents" — https://learn.deeplearning.ai/functions-tools-agents-langchain

---

## Phase 4: Document Output Formats (Week 4)

### Core Concepts
- Generating Markdown, HTML, and plain text
- Producing DOCX files with `python-docx`
- PDF generation with `reportlab` or `WeasyPrint`
- Confluence API for publishing to enterprise wikis

### Resources
- **YouTube**: "Create Word Documents with Python" by NeuralNine — https://www.youtube.com/watch?v=G08UPMSzf1M
- **Docs**: python-docx — https://python-docx.readthedocs.io/en/latest/
- **Docs**: Confluence REST API — https://developer.atlassian.com/cloud/confluence/rest/v2/

---

## Phase 5: Review and Approval Workflow (Week 5)

### Core Concepts
- Diff-based review showing what changed
- Human-in-the-loop review before publishing
- Version control for generated docs
- Feedback loops for improving generation quality

### Resources
- **Article**: "Human-in-the-Loop AI" — https://humanloop.com/blog/human-in-the-loop
- **Library**: `deepdiff` for text comparison — https://github.com/seperman/deepdiff

---

## Phase 6: Integration with Developer Tools (Week 6)

### Core Concepts
- GitHub Actions workflow for automatic doc generation on commit
- Pre-commit hooks for documentation checks
- Publishing to GitHub Pages or Confluence

### Resources
- **YouTube**: "GitHub Actions for CI/CD" by TechWorld with Nana — https://www.youtube.com/watch?v=R8_veQiYBjI
- **Docs**: GitHub Actions — https://docs.github.com/en/actions

---

## Suggested Learning Sequence

| Week | Focus |
|------|-------|
| 1 | Python AST and OpenAPI parsing |
| 2 | Prompt engineering for structured text |
| 3 | Templates (Jinja2) and Pydantic output |
| 4 | Document output formats |
| 5 | Review and approval workflow |
| 6 | CI/CD integration |
| 7 | End-to-end test with real codebase |

---

## Books and Deeper Resources

- *Automate the Boring Stuff with Python* by Al Sweigart (free online) — https://automatetheboringstuff.com/
- *Clean Code* by Robert C. Martin — context on what good code documentation looks like
- OpenAI Prompt Engineering Guide — https://platform.openai.com/docs/guides/prompt-engineering
