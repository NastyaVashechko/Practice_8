```
from flask import Flask, jsonify, request, render_template, redirect, url_for
from pymongo import MongoClient
from bson import ObjectId

app = Flask(__name__)

client = MongoClient('mongodb://localhost:27017/')
db = client.blog_database

@app.route('/')
def home():
    all_posts = list(db.posts.find({}, {'title': 1, 'content': 1}))
    return render_template('home.html', posts=all_posts)

@app.route('/create', methods=['GET'])
def show_create_post():
    return render_template('create_post.html')

@app.route('/create', methods=['POST'])
def create_post():
    title = request.form.get('title')
    content = request.form.get('content')
    if title and content:
        result = db.posts.insert_one({'title': title, 'content': content})
        new_post_id = str(result.inserted_id)
        return redirect(url_for('home'))
    else:
        return render_template('create_post.html', message='Title and content required')

@app.route('/edit/<post_id>', methods=['GET'])
def show_edit_post(post_id):
    post = db.posts.find_one({'_id': ObjectId(post_id)})
    return render_template('edit_post.html', post=post)

@app.route('/edit/<post_id>', methods=['POST'])
def edit_post(post_id):
    title = request.form.get('title')
    content = request.form.get('content')
    if title and content:
        db.posts.update_one({'_id': ObjectId(post_id)}, {'$set': {'title': title, 'content': content}})
        return redirect(url_for('home'))
    else:
        post = db.posts.find_one({'_id': ObjectId(post_id)})
        return render_template('edit_post.html', post=post, message='Title and content required')

@app.route('/posts/<post_id>', methods=['DELETE'])
def delete_post(post_id):
    db.posts.delete_one({'_id': ObjectId(post_id)})
    return jsonify({'message': 'Post deleted successfully!'})

if __name__ == '__main__':
    app.run(debug=True)
```
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Blog Home</title>
    <style>
        body {
            background-color: rgb(181, 173, 234);
            font-family: 'Kode Mono', sans-serif;
        }
    </style>
</head>
<body>
    <h1 style="text-align: center;">Blog Posts</h1>
    <a href="/create">Create New Post</a> 
    <ul>
        {% for post in posts %}
            <li>
                <h2>{{ post.title }}</h2>
                <p>{{ post.content }}</p>
                <a href="/edit/{{ post._id }}">Edit</a>
                <button onclick="deletePost('{{ post._id }}')">Delete</button>
            </li>
        {% endfor %}
    </ul>
    <script>
        function deletePost(postId) {
            if (confirm("Are you sure you want to delete this post?")) {
                fetch(`/posts/${postId}`, {
                    method: 'DELETE'
                })
                .then(response => {
                    if (!response.ok) {
                        throw new Error('Network response was not ok');
                    }
                    return response.json();
                })
                .then(data => {
                    alert(data.message);
                    location.reload();
                })
                .catch(error => {
                    console.error('There was an error!', error);
                });
            }
        }
    </script>
</body>
</html>
```
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Create Post</title>
    <style>
        body {
            background-color: rgb(181, 173, 234);
            font-family: 'Kode Mono', sans-serif;
        }
    </style>
</head>
<body>
    <h1 style="text-align: center;">Create a New Blog Post</h1>
    <form action="/create" method="post">
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
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Edit Post</title>
    <style>
        body {
            background-color: rgb(181, 173, 234);
            font-family: 'Kode Mono', sans-serif;
        }
    </style>
</head>
<body>
    <h1 style="text-align: center;">Edit Blog Post</h1>
    <form action="/edit/{{ post._id }}" method="post">
        <label for="title">Title:</label><br>
        <input type="text" id="title" name="title" value="{{ post.title }}" required><br>

        <label for="content">Content:</label><br>
        <textarea id="content" name="content" required>{{ post.content }}</textarea><br>

        <input type="submit" value="Update">
    </form>
    <a href="/">Back to Home</a>
</body>
</html>
```
