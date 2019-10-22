title: Predicting Equipment failures from sensor data
summary: An example of using the in-built project page.
tags:
- Machine learning
date: "2019-10-21"

# Optional external URL for project (replaces project detail page).
external_link: ""

image:
  caption: Luke Sharrett — Bloomberg via Getty Images
  focal_point: Smart

#links:
#- icon: twitter
#  icon_pack: fab
#  name: Follow
#  url: https://twitter.com/georgecushen
url_code: "https://github.com/agleontyev/TAMU_Datathon_2019/blob/master/Conoco%20Competition5.ipynb"
#url_pdf: ""
#url_slides: ""
# url_video: ""

# Slides (optional).
#   Associate this project with Markdown slides.
#   Simply enter your slide deck's filename without extension.
#   E.g. `slides = "example-slides"` references `content/slides/example-slides.md`.
#   Otherwise, set `slides = ""`.
# slides: example
---
This project was done as a part of a ConocoPhilips challenge for 2019 Texas A&M University Datathon competition. The task is to predict which equipment  will fail on the surface (coded 0) or underground (coded 1) based on the sensor data. 
One of the biggest challenges in this project was the imbalance of classes (1:60 ration of class 1 to class 0). We resolved it by bootstrapping the underrepresented class.
Another challenge was that the difference in data types: some sensor data were presented over time and some as singular measurements. We averaged the over-time measurements to create a singular measurement.
The predictions made by our model achieved 98% accuracy in predicting underground failures on the company's data.
