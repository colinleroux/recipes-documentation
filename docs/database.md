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
        INT id PK
        VARCHAR username UNIQUE
        VARCHAR password_hash
        VARCHAR email
        TIMESTAMP created_at
    }

    RECIPES {
        INT id PK
        VARCHAR name
        TEXT notes
        VARCHAR image
        VARCHAR author
        INT category_id FK
        INT serves
        BOOLEAN is_favorite
    }

    IMAGES {
        INT id PK
        INT recipe_id FK
        VARCHAR image
        VARCHAR image_description
        INT image_order_number
    }

    INSTRUCTIONS {
        INT id PK
        INT recipe_id FK
        INT instruction_id
        INT instruction_order
        TEXT instruction_step
    }

    INGREDIENTS {
        INT id PK
        INT recipe_id FK
        TEXT quantity
        VARCHAR ingredient_name
    }

    CATEGORIES {
        INT id PK
        VARCHAR name UNIQUE
    }

    RECIPES ||--o{ IMAGES : "has"
    RECIPES ||--o{ INSTRUCTIONS : "has"
    RECIPES ||--o{ INGREDIENTS : "has"
    CATEGORIES ||--o{ RECIPES : "has"
```