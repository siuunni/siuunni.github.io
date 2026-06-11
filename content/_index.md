---
# Leave the homepage title empty to use the site title
title: ''
summary: ''
date: 2022-10-24
type: landing

sections:
  - block: resume-biography-3
    content:
      # Choose a user profile to display (a folder name within `content/authors/`)
      username: me
      text: '제조 현장의 센서 데이터와 통계적 신뢰성 연구를 바탕으로 이상 탐지, 예측, 최적화, 생성형 AI Agent를 설계합니다.'
      # Show a call-to-action button under your biography? (optional)
      button:
        text: Resume
        url: uploads/Sieun_Kim_Resume.pdf?v=20260611
      headings:
        about: 'About'
        education: 'Education'
        interests: 'Research Interests'
    design:
      # Use the new Gradient Mesh which automatically adapts to the selected theme colors
      background:
        gradient_mesh:
          enable: true

      # Name heading sizing to accommodate long or short names
      name:
        size: md # Options: xs, sm, md, lg (default), xl

      # Avatar customization
      avatar:
        size: medium # Options: small (150px), medium (200px, default), large (320px), xl (400px), xxl (500px)
        shape: square # Options: circle (default), square, rounded
  - block: resume-skills
    id: skills
    content:
      title: Skills
      username: me
  - block: collection
    id: featured-projects
    content:
      title: Featured Projects
      filters:
        folders:
          - projects
        exclude_featured: false
      sort_by: Weight
      sort_ascending: true
      count: 6
    design:
      view: article-grid
      columns: 3
      show_date: false
      show_read_more: false
---
