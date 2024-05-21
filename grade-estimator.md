---
layout: default
title: Grade Estimator
---

<script type="text/javascript" src="{{ site.baseurl }}/assets/brython.js"></script>

<script type="text/python">
from browser import document, html

class Grade_Projection:
    def __init__(self, W):
        self.upper_est = 0
        self.lower_est = 100
        self.weights = W
        self.known_grades = {}

    def add(self, x):
        if not x.type in self.weights.keys():
            return
        if not x.type in self.known_grades:
            self.known_grades[x.type] = []
        self.known_grades[x.type].append(x.score)
        return 

    def high_impact(self):
        highest_impact_item = None
        highest_impact_percentage = 0
        for type in self.weights.keys():
            weight = self.weights[type][0]
            number = self.weights[type][1]
            if type in self.known_grades:
                accounted_for = (len(self.known_grades[type])/number) * weight
                if weight-accounted_for > highest_impact_percentage:
                    highest_impact_item = type
                    highest_impact_percentage = weight-accounted_for
            else:
                if weight > highest_impact_percentage:
                    highest_impact_item = type
                    highest_impact_percentage = weight
        return highest_impact_item, highest_impact_percentage

    def project_grade(self):
        low_est = 0
        total_weight = 0
        for type in self.weights.keys():
            weight = self.weights[type][0]
            number = self.weights[type][1]
            if type in self.known_grades:
                for score in self.known_grades[type]:
                    total_impact = 1/number
                    total_weight += total_impact * weight
                    low_est += total_impact * score * weight

        high_est = low_est + (1-total_weight)
        return f"Low estimate: {low_est:.2f}, High estimate: {high_est:.2f}"

class Grade_Component:
    def __init__(self, type, score):
        self.type = type
        self.score = score

grade_projection = None

def initialize_projection(event):
    weights_str = document['weights'].value
    weights = eval(weights_str)
    global grade_projection
    grade_projection = Grade_Projection(weights)
    document['output'].html = "<h3>Grade projection initialized!</h3>"

def add_grade_item(event):
    type = document['type'].value
    score = float(document['score'].value)
    grade_component = Grade_Component(type, score)
    grade_projection.add(grade_component)
    document['output'].html = f"<h4>Added: {type} with score {score}</h4>"

def project_grade(event):
    projection = grade_projection.project_grade()
    document['output'].html = f"<h3>{projection}</h3>"

document <= html.INPUT(id="weights", type="text", value='{"Assignment": (0.4, 3), "Exam 1": (0.2, 1), "Final": (0.4, 1)}', style={'width': '50%'})
document <= html.BUTTON("Initialize Projection", id="initialize")
document["initialize"].bind("click", initialize_projection)

document <= html.BR()
document <= html.INPUT(id="type", type="text", placeholder="Type")
document <= html.INPUT(id="score", type="number", placeholder="Score")
document <= html.BUTTON("Add Grade Item", id="add")
document["add"].bind("click", add_grade_item)

document <= html.BR()
document <= html.BUTTON("Project Grade", id="project")
document["project"].bind("click", project_grade)

document <= html.DIV(id="output")
</script>
