description: seed random dummy expenses for an existing user
allowed tools : Read, Bash(python3:*)

Arguments (positional, space-separated): $1 = user_id, $2 = count, $3 = month (1-12, current year)

Read database/db.py to understand the expenses table schema, the CATEGORIES list, and the get_db() helper.

Then write and run a python script using bash that:
1. Looks up the user by id ($1) via get_db(); if not found, print an error and stop.
2. Generates $2 expenses dated in month $3 of the current year, each with:
   - category: randomly chosen from CATEGORIES in db.py
   - description: a realistic description matching the category (e.g. Food -> "Groceries", "Restaurant", "Zomato order"; Transport -> "Auto fare", "Petrol", "Metro card recharge"; Bills -> "Electricity bill", "Wifi bill", "Water bill"; Health -> "Pharmacy", "Doctor visit", "Medicines"; Entertainment -> "Movie tickets", "Netflix subscription"; Shopping -> "Clothing", "Electronics"; Other -> "Miscellaneous", "Gift", "Donation")
   - amount: a random realistic INR amount (roughly ₹50-₹3000, 2 decimal places)
   - date: a random day (1-28) within the given month/current year
3. Inserts each expense into the expenses table using the same get_db() pattern found in db.py.
4. Prints a confirmation table with id, amount, category, description, and date for each inserted expense.
