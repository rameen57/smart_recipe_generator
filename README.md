# smart_recipe_generator
import streamlit as st
import os
import requests
from dotenv import load_dotenv

# Load environment variables
load_dotenv()

# Get API keys and URLs from .env file
GROQ_API_KEY = os.getenv("GROQ_API_KEY")
GROQ_API_URL = os.getenv("GROQ_API_URL")

# File paths for storing recipes and recipe names
RECIPES_FILE = "recipes.txt"
RECIPES_NAME_FILE = "recipes_name.txt"

# Streamlit UI
st.title('🌟 Smart Recipe Generator 🌟')
st.write("Welcome! Let's generate some delicious recipes. 🥘")

# User input for ingredients
ingredients = st.text_input("Add ingredients (comma-separated): 🥦🍅🥩")

# User input for selecting cuisine
cuisine_options = ["Italian", "Indian", "Chinese", "Mexican", "American"]
cuisine = st.selectbox("Select Cuisine: 🌍", cuisine_options)

# Initialize session state for generated recipe
if 'generated_recipe' not in st.session_state:
    st.session_state.generated_recipe = None

# Button to find recipes
if st.button("Find Recipes 🔍"):
    if ingredients:
        # Prepare the request body
        messages = [
            {"role": "user", "content": f"Generate a recipe using the following ingredients: {ingredients} and cuisine: {cuisine}."}
        ]
        payload = {
            "model": "gemma2-9b-it",  # Choose the appropriate model ID from your GROQ documentation
            "messages": messages
        }

        # Set the headers for the request
        headers = {
            "Authorization": f"Bearer {GROQ_API_KEY}",
            "Content-Type": "application/json"
        }

        # Make the API request
        response = requests.post(GROQ_API_URL, json=payload, headers=headers)

        # Check the response status
        if response.status_code == 200:
            # Extract recipe from response
            recipe_data = response.json()
            st.session_state.generated_recipe = recipe_data.get('choices')[0].get('message').get('content')

            # Display the fetched recipe
            st.success("Here's a delicious recipe for you! 🍽️")
            st.write(st.session_state.generated_recipe)
        else:
            st.error(f"⚠️ Failed to fetch recipes. Status code: {response.status_code}")
            st.write("⚠️ No recipes found.")
    else:
        st.warning("Please enter some ingredients to find recipes.")

# Button to save the recipe
if st.button("Save Recipe 💾"):
    if st.session_state.generated_recipe:
        recipe_name = st.session_state.generated_recipe.split('\n')[0]  # Get the first line as the recipe name

        # Save recipe name to recipes_name.txt with UTF-8 encoding
        with open(RECIPES_NAME_FILE, "a", encoding="utf-8") as name_file:
            name_file.write(recipe_name + "\n")

        # Save full recipe to recipes.txt with a clear format and UTF-8 encoding
        with open(RECIPES_FILE, "a", encoding="utf-8") as recipe_file:
            recipe_file.write(st.session_state.generated_recipe.strip() + "\n\n")  # Ensure recipes are separated by a newline

        st.success(f"Recipe '{recipe_name}' saved successfully! 🎉")
    else:
        st.warning("No recipe to save. Please generate a recipe first.")

# Function to delete a recipe from both files
def delete_recipe(selected_recipe_name):
    # Read all recipes from recipes.txt
    with open(RECIPES_FILE, "r", encoding="utf-8") as file:
        recipes = file.read().strip().split("\n\n")  # Split the file into individual recipes

    # Filter out the recipe to be deleted
    new_recipes = [recipe for recipe in recipes if not recipe.startswith(selected_recipe_name)]

    # Write the updated recipes back to recipes.txt with UTF-8 encoding
    with open(RECIPES_FILE, "w", encoding="utf-8") as file:
        file.write("\n\n".join(new_recipes).strip() + "\n\n")  # Ensure recipes are separated by double newline

    # Remove the recipe name from recipes_name.txt
    with open(RECIPES_NAME_FILE, "r", encoding="utf-8") as name_file:
        recipe_names = name_file.readlines()

    # Filter out the selected recipe name
    updated_names = [name.strip() for name in recipe_names if name.strip() != selected_recipe_name]

    # Write the updated names back to recipes_name.txt with UTF-8 encoding
    with open(RECIPES_NAME_FILE, "w", encoding="utf-8") as name_file:
        for name in updated_names:
            name_file.write(name + "\n")

    st.success(f"Recipe '{selected_recipe_name}' deleted successfully! ❌")

# Display saved recipes for deletion
st.write("Your Saved Recipes 📚")

# Load the recipe names from recipes_name.txt
if os.path.exists(RECIPES_NAME_FILE):
    with open(RECIPES_NAME_FILE, "r", encoding="utf-8") as name_file:
        saved_recipe_names = [name.strip() for name in name_file.readlines()]

    if saved_recipe_names:
        # Let the user select a recipe to delete
        selected_recipe = st.selectbox("Select a recipe to delete:", saved_recipe_names)
