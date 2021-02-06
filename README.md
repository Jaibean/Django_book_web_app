# Django Web App
Introduction
===========================
This Web App was built during a two week sprint called "Python Live Project" as an assignment for The Tech Academy's Software Developer Boot Camp. Prior to the project I completed a month long Python course that introduced the language syntax, history, capabilities, and challenged me with small projects/exercises throughout. Upon completion, I moved on to the Live Project portion that utilized agile project managemnt methodoligies including SCRUM. I met with a group of developers and our SCRUM master that led daily stand ups. To organize this sprint, we utilized Azure DevOps which had been introduced to use during the bootcamp. Each developer chose their own topic and built their app at their own pace.  This project is the interactive website for managing one's collections of things related to various hobbies, as well as API and Data Scraped content for those hobbies. I chose books, because I have accumulated many over the years and wanted a way to see what I have and where they are located through an online application. My Web App was built using the Django Framework in PyCharm where I uesd the terminal to create a virtual environment and install the packages needed for the project. This project exercized my skills in version control, research capabilities as well as my understanding for the MTV design pattern. Not only do I feel more confident building projects with this organization, I am excited to do more paired programming in the future. Below are a break down of the stories I completed and details of where I ran into roadblocks. 


## Story 1 - Setting up the App 
-------------
Cloned master Project from Azure that is the base of each developers app. When project branches are completed and approved, they are then merged in to this master. 

Created a new app for the project, named LibraryApp
1) Created a new app using manage.py startapp
2) Registered app from within MainProject>MainProject>settings.py

        INSTALLED_APPS = [
            'django.contrib.admin',
            'django.contrib.auth',
            'django.contrib.contenttypes',
            'django.contrib.sessions',
            'django.contrib.messages',
            'django.contrib.staticfiles',
            'rest_framework',
            'bootstrap4',
            'crispy_forms',
            'LibraryApp',

3) Created base and home templates in a new template folder

        urlpatterns = [
            path('', views.lib_home, name='homeLib'),
            path('create/', views.create_library, name='create'),
            path('add/', views.add_book, name='add'),

4) Added function to views to render the home page

        def lib_home(request):
        form = AddBookForm(data=request.POST or None)
        if request.method == 'POST':
            pk = request.POST['library']  # populates drop down of existing accounts based off their foreign key
            return your_library(request, pk)
        content = {'form': form}
        return render(request, 'library/home.html', content)

5) Registered urls with MainApp and create urls.py for my app and homepage

        urlpatterns = [
            path('admin/', admin.site.urls),
            path('LibraryApp/', include('LibraryApp.urls')),

6) Linked app's home page to the project's home page (index.html) by adding an image link. Added image into static > images folder 

          <!-- BOOKS APP -->
             <a href="{% url 'books' %}" class="grid-item app-image">
                  <img src="{% static 'images/books.jpg' %}" alt="Collection of books">
                    <div class="fadedbox">
                        <div class="title text"> Books </div>
                    </div>
              </a>
              <!-- END BOOKS APP-->

7) Added minimal content and some basic styling to my base and home including, Navbar, Background, Title, and Footer


        Base.html template
        {% load static %}
        <!DOCTYPE html>
        <html lang="en">
            <head>
                <meta charset="UTF-8">
                <meta name="viewport" content="width=device-width, initial-scale=1">
                <link rel="stylesheet" href="{% static '/CSS/LibraryApp.css' %}">
                <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css">
                <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/font-awesome/4.7.0/css/font-awesome.min.css">
                <script src="https://ajax.googleapis.com/ajax/libs/jquery/3.3.1/jquery.min.js"></script>
                <script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.14.7/umd/popper.min.js"></script>
                <script src="https://cdnjs.cloudflare.com/ajax/libs/materialize/0.100.2/js/materialize.min.js"></script>
                <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/js/bootstrap.min.js"></script>
                <title>{% block title%}{% endblock %}</title>
            </head>
            <header>
                <div class="JavaScriptDropdown">
                    <div class="btn-group">
                        <button type="button" class="btn btn-secondary dropdown-toggle" data-toggle="dropdown">Menu</button>
                        <div class="dropdown-menu"><!--  links based on  registered URLs -->
                            <a href="{% url 'homeLib' %}" class="dropdown-item">Manage Libraries</a>
                            <a href="{% url 'create' %}" class="dropdown-item">Create New Library</a>
                            <a href="{% url 'add' %}" class="dropdown-item">Add New Book</a>
                            <a href="{% url 'Json' %}" class="dropdown-item">Search</a>
                        </div>
                    </div>
                </div>
            </header>
            <body id="body">
                <div class="library-container">
                    {% block content %}{% endblock %}
                    <br><br>
                </div>
                <section id="block-footer">
                    <p id="footer-right"><a href="#">Terms</a>|<a href="#">Privacy</a></p>
                    <p id="footer-left">&copy;2021 - Personal Library</p>
                </section>
            </body>
        </html>

# CRUD Functionality

## Story 2 - Creating models and model forms

1) Created a model for the creation of libraries and a model for adding books. The Library is a foreign key to the book model so the user can add books to their specific library. Book model has categories of book generes the user can choose from. Each have a model manager for accessing the database.  

        from django.db import models

        class Library(models.Model):
            first_name = models.CharField(max_length=50)
            last_name = models.CharField(max_length=50)

            Library = models.Manager()

            # Allows references to specific library to be returned as the owners name not the primary key

            def __str__(self):
                return self.first_name + " " + self.last_name + "'s Library"
            
        # CHOICES FOR GENRE IN ADD BOOK FORM
        GenreTypes = [('Spiritual', 'Spiritual'), ('Biography', 'Biography'), ('Cookbook', 'Cookbook'),
                      ('Poetry', 'Poetry'), ('Self-Help', 'Self-Help'), ('Sci-Fi', 'Sci-Fi'), ('Learning', 'Learning')]


        class AddBook(models.Model):
            library = models.ForeignKey(Library, on_delete=models.CASCADE)
            book_title = models.CharField(max_length=250)
            book_author = models.CharField(max_length=250,)
            genre = models.CharField(max_length=100, choices=GenreTypes)
            description = models.TextField(blank=True)

            addBook = models.Manager()


2) Created a model form for adding a library and a model form for adding a book. 

            from django.forms import ModelForm
            from .models import Library, AddBook

            class LibraryForm(ModelForm):
                class Meta:
                    model = Library
                    fields = '__all__'

            class AddBookForm(ModelForm):
                class Meta:
                    model = AddBook
                    fields = '__all__'
                    
3) Added a template for viewing each library that will populate the books in the given library 

        This is serving at the home.html template:
        {% extends "library/base.html" %}
        {% block title %}Home Library{% endblock %}
        {% block content %}
            <h3>Welcome to your Library Database</h3>
            <hr>
            <div class="text-center">
                <h5>Manage your existing library</h5>
                <form method="POST">
                    <label id="index-label">Choose library:</label>
                    {% csrf_token %}
                    {{ form.library }}  <!--form is the variable, library is the attribute -->
                    <button type="submit">Go to library</button>
                </form>
            </div>
            <hr>
            <div class="text-center">
                <h5>Add another library</h5>
                <button type="submit" class="new-library-button"><a href="{% url 'create' %}">Create Library</a></button>
                <br>
            </div>
            <section id="block-footer">
                    <p id="footer-right"><a href="#">Terms</a> | <a href="#">Privacy</a></p>
                    <p id="footer-left">&copy;2021 - Personal Library</p>
            </section>
        {% endblock %}


4) Created a template for creating a new book instance 

        {% extends 'library/base.html' %}
        {% block title%}Add New Book{% endblock %}
            {% block content %}
                    <h3>Add Book to Library</h3>
                    <div class="new-book-form">
                        <form method="POST">
                            {% csrf_token %}
                            {{form.as_p}}
                            <button id="new-book-button" type="submit" name="Save_Book">Add Book</button>
                        </form>
                        <br>
                    </div>
            {% endblock

5) Add a views function that renders the create page and utilizes the model form to save the collection item to the database.

        def create_library(request):
            form = LibraryForm(data=request.POST or None)
            if request.method == 'POST':
                if form.is_valid():
                    form.save()
                    return redirect('homeLib')
            content = {'form': form}  # creates a dictionary as a key value pair
            return render(request, 'library/Create_Library.html', content)


        def add_book(request):
            form = AddBookForm(data=request.POST or None)
            if request.method == 'POST':
                if form.is_valid():
                    form.save()
                return redirect('../')
            content = {'form': form}  # creates a dictionary as a key value pair
            return render(request, 'library/AddBook.html', content)
            
            URL Paths in urls.py file
            path('create/', views.create_library, name='create'),
            path('add/', views.add_book, name='add'),

## Story 3 & 4 - READ

Displaying information from the database on the "your library" page.

1) views function that display all the books in your library from passing through the primary key and sending to your library template

        URL path:
           path('<int:pk>/library/', views.your_library, name='libraryList'),
 
        Function:
        def your_library(request, pk):
            lib = get_object_or_404(Library, pk=pk)  # retrieving a single library based off the primary key
            books = AddBook.addBook.filter(library=pk)  # using AddBook model manager to filter through data of an instance
            content = {'library': lib, 'books': books}
            return render(request, 'library/YourLibrary.html', content)

        Template: 
        {% extends 'library/base.html' %}
        {% block title %}Your Library{% endblock %}
        {% block content %}
                    <div id="all-books-form">
                        <form>    <!-- pulling in selection options for users libraries -->
                            <br>
                            <label>Library:</label>
                            <input id="libraryholder" type="text" name="libraryHolder" value="{{ library }}" readonly>
                            <br><br>
                            <button type="button"><a href="{% url 'add' %}">Add Book</a></button>
                        </form>
                        <br><br>
                    </div>
                    <h2>Your Books</h2>

                    <table class="table table-responsive" id ="book-table">

                        <tr id="headerRow">
                            <th>Title</th>
                            <th>Author</th>
                            <th>Genre</th>
                            <th>Meta</th>
                        </tr>
                        {% for book in books %}
                        <tr>
                            <td class="center">{{ book.book_title }}</td>
                            <td class="center">{{ book.book_author }}</td>
                            <td class="center">{{ book.genre }}</td>
                            <td class ="center"><a href="{% url 'singleDetail' pk=book.id %}">More Details</a></td>
                        </tr>
                        {% endfor %}
                    </table>
                    <br><br>
        {% endblock %}



## Story 5 - UPDATE & DELETE

Allowing for edits and delete functions to be done from the details page of each specific book, bounce to confirmation before deleting.

1) template for viewing a single book instance that has an option to edit or delete(functionality for that to come)

        URL path:
         path('details/<int:pk>/', views.single_detail, name="singleDetail"),
   
   
        Fucntion: 
        
        def single_detail(request, pk):
            book_details = get_object_or_404(AddBook, pk=pk)  # retrieving a single book based off the primary key
            content = {'book_details': book_details}
            return render(request, 'library/details.html', content)
            
        Template: 
        
        {% extends 'library/base.html' %}
        {% block title %}Your Library{% endblock %}
        {% block content %}
                <h3>More Details</h3>
                        <table class="table table-responsive" id ="book-table">

                        <tr id="headerRow">
                            <th>Title</th>
                            <th>Author</th>
                            <th>Genre</th>
                            <th>Library</th>
                            <th>Description</th>
                            <th>Make Edits</th>
                        </tr>

                        <tr>
                            <td class="center">{{ book_details.book_title }}</td>
                            <td class="center">{{ book_details.book_author }}</td>
                            <td class="center">{{ book_details.genre }}</td>
                            <td class="center">{{ book_details.library }}</td>
                            <td class="center">{{ book_details.description }}</td>
                            <td class="center"><a href=" {% url 'update' pk=book_details.id %}">Make edits</a>
                                <br><a href="{% url 'delete' pk=book_details.id %}">Delete&nbsp;Entry</td>
                        </tr>

                    </table>
        {% endblock %}

3) template, url patterm, and view function for when user clicks on 'Make edits'. Using the AddBookForm to pupulate same forms user had when creating the book instance. This way they can make edits to those exact fields then override the information by hitting save. pk being passed through to know exactly what book is being updated

           URL Path:
               path('edit/<int:pk>/', views.update, name='update'),
               
            view function
                    def update(request, pk):
                    book_edit = AddBook.addBook.get(pk=pk)
                    form = AddBookForm(data=request.POST or None, instance=book_edit)
                    if request.method == 'POST':
                        if form.is_valid():
                            form.save()
                        return redirect('../../')
                    context = {'form': form}
                    return render(request, 'library/edit.html', context)
                    
             template:
             
                {% extends 'library/base.html' %}
                {% block title %}Your Library{% endblock %}
                {% block content %}
                        <h3>Edit Details</h3>
                                <div class="new-book-form">
                                                <form method="POST">
                                                        {% csrf_token %}
                                                        {{form.as_p}}
                                                        <button id="new-book-button" type="submit" name="Edit_Book">Save Edits</button>
                                                </form>
                                                <br>
                {% endblock %}



4) Option to delete book from the details page of that instance. using the pk to let the database know what is being removed

         delete url path:
            path('delete/<int:pk>/', views.delete, name='delete'),
        
        delete views function: 
            def delete(request, pk):
            book_delete = AddBook.addBook.get(pk=pk)
            # POST request
            if request.method == "POST":
                # confirm delete
                book_delete.delete()
                return redirect('../../')
            context = {'book_delete': book_delete}
            return render(request, 'library/deleteConfirm.html', context)
    
        delete confirm template:
                {% extends 'library/base.html' %}
                {% block title %}Your Library{% endblock %}
                {% block content %}
                        <h3>Confirm Delete</h3>

                            <form method="POST">
                                {% csrf_token %}
                                <h2>Are you sure you want to delete this book?</h2>
                                <button id="new-library-button" type="submit">Yes</button><br>
                                <button id="new-book-button" type="button"><a href="{% url 'singleDetail' pk=book_delete.id %}">Cancel</a></button>
                        </form>
                {% endblock %}


## Story 6 - API integration 

For this story, I chose to integrate Google Books API so a user can search for books on the web. I registered a key with Google console and integrated that key to create an endpoint for each search. Athough I did not fully complete this story to where I would have liked it to be, I took a great deal of time learning about APIs and integrating them into apps. I am going to come back to this when I have a better understanding of what I want the API connection to do for the user. 
        
         URL path:
         path('JSON/', views.google_books_api, name='Json'),
         
         
         template: 
        {% extends 'library/base.html' %}
        {% block title %}JSON{% endblock %}
        {% block content %}

         <h3>Google books</h3>
          <form method="POST">
              {% csrf_token %}
              {{ form.as_p }}
            <button type="Submit" value="Submit">search books</button>
          </form>

        <div class=results>
        
        </div>

        {% endblock %}
         
        views function:
            def google_books_api(request):
            # connecting the API query as an URI
            search_field = ''
            api = 'https://www.googleapis.com/books/v1/volumes'
            api_key = 'NOT SHOWING FOR SECURITY PURPOSES'
            form = GoogleBooksForm(data=request.POST or None)
            if request.method == 'POST':
                if form.is_valid():
                    search_field = form.cleaned_data.get("search")
            book_uri = api + '?q=' + search_field + '+intitle:' + 'keyes&key=' + api_key
            response = requests.get(book_uri)
            # print(response.status_code)
            # prints json details for search_field attribute
            # print(response.json())
            context = {'form': form, 'response': response}
            print(book_uri)
            return render(request, 'library/JSON.html', context)


# Skills Acquired

The skills I acquired through building this app were project management/version control via Azure DevOps, a better understanding of the MVT design pattern, how to research specific roadblocks, and how integrating an API can be achieved. I have not used Azure DevOps in previous work environments so it was cool to exlpore some of what is offers. I enjoyed looking at the boards to know what I had incoming, and if my previous story was approved or not. The Wiki section was a huge reference point for me in this project to make sure I was staying on track, accomplishing the tasks at hand, and not over asking the SCRUM master questions when the information was laid out. I reached roadblocks in aspects of getting my code to work the way I hoped which led to me improving my research skills. I joined stack overflow (finally, after just being a viewer for years) and created a fodler of bookmarked pages to reference later. I enjoyed seeing what Python has to offer through building this app and researched other ways to strengthen my skills with similar projects. I look forward to working more closely with programmers in the future to build an app that we are all working on. 

