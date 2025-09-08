# Chapter 3: A Framework for System Design Interviews

## Purpose of System Design Interviews

System design interviews simulate real-life problem solving and collaboration. They assess:
- Design skills and ability to defend choices
- Collaboration under pressure
- Ability to resolve ambiguity
- Skill in asking good questions

**Red flags to avoid:** Over-engineering, narrow mindedness, stubbornness

## 4-Step Framework

### Step 1: Understand the Problem (3-10 minutes)
**Don't be like Jimmy** - don't jump to solutions immediately.

**Key questions to ask:**
- What specific features are we building?
- How many users does the product have?
- What are the anticipated scales (3 months, 6 months, 1 year)?
- What's the company's technology stack?

**Example questions for news feed system:**
- Mobile app, web app, or both?
- Most important features?
- Feed sorting (chronological vs weighted)?
- How many friends per user?
- Traffic volume (DAU)?
- Content types (text, images, videos)?

### Step 2: High-Level Design (10-15 minutes)
- Develop initial blueprint and get interviewer buy-in
- Draw box diagrams with key components
- Do back-of-the-envelope calculations
- Go through concrete use cases
- Collaborate with interviewer as teammate

**Components to consider:** clients, APIs, web servers, data stores, cache, CDN, message queue

### Step 3: Design Deep Dive (10-25 minutes)
Focus on specific components based on interviewer feedback:
- Identify and prioritize architecture components
- Dig into system performance characteristics
- Discuss bottlenecks and resource estimations
- Avoid unnecessary details that don't demonstrate abilities

**Time management is essential** - don't get carried away with minute details.

### Step 4: Wrap Up (3-5 minutes)
- Identify system bottlenecks and improvements
- Recap your design
- Discuss error cases (server failure, network loss)
- Cover operational issues (monitoring, metrics, rollout)
- Address next scale curve challenges
- Propose refinements with more time

## Dos and Don'ts

### Dos
- Always ask for clarification
- Understand requirements thoroughly
- Communicate your thinking process
- Suggest multiple approaches
- Design critical components first
- Bounce ideas off interviewer
- Never give up

### Don'ts
- Don't be unprepared
- Don't jump to solutions without clarifying requirements
- Don't go into too much detail initially
- Don't hesitate to ask for hints
- Don't think in silence
- Don't assume you're done until interviewer says so

## Time Allocation (45-minute session)
- **Step 1:** 3-10 minutes
- **Step 2:** 10-15 minutes  
- **Step 3:** 10-25 minutes
- **Step 4:** 3-5 minutes