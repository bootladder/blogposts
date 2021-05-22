---
layout: affiliates-page
title: "Case Studies: Excel"
comments: true
---
### Excel Automation
If you use Excel at your company, you need to **STOP** and contact me immediately for a consultation.  

Using Microsoft Excel basically says you have something that should be automated but isn't.  Let me help you with that.  I've done a wide variety of Excel automation projects, each of which saved my client from disaster.  

### #1 Excel --> Database + Web App
I completely eliminated an Excel based workflow.  The project involved graphing time series data.  The workflow consisted of manual data collection and preprocessing which then went into charts and graphs in Excel.  This happened every day.  Mistakes were abound and the poor users, ACADEMIC RESEARCHERS, were stuck looking at reams of data and invalid charts.  
  
The solution was simple:  replace Excel with a database, hook up the data source to populate the database automatically, create a web app to present the charts and a UI to select which charts to view.  
  
  
  

  
### #2 Excel --> Database + Excel
In this project the solution was similar, except the clients really wanted to continue using Excel for charting and presentation purposes.  That's totally fine because outputting Excel documents is actually just a special case of being able to output ANY FORMAT YOU WANT, once you have switched to using a database.  And that's exactly what was done.  The data collection and processing steps were automated and converted to use a database, and Excel became simply a presentation medium.
  
  

### #3 Excel Based Testing Framework
  
I revived a derelict testing framework for a major company producing an embedded system.  
  
The test cases were expressed in the form of an Excel spreadsheet.   
While not a terrible idea, its implementation was... terrible.  My friends wouldn't believe me when I tried to describe it.  This inspired me to create a series called "Refactoring Safari", as it was a great way to demonstrate TDD and the SOLID principles.
   
  
The whole project was soon to go down the tubes, but there was valuable information captured in the "step definitions".  This was product and domain specific knowledge that took time for personnel to acquire, and would be a shame to waste just because the framework that executed it was garbage.

My approach for this situation was to do "surgical refactoring" of the test framework and test "step definitions" such that the original test suite could execute, but I could rescue the "step definitions" from the clutches of the framework.  The "step definitions" could then be maintained in good quality, and be used in any other framework.


