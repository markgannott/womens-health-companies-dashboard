import streamlit as st
import pandas as pd
import plotly.express as px
import plotly.graph_objects as go

# Set page config
st.set_page_config(page_title="Women's Health Companies Analysis", layout="wide")

# Create the dataset
data = {
    'Company': [
        'AbbVie', 'Abbott Laboratories', 'Fuji Pharma Co. Ltd.', 'Veru Inc.', 
        'Pulsenmore Ltd.', 'Daré Bioscience Inc.', 'Femasys Inc.', 
        "Aspira Women's Health", 'Palatin Technologies Inc.', 'Mithra Pharmaceuticals',
        'The Cooper Companies', 'Hologic Inc.', 'Creative Medical Technology', 
        'Minerva Surgical', 'Organon & Co.', 'INVO Bioscience', 'Agile Therapeutics',
        'Bonzun', 'Evofem Biosciences Inc.', 'Callitas Therapeutics'
    ],
    'Market_Cap': [
        319.84, 207.08, 0.2819, 0.406, 0.175, 0.185, 0.203, 0.247, 0.275, 0.289,
        20.3, 19.44, 0.302, 0.357, 40.1, 2.69, 2.03, 0.339, 0.128, 0.0033
    ],
    'Founded': [
        2013, 1888, 1965, 1971, 2014, 2004, 2004, 1993, 1986, 1999,
        1958, 1985, 1998, 2008, 2021, 2007, 1997, 2012, 2007, 2003
    ],
    'Employees': [
        50000, 113000, 1600, 190, 50, 40, 40, 100, 20, 500,
        15000, 6940, 10, 240, 9000, 20, 20, 50, 40, 0
    ],
    'Location': [
        'Illinois, USA', 'Illinois, USA', 'Tokyo, Japan', 'Florida, USA', 
        'Omer, Israel', 'California, USA', 'Georgia, USA', 'Texas, USA',
        'New Jersey, USA', 'Liège, Belgium', 'California, USA', 'Massachusetts, USA',
        'Arizona, USA', 'California, USA', 'New Jersey, USA', 'Florida, USA',
        'New Jersey, USA', 'Stockholm, Sweden', 'California, USA', 'British Columbia, CA'
    ],
    'Exchange': [
        'NYSE', 'NYSE', 'TYO', 'NASDAQ', 'TASE', 'NASDAQ', 'NASDAQ', 'NASDAQ',
        'NYSE', 'EBR', 'NASDAQ', 'NASDAQ', 'NASDAQ', 'NASDAQ', 'NYSE', '',
        'NASDAQ', 'STO', 'OTCMKTS', 'OTCMKTS'
    ]
}

df = pd.DataFrame(data)

# Title and description
st.title("🏥 Women's Health Companies Analysis")
st.markdown("Analysis of top 20 women's health companies by market capitalization")

# Key metrics
col1, col2, col3 = st.columns(3)

with col1:
    st.metric(
        "Total Market Cap",
        f"${df['Market_Cap'].sum():.2f}B",
        f"{len(df)} Companies"
    )

with col2:
    st.metric(
        "Average Company Age",
        f"{2024 - df['Founded'].mean():.0f} years",
        f"Oldest: {df['Founded'].min()}"
    )

with col3:
    st.metric(
        "Total Employees",
        f"{df['Employees'].sum():,.0f}",
        f"Avg: {df['Employees'].mean():,.0f}"
    )

# Market Cap Visualization
st.subheader("Market Capitalization Distribution")
fig_market = px.treemap(
    df,
    path=[px.Constant("All"), 'Company'],
    values='Market_Cap',
    title='Company Market Caps (Billion USD)',
    color='Market_Cap',
    color_continuous_scale='Viridis'
)
st.plotly_chart(fig_market, use_container_width=True)

# Geographic Distribution
col1, col2 = st.columns(2)

with col1:
    # Company age distribution
    fig_age = px.scatter(
        df,
        x='Founded',
        y='Market_Cap',
        size='Employees',
        hover_name='Company',
        title='Company Age vs Market Cap',
        labels={'Founded': 'Founded Year', 'Market_Cap': 'Market Cap (Billion USD)'}
    )
    st.plotly_chart(fig_age, use_container_width=True)

with col2:
    # Count companies by location
    location_counts = df['Location'].str.extract(r'(.*,\s*[A-Z]{2,3})$')[0].value_counts()
    fig_loc = px.pie(
        values=location_counts.values,
        names=location_counts.index,
        title='Geographic Distribution'
    )
    st.plotly_chart(fig_loc, use_container_width=True)

# Stock Exchange Distribution
st.subheader("Distribution by Stock Exchange")
exchange_counts = df['Exchange'].value_counts()
fig_exchange = px.bar(
    x=exchange_counts.index,
    y=exchange_counts.values,
    labels={'x': 'Stock Exchange', 'y': 'Number of Companies'},
    title='Companies per Stock Exchange'
)
st.plotly_chart(fig_exchange, use_container_width=True)

# Detailed Company Data
st.subheader("Company Details")
detailed_df = df.sort_values('Market_Cap', ascending=False)
detailed_df['Market_Cap'] = detailed_df['Market_Cap'].apply(lambda x: f"${x:.3f}B")
detailed_df['Employees'] = detailed_df['Employees'].apply(lambda x: f"{x:,.0f}")
st.dataframe(
    detailed_df,
    use_container_width=True,
    hide_index=True
)

# Add download capability
csv = df.to_csv(index=False)
st.download_button(
    label="Download Data as CSV",
    data=csv,
    file_name="womens_health_companies.csv",
    mime="text/csv"
)