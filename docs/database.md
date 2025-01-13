### **Proposed Database Schema**

#### **1. Recipes Table**
Holds basic information about each recipe.
```sql
CREATE TABLE recipes (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    notes TEXT,
    image VARCHAR(255), -- Main or default image
    author VARCHAR(100),
    serves INT, -- Number of people the recipe serves
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

---

#### **2. Images Table**
Stores additional images associated with a recipe.
```sql
CREATE TABLE images (
    id INT AUTO_INCREMENT PRIMARY KEY,
    recipe_id INT NOT NULL,
    image_url VARCHAR(255) NOT NULL,
    image_description VARCHAR(255), -- Description for context
    image_order_number INT, -- Order of images
    FOREIGN KEY (recipe_id) REFERENCES recipes(id) ON DELETE CASCADE
);
```

---

#### **3. Instructions Table**
Manages instructions as ordered steps for each recipe.
```sql
CREATE TABLE instructions (
    id INT AUTO_INCREMENT PRIMARY KEY,
    recipe_id INT NOT NULL,
    instruction_order INT NOT NULL, -- Order of the step
    instruction_step TEXT NOT NULL, -- The actual instruction
    FOREIGN KEY (recipe_id) REFERENCES recipes(id) ON DELETE CASCADE
);
```

---

#### **4. Ingredients Table**
Stores ingredients for recipes.
```sql
CREATE TABLE ingredients (
    id INT AUTO_INCREMENT PRIMARY KEY,
    recipe_id INT NOT NULL,
    ingredient_detail VARCHAR(255) NOT NULL, -- "a pinch of salt", "3 large eggs"
    FOREIGN KEY (recipe_id) REFERENCES recipes(id) ON DELETE CASCADE
);
```

---

#### **5. Users Table (Optional)**
If you decide to add user management later:
```sql
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(100) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    email VARCHAR(255),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```
To add a `categories` table and link it to the `recipes` table, we can use a foreign key relationship. Here’s how we can amend the database structure:

---

### **Updated Database Schema**

#### **1. Categories Table**
This table will store the categories:
```sql
CREATE TABLE categories (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL UNIQUE
);
```

#### **2. Modify Recipes Table**
Add a foreign key to link a recipe to a category:
```sql
ALTER TABLE recipes ADD COLUMN category_id INT, ADD FOREIGN KEY (category_id) REFERENCES categories(id);
```

---

### **Relationship Details**
1. **One-to-Many Relationship**:
   - Each category can have multiple recipes.
   - Each recipe belongs to a single category (optional, as some recipes may not fit a category initially).

2. **Default Behavior**:
   - Set `category_id` as nullable (`NULL`) for recipes that don’t belong to a category.

---

### **Seeding Categories**
Use a seeder script to populate categories:
```python
from app import db
from app.models import Category

def seed_categories():
    categories = ["Desserts", "Main Courses", "Appetizers", "Drinks", "Snacks"]
    for category_name in categories:
        category = Category(name=category_name)
        db.session.add(category)
    db.session.commit()

if __name__ == "__main__":
    seed_categories()
```



### **Using Categories in Queries**
1. **Query Recipes by Category**:
   ```python
   category = Category.query.filter_by(name="Desserts").first()
   recipes = Recipe.query.filter_by(category_id=category.id).all()
   ```

2. **Include Category in Recipe Forms**:
   Add a dropdown to select the category when creating or editing a recipe.

## Database Structure
```mermaid
erDiagram
    USERS {
        INT id
        VARCHAR username
        VARCHAR password_hash
        VARCHAR email
        TIMESTAMP created_at
    }

    RECIPES {
        INT id
        VARCHAR name
        TEXT notes
        VARCHAR image
        VARCHAR author
        INT category_id
        INT serves
        BOOLEAN is_favorite
    }

    IMAGES {
        INT id
        INT recipe_id
        VARCHAR image
        VARCHAR image_description
        INT image_order_number
    }

    INSTRUCTIONS {
        INT id
        INT recipe_id
        INT instruction_id
        INT instruction_order
        TEXT instruction_step
    }

    INGREDIENTS {
        INT id
        INT recipe_id
        TEXT quantity
        VARCHAR ingredient_name
    }

    CATEGORIES {
        INT id
        VARCHAR name
    }

    RECIPES ||--o{ IMAGES : "has"
    RECIPES ||--o{ INSTRUCTIONS : "has"
    RECIPES ||--o{ INGREDIENTS : "has"
    CATEGORIES ||--o{ RECIPES : "categorizes"
```

## SQAlchemy

1. **Install Required Packages**
   Ensure you have the `mysql-connector-python` or `pymysql` library installed to work with MySQL. Run one of the following commands:

   ```bash
   pip install flask-mysql
   pip install pymysql
   ```

   Or, if you prefer `mysqlclient` (a C-based MySQL library):

   ```bash
   pip install mysqlclient
   ```

2. **Update Your Configuration**
   Modify the `SQLALCHEMY_DATABASE_URI` to use a MySQL connection string.

3. **Code with MySQL Configuration**
   Here’s the updated code for MySQL:

   ```python
   import os
   from flask import Flask
   from flask_sqlalchemy import SQLAlchemy
   from sqlalchemy.sql import func

   # App and Database Setup
   app = Flask(__name__)
   app.config['SQLALCHEMY_DATABASE_URI'] = (
       "mysql+pymysql://<username>:<password>@<hostname>/<database_name>"
   )
   app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False

   db = SQLAlchemy(app)

   # User Model
   class User(db.Model):
       __tablename__ = 'users'
       id = db.Column(db.Integer, primary_key=True)
       username = db.Column(db.String(100), nullable=False, unique=True)
       password_hash = db.Column(db.String(255), nullable=False)
       email = db.Column(db.String(255), unique=True)
       created_at = db.Column(db.DateTime(timezone=True), server_default=func.now())

       def __repr__(self):
           return f'<User {self.username}>'

   # Category Model
   class Category(db.Model):
       __tablename__ = 'categories'
       id = db.Column(db.Integer, primary_key=True)
       name = db.Column(db.String(100), nullable=False, unique=True)
       recipes = db.relationship('Recipe', back_populates='category')

       def __repr__(self):
           return f'<Category {self.name}>'

   # Recipe Model
   class Recipe(db.Model):
       __tablename__ = 'recipes'
       id = db.Column(db.Integer, primary_key=True)
       name = db.Column(db.String(255), nullable=False)
       notes = db.Column(db.Text)
       detail = db.Column(db.Text)
       image = db.Column(db.String(255))
       author = db.Column(db.String(100), nullable=False)
       category_id = db.Column(db.Integer, db.ForeignKey('categories.id'), nullable=True)
       serves = db.Column(db.Integer, nullable=True)
       is_favorite = db.Column(db.Boolean, default=False)
       created_at = db.Column(db.DateTime(timezone=True), server_default=func.now())

       category = db.relationship('Category', back_populates='recipes')
       images = db.relationship('Image', back_populates='recipe', cascade='all, delete')
       instructions = db.relationship('Instruction', back_populates='recipe', cascade='all, delete')
       ingredients = db.relationship('Ingredient', back_populates='recipe', cascade='all, delete')

       def __repr__(self):
           return f'<Recipe {self.name}>'

   # Image Model
   class Image(db.Model):
       __tablename__ = 'images'
       id = db.Column(db.Integer, primary_key=True)
       recipe_id = db.Column(db.Integer, db.ForeignKey('recipes.id'), nullable=False)
       image = db.Column(db.String(255), nullable=False)
       image_description = db.Column(db.String(255))
       image_order_number = db.Column(db.Integer, nullable=True)

       recipe = db.relationship('Recipe', back_populates='images')

       def __repr__(self):
           return f'<Image for Recipe {self.recipe_id}>'

   # Instruction Model
   class Instruction(db.Model):
       __tablename__ = 'instructions'
       id = db.Column(db.Integer, primary_key=True)
       recipe_id = db.Column(db.Integer, db.ForeignKey('recipes.id'), nullable=False)
       instruction_order = db.Column(db.Integer, nullable=False)
       instruction_step = db.Column(db.Text, nullable=False)

       recipe = db.relationship('Recipe', back_populates='instructions')

       def __repr__(self):
           return f'<Instruction {self.instruction_order} for Recipe {self.recipe_id}>'

   # Ingredient Model
   class Ingredient(db.Model):
       __tablename__ = 'ingredients'
       id = db.Column(db.Integer, primary_key=True)
       recipe_id = db.Column(db.Integer, db.ForeignKey('recipes.id'), nullable=False)
       quantity = db.Column(db.String(100), nullable=False)
       ingredient_name = db.Column(db.String(255), nullable=False)

       recipe = db.relationship('Recipe', back_populates='ingredients')

       def __repr__(self):
           return f'<Ingredient {self.ingredient_name} for Recipe {self.recipe_id}>'

   # Initialize the database
   if __name__ == '__main__':
       with app.app_context():
           db.create_all()
   ```

---

### **Notes**
1. Replace `<username>`, `<password>`, `<hostname>`, and `<database_name>` in the `SQLALCHEMY_DATABASE_URI` with your MySQL credentials and database name.
   - Example:
     ```python
     app.config['SQLALCHEMY_DATABASE_URI'] = "mysql+pymysql://root:password@localhost/recipe_app"
     ```

2. **Create the Database**:
   - Log in to MySQL and create the database before running the app:
     ```sql
     CREATE DATABASE recipe_app;
     ```

3. **Test the Setup**:
   - Run the Flask app locally and ensure the tables are created in your MySQL database.
   - Use sample data to test the relationships.

4. **PythonAnywhere**:
   - Update the `SQLALCHEMY_DATABASE_URI` for your PythonAnywhere database credentials when deploying.

