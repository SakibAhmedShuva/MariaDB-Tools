# MariaDB Tools

A collection of Python scripts within a Jupyter Notebook (`MariaDB-Tools.ipynb`) designed to facilitate common data operations with a MariaDB database. It provides functionalities for downloading table data into CSV files or Pandas DataFrames, and uploading CSV data into MariaDB tables.

## Table of Contents

- [Features](#features)
- [Prerequisites](#prerequisites)
- [Setup](#setup)
  - [1. Clone Repository](#1-clone-repository)
  - [2. Create a Virtual Environment (Recommended)](#2-create-a-virtual-environment-recommended)
  - [3. Install Dependencies](#3-install-dependencies)
  - [4. Configure Database Connection](#4-configure-database-connection)
- [Usage](#usage)
  - [Running the Notebook](#running-the-notebook)
  - [Core Functions](#core-functions)
    - [Connecting to the Database](#connecting-to-the-database)
    - [Downloading Data](#downloading-data)
      - [List Available Tables](#list-available-tables)
      - [Download Table(s) to CSV](#download-tables-to-csv)
      - [Download Table(s) to Pandas DataFrame(s)](#download-tables-to-pandas-dataframes)
    - [Uploading Data](#uploading-data)
      - [Upload CSV to Database Table](#upload-csv-to-database-table)
- [Code Structure](#code-structure)
- [Important Notes](#important-notes)
- [Contributing](#contributing)
- [License](#license)

## Features

* **Database Connection:** Securely connect to a MariaDB instance using credentials stored in a `.env` file.
* **Data Download:**
  * List all available tables in the connected database.
  * Download specific tables, a list of tables, or all tables.
  * Export data to CSV files.
  * Load data directly into Pandas DataFrames.
* **Data Upload (from CSV):**
  * Upload data from a CSV file to a specified MariaDB table.
  * **Table Handling:**
    * Automatically create the table if it doesn't exist, inferring column types from the CSV data.
    * Options for handling existing tables: `replace` (drop and recreate), `append` (add data), or `fail`.
  * **Large File Support:** Process CSV files in chunks to manage memory usage for large datasets.
  * **Column Name Sanitization:** Cleans CSV column names to be SQL-friendly (alphanumeric and underscores).
  * Handles `NaN` values by converting them to `NULL` for database insertion.

## Prerequisites

* Python 3.7+
* Access to a MariaDB server instance.
* `pip` for installing Python packages.
* Jupyter Notebook or JupyterLab (if you want to run the `.ipynb` file interactively).

## Setup

### 1. Clone Repository

```bash
git clone https://github.com/SakibAhmedShuva/MariaDB-Tools.git
cd MariaDB-Tools
```

### 2. Create a Virtual Environment (Recommended)

```bash
python -m venv venv

# On Windows
venv\Scripts\activate

# On macOS/Linux
source venv/bin/activate
```

### 3. Install Dependencies

The project relies on several Python libraries. Install them using the provided requirements.txt file (if it exists) or install them manually:

```bash
pip install mysql-connector-python pandas python-dotenv numpy jupyterlab
```

If a requirements.txt file is added to the repository, you can simply run:

```bash
pip install -r requirements.txt
```

### 4. Configure Database Connection

The scripts use a `.env` file to store database connection details.

1. Create a file named `.env` in the root directory of the project.
2. Add your MariaDB connection details to this file. **Never commit your `.env` file to version control if it contains sensitive credentials.**

Example `.env` content:

```env
DB_HOST=your_mariadb_host
DB_USERNAME=your_username
DB_PASS=your_password
DB_NAME=your_database_name
DB_PORT=3306
DB_CHARSET=utf8mb4
DB_COLLATION=utf8mb4_general_ci
```

Ensure `.env` is listed in your `.gitignore` file:

```gitignore
.env
venv/
__pycache__/
*.csv
downloaded_db_tables_csv/
```

## Usage

### Running the Notebook

1. Ensure your virtual environment is activated and you have configured your `.env` file.
2. Start JupyterLab or Jupyter Notebook:

```bash
jupyter lab
# or
# jupyter notebook
```

3. Open the `MariaDB-Tools.ipynb` file.
4. You can run the cells defining the functions first, and then use the example usage cells or your own custom code in new cells.

### Core Functions

The notebook is divided into two main sections: Download Utilities and Upload Utilities.

#### Connecting to the Database

Both utility sections use helper functions (`get_db_config`, `create_connection`) to establish a connection based on the `.env` file. These are typically called internally by the main download/upload functions.

#### Downloading Data

The primary functions for downloading are `list_tables()` and `download_data()`.

##### List Available Tables

```python
# (Assuming the function definitions from the notebook are loaded)
# from your_module import list_tables # If you were to refactor to .py files

available_tables = list_tables()
if available_tables:
    print("Available tables:", available_tables)
```

##### Download Table(s) to CSV

```python
# Download a single table to CSV
TABLE_NAME_SINGLE = "my_data_table"
CSV_OUTPUT_DIR = "downloaded_csvs"
success_single_csv = download_data(TABLE_NAME_SINGLE, output_dir=CSV_OUTPUT_DIR, output_format='csv')
print(f"Single table CSV download: {'Success' if success_single_csv else 'Failed'}")

# Download multiple tables to CSV
TABLE_NAMES_MULTIPLE = ["table1", "table2"]
success_multiple_csv = download_data(TABLE_NAMES_MULTIPLE, output_dir=CSV_OUTPUT_DIR, output_format='csv')
print(f"Multiple tables CSV download: {'Success' if success_multiple_csv else 'Failed'}")

# Download all tables to CSV
ALL_TABLES_CSV_DIR = "downloaded_csvs/all_tables"
success_all_csv = download_data('all', output_dir=ALL_TABLES_CSV_DIR, output_format='csv')
print(f"All tables CSV download: {'Success' if success_all_csv else 'Failed'}")
```

##### Download Table(s) to Pandas DataFrame(s)

```python
# Download a single table to a Pandas DataFrame
TABLE_NAME_SINGLE = "my_data_table"
df_single = download_data(TABLE_NAME_SINGLE, output_format='pandas')
if df_single is not None:
    print(f"DataFrame for '{TABLE_NAME_SINGLE}' loaded with {len(df_single)} rows.")
    # print(df_single.head())

# Download multiple specified tables to a dictionary of DataFrames
TABLE_NAMES_MULTIPLE = ["table1", "table2"]
dict_of_dfs_multiple = download_data(TABLE_NAMES_MULTIPLE, output_format='pandas')
if dict_of_dfs_multiple:
    for name, df in dict_of_dfs_multiple.items():
        print(f"DataFrame '{name}' loaded with {len(df)} rows.")
        # print(df.head())

# Download all tables to a dictionary of DataFrames
dict_of_dfs_all = download_data('all', output_format='pandas')
if dict_of_dfs_all:
    for name, df in dict_of_dfs_all.items():
        print(f"DataFrame '{name}' (all) loaded with {len(df)} rows.")
        # print(df.head())
```

#### Uploading Data

##### Upload CSV to Database Table

The primary function for uploading is `upload_csv_to_db()`.

```python
# (Assuming the function definitions from the notebook are loaded)
CSV_FILE_PATH = "path/to/your/data.csv"
DB_TABLE_NAME = "new_target_table"

# Example 1: Replace table if exists, process full file
success_upload_replace = upload_csv_to_db(
    csv_file_path=CSV_FILE_PATH,
    table_name=DB_TABLE_NAME,
    if_exists='replace'  # Options: 'replace', 'append', 'fail'
)
print(f"Upload (replace): {'Success' if success_upload_replace else 'Failed'}")

# Example 2: Append to table if exists, process in chunks
success_upload_append_chunked = upload_csv_to_db(
    csv_file_path=CSV_FILE_PATH,
    table_name=DB_TABLE_NAME,
    if_exists='append',
    chunk_size=10000  # Process 10,000 rows at a time
)
print(f"Upload (append, chunked): {'Success' if success_upload_append_chunked else 'Failed'}")

# Example 3: Fail if table exists
success_upload_fail = upload_csv_to_db(
    csv_file_path=CSV_FILE_PATH,
    table_name=DB_TABLE_NAME,
    if_exists='fail'
)
print(f"Upload (fail if exists): {'Success' if success_upload_fail else 'Failed'}")
```

## Code Structure

The `MariaDB-Tools.ipynb` notebook is organized into cells containing:

* **Imports:** Necessary libraries (`os`, `mysql.connector`, `pandas`, `dotenv`, `numpy`).
* **Download Utilities:**
  * `get_db_config()`: Reads connection parameters from `.env`.
  * `create_connection()`: Establishes and returns a database connection.
  * `list_tables()`: Public function to list tables (manages its own connection).
  * `_list_all_tables_internal()`: Helper to list tables using an existing connection.
  * `_download_single_table_to_df_internal()`: Helper to fetch a single table into a DataFrame.
  * `download_data()`: Main public function for downloading data to CSV or Pandas.
  * Example usage cells for download functions.
* **Upload Utilities:**
  * (Re-imports and re-definition of `get_db_config` and `create_connection` - could be refactored for DRYness if moved to .py modules).
  * `upload_csv_to_db()`: Main public function for uploading CSV data.
  * Example usage cell for the upload function.

## Important Notes

* **Security:** Always ensure your `.env` file with database credentials is not committed to your Git repository. Add `.env` to your `.gitignore` file.
* **Error Handling:** The scripts include basic error handling, printing messages to the console upon connection failures or query errors. Review console output for diagnostics.
* **Column Type Inference (Upload):** The `upload_csv_to_db` function attempts to infer SQL column types by sampling the CSV. For complex datasets or specific requirements, you might need to pre-define the table schema in MariaDB or adjust the type inference logic. The script includes special handling for a column named 'MoveIn' to force it to TEXT, which can be adapted for other specific columns.
* **Large Files:** The `chunk_size` parameter in `upload_csv_to_db` is crucial for handling large CSV files efficiently without running out of memory.
* **Notebook Context:** The code is presented within a Jupyter Notebook. For production use or integration into larger applications, consider refactoring the functions into Python (`.py`) modules.

## Contributing

Contributions are welcome! If you have suggestions for improvements, bug fixes, or new features:

1. Fork the repository.
2. Create a new branch (`git checkout -b feature/YourFeature` or `bugfix/YourBugfix`).
3. Make your changes.
4. Commit your changes (`git commit -m 'Add some feature'`).
5. Push to the branch (`git push origin feature/YourFeature`).
6. Open a Pull Request.

Please ensure your code follows a consistent style and includes clear documentation or comments where necessary.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
