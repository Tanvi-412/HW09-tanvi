# SSW810
"""Name: Tanvi Hanamshet
Course: SSW 810
Script: create a data repository of courses, students, and instructors 
"""

import os
import unittest
from collections import defaultdict
from prettytable import PrettyTable


def file_read(path, num_fields, seperator=','or'|'or'*', header=False):
    """returns the tuple with number of fields
    """
    try:
        fp = open(path, "r", encoding="utf-8")
    except FileNotFoundError:
        raise FileNotFoundError("Can't open", path)
    else:
        with fp:
            for n, line in enumerate(fp):
                if n == 0 and header:
                    continue  
                else:
                    fields = line.strip()
                    fields = fields.split(seperator)
                    if len(fields) != num_fields:
                        raise ValueError("*** number of fields in the file is not equal to expected numnber of fields ***")
                    else:
                        yield fields
                        
class major:
    def __init__(self, name, passing=None):
        self._name = name
        self._requried = set()
        self._electives = set()
        self._passing_grade = {'A', 'A-', 'B+', 'B', 'B-', 'C+', 'C'}

    def pt_row(self):
        return[self._name, self._requried, self._electives]

    def add_course(self, flag, course):
        if flag == 'E':
            self._electives.add(course)
        elif flag == 'R':
            self._requried.add(course)
        else:
            raise ValueError("value error forund unexpected flag in majors file")

    def grade_check(self, courses):
        completed_courses = {course for course, grade in courses.items() if grade in self._passing_grade}
        remaining_requried_course = self._requried - completed_courses
        if len(self._electives.intersection(completed_courses)) >= 1:
            remaining_elective_course = None
        else:
            remaining_elective_course = self._electives

        return[completed_courses, remaining_requried_course, remaining_elective_course]



class Repository:
    def __init__(self, path_dir):
        """taking path for the files student, grades, instructors
        """
        self.path_dir = path_dir
        self._students = dict()
        self._instructors = dict()
        self._major = dict()
        

        self.add_instructors(os.path.join(path_dir, "instructors.txt"))
        self.add_students(os.path.join(path_dir, "students.txt"))
        self.add_grades(os.path.join(path_dir, "grades.txt"))
        self.add_major(os.path.join(path_dir, "majors.txt"))

   
        self.student_pt()
        self.instructor_pt()
        self.major_pt()

    
    
    def add_students(self, path_dir):
        """Adding student information from the file students.txt
        """
        try:
            for cwid, name, major in file_read(path_dir, 3, seperator='\t', header=False):
                if cwid in self._students:
                    print(f"cwid {cwid} already read from the file")
                else:
                    self._students[cwid] = Student(cwid, name, major, self._major[major])
        except ValueError:
            print("value error")

    
    
    def add_instructors(self, path_dir):
        """Adding instructor information from the file instructors.txt
        """
        try:
            for cwid, name, dept in file_read(path_dir, 3, seperator='\t', header=False):
                if cwid in self._instructors:
                    print(f"cwid {cwid} already read from the list")
                else:
                    self._instructors[cwid] = Instructor(cwid, name, dept)
        except ValueError:
            print("value error")

    
    
    def add_grades(self, path_dir):
        """Adding grade information from the file grades.txt
        """
        for student_cwid, course, grade, instructor_cwid in file_read(path_dir, 4, seperator='\t', header=False):

            if student_cwid in self._students:
                self._students[student_cwid].add_course(course, grade)
            else:
                print(f"student cwid {student_cwid} not in the student file")

            if instructor_cwid in self._instructors:
                self._instructors[instructor_cwid].add_course(course)
            else:
                print(f"student cwit {instructor_cwid} not in file")

    
    def add_major(self, path_dir):
        try:
            for dept, requried, electives in file_read(path_dir, 3, seperator='\t', header=False):
                if dept in self._major:
                    self._major[dept].add_course(requried, electives)
                else:
                    self._major[dept] = major(dept)
                    self._major[dept].add_course(requried, electives)
        except ValueError:
            print("value error: ")


    
    
    def student_pt(self):
        pt = PrettyTable(
            field_names=['cwid', 'name', 'major', 'completed', 'remaining', 'elective'])
        for Student in self._students.values():
            pt.add_row(Student.pt_row())
        print("student pretty table")
        print(pt)
    
    
    
    def major_pt(self):
        pt = PrettyTable(
            field_names=['major', 'requried course', 'elective course'])
        for major in self._major.values():
            pt.add_row(major.pt_row())
        print(pt)

    
    
    def instructor_pt(self):
        pt = PrettyTable(field_names=['cwid', 'name', 'dept', 'course', 'students'])
        for Instructor in self._instructors.values():
            for course in Instructor.pt_row():
                pt.add_row(course)
        print("instructor pretty table")
        print(pt)


class Student:
    """Returns a dictionary with the student information
    """
    def __init__(self, cwid, name, major):
        self._cwid = cwid
        self._name = name
        self._major = major
        self._course = dict()  

    
    
    def add_course(self, course, grade):
        self._course[course] = grade

    
    
    def details(self):
        return[self._cwid,self._name,sorted(self._course.keys())]

  
    
    @staticmethod
    def fields():
        return['CWID','name','course']
    


class Instructor:
    """Returns a instructor with the student information
    """

    def __init__(self, cwid, name, dept):
        self._cwid = cwid
        self._name = name
        self.dept = dept
        self.course = defaultdict(int)

    
    
    def add_course(self, course):
        self.course[course] += 1

    
    
    def pt_row(self):
        for cr, st in self.course.items():
            yield [self._cwid, self._name, self.dept, cr, st]




class StudentTest(unittest.TestCase):
    
    def test_init(self):
        student = Student("10445200", "Tanvi", "Software Engineering")

        self.assertEqual(student._cwid, "10445200")
        self.assertEqual(student._name, "Tanvi")
        self.assertEqual(student._major, "Software Engineering")
        self.assertEqual(student._course, {})

    
    def test_add_course(self):
        student = Student("10445200", "Tanvi", "Software Engineering")
        student.add_course("SSW 810", "B")
        student.add_course("SSW 564", "A")

        self.assertEqual(student._course, {'SSW 810': 'B', 'SSW 564': 'A'})

    
    def test_details(self):
        student = Student("10103", "Baldwin, C", "SFEN")
        student.add_course("SSW 567", "A")
        student.add_course("SSW 564", "A-")

        student2 = Student("10115", "CS 545", "A")
        student2.add_course("SSW 540", "A-")
        student2.add_course("SSW 810", "A")

        self.assertEqual(student.details(), ['10103', 'Baldwin, C', ['SSW 564', 'SSW 567']])
        self.assertEqual(student.details(), ['10103', 'Baldwin, C', ['SSW 564', 'SSW 567']])

   
        

class InstructorTest(unittest.TestCase):
    
    def test_init(self):
        inst = Instructor("10445200", "Tanvi", "SE")
        self.assertEqual(inst._cwid, "10445200")
        self.assertEqual(inst._name, "Tanvi")
        self.assertEqual(inst.dept, "SE")
        self.assertEqual(inst.course, {})


    def test_add_course(self):
        inst = Instructor("10445200", "Tanvi", "Software Engineering")
        inst.add_course("SSW 810")
        inst.add_course("SSW 564")
        inst.add_course("SSW 810")

        self.assertEqual(inst.course, {'SSW 810' : 2, 'SSW 564' : 1})




class RepositoryTest(unittest.TestCase):

    def test_init(self):
        path_dir = "/Users/tanvihanamshet/Documents/stevens/810/HW09"
        rep1 = Repository(path_dir)
        expected_dict = {'10103': ['10103', 'Baldwin, C', ['CS 501', 'SSW 564', 'SSW 567', 'SSW 687']], '10115': ['10115', 'Wyatt, X', ['CS 545', 'SSW 564', 'SSW 567', 'SSW 687']], '10172': ['10172', 'Forbes, I', ['SSW 555', 'SSW 567']], '10175': ['10175', 'Erickson, D', ['SSW 564', 'SSW 567', 'SSW 687']], '10183': ['10183', 'Chapman, O', ['SSW 689']], '11399': ['11399', 'Cordova, I', ['SSW 540']], '11461': ['11461', 'Wright, U', ['SYS 611', 'SYS 750', 'SYS 800']], '11658': ['11658', 'Kelly, P', ['SSW 540']], '11714': ['11714', 'Morton, A', ['SYS 611', 'SYS 645']], '11788': ['11788', 'Fuller, E', ['SSW 540']]}
        result_dict = dict()

        for student_cwid, values in rep1._students.items():
            result_dict[student_cwid] = values.details()

        self.assertEqual(expected_dict, result_dict)

   
def main():
    unittest.main(exit=False, verbosity=2)

if __name__ == '__main__':
    main()
