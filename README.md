# All the details codes for this project are given below:
# Import the nesessaries libraries
import pandas as pd
import numpy as np
# Set the pandas option to display full column width
pd.set_option('display.max_colwidth', None)

# Read an excelfile into Pands DataFrame
df_orig = pd.read_excel('C:/Users/bhojr/Desktop/Reptile_bibliography/Reptile_bibliography Nov 2022.xlsx')
# Create the copy of the original DataFrame
df = df_orig.copy()
# Remove the duplicated column
df = df.loc[:, ~df.columns.duplicated()]
# Select only the email and authors column from the DataFrame
#df = df[['email', 'author']]
# Convert the email and author columns to strings
df['email'] = df['email'].astype(str)
df['author'] = df['author'].astype(str)

# Check the column
df.columns
# Define function to clean the text data
import re
def clean_corpus(nlp):
    words= nlp.lower()
    mytext=re.sub(r'[^a-zA-Z0-9\,\@\.]', ' ',words)
    mytext=re.sub(r' +', ' ', mytext)
    return mytext.strip()
    # Apply the 'clean_corpus' function to the email and author columns
    
df['email_clean'] =  df.email.apply(clean_corpus)
df['author_clean'] =  df.author.apply(clean_corpus)

# Strip whitespace, replace empty string with Nan and replace 'nan'
df = df.replace(r'^\s*$', np.nan, regex=True)
df['email_clean'] = df[['email_clean']].apply(lambda x: x.str.strip()).replace('', np.nan).replace('nan', None)
df['author_clean'] = df[['author_clean']].apply(lambda x: x.str.strip()).replace('', np.nan).replace('nan', None)
# Rename columns to author and email after the cleaning
df  = df[['author_clean', 'email_clean', 'year', 'title', 'source', 'ref#', 'kind', 'url']]
df.rename({'author_clean':'author', 'email_clean': 'email'}, axis =1 , inplace = True)

#Split the data by author name
df['first_author'] = df['author'].str.split(',').str[0]
df['second_author'] = df['author'].str.split(',').str[1]
df['third_author'] = df['author'].str.split(',').str[2]
df['forth_author'] = df['author'].str.split(',').str[3]
df['fifth_author'] = df['author'].str.split(',').str[4]
df['last_author'] = df['author'].str.split(',').str[-1]
df['sndlast_author'] = df['author'].str.split(',').str[-2]


# Split the data by email address components
df['username_email'] = df['email'].str.split('@').str[0]
df['first_author'] = df['first_author'].astype(str)
df['second_author'] = df['second_author'].astype(str)
df['third_author'] = df['third_author'].astype(str)
df['forth_author'] = df['forth_author'].astype(str)
df['fifth_author'] = df['fifth_author'].astype(str)
df['sndlast_author'] = df['sndlast_author'].astype(str)
df['last_author'] = df['last_author'].astype(str)
df['username_email'] = df['username_email'].astype(str)
df['Likely_Author1'] = df.apply(lambda x: 'Yes' if x['first_author'] in x['username_email'] else 'No',axis=1)
df['Likely_Author2'] = df.apply(lambda x: 'Yes' if x['second_author'] in x['username_email'] else 'No',axis=1)
df['Likely_Author3'] = df.apply(lambda x: 'Yes' if x['third_author'] in x['username_email'] else 'No',axis=1)
df['Likely_Author4'] = df.apply(lambda x: 'Yes' if x['forth_author'] in x['username_email'] else 'No',axis=1)
df['Likely_Author5'] = df.apply(lambda x: 'Yes' if x['fifth_author'] in x['username_email'] else 'No',axis=1)
df['Likely_Author6'] = df.apply(lambda x: 'Yes' if x['sndlast_author'] in x['username_email'] else 'No',axis=1)
df['Likely_Author7'] = df.apply(lambda x: 'Yes' if x['last_author'] in x['username_email'] else 'No',axis=1)
df['email_duplicate'] = df.loc[:, 'email']
# List the authors having similar emails
correct_author = []
author = []
emailaddress = []
for index, row in df.iterrows():
    if row['Likely_Author1'] == 'Yes' and row['first_author'] != 'nan':
        if row['first_author'] not in author:
            author.append(row['first_author'])
            emailaddress.append(row['email'])
    elif row['Likely_Author2'] == 'Yes' and row['second_author'] != 'nan':
        if row['second_author'] not in author:
            author.append(row['second_author'])
            emailaddress.append(row['email'])
    elif row['Likely_Author3'] == 'Yes' and row['third_author'] != 'nan':
        if row['third_author'] not in author:
            author.append(row['third_author'])
            emailaddress.append(row['email'])
    elif row['Likely_Author4'] == 'Yes' and row['forth_author'] != 'nan':
        if row['forth_author'] not in author:
            author.append(row['forth_author'])
            emailaddress.append(row['email'])
    elif row['Likely_Author5'] == 'Yes' and row['fifth_author'] != 'nan':
        if row['fifth_author'] not in author:
            author.append(row['fifth_author'])
            emailaddress.append(row['email'])
    elif row['Likely_Author6'] == 'Yes' and row['sndlast_author'] != 'nan':
        if row['sndlast_author'] not in author:
            author.append(row['sndlast_author'])
            emailaddress.append(row['email'])
    if row['Likely_Author7'] == 'Yes' and row['last_author'] != 'nan':
        if row['last_author'] not in author:
            author.append(row['last_author'])
            emailaddress.append(row['email'])
df['author'] = df['author'].astype(str)
# Dictionary for mapping the authors and email address
author_dict = dict(zip(author, emailaddress))
#author_dict
df = df.replace({"first_author": author_dict,
                 "second_author": author_dict,
                "third_author": author_dict,
                "forth_author": author_dict,
                "fifth_author": author_dict,
                "sndlast_author": author_dict,
                "last_author": author_dict})
def find_email(text):
    email = re.findall(r'[\w\.-]+@[\w\.-]+',str(text))
    return ",".join(email)

columns = ['email', 'first_author', 'second_author', 'third_author',
       'forth_author', 'fifth_author', 'last_author', 'sndlast_author']
for column in columns:
    df[column] = df[column].apply(lambda x: find_email(x))
df.columns
# df = df[['author', 'email', 'first_author', 'second_author', 'third_author',
# 'forth_author', 'fifth_author', 'last_author', 'sndlast_author']]
df["email_final"] = df["email"].map(str) + ' ' + \
                    df["first_author"].map(str) + ' ' + \
                    df["second_author"].map(str) + ' ' +\
                    df["third_author"].map(str) + ' ' +\
                    df["forth_author"].map(str) + ' ' +\
                    df["fifth_author"].map(str) + ' ' +\
                    df["sndlast_author"].map(str) + ' ' +\
                    df["last_author"].map(str)

df["email_final"] = df["email_final"].apply(lambda x: ' '.join(pd.unique(x.split())))
df['email_final'] = df['email_final'].str.split(',').str[0]
df.columns

#df = df[['author', 'email', 'email_final']]
df.sample(2)
df = df[['author', 'email', 'year', 'title', 'source', 'ref#', 'kind', 'url','email_final']]

# Total length of Data
len(df)
# Replace the empty rows with new email
df ['email_final']= df[['email_final']].apply(lambda x: x.str.strip()).replace('', np.nan).replace('nan',None)
df ['email']= df[['email']].apply(lambda x: x.str.strip()).replace('', np.nan).replace('nan',None)

# List the five samples in a data
df.sample(5)
# Check the total null email in a data
sum(pd.isnull(df['email']))

# Total email in the data 
sum(pd.notnull(df['email']))
#Total null email after the new email refilled in a data 
sum(pd.isnull(df['email_final']))
# Total emails after filled in the data
sum(pd.notnull(df['email_final']))
# Create the out put file with filled emails into csv format 
df.to_csv('Reptile_final_filled_new_email.csv')

# Create the output file with filled emails into xlsx format
df.to_excel('Reptile_filled_new_email.xlsx')
