Splitting up django models 
###########################
:date: 2011-08-25 10:07
:author: mrunge
:category: Programmieren
:tags: django, Python
:slug: splitting-up-django-models

#. Create models folder unter *myapp*
#. Delete models.py under *myapp*
#. Add

   ::

       class Meta:
           app_label='myapp'

   .. raw:: html

      <p>

   to each model

#. Add for each model something like

   ::

       from myModelFile import myModel

   .. raw:: html

      <p>

   to models/\_\_init\_\_.py


