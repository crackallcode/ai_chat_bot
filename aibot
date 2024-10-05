import openai
import pandas as pd
import csv
import ast
import subprocess

openai.api_key = 'your-openai-api-key'

METASPLOIT_PASSWORD = "kali"

# Function to load CSV files
def load_csv(filename):
    try:
        return pd.read_csv(filename, encoding='ISO-8859-1')
    except Exception as e:
        print(f"Error loading CSV {filename}: {e}")
        return None

# Function to print columns of the Exploit-DB CSV for verification
def check_exploitdb_columns(df_exploitdb):
    print("Exploit-DB CSV Columns:", df_exploitdb.columns)

# Function to search across Metasploit, SSTI, and Exploit-DB CSVs
def search_all_csvs(df_metasploit, df_ssti, df_exploitdb, keyword):
    # Existing search for Metasploit CSV
    result_metasploit = df_metasploit[(df_metasploit['Module'].str.contains(keyword, case=False, na=False)) |
                                      (df_metasploit['Description'].str.contains(keyword, case=False, na=False))]
    
    # Existing search for SSTI CSV
    result_ssti = df_ssti[(df_ssti['Platform'].str.contains(keyword, case=False, na=False)) |
                          (df_ssti['Category'].str.contains(keyword, case=False, na=False)) |
                          (df_ssti['Payload'].str.contains(keyword, case=False, na=False)) |
                          (df_ssti['Description'].str.contains(keyword, case=False, na=False))]

    # Existing search for Exploit-DB CSV
    df_exploitdb['SEARCH'] = df_exploitdb['SEARCH'].astype(str)
    result_exploitdb = df_exploitdb[df_exploitdb['SEARCH'].str.contains(keyword, case=False, na=False)]

    # Add code here to search your new CSV (e.g., xxs.csv)
    # Example:
    # result_xxs = df_xxs[(df_xxs['SomeColumn'].str.contains(keyword, case=False, na=False))]

    extracted_titles = []
    if not result_exploitdb.empty:
        for row in result_exploitdb['RESULTS_EXPLOIT']:
            try:
                exploits_list = ast.literal_eval(row)
                for exploit in exploits_list:
                    extracted_titles.append(exploit.get('Title', 'No Title'))
            except (ValueError, SyntaxError):
                continue

    # Return results from all CSVs, including your new one (e.g., result_xxs)
    return result_metasploit, result_ssti, extracted_titles

# Function to display search results
def display_results(results, columns=None, is_exploitdb=False, color_code=None):
    if is_exploitdb:
        if len(results) == 0:
            print("No matching results found.")
            return
        for title in results:
            print(f"Title: {title}\n")
    else:
        if results.empty:
            print("No matching results found.")
        else:
            for _, row in results[columns].iterrows():
                output = " | ".join(map(str, row.values)) + "\n"
                if color_code:
                    print(f"\033[{color_code}m{output}\033[0m")
                else:
                    print(output)

def run_searchsploit(keyword):
    try:
        result = subprocess.run(['searchsploit', keyword], capture_output=True, text=True)
        print(result.stdout)
    except Exception as e:
        print(f"Error running searchsploit: {e}")

# Function to query OpenAI GPT API
def ask_openai_gpt(query):
    try:
        response = openai.ChatCompletion.create(
            model="gpt-4", 
            messages=[
                {"role": "system", "content": "You are an AI assistant helping a pentester in a chatbot api can you respond in less than 5 sentances."},
                {"role": "user", "content": query}
            ]
        )
        ai_response = response['choices'][0]['message']['content']
        
        print(f"\033[95m{ai_response}\033[0m")
        
        return ai_response
    except Exception as e:
        print(f"Error with OpenAI API: {e}")
        return None

# Unified chatbot function with AI prompt
def unified_chatbot(df_metasploit, df_ssti, df_exploitdb):
    print("Welcome to the Unified Metasploit, SSTI, Exploit-DB Chatbot, and Searchsploit! Type 'exit' to quit.")
    while True:
        query = input("Enter a keyword to search (or type 'exit' to quit): ")
        if query.lower() == 'exit':
            break

        # Ask if the user wants to use AI for query interpretation
        use_ai = input("Would you like AI to interpret your query? (y/n): ").lower()

        if use_ai == 'y':
            ai_query = ask_openai_gpt(f"Search for vulnerabilities related to: {query}")
            if ai_query:
                print(f"AI interpreted your query as: {ai_query}")
            else:
                print("AI failed to interpret the query.")
        
            continue  

        # Perform search across all CSVs (including your new CSV if added)
        result_metasploit, result_ssti, result_exploitdb_titles = search_all_csvs(df_metasploit, df_ssti, df_exploitdb, query)
        
        # Display results from Metasploit
        print("\nMetasploit Results:")
        display_results(result_metasploit, ['Module', 'Description'], color_code='32')
        
        # Display results from SSTI
        print("\nSSTI Results:")
        display_results(result_ssti, ['Category', 'Platform', 'Payload', 'Description'], color_code='34')
        
        # Display results from Exploit-DB
        print("\nExploit-DB Results (Titles):")
        display_results(result_exploitdb_titles, is_exploitdb=True)

        # Add code here to display results from your new CSV (e.g., xxs.csv)
        # Example:
        # print("\nXXS CSV Results:")
        # display_results(result_xxs, ['Column1', 'Column2'], color_code='36')

        # Display Searchsploit results
        print("\nSearchsploit Results:")
        run_searchsploit(query)

# Main function to load CSV files and start the chatbot
def main():
    # Load Metasploit CSV
    df_metasploit = load_csv('metasploit_data.csv')
    
    # Load SSTI CSV
    df_ssti = load_csv('ssti_payloads_full.csv')
    
    # Load Exploit-DB CSV
    df_exploitdb = load_csv('exploitdb.csv')

    # Add code here to load your new CSV (e.g., xxs.csv)
    # Example:
    # df_xxs = load_csv('xxs.csv')

    # Check if Exploit-DB was loaded and display sample data
    if df_exploitdb is not None:
        df_exploitdb['SEARCH'] = df_exploitdb['SEARCH'].astype(str)
        df_exploitdb['SEARCH'].replace('nan', '', inplace=True)
        print("First 10 entries of the 'SEARCH' column in Exploit-DB:")
        print(df_exploitdb['SEARCH'].head(10))
        print("First 5 rows of Exploit-DB CSV data:")
        print(df_exploitdb.head(5))
        check_exploitdb_columns(df_exploitdb)

    # Ensure all required datasets are loaded before starting the chatbot
    if df_metasploit is not None and df_ssti is not None and df_exploitdb is not None:
        unified_chatbot(df_metasploit, df_ssti, df_exploitdb)

if __name__ == "__main__":
    main()
