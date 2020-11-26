---
# Documentation: https://wowchemy.com/docs/managing-content/

title: "Big Data Realtime Project"
summary: "GULI mall realtime user behavior analysis"
authors: [Xin LI]
tags: [Kafka, SparkStreaming, HBase, ElasticSearch, Kibana, SpringBoot]
categories: [Big Data]
date: 2020-11-26T15:48:39+01:00

# Optional external URL for project (replaces project detail page).
external_link: ""

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: "project structure"
  focal_point: ""
  preview_only: false

# Custom links (optional).
#   Uncomment and edit lines below to show custom links.
# links:
# - name: Follow
#   url: https://twitter.com
#   icon_pack: fab
#   icon: twitter

url_code: ""
url_pdf: ""
url_slides: ""
url_video: ""

# Slides (optional).
#   Associate this project with Markdown slides.
#   Simply enter your slide deck's filename without extension.
#   E.g. `slides = "example-slides"` references `content/slides/example-slides.md`.
#   Otherwise, set `slides = ""`.
slides: ""
---

Collected real-time user behavior data and sent it to **Kafka**.

Calculated daily and timesharing DAU and transaction amount using **SparkStreaming** and saved data to **Hbase**.

Joined data streams to merge order information and user together, then sent to **ElasticSearch**for further query.

Analyzed user purchase behavior from **Kibana**.
