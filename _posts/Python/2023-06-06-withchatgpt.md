---
layout: post
title:  "和chatgpt一起迅速完成了一个python小工具"
date:   2023-06-06
category: Python
---

### 环境 

python 3.10.9

windows10

### 目的
我们公司是用jira这个网站进行任务管理的。我近几个月才开始接触，很多地方不知道怎么看，比如**我想要看分配给自己的任务有哪些，并希望它们按照计划开始时间和计划完成时间显示在甘特图上**。搜索了几次也没有结果，就准备自己用python写一个了。不懂的地方就问chatgpt，或者干脆让它写，我负责运行看效果，提意见，几个来回就改好了，全程不到一个小时。

### 效果图
左侧是任务的ID和标题，右侧是该任务对应的时间段，色块中显示该任务的状态。

![Pasted image 20230606085037](https://github.com/Minoa0313/MyImage/blob/main/img/20230606091018.png?raw=true)



### 代码
需要将账户token以及网址部分进行替换。
```python
import requests
from jira import JIRA
from jira.exceptions import JIRAError
import matplotlib.pyplot as plt
from datetime import datetime
import matplotlib.dates as mdates

user = "my_acount"
token = "my_token"

class jira_gantt_generator:
    def __init__(self):
        options = {'server': "jira_url" }
        
        try:
            jira = JIRA(options=options, basic_auth=(user, token), timeout=30, max_retries=3)
            user_info = jira.myself()
            print("Jira Login Success!!")
        except JIRAError as e:
            if e.status_code == 401:
                raise Exception("Login to JIRA failed(401).")
            elif isinstance(e, requests.exceptions.Timeout):
                raise Exception("Timeout")
            elif isinstance(e, requests.exceptions.TooManyRedirects):
                raise Exception("TooManyRedirect")
            else:
                raise Exception(e)

        self.jira = jira

    def get_my_issues(self):
        issues = self.jira.search_issues('assignee = currentUser() AND resolution = Unresolved')

        tasks = []

        for issue in issues:
            summary = issue.fields.summary
            status = issue.fields.status
            plan_start_date = issue.fields.customfield_10135  
            plan_end_date = issue.fields.customfield_10102  
            tasks.append({
                'summary': summary,
                'issue': issue.key,
                'status': status,
                'plan_start_date': plan_start_date,
                'plan_end_date': plan_end_date
            })

        return tasks

    def gantt_generator(self):
        tasks = self.get_my_issues()
        tasks_with_start_dates = [task for task in tasks if task['plan_start_date'] is not None and task['plan_end_date'] is not None]
        for task in tasks_with_start_dates:
            task['plan_start_date'] = datetime.strptime(task['plan_start_date'], "%Y-%m-%d")
            task['plan_end_date'] = datetime.strptime(task['plan_end_date'], "%Y-%m-%d")
            task['plan_duration'] = (task['plan_end_date'] - task['plan_start_date']).days

        plt.figure(figsize=(10, 6))
        plt.rcParams['font.family'] = 'MS Gothic'
        ax = plt.gca()
        ax.xaxis.set_major_formatter(mdates.DateFormatter('%m-%d'))  
        ax.xaxis_date() 

        for i, task in enumerate(tasks_with_start_dates):
            plt.barh(i, task['plan_duration'], left=task['plan_start_date'], height=0.8, align='center')
            plt.text(task['plan_start_date'], i, task['plan_start_date'].strftime('%m-%d'), ha='right', va='center')
            plt.text(task['plan_end_date'], i, task['plan_end_date'].strftime('%m-%d'), ha='left', va='center')
            mid_date = task['plan_start_date'] + (task['plan_end_date'] - task['plan_start_date']) / 2
            plt.text(mid_date, i, task['status'], ha='center', va='center', color='white')
        plt.yticks(range(len(tasks_with_start_dates)), [task['issue'] + ": " + task['summary'] for task in tasks_with_start_dates]) 

        plt.xlabel('Date')
        plt.ylabel('Task')
        plt.grid(axis='x')
        plt.tight_layout()
        plt.show()

gg = jira_gantt_generator()
gg.gantt_generator()
```


### 遇到的问题
1. 我一开始以为是要显示开始时间和结束时间，但其实是计划时间。因此最初获得的结束时间为空，图标上什么也没有显示出来。

2. 每个公司为计划时间分配的ID字段不同，这个需要自己去查。

   可以通过以下几句代码，将ID和字段打印出来，以便查看。

      ```python 
   fields = jira.fields() 
   for field in fields: 
       print(f"Name: {field['name']}, ID: {field['id']}")
      ```

3. 在显示日语时出现了错误，日语字都变成方框，无法显示。后通过更换显示字体解决。 
   `plt.rcParams['font.family'] = 'MS Gothic'`

