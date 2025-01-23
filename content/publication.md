---
title: Publications
date: 2022-10-24
type: landing
# cms_exclude: true

# # View.
# view: citation

# # Optional header image (relative to `static/media/` folder).
# banner:
#   caption: ''
#   image: ''

sections:
  - block: collection
    id: papers
    content:
      title: 论文发表
      text: ""
      filters:
        folders:
          - papers
        exclude_featured: false
    design:
      view: paper
  - block: collection
    id: books
    content:
      title: 专著
      text: ""
      filters:
        folders:
          - book
    design:
      view: citation
  - block: collection
    id: patents
    content:
      title: 专利
      text: ""
      filters:
        folders:
          - patent
    design:
      view: patent
---