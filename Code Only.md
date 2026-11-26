
# Personal Sleep Statistics (Exam for PSY-3035)
## Marie Bøe Aaberg
This document is for the code associated with the exam. For the full exam with text please refer to the document "Exam". For text and figures please refer to the document "Exam NO Code". For the dataset please refer to the document "sleep_avg".

Explanations of the code is provided on the same line as said code. Most code will only be commented on the first time it appears to reduce clutter. #===#X Is utilized as to make it easier to find sections within the code. A keyword is added at the end(x) for further convenience.

AI has throughout this exam been used to code time conversion. Although some other sources were available for this code (pandas.Timedelta, 2026; Time deltas, 2026), I found them difficult to understand and make use of. I have therefore decided to leave in the AI code while providing the original source material I asked it to base it on.

---

## Dataset

``` 
import pandas as pd
import matplotlib.pyplot as plt
import matplotlib.ticker as ticker
import numpy as np
from scipy import stats
from matplotlib.ticker import PercentFormatter
import matplotlib.ticker as mtick
from matplotlib.ticker import FuncFormatter
from sklearn.linear_model import LinearRegression
```
```
filename = "sleep_avg.xlsx"
data = pd.read_excel(filename)

print(data)
```
This dataset is made using weekly averages. It includes information on:

- The week
- Time of getting into bed but not deciding to go to sleep (S1, referred to as "prior to bedtime")
Time of deciding when to go to sleep (S2, referred to as "bedtime")
- Sleep onset time (S3)
- Wake-up time (M1)
- Time of when I left the bed (M2, referred to as "get-up time")
- Total time in bed excluding S1(ST = M2 - S2)
- Time in bed spent awake (VT = (S3-S2) + Sum of times waking up during the night + (M2-M1))
- Time in bed spent asleep (sleep = ST - VT)
- The effiency of said sleep in percentages (coded as SE, SE = sleep / ST)
- Sleep quality and day feeling on a 1-5 scale (very bad-very good, coded as SQ and DF)

ST, VT, and sleep are recorded in both an hour and minute format and a minute format for personal convenience, but only the minute format (x_duration_minutes) is utilized in this exam.

---

##  1. Time Spent In Bed
### 1.1 Overall Time Spent in Bed

```
#==========================================================================================================================================================================================================================================================#Figure
#Figure 1.1
#==========================================================================================================================================================================================================================================================#Calculations
#Calculations
##Converts clock times to minutes after midnight. AI has been used for this (based on: Hunter et al., 2026b; Hunter et al., 2026d; pandas.Timedelta, 2026; Time deltas, 2026).
def to_minutes(column):
    return pd.to_timedelta(
        data[column].astype(str)
    ).dt.total_seconds() / 60

##To get the program to recognize the data and converts the 24-hour format to minutes in order to calculate awake times.
S1=to_minutes("S1")
S2=to_minutes("S2")
S3=to_minutes("S3")
M1=to_minutes("M1")
M2=to_minutes("M2")

data["awake_pre_bedtime"]=(S2-S1) % 1440                #Calculates awake time in the evening before trying to sleep from the 24-hour clock. Bedtime(S2) - Pre-Bedtime(S1).  %1440 is an additional step to account for midnight changing the values from 23:xx to 00:xx
data["awake_bedtime"]=(S3-S2) % 1440                    #Calculates awake time in the evening when trying to sleep from the 24-hour clock. Sleep onset time(S3) - Bedtime(S2).
data["awake_morning"]=(M2-M1)                           #Calculates awake time in the morning from the 24-hour clock. Get-up time(M2) - Wake-up time(M1).

##Calculates the average durations across weeks so the pie chart gets a single value to deal with
sleep=data["sleep_duration_minutes"].mean()
pre_bedtime=data["awake_pre_bedtime"].mean()
bedtime=data["awake_bedtime"].mean()
morning=data["awake_morning"].mean()

#To calculate the average total time spent in bed for plt.text() using hours and minutes
time_in_bed = sleep + pre_bedtime + bedtime + morning
hours = int(time_in_bed // 60)
minutes = int(time_in_bed % 60)

#To display the amount of time spent in the sleep slice #AI has been used (based on: Hunter et al., 2026b; Hunter et al., 2026d; pandas.Timedelta, 2026; Time deltas, 2026).
def format_time_ST(value):
    hours = int(value // 60)
    minutes = int(value % 60)
    return f"{hours} h {minutes} min"

#To display the amount of time spent in each awake slice #AI has been used (based on: Hunter et al., 2026b; Hunter et al., 2026d; pandas.Timedelta, 2026; Time deltas, 2026).
def format_time_awake(value):
    minutes = int(value % 60)
    return f"{minutes} min"

#For the Pie Char and Visuals
values=[sleep, pre_bedtime, bedtime, morning]               #Sets the values. Could also be done in plt.pie() but with all the necessary details for the chart this is a clearer and easier to understand structure (Simply Coding, 2020)
labels=[                                                    #Names the labels in the same order as values(sleep, then evening, then morning)
    f"Sleeping ({format_time_ST(sleep)})",                  #AI has been used for the time formatting only (based on: Hunter et al., 2026b; Hunter et al., 2026d; pandas.Timedelta, 2026; Time deltas, 2026)
    f"Prior to Deciding to Sleep ({format_time_awake(pre_bedtime)})",
    f"Trying to Sleep ({format_time_awake(bedtime)})",
    f"Awake in the Morning ({format_time_awake(morning)})"
]

#==========================================================================================================================================================================================================================================================#Display and Visuals
#Display and Visuals
plt.pie(                                                    #(Simply Coding, 2020)
    values,                                                 #Values because it is defined above
    labels=labels,                                          #Labels that were decided upon above (Hunter et al., 2026c)
    autopct="%2.1f%%",                                      #Percentages
    colors=["indigo", "sienna", "red", "gold"]              #Colors. Follows the same order as values(sleep, then evening, then morning) (Hunter et al., 2026a; Phillips, 2022).
    )   

plt.title(                                                  #Title
    "Time Spent in Bed (Fig. 1.1)", 
    fontsize=20, 
    fontweight="bold",
    loc="right"
    )    
plt.text(                                                   #Adds average time spent in bed text at the bottom (Kimberly Fessel, 2021).
    -0.3, -0.05,                                            #So that it has a grid to place the text by the bottom (Hunter et al., 2026e; Kimberly Fessel, 2021)
    f"Average Time Spent in Bed:\n{hours} h {minutes} min", #f before quotation marks lets you insert python variables directly into text. Hours/minutes transforms the minutes into text. /n is for a new line.
    transform=plt.gca().transAxes,                          #So that it has a grid to place the text
    fontsize=10,                                            #Fontsize
    verticalalignment="bottom",                             #Places text at bottom
    bbox=dict(                                              #Adds a box outlining the text
    facecolor="none",                                       #No background color
        edgecolor="black",                                  #Border color
    )
)

plt.show()                                                  #Displays the piechart
```
### 1.2 Longitudinal Data for Time Spent in Bed
```
#==========================================================================================================================================================================================================================================================#Figure
#Figure 1.2
#==========================================================================================================================================================================================================================================================#General
#General
fig, (ax1,ax2)=plt.subplots(2,1, figsize=(12, 10))
#The code above is to get the Evening Habits and Morning Habits in the same figure, but as two seperate plots and with enough size for the legends to not overlap with the graphs (Matplotlib Subplots, 2025)
#Almost all plt code turns into ax1/ax2 code.

week = data["week"]           #To get the program to recognize the data for the x-axis.

## To convert time into hours and fractions of hours
def convert_time(column):
    time = pd.to_datetime(data[column].astype(str))
    time = time.dt.hour + time.dt.minute / 60
    return time.where(time >= 12, time + 24)

S1 = convert_time("S1")
S2 = convert_time("S2")
S3 = convert_time("S3")
M1 = convert_time("M1")
M2 = convert_time("M2")

##To format time
def format_time(x, pos):
    hour = int(x) % 24
    minutes = int(round((x - int(x)) * 60))
    return f"{hour:02d}:{minutes:02d}"

ax1.yaxis.set_major_formatter(ticker.FuncFormatter(format_time))
ax1.yaxis.set_major_locator(ticker.MultipleLocator(0.5))
ax2.yaxis.set_major_formatter(ticker.FuncFormatter(format_time))
ax2.yaxis.set_major_locator(ticker.MultipleLocator(0.5))

#The code above from after defining week up until this point has derived from AI. Finding information on how to do this through traditional methods proved too difficult. The hour + minute columns in the datasets are not registered in the necessary form for python to properly read it.
#This is in order to make the code have a 24 hour clock format and so that the graph goes from 23:59 to 00:00 without jumping from the top of the graph to the bottom and thus include all 24 hours (based on: Hunter et al., 2026b; Hunter et al., 2026d; pandas.Timedelta, 2026; Time deltas, 2026).

##Averages
avg_S1 = sum(S1) / len(S1)         #To get an average for Pre-Bedtime to make the graph more managable once the dataset grows.
avg_S2 = sum(S2) / len(S2)         #Average for Bedtime
avg_S3 = sum(S3) / len(S3)         #Average for Sleep Onset Time
avg_M1 = sum(M1) / len(M1)         #Average for Wake-Up Time
avg_M2 = sum(M2) / len(M2)         #Average for Get-Up Time

#==========================================================================================================================================================================================================================================================#Evening Subplot
#Evening Habits Figure 1.2.a
ax1.plot(                          #To get the plotted line for Pre-Bedtime (Matplotlib Subplots, 2025; Visually Explained)
     week,                         #x-axis
     S1,                           #y-axis
     marker="o",                   #To make each datapoint more visible
     label="Prior to Bedtime",     #For plt.legend() to have the name of the line for Pre-Bedtime in the plot (Hunter et al., 2026c)
     color="peru",                 #Color (Hunter et al., 2026a; Phillips, 2022).
     linewidth=4                   #Width of the line
)

ax1.plot(                          #Same as above but for Bedtime (Matplotlib Subplots, 2025; Visually Explained)
     week,                        
     S2,                          
     marker="o",                  
     label="Bedtime",              #(Hunter et al., 2026c)
     color="orangered",            #(Hunter et al., 2026a; Phillips, 2022)
     linewidth=4                       
)

ax1.plot(                          #Same as above but for Sleep Onset Time (Matplotlib Subplots, 2025; Visually Explained)
     week,
     S3,
     marker="o",
     label="Sleep Onset Time",     #(Hunter et al., 2026c)
     color="mediumblue",           #(Hunter et al., 2026a; Phillips, 2022)
     linewidth=4
)

##Visuals
ax1.set_title("Average Weekly Evening Habits (a)", fontsize=20)                       #Title label with an appropriate fontsize")
ax1.set_xlabel("Week", fontsize=14)                                                            #Label on x-axis
ax1.set_ylabel("Hour", fontsize=14)                                                            #Label on y-axis
ax1.axhline(avg_S1, linestyle="--", color="peru", label="Average Time Prior to Bedtime")       #The average for Pre-Bedtime with label for plt.legend() and appropriate styling (Hunter et al., 2026a; Hunter et al., 2026c; Phillips, 2022).
ax1.axhline(avg_S2, linestyle="--", color="orangered", label="Average Bedtime")                #The average for Bedtime with label for plt.legend() and appropriate styling (Hunter et al., 2026a; Hunter et al., 2026c; Phillips, 2022).
ax1.axhline(avg_S3, linestyle="--", color="mediumblue", label="Average Sleep Onset Time")      #The average for Sleep Onset Time with label for plt.legend() and appropriate styling (Hunter et al., 2026a; Hunter et al., 2026c; Phillips, 2022).
ax1.grid(True, linestyle="--", alpha=0.5)                                                      #To get an appropriately stylized grid along x- and y-axis so it is easier to see the where the values are. Alpha makes it more transparent and thus less visible (Grids in Matplotlib, 2026)

ax1.legend(                                                                                    #To display names for datalines (Hunter et al., 2026c)
    loc="upper center",                                                                        #To decide on location
    bbox_to_anchor=(0.5, -0.15),                                                               #Anchors the box in place relative to graph so it doesn't interfere with the lines
    ncol=2,                                                                                    #Makes the legend two columns to save space
    fontsize=12                                                                                #Fontsize
)

#==========================================================================================================================================================================================================================================================#Morning Subplot
#Morning Habits, same code and justification as above. Figure 1.2.b
ax2.plot(                                                                                      #Wake-Up Time(Matplotlib Subplots, 2025; Visually Explained)
     week,
     M1,
     marker="o",
     label="Wake-Up Time",                                                                     #(Hunter et al., 2026c)
     color="yellow",                                                                           #(Hunter et al., 2026a; Phillips, 2022)
     linewidth=4
)

ax2.plot(                                                                                      #Get-Up Time(Visually Explained, 2025)
     week,
     M2,
     marker="o",
     label="Get-Up Time",                                                                      #(Hunter et al., 2026c)
     color="hotpink",                                                                          #(Hunter et al., 2026a; Phillips, 2022)
     linewidth=4
)

##Visuals
ax2.set_title("Average Weekly Morning Habits (b)", fontsize=20)
ax2.set_xlabel("Week", fontsize=14)
ax2.set_ylabel("Hour", fontsize=14)
ax2.axhline(avg_M1, linestyle="--", color="yellow", label="Average Wake-Up Time")              #(Hunter et al., 2026a; Hunter et al., 2026c; Phillips, 2022)
ax2.axhline(avg_M2, linestyle="--", color="hotpink", label="Average Get-Up Time")              #(Hunter et al., 2026a; Hunter et al., 2026c; Phillips, 2022)
ax2.grid(True, linestyle="--", alpha=0.5)                                                      #(Grids in Matplotlib, 2026)

ax2.legend(                                                                                    #(Hunter et al., 2026c)
    loc="upper center",                                                                        
    bbox_to_anchor=(0.5, -0.15),                                                               
    ncol=2,                                                                                    
    fontsize=12                                                                                
)

#==========================================================================================================================================================================================================================================================#Display
#Display and Visuals
fig.suptitle(                                                                                       #(Kumar, 2026a)
    "Longitudinal Data for Time Spent in Bed (Fig. 1.2)",
    fontsize=28,
    fontweight="bold",
    y=0.99
)

plt.tight_layout()                                                                             #To prevent overlapping information
plt.show()
```
---

## 2. Sleep Characteristics and Well-Being
### 2.1 Longitudinal Data for Sleep Characteristics and Well-Being

```
#==========================================================================================================================================================================================================================================================#Figure
#Figure 2.1
#==========================================================================================================================================================================================================================================================#General
#General
fig, ((ax1,ax2),(ax3,ax4))=plt.subplots(2,2, figsize=(12, 10))
##The code above is to get the various plots in the same figure, but as seperate plots and with enough size for the legends to not overlap with the graphs (Matplotlib Subplots, 2025)
##Almost all plt code turns into ax code with this.

##Variables
week=data["week"]                                         
SQ=data["sleep_quality"]                                  
DF=data["day_feeling"]
SE=data["sleep_efficiency"]                                     

##Averages
avg_SQ = sum(SQ) / len(SQ)                                #To get the average sleep quality, useful for when dataset has grown larger
avg_DF = sum(DF) / len(DF)                                #To get the average daytime feeling, useful for when dataset has grown larger
avg_SE = sum(SE) / len(SE)                                #To get the average sleep efficiency, useful for when dataset has grown larger

##For Density Plots                                       #(Density Plots with Pandas in Python, 2025)
d_SE=pd.DataFrame({"Sleep Efficiency": SE})
d_SQDF=pd.DataFrame({
    "Sleep Quality": SQ,
    "Daytime Feeling": DF
})

#==========================================================================================================================================================================================================================================================#Subplot Weekly SE
#Figure 2.1.a
ax1.plot(                                                                                               #To get the subplot for weekly sleep efficiency(SE) (Visually Explained, 2025)                                                    #x-axis
week,                                                     
SE,                                                       
marker="o",                                               
label= "Sleep Efficiency",                                                                              #(Hunter et al., 2026c)
color="black",                                                                                          #(Hunter et al., 2026a; Phillips, 2022)
linewidth=2                                               
)

##Visuals
ax1.yaxis.set_major_formatter(PercentFormatter(xmax=1, decimals=0))                                     #To get the x-axis to show percentages as f.eks. 80% instead of 0.800 (Matplotlib.ticker.PercentFormatter class in Python, 2025)
ax1.set_title("Weekly Trends in Sleep Efficiency (a)", fontsize=20)                                         
ax1.set_xlabel("Week", fontsize=14)                                                                     
ax1.set_ylabel("Sleep Efficiency", fontsize=14)                                                         
ax1.axhline(avg_SE, linestyle="--", color="black", label="Average Sleep Efficiency")                    #(Hunter et al., 2026a; Hunter et al., 2026c; Phillips, 2022).

#==========================================================================================================================================================================================================================================================#Subplot SE Density
#Figure 2.1.b
d_SE.plot.density(                                                                                      #(Density Plots with Pandas in Python, 2025)
    bw_method='silverman', 
    color="black",
    linewidth=2,
    ax=ax2,
    legend=False
    )

##Visuals
ax2.xaxis.set_major_formatter(PercentFormatter(xmax=1, decimals=0))                                     #(Matplotlib.ticker.PercentFormatter class in Python, 2025)
ax2.xaxis.set_major_locator(mtick.MultipleLocator(0.05))                                                #To have percentages shown per 5% (Matplotlib.ticker.PercentFormatter class in Python, 2025)

ax2.set_title("Sleep Efficiency Density (b)", fontsize=20)                                                              
ax2.set_xlabel("Sleep Efficiency", fontsize=14)                                                                     
ax2.set_ylabel("Probability Density", fontsize=14)                                                      #Generalised to fit both SQ and DF

#==========================================================================================================================================================================================================================================================#Subplot Weekly SQ and DF
#Figure 2.1.c
ax3.plot(                                                                                               #For Sleep Quality(Visually Explained, 2025)
    week,
    SQ,                                                      
    marker="o",                                              
    label= "Sleep Quality",                                                                             #(Hunter et al., 2026c)
    color="purple",                                                                               #(Hunter et al., 2026a; Phillips, 2022)
    linewidth=2                                              
)

ax3.axhline(avg_SQ, linestyle="--", color="purple", label="Average Sleep Quality")                #(Hunter et al., 2026a; Hunter et al., 2026c; Phillips, 2022)

ax3.plot(                                                                                               #Same as above but for Daytime Feeling (Visually Explained, 2025)
    week,
    DF,
    marker="o",
    label="Daytime Feeling",                                                                            #(Hunter et al., 2026c)
    color="green",                                                                                 #(Hunter et al., 2026a; Phillips, 2022)
    linewidth=2
)

ax3.axhline(avg_DF, linestyle="--", color="green", label="Average Daytime Feeling")                #(Hunter et al., 2026a; Hunter et al., 2026c; Phillips, 2022)

##Visuals
ax3.set_title("Weekly Trends in SQ and DF (c)", fontsize=20)                                            
ax3.set_xlabel("Week", fontsize=14)                                                                     
ax3.set_ylabel("Feeling", fontsize=14)                                                                  #Generalised to fit both SQ and DF
ax3.set_ylim(1, 5)                                                                                      #To get the graph to show 1-5 instead of 2-3.75 where my values lie (Matplotlib.pyplot.yticks(), 2025)
ax3.set_yticks([1, 2, 2.5, 3, 3.5, 4, 5],                                                               #To not get .5 or .75 values, but then added 2.5 and 3.5 since those are two common values for my graph to be within (Matplotlib.pyplot.yticks(), 2025)
               ['Very Bad 1', '2', '2.5', 'Neither 3', '3.5', '4', 'Very Good 5'])                      #To add explanatory text (very good/bad) for convenience

#==========================================================================================================================================================================================================================================================#Subplot SQ and DF Density
#Figure 2.1.d
d_SQDF.plot.density(                                                                                    #(Density Plots with Pandas in Python, 2025)
    bw_method='silverman', 
    color=["purple", "green"],                                                               #(Hunter et al., 2026a; Phillips, 2022)
    linewidth=2,
    ax=ax4,
    legend=False
    )

##Visuals
ax4.set_xlim(1, 5)                                                                                      #(Matplotlib.pyplot.yticks(), 2025)
ax4.set_xticks([1, 2, 3, 4, 5],
               ['1\nVery Bad', '2', '3\nNeither Good nor Bad', '4', '5\nVery Good'])                                        
ax4.set_title("SQ and DF Density (d)", fontsize=20)                             
ax4.set_xlabel("Feeling", fontsize=14)                                          
ax4.set_ylabel("Probability Density", fontsize=14)                              

#==========================================================================================================================================================================================================================================================#Display and Visuals
#Display and visuals

for ax in [ax1, ax2, ax3, ax4]:                                                                     #Grid to shorten coding lines (Grids in Matplotlib, 2026)
    ax.grid(True, linestyle="--", alpha=0.5)

##To make the legends shared and outside of the subplots (Hunter et al., 2026c)
handles, labels = [], []

for ax in [ax1, ax2, ax3, ax4]:
    h, l = ax.get_legend_handles_labels()
    handles.extend(h)
    labels.extend(l)

unique_handles = dict(zip(labels, handles))

fig.legend(
    unique_handles.values(),
    unique_handles.keys(),
    loc="upper center",
    bbox_to_anchor=(0.5, 0.02),
    ncol=3,
    fontsize=11
)

##Main Title
fig.suptitle(                                                                                       #(Kumar, 2026a)
    "Longitudinal Data and Density for Sleep Efficiency, \nSleep Quality, and Daytime Feeling (Fig. 2.1)",
    fontsize=28,
    fontweight="bold",
    y=0.99
)

##Display
plt.tight_layout()                                                                                  #To prevent overlap
plt.show()                                                                                          #Displays graph
```

### 2.2 How Sleep Efficiency Influences Sleep Quality

```
#==========================================================================================================================================================================================================================================================#Figure
#Figure 2.2
#==========================================================================================================================================================================================================================================================#General
#General
SQ = data["sleep_quality"]
SE = data["sleep_efficiency"]

#==========================================================================================================================================================================================================================================================#Scatter Plot
#Scatter Plot (Bro Code, 2025; Karina Adcock, 2023)
plt.scatter(
    SE,
    SQ,
    color="purple",                                                                                     #(Hunter et al., 2026a; Phillips, 2022)
    alpha=0.8,
)

##Enumerating so you see the week for each scatter point (Kimberly Fessel, 2021)
for i, week in enumerate(data["week"]):
    plt.annotate(
        week,                                                                                           #Data from dataset for the program to use
        (SE.iloc[i], SQ.iloc[i]),
        ###"SE.iloc" is for the x-axis so essentially "x.iloc". Same for "SQ.iloc" but for the y axis so essentially "y.iloc". i is for coordinates for each datapoint
        fontsize=9,
        xytext=(5, 5),
        textcoords="offset points"
    )

#==========================================================================================================================================================================================================================================================#Regression Line
#Regression Line (Stojiljković, 2024)
model = LinearRegression()
model.fit(SE.values.reshape(-1, 1), SQ)

y_pred = model.predict(SE.values.reshape(-1, 1))

r_sq = model.score(SE.values.reshape(-1, 1), SQ)

x_line = np.linspace(SE.min(), SE.max(), 100).reshape(-1, 1)
y_line = model.predict(x_line)

plt.plot(                                                                                               #To plot in the regression line (Bro Code, 2025; Karina Adcock, 2023)
    x_line,
    y_line,
    color="mediumorchid",                                                                               #(Hunter et al., 2026a; Phillips, 2022)
    linestyle="--",
    linewidth=2,
    label="Regression Line"
)

#==========================================================================================================================================================================================================================================================#Visuals and Display
#Visuals and Display

plt.gca().xaxis.set_major_formatter(PercentFormatter(xmax=1, decimals=0))                               #(Matplotlib.ticker.PercentFormatter class in Python, 2025)

plt.ylim(1, 5)                                                                                          #(Matplotlib.pyplot.yticks(), 2025)
plt.yticks([1, 2, 2.5, 3, 3.5, 4, 5],                                                                   #(Matplotlib.pyplot.yticks(), 2025)
               ['Very Bad 1', '2', '2.5', 'Neither 3', '3.5', '4', 'Very Good 5'])                      

plt.grid(True, linestyle="--", alpha=0.3)                                                               #(Grids in Matplotlib, 2026)

plt.text(                                                                                               #(Hunter et al., 2026e; Kimberly Fessel, 2021)
    0.02, 0.98,
    "Numbers Indicate Week",
    transform=plt.gca().transAxes,
    fontsize=10,
    verticalalignment="top",
    bbox=dict(facecolor="white", alpha=0.8, edgecolor="gray")                                           #(Hunter et al., 2026a; Phillips, 2022)
)

plt.xlabel("Sleep Efficiency")
plt.ylabel("Sleep Quality")
plt.title("Association Between Sleep Efficiency\nand Sleep Quality (Fig. 2.2)", fontweight="bold", fontsize="14")

##Display
plt.legend()                                                                                            #(Hunter et al., 2026c)
plt.show()
```

### 2.3 How Sleep Quality and Efficiency Influence Daytime Feeling
```
#==========================================================================================================================================================================================================================================================#Figure
#Figure 2.3
#==========================================================================================================================================================================================================================================================#General
#General
DF=data["day_feeling"]
SQ=data["sleep_quality"]
SE=data["sleep_efficiency"]
#==========================================================================================================================================================================================================================================================#Scatter Plot
#Bubble plot (Karina Adcock, 2021; Ms. Simmons' Math Class, 2025; Ryan Noonan, 2018)
plt.scatter(
    SE, 
    SQ, 
    s=DF**5*2,                                                                                          #Bubble size where there bubbles visibly grow for each new value (Ryan Noonan, 2018)
    c=DF,                                                                                               #Bubble Color
    edgecolor="darkgreen",                                                                              #(Hunter et al., 2026a; Phillips, 2022)
    alpha=0.8,
    cmap=plt.get_cmap("Greens",5)                                                                       #Color on the colorbar on the side split into only 5 colors (Hunter et al., 2026a; Phillips, 2022)
    ) 

##Enumeration (Kimberly Fessel, 2021)
for i, week in enumerate(data["week"]):                         
    plt.annotate(
        week,                             
        (SE.iloc[i], SQ.iloc[i]),
        fontsize=9,
        xytext=(5, 5),
        textcoords="offset points"
    )

#==========================================================================================================================================================================================================================================================#Visuals
#Visuals
plt.title("How Sleep Quality and Efficiency \nInfluence Daytime Feeling (Fig. 2.3)", fontsize=14, fontweight="bold")
plt.xlabel("Sleep Efficiency")
plt.ylabel("Sleep Quality")

cbar=plt.colorbar(label="Daytime Feeling")                                                              #Creates the colorbar on the side of the figure (Karina Adcock, 2021; Ms. Simmons' Math Class, 2025)
plt.clim(1,5)                                                                                           #Sets the min and max values on the colorbar (Karina Adcock, 2021)
cbar.set_ticks([1,2,3,4,5])                                                                             #Sets the visible values on the colorbar (Karina Adcock, 2021)
cbar.set_ticklabels(["1 Very Bad", "2", "3 Neither", "4", "5 Very Good"])                               #Labels for the different colorbar parameters (Karina Adcock, 2021)

plt.gca().xaxis.set_major_formatter(PercentFormatter(xmax=1, decimals=0))                               #(Matplotlib.ticker.PercentFormatter class in Python, 2025)

plt.ylim(1, 5)                                                                                          #(Matplotlib.pyplot.yticks(), 2025)
plt.yticks([1, 2, 2.5, 3, 3.5, 4, 5],                                                                   #(Matplotlib.pyplot.yticks(), 2025)
               ['Very Bad 1', '2', '2.5', 'Neither 3', '3.5', '4', 'Very Good 5'])                      

plt.grid(True, linestyle="--", alpha=0.3)                                                               #(Grids in Matplotlib, 2026)

plt.text(                                                                                               #(Hunter et al., 2026e; Kimberly Fessel, 2021)
    0.02, 0.98,
    "Numbers Indicate Week",
    transform=plt.gca().transAxes,
    fontsize=10,
    verticalalignment="top",
    bbox=dict(facecolor="white", alpha=0.8, edgecolor="gray")                                           #(Hunter et al., 2026a; Phillips, 2022)
)

#A secondary scatter plot for the plus sign inside the bubbles (Bro Code, 2025; Karina Adcock, 2023)
plt.scatter(
    SE, SQ,
    marker="+",
    color="darkgreen",                                                                                  #(Hunter et al., 2026a; Phillips, 2022)
    s=20,
    linewidths=1,
)

#==========================================================================================================================================================================================================================================================#Display
#Display
plt.show()
```

---

## 3. Sleep Characteristics and Well-Being in Relation to Time Spent in Bed

```
#==========================================================================================================================================================================================================================================================#Figure
#Figure 3
#==========================================================================================================================================================================================================================================================#General
#General for all subplots

##Converts clock times to minutes after midnight. AI has been used for this (based on: Hunter et al., 2026b; Hunter et al., 2026d; pandas.Timedelta, 2026; Time deltas, 2026).
def to_minutes(column):
    return pd.to_timedelta(
        data[column].astype(str)
    ).dt.total_seconds() / 60

##To get the program to recognize the data
M1 = to_minutes("M1")
M2 = to_minutes("M2")
S1 = to_minutes("S1")
S2 = to_minutes("S2")
S3 = to_minutes("S3")

##For x- and y-axes
TIBM = data["time_in_bed_morning"] = (M2-M1) % 1440
TIBP = data["time_in_bed_pre"] = (S2-S1) % 1440
TIBB = data["time_in_bed_bedtime"] = (S3-S2) % 1440
ST = data["ST_duration_minutes"]
DF = data["day_feeling"]
SQ = data["sleep_quality"]
SE = data["sleep_efficiency"]

##For plt.text/ax.text 
data["awake_pre"]=(S2-S1)                               
pre=data["awake_pre"].mean()                        
hours_p = int(pre // 60)                                      #Taken from Pie Chart (based on: Hunter et al., 2026b; Hunter et al., 2026d; pandas.Timedelta, 2026; Time deltas, 2026)
minutes_p = int(pre % 60)                                     #Taken from Pie Chart (based on: Hunter et al., 2026b; Hunter et al., 2026d; pandas.Timedelta, 2026; Time deltas, 2026)

sleeping = ST.mean()                   
hours_s = int(sleeping // 60)                                 #Taken from Pie Chart (based on: Hunter et al., 2026b; Hunter et al., 2026d; pandas.Timedelta, 2026; Time deltas, 2026)
minutes_s = int(sleeping % 60)                                #Taken from Pie Chart (based on: Hunter et al., 2026b; Hunter et al., 2026d; pandas.Timedelta, 2026; Time deltas, 2026)

data["awake_bedtime"]=(S3-S2)% 1440
bedtime=data["awake_bedtime"].mean()
hours_b = int(bedtime // 60)                                  #Taken from Pie Chart (based on: Hunter et al., 2026b; Hunter et al., 2026d; pandas.Timedelta, 2026; Time deltas, 2026)
minutes_b = int(bedtime % 60)                                 #Taken from Pie Chart (based on: Hunter et al., 2026b; Hunter et al., 2026d; pandas.Timedelta, 2026; Time deltas, 2026)

data["awake_morning"]=(M2-M1)                               
morning=data["awake_morning"].mean()                        
hours_m = int(morning // 60)                                  #Taken from Pie Chart (based on: Hunter et al., 2026b; Hunter et al., 2026d; pandas.Timedelta, 2026; Time deltas, 2026)
minutes_m = int(morning % 60)                                 #Taken from Pie Chart (based on: Hunter et al., 2026b; Hunter et al., 2026d; pandas.Timedelta, 2026; Time deltas, 2026)

#==========================================================================================================================================================================================================================================================#Subplots
##Creates the figure with six subplots
fig, ((ax1,ax2,ax3,ax4),(ax5,ax6,ax7,ax8)) = plt.subplots(2, 4, figsize=(38, 18))           #(Matplotlib Subplots, 2025)
fig.tight_layout(                                                                           #Spacing
    rect=[0.07, 0.08, 0.98, 0.88],
    w_pad=6,
    h_pad=5
)

#==========================================================================================================================================================================================================================================================#Subplot 1/Fig. 3.1a
#Figure 3.1a (SQ and DF in relation to TIBP)
##Scatter plots
ax1.scatter(                                                                #Scatter plot for Sleep Quality (SQ) (Bro Code, 2025; Karina Adcock, 2023)
    TIBP, SQ,                   
    color="purple",                                                         #(Hunter et al., 2026a; Phillips, 2022)
    alpha=0.5,                  
    s=100,                      
    label="Sleep Quality",                                                  #(Hunter et al., 2026c)
)

ax1.scatter(                                                                #Scatter points for daytime feeling (Bro Code, 2025; Karina Adcock, 2023)
    TIBP, DF,
    color="green",                                                          #(Hunter et al., 2026a; Phillips, 2022)
    alpha=0.75,
    s=100,
    label="Day Feeling",                                                    #(Hunter et al., 2026c)
)

###For 10 minute intervals. From AI (Based on: Group by: split-apply-combine, 2026; pandas.Series, 2026; Series, 2026)
data["time_interval_p"] = (data["time_in_bed_pre"] // 10) * 10

averages_p = data.groupby("time_interval_p")[
    ["sleep_quality", "day_feeling"]
].mean().sort_index()


##Line Graphs
ax1.plot(                                                                       #For sleep quality (Visually Explained, 2025)
    averages_p.index,
    averages_p["sleep_quality"],
    color="purple",                                                             #(Hunter et al., 2026a; Phillips, 2022)
    linewidth=4,
    marker="o",
    label="Average Sleep Quality",                                              #(Hunter et al., 2026c)
)

ax1.plot(                                                                       #For daytime feeling (Visually Explained, 2025)
    averages_p.index,
    averages_p["day_feeling"],
    color="green",                                                              #(Hunter et al., 2026a; Phillips, 2022)
    linewidth=4,
    marker="o",
    label="Average Day Feeling",                                                #(Hunter et al., 2026c)
)

##Visuals
ax1.set_ylim(1, 5)                                                              #(Matplotlib.pyplot.yticks(), 2025)
ax1.set_yticks([1, 2, 2.5, 3, 3.5, 4, 5],                                       #(Matplotlib.pyplot.yticks(), 2025)
['Very bad 1', '2', '2.5', 'Neither 3', '3.5', '4', 'Very good 5'])
ax1.set_facecolor((0.82, 0.71, 0.55, 0.2))                                      #Subplot background color (Hunter et al., 2026a; Kumar, 2026a; Phillips, 2022)

#==========================================================================================================================================================================================================================================================#Subplot 5/Fig. 3.1b
#Figure 3.1b (SE in relation to TIBP)
##Scatter plot (Bro Code, 2025; Karina Adcock, 2023)
ax5.scatter(                     
    TIBP, SE,                   
    color="black",                                                          #(Hunter et al., 2026a; Phillips, 2022)
    alpha=0.5,                  
    s=100,                      
    label="Sleep Efficiency",                                               #(Hunter et al., 2026c)
)

###For 10 minute intervals. From AI (Based on: Group by: split-apply-combine, 2026; pandas.Series, 2026; Series, 2026)
data["time_interval_p"] = (data["time_in_bed_pre"] // 10) * 10

averages_p = data.groupby("time_interval_p")[
    ["sleep_efficiency"]
].mean().sort_index()

##Line Graph                                                                 #(Visually Explained)
ax5.plot(                            
    averages_p.index,
    averages_p["sleep_efficiency"],
    color="black",                                                           #(Hunter et al., 2026a; Phillips, 2022)
    linewidth=4,
    marker="o",
    label="Average Sleep Efficiency",                                        #(Hunter et al., 2026c)
)

##Visuals
ax5.set_facecolor((0.82, 0.71, 0.55, 0.2))                                   #(Hunter et al., 2026a; Phillips, 2022)

#==========================================================================================================================================================================================================================================================#Subplot 2/Fig. 3.2a
#Figure 3.2a (SQ and DF in relation to TIBB)
##Scatter plot
ax2.scatter(                                                #Scatter plot for Sleep Quality (SQ) (Bro Code, 2025; Karina Adcock, 2023)
    TIBB, SQ,                   
    color="purple",                                         #(Hunter et al., 2026a; Phillips, 2022)
    alpha=0.5,                  
    s=100,                      
    label="Sleep Quality",                                  #(Hunter et al., 2026c)
)

ax2.scatter(                                                #Scatter points for daytime feeling (Bro Code, 2025; Karina Adcock, 2023)
    TIBB, DF,
    color="green",                                          #(Hunter et al., 2026a; Phillips, 2022)
    alpha=0.75,
    s=100,
    label="Day Feeling",                                    #(Hunter et al., 2026c)
)

###For 10 minute intervals. From AI (Based on: Group by: split-apply-combine, 2026; pandas.Series, 2026; Series, 2026)
data["time_interval_b"] = (data["time_in_bed_bedtime"] // 10) * 10

averages_b = data.groupby("time_interval_b")[
    ["sleep_quality", "day_feeling"]
].mean().sort_index()


##Line Graph
ax2.plot(                                                   #For sleep quality (Visually Explained, 2025)
    averages_b.index,
    averages_b["sleep_quality"],
    color="purple",                                         #(Hunter et al., 2026a; Phillips, 2022)
    linewidth=4,
    marker="o",
    label="Average Sleep Quality",                          #(Hunter et al., 2026c)
)

ax2.plot(                                                   #For daytime feeling (Visually Explained, 2025)
    averages_b.index,
    averages_b["day_feeling"],
    color="green",                                          #(Hunter et al., 2026a; Phillips, 2022)
    linewidth=4,
    marker="o",
    label="Average Day Feeling",                            #(Hunter et al., 2026c)
)


##Visuals
ax2.set_ylim(1, 5)                                          #(Matplotlib.pyplot.yticks(), 2025)
ax2.set_yticks([1, 2, 2.5, 3, 3.5, 4, 5],
['1', '2', '2.5', '3', '3.5', '4', '5'])

ax2.set_facecolor((1.0, 0.0, 0.0, 0.06))                    #(Hunter et al., 2026a; Kumar, 2026a; Phillips, 2022)

#==========================================================================================================================================================================================================================================================#Subplot 6/Fig. 3.2b
#Figure 3.2b (SE in relation to TIBB)
##Scatter plot (Bro Code, 2025; Karina Adcock, 2023)
ax6.scatter(                     
    TIBB, SE,                   
    color="black",                          #(Hunter et al., 2026a; Phillips, 2022)
    alpha=0.5,                  
    s=100,                      
    label="Sleep Efficiency",               #(Hunter et al., 2026c)
)

###For 10 minute intervals. From AI (Based on: Group by: split-apply-combine, 2026; pandas.Series, 2026; Series, 2026)
data["time_interval_b"] = (data["time_in_bed_bedtime"] // 10) * 10

averages_b = data.groupby("time_interval_b")[
    ["sleep_efficiency"]
].mean().sort_index()

##Line Graph                                                                #(Visually Explained)
ax6.plot(                            
    averages_b.index,
    averages_b["sleep_efficiency"],
    color="black",                                                          #(Hunter et al., 2026a; Phillips, 2022)
    linewidth=4,
    marker="o",
    label="Average Sleep Efficiency",                                       #(Hunter et al., 2026c)
)

##Visuals
ax6.set_facecolor((1.0, 0.0, 0.0, 0.06))                                    #(Hunter et al., 2026a; Kumar, 2026a; Phillips, 2022)

#==========================================================================================================================================================================================================================================================#Subplot 3/Fig. 3.3a
#Figure 3.3a
##Scatter plots
ax3.scatter(                                                                #Scatter plot for Sleep Quality (SQ) (Bro Code, 2025; Karina Adcock, 2023)
    ST, SQ,                   
    color="purple",                                                         #(Hunter et al., 2026a; Phillips, 2022)
    alpha=0.5,                  
    s=100,                      
    label="Sleep Quality",                                                  #(Hunter et al., 2026c)
)

ax3.scatter(                                                                #Scatter points for daytime feeling (Bro Code, 2025; Karina Adcock, 2023)
    ST, DF,
    color="green",                                                          #(Hunter et al., 2026a; Phillips, 2022)
    alpha=0.75,
    s=100,
    label="Day Feeling",                                                    #(Hunter et al., 2026c)
)


###For 10 minute intervals. From AI (Based on: Group by: split-apply-combine, 2026; pandas.Series, 2026; Series, 2026)
data["time_interval_s"] = (data["ST_duration_minutes"] // 10) * 10

averages_s = data.groupby("time_interval_s")[
    ["sleep_quality", "day_feeling"]
].mean().sort_index()

##Line Graphs
ax3.plot(                                                                   #For sleep quality (Visually Explained, 2025)
    averages_s.index,
    averages_s["sleep_quality"],
    color="purple",                                                         #(Hunter et al., 2026a; Phillips, 2022)
    linewidth=4,
    marker="o",
    label="Average Sleep Quality",                                          #(Hunter et al., 2026c)
)

ax3.plot(                                                                   #For daytime feeling (Visually Explained, 2025)
    averages_s.index,
    averages_s["day_feeling"],
    color="green",                                                          #(Hunter et al., 2026a; Phillips, 2022)
    linewidth=4,
    marker="o",
    label="Average Day Feeling",                                            #(Hunter et al., 2026c)
)

##Visuals
ax3.xaxis.set_major_formatter(                                              #Formats the x-axis into hours and minutes (Hunter et al., 2026d)
    FuncFormatter(lambda x, pos: f"{int(x // 60)}h \n{int(x % 60)}m")
)
ax3.set_ylim(1, 5)                                                          #(Matplotlib.pyplot.yticks(), 2025)
ax3.set_yticks([1, 2, 2.5, 3, 3.5, 4, 5],
['1', '2', '2.5', '3', '3.5', '4', '5'])
ax3.set_facecolor((0.25, 0.05, 0.4, 0.06))                                  #(Hunter et al., 2026a; Kumar, 2026a; Phillips, 2022)

#==========================================================================================================================================================================================================================================================#Subplot 7/Fig. 3.3b
#Figure 3.3b (SE in relation to Time Spent Sleeping)
##Scatter plot (Bro Code, 2025; Karina Adcock, 2023)
ax7.scatter(                     
    ST, SE,                   
    color="black",                                                          #(Hunter et al., 2026a; Phillips, 2022)
    alpha=0.5,                  
    s=150,
    edgecolors="black",                   
    label="Sleep Efficiency",                                               #(Hunter et al., 2026c)
)

###For 10 minute intervals. From AI (Based on: Group by: split-apply-combine, 2026; pandas.Series, 2026; Series, 2026)
data["time_interval_s"] = (data["ST_duration_minutes"] // 10) * 10

averages_s = data.groupby("time_interval_s")[
    ["sleep_efficiency"]
].mean().sort_index()

##Line Graph                                                                #(Visually Explained)
ax7.plot(                            
    averages_s.index,
    averages_s["sleep_efficiency"],
    color="black",                                                          #(Hunter et al., 2026a; Phillips, 2022)
    linewidth=4,
    marker="o",
    markersize=9,
    markeredgecolor="black",
    label="Average Sleep Efficiency",                                       #(Hunter et al., 2026c)
)

##Visuals
ax7.xaxis.set_major_formatter(
    FuncFormatter(lambda x, pos: f"{int(x // 60)}h \n{int(x % 60)}m")       #(Hunter et al., 2026d)
)
ax7.set_facecolor((0.25, 0.05, 0.4, 0.06))                                  #(Hunter et al., 2026a; Kumar, 2026a; Phillips, 2022)

#==========================================================================================================================================================================================================================================================#Subplot 4/Fig. 3.4a
#Figure 3.4a (SQ and DF in relation to TIBM)
##Scatter plots
ax4.scatter(                                                #Scatter plot for Sleep Quality (SQ) (Bro Code, 2025; Karina Adcock, 2023)
    TIBM, SQ,                   
    color="purple",                                         #(Hunter et al., 2026a; Phillips, 2022)
    alpha=0.5,                  
    s=100,                      
    label="Sleep Quality",                                  #(Hunter et al., 2026c)
)

ax4.scatter(                                                #Scatter points for daytime feeling (Bro Code, 2025; Karina Adcock, 2023)
    TIBM, DF,
    color="green",                                          #(Hunter et al., 2026a; Phillips, 2022)
    alpha=0.75,
    s=100,
    label="Day Feeling",                                    #(Hunter et al., 2026c)
)

###For 10 minute intervals. From AI (Based on: Group by: split-apply-combine, 2026; pandas.Series, 2026; Series, 2026)
data["time_interval_m"] = (data["time_in_bed_morning"] // 10) * 10

averages_m = data.groupby("time_interval_m")[
    ["sleep_quality", "day_feeling"]
].mean().sort_index()

##Line Graphs
ax4.plot(                                                   #For sleep quality (Visually Explained, 2025)
    averages_m.index,
    averages_m["sleep_quality"],
    color="purple",                                         #(Hunter et al., 2026a; Phillips, 2022)
    linewidth=4,
    marker="o",
    label="Average Sleep Quality",                          #(Hunter et al., 2026c)
)

ax4.plot(                                                   #For daytime feeling (Visually Explained, 2025)
    averages_m.index,
    averages_m["day_feeling"],
    color="green",                                          #(Hunter et al., 2026a; Phillips, 2022)
    linewidth=4,
    marker="o",
    label="Average Day Feeling",                            #(Hunter et al., 2026c)
)

##Visuals
ax4.set_ylim(1, 5)                                          #(Matplotlib.pyplot.yticks(), 2025)
ax4.set_yticks([1, 2, 2.5, 3, 3.5, 4, 5])

ax4.set_facecolor((1.0, 0.98, 0.8, 0.2))                    #(Hunter et al., 2026a; Kumar, 2026a; Phillips, 2022)

#==========================================================================================================================================================================================================================================================#Subplot 8/Fig.3.4b
#Figure 3.4b (SE in relation to TIBM)
##Scatter plot (Bro Code, 2025; Karina Adcock, 2023)
ax8.scatter(                     
    TIBM, SE,                   
    color="black",                                                          #(Hunter et al., 2026a; Phillips, 2022)
    alpha=0.5,                  
    s=100,                      
    label="Sleep Efficiency",                                               #(Hunter et al., 2026c)
)

###For 10 minute intervals. From AI (Based on: Group by: split-apply-combine, 2026; pandas.Series, 2026; Series, 2026)
data["time_interval_m"] = (data["time_in_bed_morning"] // 10) * 10

averages_m = data.groupby("time_interval_m")[
    ["sleep_efficiency"]
].mean().sort_index()

##Line Graph                                                                #(Visually Explained)
ax8.plot(                            
    averages_m.index,
    averages_m["sleep_efficiency"],
    color="black",                                                          #(Hunter et al., 2026a; Phillips, 2022)
    linewidth=4,
    marker="o",
    label="Average Sleep Efficiency",                                       #(Hunter et al., 2026c)
)

##Visuals
ax8.set_facecolor((1.0, 0.98, 0.8, 0.2))                                    #(Hunter et al., 2026a; Kumar, 2026a; Phillips, 2022)

#==========================================================================================================================================================================================================================================================#Display and Visuals
#Display and Visuals

##Grid (Grids in Matplotlib, 2026)
for ax in [ax1, ax2, ax3, ax4, ax5, ax6, ax7,ax8]:
    ax.grid(True, linestyle="--", alpha=0.5)

##To get percentages for Sleep Efficiency (Matplotlib.ticker.PercentFormatter class in Python, 2025)
for ax in [ax5,ax6,ax7,ax8]:
    ax.yaxis.set_major_formatter(PercentFormatter(xmax=1, decimals=0))                                     #(Matplotlib.ticker.PercentFormatter class in Python, 2025)

##Fontsizes
for ax in [ax1, ax2, ax3, ax4, ax5, ax6, ax7, ax8]:                                                        #Moved functions from subplots to here to reduce coding lines. Therefore no specific reference.
    ax.tick_params(axis="both", labelsize=20)

##Subplot Labels 
subplot_labels = [
    (ax1, "3.1a"),
    (ax2, "3.2a"),
    (ax3, "3.3a"),
    (ax4, "3.4a"),
    (ax5, "3.1b"),
    (ax6, "3.2b"),
    (ax7, "3.3b"),
    (ax8, "3.4b")
]

for ax, label in subplot_labels:                                                                            #(Hunter et al., 2026e; Kimberly Fessel, 2021)
    ax.text(
        0.02, 0.97,
        label,
        transform=ax.transAxes,
        fontsize=18,
        verticalalignment="top",
        horizontalalignment="left"
    )

##Column Titles (Kimberly Fessel, 2021; Kumar, 2026b)
fig.text(0.17, 0.92, "Prior to Going to Sleep (Minutes)", ha="center", fontsize=24, weight="bold")
fig.text(0.395, 0.92, "Trying to Sleep (Minutes)", ha="center", fontsize=24, weight="bold")
fig.text(0.625, 0.92, "Sleeping", ha="center", fontsize=24, weight="bold")
fig.text(0.855, 0.92, "In the Morning (Minutes)", ha="center", fontsize=24, weight="bold")

##Row Titles (Kimberly Fessel, 2021; Kumar, 2026b)
fig.text(0.03, 0.72, "Sleep Quality and Day Feeling",
         rotation=90, va="center", fontsize=22, weight="bold")
fig.text(0.03, 0.28, "Sleep Efficiency",
         rotation=90, va="center", fontsize=22, weight="bold")

##Main Title (Kimberly Fessel, 2021; Kumar, 2026b)
fig.suptitle(
    "The Impact of Time Spent in Bed on Daytime Feeling, Sleep Quality, and Sleep Efficiency (Fig. 3)",
    fontsize=38,
    fontweight="bold",
    y=0.99
)

##To make the legends shared and outside of the subplots (Hunter et al., 2026c)
handles, labels = [], []

for ax in [ax1, ax2, ax3, ax4, ax5, ax6, ax7, ax8]:
    h, l = ax.get_legend_handles_labels()
    handles.extend(h)
    labels.extend(l)

unique_handles = dict(zip(labels, handles))

fig.legend(
    unique_handles.values(),
    unique_handles.keys(),
    loc="upper center",
    bbox_to_anchor=(0.2, 0.02),
    ncol=3,
    fontsize=22
)

##Text
fig.text(                                                                                   #(Hunter et al., 2026e; Kimberly Fessel, 2021)
    0.8, -0.05,
    r"$\bf{Average\ Time\ Spent\ in\ Bed}$" + "\n"                                          #r"$\bf is a custom string to bolden font (Jonatan Hildén, 2025)
    f"Prior to Going to Sleep:                     {minutes_p} min\n"
    f"Trying to Sleep:                                 {minutes_b} min\n"
    f"Sleeping:                                   {hours_s} h {minutes_s} min\n"
    f"In the Morning:                                  {minutes_m} min",
    transform=fig.transFigure,
    verticalalignment="bottom",
    fontsize=24,
    bbox=dict(
        facecolor="white",
        edgecolor="black"
    )
)

##Display Figure
plt.show()
```

---

## Reference List
Bro Code (2025, October 5) Matplotlib scatter plots in 6 minutes! ✨ [Video]. YouTube. https://youtu.be/HHrG5W530fk?si=IiU965FuQ5woq7kd

Density Plots with Pandas in Python (2025, July 23). geeksforgeeks. https://www.geeksforgeeks.org/python/density-plots-with-pandas-in-python/

Grids in Matplotlib (2026, January 22). geeksforgeeks. https://www.geeksforgeeks.org/python/grids-in-matplotlib/

Group by: split-apply-combine (n.d.). pandas. Retrieved November 4, 2026 from https://pandas.pydata.org/docs/user_guide/groupby.html

Hunter, J., Dale, D., Firing, E., Droettboom, M., & the Matplotlib development team. (2026). List of named colors. matplotlib. https://matplotlib.org/stable/api/_as_gen/matplotlib.pyplot.legend.html

Hunter, J., Dale, D., Firing, E., Droettboom, M., & the Matplotlib development team. (2026). Major and minor ticks. matplotlib. https://venus.matplotlib.org/stable/gallery/ticks/major_minor_demo.html

Hunter, J., Dale, D., Firing, E., Droettboom, M., & the Matplotlib development team. (2026). matplotlib.pyplot.legend. matplotlib. https://matplotlib.org/stable/api/_as_gen/matplotlib.pyplot.legend.html

Hunter, J., Dale, D., Firing, E., Droettboom, M., & the Matplotlib development team. (2026). matplotlib.ticker. matplotlib. https://matplotlib.org/stable/api/ticker_api

Hunter, J., Dale, D., Firing, E., Droettboom, M., & the Matplotlib development team. (2026). Placing text boxes. matplotlib. https://matplotlib.org/stable/gallery/text_labels_and_annotations/placing_text_boxes.html

Jonatan Hildén. (2025, September 9). [Post]. Stack Overflow. https://stackoverflow.com/questions/34937048/make-part-of-a-title-bold-and-a-different-color#comment140724228_44123579

Karina Adcock (2021, March 3) Scatter plot with third variable as color | Python Matplotlib [Video]. Youtube. https://youtu.be/VolIkTkYqMw?si=DhuETOkYruWQE8Hb

Karina Adcock (2023, September 20) How to make scatter plot with trendline and stats in python [Video]. YouTube. https://youtu.be/lapKRrSdn_U?si=xFho8jFS_p8E2aA1

Kimberly Fessel (2021, February 8) Add TEXT to MATPLOTLIB figures || Python matplotlib text with plt.text() || Matplotlib Tips [Video]. YouTube. https://youtu.be/NBYzSaTbodM?si=iyU5VYPyu24Pet_4

Kumar, B. (2026, September 22). How to Change the Background Color in Matplotlib Python Guides. https://pythonguides.com/matplotlib-change-background-color/

Kumar, B. (2026, September 24). Matplotlib Subplot Titles and the Figure suptitle Python Guides. https://pythonguides.com/set-titles-for-each-subplot-and-overall-title-matplotlib/

Matplotlib.pyplot.yticks() in Python (2025, July 12). geeksforgeeks. https://www.geeksforgeeks.org/python/matplotlib-pyplot-yticks-in-python/

Matplotlib Subplots (2025, October 14). geeksforgeeks. https://www.geeksforgeeks.org/python/matplotlib-pyplot-subplots-in-python/

Matplotlib.ticker.PercentFormatter class in Python (2025, July 12). geeksforgeeks. https://www.geeksforgeeks.org/python/matplotlib-ticker-percentformatter-class-in-python/

Ms. Simmons' Math Class (2025, March 18) Python: Bubble Charts [Video]. Youtube. https://youtu.be/0wLB9YJn0hs?si=4-7gHhRMWe5AhinE

pandas.Series (n.d.). pandas. Retrieved November 4, 2026 from https://pandas.pydata.org/docs/reference/api/pandas.Series.html

pandas.Timedelta (n.d.). pandas. Retrieved October 21, 2026 from https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.Timedelta.html

Phillips, A. (2022, June 29). Colorblind Safe Color Schemes NCEAS Science Communication Resource Corner. https://www.nceas.ucsb.edu/sites/default/files/2022-06/Colorblind%20Safe%20Color%20Schemes.pdf

Ryan Noonan (2018, November 7) Python Bubble Chart with Labels and Legend [Video]. Youtube. https://youtu.be/6fPBNp_XLxU?si=hVvRPZlsxJ-c2usp

Series (n.d.). pandas. Retrieved November 4, 2026 from https://pandas.pydata.org/docs/reference/series.html

Simply Coding (2020, April 2) Python Pie Chart [Video]. YouTube. https://youtu.be/8GdvRNblTqg?si=bjGwWTz0h6Ph5fmg

Stojiljković, M. (2024, December 7). Linear Regression in Python Real Python. https://realpython.com/linear-regression-in-python/

Time deltas (n.d.). pandas. Retrieved October 21, 2026 from https://pandas.pydata.org/docs/user_guide/timedeltas.html

Visually Explained (2025, January 20) Matplotlib (Python Library) - Visually Explained [Video]. YouTube. https://youtu.be/gamozygB2xE?si=RlCxFo0N7PCMSZPz