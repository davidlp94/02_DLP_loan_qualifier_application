# Loan Qualifier Application

Hello and welcome to my Loan Qualifier Application project! This project is about helping individuals seek financial loans from qualified banks using the information provided at the time of application. The reason why this project hhas been created is to mitigate the problem of unneccessary loan applications that may hurt an individual's credit score. When using this Loan Qualifier Application project, you are able to narrow down a list of banks that you are qualified to apply for; hence reducing the amount of applications needed to find the right bank and loan for your need! This application also allows the user to save the qualifying loans to a CSV file that can be shared as a spreadsheet. Please see below for more information on how this application works.

---

## Technologies

The following technology/software was used for this application:


-Python 3.7.11 (Programming language)

    Imported Python Libraries:
  
    -sys
    
    -fire
    
    -questionary
    
    -Path from pathlib
    
-Visual Studio Code v1.63.2 (Script writing software)

-Git version 2.34.11.windows.1

-Windows 10 Operating System

---

## Installation Guide

To install and run this application on your machine, download all files contained in this repository davidlp94/loan_qualifier_application_02_DLP. Open GitBash (Windows) or Terminal (MacOS), change the directory to the folder containing the app.py file and run the python script as shown below and follow the CLI.

```
python app.py
```

---

## Usage

This Loan Qualifier Application contains the following files/directories:
```
data/
qualifier/filters
qualifier/utils
app.py
README.md
```
The data/ directory contains a CSV file that shows qualifying loan data parameters for each Bank listed

The qualifier/filters directory contains (4) different modules named credit_score.py, debt_to_income.py, loan_to_value.py and max_loan_size.py.

The qualifier/utils directory contains (2) different files named calculators.py and fileio.py.

The app.py is a Python script written in Visual Studio Code which houses the main code for this application.

The README.md file is an overview of the application.

The first portion of the app.py script imports the necessary dependencies (libraries and modules) for this application to run.

```
import sys
import fire
import questionary
from pathlib import Path

from qualifier.utils.fileio import load_csv

from qualifier.utils.calculators import (
    calculate_monthly_debt_ratio,
    calculate_loan_to_value_ratio,
)

from qualifier.filters.max_loan_size import filter_max_loan_size
from qualifier.filters.credit_score import filter_credit_score
from qualifier.filters.debt_to_income import filter_debt_to_income
from qualifier.filters.loan_to_value import filter_loan_to_value
```

Once all the necessary dependencies are imported, the following functions are defined:

load_bank_data():

This function runs a CLI using questionary to ask the user for a file path to load a CSV file containing Bank qualifying loan parameters.
```
def load_bank_data():
    """Ask for the file path to the latest banking data and load the CSV file.

    Returns:
        The bank data from the data rate sheet CSV file.
    """

    csvpath = questionary.text("Enter a file path to a rate-sheet (.csv):").ask()
    csvpath = Path(csvpath)
    if not csvpath.exists():
        sys.exit(f"Oops! Can't find this path: {csvpath}")

    return load_csv(csvpath)
```

get_applicant_info():

This function runs a CLI using questionary and asks the user for thier financial application information:

```
def get_applicant_info():
    """Prompt dialog to get the applicant's financial information.

    Returns:
        Returns the applicant's financial information.
    """

    credit_score = questionary.text("What's your credit score?").ask()
    debt = questionary.text("What's your current amount of monthly debt?").ask()
    income = questionary.text("What's your total monthly income?").ask()
    loan_amount = questionary.text("What's your desired loan amount?").ask()
    home_value = questionary.text("What's your home value?").ask()

    credit_score = int(credit_score)
    debt = float(debt)
    income = float(income)
    loan_amount = float(loan_amount)
    home_value = float(home_value)

    return credit_score, debt, income, loan_amount, home_value
```

find_qualifying_loans(bank_data, credit_score, debt, income, loan, home_value):

This function determines which loans the user qualifies for by inputting the returned values from the above get_applicant_info function into the calculators.py module to calculate the monthly debt ratio and loan to value ratio which then prints a statement. This function then takes the inputted data and filters the qualified banks by running the filters modules: credit_score.py, debt_to_income.py, loan_to_value.py and max_loan.py in the qualifier/filters directory; which then outputs and returns a list of banks willing to undertake the loan.

```
def find_qualifying_loans(bank_data, credit_score, debt, income, loan, home_value):
    """Determine which loans the user qualifies for.

    Loan qualification criteria is based on:
        - Credit Score
        - Loan Size
        - Debit to Income ratio (calculated)
        - Loan to Value ratio (calculated)

    Args:
        bank_data (list): A list of bank data.
        credit_score (int): The applicant's current credit score.
        debt (float): The applicant's total monthly debt payments.
        income (float): The applicant's total monthly income.
        loan (float): The total loan amount applied for.
        home_value (float): The estimated home value.

    Returns:
        A list of the banks willing to underwrite the loan.

    """

    # Calculate the monthly debt ratio
    monthly_debt_ratio = calculate_monthly_debt_ratio(debt, income)
    print(f"The monthly debt to income ratio is {monthly_debt_ratio:.02f}")

    # Calculate loan to value ratio
    loan_to_value_ratio = calculate_loan_to_value_ratio(loan, home_value)
    print(f"The loan to value ratio is {loan_to_value_ratio:.02f}.")

    # Run qualification filters
    bank_data_filtered = filter_max_loan_size(loan, bank_data)
    bank_data_filtered = filter_credit_score(credit_score, bank_data_filtered)
    bank_data_filtered = filter_debt_to_income(monthly_debt_ratio, bank_data_filtered)
    bank_data_filtered = filter_loan_to_value(loan_to_value_ratio, bank_data_filtered)

    print(f"Found {len(bank_data_filtered)} qualifying loans")

    return bank_data_filtered
```

save_qualifying_loans(qualifying_loans):

Using questionary, this function prompts the user if they would like to save the qualified loans to a CSV file. If the user prompts yes, a CLI prompt asks the user to input a file_path to save the CSV file to and prints a statement. If the user prompts 'no', a difirent statement is printed.

```
def save_qualifying_loans(qualifying_loans):
    """Saves the qualifying loans to a CSV file.

    Args:
        qualifying_loans (list of lists): The qualifying bank loans.
    """
    # @TODO: Complete the usability dialog for saving the CSV Files.
    confirmation = questionary.confirm("Would you like to save these qualified loans to a CSV file?").ask()
    if confirmation:
        file_path = questionary.text("Where would you like to save this CSV file?").ask()
        print(f"No problem, a CSV file has been created and saved in {file_path}.")
    else:
        print("A CSV file will not be created.")
```

run():

Finally, the last function is the mainframe of this application, which takes the above defined functions and is ran when this pyton script is executed in a terminal. To make sure all CLIs are executed properly, we need to call the Python Fire package which accepts the name of the function we would like to run, in this case is 'run'.

```
def run():
    """The main function for running the script."""

    # Load the latest Bank data
    bank_data = load_bank_data()

    # Get the applicant's information
    credit_score, debt, income, loan_amount, home_value = get_applicant_info()

    # Find qualifying loans
    qualifying_loans = find_qualifying_loans(
        bank_data, credit_score, debt, income, loan_amount, home_value
    )

    # Save qualifying loans
    save_qualifying_loans(qualifying_loans)



if __name__ == "__main__":
    fire.Fire(run)
```

---

## Contributors

David Lee Ping

email: davidleeping@gmail.com
Phone: 570.269.5973
LinkedIn: TBD

---

## License

MIT License

Copyright (c) [2022] [David Lee Ping]

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
