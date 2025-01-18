---
title: "组内新闻"
date: 2023-10-24
type: landing

design:
    spacing: "6rem"

pagination:
  disableAliases: false
  pagerSize: 10
  path: page

sections:
  - block: collection
    id: news
    content:
      title: ''
      subtitle: ''
      text: ''
      page_type: post
      count: 5
      filters:
        author: ""
        category: ""
        tag: ""
        exclude_featured: false
        exclude_future: false
        exclude_past: false
        publication_type: ""
      offset: 0
      order: desc
    design:
      view: date-title-summary
      spacing:
        padding: [0, 0, 0, 0]
---    