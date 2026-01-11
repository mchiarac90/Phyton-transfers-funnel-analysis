###  Transfers Funnel Analysis using Phyton

#### This project evaluates the operational health of a transfer route.
#### The goal is to analyze the data flow to identify systemic bottlenecks.

#### Toos: Excel, Colab

#### Data: [Github](https://github.com/mchiarac90/Phyton-transfers-funnel-analysis/blob/main/%20funnel_data_sample.xlsx)
#### Data Glossary:[Github](https://github.com/mchiarac90/Phyton-transfers-funnel-analysis/blob/main/Glossary.png)

### 1. Import libraries
<pre><code>
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
import plotly.graph_objects as go
</code></pre>

### 2. Import the excel file 
<pre><code>df = pd.read_excel('funnel_data_sample.xlsx') </code></pre>

### 3. Count each unique event name to build the funnel stages
<pre><code>funnel_counts = df['event_name'].value_counts() </code></pre>

### 4. Extract specific counts for the core funnel stages
<pre><code>created_count = funnel_counts.get('Transfer Created', 0)
funded_count = funnel_counts.get('Transfer Funded', 0)
transferred_count = funnel_counts.get('Transfer Transferred', 0) </code></pre>

### 5. Calculate Conversion Metrics
<pre><code>perc_funded = (created_count / created_count * 100) if created_count > 0 else 0
perc_funded_to_created = (funded_count / created_count * 100) if created_count > 0 else 0
perc_transferred_to_funded = (transferred_count / funded_count * 100) if funded_count > 0 else 0 </code></pre>

### 6. Output Results
<pre><code>print(f"Percentage of transfers 'funded' to 'created': {perc_funded_to_created:.2f}%")
print(f"Percentage of transfers 'transferred' to 'funded': {perc_transferred_to_funded:.2f}%") </code></pre>

### 7. Create funnel table
#### 7.1 Define the order of stages and construct a specific DataFrame for the chart
<pre><code>funnel_steps_order = ['Transfer Created', 'Transfer Funded', 'Transfer Transferred']
funnel_data_for_chart = pd.DataFrame({
    'Stage': funnel_steps_order,
    'Count': [created_count, funded_count, transferred_count] </code></pre>
})

#### 7.2 Normalize calculations: calculate the percentage of total based on the 'Transfer Created' volume
<pre><code>initial_count = funnel_data_for_chart.loc[funnel_data_for_chart['Stage'] == 'Transfer Created', 'Count'].iloc[0]
funnel_data_for_chart['Percentage'] = (funnel_data_for_chart['Count'] / initial_count * 100).round(2) </code></pre> 

#### 7.3 Build the Funnel Chart
<pre><code>fig = go.Figure(go.Funnel(
    y = funnel_data_for_chart['Stage'],
    x = funnel_data_for_chart['Count'],
    textposition = "outside",
    # Shows the raw count + percentage retained from the previous stage
    textinfo = "value+percent previous",
    marker = {"color": ["deepskyblue", "salmon", "lightgreen"]},
    connector = {"line": {"color": "royalblue", "dash": "dot", "width": 2}} </code></pre>

#### 7.4 Custom Styling & Layout
<pre><code>fig.update_layout(
    title_text="<b>Transfer Funnel Progression</b>",
    title_x=0.5,
    xaxis_title="<b>Number of Transfers</b>",
    plot_bgcolor='white',
    paper_bgcolor='white',
    font=dict(size=15, color="black"),
    height=800,
    width=1300,
    margin=dict(l=100, r=40, t=100, b=100)
) 
  
fig.update_traces(textfont=dict(color="black", size=15, family="Arial", weight="bold")) </code></pre>

#### 7.6 Display the chart
<pre><code>fig.show() </code></pre>

#### Result: [Github](https://github.com/mchiarac90/Phyton-transfers-funnel-analysis/blob/main/Funnel%20chart.png)

### 8. Analyze correlations
#### 8.1 Create binary flags (1 or 0) for transfer state to allow easy summation during grouping
 <pre><code>df['is_funded'] = np.where(df['event_name'] == 'Transfer Funded', 1, 0)
df['is_created'] = np.where(df['event_name'] == 'Transfer Created', 1, 0)  </code></pre>

#### 8.2 Group by the dimensions that impact user behavior: Region, Experience, and Platform
<pre><code>grouped_df = df.groupby(['region', 'experience', 'platform']).agg(
    funded_count=('is_funded', 'sum'),
    created_count=('is_created', 'sum')
).reset_index()  </code></pre>

#### 8.3 Calculate the percentage of funded transfers per group
<pre><code>grouped_df['perc_funded_of_created'] = (
    (grouped_df['funded_count'] / grouped_df['created_count'].replace(0, np.nan)) * 100
).fillna(0) </code></pre>

#### 8.4 Sort by the highest performing segments
<pre><code>final_result = grouped_df[['region', 'experience', 'platform', 'perc_funded_of_created']]
final_result = final_result.sort_values(by='perc_funded_of_created', ascending=False) </code></pre>

#### 8.5 Display the result
<pre><code>print(final_result) </code></pre>

### 9. Create heatmap
<pre><code>heatmap_data = final_result.pivot_table(
index=['region', 'experience'],
columns='platform',
values='perc_funded_of_created')

plt.figure(figsize=(12, 8))
sns.heatmap(heatmap_data, annot=True, fmt=".1f", cmap="YlOrRd_r", linewidths=.5, linecolor='black')
plt.title('Percentage Funded of Created by Region, Experience and Platform')
plt.xlabel('Platform')
plt.ylabel('Region, Experience')
plt.tight_layout()
plt.show() </code></pre>

#### Result: [Github](https://github.com/mchiarac90/Phyton-transfers-funnel-analysis/blob/main/Funnel%20chart.png)

### 10. Check absolute number of users by platform to assess impact of UX enhancement plan
<pre><code>filtered_web = df[df['platform'] == 'Web']
distinct_users_on_web = filtered_df['user_id'].nunique()
filtered_android = df[df['platform'] == 'Android']
distinct_users_on_android = filtered_android['user_id'].nunique()
print(distinct_users_on_web)
print(distinct_users_on_android)</code></pre>






