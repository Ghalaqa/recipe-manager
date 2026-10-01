#Meal manager
#import tkinter, os and csv
import os
import csv


#create the file names to store the users, recipes and meal plans
user_file = "user.csv"
recipes_file = "recipe.csv"
meal_file = "meal.csv"

def initialize_files():
    if not os.path.exists(user_file):
        with open(user_file, "w", newline="") as csvfile:
            writer = csv.writer(csvfile)
            writer.writerow(["username", "password"])

    if not os.path.exists(recipes_file):
        with open(recipes_file, "w", newline="") as csvfile:
            writer = csv.writer(csvfile)
            writer.writerow(["username", "recipe_name", "ingredients", "steps"])

    if not os.path.exists(meal_file):
        with open(meal_file, "w", newline="") as csvfile:
            writer = csv.writer(csvfile)
            writer.writerow(["username", "day", "recipe_name"])


#the meal plan will be created for everyday of the week
days= ['Mon',"Tues","Wed","Thurs","Fri","Sat","Sun"]

#user management functions
def new_user(username, password):
    #check to make sure the inputs are not empty
    if not username  or not password:
        return False, "Username and/or password cannot be empty"

    if not os.path.exists(user_file):
        with open(user_file, "w", newline="") as csvfile:
            writer = csv.writer(csvfile)
            writer.writerow(["username", "password"])

    #open the file to read and check for existing users
    #the following lines of code were adapted from codecademy and stackoverflow :https://www.codecademy.com/article/python-csv-file and https://stackoverflow.com/questions/46206297/how-do-i-write-to-a-csv-file-with-python
    with open(user_file,"r",newline='') as csvfile:
        reader = csv.DictReader(csvfile)
        for row in reader:
            if row["username"] == username:
                return False, "Username already exists"

    with open(user_file,"a",newline='') as csvfile:
        writer = csv.writer(csvfile)
        writer.writerow([username,password])

    return True, "User created successfully"
def login_user(username,password):
    with open(user_file,"r",newline='') as csvfile:
        reader = csv.DictReader(csvfile)
        for row in reader:
            if row["username"] == username and row["password"] == password:
                return True, "Login successfully"
    return False, "Login Unsuccessful"


#recipe management functions
#function load the recipes
def load_recipes(username):
    #create an empty list to store the recipes
    recipes = []
    with open(recipes_file,"r",newline='') as csvfile:
        reader = csv.DictReader(csvfile)

        for row in reader:
            #this will make sure that the recipes that will load with be specific for the current user
            if row["username"] == username:
                row["ingredients"]= [item.strip() for item in row["ingredients"].split(",")]
                row['steps']= [item.strip()for item in row["steps"].split(",")]
                recipes.append(row)

    return recipes

def save_recipes(username,recipe_name, ingredients, steps):
    #check that name, ingedients and steps are not blank
    if not recipe_name.strip() or not ingredients.strip() or not steps.strip():
        return False, "Recipe name, ingredients and steps cannot be empty"
    #load the current users recipes to avoid duplicates
    recipes= load_recipes(username)
    for recipe in recipes:
        if recipe["recipe_name"].lower() == recipe_name.lower():
            return False, "Recipe already exists"
    #open recipes file in append mode to add a new recipe
    with open(recipes_file,"a",newline='') as csvfile:
        writer = csv.writer(csvfile)
        #save username, recipe name, ingredients and steps
        writer.writerow([username,recipe_name,",".join(ingredients.split(",")), ",".join(steps.split(","))])

    return True, "Recipe saved successfully"

def update_recipes(username,old_name,new_name, ingredients, steps):
    updated=False
    #list to store all rows before rewriting the csv file
    all_rows=[]
    with open(recipes_file,"r",newline='') as csvfile:
        reader = csv.DictReader(csvfile)
        for row in reader:
            if row["username"] == username and row["recipe_name"].lower() == old_name.lower():
                row["recipe_name"] = new_name.strip()
                row["ingredients"] = ",".join([item.strip() for item in ingredients.split(",") if item.strip()])
                row["steps"] = ",".join([item.strip() for item in steps.split(",") if item.strip()])
                updated = True

            #store each row in the list
            all_rows.append(row)

    #write the whole recipes file with updated recipe -updated data
    # the following lines of code were adapted from codecademy and stackoverflow :https://www.codecademy.com/article/python-csv-file and https://stackoverflow.com/questions/46206297/how-do-i-write-to-a-csv-file-with-python
    with open(recipes_file,"w",newline='') as csvfile:
        fieldnames = ['username','recipe_name','ingredients','steps']
        writer = csv.DictWriter(csvfile,fieldnames=fieldnames)
        writer.writeheader()
        writer.writerows(all_rows)

    #update the meal plan according to the new recipe name
    if updated:
        return True, "Recipe updated successfully"
    else:
        return False,"Recipe not Found"

def delete_recipes(username,recipe_name):
    deleted=False

    all_rows=[]
    with open(recipes_file,"r",newline='') as csvfile:
        reader = csv.DictReader(csvfile)
        for row in reader:
            if row["username"] == username and row["recipe_name"] == recipe_name:
                deleted=True
            else:
                all_rows.append(row)
    #rewrite the file without the deleted recipe
    with open(recipes_file,"w",newline='') as csvfile:
        fieldnames = ['username','recipe_name','ingredients','steps']
        writer = csv.DictWriter(csvfile,fieldnames=fieldnames)
        writer.writeheader()
        writer.writerows(all_rows)

    if deleted:
        return True, "Recipe deleted successfully."

    else:
        return False, "Recipe not found"

def search_recipes(username,keyword):

    keyword=keyword.lower().strip()
    results=[] # to store search results

    #search the recipes using the keyword
    for recipe in load_recipes(username):
        if keyword in recipe["recipe_name"].lower() or keyword in recipe["ingredients"]:
            results.append(recipe)

    return results

#meal plan and shopping list features
meal_file ="meal.csv"
def create_plan(username):
    """
    Create a meal plan for the week (monday to sunday)
    :param username:
    :return:
    """
    recipes= load_recipes(username)
    if not recipes:
        return False, "No recipes saved. Please add the recipes first"
    print("Available recipes")
    for i, recipe in enumerate(recipes, start=1):
        print(f"{i}. {recipe['recipe_name']}")
    meal_plan=[]
    for day in days:
        while True:
            choice= input(f"Choose recipe number for {day}: ").strip()

            if not choice.isdigit():
                print("Enter a valid number")
                continue
            choice=int(choice)

            if  1<= choice <= len(recipes):
                recipe_name =recipes[choice -1]["recipe_name"]
                meal_plan.append([username,day,recipe_name])
                break
            else:
                print("Please choose a number from the recipe list")

    all_rows =[]

    with open(meal_file, "r", newline="") as csvfile:
        reader = csv.DictReader(csvfile)
        for row in reader:
            if row["username"] != username:
                all_rows.append(row)

    with open(meal_file, "w", newline="") as csvfile:
        fieldnames = ["username", "day", "recipe_name"]
        writer = csv.DictWriter(csvfile, fieldnames=fieldnames)
        writer.writeheader()

        for row in all_rows:
            writer.writerow(row)

        for row in meal_plan:
            writer.writerow({
                "username": row[0],
                "day": row[1],
                "recipe_name": row[2]
            })

        return True, "Meal plan created successfully."

def load_meal_plan(username):
    """
    This function will read the meal plan for this user
    :param username:
    :return:
    """
    meal_plan = {}

    with open(meal_file, "r", newline="") as csvfile:
        reader = csv.DictReader(csvfile)
        for row in reader:
            if row["username"] == username:
                meal_plan[row["day"]] = row["recipe_name"]

    if not meal_plan:
        print("No saved meal plan found.")
    else:
        print("\nWeekly Meal Plan")
        for day in days:
            if day in meal_plan:
                print(f"{day}: {meal_plan[day]}")
            else:
                print(f"{day}: No meal planned")

def shopping_list(username):
    """
    Generate the shopping list for this user
    """
    meal_plan = {}  # ✅ FIX: initialize dictionary

    # load the meal plan
    with open(meal_file, "r", newline="") as csvfile:
        reader = csv.DictReader(csvfile)
        for row in reader:
            if row["username"] == username:
                meal_plan[row["day"]] = row["recipe_name"]

    if not meal_plan:
        print("No saved meal plan found. Create a meal plan first")
        return

    # load the recipes
    recipes = load_recipes(username)

    # count ingredients
    shopping_list = {}

    for day in days:
        recipe_name = meal_plan.get(day)

        if recipe_name:
            for recipe in recipes:
                if recipe["recipe_name"].lower() == recipe_name.lower():
                    for ingredient in recipe["ingredients"]:
                        ingredient = ingredient.lower().strip()

                        if ingredient in shopping_list:
                            shopping_list[ingredient] += 1
                        else:
                            shopping_list[ingredient] = 1

    if not shopping_list:
        print("No ingredients found.")
    else:
        print("\nShopping list for the week")
        for ingredient, count in shopping_list.items():
            print(f"{ingredient}: {count}")

def main():
    initialize_files()
    current_user =None

    while True:
        if current_user is None:
            print("\n Welcome to the Recipe Manager \n")
            print("1. Register a new user")
            print("2. Login")
            print("3. Exit")

        choice = input("How would you like to proceed? : ")
        if choice == "1":
            while True:
                username = input("Enter your username : ").strip()
                password = input("Enter your password : ").strip()

                if not username or not password:
                    print("Username and password cannot be empty. Please try again.\n")
                else:
                    break

            success, message = new_user(username, password)
            print(message)

        elif choice == "2":
            username = input("Enter your username : ")
            password = input("Enter your password : ")
            success, message =login_user(username,password)
            print(message)

            if success:
                #current_user = username
                user(username)
                #current_user = None
                break
        elif choice == "3":
            print("Thank you for your time")
            break

        else:
            print("Please enter a valid option")

#current user function
def user(username):
    while True:
        print(f"Welcome {username}")
        print("1. Add a new recipe")
        print("2. View recipes")
        print("3. Search recipes")
        print("4. Update recipes")
        print("5. Delete recipes")
        print("6. Create meal plan")
        print("7. Load meal plan")
        print("8. Create a shopping list")
        print("9. Log out")

        choice = input("How would you like to proceed? : ")
        if choice == "1":
            recipe_name = input("Enter recipe name : ").strip()
            ingredients = input("Enter ingredients (comma separated): ").strip()
            steps = input("Enter steps : ").strip()

            success, message = save_recipes(username,recipe_name, ingredients, steps)
            print(message)

        elif choice == "2":
            recipes =load_recipes(username)

            if not recipes:
                print("Recipe not found. Please try again")
            else:
                for recipe in recipes:
                    print(f"Recipe name: {recipe['recipe_name']}")
                    print("Ingredients: ")
                    for ingredient in recipe["ingredients"]:
                        print(f"Ingredient: {ingredient}")
                    print("Steps: ")
                    for step in recipe["steps"]:
                        print(f"Step: {step}")

        elif choice == "3":
            keyword=input("Enter recipe name or ingredient to search: ").strip()
            results=search_recipes(username,keyword)

            if not results:
                print("Recipe not found. Please try again")

            else:
                for recipe in results:
                    print(f"Recipe name: {recipe['recipe_name']}")
                    print("Ingredients: ")
                    for ingredient in recipe["ingredients"]:
                        print(f"- {ingredient}")
                    print("Steps: ")
                    for i, step in enumerate(recipe["steps"], start=1):
                        print(f"{i}: {step}")

        elif choice == "4":
            old_name = input("Enter recipe name to update: ").strip()
            new_name = input("Enter new recipe name: ").strip()
            ingredients = input("Enter ingredients (comma separated): ").strip()
            steps = input("Enter steps (comma separated): ").strip()

            success, message = update_recipes(username,old_name,new_name, ingredients, steps)
            print(message)

        elif choice =="5":
            recipe_name = input("Enter recipe name to delete: ").strip()
            success, message = delete_recipes(username,recipe_name)
            print(message)

        elif choice == "6":
            success, message = create_plan(username)
            print(message)

        elif choice == "7":
            load_meal_plan(username)

        elif choice == "8":
            shopping_list(username)

        elif choice == "9":
            print(f"Thank you {username}")
            break

        else:
            print("Please enter a valid option")

if __name__ == "__main__":
    main()
