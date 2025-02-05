---
title: 师资力量
date: 2022-10-24
type: landing

design:
  # Default section spacing
  spacing: "6rem"

sections:
  - block: markdown
    id: teacher
    content:
      text: |-
        {{< team-teacher-list >}}
    design:
      columns: '1'
      spacing: 
        padding: [0, 0, 0, 0]
  - block:
    id: student
    content:
      text: |-
        {{< team-student-list >}}
    design:
      columns: '1'
      spacing: 
        padding: [0, 0, 0, 0]    
  - block:
    id: alumnus
    content:
      text: |-
        {{< team-alumnus-list >}}
    design:
      columns: '1'
      spacing: 
        padding: [0, 0, 0, 0]    
---