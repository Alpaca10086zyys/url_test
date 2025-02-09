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
  # - block: collection
  #   id: papers
  #   content:
  #     title: 论文发表
  #     filters:
  #       folders:
  #         - papers
  #       exclude_featured: false
  #   design:
  #     view: paper
  - block: markdown
    id: papers
    content:
      title: 论文
      text: |-
        {{< paper-list >}}
  - block: markdown
    id: books
    content:
      title: 专著
      text: |-
        {{< book-section >}}
    design:
      view: paper
  # - block: collection
  #   id: patents
  #   content:
  #     title: 专利
  #     text: ""
  #     filters:
  #       folders:
  #         - patent
  #   design:
  #     view: patent
  - block: markdown
    id: patents
    content: 
      title: 专利
      text: |-
        {{< patent-list >}}
---