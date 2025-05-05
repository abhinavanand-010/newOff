# newOff

- Code refactor: No changes related to app.py. Only commented out NLP imports and methods.
```python
import streamlit as st
import pandas as pd

# # NLP pkgs
# import spacy
# from spacy import displacy
# nlp = spacy.load('en')

# Database Functions
from db_fxns import *

from wordcloud import WordCloud, STOPWORDS, ImageColorGenerator

import matplotlib.pyplot as plt
import matplotlib
matplotlib.use('Agg')


# Avatar Image using a url
avatar1 ="https://www.w3schools.com/howto/img_avatar1.png"
avatar2 ="https://www.w3schools.com/howto/img_avatar2.png"

# Reading Time
def readingTime(mytext):
	total_words = len([ token for token in mytext.split(" ")])
	estimatedTime = total_words/200.0
	return estimatedTime

# def analyze_text(text):
# 	return nlp(text)

# Layout Templates
title_temp ="""
	<div style="background-color:#464e5f;padding:10px;border-radius:10px;margin:10px;">
	<h4 style="color:white;text-align:center;">{}</h1>
	<img src="https://www.w3schools.com/howto/img_avatar.png" alt="Avatar" style="vertical-align: middle;float:left;width: 50px;height: 50px;border-radius: 50%;" >
	<h6>Author:{}</h6>
	<br/>
	<br/>	
	<p style="text-align:justify">{}</p>
	</div>
	"""
article_temp ="""
	<div style="background-color:#464e5f;padding:10px;border-radius:5px;margin:10px;">
	<h4 style="color:white;text-align:center;">{}</h1>
	<h6>Author:{}</h6> 
	<h6>Post Date: {}</h6>
	<img src="https://www.w3schools.com/howto/img_avatar.png" alt="Avatar" style="vertical-align: middle;width: 50px;height: 50px;border-radius: 50%;" >
	<br/>
	<br/>
	<p style="text-align:justify">{}</p>
	</div>
	"""
head_message_temp ="""
	<div style="background-color:#464e5f;padding:10px;border-radius:5px;margin:10px;">
	<h4 style="color:white;text-align:center;">{}</h1>
	<img src="https://www.w3schools.com/howto/img_avatar.png" alt="Avatar" style="vertical-align: middle;float:left;width: 50px;height: 50px;border-radius: 50%;">
	<h6>Author:{}</h6> 		
	<h6>Post Date: {}</h6>		
	</div>
	"""
full_message_temp ="""
	<div style="background-color:silver;overflow-x: auto; padding:10px;border-radius:5px;margin:10px;">
		<p style="text-align:justify;color:black;padding:10px">{}</p>
	</div>
	"""

HTML_WRAPPER = """<div style="overflow-x: auto; border: 1px solid #e6e9ef; border-radius: 0.25rem; padding: 1rem">{}</div>"""

def main():
	"""A Simple CRUD Blog App"""
	html_temp = """
		<div style="background-color:{};padding:10px;border-radius:10px">
		<h1 style="color:{};text-align:center;">Simple Blog </h1>
		</div>
		"""
	st.markdown(html_temp.format('royalblue','white'),unsafe_allow_html=True)
		

	menu = ["Home","View Post","Add Post","Search","Manage Blog"]
	choice = st.sidebar.selectbox("Menu",menu)

	if choice == "Home":
		st.subheader("Home")		
		result = view_all_notes()
		for i in result:
			# short_article = str(i[2])[0:int(len(i[2])/2)]
			short_article = str(i[2])[0:50]
			st.write(title_temp.format(i[1],i[0],short_article),unsafe_allow_html=True)

		# st.write(result)
	elif choice == "View Post":
		st.subheader("View Post")

		all_titles = [i[0] for i in view_all_titles()]
		postlist = st.sidebar.selectbox("Posts",all_titles)
		post_result = get_blog_by_title(postlist)
		for i in post_result:
			st.text("Reading Time:{} minutes".format(readingTime(str(i[2]))))
			st.markdown(head_message_temp.format(i[1],i[0],i[3]),unsafe_allow_html=True)
			st.markdown(full_message_temp.format(i[2]),unsafe_allow_html=True)

			# if st.button("Analyze"):
	
			# 	docx = analyze_text(i[2])
			# 	html = displacy.render(docx,style="ent")
			# 	html = html.replace("\n\n","\n")
			# 	st.write(HTML_WRAPPER.format(html),unsafe_allow_html=True)

			


	elif choice == "Add Post":
		st.subheader("Add Your Article")
		create_table()
		blog_title = st.text_input('Enter Post Title')
		blog_author = st.text_input("Enter Author Name",max_chars=50)
		blog_article = st.text_area("Enter Your Message",height=200)
		blog_post_date = st.date_input("Post Date")
		if st.button("Add"):
			add_data(blog_author,blog_title,blog_article,blog_post_date)
			st.success("Post::'{}' Saved".format(blog_title))


	elif choice == "Search":
		st.subheader("Search Articles")
		search_term = st.text_input("Enter Term")
		search_choice = st.radio("Field to Search",("title","author"))
		if st.button('Search'):
			if search_choice == "title":
				article_result = get_blog_by_title(search_term)
			elif search_choice =="author":
				article_result = get_blog_by_author(search_term)
			
			# Preview Articles
			for i in article_result:
				st.text("Reading Time:{} minutes".format(readingTime(str(i[2]))))
				# st.write(article_temp.format(i[1],i[0],i[3],i[2]),unsafe_allow_html=True)
				st.write(head_message_temp.format(i[1],i[0],i[3]),unsafe_allow_html=True)
				st.write(full_message_temp.format(i[2]),unsafe_allow_html=True)
			

	elif choice == "Manage Blog":
		st.subheader("Manage Blog")
		result = view_all_notes()
		clean_db = pd.DataFrame(result,columns=["Author","Title","Article","Date","Index"])
		st.dataframe(clean_db)
		unique_list = [i[0] for i in view_all_titles()]
		delete_by_title =  st.selectbox("Select Title",unique_list)
		if st.button("Delete"):
			delete_data(delete_by_title)
			st.warning("Deleted: '{}'".format(delete_by_title))

		if st.checkbox("Metrics"):
			new_df = clean_db
			new_df['Length'] = new_df['Article'].str.len() 


			st.dataframe(new_df)
			# st.dataframe(new_df['Author'].value_counts())
			st.subheader("Author Stats")
			new_df['Author'].value_counts().plot(kind='bar')
			st.pyplot()

			new_df['Author'].value_counts().plot.pie(autopct="%1.1f%%")
			st.pyplot()

		if st.checkbox("WordCloud"):
			# text = clean_db['Article'].iloc[0]
			st.subheader("Word Cloud")
			text = ', '.join(clean_db['Article'])
			# Create and generate a word cloud image:
			wordcloud = WordCloud().generate(text)

			# Display the generated image:
			plt.imshow(wordcloud, interpolation='bilinear')
			plt.axis("off")
			st.pyplot()

		if st.checkbox("BarH Plot"):
				st.subheader("Length of Articles")
				new_df = clean_db
				new_df['Length'] = new_df['Article'].str.len() 
				barh_plot = new_df.plot.barh(x='Author',y='Length',figsize=(10,10))
				st.write(barh_plot)
				st.pyplot()



if __name__ == '__main__':
	main()
```
---
- refactoring the database python code (db_fxn.py):
import psycopg2
from psycopg2.extras import RealDictCursor
from dotenv import load_dotenv
import os

# Load environment variables from .env file
load_dotenv()

DB_HOST = os.getenv("DB_HOST")
DB_NAME = os.getenv("DB_NAME")
DB_USER = os.getenv("DB_USER")
DB_PASS = os.getenv("DB_PASS")
def get_connection():
    return psycopg2.connect(host=DB_HOST, dbname=DB_NAME, user=DB_USER, password=DB_PASS, port="5432", sslmode="disable")

def create_table():
    with get_connection() as conn:
        with conn.cursor() as cur:
            cur.execute("""
                CREATE TABLE IF NOT EXISTS blogtable(
                    author TEXT,
                    title TEXT,
                    article TEXT,
                    postdate DATE
                )
            """)
            conn.commit()

def add_data(author, title, article, postdate):
    with get_connection() as conn:
        with conn.cursor() as cur:
            cur.execute("""
                INSERT INTO blogtable(author, title, article, postdate)
                VALUES (%s, %s, %s, %s)
            """, (author, title, article, postdate))
            conn.commit()

def view_all_notes():
    with get_connection() as conn:
        with conn.cursor() as cur:
            cur.execute("SELECT * FROM blogtable")
            return cur.fetchall()

def view_all_titles():
    with get_connection() as conn:
        with conn.cursor() as cur:
            cur.execute("SELECT DISTINCT title FROM blogtable")
            return cur.fetchall()

def get_blog_by_title(title):
    with get_connection() as conn:
        with conn.cursor() as cur:
            cur.execute("SELECT * FROM blogtable WHERE title=%s", (title,))
            return cur.fetchall()

def get_blog_by_author(author):
    with get_connection() as conn:
        with conn.cursor() as cur:
            cur.execute("SELECT * FROM blogtable WHERE author=%s", (author,))
            return cur.fetchall()

def get_blog_by_msg(article):
    with get_connection() as conn:
        with conn.cursor() as cur:
            cur.execute("SELECT * FROM blogtable WHERE article ILIKE %s", (f'%{article}%',))
            return cur.fetchall()

def edit_blog_author(author, new_author):
    with get_connection() as conn:
        with conn.cursor() as cur:
            cur.execute("UPDATE blogtable SET author=%s WHERE author=%s", (new_author, author))
            conn.commit()

def edit_blog_title(title, new_title):
    with get_connection() as conn:
        with conn.cursor() as cur:
            cur.execute("UPDATE blogtable SET title=%s WHERE title=%s", (new_title, title))
            conn.commit()

def edit_blog_article(article, new_article):
    with get_connection() as conn:
        with conn.cursor() as cur:
            cur.execute("UPDATE blogtable SET article=%s WHERE article=%s", (new_article, article))
            conn.commit()

def delete_data(title):
    with get_connection() as conn:
        with conn.cursor() as cur:
            cur.execute("DELETE FROM blogtable WHERE title=%s", (title,))
            conn.commit()
```
Imported data through env and changed the database from sqlite3 to postgresql.

---

- setting up database:
```bash
minikube start --memory=5120 --cpus=4 --disk-size=30g
```
<img width="960" alt="{46BB1D51-67E2-413B-86DE-90BC1EFECF7E}" src="https://github.com/user-attachments/assets/37799a3c-a368-46cb-9543-73ab98d40de8" />

```bash
kubectl create namespace postgres-operator
kubectl apply --server-side -f https://raw.githubusercontent.com/percona/percona-postgresql-operator/v2.6.0/deploy/bundle.yaml -n postgres-operator
```
<img width="944" alt="{528006D4-BDA3-4035-9BEF-F66462531935}" src="https://github.com/user-attachments/assets/095e3bb2-528f-4ba6-afe0-951501f128ca" />

```bash
kubectl apply -f https://raw.githubusercontent.com/percona/percona-postgresql-operator/v2.6.0/deploy/cr.yaml -n postgres-operator
```
<img width="873" alt="{2372585C-2AF0-421A-BD48-4C65D5E280D0}" src="https://github.com/user-attachments/assets/d1d8da22-69e7-49f6-afa1-c84c8cc67730" />
```bash
kubectl get pods -n postgres-operator
```
<img width="760" alt="{762CC190-8E14-4500-BCBC-DED4732BB677}" src="https://github.com/user-attachments/assets/27c18283-c3c6-42ed-8910-261dbcb45a9c" />
- waiting till rinning status
<img width="673" alt="{65BEC6C7-A276-4A11-A580-350CC904F536}" src="https://github.com/user-attachments/assets/c1407282-44de-45ce-989e-aecb063f11bd" />
<img width="630" alt="{965F4307-FE52-4C41-AC79-9646F1753740}" src="https://github.com/user-attachments/assets/3b59cae1-9ec7-4ea6-9f2d-2fd90586db8e" />

- checking for service ip:
```bash
 kubectl get svc -n postgres-operator
```
<img width="765" alt="{E94D0EF6-78F5-43ED-B6FC-C44B26722193}" src="https://github.com/user-attachments/assets/12050900-a7cd-47cf-9035-9fcfa75b7275" />

- applied the cr.yaml
```bash
kubectl apply -f https://raw.githubusercontent.com/percona/percona-postgresql-operator/main/deploy/cr.yaml
```
<img width="736" alt="{5C6EBCFA-DA7A-4E83-A79E-FAA0F1959A94}" src="https://github.com/user-attachments/assets/eaedb607-e10f-483c-ad47-191b0f4af9fd" />
- checking for secrets. decrypted the secret.
- decoded to base64 url and then to data.
<img width="526" alt="image" src="https://github.com/user-attachments/assets/f8d1431f-80ea-46e9-bef5-438c44c031b4" />

<img width="940" alt="{FA7D33EA-7AD1-4733-A78D-212E707B4AED}" src="https://github.com/user-attachments/assets/768593c5-d771-4e1d-a1b3-e75b605cd301" />

- connected to database.
- Issues: current user has no previllages, no superuser, localhost not communicating.
<img width="947" alt="{5DDCCC8F-1CD2-455B-87ED-06F6B17E0E2B}" src="https://github.com/user-attachments/assets/e80518d6-4e75-41c5-a53d-199334d3c57d" />

- created `minikube tunnel`
<img width="955" alt="{75F00580-D53D-41C0-A610-F675F894A999}" src="https://github.com/user-attachments/assets/70e6befa-a26c-4613-bee5-7d1190244417" />
**yet localhost was not able to communicate with user**

now making changes to cr.yaml
<img width="753" alt="{91B8778A-A604-43EF-AD97-32DBB0D9C986}" src="https://github.com/user-attachments/assets/b71b485d-bcfd-4d96-bf25-00d1945f4b50" />
- ADDING USERS
<img width="327" alt="{A3192AE3-2D22-466A-A028-001DF8DADA39}" src="https://github.com/user-attachments/assets/786e6d9a-b761-4b40-9c8f-837bea909098" />
- EXPOSING BY LOADBALANCERS
<img width="342" alt="{1162862B-0048-4517-8DAC-A4F1F72A7C50}" src="https://github.com/user-attachments/assets/28e3f8bf-7055-4c82-9e90-8cc4239b915b" />
<img width="583" alt="{1C6B190C-3EB2-4C83-B5A7-C7C92E23483D}" src="https://github.com/user-attachments/assets/471b3f95-9044-448e-b5d2-c624338e8c12" />
- pg_hba
<img width="374" alt="{22FE45B7-1413-4936-A234-98AE9333A66D}" src="https://github.com/user-attachments/assets/60fa7fe4-0a55-4353-a748-f46a1ec282dd" />
---
**working on localhost**
<img width="958" alt="{820CDD4F-1783-47EC-A8D1-2FEB87704F89}" src="https://github.com/user-attachments/assets/d8f16c9d-f7cd-4e53-9a66-f37030378321" />

---

