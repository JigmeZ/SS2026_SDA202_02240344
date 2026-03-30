# SDA202 Report  


## Introduction

The practical aims to replace the current manual grading process for programming assignments with an automated solution. Currently, students upload their work to GitHub, and professors grade each submission manually and process that is inefficient and prone to errors, especially with over 300 enrolled students.

The proposed system will allow multiple student submissions and also to check plagiarism through Turnitin.moreover it is to grade based on professor-defined criteria and finally send the final grades to the LMS  

This report models the system using UML from three complementary viewpoints.

---

## 1. Interaction Overview Diagram – Context Viewpoint

This diagram presents the high-level workflow of the assignment-handling process.

Workflow summary:

1. Professor sets assignment details, due date, and grading criteria.  
2. Students can submit their code multiple times.  
3. For each submission:
   - System checks if it is on time.
   - Late submissions are rejected.
   - On-time submissions undergo plagiarism checking via Turnitin.
   - If plagiarism is detected:
     - The system flags the submission.
     - Alerts are sent to the professor and academic regulator.
   - If no plagiarism is detected:
     - The system executes the code.
     - Grades the submission.
     - Sends the final grade to the LMS.

![alt text](<PART 1.png>)

---

## 2. Use Case Diagram – Functional Viewpoint

### Actors

- **Professor** – Creates assignments, deadlines, and grading criteria  
- **Student** – Submits assignments and views results  
- **Turnitin** – External plagiarism checker  
- **LMS** – Stores final grades  

### Key Use Case Relationships

- **Submit Assignment → includes → Check Plagiarism**  
  Mandatory step for every submission.

- **Grade Assignment → extends → Submit Assignment**  
  Grading happens only when submission conditions are satisfied.

- **Grade Assignment → includes → Sync Final Grade to LMS**  
  Synchronization is mandatory after grading.

- **Flag Plagiarism**  
  Triggered when plagiarism is detected, resulting in notifications to the professor and regulator.

![alt text](<PART 2.drawio.png>)

---

## 3. Interaction Overview Diagram – System Viewpoint

This diagram details the internal flow of system operations, structured into three main interaction frames.

### A. Professor Manages Assignment

- Professor logs in  
- Creates assignment  
- Sets deadlines  
- Defines grading criteria  

### B. Student Submits Assignment

- Student uploads code  
- Loop allows multiple submissions  
- System checks if submission is on time  
  - If late → reject submission  
  - If on time → proceed to plagiarism check  

### C. System Processes Submission

- If plagiarism is detected:
  - System flags submission  
  - Notifies professor and regulator  
- If plagiarism is not detected:
  - System executes the code  
  - Grades submission  
  - Synchronizes final grade with LMS  

![alt text](<Part 3.drawio.png>)

---

## Reflection

* While creating the UML diagrams for this system, I gained a much clearer understanding of how different viewpoints work together to describe a complete solution. Modeling the system from the context, functional, and system perspectives helped me see how each actor interacts with the automated grading process and how the internal workflow supports those interactions. I realized that separating these viewpoints makes it easier to understand the system’s structure, starting from high-level interactions and gradually moving toward detailed internal behavior. This process improved my appreciation for how UML helps break down complex requirements into manageable parts.

* During the modeling process, I faced a few challenges, especially when deciding whether to use include or extend relationships in the use case diagram. It took some effort to understand which processes were mandatory and which were conditional. I also struggled a bit with organizing the interaction flows in the system-level diagram, particularly with loops for multiple submissions and the branching logic for plagiarism detection and late submissions. Representing external systems like Turnitin and the LMS clearly was also something I had to think through carefully. Despite these challenges, the exercise helped me build confidence in analyzing system behavior and applying UML modeling techniques effectively.
---

## Conclusion

The three UML diagrams gave me a clear and organized understanding of how the automated grading system should work. They helped show what features the system needs, who is involved, and how important processes like plagiarism checking, automated grading, and sending grades to the LMS should happen. By putting everything in a structured visual form, the diagrams make it much easier for developers to understand the system and move forward with its implementation.

---

