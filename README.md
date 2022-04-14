# Python Live Project
After completing the Python course, I participated in a two week sprint. I was tasked with using Python within the Django framework to create a dynamic web application. I chose cartoons as my topic and I built a destinations website that allows a user to create, review, update and delete information about their favorite cartoon series. I used several functions in the views in order to retrieve data and display them on the applicable HTML template. Additionally, I connected an API and wrote a basic JSON response that allows the user to search any word in the Oxford Dictionary and view it's definition. I also used Beautiful Soup to scrape/parse data from other sites to display on my site.

## Table of Contents
- [Creating The Basic App](#Creating-The-Basic-App)
- [Creating The Model and Form](#Creating-The-Model-and-Form)
- [CRUD Functionality](#CRUD-Functionality)
- [Connect to API](#Connect-to-API)
- [Beautiful Soup](#Beautiful-Soup)
- [Conclusion](#Conclusion)

## Creating the Basic App
To start, I created a new application within the Django framework. I created base and home templates and then added function to the views in order for a homepage with a navbar to render. I then registered my URL's and linked my application to the main project home page.

![Home](/Images/Home.png)

## Creating the Model and Form
I created an object model class with a manager and defined it's attributes.

models.py:
```cs
    Genre_Choices = (
        ('comedy','Comedy'),
        ('anime', 'Anime'),
        ('horror','Horror'),
        ('sitcom', 'Sitcom'),
        ('musical', 'Musical'),
        ('kids', 'Kids'),
        ('slice of life','Slice of LIfe'),
        ('early animation','Early Animation'),
    )

    class Cartoon(models.Model):
        title = models.CharField(max_length=50)
        genre = models.CharField(max_length=20, choices=Genre_Choices, default='Comedy')
        network = models.CharField(max_length=30)
        premier_date = models.DateField()
        brief_description = models.TextField(max_length=1000, default="")

        Cartoons = models.Manager()

        def __str__(self):
            return self.title
```
I then created a model form that includes any inputs the user needs to make. 

forms.py:
```cs
    from django import forms
    from .models import Cartoon

    # creating our modelform
    class CartoonForm(forms.ModelForm):
        class Meta:
            model = Cartoon
            fields = "__all__"
```
I made a template page for the form and then created a views function that renders all cartoons in the database. This page only displays the title and premier date of the cartoon in the database. I created another view function that allows User's to click on a cartoon's title to bring them to another page that displays the additional details of that cartoon.

views.py:
```cs
    def DisplayCartoons(request):
        cartoon_list = Cartoon.Cartoons.all().order_by("premier_date")
        context = {'cartoon_list': cartoon_list}
        return render(request, 'Cartoons/Cartoons_list.html', context)
```
```cs
    def DisplayDetails(request, pk):
        item = get_object_or_404(Cartoon, pk=pk)
        context = {'item': item}
        return render(request, 'Cartoons/Cartoons_details.html', context)
```
* Insert GIF of displaying cartoon list/details
## CRUD Functionality
This view function saves the users new destination details to the database.
```cs
    def CreateCartoon(request):
        form = CartoonForm(data=request.POST or None)
        if request.method=='POST':
            if form.is_valid():
                form.save()
                return redirect('Cartoons_home')
        context = {'form': form}
        return render(request, "Cartoons/Cartoons_create.html", context)
```
This view function allows the user to edit/update a cartoons details.
```cs
    def UpdateItem(request, pk):
        item = Cartoon.Cartoons.get(pk=pk)
        form = CartoonForm(request.POST or None, instance=item)

        if form.is_valid():
            form.save()
            return redirect('Cartoons_list')

        context = {'item': item, 'form': form}
        return render(request, 'Cartoons/Cartoons_up_date.html', context)
```
This view function enables a user to delete trips from database.
```cs
    def DeleteItem(request, pk):
        context = {}
        item = get_object_or_404(Cartoon, pk=pk)

        if request.method == "POST":
            item.delete()
            return redirect("Cartoons_list")

        return render(request, "Cartoons/Cartoons_delete.html", context)
```
* Insert GIF of creating/updating/delete
## Connect to API
I created a new API template and rendered it with a view function. I then researched API documentation in order to connect the API and write a basic JSON response that allows users to search a word in the Oxford Dictionary and view it's definition. I added additional functionality that saves each word and definiton that was searched into the database. I then created a new  template for displaying the previously searched words/definitions.

views.py:
```cs
    def OxfordAPI(request):
        # set up api connection
        app_id='591386c7'
        app_key='ea768fd0e3d3a96ec8b39d08533c1f36'
        language='en-us'
        fields ='definitions'
        strictMatch ='false'
        # create empty list
        definition=[]
        if request.method=='POST':
            value=request.POST['word_id'].lower()
            # if input is blank it will display this message
            if value=="":
                messages.info(request, 'Please enter a search term')
            else:
                url ='https://od-api.oxforddictionaries.com:443/api/v2/entries/' + language + '/' + value + '?fields=' + fields + '&strictMatch=' + strictMatch;
                info=requests.get(url,headers={'app_id':app_id, 'app_key':app_key})
                oxford_info=info.json()
                result=oxford_info['results'][0]['lexicalEntries'][0]['entries'][0]['senses'][0]['definitions'][0]
                definition.append(result)

                save_definition = Definition.Definitions.create(
                    value=value,
                    definition=definition
                )
                save_definition.save()

            context={'value':value,'definition':definition}
            # if input is not blank then it will return the context
            return render(request, 'Cartoons/Cartoons_api.html', context)
        else:
            return render(request, 'Cartoons/Cartoons_api.html')
```
![Dictionary](/Images/Dictionary.png)

I then created a new object model class with a manager to our models.py for saving words/definitions.

models.py:
```cs
    class Definition(models.Model):
        value = models.CharField(max_length=50)
        definition = models.TextField(max_length=100)

        Definitions = models.Manager()

        def __str__(self):
            return self.value
```
Finally, I added a views function for rendering all the words/definitions in the database.

views.py:
```cs
    def DisplayDefinitions(request):
        definition_list = Definition.Definitions.all().order_by("value")
        context = {'definition_list': definition_list}
        return render(request, 'Cartoons/Cartoons_definitions.html', context)
```

![Definitions](/Images/Definitions.png)

## Beautiful Soup
Using the Python package Beautiful Soup, I parsed data from two websites to display a top ten animated series list and a top 100 animated movies list  on  two newly created templates that can be accessed from the nav bar and home page.

The below code scrapes the number and titles of the cartoons from indiewires website where they are ranking the best 
animated series of all time. There are over 60 cartoons in the list and about 10 are displayed per page. I selected the 
specific page that named the top 10 cartoons. The titles were all under "h3" tags in the html code.

views.py:
```cs
    def CartoonScrape(request):
        # Create empty list.
        top_cartoons = []
        # Set up BeautifulSoup.
        source = requests.get("https://www.indiewire.com/feature/best-animated-series-all-time-cartoons-anime-tv-1202021835/5/")
        bs = BeautifulSoup(source.content, 'html.parser')
        # Get all the h3 tags under the div 'entry-content' from source site.
        rankings = bs.find('div', class_='entry-content')
        rank = rankings.find_all('h3')
        # For loop through the h3 tags but in reverse order (because they are displayed reversed on the source page).
        for h3 in reversed(rank):
            titles = h3.text
            top_cartoons.append(titles)
        # Delete indexes 8-9 which are irrelevant h3 tags.
        del top_cartoons[8:10]

        context = {'top_cartoons': top_cartoons}
        return render(request, 'Cartoons/Cartoons_rankings.html', context)
```
![Cartoon Rankings](/Images/Rankings_Cartoons.png)

The below code scrapes the ranking/rating/titles of the top 100 animated movies on rotten tomatoes website.

views.py:
```cs
    def MovieScrape(request):
        # Create empty list.
        top_movies = []
        # Set up BeautifulSoup.
        source = requests.get("https://www.rottentomatoes.com/top/bestofrt/top_100_animation_movies/")
        bs = BeautifulSoup(source.content, 'html.parser')
        # Get all the "a" tags under the table 'table' from source site.
        rankings = bs.find('table', class_='table')
        rank = rankings.find_all('tr')
        # For loop that iterates through all the "a" tags.
        for tr in rank:
            titles = tr.text
            top_movies.append(titles)
        # Delete index 0, which is not needed.
        del top_movies[0]

        context = {'top_movies': top_movies}
        return render(request, 'Cartoons/Cartoons_movies.html', context)
```
![Movie Rankings](/Images/Rankings_Movies.png)

## Conclusion
The Python live project provided me with my first opportunity to utilize project methodologies and gain a detailed understanding of version control. I worked with other students within an Agile framework environment on the Microsoft Azure DevOps platform. I was able to make commits, merges and push/pulls in real time while being aware of how to minimize merge conflicts. I participated in daily standup meetings to discuss progress and roadblocks as well as a retrospective meeting upon completion. I really enjoyed this process and look forward to utilizing everything learned from this sprint in future projects!
