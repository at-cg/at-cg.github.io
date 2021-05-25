To edit and update your page:-
	Go to:
	tabs/{pageName.md}

To edit Home page:-
	Go to:
	_layouts/home.html
	(This page is written in html and it is because of Ruby and Jekyll requirement,
	 only .html home pages can be hosted on github, So that's the reason)
	NOTE: You can also write in markdown syntax here

For editing the page use markdown syntax and follow md_syntax.txt for it,
with addition of markdown you can use HTML and well.

To access the blog and my teams page:-
	Go to:
	_data/tabs.yml
	Uncomment the commented part

To create a new page:-
	1. Go to:
	2. tabs/    and make a new file with .md extention
	   and paste:- 

		---
		title: {title name}
		type: {filename}
		---
		<head>
		  <link
		    href="https://fonts.googleapis.com/css?family=Montserrat" rel="stylesheet"/>
		  <link rel="stylesheet" href="../../assets/css/main.css" />
		</head>
	
	3. Go to:
	4. _data/tabs.yml    
	   and paste this:-        name: "{filename title}"
				   icon: "fas fa-info"
				   path: tabs
				   url: {exact filename without .md}


To add a photo in a page:-
	Go to:
	tabs/assets/img    and upload your image
	in markdown syntax to float image to right like in projects page add:- <img class="image" style="float: right;" src="./../assets/img/{imgName.jpg}">

	To upload image without panning to write add:- <img src="./../assets/img/{imgName.jpg}">


Add <br> for a line break

To add buttons in publications just add this syntax below after your title and authors
It has Paper, Code, Poster, News buttons accordingly use whatever is necessary. 
href="#" is where you add your link by replacing # with your link

<button type="button" class="btn btn-outline-info btn-sm">
<a href="#" style="all: unset; color: inherit">Paper</a>
</button> 

<button type="button" class="btn btn-outline-secondary btn-sm">
<a href="#" style="all: unset; color: inherit">Code</a>
</button> 

<button type="button" class="btn btn-outline-info btn-sm">
<a href="#" style="all: unset; color: inherit">Poster</a>
</button>

<button type="button" class="btn btn-outline-success btn-sm">
<a href="#" style="all: unset; color: inherit">News</a>
</button> 

To add CSS [ Will not be required :) ]
	Go to:
	assets/css/main.css   and add you css file there
