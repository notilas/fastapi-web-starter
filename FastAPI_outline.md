# FastAPI



* Structure

```
├── app
│   ├── library
│   │   └── helpers.py
│   ├── main.py
│   ├── pages
│   │   ├── about.md
│   │   ├── contact.md
│   │   ├── home.md
│   │   ├── info.md
│   │   └── portfolio.md
│   └── routers
│       ├── accordion.py
│       ├── twoforms.py
│       └── unsplash.py
├── static
│   ├── css
│   │   ├── mystyle.css
│   │   └── style3.css
│   ├── images
│   │   ├── favicon.ico
│   │   └── logo.svg
│   └── js
│       └── index.js
├── templates
│   ├── accordion.html
│   ├── base.html
│   ├── include
│   │   ├── sidebar.html
│   │   └── topnav.html
│   ├── page.html
│   ├── twoforms.html
│   └── unsplash.html
```

## APP - `main.py`

```python
from .library.helpers import *      											# library (python files)
from app.routers import twoforms, unsplash, accordion 		# router functions

templates = Jinja2Templates(directory="templates")				# Templates

app = FastAPI()
app.mount("/static", StaticFiles(directory="static"), name="static")  # Sattic

app.include_router(unsplash.router)
app.include_router(twoforms.router)
app.include_router(accordion.router)

# Path handlers : Path --> return HTML pages
@app.get("/", response_class=HTMLResponse)
async def home(request: Request):
    data = openfile("home.md")     # convert MD to HTML 
    return templates.TemplateResponse("page.html", {"request": request, "data": data})

@app.get("/page/{page_name}", response_class=HTMLResponse)
async def show_page(request: Request, page_name: str):
    data = openfile(page_name+".md")
    return templates.TemplateResponse("page.html", {"request": request, "data": data})
```

## Templates -- `html`

* `base.html`

```html
   <!-- Bootstrap 5 CSS CDN -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-1BmE4kWBq78iYhFldvKuhfTAU6auU8tT94WrHftjDbrCEXSU1oBoqyl2QvZ6jIW3" crossorigin="anonymous">
    <!-- Custom CSS -->
    <link href="{{ url_for('static', path='/css/style3.css') }}" rel="stylesheet">
    <link href="{{ url_for('static', path='/css/mystyle.css') }}" rel="stylesheet">


    <div class="wrapper">
        {% include 'include/sidebar.html' %}				<!-- sidebar -->
        <!-- Page Content  -->
        <div id="content">
            {% include 'include/topnav.html' %}
            {% block page_content %}
            {% endblock %}
        </div>
    </div>
    <div class="overlay"></div>


```

* `page.html`

```html
{% extends "base.html" %}

{% block title %}FastAPI Starter{% endblock %}  <!--BLOCK TITLE-->
{% block head %}
{{ super() }}
{% endblock %}

{% block page_content %}												 <!--BLOCK CONTENT-->
{% include 'include/sidebar.html' %}
<main role="main" class="container">
    <div class="row">
        <div class="col">
            {{data.text|safe}}
        </div>
    </div>
</main><!-- /.container -->
{{data.page}}
{% endblock %}

{% block scripts %}																<!--BLOCK SCRIPT-->
{{ super() }}
{% endblock %}

```

* `include - sidebar.html`

  ```html
  <li class="dropdown">
      <a href="#"class="dropdown-toggle" id="dropdownMenuButton1"  data-bs-toggle="dropdown" aria-expanded="false">Pages</a>
      <ul class="dropdown-menu" id="dropdownMenuButton1">
              <a href="/page/about" class="dropdown-item">About</a>
              <a href="/page/info"class="dropdown-item">Information</a>
      <a href="/page/portfolio">Portfolio</a>
      <a href="/page/contact">Contact</a>
  
  ```

  

* `include - topnav.html`

  ```html
  <nav class="navbar navbar-expand-lg navbar-light bg-light">
      <div class="container-fluid">
          <button type="button" id="sidebarCollapse" class="btn btn-info">
              <i class="fas fa-align-left"></i>
          <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarSupportedContent" aria-controls="navbarSupportedContent" aria-expanded="false" aria-label="Toggle navigation">
              <span class="navbar-toggler-icon"></span>
  
          <div class="collapse navbar-collapse" id="navbarSupportedContent">
              <ul class="navbar-nav me-auto mb-2 mb-lg-0">
                  <li class="nav-item {{'active' if active_page == 'unsplash' }}">
                      <a class="nav-link" href="/unsplash">Unsplash</a>
                  <li class="nav-item {{'active' if active_page == 'twoforms' }}">
                      <a class="nav-link" href="/twoforms">Two forms</a>
                  <li class="nav-item {{'active' if active_page == 'accordion' }}">
                      <a class="nav-link" href="/accordion">Accordion</a>
  ```

  

`openfile("home.md")` --> `\library\helper.py`

```python		
def openfile(filename):
    filepath = os.path.join("app/pages/", filename)
    with open(filepath, "r", encoding="utf-8") as input_file:
        text = input_file.read()

    html = markdown.markdown(text)  # ---> convert MK to HTML
    data = { "text": html}
    return data
```



