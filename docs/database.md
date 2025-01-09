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
