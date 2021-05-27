# Day 11 - More Visualization and Coaching

## 26th May, 2021


*Protocol by Damian*

---

## 1. Visualisation Exercise

> Usual MO

- set up your virtual environment and update the components


- for the visualizations we will be working with plotting modules
  * matplotlib
  * seaborn
  * plotly

- So basically import it all...

```Bash
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
import plotly.express as px 
```

- ... and read the dataframe provided

```Bash
# Import dataframe
df = pd.read_csv('kaggle_survey_dataset_small.csv')
```
>This is where the fun starts!


- If we print the first rows of the dataframe using the .head() method we end up with this:

![The Raw Dataframe](https://raw.githubusercontent.com/N4v1ds0n/Protocol_pics/master/pics/Dataframe%201.png)

- The Dataset looks just like the others we have used before, right?

> Wrong!![](https://media1.giphy.com/media/hKO8eHNi2OwX3PBUm6/giphy.gif?cid=790b76119d52ff0deb94de67914266fa76d702b899bfea78&rid=giphy.gif&ct=g)Wrong!


> Things different about this data
- the first row consists of strings of the questions for each column
- none of the columns contains actual simple integer values

![The Dataframe legend](https://raw.githubusercontent.com/N4v1ds0n/Protocol_pics/master/pics/Dataframe%202.png)

> How do we work with this dataframe?

- well first we have to change a few things to make it more accessible for our needs
  - so we change the column names to actually represent what each column contains
  - then we get rid of that nasty row full of questions
  - and finally we opted to remove the spaces to replace them with underscores (good practice I've been told) 

```Bash
df.rename(columns={'Q1': 'age', 'Q2': 'gender', 'Q3': 'country', 'Q4':'education', 'Q5':'profession', 'Q6':'coding_exp_years', 'Q8':'prog_lang_sugg', 'Q11':'platform', 'Q13':'TPU_usage', 'Q15':'machine_learning_exp_yrs', 'Q20':'company_size', 'Q21':'#_respons/workload', 'Q22':'mach_learn_Y/N', 'Q24':'compensation', 'Q25':'mach_learn_spent', 'Q30':'big_data_products', 'Q32':'business_intelligence_tools', 'Q38':'primary_analysis_tool', }, inplace=True)
df2 = df.copy()

df2.replace(' ', '_', regex = True, inplace = True)
df2.drop(axis =0, index = 0, inplace= True)

df2.head(10)
```


> Now we end up with this table

![Dataframe after doctoring](https://raw.githubusercontent.com/N4v1ds0n/Protocol_pics/master/pics/Dataframe%204.png)

> This looks better right?

![](https://i.imgur.com/U3Z43nM.gif)
- Now for the first task: Your stakeholder wants to have a visual comparison between the yearly compensation of people who work as Data Scientists, Data Analysts, and Data Engineers.

```Bash
#relevant columns: Q5 profession
#next relevant column: Q24 yearly compensation in USD
df['profession'].unique()
df['salary'].unique()
````
- now to get a new set

```Bash
# create a count based on col 5 (job) and col 24 (income cat)
# .size() creates count of a grouping
# to_frame("Count") creates a new column with stated title
# .reset_index() ensures that the "Count" heading is on the same hierarchical level as the rest of the headings
groupby_job_salary = df[["Q5", "Q24"]].groupby(["Q5","Q24"]).size().to_frame("Count").reset_index()
groupby_job_salary
```
![New set](https://raw.githubusercontent.com/N4v1ds0n/Protocol_pics/master/pics/Dataframe%20%20Plot%201.png)

- suprisingly the column with the salary is messed up, so we need to address that

```Bash
#goes into salary column and removes the "$" symbol (replaces with nothing)
groupby_job_salary["Q24"] = groupby_job_salary["Q24"].str.replace("$", "")
#goes into salary column and removes the ">" symbol, "," and " " (replaces with nothing)
#Outputs the tidied string into a new column called "Salary" (NB: still a string)
# ">| |," is a regex expression
groupby_job_salary["Salary"] = groupby_job_salary["Q24"].str.replace(">| |,", "")
groupby_job_salary.head()
'''

![New Set b](https://raw.githubusercontent.com/N4v1ds0n/Protocol_pics/master/pics/Dataframe%20plot%201b.png)

- more adjustment 
```Bash
#function to extract the first part of the salary range and convert it to an integer

def get_first_number(x):
    dash_index = x.find("-")
    n = int(x[0 : dash_index])
    return n
```
> Many errors later...

![Frustating](https://i.imgur.com/sUnfC5q.gif)

> We can finally plot

```Bash
#visualisation, graph 1

#seaborn #barplot

#overall appearance
sns.set_theme(style="whitegrid")
sns.set_style("ticks", {"xtick.major.side": 8, "ytick.major.size": 8})

#figure size
plt.figure(figsize=(20,16))

#bar chart; count vs salary category (numberical salary category used, so sorted automatically)
#subgrouping of bars determined by role (hue = "Q5")
#capsize = error bar caps
#palette = overall colouration https://seaborn.pydata.org/tutorial/color_palettes.html
#.isin() filters for desired roles

compensation_by_role = sns.barplot(y = "Count", x = "Salary_num",  hue = "Q5", errwidth = 0.6, errcolor = "black", capsize = 0.1, palette="colorblind", data=groupby_job_salary[groupby_job_salary["Q5"].isin(["Data Analyst", "Data Engineer", "Data Scientist"])])

#title, axes and legend
#NB: the axis labelling needs to come after setting the data source for the axes--otherwise, the data label gets used (e.g. "Salary_num" instead of "Annual Salary (US$)" for x axis)
plt.title("Annual Compensation of Data Professionals", size = 36)
plt.ylabel("Count per salary category", size = 18)
plt.xlabel("Annual Salary (US$)", size = 18)
plt.xticks(rotation = 45, size = 14)
plt.yticks(size = 14)
plt.legend(loc = "upper right", title = "Role", frameon = False, labelspacing = 1.5)
plt.setp(compensation_by_role.get_legend().get_title(), fontsize='24') # set text size of legend title
plt.setp(compensation_by_role.get_legend().get_texts(), fontsize='18') #  set text size of legend contents
````

![Plot 1](https://raw.githubusercontent.com/N4v1ds0n/Protocol_pics/master/pics/Plot%201.png)



>Thankfully Christina provided me with their solutions we also have a sketch for the second task:

![](https://i.imgur.com/lNUh0uV.gif)


```Bash
#Q2 is gender
#Q3 is countries
df.head()
df["Q3"].count()

#sort data by country, descending
#top_10_countries = df.sort_values(by='Q3',ascending=False).iloc[:10,:]
#top_10_countries.head()
#iloc to get top 10 countries

#create a dataframe with country and gender--NOT NEEDED RIGHT?
groupby_country_gender = df[["Q3", "Q2"]].groupby(["Q3", "Q2"]).size().to_frame("Gender_count").reset_index()
#groupby_country_gender_sorted = groupby_country_gender.sort_values(by="Count", ascending=False)
groupby_country_gender.head()


#create dataframe with count by country
#groupby_country = df.groupby("Q3").size().to_frame("Count").reset_index()
#groupby_country.head(20)
```

![Data Plot 2a](https://raw.githubusercontent.com/N4v1ds0n/Protocol_pics/master/pics/Data%20Plot%202%20a.png)

```Bash
df_test = df

df_test['count'] = df_test.groupby('Q3')['Q3'].transform('count')


#top_10_countries = df_test.sort_values(by='count',ascending=False)
#top_10_countries[top_10_countries["Q3"] == "Germany"] 

#Germany is the bottom range for the top 10 countries--figure out the count (404)

#list of top 10 countries: 'India', 'United States of America', 'Other', 'Brazil', 'Japan',
       #'Russia', 'United Kingdom of Great Britain and Northern Ireland',
       #'Nigeria', 'China', 'Germany'
```

```Bash
df_countries_filtered = df_test[df_test['count'] > 403] #filter for top ten countries

#we make a new dataframe based on this filter
#then perform a groupby on this new dataframe, based on gender and put in counts--graph this

groupby_country_gender = df_countries_filtered[["Q3", "Q2"]].groupby(["Q3", "Q2"]).size().to_frame("Gender_count").reset_index()
groupby_country_gender
```

![Data Plot 2b](https://raw.githubusercontent.com/N4v1ds0n/Protocol_pics/master/pics/Data%20Plot%202b.png)

```Bash
#create missing rows
df_to_inc = pd.DataFrame([["Brazil", "Prefer to self-describe", 0],
                         ["Japan", "Nonbinary", 0],
                         ["Japan", "Prefer to self-describe", 0],
                         ["Nigeria", "Prefer to self-describe", 0]],
                         columns = ["Q3", "Q2", "Gender_count"])
```

```Bash
#insert missing rows
groupby_country_gender_full = pd.concat([groupby_country_gender, df_to_inc], ignore_index = True)
groupby_country_gender_full = groupby_country_gender_full.sort_values(["Q3", "Q2"])
groupby_country_gender_full
```

![Data Plot 2c](https://raw.githubusercontent.com/N4v1ds0n/Protocol_pics/master/pics/Data%20plot%202c.png)

> And a first plot:
```Bash
sns.set_theme(style="whitegrid")
plt.figure(figsize=(20,16))
ax = sns.barplot(x="count", y="Q3", hue="Q2", data=df_test[df_test['count'] > 403])
```

![Plot 2](https://raw.githubusercontent.com/N4v1ds0n/Protocol_pics/master/pics/Plot%202.png)

 > And the top 10

 ```Bash
top_10 = sorted(['India', 'United States of America', 'Other', 'Brazil', 'Japan',
       'Russia', 'United Kingdom of Great Britain and Northern Ireland',
       'Nigeria', 'China', 'Germany'])

#make dataframes filtered by gender
man = groupby_country_gender_full.query("Q2 == 'Man'")
nonbinary = groupby_country_gender_full.query("Q2 == 'Nonbinary'")
prefer_not = groupby_country_gender_full.query("Q2 == 'Prefer not to say'")
prefer_self_desc = groupby_country_gender_full.query("Q2 == 'Prefer to self-describe'")
woman = groupby_country_gender_full.query("Q2 == 'Woman'")

fig = go.Figure(data=[
    go.Bar(name='Man', x=top_10, y=man["Gender_count"]),
    go.Bar(name='Nonbinary', x=top_10, y=nonbinary["Gender_count"]),
    go.Bar(name='Prefer not to say', x=top_10, y=prefer_not["Gender_count"]),
    go.Bar(name='Prefer to self-describe', x=top_10, y=prefer_self_desc["Gender_count"]),
    go.Bar(name='Woman', x=top_10, y=woman["Gender_count"]),
])
# Change the bar mode
fig.update_layout(barmode='group')
fig.show()
```

![Data Plot 2b](https://raw.githubusercontent.com/N4v1ds0n/Protocol_pics/master/pics/Plot%202b.png)

>So this is it so far! (to be continued...)

![](https://i.imgur.com/70IqCOb.gif)






## 2. Application Coaching

>What to put in your CV (and what not)

- some general tips
  - personalize your CV, make it about you
  - good overview and consistent structure
  - clear and attractive design, maybe buy a template from a professional
  - drop some relevant keywords in
  - max. 2 pages
  - anti chronological
  - always use PDF as file format
  - use a clear filename e.g.: jane_doe_CV_2012_05_26

- adjust the language to the ad, location and the addressant

- Push the Bootcamp in the center of attention and mark it as Trainee experience
  - fulltime
  - intensive
  - 540 practical hrs
  - mention tools used (Python, SQL, AWS, EDA, machine learning, etc...)

- mention final project

- Pro moves:
  - Use colour palette of prospective employer

- individual training with coaches hand-in deadline for CV 25.06.21!
  -



>How to write a cover letter

- checklist
  - be sure to use the correct spelling for the company name
  - meet the requirements of the ad and address them
  - highlight relevant keyboards
  - make it personal: put in a few hobbies (not the risky ones)
  - don't use standard phrasing except starting date and salary expectations
  - mirror the style of the ad
  - use positive and active wording
  - don't just repeat your CV info or list soft skills 

>You can do this!

![Frustating](https://i.imgur.com/rvKFwxD.gif)

> Useful tips for your job search

- list of links provided
  - https://stackoverflow.com
  - https://honeypot.io
  - https://jobboerse.arbeitsagentur.de
  - https://stepstone
  - https://remoteok.io
  - https://4scotty.com
  - https://startups.com


>The Job interview

- a few tips
  - try to read the interview room and use the opportunity to check the company climate
  - be authentic and if you're nervous try to be open about
  
- Interview preparation
  - save the job ads as PDFs for your reference
  - gather info in advance using glassdor etc.
  - know your facts, location, products, no of employees...
  - not the name of your contact person
    - stalk them on networks like linkedin to get more info

- Scenes of a job interview
  1. the intro
    - opportunity for small talk and first impressions
  2. questions
    - vita
    - motivation
    - technical
  3. conditions
  4. questions from applicant
    - your time to inquire. Use it!
  5. possible task or coding challenge
  6. discussing the next steps
    - it should be 60% you speaking

- be prepared for some uncomfortable questions
  - be honest, but not too honest
  - be self reflective


  >Salary - know your worth

 ![](https://i.pinimg.com/originals/e3/1b/00/e31b0012732d48a24d7e013fb8d819fc.gif)

- Use Glassdoor or stepstone Gehaltsreport for your estimation
- depending on size of the company, branch, location, working exp, education

- a realistic estimation for the beginning would be between 50k and 60k
- don't be too modest
- state your target salary + 5-10%

- negotiation - how to
  - name it
  - stay cool
  - don't explain yourself
  - try to negotiate other benefits like holidays, home office bonus...
  - schedule either a raise or a new negotiation in 6 months time


>That was helpful!

![](https://i.imgur.com/OAf4tXb.gif)





