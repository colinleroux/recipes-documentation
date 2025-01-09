
### **1. Install Flask-Login**
Ensure you have Flask-Login installed in your environment:
```bash
pip install flask-login
```

---

### **2. Set Up the `users` Table**
Define your `users` table for storing user credentials. Use `password_hash` for securely storing passwords.

```sql
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(100) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    email VARCHAR(255),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

### **3. Configure Flask-Login**
Set up Flask-Login in your application:

#### **User Model**
Create a user class that works with Flask-Login. It should include a `get_id` method and inherit `UserMixin`.

```python
from flask_login import UserMixin

class User(UserMixin):
    def __init__(self, id, username, password_hash):
        self.id = id
        self.username = username
        self.password_hash = password_hash
```

#### **Login Manager**
Initialize Flask-Login's `LoginManager`.

```python
from flask import Flask
from flask_login import LoginManager

app = Flask(__name__)
app.secret_key = 'your_secret_key'

login_manager = LoginManager()
login_manager.init_app(app)
login_manager.login_view = "login"  # Redirect here for @login_required
```

---

### **4. Implement User Loader**
Define a function to load a user from the database based on their ID.

```python
@login_manager.user_loader
def load_user(user_id):
    # Replace this with a query to fetch the user from your database
    user = query_user_from_database(user_id)
    if user:
        return User(user["id"], user["username"], user["password_hash"])
    return None
```

---

### **5. Add Routes for Login and Logout**
Create routes to handle user login and logout.

```python
from flask import request, redirect, url_for, flash, render_template
from flask_login import login_user, logout_user, login_required, current_user
from werkzeug.security import check_password_hash

@app.route("/login", methods=["GET", "POST"])
def login():
    if request.method == "POST":
        username = request.form["username"]
        password = request.form["password"]
        user = query_user_by_username(username)  # Fetch user from database
        if user and check_password_hash(user["password_hash"], password):
            user_obj = User(user["id"], user["username"], user["password_hash"])
            login_user(user_obj)
            return redirect(url_for("dashboard"))
        flash("Invalid username or password")
    return render_template("login.html")

@app.route("/logout")
@login_required
def logout():
    logout_user()
    return redirect(url_for("login"))
```

---

### **6. Protect Routes**
Use `@login_required` to secure routes.

```python
@app.route("/dashboard")
@login_required
def dashboard():
    return f"Welcome {current_user.username}!"
```

---

### **7. Test Your Setup**
1. Create a test user with a hashed password:
   ```python
   from werkzeug.security import generate_password_hash
   hashed_password = generate_password_hash("password123")
   print(hashed_password)
   ```

2. Insert the user into the database:
   ```sql
   INSERT INTO users (username, password_hash, email) VALUES ('test_user', '<hashed_password>', 'test@example.com');
   ```

3. Start your Flask app and test the login flow.

For a simple setup, you can implement login and logout functionality using Flask-Login without worrying about additional features like password reset or user registration. Here's how you can achieve this:

---

### **Basic Setup for Login and Logout**

#### **1. Database Table for Users**
Keep your `users` table simple:
```sql
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(100) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL
);
```

Manually add your user with a hashed password:
```python
from werkzeug.security import generate_password_hash

hashed_password = generate_password_hash("your_password")
print(hashed_password)  # Use this hash in your database
```

Insert the user into the database:
```sql
INSERT INTO users (username, password_hash) VALUES ('your_username', '<hashed_password>');
```

---

#### **2. Flask-Login Integration**

##### **User Model**
Create a `User` class compatible with Flask-Login:
```python
from flask_login import UserMixin

class User(UserMixin):
    def __init__(self, id, username, password_hash):
        self.id = id
        self.username = username
        self.password_hash = password_hash
```

##### **Login Manager**
Set up the `LoginManager` to manage sessions:
```python
from flask_login import LoginManager

login_manager = LoginManager()
login_manager.init_app(app)
login_manager.login_view = "login"  # Redirect to login page for @login_required
```

##### **User Loader**
Fetch the user from the database by ID:
```python
@login_manager.user_loader
def load_user(user_id):
    # Query the database to get user details
    user = query_user_from_database(user_id)  # Replace with your database query
    if user:
        return User(user["id"], user["username"], user["password_hash"])
    return None
```

---

#### **3. Routes for Login and Logout**

##### **Login Route**
Authenticate the user and start a session:
```python
from flask import request, redirect, url_for, flash, render_template
from flask_login import login_user
from werkzeug.security import check_password_hash

@app.route("/login", methods=["GET", "POST"])
def login():
    if request.method == "POST":
        username = request.form["username"]
        password = request.form["password"]

        # Query the user from the database
        user = query_user_by_username(username)  # Replace with your database query
        if user and check_password_hash(user["password_hash"], password):
            user_obj = User(user["id"], user["username"], user["password_hash"])
            login_user(user_obj)
            return redirect(url_for("dashboard"))
        flash("Invalid username or password")
    return render_template("login.html")  # Create a simple login template
```

##### **Logout Route**
End the user session:
```python
from flask_login import logout_user

@app.route("/logout")
def logout():
    logout_user()
    return redirect(url_for("login"))
```

---

#### **4. Protect Routes with `@login_required`**
Secure pages that should only be accessible to logged-in users:
```python
from flask_login import login_required, current_user

@app.route("/dashboard")
@login_required
def dashboard():
    return f"Welcome, {current_user.username}!"
```
