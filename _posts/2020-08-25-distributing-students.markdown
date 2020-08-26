---
layout:	post
title:	"One Way To Select Two Million Students"
date:	2020-08-26 17:05:00 +0300
categories: Data
---

In late 60s, Turkish universities banded together and created Board of Higher Education to streamline their application processes. They decided, rather than each student applying to a number of universities, instead, they should enter a central exam, get a grade, list their preferences, and then be placed to a university, based **solely** on that grade.

With each passing year, the number of students taking the exam increases, as of 2019, some two and a half million students entered the exam.

I wondered how would one devise an algorithm to place the students. Surely, it must be simple, start from the highest grade, in each student, try to place the student to their highest available choice, and so on... right?

No.

See, I lied, it is not *one* exam, there are two exams, with four tests in each resulting in five different grade types, any school can chose to select their students in any one of those grade types (TYT, MF, TM, TS, DİL) *and* students can list 24 programs (majors in schools) in their application list, of any grade type. So you can chose 4 from each grade type and mix them together. Not only can you sort the students in one definitive way, you can't even group them in a definitive way, so we have an issue here.

So I thought about this problem, because it is a nice problem to ponder on, and did found an algorithm, there will be two sections to this post, first I will discus the algorithm in a theoretical sense and then I will discuss how I implemented the algorithm with slight variations in Python 3.8.

Before we go, I do want to note, in real life, these systems are often slow not because the ones who program or maintain them are not talented, they have to be able to perform their duties in very specific ways, within very tight constraints, and often must integrate with older, unmaintained systems, moreover there probably is a better way to do this, but still, let's take a look.


# The Algorithm

We start with a queue of students (`Q`), each student also has a queue of programs (`P`) sorted by the order of their preference, each program has a priority queue of students (`S`) inside it, this priority queue is sorted according to the student's grade in the preferred grade type of the program (for instance, IZTECH's Computer Engineering department selects its student based only on their MF grade, so this program's student priority queue is sorted according to the student's MF grade). When a new element (a `student`) is enqueued on this priority queue, this sorting is always uphold, therefore, any time a `student` is dequeued from this priority queue, it will always be the student with lowest grade in the preferred grade type.

Every program also has a limit on the students it can take, they have to stay below this limit. 

Then we do the following:

1. If `Q` is not empty, dequeue a student from Q, assign it to a variable `student`, if `Q` is empty, end the procedure.
2. If `student.P` (the `student`'s preference queue) is not empty, dequeue an item from `student.P` andd assign it to a variable `program`. If `student.P` is empty, jump to (1).
3. Check if `program.S` has space left in it.
    
    * **If there is space** enqueue `student` to `program.S`. Jump to (1).
    * **If there is no space, but the element at the front of the `program.S` has a lower grade then the `student`** dequeue an element from `program.S`, and enqueue it to `Q`, enqueue `student` to `program.S`. Jump to (1).
    * **If there is no space** Jump to (2)


# The Implementation

For implementation, I wrote two scripts, a generator script, and a sorting script, generator script is nothing special, it just generates a 3.5 million student records and ten thousand programs, generates preference lists, etc. Let's go over the sorting script!

Oh, one note, I am using a small package I programmed called [datalite], it allows Python 3.7+ dataclasses to be represented in a SQLite database one-to-one.

## Classes

There are multiple classes here, with multiple categories to go, there are classes that represent data in the database to be read, classes that represent the data during runtime which, some helper classes, and finally a simple class to write the end results.

All of these classes are contained in the [classes file].

### Helper Classes

To start, we have `Queue` and `PriorityQueue` classes. These classes facilitate `enqueue` and `dequeue` operations needed for the algorithm, although `list` could be used for the very same reason, these classes streamline the process.

```python
class Queue:
    def __init__(self):
        self._items: List[T] = []

    @property
    def is_empty(self) -> bool:
        return not len(self._items)

    def enqueue(self, element: T):
        self._items.insert(0, element)

    def dequeue(self) -> T:
        return self._items.pop()

    def peek(self) -> T:
        return self._items[-1]
```

A very simple queue class, it is implemented such that the back of the queue is the first element of the list, and the front of the queue is its last element, it also has a property to check if it is empty. Let's move onto the `PriorityQueue`.

```python
class PriorityQueue(Queue):
    def __init__(self, limit: int, sort_func: Callable):
        super().__init__()
        self.sort_func = sort_func
        self.limit = limit

    @property
    def is_full(self) -> bool:
        return len(self._items) >= self.limit

    def enqueue(self, obj_) -> None:
        self._items.append(obj_)
        self._items.sort(key=self.sort_func)

    def can_replace(self, obj) -> bool:
        return self.sort_func(obj) > self.sort_func(self.peek())
```

This is a subclass of the `Queue` that automatically sorts itself. This way, the front of the Queue will hold the smallest value and the back will hold the largest, the call the sort might seem like a bad idea here, since sorting it in every enqueue would harm the performance. However, my rationale was that, whatever insertion algorithm I would devise to keep the list sorted would be slower than the built-in method `sort`, since it is written in `C`. Sorting key is a function.

In either case, this class also adds in `can_replace`, which can be used to check if an item is larger than any item in the `PriorityQueue`, this will be used to check for the second condition of the (3)rd step of the algorithm. Finally, we have a method to check if the queue is full, as `PriorityQueue` has a limit.

### Data Holder Classes

The data was originally generated as, and was saved to the database using these classes, keep in mind this class is not appropriate for our algorithm, so we will convert these to the actual, runtime classes, they will be used to make our distribution. But first, let's take a look at what is holding our data.

```python
@datalite(db_path="test.db")
@dataclass
class Student:
    type_0_grade: float
    type_1_grade: float
    type_2_grade: float
    type_3_grade: float
    type_4_grade: float
    comma_sep_ids: str
```

Here, first five fields hold grades of different types, whereas, `comma_sep_ids` hold a student's preference of programs, separated by commas, with each id representing a program. This data is converted to a table called `student` in database file `test.db` and looks something like this:

| obj_id | comma_sep_ids | type_0_grade | ... |
| --- | --- | --- | --- |
| 1 | 3243, 2343, ... | 434.43 | ...
| 2 | 2412, 1678, ... | 345.23 | ...

```python
@datalite(db_path="test.db")
@dataclass
class Program:
    quota: int
    type_: int
```

`quota` is the maximum number of students that this program takes, randomly selected, `type_` denotes which grade type is used to select students to this program. This, in turn, looks like this:

| obj_id | quota | type_ | ... |
| --- | --- | --- | --- |
| 1 | 90 | 3 | ...
| 2 | 34 | 1 | ...

You might have realized that, both of these classes generate a secret column, `obj_id` is used to distinguish the objects.

### Runtime Classes

There are two classes we will be using to hold `Program` and `Student` classes, these are `UniversityProgram` and `StudentCandidate`, I know, not very creative. Let's take a look at these more conventional classes:

```python
@dataclass
class StudentCandidate:
    """
    This is to represent a student in
    the program rather than in the
    database.
    """
    uuid: int
    selected_program: int
    selections: Queue
    grades: tuple
```

This is a very simple class, it has a `UUID`, which mirrors `obj_id`, `selected_program`, which holds which program this student has chosen at any given moment in the runtime, if this value is `-1` then it means this student hasn't chosen one yet. `selections` is a `Queue` that holds this student's selected programs in order of their preference and finally, `grades` which hold a tuple of five grades, indexed by their grade type.

```python
class UniversityProgram:
    """
    This represents a program
    in the well, program.
    """
    def __init__(self, uuid: int, type_: int, quota: int):
        self.uuid = uuid  # Object ID of the university program.
        self.type_ = type_  # Type of the program
        self.selected_students = PriorityQueue(quota, lambda x: x.grades[self.type_])
```

This is a conventional `Python 3.x` class, it again has a `uuid` field corresponding to the `obj_id`, it has a `self.type_` that holds the grade type used to select students, and a `PriorityQueue` of selected students at any given time in runtime, its key is a lambda function that basically tells the `PriorityQueue` to look at a `StudentCandidate` object's preferred grade to sort them.

 We will have to convert Data Holders to Runtime Classes in the program. But first, we have a final type of class.

 ### Record Class

 Finally, we have a class to hold the final student records, a very minimal one.

 ```python
@datalite(db_path='db.db')
@dataclass
class StudentRecord:
    uuid: int
    program: int
    has_entered: str
 ```

Although `datalite` will assign an `obj_id` to each of these, positions of the students *might* get mixed up, so I put a field for the original `uuid` as well as which `program` the student is assigned at, and a `has_entered` field (Which is redundant but might be helpful for poor people working on statistics).

With the classes over, let's check the actual code.

## The Actual Code

Ah, the actual code, finally, let's start with our entry point in the [main script file]

```python
if __name__ == '__main__':
    run_the_main_program()
```

This runs a function called `run_the_main_program`, function names are named a bit long because I am using them for logging. I will reefer to this function as the main function for everyone's convenience.

```python
def run_the_main_program() -> None:
    programs, students = fetch_records()
    uni_programs = translate_programs(programs)
    students_queue, student_refs = translate_students(students, uni_programs)
    place_students(students_queue)
    student_records = convert_final_records(student_refs)
    save_results_to_database(student_records)
```

This does all the work, first it fetches the records, then converts them to runtime classes, then places the students, converts them to recording classes and finally saves the results in a database. To start, let's go line by line:

```python
programs, students = fetch_records()
```

This is a function call to:

```python
def fetch_records() -> Tuple[Tuple[Program, ...], Tuple[Student, ...]]:
    programs = fetch_all(Program)
    students = fetch_all(Student)
    return programs, students
```

This uses `datalite`'s `fetch_all` function and convert all the information in tables to their corresponding objects in tuples, returns both tuples. Next line in the main:

```python
uni_programs = translate_programs(programs)
```

Passes the the tuple of `Program` objects to `translate_programs`, this will translate them to a list of `UniversityProgram` objects. Their corresponding runtime class.

```python
def translate_programs(programs) -> List[UniversityProgram]:
    uni_programs = []
    for program in programs:
        uni_programs.append(UniversityProgram(program.obj_id, int(program.type_), int(program.quota)))
    del programs
    return uni_programs
```

Only thing to not here is that we are deleting the programs, this is because we want this variable to be garbage collected, though this is not necessarily gonna work. The next line in the main function.

```python
students_queue, student_refs = translate_students(students, uni_programs)
```

This is an interesting one, it is obvious we are converting `Student` tuple to a `StudentCandidate` queue, but what is that second variable?

```python
def translate_students(students, uni_programs) -> Tuple[Queue, List[StudentCandidate]]:
    students_queue = Queue()
    for student in students:
        selections_list = [uni_programs[int(i) - 1] 
                            for i in student.comma_sep_ids.split(', ')]
        selections_queue = Queue()
        for selection in selections_list:
            selections_queue.enqueue(selection)
        new_student = StudentCandidate(...)
        students_queue.enqueue(new_student)
    del students
    return students_queue, students_queue.get_list()

```

This may look a bit more complicated but it is pretty simple, first we create a queue for our `StudentCandidate` objects, and we convert and transfer the `Student` instances, but alongside with the `Queue`, we are also returning a list containing a reference to every `StudentCandidate` object in the program. this is what the `student_refs` variable was, it is holding onto these references, because by the time our algorithm is complete, the `Queue` will be empty, but since they are referring to the same objects, this list will also contain the updates. So we will use this class to actually save the records to databases in the end.

The next line is why we are here:

```python
place_students(students_queue)
```

And this function is basically the algorithm we devised.

```python
def place_students(students_queue) -> None:
    while not students_queue.is_empty:
        s: StudentCandidate = students_queue.dequeue()
        while not s.selections.is_empty:
            p: UniversityProgram = s.selections.dequeue()
            if not p.selected_students.is_full:
                s.selected_program = p.uuid
                p.selected_students.enqueue(s)
                break
            elif p.selected_students.is_full and p.selected_students.can_replace(s):
                toss: StudentCandidate = p.selected_students.dequeue()
                toss.selected_program = -1
                students_queue.enqueue(toss)
                p.selected_students.enqueue(s)
                break
```

This is more or less identical to the algorithm we devised, other than the `toss.selected_program = -1`, we set this in order to see where (if anywhere) did the student got placed in the end.

Oh, and one small note, since we are `dequeue`ing constantly, objects no longer references might be garbage collected in this step, freeing some memory. This *may* occur with the `UniversityProgram` objects.

The `student_queue` is now empty, but the `student_refs` have all the student information updated, we have distributed all the students to their programs, now it is time to record this information back to the database, the next line calls a function to convert the `student_refs` list to a list of `StudentRecords` our final data holder class for recording data.

```python
student_records = convert_final_records(student_refs)
```

This function is quite straightforward as well:

```python
def convert_final_records(student_refs) -> List[StudentRecord]:
    student_records = []
    while student_records:
        student = student_records.pop()
        student_records.append(StudentRecord(student.uuid,
                                             student.selected_program,
                                             str(student.selected_program != -1)))
    return student_records
```

Because we used `pop` we have once again freed references to `StudentCandidate` objects, since they are no longer useful, this *might* signal the garbage collector to clean them, freeing memory once more. Otherwise, this is quite straightforward, converting data once more, and then returning the list.

This brings us to our final line:

```python
save_results_to_database(student_records)
```

This is actually a one line function:

```python
def save_results_to_database(student_results) -> None:
    create_many(student_results, False)
```

This basically saves all the records to disk, to the `db.db` file, to be more specific, using [a special function from datalite], `create_many` is optimized to insert many records to a SQLite database at once, the second `False` argument turns off some memory protections that would stop `db.db` from corrupting had the process was cut short, you *probably* don't want to do this in production.

This will result in a database that looks like this:

| obj_id | has_entered | program | uuid |
| --- | --- | --- | --- |
| 1 | True | 4356 | 3500000
| 2 | False | -1 | 3499999

# How long Does It Take?

## The Implementation

I programmed the following decorator:

```python
def record_time(f):
    def decorator(*args, **kwargs):
        time_before = time()
        vals = f(*args, **kwargs)
        print(f"Time it took to {f.__name__.replace('_', ' ')}",
              f"is {round(time() - time_before, 3)} seconds")
        return vals
    return decorator

```

Using this, it was quite trivial to get the time of each function by simply putting `@record_time` on top of them. For reference, I should mention the machine this was run on is a i7-9700.

|Function|Time (s)|
| --- | --- |
| `fetch_records` | 16.472 |
| `translate_programs` | 0.09 |
| `translate_students` | 5791.643 |
| `place_students` | 230.922 |
| `convert_final_records` | 2.141 |
| `save_results_to_database` | 39.836 |
| `run_the_main_program` (total) | 6081.862 |

So, as you can see, whole process takes around an hour and a half with the largest time spent being `translate_students`. Once the objects are loaded to the memory, the entire process only takes around four minutes, quite nice if you ask me.

## The Algorithm

Although, obviously we cannot mention a concrete time for a theoretical algorithm, I have calculated the worst-case complexity of the algorithm in this [poorly written paper].

To cut it short, it is O(n³)... sort of, the problem arises because the maximum amount of steps it would take the complete algorithm comes down to `student_count * (choice_count**2)/2`, here `choice_count` reefers to the number of selections each student can make, which is *miniscule* compared to the number of students, and doesn't change often, therefore, practically, I think this algorithm has a practical complexity of O(n).

# Closing Remarks

Well, this was quite a nice problem to solve. The algorithm seems to be working quite nice and as expected, there is room to simplify the class structure, perhaps by unifying the data classes and runtime classes. Also, implementing the algorithm in a compiled language with manual memory management *cough* `C`, would probably significantly speed up the program as well.

There likely is a better algorithm out there, maybe even that is used right now, but for the time being, this is a small algorithm to solve the problem of placing students.

[datalite]: https://github.com/ambertide/datalite
[classes file]: https://github.com/ambertide/ExamTest/blob/master/src/internals/classes.py
[main script file]: https://github.com/ambertide/ExamTest/blob/master/src/sort_students.py
[a special function from datalite]: https://datalite.readthedocs.io/en/latest/datalite.html#datalite.mass_actions.create_many
[poorly written paper]: https://github.com/ambertide/ExamTest/raw/master/docs/article.pdf