import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
import streamlit as st
from babel.numbers import format_currency
sns.set(style='dark')

#create_monthlybikesharing_df sebagai helper function
def create_monthly_bikesharing_df(bikesharing_df):
    monthly_bikesharing_df = bikesharing_df.resample(rule='M', on='dteday').agg({
        "instant": "nunique",
        "cnt": "sum"
    })

    monthly_bikesharing_df = monthly_bikesharing_df.reset_index()
    monthly_bikesharing_df.rename(columns={
        "cnt": "total_user"
    }, inplace=True)

    return monthly_bikesharing_df

#create_weathersit sebagai helper function
def create_weathersit_bikesharing_df(bikesharing_df):
    weathersit_bikesharing_df = bikesharing_df.groupby(by="weathersit").agg({
                                    "instant": "nunique",
                                    "cnt": "sum"
                                })

    weathersit_bikesharing_df = weathersit_bikesharing_df.reset_index()
    weathersit_bikesharing_df.rename(columns={
        "instant": "Number of Days",
        "cnt": "Total Users"
    }, inplace=True)

    return weathersit_bikesharing_df

#create_day_clustering sebagai helper function
def create_day_clustering_df(bikesharing_df):
    bikesharing_df['t_rank'] = bikesharing_df['temp'].rank(ascending=False)
    bikesharing_df['h_rank'] = bikesharing_df['hum'].rank(ascending=True)
    bikesharing_df['w_rank'] = bikesharing_df['windspeed'].rank(ascending=True)

    bikesharing_df['t_rank_norm'] = (bikesharing_df['t_rank']/bikesharing_df['t_rank'].max())*100
    bikesharing_df['h_rank_norm'] = (bikesharing_df['h_rank']/bikesharing_df['h_rank'].max())*100
    bikesharing_df['w_rank_norm'] = (bikesharing_df['w_rank']/bikesharing_df['w_rank'].max())*100

    bikesharing_df.drop(columns=['t_rank', 'h_rank', 'w_rank'], inplace=True)

    bikesharing_df['day_score'] = 0.6*bikesharing_df['t_rank_norm']+0.2 * \
    bikesharing_df['h_rank_norm']+0.2*bikesharing_df['w_rank_norm']
    bikesharing_df['day_score'] *= 0.05
    bikesharing_df = bikesharing_df.round(2)

    bikesharing_df["day_clustering"] = np.where(
        bikesharing_df['day_score'] > 4.2, "Bad day", ( np.where(
            bikesharing_df['day_score'] > 3.5, 'Normal day', 'Good day')))

    day_clustering_df = bikesharing_df.groupby(by="day_clustering", as_index=False).dteday.nunique()

    day_clustering_df['day_clustering'] = pd.Categorical(day_clustering_df['day_clustering'], [
    "Good day", "Normal day", "Bad day"
    ])

    return day_clustering_df

#Load Cleaned Data
bikesharing_df = pd.read_csv("proyekanalisisbikesharing.csv")

datetime_columns = ["dteday"]
bikesharing_df.sort_values(by="dteday", inplace=True)
bikesharing_df.reset_index(inplace=True)
 
for column in datetime_columns:
    bikesharing_df[column] = pd.to_datetime(bikesharing_df[column])

#Filter data
min_date = bikesharing_df["dteday"].min()
max_date = bikesharing_df["dteday"].max()
 
with st.sidebar:
    # Menambahkan logo perusahaan
    st.image("https://github.com/Mukabatak/bikesharingdashboard/blob/main/dicoding-header-logo.png?raw=true")
    
    # Mengambil start_date & end_date dari date_input
    start_date, end_date = st.date_input(
        label='Rentang Waktu',min_value=min_date,
        max_value=max_date,
        value=[min_date, max_date]
    )

main_bikesharing_df = bikesharing_df[(bikesharing_df["dteday"] >= str(start_date)) & 
                (bikesharing_df["dteday"] <= str(end_date))]

#Menyiapkabn beberapa dataframe
monthly_bikesharing_df = create_monthly_bikesharing_df(main_bikesharing_df)
weathersit_bikesharing_df = create_weathersit_bikesharing_df(main_bikesharing_df)
day_clustering_df =  create_day_clustering_df(main_bikesharing_df)

#title header
st.header ('Dicoding Bike Sharing Dashboard :sparkles:') 

#plot number user monthly
st.subheader('Insights 1')

total_users = bikesharing_df.cnt.sum()
st.metric("Total Users", value=total_users)

fig, ax = plt.subplots(figsize=(16, 8))
ax.plot(
    monthly_bikesharing_df["dteday"],
    monthly_bikesharing_df["total_user"],
    marker='o',
    linewidth=2,
    color="#72BCD0"
)
ax.set_title("Monthly Bike Sharing Users", loc="center", fontsize=30)
ax.tick_params(axis='y', labelsize=20)
ax.tick_params(axis='x', labelsize=15)

st.pyplot(fig)

#plot weathersit
fig,ax = plt.subplots(figsize=(8,8))
colors = ["#72BCD4", "#D3D3D3", "#D3D3D3", "#D3D3D3", "#D3D3D3"]
sns.barplot(
    x="weathersit",
    y="Total Users",
    data=weathersit_bikesharing_df,
    estimator=sum,
    ci=None,
    palette=colors
)
ax.set_title("Number of Bike Sharing Users based on Weather Situation", loc="center", fontsize=30)
ax.set_ylabel(None)
ax.set_xlabel(None)
ax.tick_params(axis='y', labelsize=20)
ax.tick_params(axis='x', labelsize=15)
st.pyplot(fig)

#plot day clustering
st.subheader("Insight 2")

fig, ax = plt.subplots(figsize=(20,10))
colors_ = ["#72BCD4", "#D3D3D3", "#D3D3D3"]
sns.barplot(
    x="dteday",
    y="day_clustering",
    data=day_clustering_df.sort_values(by="day_clustering", ascending=False),
    palette=colors_
)
ax.set_title("Number of Good Day for Riding a Bike (2011-2012)", loc="center", fontsize=30)
ax.set_ylabel(None)
ax.set_xlabel(None)
ax.tick_params(axis='y', labelsize=20)
ax.tick_params(axis='x', labelsize=15)
st.pyplot(fig)
