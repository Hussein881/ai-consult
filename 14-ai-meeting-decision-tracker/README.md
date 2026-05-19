# AI Meeting and Decision Tracker

## Overview

An AI assistant that processes meeting transcripts or notes to automatically create decision logs, action items, follow-up tasks, and meeting summaries. This system ensures decisions aren't lost and action items are tracked.

## Business Problem

Teams make decisions in meetings but often **lose track of action items, owners, deadlines, and rationale**. As a result, decisions get forgotten, action items slip, and accountability is unclear, leading to rework and missed deadlines.

## Target Users

- Engineering managers
- Product managers
- Executive teams
- Project managers
- Teams with frequent meetings

## AI Solution

Build an assistant that:
- Processes meeting notes or transcripts
- Summarizes key decisions
- Extracts action items with owners and dates
- Identifies unresolved questions
- Generates follow-up emails
- Tracks decision rationale
- Creates accountability

## Project Scope

**Included:**
- Upload meeting notes/transcripts
- Summarize key decisions
- Extract action items with owners and dates
- Identify unresolved questions
- Generate follow-up email draft
- Export task list
- Decision log

**Out of Scope:**
- Real-time meeting transcription
- Calendar integration

## MVP Features

1. **Meeting Input**
   - Text notes upload
   - Transcript upload
   - Manual meeting summary
   - Participant list

2. **Decision Summary**
   - Key decisions identified
   - Decision rationale
   - Alternatives considered
   - Approved decision

3. **Action Item Extraction**
   - Action item text
   - Owner assignment
   - Due date
   - Priority level

4. **Unresolved Questions**
   - Questions that need follow-up
   - Owners for investigation
   - Suggested next steps
   - Follow-up meeting needed

5. **Follow-up Email**
   - Meeting summary email
   - Key decisions
   - Action items with owners
   - Next steps

6. **Task Export**
   - Markdown task list
   - JSON export
   - Assignee notifications
   - Calendar integration

## Advanced Features

1. **Decision Tracking**
   - Historical decision log
   - Search previous decisions
   - Compare related decisions
   - Reversals and updates

2. **Action Item Follow-up**
   - Automated reminders
   - Status updates
   - Completion tracking
   - Accountability metrics

3. **Integration with Tools**
   - Jira ticket creation
   - GitHub issue creation
   - Slack notifications
   - Calendar invites

4. **Analytics**
   - Decision velocity
   - Action item completion rate
   - Decision patterns
   - Team insights

5. **Video/Audio Support**
   - Meeting recording transcription
   - Speaker identification
   - Timestamp tracking
   - Searchable transcripts

## Example Inputs

**Meeting Transcript:**
```
Attendees: Sarah (Product), John (Engineering), Maria (Design)

Sarah: We need to decide on the new dashboard design. 
John: I prefer option A - it's faster to build.
Maria: I agree, option A is cleaner too.
Decision: Go with dashboard design option A.

Sarah: Who's going to implement?
John: I'll start Monday, should take 2 weeks.
Action: John to implement dashboard by June 2.

Maria: We should validate with customers first.
Sarah: Good point. I'll set up customer interviews this week.
Action: Sarah to schedule customer interviews by May 24.
```

## Example Outputs

**Decision Log Entry:**

```markdown
# Dashboard Design Decision
**Date:** May 19, 2024
**Meeting:** Product Planning
**Decision:** Go with design option A

## Rationale
- Faster implementation timeline (2 weeks vs 4)
- Cleaner user interface
- Aligns with design system

## Alternatives Considered
- Option B: More comprehensive, longer timeline
- Option C: Custom design, highest effort

## Next Steps
- Customer validation (Sarah)
- Implementation begins (John)
```

**Action Items Task List:**

```markdown
# Follow-up Actions
Last Updated: May 19, 2024

## HIGH PRIORITY (This Week)
- [ ] Schedule customer interviews for dashboard validation
  - Owner: Sarah
  - Due: May 24
  - Status: Not started

## MEDIUM PRIORITY (Next 2 Weeks)
- [ ] Implement dashboard design option A
  - Owner: John
  - Due: June 2
  - Status: Not started

## QUESTIONS FOR FOLLOW-UP
- What's the rollout timeline for the new dashboard?
- Do we need to migrate existing data?
- Should we maintain the old dashboard in parallel?
```

**Follow-up Email:**

```
Subject: Follow-up: May 19 Product Planning Meeting

Hi Team,

Thanks for joining today's meeting. Below is a summary of key decisions and 
action items.

## Key Decision
We've approved Dashboard Design Option A. This design balances user experience 
with implementation timeline (2 weeks) and aligns with our design system.

## Action Items
1. **Sarah:** Schedule customer interviews for dashboard validation
   - Due: May 24
   
2. **John:** Implement dashboard design option A
   - Due: June 2
   - Can start Monday May 20

## Open Questions
Please respond to these questions before our next meeting:
- What's the target rollout date?
- Do we need backward compatibility?

Next meeting: May 26 at 2pm

Thanks,
[Meeting Organizer]
```

## Suggested Tech Stack

**Backend:**
- Python with FastAPI
- OpenAI API for summarization
- Speech-to-text API for transcripts

**Frontend:**
- Streamlit for MVP
- React for production UI

**Database:**
- PostgreSQL for decisions and actions

**Integrations:**
- Jira API
- GitHub API
- Slack API

## Architecture

```
Meeting Notes/Transcript
       ↓
Input Processing
       ↓
Text Extraction
       ↓
LLM Analysis
(decisions, actions)
       ↓
Information Extraction
       ↓
Decision Logging
       ↓
Task Creation
       ↓
Notification & Export
```

## Implementation Plan

### Phase 1: MVP (1-2 weeks)
- [ ] Note/transcript input
- [ ] Decision extraction
- [ ] Action item parsing
- [ ] Email generation

### Phase 2: Usability (1-2 weeks)
- [ ] Task export formats
- [ ] Follow-up reminders
- [ ] Decision history
- [ ] Search

### Phase 3: Production (1-2 weeks)
- [ ] Integrations (Jira, Slack)
- [ ] Analytics
- [ ] Video transcription
- [ ] Team collaboration

## Success Metrics

- Action item completion: > 90%
- Meeting follow-up time: 80% reduction
- Decision clarity: > 4/5
- Adoption: > 70% of teams

## Consulting Angle

Meeting and decision management automation for teams and enterprises.

**Implementation: $20-40K**
**Annual SaaS: $10-20K**

## Future Enhancements

1. Video meeting support
2. Automated reminders
3. Decision analytics
4. Historical trend analysis
5. Cross-team decision tracking

## Notes

- **Key Challenge:** Accurate action item extraction
- **Success Factor:** Team adoption and discipline
