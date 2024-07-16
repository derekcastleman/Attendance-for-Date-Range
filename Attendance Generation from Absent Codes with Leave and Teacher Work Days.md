# Attendance Generation from Absent Codes with Leave and Teacher Work Days

Aeries does not allow for us to find the percent present for a student based on a date range only allowing for a search on Year to Date attendance. This creates issues if we want to look at the quarter, semester or monthly attendance for students.

The following code allows for the percent attendance for a student to be calculated using a query that searches for the All Day codes for the year as well as the Enrollment Data for the students.

The only input that is required is the date range of interest as well as answering questions on the days off of school pertaining to particular school holidays. Only holidays within the time range of interest have to be input. The rest can be skipped by hitting enter.

__Query for Absent Codes__: LIST ATT STU ATT.SC STU.ID ATT.DY ATT.AL ATT.DT ATT.RS ATT.DTS ATT.ACO

__Query for Enrollment__: LIST STU ID LN FN SC GR ED LD

Run the queries through Aeries and input the file destination into the appropriate spots.


```python
import numpy as np
import pandas as pd
```


```python
# Query for the All Day codes for the school year
# LIST ATT STU ATT.SC STU.ID ATT.DY ATT.AL ATT.DT ATT.RS ATT.DTS ATT.ACO

absent_codes = pd.read_excel(r"C:\Users\derek.castleman\Desktop\PrintQueryToExcel_20240715_130847_6503179.xlsx")

# Obtain enrollment data for the students
# LIST STU ID LN FN SC GR ED LD

enrollment = pd.read_excel(r"C:\Users\derek.castleman\Desktop\PrintQueryToExcel_20240715_131621_02776e7.xlsx")

# Destination of output file
output = "C:\\Users\\derek.castleman\\Desktop\\Attendance.xlsx"
```


```python
absent_codes
```

## Selecting Date Range

Inputting the date range of interest that you want to generate the attendance data for and then converting it into datetime.


```python
absent_codes['Date']= pd.to_datetime(absent_codes['Date']) # Changes absent date to datetime
absent_codes
```


```python
a = input('What is the start date you are interested in (mm/dd/yyyy):          ') #Input start date
```


```python
a = pd.to_datetime(a) # Change start date to datetime
a
```


```python
b = input('What is the end date you are interested in (mm/dd/yyyy):          ') #Input end date
```


```python
b = pd.to_datetime(b) # Turn end date to date time
b
```


```python
# Filters date range from All Day code table
dates_interested = absent_codes[(absent_codes['Date'] >=a) & (absent_codes['Date'] <=b)]
dates_interested
```

## Calculating Absences, Tardies and Truancies

Absences will be calculated using the All Day codes which coincide with an absent for the student for the day.

Unexcused absences will be filtered by the codes that relate to this kind of absence.

Tardies will focus on the codes that are related to tardies.

Truancies will be students that have an All Day code of >30.


```python
# Filtering for rows that correspond to absences
absent_students = dates_interested[(dates_interested['All day'] == 'R') | (dates_interested['All day'] == '0') |
                                  (dates_interested['All day'] == 'I') | (dates_interested['All day'] == 'L') | 
                                  (dates_interested['All day'] == 'M') | (dates_interested['All day'] == 'X') |
                                  (dates_interested['All day'] == '7') | (dates_interested['All day'] == 'A') |
                                  (dates_interested['All day'] == 'Q') | (dates_interested['All day'] == 'S') |
                                  (dates_interested['All day'] == 'U') | (dates_interested['All day'] == 'P')]
absent_students
```


```python
# Adding a column that gives one day for each absent code
absent_students['Absent'] = 1
absent_students
```


```python
# Grouping by school and student ID to calculate total number of days absent
absent = absent_students.groupby(by=['School', 'Student ID'])['Absent'].sum().reset_index()
absent
```


```python
# Filters for the codes that relate to unexcused absences
unexcused_absent_students = dates_interested[(dates_interested['All day'] == '7') |
                                  (dates_interested['All day'] == 'Q') | (dates_interested['All day'] == 'S') |
                                  (dates_interested['All day'] == 'U')]
unexcused_absent_students
```


```python
# Gives one day for each unexcused absence
unexcused_absent_students['Unexcused Absences'] = 1
unexcused_absent_students
```


```python
# Sums up the number of unexcused absences for each students
unexcused_absent = unexcused_absent_students.groupby(by=['School', 'Student ID'])['Unexcused Absences'].sum().reset_index()
unexcused_absent
```


```python
# Filters for truancies
truancies = dates_interested[dates_interested['All day'] == 'Z']
truancies
```


```python
# Gives on truancy for each day
truancies['Truant'] = 1
truancies
```


```python
# Summs up the truancies for each student
truant = truancies.groupby(by=['School', 'Student ID'])['Truant'].sum().reset_index()
truant
```


```python
# Filters for the tardies for each student
tardy_students = dates_interested[(dates_interested['All day'] == 'T') | (dates_interested['All day'] == 'D') |
                                  (dates_interested['All day'] == 'C')]
tardy_students
```


```python
# Gives one tardy for each day
tardy_students['Tardy'] = 1
tardy_students
```


```python
# Sums up the tardies for each student
tardies = tardy_students.groupby(by=['School', 'Student ID'])['Tardy'].sum().reset_index()
tardies
```

## Calculating Days Enrolled

The days that the students are enrolled at the school for the time period that is selected will be calculated.


```python
enrollment
```


```python
# Changing the enter date to datetime format
enrollment['Enter Date']= pd.to_datetime(enrollment['Enter Date'])
enrollment
```


```python
# Creating a function that sets different dates based on when the student enrolls and time period selected
def f(row):
    if row['Enter Date'] <= a: #Enter date is first date selected if student enrolled prior
        val = a
    else:
        val = row['Enter Date'] #Enter date is date of actual enrollment if after start date
    return val
```


```python
# Creates enrollment column using function defined above
enrollment['Enrollment'] = enrollment.apply(f, axis=1)
enrollment
```

## Inputing Holidays

The dates for holidays can be input for the time range that is of concern. Any other holiday outside of the range can be skipped by hitting enter.


```python
# Takes an input for the date then converts it to datetime and a dataframe
c = input('When is Labor Day (mm/dd/yyyy) - Hit enter if not in time range?:      ')
c = pd.to_datetime(c)
c=[c]
c = pd.DataFrame(c, columns=['Dates'])
c
```


```python
c["Date"] = pd.to_datetime(c['Dates']).dt.date
c.info()
```


```python
d = input('When is first date of Fall Break (mm/dd/yyyy)? - Hit enter if not in time range:      ')
d = pd.to_datetime(d)
d
```


```python
e = input('When is last date of Fall Break (mm/dd/yyyy)? - Hit enter if not in time range:      ')
e = pd.to_datetime(e)
e
```


```python
# If the start and end date are not null it will create a dataframe between the date range
if pd.notna(d) and pd.notna(e):
    fall_break = pd.date_range(d,e,freq='d')
    fall_break = pd.DataFrame(fall_break, columns =['Dates'])
    fall_break["Date"] = fall_break['Dates'].dt.date
else:
    fall_break = None # Returns null if the start and end date are not entered
```


```python
fall_break
```


```python
f = input('When is Veterans Day (mm/dd/yyyy)? - Hit enter if not in time range:      ')
f = pd.to_datetime(f)
f=[f]
f = pd.DataFrame(f, columns=['Dates'])
f["Date"] = f['Dates'].dt.date
f
```


```python
g = input('When is first date of Thanksgiving Break (mm/dd/yyyy)? - Hit enter if not in time range:      ')
g = pd.to_datetime(g)
g
```


```python
h = input('When is last date of Thanksgiving Break (mm/dd/yyyy)? - Hit enter if not in time range:      ')
h = pd.to_datetime(h)
h
```


```python
if pd.notna(g) and pd.notna(h):
    thanksgiving_break = pd.date_range(g,h,freq='d')
    thanksgiving_break = pd.DataFrame(thanksgiving_break, columns =['Dates'])
    thanksgiving_break["Date"] = thanksgiving_break['Dates'].dt.date
else:
    thanksgiving_break = None
```


```python
i = input('List first date of Winter Break (mm/dd/yyyy)? - Hit enter if not in time range:      ')
i = pd.to_datetime(i)
i
```


```python
j = input('List last date of Winter Break (mm/dd/yyyy)? - Hit enter if not in time range:      ')
j = pd.to_datetime(j)
j
```


```python
if pd.notna(i) and pd.notna(j):
    winter_break = pd.date_range(i,j,freq='d')
    winter_break = pd.DataFrame(winter_break, columns =['Dates'])
    winter_break["Date"] = winter_break['Dates'].dt.date
else:
    winter_break = None
```


```python
k = input('When is MLK Day (mm/dd/yyyy)? - Hit enter if not in time range:      ')
k = pd.to_datetime(k)
k=[k]
k = pd.DataFrame(k, columns=['Dates'])
k["Date"] = k['Dates'].dt.date
k
```


```python
s = input('When is Lincolns Birthday if applicable (mm/dd/yyyy)? - Hit enter if not in time range:      ')
s = pd.to_datetime(s)
s=[s]
s = pd.DataFrame(s, columns=['Dates'])
s["Date"] = s['Dates'].dt.date
s
```


```python
l = input('When is Presidents Day (mm/dd/yyyy)? - Hit enter if not in time range:      ')
l = pd.to_datetime(l)
l=[l]
l = pd.DataFrame(l, columns=['Dates'])
l["Date"] = l['Dates'].dt.date
l
```


```python
m = input('When does Spring Break begin (mm/dd/yyyy)? - Hit enter if not in time range:      ')
m = pd.to_datetime(m)
m
```


```python
n = input('When does Spring Break end (mm/dd/yyyy)? - Hit enter if not in time range:      ')
n = pd.to_datetime(n)
n
```


```python
if pd.notna(m) and pd.notna(n):
    spring_break = pd.date_range(m,n,freq='d')
    spring_break = pd.DataFrame(spring_break, columns =['Dates'])
    spring_break["Date"] = spring_break['Dates'].dt.date
else:
    spring_break = None
```


```python
o = input('When is Cesar Chavez Day (mm/dd/yyyy)? - Hit enter if not in time range:      ')
o = pd.to_datetime(o)
o=[o]
o = pd.DataFrame(o, columns=['Dates'])
o["Date"] = o['Dates'].dt.date
o
```


```python
p = input('When is Easter Holiday (mm/dd/yyyy)? - Hit enter if not in time range:      ')
p = pd.to_datetime(p)
p=[p]
p = pd.DataFrame(p, columns=['Dates'])
p["Date"] = p['Dates'].dt.date
p
```


```python
q = input('When is Memorial Day (mm/dd/yyyy)? - Hit enter if not in time range:      ')
q = pd.to_datetime(q)
q=[q]
q = pd.DataFrame(q, columns=['Dates'])
q["Date"] = q['Dates'].dt.date
q
```

## Teacher Work Days


```python
aa = input('How many teacher work days are there?:          ')
```


```python
if aa == '0':
    zz = None
elif aa == '1':
    bb = input('When is first teacher day (mm/dd/yyyy)?:      ')
    bb = pd.to_datetime(bb)
    bb=[bb]
    bb = pd.DataFrame(bb, columns=['Dates'])
    bb["Date"] = bb['Dates'].dt.date
    zz = bb
elif aa == '2':
    bb = input('When is first teacher day (mm/dd/yyyy)?:      ')
    bb = pd.to_datetime(bb)
    bb=[bb]
    bb = pd.DataFrame(bb, columns=['Dates'])
    bb["Date"] = bb['Dates'].dt.date
    cc = input('When is second teacher day (mm/dd/yyyy)?:      ')
    cc = pd.to_datetime(cc)
    cc=[cc]
    cc = pd.DataFrame(cc, columns=['Dates'])
    cc["Date"] = cc['Dates'].dt.date
    zz = pd.concat([bb, cc])
elif aa == '3':
    bb = input('When is first teacher day (mm/dd/yyyy)?:      ')
    bb = pd.to_datetime(bb)
    bb=[bb]
    bb = pd.DataFrame(bb, columns=['Dates'])
    bb["Date"] = bb['Dates'].dt.date
    cc = input('When is second teacher day (mm/dd/yyyy)?:      ')
    cc = pd.to_datetime(cc)
    cc=[cc]
    cc = pd.DataFrame(cc, columns=['Dates'])
    cc["Date"] = cc['Dates'].dt.date
    dd = input('When is third teacher day (mm/dd/yyyy)?:      ')
    dd = pd.to_datetime(dd)
    dd=[dd]
    dd = pd.DataFrame(dd, columns=['Dates'])
    dd["Date"] = dd['Dates'].dt.date
    zz = pd.concat([bb, cc, dd])
elif aa == '4':
    bb = input('When is first teacher day (mm/dd/yyyy)?:      ')
    bb = pd.to_datetime(bb)
    bb=[bb]
    bb = pd.DataFrame(bb, columns=['Dates'])
    bb["Date"] = bb['Dates'].dt.date
    cc = input('When is second teacher day (mm/dd/yyyy)?:      ')
    cc = pd.to_datetime(cc)
    cc=[cc]
    cc = pd.DataFrame(cc, columns=['Dates'])
    cc["Date"] = cc['Dates'].dt.date
    dd = input('When is third teacher day (mm/dd/yyyy)?:      ')
    dd = pd.to_datetime(dd)
    dd=[dd]
    dd = pd.DataFrame(dd, columns=['Dates'])
    dd["Date"] = dd['Dates'].dt.date
    ee = input('When is fourth teacher day (mm/dd/yyyy)?:      ')
    ee = pd.to_datetime(ee)
    ee=[ee]
    ee = pd.DataFrame(ee, columns=['Dates'])
    ee["Date"] = ee['Dates'].dt.date
    zz = pd.concat([bb, cc, dd, ee])
elif aa == '5':
    bb = input('When is first teacher day (mm/dd/yyyy)?:      ')
    bb = pd.to_datetime(bb)
    bb=[bb]
    bb = pd.DataFrame(bb, columns=['Dates'])
    bb["Date"] = bb['Dates'].dt.date
    cc = input('When is second teacher day (mm/dd/yyyy)?:      ')
    cc = pd.to_datetime(cc)
    cc=[cc]
    cc = pd.DataFrame(cc, columns=['Dates'])
    cc["Date"] = cc['Dates'].dt.date
    dd = input('When is third teacher day (mm/dd/yyyy)?:      ')
    dd = pd.to_datetime(dd)
    dd=[dd]
    dd = pd.DataFrame(dd, columns=['Dates'])
    dd["Date"] = dd['Dates'].dt.date
    ee = input('When is fourth teacher day (mm/dd/yyyy)?:      ')
    ee = pd.to_datetime(ee)
    ee=[ee]
    ee = pd.DataFrame(ee, columns=['Dates'])
    ee["Date"] = ee['Dates'].dt.date
    ff = input('When is fifth teacher day (mm/dd/yyyy)?:      ')
    ff = pd.to_datetime(ff)
    ff=[ff]
    ff = pd.DataFrame(ff, columns=['Dates'])
    ff["Date"] = ff['Dates'].dt.date
    zz = pd.concat([bb, cc, dd, ee, ff])
elif aa == '6':
    bb = input('When is first teacher day (mm/dd/yyyy)?:      ')
    bb = pd.to_datetime(bb)
    bb=[bb]
    bb = pd.DataFrame(bb, columns=['Dates'])
    bb["Date"] = bb['Dates'].dt.date
    cc = input('When is second teacher day (mm/dd/yyyy)?:      ')
    cc = pd.to_datetime(cc)
    cc=[cc]
    cc = pd.DataFrame(cc, columns=['Dates'])
    cc["Date"] = cc['Dates'].dt.date
    dd = input('When is third teacher day (mm/dd/yyyy)?:      ')
    dd = pd.to_datetime(dd)
    dd=[dd]
    dd = pd.DataFrame(dd, columns=['Dates'])
    dd["Date"] = dd['Dates'].dt.date
    ee = input('When is fourth teacher day (mm/dd/yyyy)?:      ')
    ee = pd.to_datetime(ee)
    ee=[ee]
    ee = pd.DataFrame(ee, columns=['Dates'])
    ee["Date"] = ee['Dates'].dt.date
    ff = input('When is fifth teacher day (mm/dd/yyyy)?:      ')
    ff = pd.to_datetime(ff)
    ff=[ff]
    ff = pd.DataFrame(ff, columns=['Dates'])
    ff["Date"] = ff['Dates'].dt.date
    gg = input('When is sixth teacher day (mm/dd/yyyy)?:      ')
    gg = pd.to_datetime(gg)
    gg=[gg]
    gg = pd.DataFrame(gg, columns=['Dates'])
    gg["Date"] = gg['Dates'].dt.date
    zz = pd.concat([bb, cc, dd, ee, ff, gg])
elif aa == '7':
    bb = input('When is first teacher day (mm/dd/yyyy)?:      ')
    bb = pd.to_datetime(bb)
    bb=[bb]
    bb = pd.DataFrame(bb, columns=['Dates'])
    bb["Date"] = bb['Dates'].dt.date
    cc = input('When is second teacher day (mm/dd/yyyy)?:      ')
    cc = pd.to_datetime(cc)
    cc=[cc]
    cc = pd.DataFrame(cc, columns=['Dates'])
    cc["Date"] = cc['Dates'].dt.date
    dd = input('When is third teacher day (mm/dd/yyyy)?:      ')
    dd = pd.to_datetime(dd)
    dd=[dd]
    dd = pd.DataFrame(dd, columns=['Dates'])
    dd["Date"] = dd['Dates'].dt.date
    ee = input('When is fourth teacher day (mm/dd/yyyy)?:      ')
    ee = pd.to_datetime(ee)
    ee=[ee]
    ee = pd.DataFrame(ee, columns=['Dates'])
    ee["Date"] = ee['Dates'].dt.date
    ff = input('When is fifth teacher day (mm/dd/yyyy)?:      ')
    ff = pd.to_datetime(ff)
    ff=[ff]
    ff = pd.DataFrame(ff, columns=['Dates'])
    ff["Date"] = ff['Dates'].dt.date
    gg = input('When is sixth teacher day (mm/dd/yyyy)?:      ')
    gg = pd.to_datetime(gg)
    gg=[gg]
    gg = pd.DataFrame(gg, columns=['Dates'])
    gg["Date"] = gg['Dates'].dt.date
    hh = input('When is seventh teacher day (mm/dd/yyyy)?:      ')
    hh = pd.to_datetime(hh)
    hh=[hh]
    hh = pd.DataFrame(hh, columns=['Dates'])
    hh["Date"] = hh['Dates'].dt.date
    zz = pd.concat([bb, cc, dd, ee, ff, gg, hh])
```


```python
zz
```

## Removing Holidays

The holidays that were input will be concatenated into one dataframe. The range of dates that were selected will be generated and matched with the holidays. Then the dates that correspond with the holidays will be removed from the time range of interest.


```python
# The input holidays will be concatenated into one dataframe
holidays = pd.concat([c, fall_break, f, thanksgiving_break, winter_break, k, l, spring_break, o, p, q, s, zz]).reset_index(drop=True)
holidays
```


```python
holidays = holidays[['Dates']] #Select the datetime column
holidays = holidays.rename(columns={"Dates": "Holidays"}) #Change the name of column to holidays
holidays
```


```python
# The dates between the selected range will be generated
date_range = pd.date_range(a,b,freq='B')
date_range = pd.DataFrame(date_range, columns =['Dates'])
date_range
```


```python
# Holidays are matched with corresponding dates in date range
holiday_match = pd.merge(date_range, holidays, how='left', left_on='Dates', right_on='Holidays')
holiday_match
```


```python
# The dates without holidays are selected
dates = holiday_match[holiday_match.Holidays.isnull()].reset_index(drop=True)
dates
```


```python
# The holidays column is dropped
dates = dates[['Dates']]
dates
```


```python
# A column for day is generated
dates['Day'] = 'Day'
dates
```


```python
# A countdown of days enrolled by date is generated
#dates['Enrolled'] = dates.groupby(['Day']).cumcount(ascending=False)+1
#dates
```


```python
# The day column is dropped leaving enrolled days for each date
dates = dates.drop(columns=['Day'])
dates
```

## Combining All Tables

All the tables will be combined in this section, giving the number of days each student has been enrolled by matching the date and the enrollment columns.

All of the attendance tables will then be added to create columns that represent each one.


```python
enrollment
```


```python
enrollment['Leave Date']= pd.to_datetime(enrollment['Leave Date'])
enrollment
```


```python
enrollment['Leave Date'].fillna(b, inplace=True)
enrollment
```


```python
def generate_date_range(row):
    return pd.date_range(start=row['Enrollment'], end=row['Leave Date'])
```


```python
enrollment['date_range'] = enrollment.apply(generate_date_range, axis=1)
enrollment = enrollment.explode('date_range').reset_index(drop=True)
enrollment
```


```python
enrollment_fixed = pd.merge(enrollment, dates, how='inner', left_on='date_range', right_on='Dates')
enrollment_fixed
```


```python
enrollment_fixed['Enrolled'] = 1
enrollment_fixed
```


```python
enrollment_fixed = enrollment_fixed.groupby(['Student ID', 'Last Name', 'First Name', 'School', 'Grade', 
                                'Enrollment', 'Leave Date'])['Enrolled'].sum().reset_index()
enrollment_fixed
```


```python
enrollment = enrollment_fixed[enrollment_fixed['Enrolled'] >= 31]
enrollment
```


```python
absent
```


```python
# Adding days absent column
absent_enrolled = pd.merge(enrollment, absent, how='left', on=['Student ID', 'School' ])
absent_enrolled
```


```python
# Giving students with no absences a zero
absent_enrolled["Absent"] = absent_enrolled["Absent"].fillna(0)
absent_enrolled
```


```python
# Create a present column by subtracting days absent from those enrolled
absent_enrolled['Present'] = absent_enrolled['Enrolled'] - absent_enrolled['Absent']
absent_enrolled
```


```python
present = absent_enrolled[['Student ID', 'Last Name', 'First Name', 'School', 'Grade', 'Enrollment', 'Enrolled', 'Present',
                          'Absent']] #Moves the present column over
present
```


```python
# Calculates percent present by dividing days present by days enrolled
present['% Present'] = present['Present'] / present['Enrolled']
present
```


```python
unexcused_absent
```


```python
# Adds unexcused absences column
unexcused = pd.merge(present, unexcused_absent, how='left', on=['Student ID', 'School'])
unexcused
```


```python
# Gives a value of zero for students who do not have one
unexcused["Unexcused Absences"] = unexcused["Unexcused Absences"].fillna(0)
unexcused
```


```python
truant
```


```python
# Adds the truant column to the dataframe
truant = pd.merge(unexcused, truant, how='left', on=['Student ID', 'School' ])
truant
```


```python
# Gives a zero to students who do not have one
truant["Truant"] = truant["Truant"].fillna(0)
truant
```


```python
tardies
```


```python
# Adds the tardies column to the dataframe
tardies = pd.merge(truant, tardies, how='left', on=['Student ID', 'School' ])
tardies
```


```python
# Gives a zero to students who do not have one
tardies["Tardy"] = tardies["Tardy"].fillna(0)
tardies
```


```python
# Generates a csv file from the final dataframe
import base64
from IPython.display import HTML

def create_download_link( df, title = "Attendance for Date Range", filename = "Attendance for Date Range"):
    csv = df.to_csv(index=False)
    b64 = base64.b64encode(csv.encode())
    payload = b64.decode()
    html = '<a download="{filename}" href="data:text/csv;base64,{payload}" target="_blank">{title}</a>'
    html = html.format(payload=payload,title=title,filename=filename)
    return HTML(html)

create_download_link(tardies)
```


```python
tardies
```


```python
school = tardies.groupby(by=['School'])['Enrolled', 'Present'].sum().reset_index()
school
```


```python
delano = school[(school['School'] == 1) | (school['School'] == 2) | (school['School'] == 4) ]
delano
```


```python
column_sums = delano.sum(axis=0)
delano = pd.DataFrame(column_sums).transpose()
delano
```


```python
delano['School'] = delano['School'].replace(7.0, 'Delano')
delano
```


```python
lh = school[(school['School'] == 6) | (school['School'] == 7) | (school['School'] == 8) ]
lh
```


```python
column_sums = lh.sum(axis=0)
lh = pd.DataFrame(column_sums).transpose()
lh
```


```python
lh['School'] = lh['School'].replace(21.0, 'Lost Hills')
lh
```


```python
school = pd.concat([school, delano, lh])
school
```


```python
replacement_dict = {1.0: 'Delano HS', 2: 'Delano MS', 4.0:'Delano ES', 6.0:'Lost Hills ES', 
                   7.0:'Lost Hills MS', 8.0: 'Lost Hills HS'}
school['School'].replace(replacement_dict, inplace=True)
school
```


```python
school['Percent Present'] = school['Present']/school['Enrolled']
school
```


```python
tardies
```


```python
def f(row):
    if row['% Present'] <= .9: #Enter date is first date selected if student enrolled prior
        val = 1
    else:
        val = 0 #Enter date is date of actual enrollment if after start date
    return val
```


```python
tardies['Chronic'] = tardies.apply(f, axis=1)
tardies
```


```python
tardies['Enrollment'] = 1
tardies
```


```python
chronic = tardies.groupby(by=['School'])['Enrollment', 'Chronic'].sum().reset_index()
chronic
```


```python
delano = chronic[(chronic['School'] == 1) | (chronic['School'] == 2) | (chronic['School'] == 4) ]
delano
```


```python
column_sums = delano.sum(axis=0)
delano = pd.DataFrame(column_sums).transpose()
delano
```


```python
delano['School'] = delano['School'].replace(7.0, 'Delano')
delano
```


```python
lh = chronic[(chronic['School'] == 6) | (chronic['School'] == 7) | (chronic['School'] == 8) ]
lh
```


```python
column_sums = lh.sum(axis=0)
lh = pd.DataFrame(column_sums).transpose()
lh
```


```python
lh['School'] = lh['School'].replace(21.0, 'Lost Hills')
lh
```


```python
chronic = pd.concat([chronic, delano, lh])
chronic
```


```python
replacement_dict = {1.0: 'Delano HS', 2: 'Delano MS', 4.0:'Delano ES', 6.0:'Lost Hills ES', 
                   7.0:'Lost Hills MS', 8.0: 'Lost Hills HS'}
chronic['School'].replace(replacement_dict, inplace=True)
chronic
```


```python
chronic['Chronic Rate'] = chronic['Chronic']/chronic['Enrollment']
chronic
```


```python
chronic = chronic[['School', 'Chronic Rate']]
chronic
```


```python
final = pd.merge(school, chronic, how='inner', on='School')
final
```


```python
# Write dataframe to file

writer = pd.ExcelWriter(output)

tardies.to_excel(writer, sheet_name = 'Attendance', index=False)
final.to_excel(writer, sheet_name='Schools', index=False)

writer.save()
```


```python

```
