# NoSQL-Practice2

## Creating a backend for a blog using MongoDB (example with Python)

### Prerequisites:
- Python installed on your system
- MongoDB installed and running
  https://www.mongodb.com/try/download/community
  
- Basic knowledge of Flask and MongoDB operations
  https://flask.palletsprojects.com/
  

### Step 1: Set Up Your Environment
1. **Create a new directory** for your project and navigate into it.
2. **Initialize a Python virtual environment** to manage your dependencies:

    ```bash
    python -m venv venv
    source venv/bin/activate  # On Windows, use `venv\Scripts\activate`
    ```

3. **Install Flask and PyMongo**:

    ```bash
    pip install Flask pymongo
    ```

4. **Set Up Your Flask Application**
Create a new directory for your project, navigate into it, and create a Python file for your application, for example, `app.py`.

### Step 2: Initialize Your Flask Application
In `app.py`, set up a basic Flask application and initialize a connection to your MongoDB:

```python
from flask import Flask, request, jsonify, render_template
from pymongo import MongoClient

app = Flask(__name__)

# Connect to the MongoDB database
client = MongoClient('mongodb://localhost:27017/')
db = client.blog_db  # 'blog_db' is the name of our database

# Define your collections, for example, 'posts'
posts = db.posts

@app.route('/')
def home():
    return "Welcome to the Blog!"

if __name__ == '__main__':
    app.run(debug=True)
```

### Step 3: Define Routes for CRUD Operations

#### Create a Post
```python
@app.route('/post', methods=['POST'])
def create_post():
    title = request.json.get('title')
    content = request.json.get('content')
    if title and content:
        post_id = posts.insert_one({'title': title, 'content': content}).inserted_id
        return jsonify({'message': 'Post created successfully', 'id': str(post_id)})
    else:
        return jsonify({'message': 'Title and content required'}), 400
```

#### Read All Posts
```python
@app.route('/posts', methods=['GET'])
def get_posts():
    all_posts = list(posts.find({}, {'_id': 0, 'title': 1, 'content': 1}))
    return jsonify(all_posts)
```

#### Update a Post
Assuming each post has a unique identifier (`post_id`), you can define a route to update a post:

```python
@app.route('/post/<post_id>', methods=['PUT'])
def update_post(post_id):
    updated_data = request.json
    result = posts.update_one({'_id': post_id}, {'$set': updated_data})
    if result.modified_count > 0:
        return jsonify({'message': 'Post updated successfully'})
    else:
        return jsonify({'message': 'No changes made or post not found'})
```

#### Delete a Post
```python
@app.route('/post/<post_id>', methods=['DELETE'])
def delete_post(post_id):
    result = posts.delete_one({'_id': post_id})
    if result.deleted_count > 0:
        return jsonify({'message': 'Post deleted successfully'})
    else:
        return jsonify({'message': 'Post not found'}), 404
```

### Step 4: Run Your Flask Application
Save your `app.py` file and run your Flask application from the terminal:

```bash
python app.py
```

Your simple blog platform should now be up and running. You can use tools like Postman or cURL to test the CRUD endpoints, or you can extend the application by adding frontend templates with Flask's `render_template` to interact with these routes through a web interface.

### Conclusion
This mini-project provides a foundation for a blog platform using Flask and MongoDB. You can expand on this by adding user authentication, comments, categories, and more, depending on your requirements and creativity.

## Simple web pages (templates) to your Flask and MongoDB blog project 

For viewing the results, you'll create HTML templates that Flask can render. These templates will allow users to see a list of blog posts and a form to add new posts.

### Step 1: Set Up Your Template Directory
Inside your project directory, create a folder named `templates`. Flask will automatically look for templates in this directory.

### Step 2: Create Templates

#### `home.html` - Home Page with List of Posts
Create a file named `home.html` inside the `templates` folder. This template will display a list of blog posts.

```html
<!-- templates/home.html -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Blog Home</title>
</head>
<body>
    <h1>Blog Posts</h1>
    <a href="/create">Create New Post</a>
    <ul>
        {% for post in posts %}
            <li>
                <h2>{{ post.title }}</h2>
                <p>{{ post.content }}</p>
            </li>
        {% endfor %}
    </ul>
</body>
</html>
```

#### `create_post.html` - Create Post Page
Create another file named `create_post.html` in the `templates` folder for the form to create a new post.

```html
<!-- templates/create_post.html -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Create Post</title>
</head>
<body>
    <h1>Create a New Blog Post</h1>
    <form action="/post" method="post">
        <label for="title">Title:</label><br>
        <input type="text" id="title" name="title" required><br>

        <label for="content">Content:</label><br>
        <textarea id="content" name="content" required></textarea><br>

        <input type="submit" value="Submit">
    </form>
    <a href="/">Back to Home</a>
</body>
</html>
```

### Step 3: Update Flask Routes to Use Templates

#### Update the Home Route
Modify the home route to render the `home.html` template and pass the list of posts to it.

```python
@app.route('/')
def home():
    all_posts = list(posts.find({}, {'_id': 0}))
    return render_template('home.html', posts=all_posts)
```

#### Add a Route for the Create Post Page
Create a new route that renders the `create_post.html` template when navigating to `/create`.

```python
@app.route('/create')
def create():
    return render_template('create_post.html')
```

#### Update the Create Post Route
Adjust the route that handles post creation to redirect to the home page after a new post is submitted.

First, import `redirect` and `url_for` at the top of your `app.py`:

```python
from flask import Flask, request, jsonify, render_template, redirect, url_for
```

Then, modify the create post function:

```python
@app.route('/post', methods=['POST'])
def create_post():
    title = request.form.get('title')
    content = request.form.get('content')
    if title and content:
        posts.insert_one({'title': title, 'content': content})
        return redirect(url_for('home'))
    else:
        return render_template('create_post.html', message='Title and content required')
```

### Step 4: Run Your Flask Application
With these changes, your simple blog platform now has basic web pages to list and create blog posts. Run your Flask application:

```bash
python app.py
```

Navigate to `http://localhost:5000` in your web browser to see the home page with the list of blog posts, and go to `http://localhost:5000/create` to access the form for creating new posts.

### Conclusion
By adding these templates and updating your Flask routes, you've created a simple web interface for your blog platform. Users can now view a list of posts and add new posts through a basic web form. This setup provides a foundation that you can further enhance with more features, styling, and interactivity.

You have to add a few additional templates (web-pages) to provide all CRUD operations and update the appropriate  the functions.


