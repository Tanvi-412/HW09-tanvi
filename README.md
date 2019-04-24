# SSW810
"""Name: Tanvi Hanamshet
Course: SSW 810
Script: New web page to display a summary of 
each Instructor with her CWID, Name, Department, 
Course, and the number of students in the course.
"""

from flask import Flask, render_template
#from gevent.pywsgi import WSGIServer
import sqlite3


app = Flask(__name__)

"""link the database file with the code"""

DB_FILE = "/Users/tanvihanamshet/hello_flask/810.db"

query = """select CWID, Name, Dept, Course, count(*) as students from HW11_instructors as i join HW11_grades
           on Instructor_CWID = CWID
           group by CWID, Name, Dept, Course"""

db = sqlite3.connect(DB_FILE)
rows = db.execute(query)
data = [{'cwid' : cwid, 'name' : name, 'dept' : dept, 'course' : course, 'students' : students }
        for cwid, name, dept, course, students in rows]

db.close()


"""To create a webpage"""

@app.route('/student_courses')
def student_courses():
    query = """select CWID, Name, Dept, Course, count(*) as students from HW11_instructors as i join HW11_grades
           on Instructor_CWID = CWID
           group by CWID, Name, Dept, Course"""

    db = sqlite3.connect("/Users/tanvihanamshet/hello_flask/810.db")
    results = db.execute(query)
    data = [{'cwid' : cwid, 'name' : name, 'dept' : dept, 'course' : course, 'students' : students }
        for cwid, name, dept, course, students in results]

    db.close()

    return render_template('student_courses.html',
                            title='Stevens Repository',
                            table_title="Number of completed courses by Student",
                            students=data)

app.run(debug = True)
   
